
## (P570) SSR에서 PreloadContext 만들기 이해하기

서버 사이드 렌더링에선 useEffect나 componentDidMount에서 설정한 작업이 호출되지 않기 때문에, <br> 
이 안에서 Data를 Fetching 하는 작업이 있다면 이를 꼭 미리 요청하여 스토어에 담아두어야 렌더링할 때 필요한 데이터를 보여줄 수 있습니다.

이를 위한 작업을 **PreloadContext**란 Context API를 만들어 미리 처리해야할 요청들을 담고 <br>
**Preloader/usePreloader**라는 요청을 담을 수 있는 함수/Hook을 만든다.

```jsx
//src/lib/PreloaderContext.js

import { createContext, useContext } from 'react';

// 클라이언트 환경: null
// 서버 환경:{ done: false, promises: [] }
const PreloadContext = createContext(null);
export default PreloadContext;

// resolve는 함수 타입입니다.
export const Preloader = ({ resolve }) => {
  const preloadContext = useContext(PreloadContext);
  if (!preloadContext) return null; // context 값이 유효하지 않다면 아무것도 하지 않음
  if (preloadContext.done) return null; // 이미 작업이 끝났다면 아무것도 하지 않음

  // promises 배열에 프로미스 등록
  // 설령 resolve 함수가 프로미스를 반환하지 않더라도, 프로미스 취급을 하기 위하여
  // Promise.resolve 함수 사용
  preloadContext.promises.push(Promise.resolve(resolve()));
  return null;
};
```
위 코드에서 핵심이 되는 코드 `preloadContext.promises.push(Promise.resolve(resolve()));` 는 
Promise.resolve로 감싼 resolve()를 Context에 추가하는 작업을 진행한다.

여기서 왜 Promise.resolve로 감싸는지가 잘 이해가 가지 않았으나 직접 여러 경우를 대입해보니 이해가 갔다.


```
const bigger_than_3 = (i) => {
    return new Promise((resolve, reject) => 
      { if (i>3) { resolve(true) } 
        else { reject(false)} }
)}



```





```jsx
import React from 'react';
import Users from '../components/Users';
import { connect } from 'react-redux';
import { getUsers } from '../modules/users';
import { Preloader } from '../lib/PreloaderContext';

const { useEffect } = React;

const UsersContainer = ({ users, getUsers }) => {
  // 컴포넌트 마운트될 때 호출
  useEffect(() => {
    if (users) return; // users가 이미 유효하다면 요청하지 않음
    getUsers();
  }, [getUsers, users]);
  return (
    <>
      <Users users={users} />
      <Preloader resolve={getUsers} />
    </>
  );
};

export default connect(
  state => ({
    users: state.users.users
  }),
  {
    getUsers
  }
)(UsersContainer);


```
