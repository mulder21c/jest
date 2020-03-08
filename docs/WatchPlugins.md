---
id: watch-plugins
title: 감시(watch) 플러그인
---

Jest 감시 플러그인 시스템은 Jest의 특정 부분에 연결하고 키 누름에서 코드를 실행하는 감시 모드 메뉴 프롬프트를 정의하는 방법을 제공합니다. 이러한 기능을 결합하면 워크플로우에 대한 맞춤형 대화형 경험을 발전시킬 수 있습니다.

## 감시 플러그인 인터페이스

```javascript
class MyWatchPlugin {
  // Jest 라이프사이클 이벤트에 연결 추가
  apply(jestHooks) {}

  // 대화형 플러그인에 대한 프롬프트 정보 얻기
  getUsageInfo(globalConfig) {}

  // `getUsageInfo`으로부터 키를 입력했을 때 실행됨
  run(globalConfig, updateConfigAndRun) {}
}
```

## Jest로 연결하기

감시 플러그인을 Jest로 연결하려면, Jest 구성에 `watchPlugins` 아래에 해당 경로를 추가하세요:

```javascript
// jest.config.js
module.exports = {
  // ...
  watchPlugins: ['path/to/yourWatchPlugin'],
};
```

사용자 감시 플러그인은 Jest 이벤트로 훅(hook)을 추가할 수 있습니다. 이 훅은 감시 모드 메뉴에서 대화형 키를 사용하거나 사용하지 않고 추가될 수 있습니다.

### `apply(jestHooks)`

Jest 훅은 `apply` 메서드 구현을 통해 붙일 수 있습니다. 이 메서드는 플러그인을 테스트 실행의 라이프 사이클의 특정 부분에 뎐결 할 수 있도록 하는 `jestHooks` 인자를 수신합니다.

```javascript
class MyWatchPlugin {
  apply(jestHooks) {}
}
```

다음은 Jest에서 사용 가능한 훅입니다.

#### `jestHooks.shouldRunTestSuite(testSuiteInfo)`

테스트가 수행되어야 하는지 아닌지를 지정하는 불리언을 (또는 비동기 연산 처리를 위한 `Promise<boolean>`를) 반환합니다.

예를 들어:

```javascript
class MyWatchPlugin {
  apply(jestHooks) {
    jestHooks.shouldRunTestSuite(testSuiteInfo => {
      return testSuiteInfo.testPath.includes('my-keyword');
    });

    // 또는 프로미스
    jestHooks.shouldRunTestSuite(testSuiteInfo => {
      return Promise.resolve(testSuiteInfo.testPath.includes('my-keyword'));
    });
  }
}
```

#### `jestHooks.onTestRunComplete(results)`

모든 테스트 수행이 끝날 때 호출됩니다. 테스트 결과를 인수로 사용합니다.

예를 들어:

```javascript
class MyWatchPlugin {
  apply(jestHooks) {
    jestHooks.onTestRunComplete(results => {
      this._hasSnapshotFailure = results.snapshot.failure;
    });
  }
}
```

#### `jestHooks.onFileChange({projects})`

파일 시스템에 변경이 있을 때마다 호출됩니다.

- `projects: Array<config: ProjectConfig, testPaths: Array<string>`: Jest가 감시하는 모든 테스트 경로를 포함합니다.

예를 들어:

```javascript
class MyWatchPlugin {
  apply(jestHooks) {
    jestHooks.onFileChange(({projects}) => {
      this._projects = projects;
    });
  }
}
```

## 감시 메뉴 통합

사용자정의 감시 플러그인은 `getUsageInfo` 메서드와 키의 실행에 대한 `run` 메서드에 키/프롬프트 쌍을 지정하여 감시 메뉴에 기능을 추가하거나 오버라이드할 수도 있습니다.

### `getUsageInfo(globalConfig)`

감시 메뉴에 키를 추가하려면, 키와 프롬프트를 반환하여 `getUsageInfo` 메서드를 구현하세요:

```javascript
class MyWatchPlugin {
  getUsageInfo(globalConfig) {
    return {
      key: 's',
      prompt: 'do something',
    };
  }
}
```

감시 모드 메뉴에 _(`› Press s to do something.`)_ 라인이 추가 될 것입니다

```text
Watch Usage
 › Press p to filter by a filename regex pattern.
 › Press t to filter by a test name regex pattern.
 › Press q to quit watch mode.
 › Press s to do something. // <-- 이게 우리 플러그인입니다
 › Press Enter to trigger a test run.
```

**Note**: 플러그인에 대한 키가 이미 디폴트 키로 존재하고 있다면, 플러그인이 해당 키를 재정의 할 것입니다.

### `run(globalConfig, updateConfigAndRun)`

`getUsageInfo`에 의해 반환된 키로부터 키 누름 이벤트를 처리하려면, `run` 메서드를 구현할 수 있습니다. 이 메서드는 플러그인이 Jest로 제어권을 반환하기 원할 때 리졸브 될 수 있는 `Promise<boolean>`를 반환합니다. `boolean`은 Jest가 제어권을 돌려받은 이후 테스트를 재실행 해야 하는지를 정의합니다.

- `globalConfig`: Jest의 현재 전역 구성 표현
- `updateConfigAndRun`: 대화형 플러그인이 수행되는 동안 테스트 실행을 트리거 할 수 있음

```javascript
class MyWatchPlugin {
  run(globalConfig, updateConfigAndRun) {
    // do something.
  }
}
```

**Note**: `updateConfigAndRun`를 호출한다면, `run` 메서드는 truthy 값은 중복 실행을 유발하기 때문에 truthy 값으로 리졸브 되지 않아야 합니다.

#### 인증 된 구성 키

안정성과 안전성의 이유로, 전역 구성 키의 일부만이 `updateConfigAndRun`로 업데이트 될 수 있습니다. 현재 화이트리스트는 다음과 같습니다:

- [`bail`](configuration.html#bail-number--boolean)
- [`changedSince`](cli.html#--changedsince)
- [`collectCoverage`](configuration.html#collectcoverage-boolean)
- [`collectCoverageFrom`](configuration.html#collectcoveragefrom-array)
- [`collectCoverageOnlyFrom`](configuration.html#collectcoverageonlyfrom-array)
- [`coverageDirectory`](configuration.html#coveragedirectory-string)
- [`coverageReporters`](configuration.html#coveragereporters-arraystring)
- [`notify`](configuration.html#notify-boolean)
- [`notifyMode`](configuration.html#notifymode-string)
- [`onlyFailures`](configuration.html#onlyfailures-boolean)
- [`reporters`](configuration.html#reporters-arraymodulename--modulename-options)
- [`testNamePattern`](cli.html#--testnamepatternregex)
- [`testPathPattern`](cli.html#--testpathpatternregex)
- [`updateSnapshot`](cli.html#--updatesnapshot)
- [`verbose`](configuration.html#verbose-boolean)

## 사용자정의

플러그인은 Jest 구성을 통해 사용자정의 될 수 있습니다.

```javascript
// jest.config.js
module.exports = {
  // ...
  watchPlugins: [
    [
      'path/to/yourWatchPlugin',
      {
        key: 'k', // <- 사용자정의 키
        prompt: 'show a custom prompt',
      },
    ],
  ],
};
```

추천 구성 이름::

- `key`: 플러그인 키를 수정합니다.
- `prompt`: 플러그인 프롬프트에 텍스트를 커스트마이즈할 수 있도록 허용합니다.

사용자가 사용자 정의 구성을 제공하는 경우, 플러그인 생성자에 인수로 전달 될 것입니다.

```javascript
class MyWatchPlugin {
  constructor({config}) {}
}
```

## 좋은 키 선택하기

Jest는 서드파티 플러그인이 빌트인 기능 키의 일부를 재정의 하는 것을 허용하지만, 전부는 아닙니다. 특히, 다음 키는 **덮어 쓸 수 없습니다**

- `c` (필터 패턴 정리)
- `i` (일치하지 않는 스냅샷을 대화식으로 업데이트)
- `q` (그만하기)
- `u` (일치하지 않는 모든 스냅샷 업데이트)
- `w` (감시 모드 사용/사용 불가능한 작업 표시)

빌트인 기능에 대한 다음 키는 **덮어 쓸 수 있습니다** :

- `p` (파일 이름 패턴 테스트)
- `t` (이름 패턴 테스트)

빌트인 기능에 의해 사용되지 않는 모든 키는 기대하는 대로 확보할 수 있습니다. 다양한 키보드에서 얻기 어렵거나 (예를 들어 `é`, `€`), 기본적으로 보여지지 않는 키 (예를 들어 많은 Mac 키보드는  `|`, `\`, `[` 등과 같은 문자에 대한 보여지는 힌트가 없습니다)를 사용하지 않도록 하세요.

### 충돌이 발생할 경우

플러그인이 예약된 키를 재정의 하려고 하면, Jest는 다음과 같은 설명 메세지와 함께 오류를 낼 것입니다:

> 감시 플러그인 YourFaultyPlugin이 키를 등록하려고 시도했습니다 <q>, 이 키는 감시 모드를 그만 두도록 내부적으로 예약된 키입니다. 이 플러그인에 대한 구성 키를 변경해주세요.

또한 서드파티 플러그인은 구성된 플러그인 목록 에 (`watchPlugins` 배열 설정) 앞서 존재하는 다른 서드파티 플러그인에 의해 이미 예약된 키를 덮어쓰는 것이 금지되어 있습니다. 이것이 발생하는 경우, 이를 해결하는데 도움이 되는 오류 메세지를 얻게 될 것입니다:

> 감시 플러그인 YourFaultyPlugin과 TheirFaultyPlugin 모두 키 등록을 시도했습니다 <x>. 겹치지 않도록 충돌하는 플러그인 중 하나에 대한 키 구성을 변경해주세요.
