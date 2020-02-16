---
id: mock-functions
title: 모의 함수
---

모의 함수는 함수의 실제 구현을 삭제, 함수 호출을 (그리고 그 호출에 전달된 파라미터) 캡쳐, `new`로 인스트턴스화 될 때 생성자 함수의 인스턴스 캡쳐, 반화 값의 테스트 시간 구성을 허용하여  코드 사이의 연결을 테스트 할 수 있게 해줍니다.

모의 함수를 위한 두 가지 방법이 있습니다: 테스트 코드에 사용할 모의 함수를 생성하거나, 의존선 모듈을 오버라이드 하기 위해 [`manual mock`](ManualMocks.md) 함수를 생성.

## 모의 함수 사용하기

제공된 배열의 각 항목에 대한 콜백을 실행하는 `forEach` 함수의 구현을 테스트 하고 있다고 생각해보세요.

```javascript
function forEach(items, callback) {
  for (let index = 0; index < items.length; index++) {
    callback(items[index]);
  }
}
```

이 함수를 테스트 하기 위해, 모의 함수를 사용하고 콜백이 예상대로 호출 되는지를 확인하기 위해 모의 함수의 상태를 검사할 수 있습니다.

```javascript
const mockCallback = jest.fn(x => 42 + x);
forEach([0, 1], mockCallback);

// 모의 함수가 두 번 호출 됩니다
expect(mockCallback.mock.calls.length).toBe(2);

// 함수에 대한 첫 번째 호출의 첫 번째 인자는 0 이었음
expect(mockCallback.mock.calls[0][0]).toBe(0);

// 함수에 대한 두 번째 호출의 첫 번째 인자는 1 이었음
expect(mockCallback.mock.calls[1][0]).toBe(1);

// 함수에 대한 첫 번째 호출의 반환 된 값은 42 이었음
expect(mockCallback.mock.results[0].value).toBe(42);
```

## `.mock` 속성(property)

모든 모의 함수는 함수가 호출된 방법과 반환된 함수가 보관하고 있는 것에 대한 데이터가 보관된 특별한 `.mock` 속성을 가지고 있습니다. `.mock` 속성은 각 호출에 대한 `this` 값도 추적하므로, 다음 사항을 검사하는 것도 가능합니다:

```javascript
const myMock = jest.fn();

const a = new myMock();
const b = {};
const bound = myMock.bind(b);
bound();

console.log(myMock.mock.instances);
// > [ <a>, <b> ]
```

이 모의 멤버는 테스트에서 이 함수들이 호출되는 방법이나 인스턴스화 되는 방법, 혹은 무엇을 반환하는지를 확고히 하는데 매우 유용합니다:

```javascript
// 함수는 정확히 한 번 호출됩니다
expect(someMockFunction.mock.calls.length).toBe(1);

// 함수에 대한 첫 번째 호출의 첫 번째 인자는 'first arg' 이었음
expect(someMockFunction.mock.calls[0][0]).toBe('first arg');

// 함수에 대한 첫 번째 호출의 두 번째 인자는 'second arg' 이었음
expect(someMockFunction.mock.calls[0][1]).toBe('second arg');

// 함수에 대한 첫 번째 호출의 반환 값은 'return value' 이었음
expect(someMockFunction.mock.results[0].value).toBe('return value');

// 이 함수는 정확히 두 번 인스턴스화 되었음
expect(someMockFunction.mock.instances.length).toBe(2);

// 값이 `test`로 설정 된 `name` 프로퍼티를 가진
// 이 함수의 첫 번째 인스턴스화에 의해 반환된 객체
expect(someMockFunction.mock.instances[0].name).toEqual('test');
```

## 모의 반환 값

모의 함수는 테스트 중에 코드에 테스트 값을 주입할 수도 있습니다:

```javascript
const myMock = jest.fn();
console.log(myMock());
// > undefined

myMock
  .mockReturnValueOnce(10)
  .mockReturnValueOnce('x')
  .mockReturnValue(true);

console.log(myMock(), myMock(), myMock(), myMock());
// > 10, 'x', true, true
```

모의 함수는 연속 전달 스타일을 함수를 사용하는 코드에서 매우 효과적입니다. 이 스타일로 작성된 코드는 사용되기 바로 전에 테스트에 직접 값을 주입시키기 위해 서있는 실제 구성 요소의 동작을 재생성하는 복잡한 스텁에 대한 필요성을 방지하는데 도움이 됩니다.

```javascript
const filterTestFn = jest.fn();

// 모의가 첫 번째 호출에 대해 `true`를 반환하도록 하고,
// 두 번째 호출에 대해 `false`를 반환하게 합니다
filterTestFn.mockReturnValueOnce(true).mockReturnValueOnce(false);

const result = [11, 12].filter(num => filterTestFn(num));

console.log(result);
// > [11]
console.log(filterTestFn.mock.calls);
// > [ [11], [12] ]
```

대부분의 현실 세계 예제는 의존성 컴포넌트에서의 모의 함수를 연결짓고 그것을 구성하여 호출 하지만, 기법은 동일합니다. 이 경우, 직접 테스트 되지 않는 함수 내부에 로직을 구현하려는 유혹을 방지하세요.

## 모의 모듈

API로부터 사용자를 가져오는 클래스가 있다고 가정해보세요. 클래스는 API를 호출하고 이후 모든 사용자를 포함하는 `data` 속성(attribute)를 반환시키기 위해 [axios](https://github.com/axios/axios)를 사용합니다:

```js
// users.js
import axios from 'axios';

class Users {
  static all() {
    return axios.get('/users.json').then(resp => resp.data);
  }
}

export default Users;
```

이제, 실제로 API에 영향을 미치지 않고 이 메서드를 테스트 하기 위해 (그리고 따라서 느리고 취약한 테스트를 생성하여), 자동적으로 axios 모듈을 모의하도록 `jest.mock(...)` 함수를 사용할 수 있습니다.

모듈을 모의 할 때 테스트가 확고히 하기 원하는 데이터를 반환하는 `.get`에 대해 `mockResolvedValue`를 제공할 수 있습니다. 실제로는, axios.get('/users.json')이 가짜 응답을 반환하기 원한다고 표현합니다.

```js
// users.test.js
import axios from 'axios';
import Users from './users';

jest.mock('axios');

test('should fetch users', () => {
  const users = [{name: 'Bob'}];
  const resp = {data: users};
  axios.get.mockResolvedValue(resp);

  // 또는 사용 사례에 따라 다음을 사용할 수 있습니다:
  // axios.get.mockImplementation(() => Promise.resolve(resp))

  return Users.all().then(data => expect(data).toEqual(users));
});
```

## 모의 구현

그럼에도 불구하고, 반환 값을 지정하는 기능을 넘어서 극단적으로 모의 함수의 구현을 대체하는 것이 유용한 경우가 있습니다. 이는 모의 함수에 `jest.fn`나 `mockImplementationOnce`로 수행 될 수 있습니다.

```javascript
const myMockFn = jest.fn(cb => cb(null, true));

myMockFn((err, val) => console.log(val));
// > true
```

`mockImplementation` 메서드는 다른 모듈로부터 생성된 모의 함수의 기본 구현을 정의할 필요가 있을 때 유용합니다:

```js
// foo.js
module.exports = function() {
  // 어떤 구현;
};

// test.js
jest.mock('../foo'); // 이것은 자동모의를 통해 자동적으로 발생합니다
const foo = require('../foo');

// foo가 모의 함수입니다
foo.mockImplementation(() => 42);
foo();
// > 42
```

여러 함수 호출이 다른 결과를 생성하는 것처럼 모의 함수의 복잡한 동작을 재생성해야 하는 경우, `mockImplementationOnce` 메서드를 사용하세요:

```javascript
const myMockFn = jest
  .fn()
  .mockImplementationOnce(cb => cb(null, true))
  .mockImplementationOnce(cb => cb(null, false));

myMockFn((err, val) => console.log(val));
// > true

myMockFn((err, val) => console.log(val));
// > false
```

모의 함수가 `mockImplementationOnce`로 정의된 구현을 소진할 경우, `jest.fn`으로 설정된 (정의가 되어 있다면) 기본 구현이 실행될 것입니다:

```javascript
const myMockFn = jest
  .fn(() => 'default')
  .mockImplementationOnce(() => 'first call')
  .mockImplementationOnce(() => 'second call');

console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
// > 'first call', 'second call', 'default', 'default'
```

일반적으로 체이닝 된 메서드가 있는 (그리고 때문에 항상 `this`를 반환할 필요가 있는) 경우에, 모든 모의에 적용되는 `.mockReturnThis()` 함수의 형식으로 이를 단순화 하는 설탕 API가 있습니다:

```javascript
const myObj = {
  myMethod: jest.fn().mockReturnThis(),
};

// 다음과 동일합니다

const otherObj = {
  myMethod: jest.fn(function() {
    return this;
  }),
};
```

## 모의 이름

선택적으로 테스트 오류 출력에 "jest.fn()" 대신 표기될, 모의 함수에 대한 이름을 제공할 수 있습니다. 테스트 출력에서 오류를 리포트하는 모의 함수를 빠르게 식별하려면 이를 사용하세요.

```javascript
const myMockFn = jest
  .fn()
  .mockReturnValue('default')
  .mockImplementation(scalar => 42 + scalar)
  .mockName('add42');
```

## 사용자 정의 매처

마지막으로, 모의 함수가 어떻게 호출되었는지를 확인하는데 부담을 덜기 위해, 몇 가지 사용자 정의 매처 함수를 추가했습니다:

```javascript
// 모의 함수가 적어도 한 번은 호출되었습니다
expect(mockFunc).toHaveBeenCalled();

// 모의 함수가 특정 인자를 가지고 적어도 한 번은 호출되었습니다
expect(mockFunc).toHaveBeenCalledWith(arg1, arg2);

// 모의 함수에 대한 마지막 호출이 특정 인자를 가지고 호출되었습니다
expect(mockFunc).toHaveBeenLastCalledWith(arg1, arg2);

// 모든 호출과 모의의 이름이 스냅샷으로 기록되었습니다
expect(mockFunc).toMatchSnapshot();
```

이 매처들은 `.mock` 속성(property) 검사의 일반적인 형태를 위한 설탕입니다. 그것이 취향에 더 맞거나 더 구체적인 무언가를 해야 하는 경우 항상 수동으로 이것을 직접 할 수 있습니다:

```javascript
// 모의 함수는 적어도 한 번 호출되었습니다
expect(mockFunc.mock.calls.length).toBeGreaterThan(0);

// 모의 함수가 특정 인자를 가지고 적어도 한 번은 호출되었습니다
expect(mockFunc.mock.calls).toContainEqual([arg1, arg2]);

// 모의 함수에 대한 마지막 호출이 특정 인자를 가지고 호출되었습니다
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1]).toEqual([
  arg1,
  arg2,
]);

// 모의 함수에 대한 마지막 호출의 첫 번째 인자가 `42` 이었습니다
// (이 특정 단언에 대한 설탕 헬퍼가 존재하지 않는 것에 주목하세요)
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1][0]).toBe(42);

// 스냅샷은 모의가 동일한 인자를 가지고 동일한 순서로, 동일한 횟수만큼 호출되었는지
// 확인할 것입니다. 이름에 대해서도 확인할 것입니다.

expect(mockFunc.mock.calls).toEqual([[arg1, arg2]]);
expect(mockFunc.getMockName()).toBe('a mock name');
```

매처의 전체 목록은 [참조 문서](ExpectAPI.md)를 확인하세요.
