---
id: getting-started
title: 시작하기
---

[`yarn`](https://yarnpkg.com/en/package/jest)을 사용하여 Jest를 설치하세요:

```bash
yarn add --dev jest
```

또는 [`npm`](https://www.npmjs.com/):

```bash
npm install --save-dev jest
```

참고: Jest 문서는 `yarn` 명령을 사용하지만, `npm` 도 동작합니다. `yarn`과 `npm` 명령은 [yarn 문서](https://yarnpkg.com/en/docs/migrating-from-npm#toc-cli-commands-comparison)에서 비교해 볼 수 있습니다.

두 숫자를 더하는 가상의 함수에 대한 테스트를 작성하는 것으로 시작해봅시다. 먼저 `sum.js` 파일을 생성합니다:

```javascript
function sum(a, b) {
  return a + b;
}
module.exports = sum;
```

이후, `sum.test.js`이라는 이름의 파일을 생성합니다. 이 파일에 실제 테스트가 포함될겁니다:

```javascript
const sum = require('./sum');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

다음의 내용을 `package.json`에 추가하세요:

```json
{
  "scripts": {
    "test": "jest"
  }
}
```

마지막으로, `yarn test` 또는 `npm run test`를 실행하면 Jest가 이 메시지를 출력할 것입니다:

```bash
PASS  ./sum.test.js
✓ adds 1 + 2 to equal 3 (5ms)
```

**Jest를 사용한 첫 번째 테스트를 성공적으로 작성하셨습니다!**

이 테스트는 두 값이 정확히 일치하는지를 테스트 하기 위해 `expect`와 `toBe`를 사용했습니다. Jest가 검사 할 수 있는 다른 것들을 학습하려면, [Matchers 사용하기](UsingMatchers.md)를 참고하세요

## 커맨드라인에서 실행하기

(`yarn global add jest` 또는 `npm install jest --global`에 의해서 `PATH`가 전역으로 사용될 수 있는 경우) Jest를 CLI에서 다양한 유용한 옵션을 사용하여 직접 실행할 수 있습니다.

다음은 설정 파일로 `config.json`을 사용하여 `my-test`와 일치하는 파일에서 Jest를 실행하고, 실행 후 네이티브 OS 알림이 표기되는 방법입니다:

```bash
jest my-test --notify --config=config.json
```

커맨드 라인을 통해 `jest`를 실행하는 것에 더 학습하기를 원하면, [Jest CLI Options](CLI.md) 페이지를 참조하세요.

## 추가 구성

### 기본 구성 파일 생성

프로젝트를 기반으로, Jest는 몇 가지 질문을 하고 각 옵션에 대한 짧은 설명을 포함하는 기본 구성 파일을 생성합니다:

```bash
jest --init
```

### Babel 사용하기

[Babel](http://babeljs.io/)을 사용하려면, `yarn`을 통해 필수 의존성을 설치하세요:

```bash
yarn add --dev babel-jest @babel/core @babel/preset-env
```

프로젝트 루트에 `babel.config.js`을 생성하여 현재 노드 버전에 맞는 Babel을 설정하세요:

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          node: 'current',
        },
      },
    ],
  ],
};
```

**Babel에 대한 이상적인 구성은 프로젝트에 따라 다릅니다.** 더 자세한 내용은 [Babel 문서](https://babeljs.io/docs/en/)를 참조하세요.

<details><summary markdown="span"><strong>jest-인식 babel 구성 만들기</strong></summary>

Jest는 `process.env.NODE_ENV`를 `test`로 설정하고 그렇지 않으면 아무 것으로도 설정하지 않습니다. Jest에 필요한 컴파일만을 조건부로 설정하기 위해서 구성에 이를 사용할 수 있습니다. 예를 들어,

```javascript
// babel.config.js
module.exports = api => {
  const isTest = api.env('test');
  // 사용될 프리셋이나 플러그인을 결정하기 위해 isTest를 사용할 수 있습니다.

  return {
    // ...
  };
};
```

> 참고: Jest를 설치할 때 `babel-jest`는 자동으로 설치되고 프로젝트에 바벨 구성이 존재하는 경우 자동으로 파일을 변환합니다. 이 동작을 방지하기위해, 명시적으로 `transform` 구성 옵션을 재설정할 수 있습니다:

```javascript
// jest.config.js
module.exports = {
  transform: {},
};
```

</details>

<details><summary markdown="span"><strong>Babel 6 지원</strong></summary>

Jest 24 버전은 babel 6에 대한 지원을 버렸습니다. 활발하게 관리되는 Bable 7으로 업그레이드하기를 강력히 추천합니다. 그러나 Babel 7로 업그레이드 할 수 없는 경우, Jest 23버전을 사용하는 것을 유지하거나 다음 예에서와 같이 23 버전으로 잠긴 `babel-jest`와 함께 Jest 24로 업그레이드 하세요:

```
"dependencies": {
  "babel-core": "^6.26.3",
  "babel-jest": "^23.6.0",
  "babel-preset-env": "^1.7.0",
  "jest": "^24.0.0"
}
```

일반적으로 모든 Jest 패키지의 동일한 버전을 사용하는 것을 권장하지만, 이 해결책은 당분간 Babel 6와 함께 Jest의 최신 버전을 계속 사용할 수 있게 해줄 것입니다.

</details>

### Webpack 사용하기

Jest는 어셋, 스타일, 컴파일을 관리하기 위해 [webpack](https://webpack.js.org/)을 사용하는 프로젝트에 사용될 수 있습니다. Webpack은 다른 도구보다 몇 가지 독특한 과제를 제공합니다. 시작하려면 [webpack 가이드](Webpack.md)를 참고하세요.

### TypeScript 사용하기

Jest는 Babel을 통해 TypeScript를 지원합니다. 먼저 위 [Babel 사용하기](#using-babel)의 지침을 따르도록 하세요. 그 다음 `yarn`을 통해 `@babel/preset-typescript`을 설치하세요:

```bash
yarn add --dev @babel/preset-typescript
```

그리고 나서 `babel.config.js`에 프리셋 목록에 `@babel/preset-typescript`를 추가하세요.

```diff
// babel.config.js
module.exports = {
  presets: [
    ['@babel/preset-env', {targets: {node: 'current'}}],
+    '@babel/preset-typescript',
  ],
};
```

하지만, Babel과 함께 TypeScript를 사용하는데에 몇 가지 [주의사항](https://babeljs.io/docs/en/next/babel-plugin-transform-typescript.html#caveats)이 있습니다. Babel에서 TypeScript 지원은 트랜스파일이기 때문에, Jest는 실행 될 때 테스트를 type-check하지 않을 것입니다. type-check를 원한다면, [ts-jest](https://github.com/kulshekhar/ts-jest)를 사용할 수 있습니다.
