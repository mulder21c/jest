---
id: asynchronous
title: 비동기 코드 테스트
---

코드가 비동기로 실행되는 것은 자바스크립트에서 일반적입니다. 비동기로 실행되는 코드가 있는 경우, Jest는 다른 테스트로 옮겨가기 이전에, 테스트 중인 코드가 언제 완료되었는지 알아야 할 필요가 있습니다. Jest는 이를 처리하기 위한 몇 가지 방법이 있습니다.

## 콜백

가장 일반적인 비동기 패턴은 콜백입니다.

예를 들어, 일부 데이터를 가져오는 `fetchData(callback)` 함수가 있고 그것이 완료될 때 `callback(data)`를 호출한다고 가정해보세요. 이 반환된 데이터가 문자열 `'peanut butter'` 인지를 테스트 하려고 합니다.

기본 적으로, Jest는 실행의 마지막에 도달하자마자 테스트가 종료됩니다. 이는 이 테스트가 의도한 대로 동작_하지 않을 것_을 의미합니다:

```js
// 이렇게 하지 마세요!
test('the data is peanut butter', () => {
  function callback(data) {
    expect(data).toBe('peanut butter');
  }

  fetchData(callback);
});
```

문제는 콜백을 호출하기도 전에, 테스트는 `fetchData`가 끝나자마자 종료될 것이라는 겁니다.

이를 해결하는 `test`의 대체 형식이 있습니다. 빈 인자를 가진 함수에 테스트를 넣는 대신, `done`이라는 단일 인자를 사용하세요. Jest는 테스트가 끝나기 전에 `done` 콜백이 호출될 때까지 기다릴 것입니다.

```js
test('the data is peanut butter', done => {
  function callback(data) {
    try {
      expect(data).toBe('peanut butter');
      done();
    } catch (error) {
      done(error);
    }
  }

  fetchData(callback);
});
```

`done()`이 절대로 호출되지 않는 경우, 테스트는 실패할 것이고(시간 초과 오류 발생), 이것이 원하는 결과입니다.

`expect` 구문이 실패하는 경우 오류가 발생하고 `done()`은 호출되지 않습니다. 실패한 이유를 테스트 로그에서 보고 싶다면, `expect`를 `try` 블럭으로 감싸고 `catch` 블럭의 오류를 `done`에게 전달해야 합니다. 그렇지 않으면 불투명한 시간 초과 오류로 종료되고 `expect(data)`에 의해 전달받은 값에 대한 정보를 알지 못합니다.

## 프로미스

코드가 프로미스를 사용하는 경우, 비동기 테스트를 처리하는 보다 간단한 방법이 있습니다. 테스트로부터 프로미스를 반환시키면, Jest는 그 프로미스가 리졸브 되기를 기다릴 겁니다. 프로미스가 거부되면 테스트는 자동으로 실패합니다.

예를 들어, 콜백을 사용하는 대신 `fetchData`가 문자열 `'peanut butter'`를 리졸브 하기로 한 프로미스를 반환한다고 가정해보세요. 다음을 이용하여 테스트 할 수 있습니다:

```js
test('the data is peanut butter', () => {
  return fetchData().then(data => {
    expect(data).toBe('peanut butter');
  });
});
```

확실하게 프로미스를 반환하세요 - 이 `return` 구문을 생락한다면, 테스트는 `fetchData`로부터 반환된 프로미스가 리졸브 되고 then()이 콜백을 실행할 기회를 가지기 이전에 종료 될 것입니다.

프로미스가 거절 될 것이 예상되는 경우 `.catch` 메서드를 사용하세요. 특정 번호의 단언이 호출되는지를 확인 하기 위해 `expect.assertions`를 추가하세요. 그렇지 않으면 수행된 프로미스는 테스트를 실패하지 않게 될 겁니다.

```js
test('the fetch fails with an error', () => {
  expect.assertions(1);
  return fetchData().catch(e => expect(e).toMatch('error'));
});
```

## `.resolves` / `.rejects`

expect 구문에 `.resolves` 매처를 사용할 수도 있으며, Jest는 그 프로미스가 리졸브 되기를 기다릴 것입니다. 프로미스가 거부된다면, 테스트는 자동으로 실패할 것입니다.

```js
test('the data is peanut butter', () => {
  return expect(fetchData()).resolves.toBe('peanut butter');
});
```

반드시 단언을 반환시키세요 — 이 `return` 구문을 생략하면, 테스트는 프로미스가 `fetchData`가 리졸브되고 then()이 콜백을 실행할 기회를 가지기 이전에 종료될 것입니다.

프로미스가 거절 될 것이 예상되는 경우 `.rejects` 매처를 사용하세요. 이것은 `.resolves` 매처와 유사하게 동작합니다. 프로미스가 수행되는 경우, 테스트는 자동으로 실패할 것 입니다.

```js
test('the fetch fails with an error', () => {
  return expect(fetchData()).rejects.toMatch('error');
});
```

## Async/Await

또 다른 대안으로, 테스트에 `async`와 `await`를 사용할 수 있습니다. 비동기 테스트를 작성하기 위해, `test`에 전달된 함수의 앞에 `async` 키워드를 사용하세요. 예를 들어, 동일한 `fetchData` 시나리오는 다음과 같이 테스트 될 수 있습니다:

```js
test('the data is peanut butter', async () => {
  const data = await fetchData();
  expect(data).toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  expect.assertions(1);
  try {
    await fetchData();
  } catch (e) {
    expect(e).toMatch('error');
  }
});
```

`async`과 `await`을 `.resolves`와 `.rejects`와 함께 조합할 수도 있습니다.

```js
test('the data is peanut butter', async () => {
  await expect(fetchData()).resolves.toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  await expect(fetchData()).rejects.toThrow('error');
});
```

이 경우에, `async`와 `await`는 프로미스 예제로서 동일한 로직을 사용하는 효과적인 문법 설탕입니다.

이러한 형식 중 어떤 것도 다른 형식들보다 특출나지 않으며, 코드 베이스에 걸쳐서 혹은 단일 파일에서도 목적에 따라 짜 맞출수 있습니다. 어떤 스타일이 테스트를 더 간단하게 느끼게 만드는가에 달려있습니다.
