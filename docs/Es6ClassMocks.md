---
id: es6-class-mocks
title: ES6 클래스 모의
---

Jest는 테스트 하기 원하는 파일로 가져온 ES6 클래스를 모의하는데 사용될 수 있습니다.

ES6 클래스는 일무 문법 설탕을 가진 생성자 함수입니다. 따라서 ES6 클래스에 대한 모의는 함수나 실제 ES6 클래스(다시 말하지만, 다른 함수) 여야 합니다. 따라서 [모의 함수](MockFunctions.md)를 사용하여 그것들을 모의할 수 있습니다.

## ES6 클래스 예

샤운드 파일을 재생하는 클래스의 인위적인 예 `SoundPlayer`와 이 클래스를 사용하는 소비 클래스 `SoundPlayerConsumer`를 사용할 겁니다. `SoundPlayerConsumer`에 대한 테스트에 `SoundPlayer`를 모의할 것입니다.

```javascript
// sound-player.js
export default class SoundPlayer {
  constructor() {
    this.foo = 'bar';
  }

  playSoundFile(fileName) {
    console.log('Playing sound file ' + fileName);
  }
}
```

```javascript
// sound-player-consumer.js
import SoundPlayer from './sound-player';

export default class SoundPlayerConsumer {
  constructor() {
    this.soundPlayer = new SoundPlayer();
  }

  playSomethingCool() {
    const coolSoundFileName = 'song.mp3';
    this.soundPlayer.playSoundFile(coolSoundFileName);
  }
}
```

## ES6 모의를 생성하는 4가지 방법

### 자동 모의

`jest.mock('./sound-player')`을 호출하는 것은 클래스 생성자와 모든 메서드 호출을 감시하는데 사용할 수 있는 유용한 "자동 모의"를 반환합니다. ES6 클래스를 모의 생성자로 교체하고, 모든 메서드를 항상 `undefined`를 반환하는 [모의 함수](MockFunctions.md)로 교체합니다. 메서드 호출은 `theAutomaticMock.mock.instances[index].methodName.mock.calls`에 저장됩니다.

클래스에 화살표 함수를 사용한다면, 그것들은 모의의 일부가 되지 _않음_을 주의하세요. 그 이유는 화살표 함수는 객체의 프로토타입에 존재하지 않고, 단지 함수에 대한 참조를 보유하는 속성(property)들일 뿐이기 때문입니다.

클래스의 구현을 교체할 필요가 없다면, 이것은 설정을 위한 가장 쉬운 옵션입니다. 예를 들어:

```javascript
import SoundPlayer from './sound-player';
import SoundPlayerConsumer from './sound-player-consumer';
jest.mock('./sound-player'); // SoundPlayer는 이제 모의 생성자입니다

beforeEach(() => {
  // 모든 인스턴스와 생성자와 모든 메서드 호출을 정리합니다:
  SoundPlayer.mockClear();
});

it('We can check if the consumer called the class constructor', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  expect(SoundPlayer).toHaveBeenCalledTimes(1);
});

it('We can check if the consumer called a method on the class instance', () => {
  // mockClear() 이 동작하고 잇는 것을 보여줍니다:
  expect(SoundPlayer).not.toHaveBeenCalled();

  const soundPlayerConsumer = new SoundPlayerConsumer();
  // 생성자는 다시 호출되지 않아야 합니다:
  expect(SoundPlayer).toHaveBeenCalledTimes(1);

  const coolSoundFileName = 'song.mp3';
  soundPlayerConsumer.playSomethingCool();

  // mock.instances는 자동 모의와 사용 가능합니다:
  const mockSoundPlayerInstance = SoundPlayer.mock.instances[0];
  const mockPlaySoundFile = mockSoundPlayerInstance.playSoundFile;
  expect(mockPlaySoundFile.mock.calls[0][0]).toEqual(coolSoundFileName);
  // 위 점검과 동일합니다:
  expect(mockPlaySoundFile).toHaveBeenCalledWith(coolSoundFileName);
  expect(mockPlaySoundFile).toHaveBeenCalledTimes(1);
});
```

### 수동 모의

`__mocks__` 폴더에 모의 구현을 저장하여 [수동 모의](ManualMocks.md)를 생성하세요. 이를 통해 구현을 지정할 수 있고, 테스트 파일 전반에 걸쳐 사용될 수 있습니다.

```javascript
// __mocks__/sound-player.js

// 이 기명 내보내기를 테스트 파일에 가져오세요:
export const mockPlaySoundFile = jest.fn();
const mock = jest.fn().mockImplementation(() => {
  return {playSoundFile: mockPlaySoundFile};
});

export default mock;
```

모든 인스턴스에 의해 공유된 모의와 모의 메서드를 가져오세요:

```javascript
// sound-player-consumer.test.js
import SoundPlayer, {mockPlaySoundFile} from './sound-player';
import SoundPlayerConsumer from './sound-player-consumer';
jest.mock('./sound-player'); // SoundPlayer is now a mock constructor

beforeEach(() => {
  // 모든 인스턴스와 생성자와 모든 메서드 호출을 정리하세요:
  SoundPlayer.mockClear();
  mockPlaySoundFile.mockClear();
});

it('We can check if the consumer called the class constructor', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  expect(SoundPlayer).toHaveBeenCalledTimes(1);
});

it('We can check if the consumer called a method on the class instance', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  const coolSoundFileName = 'song.mp3';
  soundPlayerConsumer.playSomethingCool();
  expect(mockPlaySoundFile).toHaveBeenCalledWith(coolSoundFileName);
});
```

### 모듈 팩토리 파라미터를 가진 [`jest.mock()`](JestObjectAPI.md#jestmockmodulename-factory-options) 호출하기

`jest.mock(path, moduleFactory)`는 **module factory** 인자를 취합니다. 모듈 팩토리는 모의를 반환하는 함수입니다.

생성자 함수를 모의하기 위해, 모듈 팩토리는 생성자 함수를 반환해야 합니다. 다른 말로, 모듈 팩토리는 함수를 반환하는 함수 - 고차 함수(HOF) 여야 합니다.

```javascript
import SoundPlayer from './sound-player';
const mockPlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: mockPlaySoundFile};
  });
});
```

팩토리 파라미터의 한계는 `jest.mock()` 호출이 파일의 최 상단으로 호이스팅 되기 때문에, 먼저 변수를 정의한 다음 팩토리에서 그것을 사용할 수 없다는 것입니다. 예외는 'mock'이라는 단어로 시작하는 변수에 대해 만들어 집니다. 제시간에 초기화 될 것이라고 보장하는 것은 여러분께 달려있습니다! 예를 들어, 다음은 변수 선언에서 'mock' 대신 'fake'의 사용으로 인해 범위를 벗어난 오류를 던질 것입니다:

```javascript
// 주목: 이것은 실패할 것입니다
import SoundPlayer from './sound-player';
const fakePlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: fakePlaySoundFile};
  });
});
```

### [`mockImplementation()`](MockFunctionAPI.md#mockfnmockimplementationfn)나 [`mockImplementationOnce()`](MockFunctionAPI.md#mockfnmockimplementationoncefn)를 사용하여 모의 반환하기

단일 테스트나 모든 테스트에 대해 구현을 변경하기 위해 위의 모든 모의를 기존의 모의에서 `mockImplementation()`를 호출하여 교체할 수 있습니다.

jest.mock 호출은 코드의 최 상단으로 호이스팅 됩니다. 나중에, 예를 들어 팩토리 파라미터를 사용하는 대신 기존 모의에 `mockImplementation()` (또는 `mockImplementationOnce()`)를 호출하여 모의를 지정할 수 있습니다. 필요하다면, 테스트 사이에 모의를 변경하는 것 역시 가능합니다:

```javascript
import SoundPlayer from './sound-player';
import SoundPlayerConsumer from './sound-player-consumer';

jest.mock('./sound-player');

describe('When SoundPlayer throws an error', () => {
  beforeAll(() => {
    SoundPlayer.mockImplementation(() => {
      return {
        playSoundFile: () => {
          throw new Error('Test error');
        },
      };
    });
  });

  it('Should throw an error when calling playSomethingCool', () => {
    const soundPlayerConsumer = new SoundPlayerConsumer();
    expect(() => soundPlayerConsumer.playSomethingCool()).toThrow();
  });
});
```

## 심화: 모의 생성자 함수 이해하기

`jest.fn().mockImplementation()`을 사용하여 생성자 함수 모의를 만드는 것은 모의를 실제 보다 더 복잡한 것 처럼 보여지게 만듭니다. 이 섹션은 모의가 어떻게 동작하는지를 설명하기 위해 자체 모의를 생성할 수 있는 방법을 보여 줍니다.

### 수동 모의는 또 다른 ES6 클래스입니다

`__mocks__` 폴더에 모의된 클래스와 동일한 파일 이름을 사용하는 ES6 클래스를 정의하는 경우, 그것은 모의로써 동작할 것입니다. 이 클래스는 실제 클래스의 위치에서 사용될 것입니다. 이것은 클래스에 대해 테스트 구현을 주입할 수 있게 하지만, 호출에 대한 감시하는 방법을 제공하지 않습니다.

인위적인 예로, 모의는 이것 처럼 보일 수 있습니다:

```javascript
// __mocks__/sound-player.js
export default class SoundPlayer {
  constructor() {
    console.log('Mock SoundPlayer: constructor was called');
  }

  playSoundFile() {
    console.log('Mock SoundPlayer: playSoundFile was called');
  }
}
```

### 모듈 팩토리 파라미터를 사용하여 모의

`jest.mock(path, moduleFactory)`으로 전달 된 모듈 팩토리 함수는 함수를 반환하는 HOF일 수 있습니다\*. 이것은 모의에서 `new` 호출을 허용할 것입니다. 다시 말하지만, 테스트를 위해 다른 동작을 주입하는 것이 가능하기는 하지만, 호출을 감시하는 방법은 제공하지 않습니다.

#### \* 모듈 팩토리 함수는 함수를 반환해야 합니다

생성자 함수를 모의 하기 위해서, 모듈 팩토리는 생성자 함수를 반환해야 합니다. 다시 말해서, 모듈 팩토리는 함수를 반환하는 함수 - 고차 함수 (HOF) 여야 합니다.

```javascript
jest.mock('./sound-player', () => {
  return function() {
    return {playSoundFile: () => {}};
  };
});
```

**_Note: 화살표 함수는 동작하지 않을 것입니다_**

화살표 함수에서 `new`를 호출하는 것은 자바스크립트에서 허용되지 않기 때문에 모의는 화살표 함수가 될 수 없음에 주목하세요. 따라서 이것은 동작하지 않을 것입니다:

```javascript
jest.mock('./sound-player', () => {
  return () => {
    // 동작하지 않습니다; 화살표 함수는 new를 가지고 호출 될 수 없습니다
    return {playSoundFile: () => {}};
  };
});
```

코드가 예를 들어 `@babel/preset-env`에 의해  ES5로 트랜스파일되지 않는 이상, 이것은 **_TypeError: \_soundPlayer2.default is not a constructor_**를 던질 것입니다. (ES5는 애로우 함수나 클래스를 가지지 않기 때문에, 순수 함수로 트랜스파일될 것입니다.)

## 사용 추척 (모의에서 감시하기)

테스트 구현을 주입하는 것은 도움이 되지만, 클래스 생성자와 메서드가 올바른 파리미터와 호출되는지를 테스트 하기 원할 수도 있을 겁니다.

### 생성자 감시

생성자를 호출하는 것을 추적하기 위해, HOF에 의해 반환되는 함수를 Jest 모의 함수로 변경하세요. [`jest.fn()`](JestObjectAPI.md#jestfnimplementation)으로 생성한 다음,  `mockImplementation()`으로 구현을 지정하세요.

```javascript
import SoundPlayer from './sound-player';
jest.mock('./sound-player', () => {
  // 생성자 호출에 대한 작업 및 확인:
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: () => {}};
  });
});
```

`SoundPlayer.mock.calls`: `expect(SoundPlayer).toHaveBeenCalled();`나 거의 등가적인 `expect(SoundPlayer.mock.calls.length).toEqual(1);`를 사용하여 모의 클래스의 사용을 검사하게 할 것입니다.

### 클래스의 메서드 감시

모의된 클래스는 테스트 동안 호출될 멤버 함수를 (예제에서 `playSoundFile`) 제공할 필요가 있거나, 그렇지 않으면 존재하지 않는 함수 호출에 대한 오류를 얻게 될 것입니다. 예상되는 파라미터를 가지고 호출되도록 하기 위해서, 그 메서드 호출도 감시하기 원할 것입니다.

테스트 동안 모의 생성자 함수가 호출 될 때마다 새로운 객체가 생성될 것입니다.
A new object will be created each time the mock constructor function is called during tests. To spy on method calls in all of these objects, we populate `playSoundFile` with another mock function, and store a reference to that same mock function in our test file, so it's available during tests.

```javascript
import SoundPlayer from './sound-player';
const mockPlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: mockPlaySoundFile};
    // Now we can track calls to playSoundFile
  });
});
```

The manual mock equivalent of this would be:

```javascript
// __mocks__/sound-player.js

// Import this named export into your test file
export const mockPlaySoundFile = jest.fn();
const mock = jest.fn().mockImplementation(() => {
  return {playSoundFile: mockPlaySoundFile};
});

export default mock;
```

Usage is similar to the module factory function, except that you can omit the second argument from `jest.mock()`, and you must import the mocked method into your test file, since it is no longer defined there. Use the original module path for this; don't include `__mocks__`.

### Cleaning up between tests

To clear the record of calls to the mock constructor function and its methods, we call [`mockClear()`](MockFunctionAPI.md#mockfnmockclear) in the `beforeEach()` function:

```javascript
beforeEach(() => {
  SoundPlayer.mockClear();
  mockPlaySoundFile.mockClear();
});
```

## Complete example

Here's a complete test file which uses the module factory parameter to `jest.mock`:

```javascript
// sound-player-consumer.test.js
import SoundPlayerConsumer from './sound-player-consumer';
import SoundPlayer from './sound-player';

const mockPlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: mockPlaySoundFile};
  });
});

beforeEach(() => {
  SoundPlayer.mockClear();
  mockPlaySoundFile.mockClear();
});

it('The consumer should be able to call new() on SoundPlayer', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  // Ensure constructor created the object:
  expect(soundPlayerConsumer).toBeTruthy();
});

it('We can check if the consumer called the class constructor', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  expect(SoundPlayer).toHaveBeenCalledTimes(1);
});

it('We can check if the consumer called a method on the class instance', () => {
  const soundPlayerConsumer = new SoundPlayerConsumer();
  const coolSoundFileName = 'song.mp3';
  soundPlayerConsumer.playSomethingCool();
  expect(mockPlaySoundFile.mock.calls[0][0]).toEqual(coolSoundFileName);
});
```
