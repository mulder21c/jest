---
id: timer-mocks
title: 타이머 모의
---

네이티브 타이머 함수(예를 들어,  `setTimeout`, `setInterval`, `clearTimeout`, `clearInterval`)는 실시간 경과에 의존되기 때문에 테스트 환경에 이상적이지 않습니다. Jest는 시간의 흐름을 제어할 수 있는 함수로 타이머를 교체할 수 있습니다. [이럴 수가!](https://www.youtube.com/watch?v=QZoJ2Pt27BY)

```javascript
// timerGame.js
'use strict';

function timerGame(callback) {
  console.log('Ready....go!');
  setTimeout(() => {
    console.log("Time's up -- stop!");
    callback && callback();
  }, 1000);
}

module.exports = timerGame;
```

```javascript
// __tests__/timerGame-test.js
'use strict';

jest.useFakeTimers();

test('waits 1 second before ending the game', () => {
  const timerGame = require('../timerGame');
  timerGame();

  expect(setTimeout).toHaveBeenCalledTimes(1);
  expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 1000);
});
```

여기에서 `jest.useFakeTimers();`를 호출하여 페이크 타이머를 활성화 합니다. 이것은 setTimeout과 다른 타이머 함수를 모의 함수로 대체합니다. 하나의 파일 또는 describe 블럭 안에서 여러 테스트가 수행하는 경우, `jest.useFakeTimers();`는 수동으로 또는 `beforeEach` 같은 설정 기능을 사용하여 각 테스트 전에 호출 될 수 있습니다. 그렇게 하지 않으면 내부 사용 카운터가 재설정 되지 않을 것입니다.

## 모든 타이머 실행

이 모듈에 대해 재작성하기 원할 수 있는 다른 테스트는 1초 후에 콜백이 호출되도록 하는 단언입니다. 이를 위해, 테스트 중간에 시간을 빨기 감기 위한 Jest의 타이머 제어 API를 사용할 것입니다:

```javascript
test('calls the callback after 1 second', () => {
  const timerGame = require('../timerGame');
  const callback = jest.fn();

  timerGame(callback);

  // 이 시점에는, 콜백은 아직 호출되지 않아야 합니다
  expect(callback).not.toBeCalled();

  // 모든 타이머가 실행될때까지 빨리감기
  jest.runAllTimers();

  // 이제 콜백이 호출되어야합니다!
  expect(callback).toBeCalled();
  expect(callback).toHaveBeenCalledTimes(1);
});
```

## 대기 타이머 실행

재귀 타이머가 있을 수 있는 시나리오도 있습니다 -- 자체 콜백에서 새로운 타이머가 설정되는 타이머 입니다. 이를 위해, 모든 타이머를 실행하는 것은 무한 루프가 될 수 있... 따라서 `jest.runAllTimers()` 같은 것은 바람직하지 않습니다. 이 경우 `jest.runOnlyPendingTimers()`를 사용할 수 있습니다:

```javascript
// infiniteTimerGame.js
'use strict';

function infiniteTimerGame(callback) {
  console.log('Ready....go!');

  setTimeout(() => {
    console.log("Time's up! 10 seconds before the next game starts...");
    callback && callback();

    // 10초 안에 다음 게임이 예정됩니다
    setTimeout(() => {
      infiniteTimerGame(callback);
    }, 10000);
  }, 1000);
}

module.exports = infiniteTimerGame;
```

```javascript
// __tests__/infiniteTimerGame-test.js
'use strict';

jest.useFakeTimers();

describe('infiniteTimerGame', () => {
  test('schedules a 10-second timer after 1 second', () => {
    const infiniteTimerGame = require('../infiniteTimerGame');
    const callback = jest.fn();

    infiniteTimerGame(callback);

    // 이 시점에, 1초 안에 게임 종료 일정을 잡기 위한
    // setTimeout에 대해 단일 호출이 있어야 합니다.
    expect(setTimeout).toHaveBeenCalledTimes(1);
    expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 1000);

    // 현재 대기 타이머만 빨리감아 고갈시킵니다
    // (그 프로세스 동안 생성되는 새로운 타이머는 없습니다)
    jest.runOnlyPendingTimers();

    // 이 시점에, 1초 타이머가 콜백을 발생시켜야 합니다
    expect(callback).toBeCalled();

    // 그리고 10초 안에 게임을 시작할 수 있는 새로운 타이머가
    // 생성되어야 합니다
    expect(setTimeout).toHaveBeenCalledTimes(2);
    expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 10000);
  });
});
```

## 시간별 진행 타이머

##### Jest **22.0.0**에서 `runTimersToTime`에서 `advanceTimersByTime`로 이름이 변경되었습니다

또 다른 가능성은 `jest.advanceTimersByTime(msToRun)`의 사용입니다. 이 API가 호출 될 경우, 모든 타이머는 `msToRun` 밀리초씩 진행됩니다. setTimeout()이나 setInterval()을 통해 대기되고 이 시간 프레임 동안 실행되어야 하는 모든 대기 중인 "macro-tasks"가 실행될 것입니다. 추가적으로 그 마이크로 태스트가 동일한 시간 프레임에 실행되어야 하는 새로운 마이크로 태스크를 예약하는 경우, msToRun 밀리초 내에 실행되어야 하는 대기열 내의 마이크로 태스크가 더 이상 없을 때 까지 실행될 것입니다.

```javascript
// timerGame.js
'use strict';

function timerGame(callback) {
  console.log('Ready....go!');
  setTimeout(() => {
    console.log("Time's up -- stop!");
    callback && callback();
  }, 1000);
}

module.exports = timerGame;
```

```javascript
it('calls the callback after 1 second via advanceTimersByTime', () => {
  const timerGame = require('../timerGame');
  const callback = jest.fn();

  timerGame(callback);

  // 이 시점에, 콜백은 아직 호출되지 않아야 합니다
  expect(callback).not.toBeCalled();

  // 모든 타이머가 실행될때까지 빨기감기
  jest.advanceTimersByTime(1000);

  // 이제 콜백이 호출 되어야 합니다!
  expect(callback).toBeCalled();
  expect(callback).toHaveBeenCalledTimes(1);
});
```

마지막으로, 가끔 일부 테스트에는 모든 대기 중인 타이머를 지울 수 있는 것이 유용할 수 있습니다. 이를 위해 `jest.clearAllTimers()`이 있습니다.

이 예에 대한 코드는  [examples/timer](https://github.com/facebook/jest/tree/master/examples/timer)에서 사용 가능합니다.
