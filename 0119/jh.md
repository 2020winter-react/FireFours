### 질문:

loading의 상태를 관리할 때 `loading.GET_POST`가 아니라 `loading['sample/GET_POST']`의 형태로 써야하는 특별한 이유가 있는지?

```javascript
export default connect(
  ({ sample, loading }) => ({
    post: sample.post,
    users: sample.users,
    loadingPost: loading["sample/GET_POST"],
    loadingUsers: loading["sample/GET_USERS"]
  }),
 (...)
```

### 해결:

`sample/GET_POST`에 /가 있기 때문에 loading과 .으로 연결할 수 없다. 그래서 []를 쓴 것.
그리고 `loading.GET_POST`을 써도 똑같이 잘 작동한다. 특별한 이유로 위와 같이 쓴 것은 아닌 것으로 함께 결론지었다.

<hr />

### 질문: 'redux-saga를 사용한 비동기 카운터'의 작동 흐름 정리

```javascript
// modules/counter.js

import { createAction, handleActions } from "redux-actions";
import { delay, put, takeLatest, takeEvery } from "redux-saga/effects";

const INCREASE = "counter/INCREASE";
const DECREASE = "counter/DECREASE";

const INCREASE_ASYNC = "counter/INCREASE_ASYNC";
const DECREASE_ASYNC = "counter/DECREASE_ASYNC";

// 4) increase, decrease가 실행되어 INCREASE, DECREASE 액션이 생성된다.
export const increase = createAction(INCREASE);
export const decrease = createAction(DECREASE);

// 1) Counter컴포넌트의 +1버튼, -1버튼을 클릭하면 increaseAsync, decreaseAsync가 실행되면서 INCREASE_ASYNC, DECREASE_ASYNC 액션을 생성한다.
export const increaseAsync = createAction(INCREASE_ASYNC, () => undefined);
export const decreaseAsync = createAction(DECREASE_ASYNC, () => undefined);

// 3) increaseSaga, decreaseSaga가 실행되면서 각각 increase, decrease 액션을 디스패치한다.
function* increaseSaga() {
  yield delay(1000);
  yield put(increase());
}

function* decreaseSaga() {
  yield delay(1000);
  yield put(decrease());
}

// 2) INCREASE_ASYNC, DECREASE_ASYNC 액션이 들어올 때 각각 increaseSaga, decreaseSaga(제너레이터함수=사가)를 실행해준다.
export function* counterSaga() {
  yield takeEvery(INCREASE_ASYNC, increaseSaga);
  yield takeLatest(DECREASE_ASYNC, decreaseSaga);
}

const initialState = 0;

const counter = handleActions(
  {
    // 5) INCREASE, DECREASE액션이 실행되며 state에 +1 혹은 -1이 적용되어 state의 값이 변한다!
    [INCREASE]: state => state + 1,
    [DECREASE]: state => state - 1
  },
  initialState
);

export default counter;
```

### 해결:

작동하는 흐름을 순서대로(step1 ~ step5) 정리하여, 위 코드에 주석으로 달아놓았다.

<hr />
