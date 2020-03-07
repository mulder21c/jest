---
id: mongodb
title: MongoDB와 함께 사용하기
---

[전역 설정/해제](Configuration.md#globalsetup-string)와 [비동기 테스트 환경](Configuration.md#testenvironment-string) API를 통해, Jest는 [MongoDB](https://www.mongodb.com/)와 함께 원활하게 작업할 수 있습니다.

## jest-mongodb 프리셋 사용

[Jest MongoDB](https://github.com/shelfio/jest-mongodb)는 MongoDB를 사용하는 테스트를 수행하는데 요구되는 모든 구성을 제공합니다.

1.  먼저 `@shelf/jest-mongodb`를 설치하세요

```
yarn add @shelf/jest-mongodb --dev
```

2.  Jest 구성에 프리셋을 지정하세요:

```json
{
  "preset": "@shelf/jest-mongodb"
}
```

3.  테스트를 작성하세요

```js
const {MongoClient} = require('mongodb');

describe('insert', () => {
  let connection;
  let db;

  beforeAll(async () => {
    connection = await MongoClient.connect(global.__MONGO_URI__, {
      useNewUrlParser: true,
    });
    db = await connection.db(global.__MONGO_DB_NAME__);
  });

  afterAll(async () => {
    await connection.close();
    await db.close();
  });

  it('should insert a doc into collection', async () => {
    const users = db.collection('users');

    const mockUser = {_id: 'some-user-id', name: 'John'};
    await users.insertOne(mockUser);

    const insertedUser = await users.findOne({_id: 'some-user-id'});
    expect(insertedUser).toEqual(mockUser);
  });
});
```

어떤 의존성도 로드할 필요가 없습니다

더 자세한 내용은 (MongoDB 버전 구성하기, 기타 등등) [문서](https://github.com/shelfio/jest-mongodb)를 참고하세요.
