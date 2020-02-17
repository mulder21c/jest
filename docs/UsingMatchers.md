---
id: using-matchers
title: 매처(matcher) 사용하기
---

Jest는 다른 방법으로 값을 테스트 하도록 "매처"를 사용합니다. 이 문서는 일반적으로 사용되는 매처를 소개할 것입니다. 전체 목록을 보려면, [`expect` API 문서](ExpectAPI.md)를 참조하세요.

## 일반 매처

값을 테스트 하기 위한 가장 단순한 방법은 정확한 일치를 사용하는 것입니다.

```js
test('two plus two is four', () => {
  expect(2 + 2).toBe(4);
});
```

이 코드에서, `expect(2 + 2)`는 "예상" 객체를 반환합니다. 일반적으로 매처를 호출하는 것을 제외하고 이 예상 객체들로 더 이상의 것을 하지 않을 겁니다. 이 코드에서 `.toBe(4)`가 매처 입니다. Jest가 실행 될 때, 모든 실패한 매처를 추적하여 멋진 오류 메세지를 출력해 줄 수 있습니다.

`toBe`는 정확한 등가를 검사하기 위해 `Object.is`를 사용합니다. 객체의 값을 확인하려면, 대신 `toEqual`을 사용하세요:

```js
test('object assignment', () => {
  const data = {one: 1};
  data['two'] = 2;
  expect(data).toEqual({one: 1, two: 2});
});
```

`toEqual`은 객체나 배열의 모든 필드를 재귀적으로 확인합니다.

매처의 반대를 테스트 할 수도 있습니다:

```js
test('adding positive numbers is not zero', () => {
  for (let a = 1; a < 10; a++) {
    for (let b = 1; b < 10; b++) {
      expect(a + b).not.toBe(0);
    }
  }
});
```

## 참

테스트에서 때때로 `undefined`, `null`, `false`를 구별해야 할 필요가 있지만, 이를 별도로 다루는 것을 원치 않을 때도 있습니다. Jest는 원하는 것에 대해 명시적이게 할 수 있는 헬퍼를 포함하고 있습니다.

- `toBeNull`은 `null`에만 일치합니다
- `toBeUndefined`는 `undefined`에만 일치합니다
- `toBeDefined`는 `toBeUndefined`의 반대입니다
- `toBeTruthy`는 `if` 구문이 true로 취급하는 모든 것과 일치합니다
- `toBeFalsy`는 `if` 구문이 false로 취급하는 모든 것과 일치합니다

예를 들어:

```js
test('null', () => {
  const n = null;
  expect(n).toBeNull();
  expect(n).toBeDefined();
  expect(n).not.toBeUndefined();
  expect(n).not.toBeTruthy();
  expect(n).toBeFalsy();
});

test('zero', () => {
  const z = 0;
  expect(z).not.toBeNull();
  expect(z).toBeDefined();
  expect(z).not.toBeUndefined();
  expect(z).not.toBeTruthy();
  expect(z).toBeFalsy();
});
```

코드가 수행하기 원하는 것과 가장 정확하게 일치하는 매처를 사용하는게 좋습니다.

## 숫자

숫자를 비교하는 대부분의 방법은 일치하는 동등한 매처를 가지고 있습니다.

```js
test('two plus two', () => {
  const value = 2 + 2;
  expect(value).toBeGreaterThan(3);
  expect(value).toBeGreaterThanOrEqual(3.5);
  expect(value).toBeLessThan(5);
  expect(value).toBeLessThanOrEqual(4.5);

  // toBe와 toEqual은 숫자에 대해 동등합니다
  expect(value).toBe(4);
  expect(value).toEqual(4);
});
```

부동 소수점 등가의 경우, 테스트가 사소한 반올림 오류에 따라 달라지는 것을 원치 않으므로 `toEqual` 대신 `toBeCloseTo`를 사용하세요.

```js
test('adding floating point numbers', () => {
  const value = 0.1 + 0.2;
  //expect(value).toBe(0.3);           반올림 오류로 동작하지 않을 것입니다
  expect(value).toBeCloseTo(0.3); // 동작합니다
});
```

## 문자열

`toMatch`로 정규식과 비교하여 문자열을 검사할 수 있습니다:

```js
test('there is no I in team', () => {
  expect('team').not.toMatch(/I/);
});

test('but there is a "stop" in Christoph', () => {
  expect('Christoph').toMatch(/stop/);
});
```

## 배열과 이터러블

`toContain`를 사용하여 배열이나 이터러블이 특정 항목을 포함하는지 여부를 확인 할 수 있습니다:

```js
const shoppingList = [
  'diapers',
  'kleenex',
  'trash bags',
  'paper towels',
  'beer',
];

test('the shopping list has beer on it', () => {
  expect(shoppingList).toContain('beer');
  expect(new Set(shoppingList)).toContain('beer');
});
```

## 예외

특정 함수가 호출될 때 오류를 발생시키는지를 테스트하려면 `toThrow`를 사용하세요.

```js
function compileAndroidCode() {
  throw new Error('you are using the wrong JDK');
}

test('compiling android goes as expected', () => {
  expect(compileAndroidCode).toThrow();
  expect(compileAndroidCode).toThrow(Error);

  // 정확한 오류 메세지나 정규식을 사용할 수도 있습니다
  expect(compileAndroidCode).toThrow('you are using the wrong JDK');
  expect(compileAndroidCode).toThrow(/JDK/);
});
```

## 그밖에

이건 단지 맛보기 일뿐입니다. 매처의 전체 목록을 보려면 [참조 문서](ExpectAPI.md)를 확인하세요.

사용 가능한 매처들에 대해 학습한 이후, Jest로 [비동기 코드 검사](TestingAsyncCode.md) 하는 방법을 확인 하는 것이 좋습니다.
