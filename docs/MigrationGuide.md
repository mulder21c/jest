---
id: migration-guide
title: Jest로 마이그레이션 하기
---

기존의 코드베이스로 Jest를 시도해보고 싶다면 Jest로 변환하는 여러가지 방법이 있습니다:

- Jasmin이나 Jasmin 같은 API를 (예를 들어 [Mocha](https://mochajs.org)) 사용 중이라면, Jest는 대부분 호환 가능하므로 마이그레이션이 덜 복잡합니다.
- AVA, Expect.js (by Automattic), Jasmine, Mocha, proxyquire, Should.js, Tape를 사용 중이라면, Jest Codemods를 사용하여 자동으로 마이그레이션 할 수 있습니다 (아래 참조).
- [chai](http://chaijs.com/)를 좋아한다면, Jest로 업그레이드하고 계속해서 chai를 사용할 수 있습니다. 하지만, Jest의 단언과 실패 메세지를 시험해 보기를 권장합니다. Jest Codemods는 chai로부터 마이그레이션 할 수 있습니다 (아래 참조).

## jest-codemods

[AVA](https://github.com/avajs/ava), [Chai](https://github.com/chaijs/chai), [Expect.js (by Automattic)](https://github.com/Automattic/expect.js), [Jasmine](https://github.com/jasmine/jasmine), [Mocha](https://github.com/mochajs/mocha), [proxyquire](https://github.com/thlorenz/proxyquire), [Should.js](https://github.com/shouldjs/should.js), [Tape](https://github.com/substack/tape)를 사용 중인 경우, 대부분의 지저분한 마이그레이션 작업을 하기 위해 서드파티 [jest-codemods](https://github.com/skovhus/jest-codemods)를 사용할 수 있습니다. 그것은 [jscodeshift](https://github.com/facebook/jscodeshift)를 사용하여 코드베이스에서 코드 변환을 실행합니다.

기존 테스트를 변환하기 위해, 테스트를 포함하는 프로젝트로 이동하여 다음을 실행하세요:

```bash
npx jest-codemods
```

더 자세한 정보는 [https://github.com/skovhus/jest-codemods](https://github.com/skovhus/jest-codemods)에서 찾을 수 있습니다.
