---
id: webpack
title: Webpack과 사용하기
---

Jest는 어셋, 스타일, 컴파일을 관리하기 위해 [webpack](https://webpack.js.org/)을 사용하는 프로젝트에 사용될 수 있습니다. webpack은 스타일시트, 자바스크립트로 컴파일 되는 언어 및 도구의 광범위한 에코 시스템과 더불어 이미지와 폰트 같은 어셋을 관리할 수 있도록 어플리케이션과 직접 통합하기 때문에 다른 도구들 보다 일부 고유한 기회를 제공_합니다_.

## webpack 예제

다소 일반적인 webpack 구성 파일을 가지고 시작하여 Jest 설정으로 변환해봅시다.

```js
// webpack.config.js
module.exports = {
  module: {
    loaders: [
      {exclude: ['node_modules'], loader: 'babel', test: /\.jsx?$/},
      {loader: 'style-loader!css-loader', test: /\.css$/},
      {loader: 'url-loader', test: /\.gif$/},
      {loader: 'file-loader', test: /\.(ttf|eot|svg)$/},
    ],
  },
  resolve: {
    alias: {
      config$: './configs/app-config.js',
      react: './vendor/react-master',
    },
    extensions: ['', 'js', 'jsx'],
    modules: [
      'node_modules',
      'bower_components',
      'shared',
      '/shared/vendor/modules',
    ],
  },
};
```

Babel에 의해 변환되는 자바스크립트 파일이 있는 경우, `babel-jest` 플러그인을 설치하여 [Babel에 대한 지원을 활성화](GettingStarted.md#using-babel) 할 수 있습니다. Babel 이외의 자바스크립트 변환은 Jest의 [`transform`](Configuration.md#transform-objectstring-pathtotransformer--pathtotransformer-object) 구성 옵션에서 처리될 수 있습니다.

### 정적 어셋 처리하기

다음으로, 스타일시트와 이미지 같은 어셋 파일을 우아하게 처리하기 위해 Jest를 구성해봅시다. 일반적으로, 이 파일들은 테스트에 특히 유용하지 않으므로 안전하게 모의 할 수 있습니다. 하지만, CSS 모듈을 사용하는 경우 클래스명 조회를 위해 프록시를 모의하는 것이 좋습니다.

```json
// package.json
{
  "jest": {
    "moduleNameMapper": {
      "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/__mocks__/fileMock.js",
      "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js"
    }
  }
}
```

그리고 모의 파일 자체는:

```js
// __mocks__/styleMock.js

module.exports = {};
```

```js
// __mocks__/fileMock.js

module.exports = 'test-file-stub';
```

### CSS 모듈 모의

[CSS 모듈](https://github.com/css-modules/css-modules)을 모의 하기 위해 [ES6 프록시](https://github.com/keyanzhang/identity-obj-proxy)를 사용할 수 있습니다:

```bash
yarn add --dev identity-obj-proxy
```

이후 스타일 객체의 모든 클래스명 조회가 그대로 (예를 들어, `styles.foobar === 'foobar'`) 반환될 것입니다. 이는 React [스냅샷 테스트 하기](SnapshotTesting.md)에 매우 편리합니다.

```json
// package.json (for CSS Modules)
{
  "jest": {
    "moduleNameMapper": {
      "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/__mocks__/fileMock.js",
      "\\.(css|less)$": "identity-obj-proxy"
    }
  }
}
```

> 프록시는 기본적으로 Node 6에서 활성화 되어 있음을 주의하세요. 아직 Node 6가 아니라면, `node --harmony_proxies node_modules/.bin/jest`를 사용하여 Jest를 호출하세요.

`moduleNameMapper`이 요구사항을 충족시킬 수 없는 경우, 어셋이 변환되는 방법을 지정하기 위해 Jest의 [`transform`](Configuration.md#transform-objectstring-pathtotransformer--pathtotransformer-object) 구성 옵션을 사용할 수 있습니다. 예를 들어, 파일의 기본 이름을 (`require('logo.jpg');`이 `'logo'`를 반환하는 것과 같은) 반환하는 변환기는 다음과 같이 작성될 수 있습니다:

```js
// fileTransformer.js
const path = require('path');

module.exports = {
  process(src, filename, config, options) {
    return 'module.exports = ' + JSON.stringify(path.basename(filename)) + ';';
  },
};
```

```json
// package.json (사용자 정의 변환기와 CSS 모듈에 대해)
{
  "jest": {
    "moduleNameMapper": {
      "\\.(css|less)$": "identity-obj-proxy"
    },
    "transform": {
      "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/fileTransformer.js"
    }
  }
}
```

Jest는 스타일시트나 이미지 확장자와 일치하는 파일을 무시하고 대신 모의 파일이 필요하다고 이야기 했습니다. 웹팩 구성이 처리하는 파일 유형과 일치하는 정규식을 조정할 수 있습니다.

_참고: 추가적인 코드 전처리기와 함께 babel-jest를 사용하는 경우, `.js` 파일을 babel-jest 모듈에 일치시키기 위해 babel-jest를 JavaScript 코드에 대한 변환기로 명시적으로 정의해야 합니다._

```json
"transform": {
  "^.+\\.js$": "babel-jest",
  "^.+\\.css$": "custom-transformer",
  ...
}
```

### Configuring Jest to find our files

Now that Jest knows how to process our files, we need to tell it how to _find_ them. For webpack's `modulesDirectories`, and `extensions` options there are direct analogs in Jest's `moduleDirectories` and `moduleFileExtensions` options.

```json
// package.json
{
  "jest": {
    "moduleFileExtensions": ["js", "jsx"],
    "moduleDirectories": ["node_modules", "bower_components", "shared"],

    "moduleNameMapper": {
      "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js",
      "\\.(gif|ttf|eot|svg)$": "<rootDir>/__mocks__/fileMock.js"
    }
  }
}
```

> Note: `<rootDir>` is a special token that gets replaced by Jest with the root of your project. Most of the time this will be the folder where your `package.json` is located unless you specify a custom `rootDir` option in your configuration.

Similarly webpack's `resolve.root` option functions like setting the `NODE_PATH` env variable, which you can set, or make use of the `modulePaths` option.

```json
// package.json
{
  "jest": {
    "modulePaths": ["/shared/vendor/modules"],
    "moduleFileExtensions": ["js", "jsx"],
    "moduleDirectories": ["node_modules", "bower_components", "shared"],
    "moduleNameMapper": {
      "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js",
      "\\.(gif|ttf|eot|svg)$": "<rootDir>/__mocks__/fileMock.js"
    }
  }
}
```

And finally, we have to handle the webpack `alias`. For that we can make use of the `moduleNameMapper` option again.

```json
// package.json
{
  "jest": {
    "modulePaths": ["/shared/vendor/modules"],
    "moduleFileExtensions": ["js", "jsx"],
    "moduleDirectories": ["node_modules", "bower_components", "shared"],

    "moduleNameMapper": {
      "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js",
      "\\.(gif|ttf|eot|svg)$": "<rootDir>/__mocks__/fileMock.js",

      "^react(.*)$": "<rootDir>/vendor/react-master$1",
      "^config$": "<rootDir>/configs/app-config.js"
    }
  }
}
```

That's it! webpack is a complex and flexible tool, so you may have to make some adjustments to handle your specific application's needs. Luckily for most projects, Jest should be more than flexible enough to handle your webpack config.

> Note: For more complex webpack configurations, you may also want to investigate projects such as: [babel-plugin-webpack-loaders](https://github.com/istarkov/babel-plugin-webpack-loaders).

## Using with webpack 2

webpack 2 offers native support for ES modules. However, Jest runs in Node, and thus requires ES modules to be transpiled to CommonJS modules. As such, if you are using webpack 2, you most likely will want to configure Babel to transpile ES modules to CommonJS modules only in the `test` environment.

```json
// .babelrc
{
  "presets": [["env", {"modules": false}]],

  "env": {
    "test": {
      "plugins": ["transform-es2015-modules-commonjs"]
    }
  }
}
```

> Note: Jest caches files to speed up test execution. If you updated .babelrc and Jest is still not working, try running Jest with `--no-cache`.

If you use dynamic imports (`import('some-file.js').then(module => ...)`), you need to enable the `dynamic-import-node` plugin.

```json
// .babelrc
{
  "presets": [["env", {"modules": false}]],

  "plugins": ["syntax-dynamic-import"],

  "env": {
    "test": {
      "plugins": ["dynamic-import-node"]
    }
  }
}
```

For an example of how to use Jest with Webpack with React, Redux, and Node, you can view one [here](https://github.com/jenniferabowd/jest_react_redux_node_webpack_complex_example).
