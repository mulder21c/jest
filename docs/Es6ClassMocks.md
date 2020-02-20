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

Import the mock and the mock method shared by all instances:

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

### Calling [`jest.mock()`](JestObjectAPI.md#jestmockmodulename-factory-options) with the module factory parameter

`jest.mock(path, moduleFactory)` takes a **module factory** argument. A module factory is a function that returns the mock.

In order to mock a constructor function, the module factory must return a constructor function. In other words, the module factory must be a function that returns a function - a higher-order function (HOF).

```javascript
import SoundPlayer from './sound-player';
const mockPlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: mockPlaySoundFile};
  });
});
```

A limitation with the factory parameter is that, since calls to `jest.mock()` are hoisted to the top of the file, it's not possible to first define a variable and then use it in the factory. An exception is made for variables that start with the word 'mock'. It's up to you to guarantee that they will be initialized on time! For example, the following will throw an out-of-scope error due to the use of 'fake' instead of 'mock' in the variable declaration:

```javascript
// Note: this will fail
import SoundPlayer from './sound-player';
const fakePlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: fakePlaySoundFile};
  });
});
```

### Replacing the mock using [`mockImplementation()`](MockFunctionAPI.md#mockfnmockimplementationfn) or [`mockImplementationOnce()`](MockFunctionAPI.md#mockfnmockimplementationoncefn)

You can replace all of the above mocks in order to change the implementation, for a single test or all tests, by calling `mockImplementation()` on the existing mock.

Calls to jest.mock are hoisted to the top of the code. You can specify a mock later, e.g. in `beforeAll()`, by calling `mockImplementation()` (or `mockImplementationOnce()`) on the existing mock instead of using the factory parameter. This also allows you to change the mock between tests, if needed:

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

## In depth: Understanding mock constructor functions

Building your constructor function mock using `jest.fn().mockImplementation()` makes mocks appear more complicated than they really are. This section shows how you can create your own mocks to illustrate how mocking works.

### Manual mock that is another ES6 class

If you define an ES6 class using the same filename as the mocked class in the `__mocks__` folder, it will serve as the mock. This class will be used in place of the real class. This allows you to inject a test implementation for the class, but does not provide a way to spy on calls.

For the contrived example, the mock might look like this:

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

### Mock using module factory parameter

The module factory function passed to `jest.mock(path, moduleFactory)` can be a HOF that returns a function\*. This will allow calling `new` on the mock. Again, this allows you to inject different behavior for testing, but does not provide a way to spy on calls.

#### \* Module factory function must return a function

In order to mock a constructor function, the module factory must return a constructor function. In other words, the module factory must be a function that returns a function - a higher-order function (HOF).

```javascript
jest.mock('./sound-player', () => {
  return function() {
    return {playSoundFile: () => {}};
  };
});
```

**_Note: Arrow functions won't work_**

Note that the mock can't be an arrow function because calling `new` on an arrow function is not allowed in JavaScript. So this won't work:

```javascript
jest.mock('./sound-player', () => {
  return () => {
    // Does not work; arrow functions can't be called with new
    return {playSoundFile: () => {}};
  };
});
```

This will throw **_TypeError: \_soundPlayer2.default is not a constructor_**, unless the code is transpiled to ES5, e.g. by `@babel/preset-env`. (ES5 doesn't have arrow functions nor classes, so both will be transpiled to plain functions.)

## Keeping track of usage (spying on the mock)

Injecting a test implementation is helpful, but you will probably also want to test whether the class constructor and methods are called with the correct parameters.

### Spying on the constructor

In order to track calls to the constructor, replace the function returned by the HOF with a Jest mock function. Create it with [`jest.fn()`](JestObjectAPI.md#jestfnimplementation), and then specify its implementation with `mockImplementation()`.

```javascript
import SoundPlayer from './sound-player';
jest.mock('./sound-player', () => {
  // Works and lets you check for constructor calls:
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: () => {}};
  });
});
```

This will let us inspect usage of our mocked class, using `SoundPlayer.mock.calls`: `expect(SoundPlayer).toHaveBeenCalled();` or near-equivalent: `expect(SoundPlayer.mock.calls.length).toEqual(1);`

### Spying on methods of our class

Our mocked class will need to provide any member functions (`playSoundFile` in the example) that will be called during our tests, or else we'll get an error for calling a function that doesn't exist. But we'll probably want to also spy on calls to those methods, to ensure that they were called with the expected parameters.

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
