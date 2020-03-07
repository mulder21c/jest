---
id: dynamodb
title: DynamoDB와 함께 사용하기
---

[전역 설정/해제](Configuration.md#globalsetup-string)와 [비동기 테스트 환경](Configuration.md#testenvironment-string) API를 통해, API를 통해, Jest는 [DynamoDB](https://aws.amazon.com/dynamodb/)와 함께 원활하게 작업할 수 있습니다.

## jest-dynamodb 프리셋 사용

[Jest DynamoDB](https://github.com/shelfio/jest-dynamodb)는 DynamoDB를 사용하는 테스트를 수행하는데 요구되는 모든 구성을 제공합니다.

1.  먼저 `@shelf/jest-dynamodb`를 설치하세요

```
yarn add @shelf/jest-dynamodb --dev
```

2.  Jest 구성에 프리셋을 지정하세요:

```json
{
  "preset": "@shelf/jest-dynamodb"
}
```

3.  `jest-dynamodb-config.js`를 생성하고 DynamoDB 테이블을 정의하세요

[테이블 생성 API](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html#createTable-property)를 참고하세요

```js
module.exports = {
  tables: [
    {
      TableName: `files`,
      KeySchema: [{AttributeName: 'id', KeyType: 'HASH'}],
      AttributeDefinitions: [{AttributeName: 'id', AttributeType: 'S'}],
      ProvisionedThroughput: {ReadCapacityUnits: 1, WriteCapacityUnits: 1},
    },
    // etc
  ],
};
```

4.  DynamoDB 클라이언트 구성

```js
const {DocumentClient} = require('aws-sdk/clients/dynamodb');

const isTest = process.env.JEST_WORKER_ID;
const config = {
  convertEmptyValues: true,
  ...(isTest && {
    endpoint: 'localhost:8000',
    sslEnabled: false,
    region: 'local-env',
  }),
};

const ddb = new DocumentClient(config);
```

5.  테스트 작성

```js
it('should insert item into table', async () => {
  await ddb
    .put({TableName: 'files', Item: {id: '1', hello: 'world'}})
    .promise();

  const {Item} = await ddb.get({TableName: 'files', Key: {id: '1'}}).promise();

  expect(Item).toEqual({
    id: '1',
    hello: 'world',
  });
});
```

어떤 의존성도 로드할 필요가 없습니다

자세한 내용은 [문서](https://github.com/shelfio/jest-dynamodb)를 참고하세요.
