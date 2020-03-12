---
id: troubleshooting
title: 문제 해결
---


아이쿠, 무언가 잘못 되었나요? Jest 관련 문제를 해결하려면 이 지침을 사용하세요.

## 테스트가 실패하고 있고 이유를 모르는 경우

Node에 내장 된 디버깅 지원을 사용해보세요. 참고: 이것은 Node.js 8+에서만 동작할 것입니다.

테스트 중 하나에 `debugger;` 구문을 배치하고, 프로젝트 디렉토리에서 다음을 실행하세요:

```bash
node --inspect-brk node_modules/.bin/jest --runInBand [다른 인수들을 여기에]
또는 윈도우라면
node --inspect-brk ./node_modules/jest/bin/jest.js --runInBand [다른 인수들을 여기에]
```

이것은 외부 디버거가 연결 할 수 있는 Node 프로세스에서 Jest를 실행합니다. 디버거가 연결될 때 까지 프로세스가 일시중지 될 것입니다.

구글 크롬에서 디버그 하려면 (또는 크로미움 기반 브라우저), 브라우저를 열고 `chrome://inspect`로 이동하여 연결 할 수 있는 사용 가능한 노드 인스턴스 목록을 제공하는 "노드 전용 개발자도구 열기"을 클릭하세요. 터미널에 표기된 주소를 클릭 한 후 (일반적으로 `localhost:9229`와 같은 것입니다) 위 명령을 수행하면 크롬 개발자도구를 사용하여 Jest를 디버그 할 수 있습니다.

크롬 개발자 도구가 표시되고, Jest CLI 스크립트의 첫 번째 라인에 중단점이 설정될 것입니다. (이것은 개발자 도구를 열 수 있는 시간을 주고 열기 전에 Jest가 실행되지 않도록 하기 위해 행해집니다.) 실행을 계속 하려면 화면의 우측 상단에 있는 "재생" 버튼과 같이 보이는 버튼을 클릭하세요. Jest가 `debugger` 구문을 포함하는 테스트를 실행할 경우, 실행이 일시 중지되고 현재 범위와 호출 스택을 검사할 수 있습니다.

> 참고: `--runInBand` cli 옵션은 개별 테스트를 위한 프로세스를 만드는 것이 아니라 동일한 프로세스에서 Jest가 테스트를 실행하도록 합니다. 일반적으로 Jest는 프로세스를 걸쳐 벼열로 테스트를 수행하지만 동시에 여러 프로세스를 디버그 하는 것은 어렵습니다.

## VS 코드에서 디버깅하기

[비주얼 스튜디오 코드](https://code.visualstudio.com)의 내장 [디버거](https://code.visualstudio.com/docs/nodejs/nodejs-debugging)를 사용하여 Jest 테스트를 디버그 하는 여러 방법이 있습니다.

내장 디버거를 연결하려면, 앞서 설명한 대로 테스트를 실행하세요:

```bash
node --inspect-brk node_modules/.bin/jest --runInBand [다른 인수들을 여기에]
또는 윈도우라면
node --inspect-brk ./node_modules/jest/bin/jest.js --runInBand [다른 인수들을 여기에]
```

그 다음 다음 `launch.json` 구성을 사용하여  VS 코드의 디버거를 연결하세요:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "attach",
      "name": "Attach",
      "port": 9229
    }
  ]
}
```

자동으로 시작시키고 테스트를 수행하는 프로세스에 연결하려면, 다음 구성을 사용하세요:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Jest Tests",
      "type": "node",
      "request": "launch",
      "runtimeArgs": [
        "--inspect-brk",
        "${workspaceRoot}/node_modules/.bin/jest",
        "--runInBand"
      ],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen",
      "port": 9229
    }
  ]
}
```

또는 윈도우라면 다음을 사용하세요:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Jest Tests",
      "type": "node",
      "request": "launch",
      "runtimeArgs": [
        "--inspect-brk",
        "${workspaceRoot}/node_modules/jest/bin/jest.js",
        "--runInBand"
      ],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen",
      "port": 9229
    }
  ]
}
```

페이스북의 [`create-react-app`](https://github.com/facebookincubator/create-react-app)를 사용하고 있다면, 다음 구성으로 Jest 테스트를 디버그 할 수 있습니다:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug CRA Tests",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/react-scripts",
      "args": ["test", "--runInBand", "--no-cache", "--env=jsdom"],
      "cwd": "${workspaceRoot}",
      "protocol": "inspector",
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
```

Node 디버깅에 대한 더 자세한 정보는 [여기](https://nodejs.org/api/debugger.html)에서 찾을 수 있습니다.

## 웹스톰에서 디버깅하기

[웹스톰](https://www.jetbrains.com/webstorm/)에서 Jest 테스트를 디버그하는 가장 쉬운 방법은 `Jest run/debug 구성`을 사용하는 것입니다.

웹스톰 메뉴 `Run`에서 `Edit Configurations...`를 선택하세요. 그 다음 `+`를 클릭하고 `Jest`를 선택하세요. 선택적으로 Jest 구성 파일, 추가 옵션, 환경 변수들을 지정하세요. 구성을 저장하고 코드에 중단점을 넣은 다음 디버깅을 시작하도록 녹색 디버그 아이콘을 클릭하세요.

페이스북의 [`create-react-app`](https://github.com/facebookincubator/create-react-app)를 사용하고 있다면, Jest run/debug 구성에서 Jest 패키지 필드의 `react-scripts` 패키지로 경로를 지정하고 Jest 옵션 필드에 `--env=jsdom`를 추가하세요.

## 캐싱 이슈

변환 스크립트가 변경되었거나 Babel이 업데이트 되었고 변경이 Jest에서 인식되지 않습니까?

[`--no-cache`](CLI.md#--cache)를 가지고 다시 시도해보세요. Jest는 테스트 실행 속도를 올리기 위해 변환된 모듈 파일들을 캐시합니다. 사용자 정의 변환기를 사용하고 있따면, `getCacheKey` 함수를 추가하는 것을 고려하세요: [getCacheKey in Relay](https://github.com/facebook/relay/blob/58cf36c73769690f0bbf90562707eadb062b029d/scripts/jest/preprocessor.js#L56-L61)

## 리졸브되지 않은 프로미스

프로미스가 전혀 리졸브되지 않는다면, 이 오류가 발생 될 수 있습니다:

```bash
- Error: Timeout - Async callback was not invoked within timeout specified by jasmine.DEFAULT_TIMEOUT_INTERVAL.`
```

가장 일반적으로 이것은 충돌하는 프로미스 구현으로 인해 발생됩니다. 전역 프로미스 구현을 자신의 것, 예를 들어 `global.Promise = jest.requireActual('promise');`으로 변경하고/또는 사용된 프로미스 라이브러리들을 단일 라이브러리로 통합하는 것을 고려해보세요.

테스트가 오래 실행되는 경우, `jest.setTimeout`을 호출하여 시간 초과를 늘리는 것을 고려할 수도 있습니다

```js
jest.setTimeout(10000); // 10 second timeout
```

## 감시자 이슈

[`--no-watchman`](CLI.md#--watchman)를 가지고 Jest를 수행하거나 `watchman` 구성 옵션을 `false`로 설정해보세요.

[감시자 문제 해결](https://facebook.github.io/watchman/docs/troubleshooting.html)도 참조하세요.

## Docker에서와/또는 지속적 통합(CI) 서버에서 테스트가 매우 느립니다.

Jest가 빠른 SSD를 가진 최신 멀티 코어 컴퓨터에서는 대부분 매우 빠르지만, 사용자들이 [발견](https://github.com/facebook/jest/issues/1524#issuecomment-260246008) [한](https://github.com/facebook/jest/issues/1395) 것과 같은 특정 설정에서 느릴 수 있습니다.

[조사 결과](https://github.com/facebook/jest/issues/1524#issuecomment-262366820)를 바탕으로, 이 문제들을 완화시키고 속도를 50%까지 향상시키는 한 가지 방법은 테스트를 순차적으로 실행하는 것입니다.

이를 위해 [`--runInBand`](CLI.md#--runinband)을 사용하여 동일한 스레드에서 테스트를 수행할 수 있습니다:

```bash
# Jest CLI 사용
jest --runInBand

# yarn 테스트 사용 (예를 들어 create-react-app과 함께)
yarn test --runInBand
```

Travis-CI와 같은 지속적 통함 서버에서 테스트 실행 시간을 신속하게 하기 위한 다른 대안은 최대 워커 풀을 ~_4_로 설정하는 것입니다. 특히 Travis-CI에서, 이 방법은 테스트 실행 시간을 반으로 줄일 수 있습니다. 참고: 오픈 소스 프로젝트에 대해 사용 가능한 Travis-CI _무료_ 요금제는 2개의 CPU 코어만을 포함합니다.

```bash
# Jest CLI 사용
jest --maxWorkers=4

# yarn 테스트 사용 (예를 들어 create-react-app과 함께)
yarn test --maxWorkers=4
```

## 호환성 이슈

Jest는 Node 6에 추가된 새로운 기능의 이점을 가집니다. Node의 최신 안정 버전으로 업그레이드 할 것을 권장합니다. 지원되는 최소 버전은 `v6.0.0`입니다. Jest에 사용된 `jsdom` 버전이 Node 4를 지원하지 않기 때문에 `0.x.x`과 `4.x.x` 버전은 지원되지 않습니다. 하지만, Node 4에서 Jest를 실행해야 한다면,  [`jest-environment-node`](https://yarnpkg.com/en/package/jest-environment-node)과 같이 Node 4를 지원하는 [사용자 정의 환경](https://jestjs.io/docs/en/configuration.html#testenvironment-string)을 사용하도록 `testEnvironment` 구성을 사용할 수 있습니다.

## `coveragePathIgnorePatterns`이 효과가 없는 것 같습니다.

`babel-plugin-istanbul`플러그인을 사용하지 않도록 하세요. Jest는 Istanbul을 감싸므로 Istanbul에 커버리지 컬렉션으로 어떤 파일을 다룰지 알려줍니다. `babel-plugin-istanbul`를 사용하는 경우, Babel에 의해 처리되는 모든 파일은 커버리지 컬렉션을 가지므로 `coveragePathIgnorePatterns`에 의해 무시되지 않습니다.

## 테스트 정의하기

테스트는 Jest가 테스트를 수집할 수 있도록 동기적으로 정의되어야 합니다.

왜 그런지를 보여주는 예로서, 다음과 같이 테스트를 작성했다고 상상해보세요:

```js
// 이렇게 하지 마세요 동작 안합니다
setTimeout(() => {
  it('passes', () => expect(1).toBe(1));
}, 0);
```

Jest가 `test`들을 수집하기 위해 수행될 때 이벤트 루프의 다음 틱(tick)에서 비동기 적으로 발생하도록 정의했기 때문에 아무것도 찾을 수가 없습니다.

_참고:_ 이것은 `test.each`를 사용 중일 경우 `beforeEach` / `beforeAll` 내에서 테이블을 비동기적으로 설정할 수 없다는 것을 의미합니다.

## 여전히 해결되지 않습니까?

[도움말](/help.html)을 참고하세요.
