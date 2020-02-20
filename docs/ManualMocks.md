---
id: manual-mocks
title: 수동 모의
---

수동 모의는 모의 데이터를 기능적으로 추출하는데 사용됩니다. 예를 들어, 웹사이트나 데이터베이스 같은 원격 리소스에 접근하는 대신 가짜 데이터를 사용하도록 수동 모의를 생성하기를 원할 수 있습니다. 이것은 테스트를 빠르고 고장이 많지 않게 합니다.

## 사용자 모듈 모의하기

수동 모의는 모듈 바로 옆 `__mocks__/` 서브디렉토리에 작성하여 정의됩니다. 예를 들어, `models` 디렉토리 내의 `user` 라는 모듈을 모의하려면, `user.js` 라는 파일을 생성하여 `models/__mocks__` 디렉토리에 넣으세요. `__mocks__` 폴더가 대소문자를 구별하기 때문에, 디렉토리를 `__MOCKS__` 으로 이름을 짓는 것은 일부 시스템에서는 깨진다는 것에 주의하세요.

> 테스트에 그 모듈이 필요한 경우, 명시적으로 `jest.mock('./moduleName')`를 호출하는 것이 **필요합니다**.

## Node 모듈 모의하기

모의하는 모듈이 Node 모듈인 경우 (예를 들어: `lodash`), 모의는 (프로젝트 루트가 아닌 폴더를 가리키도록 [`roots`](Configuration.md#roots-arraystring)를 설정하지 않은 한) `node_modules`에 인접한 `__mocks__` 디렉토리에 위치해야 하고 **자동으로** 모의될 것입니다. 명시적으로 `jest.mock('module_name')`를 호출 할 필요는 없습니다.

범위가 지정된 모듈은 범위가 지정된 모듈의 이름과 일치하는 디렉토리 구조에 파일을 생성하여 모의될 수 있습니다. 예를 들어, `@scope/project-name`라는 범위가 지정된 모듈을 모의하려면, `@scope/` 디렉토리에 맞춰 생성하여 `__mocks__/@scope/project-name.js`에 파일을 생성하세요.

> 주의: Node의 코어 모듈을 (예를 들어: `fs`나 `path`) 모의하려면,  명시적으로 호출 하는 것 예를 들어 코어 Node 모듈은 기본적으로 모의되지 않기 때문에 `jest.mock('path')`이 **필수**입니다.

## Examples

```bash
.
├── config
├── __mocks__
│   └── fs.js
├── models
│   ├── __mocks__
│   │   └── user.js
│   └── user.js
├── node_modules
└── views
```

수동 모의가 주어진 모듈에 대해 존재하는 경우, Jest의 모듈 시스템은 명시적으로 `jest.mock('moduleName')`를 호출할 때 그 모듈을 사용합니다. 하지만, `automock`이 `true`로 설정된 경우, `jest.mock('moduleName')`가 호출되지 않는다하더라도, 수동 모의 구현이 자동으로 생성된 모의 대신 사용될 것입니다. 이 동작에서 손을 떼려면 실제 모듈 구현을 사용해야 하는 테스트에서 `jest.unmock('moduleName')`를 명시적으로 호출해야 할 것입니다.

> 참고: 적절한 모의를 위해, Jest는 `require/import` 구문과 동일한 범위에 있으려면 `jest.mock('moduleName')`이 필요합니다.

다음은 주어진 디렉터리의 모든 파일의 요약을 제공하는 모듈이 있는 인위적인 예입니다. 이 경우 코어(내장) `fs` 모듈을 사용합니다.

```javascript
// FileSummarizer.js
'use strict';

const fs = require('fs');

function summarizeFilesInDirectorySync(directory) {
  return fs.readdirSync(directory).map(fileName => ({
    directory,
    fileName,
  }));
}

exports.summarizeFilesInDirectorySync = summarizeFilesInDirectorySync;
```

테스트가 실제로 디스크에 타격을 주는 것을 (꽤 느려지고 깨지기 쉽습니다) 방지하고자 하기 때문에, 자동 모의를 확장하여 `fs` 모듈에 대한 수동 모의를 생성합니다. 수동 모의는 테스트를 기반으로하는 `fs` API의 사용자 정의 버전을 구현할 것입니다:

```javascript
// __mocks__/fs.js
'use strict';

const path = require('path');

const fs = jest.genMockFromModule('fs');

// 이것은 "mock" 파일시스템에서 파일이 `fs` API가 사용될 때
// 무엇처렴 보여야 하는지를 명시하기 위해
// 설정 중에 사용할 수 있는 사용자 정의 함수입니다.
let mockFiles = Object.create(null);
function __setMockFiles(newMockFiles) {
  mockFiles = Object.create(null);
  for (const file in newMockFiles) {
    const dir = path.dirname(file);

    if (!mockFiles[dir]) {
      mockFiles[dir] = [];
    }
    mockFiles[dir].push(path.basename(file));
  }
}

// __setMockFiles를 통해 설정된
// 특정 모의 파일 리스트로부터 읽는
// `readdirSync`의 사용자 정의 버전
function readdirSync(directoryPath) {
  return mockFiles[directoryPath] || [];
}

fs.__setMockFiles = __setMockFiles;
fs.readdirSync = readdirSync;

module.exports = fs;
```

이제 테스트를 작성합니다. 코어 Node 모듈이기 때문에 `fs` 모듈을 모의하기를 원한다고 명시적으로 전달할 필요가 있음에 주목하세요:

```javascript
// __tests__/FileSummarizer-test.js
'use strict';

jest.mock('fs');

describe('listFilesInDirectorySync', () => {
  const MOCK_FILE_INFO = {
    '/path/to/file1.js': 'console.log("file1 contents");',
    '/path/to/file2.txt': 'file2 contents',
  };

  beforeEach(() => {
    // 각 테스트 전에 모의 파일 정보를 설정하세요
    require('fs').__setMockFiles(MOCK_FILE_INFO);
  });

  test('includes all files in the directory in the summary', () => {
    const FileSummarizer = require('../FileSummarizer');
    const fileSummary = FileSummarizer.summarizeFilesInDirectorySync(
      '/path/to',
    );

    expect(fileSummary.length).toBe(2);
  });
});
```

여기서 예시 모의는 자동 모의를 생성하기 위해 [`jest.genMockFromModule`](JestObjectAPI.md#jestgenmockfrommodulemodulename)를 사용하고, 기본 동작을 오버라이드 합니다. 이는 권장되는 접근 방법이지만, 온전히 선택적입니다. 자동 모의를 전혀 사용하지 않으려면, 모의 파일로부터 자체 함수를 내보낼 수 있습니다. 완전 수동 모의에 대한 한 가지 단점은 그것이 수동이라는 것입니다 - 즉, 변경 사항을 모의하는 모듈이 있을 때 마다 수동으로 업데이트 해야 합니다. 이 때문에, 필요에 따라 자동 모의를 사용하거나 확장하는 것이 가장 좋습니다.

수동 모의와 그것의 실제 구현이 동기화 상태를 이루도록, 수동 모의에 [`jest.requireActual(moduleName)`](JestObjectAPI.md#jestrequireactualmodulename)을 사용하고 그것을 내보내기 전에 모의 함수로 갱신하여 실제 모듈을 요구하는 것이 유용할 수 있습니다.

이 예제 코드는 [examples/manual-mocks](https://github.com/facebook/jest/tree/master/examples/manual-mocks)에서 사용 가능 합니다.

## ES 모듈 imports 사용하기

[ES 모듈 imports](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)을 사용하고 있다면, 일반적으로 테스트 파일의 최 상위에 `import` 구문을 두는 경향이 있습니다. 하지만 종종 모듈이 그것을 사용하기 이전에 모의가 사용하도록 Jest에 지시해야 할 필요가 있습니다. 이 이유로, Jest는 자동으로 `jest.mock` 호출을 모듈의 처음으로 (imports 이전으로) 호이스트 할 것입니다. 이에 대해 더 자세히 학습하고 실제로 그것을 보려면, [이 저장소](https://github.com/kentcdodds/how-jest-mocking-works)를 참고하세요.

## JSDOM에 구현되지 않은 모의 메서드

일부 코드는 아직 구현되지 않은 JSDOM (Jest에 의해 사용되는 DOM 구현) 메서드를 사용하는 경우, 테스트하는 것은 쉽지 않습니다. 이것은 예를 들어 `window.matchMedia()`와 같은 예입니다. Jest는 `TypeError: window.matchMedia is not a function`를 반환하고 테스트를 올바르게 실행하지 않습니다.

이 경우, 테스트 파일의 `matchMedia`를 모의 하는 것이 이슈를 해결할 수 있습니다:

```js
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(), // deprecated
    removeListener: jest.fn(), // deprecated
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```

`window.matchMedia()`이 테스트에서 호출되는 함수에 (또는 메서드에) 사용되는 경우 동작합니다. `window.matchMedia()`가 테스트 된 파일에 직접적으로 실행되는 경우, Jest는 동일한 오류를 보고합니다. 이 경우, 해결 방법은 별도의 파일로 수동 모의를 옮기고 테스트 된 파일보다 *이전* 테스트에 포함시키는 것입니다:

```js
import './matchMedia.mock'; // 테스트 된 파일 이전에 import되어야 합니다
import {myMethod} from './file-to-test';

describe('myMethod()', () => {
  // 여기에서 메서드를 테스트 하세요...
});
```
