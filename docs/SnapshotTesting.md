---
id: snapshot-testing
title: 스냅샷 테스팅
---

스냅샷 테스트는 UI가 예상 밖으로 변경되지 않도록 하기 원하는 경우 매우 유용한 도구입니다.

모바일 앱에 대한 일반적인 스냅샷 테스트 케이스는 UI 컴포넌트를 렌더링 하고, 스냅샷을 찍은 다음, 참조 저장된 스냅샷 파일과 테스트를 나란히 비교합니다. 두 스냅샷이 일치하지 않는다면 테스트는 실패할 것입니다: 변경이 예상 밖이거나, 참조 스냅샷이 UI 컴포넌트의 새로운 버전으로 업데이트 될 필요가 있는 경우.

## Jest로 스냅샷 테스트 하기

React 컴포넌트를 테스트 할 때 비슷한 방법을 취할 수 있습니다. 전체 앱을 빌드해야 햐는 그래픽 UI를 렌더링 하는 대신, React 트리에 대해 연속될 수 있는 값을 빠르게 생성하도록 테스트 렌더러를 사용할 수 있습니다. [Link 컴포넌트](https://github.com/facebook/jest/blob/master/examples/snapshot/Link.react.js)에 대한 이 [예제 테스트](https://github.com/facebook/jest/blob/master/examples/snapshot/__tests__/link.react.test.js)를 자세히 살펴보세요:

```javascript
import React from 'react';
import Link from '../Link.react';
import renderer from 'react-test-renderer';

it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="http://www.facebook.com">Facebook</Link>)
    .toJSON();
  expect(tree).toMatchSnapshot();
});
```

이 테스트가 처음 수행될 때, Jest는 이와 같이 보이는 [스냅샷 파일](https://github.com/facebook/jest/blob/master/examples/snapshot/__tests__/__snapshots__/link.react.test.js.snap)을 생성합니다:

```javascript
exports[`renders correctly 1`] = `
<a
  className="normal"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}
>
  Facebook
</a>
`;
```

스냅샷 산출물은 코드 변경과 함께 커밋되어야 하고, 코드 리뷰 절차의 일부로 검토되어야 합니다. Jest는 코드 리뷰 동안 사람이 읽을 수 있는 스냅샷을 만들기 위해 [pretty-format](https://github.com/facebook/jest/tree/master/packages/pretty-format)을 사용합니다. 후속 테스트가 수행 될 때 Jest는 이전 스냅샷과 렌더링 된 출력을 비교할 것입니다. 그것들이 일치한다면, 테스트는 통과할 것입니다. 일치하지 않는다면, 테스트 실행기는 수정되어야 하는 코드 상의 버그를  (이 경우, 그것은 `<Link>` 컴포넌트입니다) 발견했거나, 구현이 변경되어 스냅샷이 업데이트 되어야 할 필요가 있습니다.

> 참고: 스냅샷은 렌더링 한 데이터로 직접 범위가 지정됩니다 - 이 예에서 그것은 전달된 page props를 가진 `<Link />` 컴포넌트입니다. 이것은 다른 파일이 `<Link />` 컴포넌트에 props를 누락시켰다 하더라도 (`App.js`라고 합시다), 테스트는 `<Link />` 컴포넌트의 사용법을 알지 못하고 그것의 범위가 오직 `Link.react.js`이기 때문에 그것이 여전히 테스트를 통과 할 것을 의미합니다. 또한 서로 다른 스냅샷 테스트에서 다른 props를 가진 동일한 컴포넌트를 렌더링하는 것은 테스트가 각 다른 것에 대해 알지 못하기 때문에 첫 번째 것에 영향이 없습니다.

스냅샷 테스팅이 어떻게 동작하는 지와 왜 우리가 그것을 구축했는지에 대한 더 자세한 내용은 [릴리즈 블로그 게시글](https://jestjs.io/blog/2016/07/27/jest-14.html)에서 찾아볼 수 있습니다. 언제 스냅샷 테스팅을 사용해야 하는지를 잘 이애하려면 [이 블로그 게시글](http://benmccormick.org/2016/09/19/testing-with-jest-snapshots-first-impressions/)을 읽어볼 것은 권합니다. Jest로 스냅샷 테스팅에 [egghead 영상](https://egghead.io/lessons/javascript-use-jest-s-snapshot-testing-feature?pl=testing-javascript-with-jest-a36c4074)을 보는 것도 추천합니다.

### 스냅샷 업데이트

버그가 들어온 이후 스냅샷 테스트가 실패하면 이를 빼내는 것은 간단합니다. 실패가 일어나면 진행하고 문제를 해결하고 다시 스냅샷 테스트가 통과하도록 합니다. 이제, 의도적인 구현 변경으로 인해 스냅샷 테스트가 실패하는 경우에 대해 이야기 하겠습니다.

예제에서 Link 컴포넌트가 가리키는 주소를 의도적으로 변경할 경우 이러한 상황이 발생될 수 있습니다.

```javascript
// 다른 주소로의 Link를 가진 업데이트 된 테스트 케이스
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="http://www.instagram.com">Instagram</Link>)
    .toJSON();
  expect(tree).toMatchSnapshot();
});
```

그 경우, Jest는 다음의 결과를 출력할 것입니다:

![](/jest/img/content/failedSnapshotTest.png)

다른 주소를 가리키도록 컴포넌트를 업데이트 했기 때문에, 이 컴포넌트에 대한 스냅샷의 변경을 예상하는 것이 합리적입니다. 이 경우 업데이트 된 컴포넌트에 대한 스냅샷이 더 이상 스냅샵 산출물과 일치하지 않기 때문에 스냅샷 테스트 케이스는 실패합니다.

이를 해결하려면, 스냅샷 산물출을 업데이트 해야 할 필요가 있습니다. 스냅샷을 재생성 하라고 전달하는 플래그를 가지고 Jest를 실행 할 수 있습니다:

```bash
jest --updateSnapshot
```

위 명령을 실행하여 계속 진행하고 변경을 적용하세요. 원한다면 스냅샷을 재생성 하도록 동등한 단일 문자 `-u` 플래그를 사용할 수도 있습니다. 이것은 모든 실패 스냅샷 테스트에 대해 스냅샨 산출물을 재생성 할 것입니다. 의도하지 않은 버그로 인해 추가적인 실패 스냅샷 테스트가 있다면, 버그가 있는 동작의 스냅샷을 기록하는 것을 방지하도록 스냅샷을 재생성하기 전에 버그를 수정 할 필요가 있습니다.

재생성되는 스냅샷 테스트 케이스를 제한하고자 한다면, 패턴과 일치하는 테스트에 대해서만 스냅샷을 재생성하도록 추가 `--testNamePattern` 플래그를 전달 할 수 잇습니다.

[스냅샷 예제](https://github.com/facebook/jest/tree/master/examples/snapshot)를 복제하고, `Link` 컴포넌트를 수정하고, Jest를 실행하여 이 기능을 테스트 해 볼 수 있습니다.

### 대화형 스냅샷 모드

실패한 스냅샷은 watch 모드에서 대화식으로 업데이트 될 수도 있습니다:

![](/jest/img/content/interactiveSnapshot.png)

대화형 스냅샷 모드로 들어 갈 때, Jest는 한 번에 실패한 스냅샷 하나의 테스트로 이동하게 하고 실패한 출력을 검토할 기회를 제공할 것입니다.

여기에서 스냅샷을 업데이트 하거나 다음으로 건너뛰는 것을 선택할 수 있습니다:

![](/jest/img/content/interactiveSnapshotUpdate.gif)

완료되면, Jest는 watch 모드로 되돌아가기 전에 요약 정보를 제공할 것입니다:

![](/jest/img/content/interactiveSnapshotDone.png)

### 인라인 스냅샷

인라인 스냅샷은 스냅 샷 값이 자동으로 소스 코드에 다시 쓰여지는 것을 제외하고, 외부 스냅샷(`.snap` 파일)과 동일하게 동작합니다. 이것은 올바른 값이 작성되도록 외부 파일로 전환할 필요 없이 자동적으로 생성되는 이접을 얻을 수 있다는 것을 의미합니다.

> 인라인 스냅샷은 [Prettier](https://prettier.io)에 의해 구동됩니다. 인라인 스냅샷을 사용하려면 프로젝트에 `prettier`가 설치되어 있어야합니다. 테스트 파일에 작성 할 때 당신의 Prettier 구성이 준수될 것입니다.
>
> Jest가 찾을 수 없는 위치에 `prettier`가 설치된 경우, [`"prettierPath"`](./Configuration.md#prettierpath-string) 구성 속성(property)를 사용하여 Jest에게 찾는 방법을 전달 할 수 있습니다.

**예제:**

먼저, 인자가 없는  `.toMatchInlineSnapshot()`를 호출하는 테스트를 작성합니다.

```javascript
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="https://prettier.io">Prettier</Link>)
    .toJSON();
  expect(tree).toMatchInlineSnapshot();
});
```

다음 Jest를 실행하면, `tree`가 평가되고, 스냅샷이 `toMatchInlineSnapshot`에 인자로 작성될 것입니다:

```javascript
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="https://prettier.io">Prettier</Link>)
    .toJSON();
  expect(tree).toMatchInlineSnapshot(`
<a
  className="normal"
  href="https://prettier.io"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}
>
  Prettier
</a>
`);
});
```

그게 다 입니다! `--updateSnapshot`나 `--watch` 모드에 `u` 키를 사용하여 스냅샷을 업데이트 할 수도 있습니다.

### 속성(property) 매처

종종 스냅샷 하려는 객체에 생성되는 필드가 있습니다 (ID나 Date 처럼). 이 객체들을 스냅샷하려고 하면, 실행 할 때마다 스냅샷이 강제로 실패하게 됩니다:

```javascript
it('will fail every time', () => {
  const user = {
    createdAt: new Date(),
    id: Math.floor(Math.random() * 20),
    name: 'LeBron James',
  };

  expect(user).toMatchSnapshot();
});

// 스냅샷
exports[`will fail every time 1`] = `
Object {
  "createdAt": 2018-05-19T23:36:09.816Z,
  "id": 3,
  "name": "LeBron James",
}
`;
```

이 경우에, Jest는 모든 속성에 대해 비대칭 매처를 제공합니다. 이 매처는 스냅샷이 작성되거나 테스트 되기 전에 확인 되고, 이후 값을 받는 대신 스냅샷 파일로 저장됩니다:

```javascript
it('will check the matchers and pass', () => {
  const user = {
    createdAt: new Date(),
    id: Math.floor(Math.random() * 20),
    name: 'LeBron James',
  };

  expect(user).toMatchSnapshot({
    createdAt: expect.any(Date),
    id: expect.any(Number),
  });
});

// 스냅샷
exports[`will check the matchers and pass 1`] = `
Object {
  "createdAt": Any<Date>,
  "id": Any<Number>,
  "name": "LeBron James",
}
`;
```

매처가 아닌 주어진 값은 정확하게 검사되고 스냅샷에 저장될 것입니다:

```javascript
it('will check the values and pass', () => {
  const user = {
    createdAt: new Date(),
    name: 'Bond... James Bond',
  };

  expect(user).toMatchSnapshot({
    createdAt: expect.any(Date),
    name: 'Bond... James Bond',
  });
});

// 스냅샷
exports[`will check the values and pass 1`] = `
Object {
  "createdAt": Any<Date>,
  "name": 'Bond... James Bond',
}
`;
```

## 모범 사례

스냅샷은 어플리케이션에서 예기치 않은 변경을 식별하는데 환상적인 도구 입니다 - 그 인터페이스가 API 응답, UI, 로그, 에러메시지인지에 관계 없이. 다른 테스트 전략과 마찬가지로, 효과적으로 사용하기 위해 알아야 할 몇 가지 모범 사례와 따라야 할 지침들이 있습니다.

### 1. 스냅샷을 코드로 취급하세요

스냅샷을 커밋하고 정기적인 코드 리뷰 프로세스의 일부로 검토하세요. 이것은 다른 유형의 테스트나 프로젝트의 코드로 스냅샷을 취급하는 것을 의미합니다.

스냅샷을 명확하고, 짧게 유지시키고 이 스타일 컨벤션을 강제하는 툴을 사용하여 읽기 쉽게 하세요.

앞서 언급된 대로, Jest는 스냅샷을 사람이 읽을 수 있도록 [`pretty-format`](https://yarnpkg.com/en/package/pretty-format)을 사용하지만, 커밋을 짧게 고취시키고 단언에 집중할 수 있도록 [`no-large-snapshots`](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md) 옵션을 가진 [`eslint-plugin-jest`](https://yarnpkg.com/en/package/eslint-plugin-jest)나, 컴포넌트 스냅샷 비교 기능이 있는 [`snapshot-diff`](https://yarnpkg.com/en/package/snapshot-diff) 같은 추가적인 도구들을 도입하는 것이 유용하다는 것을 발견할 수도 있습니다.

목표는 풀 리퀘스트에서 스냅샷을 검토하기 쉽게 하고 테스트 스위트가 실패할 때 실패의 근본적인 원인을 조사하는 대신 스냅샷을 재생성하는 습관과 싸우는 것입니다.

### 2. 테스트는 결정론적이어야 합니다

테스트는 결정론적이어야 합니다. 동일한 테스트를 변경되지 않은 컴포넌트에 여러 번 수행하는 것은 매 번 동일한 결과를 만들어 냅니다. 생성 된 스냅샷이 플랫폼 특성이나 다른 비결정론적 데이터를 포함하지 않도록 해야 할 책임이 있습니다.

예를 들어, `Date.now()`를 사용하는 [Clock](https://github.com/facebook/jest/blob/master/examples/snapshot/Clock.react.js) 컴포넌트가 있다면, 이 컴포넌트로부터 생성 된 스냅샷은 테스트 케이스가 수행될 때마다 달라질 것입니다. 이 경우에 테스트가 수행될 때마다 일관된 값을 반환하도록 [Date.now() 메서드를 모의](MockFunctions.md) 할 수 있습니다:

```js
Date.now = jest.fn(() => 1482363367071);
```

이제, 테스트 케이스가 수행될 때마다, `Date.now()` 는 일관되게 `1482363367071`를 반환할 것입니다. 이는 테스트가 수행되는 때와 상관 없이 이 컴포넌트에 대해 동일한 스냅샷의 결과를 생성되게 할 것입니다.

### 3. 서술적인 스냅샷 이름을 사용하세요

항상 스냅샷에 대해 서술적인 테스트나 스냅샷 이름을 사용하도록 노력하세요. 가장 좋은 이름은 예상되는 스냅샷 컨텐츠를 설명합니다. 이것은 검토자가 검토하는 동안 스냅샷을 확인하는 것을 쉽게 하고, 오래된 스냅샷이 업데이트 전에 올바르게 동작하는지의 여부를 누구나 알기 쉽게 합니다.

예를 들어 비교해보세요:

```js
exports[`<UserName /> should handle some test case`] = `null`;

exports[`<UserName /> should handle some other test case`] = `
<div>
  Alan Turing
</div>
`;
```

이를 다음으로:

```js
exports[`<UserName /> should render null`] = `null`;

exports[`<UserName /> should render Alan Turing`] = `
<div>
  Alan Turing
</div>
`;
```

후자는 출력에서 예상되는 것이 무엇인지를 정확히 설명하기 때문에, 잘못되었을 경우 더 보기에 명확합니다:

```js
exports[`<UserName /> should render null`] = `
<div>
  Alan Turing
</div>
`;

exports[`<UserName /> should render Alan Turing`] = `null`;
```

## 자주 묻는 질문과 답변

### 지속적 통합 (CI) 시스템에서 자동으로 스냅샷이 작성 되나요?

아뇨, Jest 20 버전부터, Jest가 명시적으로 `--updateSnapshot`를 전달하지 않고 CI 시스템에서 수행 되는 경우 스냅샷은 자동으로 작성되지 않습니다. 모든 스냅샷이 CI에서 수행되는 코드의 일부로 예상되고 새로운 스냅샷이 자동으로 전달되기 때문에, CI 시스템에서 테스트 수행을 통과하지 않아야 합니다. 항상 모든 스냅샷을 커밋하고 버전 관리를 유지하는 것이 권장됩니다.

### 스냅샷 파일은 커밋되어야 하나요?

네, 모든 스냅샷 파일이 다루는 모듈 및 테스트와 함께 커밋 되어야 합니다. 그것들은 Jest에서 다른 단언 값과 유사하게 테스트의 일부로 간주되어야 합니다. 실제로 스냅샷은 주어진 시점의 소스 모듈의 상태를 나타냅니다. 이 방법으로, 소스 모듈이 수정되는 경우, Jest는 이전 버전에서 무엇이 변경되었는지를 알려줄 수 있습니다. 또한 검토자들이 코드 검토 동안 변경을 더 잘 연구할 수 있는 많은 추가적인 맥락을 제공할 수 있습니다.

### 스냅샷 테스트는 React 컴포넌트에서만 동작하나요?

[React](TutorialReact.md)와 [React Native](TutorialReactNative.md) 컴포넌트는 스냅샷 테스트에 대한 좋은 사용예입니다. 하지만, 스냅샷은 모든 직렬화 값을 캡쳐 할 수 있어야 하고 결과물이 올바른지를 테스트 하는 모든 목적에 대해 언제든지 사용되어야 합니다. Jest 저장소는 Jest 자체의 많은 테스팅 결과 예제, Jest 코드베이스의 여러 부분에서의 로그 메세지들 뿐만 아니라 Jest의 단언 라이브러리 출력물을 포함합니다. Jest 저장소에서 [CLI 출력 스냅샷하기](https://github.com/facebook/jest/blob/master/e2e/__tests__/console.test.ts)의 예제를 참고하세요

### 스냅샷 테스트와 시각적 회귀 테스트의 차이는 무엇인가요?

스냅샷 테스트와 시각적 회귀 테스트은 UI를 테스트 하는 두 가지 별개의 방법이며, 서로 다른 목적을 제공합니다. 시각적 회귀 테스트 도구는 웹 페이지의 스크린샷과 결과 이미지를 한 픽셀씩 비교합니다. 스냅샷 테스트를 통해 값이 직렬화되고 텍스트 파일로 저장되어 비교 알고리즘을 사용하여 비교됩니다. 고려 되어야 할 서로 다른 절충점이 있으며 스냅샷 테스트가 내장된 이유를 [Jest 블로그](https://jestjs.io/blog/2016/07/27/jest-14.html#why-snapshot-testing)에 기록해 두었습니다.

### 스냅샷 테스트가 유닛 테스트를 대체하나요?

스냅샷 테스트는 Jest와 제공되는 20 가지를 넘는 단언들 중 하나입니다. 스냅샷 테스트의 목표는 기존 유닛 테스트를 대체하는 것이 아니라, 추가적인 가치를 제공하고 테스트를 괴롭지 않게 하는 것입니다. 일부 시나리오에서 스냅샷 테스트는 특정 기능 세트(예를 들어 React 컴포논트)에 대한 유닛 테스트의 필요성을 제거할 수도 있지만, 함께 동작 할 수도 있습니다.

### 속도와 생성된 파일의 크기에 대한 스냅샷 테스트의 성능은 어떤가요?

Jest는 성능을 고려하여 재작성 되었고, 스냅샷 테스트도 예외는 아닙니다. 스냅샷은 텍스트 파일로 저장되므로, 이 테스트 방법은 빠르고 안정적입니다. Jest는 `toMatchSnapshot` 매처를 호출하는 각 테스트 파일에 대한 새로운 파일을 생성합니다. 스냅샷의 크기는 매우 작습니다: 참고로, Jest 코드베이스의 모든 스냅샷 파일의 크기는 300KB보다 작습니다.

### 스냅샷 파일에서 충돌을 어떻게 해결하나요?

스냅샷 파일은 항상 그것이 다루는 모듈의 현재 상태를 나타내야 합니다. 따라서, 두 브랜치를 병합하고 스냅샷 파일에서 충돌이 발생하면, 수동으로 충돌을 해결하거나 Jest를 실행하고 결과를 검사하여 스냅샷 파일을 업데이트 할 수 있습니다.

### 스냅샷 테스트로 테스트 주도 개발 원칙을 적용할 수 있습니까?

스냅샷 파일을 수동으로 작성할 수 있기는 하지만, 그건 일반적으로 쉽지 않습니다. 스냅샷은 처음에 코드를 설계하는 지침을 제공하는 것 보다는 테스트에 의해 다루어진 모듈의 출력이 변경되는지를 이해하는데 도움이 됩니다.

### 코드 적용 범위가 스냅샷 테스트와 함께 동작하나요?

네, 다른 테스트와 마찬가지입니다.
