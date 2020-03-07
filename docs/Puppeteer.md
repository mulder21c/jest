---
id: puppeteer
title: puppeteer와 함께 사용하기
---

[전역 설정/해제](Configuration.md#globalsetup-string)와 [비동기 테스트 환경](Configuration.md#testenvironment-string) API를 통해 Jest는 [puppeteer](https://github.com/GoogleChrome/puppeteer)와 함께 원활하게 작업할 수 있습니다.

> Puppeteer를 사용하여 테스트 파일에 대한 코드 커버리지를 생성하는 것은 테스트가 `page.$eval`, `page.$$eval` 또는`page.evaluate`를 Jest의 범위 밖에서 실행되는 전달된 함수로 사용하는 경우, 현재 가능하지 않습니다. 해결 방법은 Github에서[issue #7962](https://github.com/facebook/jest/issues/7962#issuecomment-495272339)를 확인하세요.

## jest-puppeteer 프리셋 사용

[Jest Puppeteer](https://github.com/smooth-code/jest-puppeteer)는 Puppeteer를 사용하는 테스트를 수행하는데 요구되는 모든 구성을 제공합니다.

1.  먼저 `jest-puppeteer`를 설치하세요

```
yarn add --dev jest-puppeteer
```

2.  Jest 구성에 프리셋을 지정하세요:

```json
{
  "preset": "jest-puppeteer"
}
```

3.  테스트를 작성하세요

```js
describe('Google', () => {
  beforeAll(async () => {
    await page.goto('https://google.com');
  });

  it('should be titled "Google"', async () => {
    await expect(page.title()).resolves.toMatch('Google');
  });
});
```

어떤 의존성도 로드할 필요가 없습니다. Puppeteer의 `page`와 `browser` 클래스는 자동으로 노출될 것입니다.

[documentation](https://github.com/smooth-code/jest-puppeteer)를 참고하세요.

## jest-puppeteer 프리셋이 없는 사용자 정의 예제

사전 준비 없이 puppeteer를 연결할 수도 있습니다. 기본적인 아이디어는 다음과 같습니다:

1.  전역 설정으로 puppeteer의 websocket 종단점을 시작하고 저장하세요
2.  각 테스트 환경으로부터 puppeteer를 연결하세요
3.  전역 해제를 통해 puppeteer를 닫으세요

전역 설정 스크립트의 예는 다음과 같습니다

```js
// setup.js
const puppeteer = require('puppeteer');
const mkdirp = require('mkdirp');
const path = require('path');
const fs = require('fs');
const os = require('os');

const DIR = path.join(os.tmpdir(), 'jest_puppeteer_global_setup');

module.exports = async function() {
  const browser = await puppeteer.launch();
  // 나중에 해제할 수 있도록 브라우저 인스턴스를 저장하세요
  // 이 전역은 해제에서만 사용가능하고 테스트 환경에서는 사용할 수 없습니다
  global.__BROWSER_GLOBAL__ = browser;

  // 테스트 환경에 대해 wsEndpoint를 노출시키기 위해 파일 시스템을 사용하세요
  mkdirp.sync(DIR);
  fs.writeFileSync(path.join(DIR, 'wsEndpoint'), browser.wsEndpoint());
};
```

그 다음 puppeteer에 대한 사용자 정의 테스트 환경이 필요합니다

```js
// puppeteer_environment.js
const NodeEnvironment = require('jest-environment-node');
const fs = require('fs');
const path = require('path');
const puppeteer = require('puppeteer');
const os = require('os');

const DIR = path.join(os.tmpdir(), 'jest_puppeteer_global_setup');

class PuppeteerEnvironment extends NodeEnvironment {
  constructor(config) {
    super(config);
  }

  async setup() {
    await super.setup();
    // wsEndpoint 얻기
    const wsEndpoint = fs.readFileSync(path.join(DIR, 'wsEndpoint'), 'utf8');
    if (!wsEndpoint) {
      throw new Error('wsEndpoint not found');
    }

    // puppeteer로 연결
    this.global.__BROWSER__ = await puppeteer.connect({
      browserWSEndpoint: wsEndpoint,
    });
  }

  async teardown() {
    await super.teardown();
  }

  runScript(script) {
    return super.runScript(script);
  }
}

module.exports = PuppeteerEnvironment;
```

마지막으로 puppeteer 인스턴스를 닫고 파일을 정리할 수 있습니다

```js
// teardown.js
const os = require('os');
const rimraf = require('rimraf');
const path = require('path');

const DIR = path.join(os.tmpdir(), 'jest_puppeteer_global_setup');
module.exports = async function() {
  // 브라우저 인스턴스 닫기
  await global.__BROWSER_GLOBAL__.close();

  // wsEndpoint 파일 정리
  rimraf.sync(DIR);
};
```

모든 것이 설정되면, 이제 다음과 같이 테스트를 작성할 수 있습니다:

```js
// test.js
const timeout = 5000;

describe(
  '/ (Home Page)',
  () => {
    let page;
    beforeAll(async () => {
      page = await global.__BROWSER__.newPage();
      await page.goto('https://google.com');
    }, timeout);

    it('should load without error', async () => {
      const text = await page.evaluate(() => document.body.textContent);
      expect(text).toContain('google');
    });
  },
  timeout,
);
```

마지막으로, 이 파일들에서 읽어들이도록 `jest.config.js`를 설정하세요. (`jest-puppeteer` 프리셋은 엔진에서 이와 같은 작업을 수행합니다.)

```js
module.exports = {
  globalSetup: './setup.js',
  globalTeardown: './teardown.js',
  testEnvironment: './puppeteer_environment.js',
};
```

[전체 동작 예제](https://github.com/xfumihiro/jest-puppeteer-example) 코드입니다.
