---
id: jest-platform
title: Jest 플랫폼
---

Jest의 특정 기능을 체리픽하여 독립형 패키지로 사용할 수 있습니다. 사용 가능한 패키지 목록은 다음과 같습니다:

## jest-changed-files

git/hg 저장소에서 수정된 파일을 식별하기 위한 도구입니다. 두 가지 기능을 내보냅니다:

- `getChangedFilesForRoots`는 변경된 파일과 저장소를 가진 객체로 리졸브 되는 프로미스를 반환합니다.
- `findRepos`는 특정 경로에 포함된 저장소 세트로 리졸브 되는 프로미스를 반환합니다.

### Example

```javascript
const {getChangedFilesForRoots} = require('jest-changed-files');

// 현재 저장소의 마지막 커밋 이후 수정된 파일 세트를 출력
getChangedFilesForRoots(['./'], {
  lastCommit: true,
}).then(result => console.log(result.changedFiles));
```

[readme 파일](https://github.com/facebook/jest/blob/master/packages/jest-changed-files/README.md)에서 `jest-changed-files`에 대해 더 많은 내용을 볼 수 있습니다.

## jest-diff

데이터 변화를 시각화하는 도구. 모든 형식의 두 값을 비교하고 두 인자 사이의 차이를 나타내는 "예쁘게 출력된" 문자열을 반환하는 함수를 내보냅니다.

### Example

```javascript
const diff = require('jest-diff');

const a = {a: {b: {c: 5}}};
const b = {a: {b: {c: 6}}};

const result = diff(a, b);

// print diff
console.log(result);
```

## jest-docblock

자바스크립트 파일의 최 상단에 잇는 주석을 추출하고 해석하는 도구. 주석 블럭 내의 데이터를 조작하기 위해 다양한 함수를 내보냅니다.

### Example

```javascript
const {parseWithComments} = require('jest-docblock');

const code = `
/**
 * This is a sample
 *
 * @flow
 */

 console.log('Hello World!');
`;

const parsed = parseWithComments(code);

// 두 속성(attribute)을 가진 객체 출력: 주석과 지시자
console.log(parsed);
```

[readme 파일](https://github.com/facebook/jest/blob/master/packages/jest-docblock/README.md)에서 `jest-docblock`에 대한 더 많은 내용을 볼 수 있습니다.

## jest-get-type

자바스크립트 값의 기본 유형(primitive type)을 식별하는 모듈. 인자로 전달된 값의 형식을 문자열로 반환하는 함수를 내보냅니다.

### Example

```javascript
const getType = require('jest-get-type');

const array = [1, 2, 3];
const nullValue = null;
const undefinedValue = undefined;

// 'array'를 출력
console.log(getType(array));
// 'null'을 출력
console.log(getType(nullValue));
// 'undefined'를 출력
console.log(getType(undefinedValue));
```

## jest-validate

사용자에 의해 제츌된 구성을 검증하는 도구. 두 개의 인자: 사용자의 구성과 예제 구성과 다른 옵션들을 포함하는 객체를 취하는 함수를 내보냅니다. 반환 값은 두 속성(attribute)를 가진 객체입니다:

- `hasDeprecationWarnings`, 제출된 구성이 중단 된 기능이라는 경고를 가지는지의 여부를 나타내는 불리언,
- `isValid`, 구성이 올바른지 여부를 나타내는 불리언.

### Example

```javascript
const {validate} = require('jest-validate');

const configByUser = {
  transform: '<rootDir>/node_modules/my-custom-transform',
};

const result = validate(configByUser, {
  comment: '  Documentation: http://custom-docs.com',
  exampleConfig: {transform: '<rootDir>/node_modules/babel-jest'},
});

console.log(result);
```

[readme 파일](https://github.com/facebook/jest/blob/master/packages/jest-validate/README.md)에서 `jest-validate`에 대한 더 많은 내용을 볼 수 있습니다.

## jest-worker

태스크 병렬화에 사용되는 모듈. Node.js 모듈의 경로를 취하는 `Worker` 클래스를 내보내고 분기 된 프로세스에서 지정된 메서드가 실행이 종료될 때 리졸브 된 프로미스를 반환하여, 클래스 메서드가 있었다면 모듈의 내보낸 메서드를 호출 하게 합니다.

### Example

```javascript
// heavy-task.js

module.exports = {
  myHeavyTask: args => {
    // 오래 실행되는 CPU 집약적인 태스크.
  },
};
```

```javascript
// main.js

async function main() {
  const worker = new Worker(require.resolve('./heavy-task.js'));

  // 다른 인자를 가진 2개의 태스크를 병렬로 실행
  const results = await Promise.all([
    worker.myHeavyTask({foo: 'bar'}),
    worker.myHeavyTask({bar: 'foo'}),
  ]);

  console.log(results);
}

main();
```

[readme 파일](https://github.com/facebook/jest/blob/master/packages/jest-worker/README.md)에서 `jest-worker`에 대한 더 많은 내용을 볼 수 있습니다..

## pretty-format

자바스크립트 값을 사람이 읽을 수 있는 문자열로 변환하는 함수를 내보냅니다. 모든 빌트-인 자바스크립트 형식을 지원하고 사용자 정의 플러그인을 통해 어플리케이션별 유형에 대한 확장을 허용합니다.

### Example

```javascript
const prettyFormat = require('pretty-format');

const val = {object: {}};
val.circularReference = val;
val[Symbol('foo')] = 'foo';
val.map = new Map([['prop', 'value']]);
val.array = [-0, Infinity, NaN];

console.log(prettyFormat(val));
```

[readme 파일](https://github.com/facebook/jest/blob/master/packages/pretty-format/README.md)에서 `pretty-format`에 대한 더 많은 내용을 볼 수 있습니다.
