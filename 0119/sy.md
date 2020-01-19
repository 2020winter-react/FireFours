
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

여기서 왜 Promise.resolve로 감싸는지가 잘 이해가 가지 않았다.
위 설명은 설령 resolve 함수가 프로미스를 반환하지 않더라도, 프로미스 취급을 하기 위한다고 하나 <br>
(1) 그럼 resolve가 **이미 프로미스인 경우 또 프로미스로 감쌌을 때 문제가 생기지 않을까? <br>
(2) 그리고 그냥 일반적인 값을 감쌌을 때 어떻게 결과가 나올까?** <br>
직접 위의 경우를 대입해보니 이해가 금방 갔다.


(1) 아래와 같이 3미만을 입력했을 때 reject가 되는 Promise를 만들고 
이를 그냥 실행했을 때와 Promise.resolve()로 감쌌을 때를 비교하였다.
**결과는 둘의 차이가 없었다.**
```js
const bigger_than_3 = (i) => {
    return new Promise((resolve, reject) => 
      { if (i>3) { resolve(true) } 
        else { reject(false)} }
)}

bigger_than_3(4)
//>> Promise {<resolved>: true}

Promise.resolve(bigger_than_3(4))
//>> Promise {<resolved>: true}

bigger_than_3(2)
//>> Promise {<rejected>: false}

Promise.resolve(bigger_than_3(2))
//>> Promise {<rejected>: false}
```


(2) 그렇다면 일반적인 값의 경우엔 어떨까? 아래의 경우를 살펴보면 어떤 값이 더라도 <br>
**resolved를 반환한다.**
```js
Promise.resolve(10)
//>> Promise {<resolved>: 10}
Promise.resolve(false)
//>> Promise {<resolved>: false}
Promise.resolve(true)
//>> Promise {<resolved>: true}
Promise.resolve(undefined)
//>> Promise {<resolved>: undefined}
```

이러한 Preloader를 컴포넌트화 하여 아래와 같이 resolve해야할 Promise를 넣어준다.
이렇게 되면 Context에 getUsers가 담기게된다.
```jsx
return (
  <>
    <Users users={users} />
    <Preloader resolve={getUsers} />
  </>
);
```

마지막 index.server.js에서 렌더링 전에 Context 속 모든 Promise를 기다리도록 합니다.

```jsx
//index.server.js
  try {
    await sagaPromise; // 기존에 진행중이던 saga 들이 모두 끝날때까지 기다립니다.
    await Promise.all(preloadContext.promises); // 모든 프로미스를 기다립니다.
  } catch (e) {
    return res.status(500);
  }
  preloadContext.done = true;
  const root = ReactDOMServer.renderToString(jsx); // 렌더링을 합니다.
```



