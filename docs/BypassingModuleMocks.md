---
id: bypassing-module-mocks
title: 우회 모듈 모의
---

Jest는 테스트에서 전체 모듈을 모의할 수 있게 해주는데, 코드가 그 모듈에서 올바르게 함수를 호출하고 있는지를 테스트 하는데 유용하게 사용될 수 있습니다. 하지만, 때때로 _테스트 파일_에서 모의된 모듈의 일부를 사용하기 원할 수 있고, 그 경우에 모의된 버전보다 원래 구현에 접근하기를 원할 수 있습니다.

이 `createUser` 함수에 대한 테스트 케이스를 작성한다고 가정해보세요:

```javascript
// createUser.js
import fetch from 'node-fetch';

export const createUser = async () => {
  const response = await fetch('http://website.com/users', {method: 'POST'});
  const userId = await response.text();
  return userId;
};
```

테스트는 실제로 네트워크 요청을 하지 않고도 함수가 호출될 수 있도록 `fetch` 함수를 모의하기 원할 것입니다. 그러나, 생성된 사용자의 ID를 얻기 위해 함수를 사용하기 때문에, (`Promise`로 싸여진) `Response`를 가진 `fetch`의 반환 된 값을 모의할 필요가 있을 것입니다. 따라서 처음에 테스트를 다음과 같이 작성해 볼 수 있습니다:

```javascript
jest.mock('node-fetch');

import fetch, {Response} from 'node-fetch';

import {createUser} from './createUser';

test('createUser calls fetch with the right args and returns the user id', async () => {
  fetch.mockReturnValue(Promise.resolve(new Response('4')));

  const userId = await createUser();

  expect(fetch).toHaveBeenCalledTimes(1);
  expect(fetch).toHaveBeenCalledWith('http://website.com/users', {
    method: 'POST',
  });
  expect(userId).toBe('4');
});
```

하지만, 그 테스트를 실행하면, `TypeError: response.text is not a function` 오류가 발생하여 `createUser` 함수가 실패하는 것을 발견할 것입니다. 이는 `node-fetch`로부터 가져온 `Response` 클래스가 모의 되었기 때문에 (테스트 파일의 최 상단에서 `jest.mock`이 호출하기 때문에) 더 이상 제 구실을 하지 않습니다.

이와 같은 문제를 해결하기 위해, Jest는 `jest.requireActual` 헬퍼를 제공합니다. 위 테스트가 동작하게 하기 위해, 테스트 파일의 가져오기를 다음과 같이 변경하세요:

```javascript
// 이전
jest.mock('node-fetch');
import fetch, {Response} from 'node-fetch';
```

```javascript
// 이후
jest.mock('node-fetch');
import fetch from 'node-fetch';
const {Response} = jest.requireActual('node-fetch');
```

이를 통해 테스트 파일은 모의된 버전 대신 `node-fetch`로부터의 실제 `Response` 객체를 가져올 수 있습니다. 이는 테스트가 이제 올바르게 통과할 것을 의미합니다.
