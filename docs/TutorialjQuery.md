---
id: tutorial-jquery
title: DOM 조작
---

테스트 하는 것이 어려운 것으로 종종 간주되는 함수의 또 다른 클래스는 DOM을 직접 조작하는 코드입니다. click 이벤트를 수신하고 비동기적으로 데이터를 가져와 콘텐츠를 설정하는 다음 jQuery 코드 조각을 테스트 하는 방법을 살펴봅시다.

```javascript
// displayUser.js
'use strict';

const $ = require('jquery');
const fetchCurrentUser = require('./fetchCurrentUser.js');

$('#button').click(() => {
  fetchCurrentUser(user => {
    const loggedText = 'Logged ' + (user.loggedIn ? 'In' : 'Out');
    $('#username').text(user.fullName + ' - ' + loggedText);
  });
});
```

다시, `__tests__/` 폴더에 테스트 파일을 생성합니다:

```javascript
// __tests__/displayUser-test.js
'use strict';

jest.mock('../fetchCurrentUser');

test('displays a user after a click', () => {
  // document body를 설정하세요
  document.body.innerHTML =
    '<div>' +
    '  <span id="username" />' +
    '  <button id="button" />' +
    '</div>';

  // 이 모듈은 사이드 이펙트를 가집니다
  require('../displayUser');

  const $ = require('jquery');
  const fetchCurrentUser = require('../fetchCurrentUser');

  // fetchCurrentUser 모의 함수가 일부 데이터를 가지고 콜백을
  // 자동으로 호출하도록 지시힙니다
  fetchCurrentUser.mockImplementation(cb => {
    cb({
      fullName: 'Johnny Cash',
      loggedIn: true,
    });
  });

  // Use jquery to emulate a click on our button
  $('#button').click();

  // Assert that the fetchCurrentUser function was called, and that the
  // #username span's inner text was updated as we'd expect it to.
  expect(fetchCurrentUser).toBeCalled();
  expect($('#username').text()).toEqual('Johnny Cash - Logged In');
});
```

테스트 되고 있는 함수는 `#button` DOM 요소에 이벤트 리스터를 추가하므로, 테스트에 대해 DOM을 올바르게 설정해야 합니다. Jest는 브라우저에 있는 것 처럼 DOM 환경을 시뮬레이션하는 `jsdom`을 탑재합니다. 이는 호출하는 모든 DOM API가 브라우저에서 보이는 것과 동일한 방식으로 보여질 수 있다는 것을 의미합니다!

테스트가 실제 네트워크 요청을 하지 않고 대신 로컬로 모의 데이터를 리졸브하도록 하기 위해 `fetchCurrentUser.js`를 모의하고 있습니다. 이것은 초 단위가 아닌 밀리 초 단위로 테스트가 완료될 수 있도록 하고 빠른 단위 테스트 반복 속도를 보장합니다.

이 예에 대한 코드는 [examples/jquery](https://github.com/facebook/jest/tree/master/examples/jquery)에서 사용 가능 합니다.
