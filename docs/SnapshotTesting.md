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

Once you enter Interactive Snapshot Mode, Jest will step you through the failed snapshots one test at a time and give you the opportunity to review the failed output.

From here you can choose to update that snapshot or skip to the next:

![](/jest/img/content/interactiveSnapshotUpdate.gif)

Once you're finished, Jest will give you a summary before returning back to watch mode:

![](/jest/img/content/interactiveSnapshotDone.png)

### Inline Snapshots

Inline snapshots behave identically to external snapshots (`.snap` files), except the snapshot values are written automatically back into the source code. This means you can get the benefits of automatically generated snapshots without having to switch to an external file to make sure the correct value was written.

> Inline snapshots are powered by [Prettier](https://prettier.io). To use inline snapshots you must have `prettier` installed in your project. Your Prettier configuration will be respected when writing to test files.
>
> If you have `prettier` installed in a location where Jest can't find it, you can tell Jest how to find it using the [`"prettierPath"`](./Configuration.md#prettierpath-string) configuration property.

**Example:**

First, you write a test, calling `.toMatchInlineSnapshot()` with no arguments:

```javascript
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="https://prettier.io">Prettier</Link>)
    .toJSON();
  expect(tree).toMatchInlineSnapshot();
});
```

The next time you run Jest, `tree` will be evaluated, and a snapshot will be written as an argument to `toMatchInlineSnapshot`:

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

That's all there is to it! You can even update the snapshots with `--updateSnapshot` or using the `u` key in `--watch` mode.

### Property Matchers

Often there are fields in the object you want to snapshot which are generated (like IDs and Dates). If you try to snapshot these objects, they will force the snapshot to fail on every run:

```javascript
it('will fail every time', () => {
  const user = {
    createdAt: new Date(),
    id: Math.floor(Math.random() * 20),
    name: 'LeBron James',
  };

  expect(user).toMatchSnapshot();
});

// Snapshot
exports[`will fail every time 1`] = `
Object {
  "createdAt": 2018-05-19T23:36:09.816Z,
  "id": 3,
  "name": "LeBron James",
}
`;
```

For these cases, Jest allows providing an asymmetric matcher for any property. These matchers are checked before the snapshot is written or tested, and then saved to the snapshot file instead of the received value:

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

// Snapshot
exports[`will check the matchers and pass 1`] = `
Object {
  "createdAt": Any<Date>,
  "id": Any<Number>,
  "name": "LeBron James",
}
`;
```

Any given value that is not a matcher will be checked exactly and saved to the snapshot:

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

// Snapshot
exports[`will check the values and pass 1`] = `
Object {
  "createdAt": Any<Date>,
  "name": 'Bond... James Bond',
}
`;
```

## Best Practices

Snapshots are a fantastic tool for identifying unexpected interface changes within your application – whether that interface is an API response, UI, logs, or error messages. As with any testing strategy, there are some best-practices you should be aware of, and guidelines you should follow, in order to use them effectively.

### 1. Treat snapshots as code

Commit snapshots and review them as part of your regular code review process. This means treating snapshots as you would any other type of test or code in your project.

Ensure that your snapshots are readable by keeping them focused, short, and by using tools that enforce these stylistic conventions.

As mentioned previously, Jest uses [`pretty-format`](https://yarnpkg.com/en/package/pretty-format) to make snapshots human-readable, but you may find it useful to introduce additional tools, like [`eslint-plugin-jest`](https://yarnpkg.com/en/package/eslint-plugin-jest) with its [`no-large-snapshots`](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md) option, or [`snapshot-diff`](https://yarnpkg.com/en/package/snapshot-diff) with its component snapshot comparison feature, to promote committing short, focused assertions.

The goal is to make it easy to review snapshots in pull requests, and fight against the habit of regenerating snapshots when test suites fail instead of examining the root causes of their failure.

### 2. Tests should be deterministic

Your tests should be deterministic. Running the same tests multiple times on a component that has not changed should produce the same results every time. You're responsible for making sure your generated snapshots do not include platform specific or other non-deterministic data.

For example, if you have a [Clock](https://github.com/facebook/jest/blob/master/examples/snapshot/Clock.react.js) component that uses `Date.now()`, the snapshot generated from this component will be different every time the test case is run. In this case we can [mock the Date.now() method](MockFunctions.md) to return a consistent value every time the test is run:

```js
Date.now = jest.fn(() => 1482363367071);
```

Now, every time the snapshot test case runs, `Date.now()` will return `1482363367071` consistently. This will result in the same snapshot being generated for this component regardless of when the test is run.

### 3. Use descriptive snapshot names

Always strive to use descriptive test and/or snapshot names for snapshots. The best names describe the expected snapshot content. This makes it easier for reviewers to verify the snapshots during review, and for anyone to know whether or not an outdated snapshot is the correct behavior before updating.

For example, compare:

```js
exports[`<UserName /> should handle some test case`] = `null`;

exports[`<UserName /> should handle some other test case`] = `
<div>
  Alan Turing
</div>
`;
```

To:

```js
exports[`<UserName /> should render null`] = `null`;

exports[`<UserName /> should render Alan Turing`] = `
<div>
  Alan Turing
</div>
`;
```

Since the later describes exactly what's expected in the output, it's more clear to see when it's wrong:

```js
exports[`<UserName /> should render null`] = `
<div>
  Alan Turing
</div>
`;

exports[`<UserName /> should render Alan Turing`] = `null`;
```

## Frequently Asked Questions

### Are snapshots written automatically on Continuous Integration (CI) systems?

No, as of Jest 20, snapshots in Jest are not automatically written when Jest is run in a CI system without explicitly passing `--updateSnapshot`. It is expected that all snapshots are part of the code that is run on CI and since new snapshots automatically pass, they should not pass a test run on a CI system. It is recommended to always commit all snapshots and to keep them in version control.

### Should snapshot files be committed?

Yes, all snapshot files should be committed alongside the modules they are covering and their tests. They should be considered part of a test, similar to the value of any other assertion in Jest. In fact, snapshots represent the state of the source modules at any given point in time. In this way, when the source modules are modified, Jest can tell what changed from the previous version. It can also provide a lot of additional context during code review in which reviewers can study your changes better.

### Does snapshot testing only work with React components?

[React](TutorialReact.md) and [React Native](TutorialReactNative.md) components are a good use case for snapshot testing. However, snapshots can capture any serializable value and should be used anytime the goal is testing whether the output is correct. The Jest repository contains many examples of testing the output of Jest itself, the output of Jest's assertion library as well as log messages from various parts of the Jest codebase. See an example of [snapshotting CLI output](https://github.com/facebook/jest/blob/master/e2e/__tests__/console.test.ts) in the Jest repo.

### What's the difference between snapshot testing and visual regression testing?

Snapshot testing and visual regression testing are two distinct ways of testing UIs, and they serve different purposes. Visual regression testing tools take screenshots of web pages and compare the resulting images pixel by pixel. With Snapshot testing values are serialized, stored within text files, and compared using a diff algorithm. There are different trade-offs to consider and we listed the reasons why snapshot testing was built in the [Jest blog](https://jestjs.io/blog/2016/07/27/jest-14.html#why-snapshot-testing).

### Does snapshot testing replace unit testing?

Snapshot testing is only one of more than 20 assertions that ship with Jest. The aim of snapshot testing is not to replace existing unit tests, but to provide additional value and make testing painless. In some scenarios, snapshot testing can potentially remove the need for unit testing for a particular set of functionalities (e.g. React components), but they can work together as well.

### What is the performance of snapshot testing regarding speed and size of the generated files?

Jest has been rewritten with performance in mind, and snapshot testing is not an exception. Since snapshots are stored within text files, this way of testing is fast and reliable. Jest generates a new file for each test file that invokes the `toMatchSnapshot` matcher. The size of the snapshots is pretty small: For reference, the size of all snapshot files in the Jest codebase itself is less than 300 KB.

### How do I resolve conflicts within snapshot files?

Snapshot files must always represent the current state of the modules they are covering. Therefore, if you are merging two branches and encounter a conflict in the snapshot files, you can either resolve the conflict manually or update the snapshot file by running Jest and inspecting the result.

### Is it possible to apply test-driven development principles with snapshot testing?

Although it is possible to write snapshot files manually, that is usually not approachable. Snapshots help to figure out whether the output of the modules covered by tests is changed, rather than giving guidance to design the code in the first place.

### Does code coverage work with snapshot testing?

Yes, as well as with any other test.
