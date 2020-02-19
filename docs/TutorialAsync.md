---
id: tutorial-async
title: 비동기 예제
---

먼저, [Getting Started](GettingStarted.md#using-babel) 가이드의 내용을 따라 Jest에 Babel 지원을 활성화하세요.

API에서 사용자 데이터를 가져오고 사용자 이름을 반환하는 모듈을 구현해 봅시다.

```js
// user.js
import request from './request';

export function getUserName(userID) {
  return request('/users/' + userID).then(user => user.name);
}
```

위 구현에서 `request.js` 모듈이 프로미스를 반환할 것으로 기대합니다. 사용자 이름을 받기 위해 `then`을 호출하도록 연결합니다.

이제 네트워크에 접근하여 사용자 데이터를 가져오는 `request.js`의 구현을 상상해보세요:

```js
// request.js
const http = require('http');

export default function request(url) {
  return new Promise(resolve => {
    // 이것이 API로부터 사용자 데이터를 가져오는 예에 대한
    // HTTP 요청의 예입니다.
    // 이 모듈은 __mocks__/request.js에 모의 되고 있습니다
    http.get({path: url}, response => {
      let data = '';
      response.on('data', _data => (data += _data));
      response.on('end', () => resolve(data));
    });
  });
}
```

테스트를 위해서 네트워크에 접근하는 것은 원치 않기 때문에, `__mocks__` 폴더에 `request.js` 모듈에 대한 수동 모의를 만들 것입니다 (폴더는 대소문자를 구분하며, `__MOCKS__`은 동작하지 않을 것입니다). 이것은 다음과 같이 보일 수 있습니다:

```js
// __mocks__/request.js
const users = {
  4: {name: 'Mark'},
  5: {name: 'Paul'},
};

export default function request(url) {
  return new Promise((resolve, reject) => {
    const userID = parseInt(url.substr('/users/'.length), 10);
    process.nextTick(() =>
      users[userID]
        ? resolve(users[userID])
        : reject({
            error: 'User with ' + userID + ' not found.',
          }),
    );
  });
}
```

이제 비동기 기능에 대한 테스트를 작성해봅시다.

```js
// __tests__/user-test.js
jest.mock('../request');

import * as user from '../user';

// 프로미스에 대한 단언이 반환되어야 합니다.
it('works with promises', () => {
  expect.assertions(1);
  return user.getUserName(4).then(data => expect(data).toEqual('Mark'));
});
```

Jest에게 수동 모의를 사용하도록 요청하는 `jest.mock('../request')`을 호출합니다. `it`는 리졸브 될 프로미스가 되는 값을 반환한 것으로 기대합니다. 마지막에 프로미스를 반환하기만 하면 원하는 만큼 많은 프로미스를 연결하고 언제든지 `expect`를 호출 할 수 있습니다.

## `.resolves`

다른 매처와 함께 수행된 프로미스의 값을 펼치기 위해 `resolves`를 사용하는 덜 장황한 방법이 있습니다. 프로미스가 거부된다면, 단언은 실패할 것입니다.

```js
it('works with resolves', () => {
  expect.assertions(1);
  return expect(user.getUserName(5)).resolves.toEqual('Paul');
});
```

## `async`/`await`

`async`/`await`를 사용하여 테스트를 작성하는 것도 가능합니다. 이전의 동일한 예제를 작성하는 방법은 다음과 같습니다:

```js
// async/await가 사용될 수 있습니다.
it('works with async/await', async () => {
  expect.assertions(1);
  const data = await user.getUserName(4);
  expect(data).toEqual('Mark');
});

// async/await는 또한 `.resolves`와 함께 사용될 수 있습니다.
it('works with async/await and resolves', async () => {
  expect.assertions(1);
  await expect(user.getUserName(5)).resolves.toEqual('Paul');
});
```

프로젝트에 async/await를 활성화 하기 위해, [`@babel/preset-env`](https://babeljs.io/docs/en/babel-preset-env)을 설치하고 `babel.config.js` 파일에 기능을 활성화 시키세요.

## 오류 처리

오류는 `.catch` 메서드를 사용하여 처리될 수 있습니다. 특정 번호의 단언이 호출되는지를 확인하도록 `expect.assertions`을 추가하세요. 그렇지 않으면 수행된 프로미스는 테스트를 실패하지 않을 것입니다:

```js
// Promise.catch를 사용하여 비동기 오류 테스트하기.
it('tests error with promises', () => {
  expect.assertions(1);
  return user.getUserName(2).catch(e =>
    expect(e).toEqual({
      error: 'User with 2 not found.',
    }),
  );
});

// 또는 async/await 사용.
it('tests error with async/await', async () => {
  expect.assertions(1);
  try {
    await user.getUserName(1);
  } catch (e) {
    expect(e).toEqual({
      error: 'User with 1 not found.',
    });
  }
});
```

## `.rejects`

`.rejects` 헬퍼는 `.resolves` 헬퍼 처럼 동작합니다. 프로미스가 수행되면, 테스트는 자동으로 실패할 것입니다. `expect.assertions(number)`가 필수는 아니지만, 테스트 동안 호출되는 특정 번호의 [assertions](https://jestjs.io/docs/en/expect#expectassertionsnumber)을 확인 하기 위해 권장됩니다. 그렇지 않으면 `.resolves` 단언을 `return`/`await` 하는 것을 잊어버리기 쉽습니다.

```js
// `.rejects`를 사용하여 비동기 오류 테스트 하기.
it('tests error with rejects', () => {
  expect.assertions(1);
  return expect(user.getUserName(3)).rejects.toEqual({
    error: 'User with 3 not found.',
  });
});

// 또는 `.rejects`와 함께 async/await 사용.
it('tests error with async/await and rejects', async () => {
  expect.assertions(1);
  await expect(user.getUserName(3)).rejects.toEqual({
    error: 'User with 3 not found.',
  });
});
```

이 예제의 코드는 [examples/async](https://github.com/facebook/jest/tree/master/examples/async)에서 사용 가능 합니다.

`setTimeout` 같은 타이머를 테스트하고 싶다면, [타이머 모의](TimerMocks.md) 문서를 살펴보세요.
