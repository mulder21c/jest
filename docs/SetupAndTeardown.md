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

## describe의 실행 순서와 테스트 블럭

Jest는 실제 테스트를 실행하기 _전에_ 테스트 파일의 모든 describe 처리기를 실행합니다. 이것은 describe 블럭 내부보다 `before*`와 `after*` 내에서 설정과 분해를 수행하는 또 다른 이유입니다. describe 블럭이 완료되면, 기본적으로 Jest는 수집 단계에서 만난 순서대로 다음 단계로 순차적으로 모든 테스트를 수행하며, 다음 단계로 이동하기 전에 각각이 완료되고 정리되기를 기다립니다.

다음 실례의 테스트 파일과 출력을 자세히 살펴보세요:

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

## 일반적인 조언

테스트가 실패하는 경우, 가장 먼저 확인해야 할 사항 중 하나는 수행할 테스트가 유일할 경우 테스트가 실패하는가의 여부여야 합니다. Jest에서 단 하나의 테스트만 수행하기 위해, 임시적으로 그 `test` 명령어를 `test.only`로 변경하세요:

```js
test.only('이 테스트는 수행할 유일한 테스트가 될 것입니다', () => {
  expect(true).toBe(false);
});

test('이 테스트는 수행되지 않을 것입니다', () => {
  expect('A').toBe('A');
});
```

규모가 큰 스위트의 일부로 수행될 때 종종 실패하지만 단독으로 수행할 때에는 실패하지 않는 테스트가 있다면, 다른 테스트로부터의 무엇인가가 이 테스트를 간섭한다는 것은 좋은 추측입니다. `beforeEach`으로 일부 공유 상태를 명확하게 하여 종종 이를 고칠 수 있습니다. 일부 공유 상태가 수정되고 있는지 확실하지 않다면 데이터를 기록하는 `beforeEach`를 시도해 볼 수도 있습니다.
