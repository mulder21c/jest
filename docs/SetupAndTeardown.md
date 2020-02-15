---
id: setup-teardown
title: 설정과 분해
---

테스트를 작성하는 동안 종종 테스트가 수행되기 전에 발생할 필요가 있는 설정 작업이 있고, 테스트가 수행 된 이후 발생해야 할 필요가 있는 마무리 작업이 있습니다. Jest는 이를 처리하는 헬퍼 함수들을 제공합니다.

## 많은 테스트를 위한 설정 반복

많은 테스트를 위해 반복적으로 수행될 필요가 있는 작업이 있다면, `beforeEach`오 `afterEach`를 사용할 수 있습니다.

예를 들어, 몇 가지 테스트가 도시의 데이테베이스와 상호작용한다고 가정해보세요. 이 각각의 테스트 전에 호출되어야 하는 메서드 `initializeCityDatabase()`와 각각의 테스트 이후에 호출되어야 하는 메서드 `clearCityDatabase()`가 있습니다. 다음과 같이 할 수 있습니다:

```js
beforeEach(() => {
  initializeCityDatabase();
});

afterEach(() => {
  clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```

`beforeEach`와 `afterEach`는 [테스트는 비동기 코드가 처리할 수 있는](TestingAsyncCode.md) 동일한 방법으로 비동기 코드를 처리할 수 있습니다 - `done` 파라미터를 취하거나 프로미스를 반환시킬 수 있습니다. 예를 들어,  `initializeCityDatabase()`가 데이터베이스가 초기화 될 때 리졸브 된 프로미스를 반환한다면, 그 프로미스를 반환하려고 합니다:

```js
beforeEach(() => {
  return initializeCityDatabase();
});
```

## 일회성 설정

경우에 따라, 파일의 시작 부분에 한 번만 설정할 필요가 있습니다. 설정이 비동기인 경우 특히 귀찮을 수 있으므로, 인라인으로 설정 할 수 없습니다. Jest는 이 상황을 처리하기 위한 `beforeAll`과 `afterAll`을 제공합니다.

예를 들어, `initializeCityDatabase`과 `clearCityDatabase`가 모두 프로미스를 반환하고 도시 데이터베이스가 테스트 사이에서 재사용 될 수 있는 경우, 테스트 코드를 다음과 같이 변경할 수 있습니다:

```js
beforeAll(() => {
  return initializeCityDatabase();
});

afterAll(() => {
  return clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```

## 살펴보기

기본적으로, `before`와 `after` 블럭은 파일의 모든 테스트에 적용됩니다. `describe` 블럭을 사용하여 테스트들을 그룹핑할 수도 있습니다. `describe` 블럭 안에 있을 경우, `before`와 `after` 블럭들은 그 `describe` 블럭 내부의 테스트에만 적용됩니다.

예를 들어, 도시 데이터 베이스 뿐만 아니라 음식 데이터 베이스도 있다고 가정해보세요. 테스트마다 다른 설정을 할 수 있습니다:

```js
// 이 파이의 모든 테스트에 적용됩니다
beforeEach(() => {
  return initializeCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});

describe('matching cities to foods', () => {
  // 이 describe 블럭의 테스트에만 적용됩니다
  beforeEach(() => {
    return initializeFoodDatabase();
  });

  test('Vienna <3 sausage', () => {
    expect(isValidCityFoodPair('Vienna', 'Wiener Schnitzel')).toBe(true);
  });

  test('San Juan <3 plantains', () => {
    expect(isValidCityFoodPair('San Juan', 'Mofongo')).toBe(true);
  });
});
```

최 상위 레벨의 `beforeEach`가 `describe` 블럭 내부의 `beforeEach`이전에 실행되는 것에 주목하세요. 모든 훅의 실행 순서를 보여주는데 도움이 될 수 있습니다.

```js
beforeAll(() => console.log('1 - beforeAll'));
afterAll(() => console.log('1 - afterAll'));
beforeEach(() => console.log('1 - beforeEach'));
afterEach(() => console.log('1 - afterEach'));
test('', () => console.log('1 - test'));
describe('Scoped / Nested block', () => {
  beforeAll(() => console.log('2 - beforeAll'));
  afterAll(() => console.log('2 - afterAll'));
  beforeEach(() => console.log('2 - beforeEach'));
  afterEach(() => console.log('2 - afterEach'));
  test('', () => console.log('2 - test'));
});

// 1 - beforeAll
// 1 - beforeEach
// 1 - test
// 1 - afterEach
// 2 - beforeAll
// 1 - beforeEach
// 2 - beforeEach
// 2 - test
// 2 - afterEach
// 1 - afterEach
// 2 - afterAll
// 1 - afterAll
```

## Order of execution of describe and test blocks

Jest executes all describe handlers in a test file _before_ it executes any of the actual tests. This is another reason to do setup and teardown inside `before*` and `after*` handlers rather than inside the describe blocks. Once the describe blocks are complete, by default Jest runs all the tests serially in the order they were encountered in the collection phase, waiting for each to finish and be tidied up before moving on.

Consider the following illustrative test file and output:

```js
describe('outer', () => {
  console.log('describe outer-a');

  describe('describe inner 1', () => {
    console.log('describe inner 1');
    test('test 1', () => {
      console.log('test for describe inner 1');
      expect(true).toEqual(true);
    });
  });

  console.log('describe outer-b');

  test('test 1', () => {
    console.log('test for describe outer');
    expect(true).toEqual(true);
  });

  describe('describe inner 2', () => {
    console.log('describe inner 2');
    test('test for describe inner 2', () => {
      console.log('test for describe inner 2');
      expect(false).toEqual(false);
    });
  });

  console.log('describe outer-c');
});

// describe outer-a
// describe inner 1
// describe outer-b
// describe inner 2
// describe outer-c
// test for describe inner 1
// test for describe outer
// test for describe inner 2
```

## General Advice

If a test is failing, one of the first things to check should be whether the test is failing when it's the only test that runs. To run only one test with Jest, temporarily change that `test` command to a `test.only`:

```js
test.only('this will be the only test that runs', () => {
  expect(true).toBe(false);
});

test('this test will not run', () => {
  expect('A').toBe('A');
});
```

If you have a test that often fails when it's run as part of a larger suite, but doesn't fail when you run it alone, it's a good bet that something from a different test is interfering with this one. You can often fix this by clearing some shared state with `beforeEach`. If you're not sure whether some shared state is being modified, you can also try a `beforeEach` that logs data.
