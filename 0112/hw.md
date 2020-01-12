### React 라우터

##### 리액트 라우터는 기본적으로 match, location, histroy라는 객체를 갖고 있습니다. 특히 URL 파라미터를 사용할 때는 라우트로 사용되는 컴포넌트에서 받아 오는 match라는 객체 안의 params 값을 참조합니다. 이 match 객체는 현재 컴포넌트가 어떤 경로 규칙에 의해 보이는지에 대한 정보가 들어 있습니다.

```jsx
const data = {
  wook: {
    name: '변형욱',
    description: '리액트 스터디'
  },
 cheolsoo: {
    name: '김철수',
    description: '앵귤러 스터디'
  }
}
```
```jsx
const Profile = ({ match }) => {
  const { username } = match.params;
  const profile = data[username];
  (...)
}
```
```jsx
(...)
<Route path="/profiles/:usename" component={Profile} />
```
##### Profile함수는 match 객체를 props로 받는 함수입니다. 함수 내부에 정의된 username은 match.params의 값으로 사용자가 임의로 설정한 상수입니다. username은 Route path에서 path 규칙을 따라 /profile/:username으로 표기됩니다. 이렇게 설정하여 match.params.username 값을 통해 현재 username 값을 조회할 수 있습니다.
##### 아래는 match 객체가 갖고 있는 값들입니다.
#### match
```jsx
{
  "path": "profile",
  "url": "/profile",
  "isExact": false,
  "params": {}
}
```

### 리덕스에서 초기값, 액션생성함수 props로 받아오기
##### redux를 사용할 때는 액션 타임, 액션 생성 함수, 리듀서 코드를 작성합니다. 이 코드들을 각각 다른 파일에 작성할 수도 있고, 기능별로 묶어서 파일 하나에 작성하는 방법도 있습니다. 이 중 기능별로 묶어서 파일 하나에 작성하는 방법을 Ducks 패턴이라고 합니다. 이 구조는 크게 components, modules, containers의 하부 디렉토리로 구성됩니다. components는 UI를 담당하는 컴포넌트들이 포함되어 있고 modules에는 액션객체, 액션 생성 함수, 리듀서가 포함되고 containers에는 액션을 dispatch하는 컴포넌트들이 속해 있습니다.
##### react-redux에서는 connect 함수를 제공합니다. 이 함수는 다음과 같이 사용합니다.
```jsx
connect(mapStateProps, mapDispatchProps)(연동할 컴포넌트)
```
##### 여기서 mapStateToProps는 리덕스 스토어 안의 상태를 컴포넌트의 props로 넘겨주기 위해 설정하는 함수이고, mapDispatchToProps는 액션 생성 함수를 컴포넌트의 props로 넘겨주기 위해 사용하는 함수입니다.
```jsx
containers/CounterContainer.js
import React from 'react';
import { connect } from 'react-redux';
import Counter from '../components/Counter';
import { increase, decrease } from '../modules/counter';

const CounterContainers = ({ number, increase, decrease }) => {
    return (
        <Counter number={number} onIncrease={increase} onDecrease={decrease)/>
    );
};

export default connect(
    state => ({
        number: state.counter.number,
    }),
    {
        increase,
        decrease
    },
)(CounterContainer);
```
##### 위의 CounterContainer.js의 코드를 살펴보면 CounterContainers 컴포넌트는 number, increase, decrease를 props로 받고 있습니다. import문에서 increase와 decrease는 불러오는데 number는 '어디서 온 것일까?'라는 의문을 가질 수 있을 것입니다. 이는 connect 함수를 코드 본문에 따로 명시하지 않고 export default를 통해 바로 export하고 있기 때문에 나타나는 상황입니다. 위에 언급했듯이 connect 함수는 첫번째 파라미터는 리덕스 스토어에서 받아온 상태 값을 넣어 줍니다. 다시 말해, props로 받아온 상태값 number는 export default connect()를 통해 받아온 값입니다.
##### 아래는 다른 예시입니다.

```jsx
containers/TodosContainer.js
import React from 'react';
import { connect } from 'react-redux';
import Todos from '../components/Todos';
import { changeInput, insert, toggle, remove } from '../modules/todos';

const TodosContainer = ({
    input,
    todos,
    changeInput,
    insert,
    toggle,
    remove,
}) => {
    (...)
};

export default connect(
    ({ todos }) => ({
        input: todos.input,
        todos: todos.todos,
    }),
    {
        changeInput,
        insert,
        toggle,
        remove,
    },
)(TodosContainer);
```
##### 참조 '리액트를 다루는 기술'
