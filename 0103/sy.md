## P.298 useState의 함수형 업데이트

useState에서 새로운 상태 값을 넣어 상태를 업데이트 시켜줄 수도 있지만, 아예 업데이트는 함수를 넣어줄 수도 있다. 이를 함수형 업데이트라 부르며 여러 장점이 존재한다. 비슷하게 함수형 업데이트에 대한 언급이 클래스 컴포넌트의 setState에서도 나온다. 이때 함수형 업데이트의 장점은 **이전 값을 받아 update**하는 것으로 setState의 비동기성 때문에 [일어나는 문제](https://link.medium.com/yjBS2DnuW2)에 대한 해결 방안이였다.<br>
특히 책에서 소개하는 장점은 이 함수형 업데이트를 통해 성능을 크게 올릴 수 있다는 점이었다.

> 단순히 `()=>` 를 추가하는 것으로 어떻게 크게 성능을 올릴 수 있었을까? 

그 물음에 대해선 단순히 ()=> 를 추가해서 성능이 좋아진 것이 아니라 useCallback과의 시너지 덕분이었다고 답할 수 있을 것 같다.<br>
onRemove 함수를 예를 들어보면, 이전엔 useCallback의 두번째 인자로 todos를 주어 todos의 값이 변할 때마다 새로운 함수를 생성해주었다.

```jsx
const onRemove = useCallback(id => {
  setTodos(todos.filter(todo => todo.id !== id));
}, [todos]);
```

다른 state가 변해도 함수가 생성하지 않는다는 장점이 있었지만, 여전히 todos의 변화에 따라 함수를 새로 생성해야한다는 오버헤드가 존재한다.<br>
그러므로 다음과 같이 **함수형 업데이트**를 사용할 수 있다.

```jsx
const onRemove = useCallback(id => {
  setTodos(todos => todos.filter(todo => todo.id !== id));
}, []);
```

이렇게 되면 두번째 인자는 빈 배열로 주어 처음 mount될 때 생성된 이후로 [새로 생성하지 않는다](https://github.com/2020winter-react/study/blob/master/1230.md#8%EC%9E%A5-usecallback-%ED%95%A8%EC%88%98%EC%97%90-%EB%8C%80%ED%95%9C-%EC%8B%AC%EC%B8%B5-%EC%9D%B4%ED%95%B4). 


만약 함수형 업데이트가 아닌 아래 코드와 같이 그냥 빈배열로만 바꾼다면 todos엔 처음 초기값만 존재하고 이후에 업데이트되는 값을 존재하지 않게 된다.

```jsx
const onRemove = useCallback(id => {
  setTodos(todos.filter(todo => todo.id !== id));
}, []); // 문제 발생, todos가 바뀌지 않음
```

함수형 업데이트는 이러한 문제를 해결하고 새롭게 업데이트되는 `todos`를 그때그때 함수로써 입력받아 실행이 가능하다. <br>
위와 같이 함수형 업데이트는 useCallback의 함수 생성을 단 한번만 해도 되는 것으로 개선시켜준다. 비슷한 효과를 주는 것이 useReducer로 이것 또한 상태의 업데이트 로직을 분리하고 업데이트에 따른 생성을 막아 성능 개선을 꾀할 수 있다.

## useState 여러 가지 유의점

useState에 대해 찾아보며 몇가지 사용의 유의점이 있었다.

### 1. State를 나눌 때는 주로 같이 update가 일어나는 것으로 묶어라

`useState({row:10, col:20, tuple:200})` 이런 식으로 여러 개의 상태 값을 오브젝트 형식으로 하나의 state에서 관리할 수도 있고 `useState(10), useState(20), useState(200)` 이런 식으로 상태값을 모두 쪼개어 관리도 가능하다. 이에 대한 기준은 [리액트 공식문서](https://reactjs.org/docs/hooks-faq.html#should-i-use-one-or-many-state-variables)에서 설명하고 있다. 

결론은 상태 업데이트에서 기존의 값을 복사하고 이를 업데이트 하는 것에 오버헤드가 크니, 주로 상태 업데이트가 같이 일어나는 상태값 위주로 묶어서 관리하라는 것이다. 너무 쪼개 놓아도 각각을 업데이트 하는 것에 오버헤드가 크고, 너무 하나의 몰아 넣는 것도 기존의 값을 복사하는 것에 오버헤드가 크므로 분리시켜주는 것이 중요하다.

### 2. event는 재사용될시 null로 변하므로 const로 값을 저장하라

```jsx
setMessage(p => {
  return { ...p, message: e.target.value };
}); // Doesn't work

```
위와 같이 useState를 사용하면 다음과 같은 오류가 난다. 

```
Warning: This synthetic event is reused for performance reasons. If you're seeing this, you're accessing the property `target` on a released/nullified synthetic event. This is set to null. If you must keep the original synthetic event around, use event.persist(). See https://fb.me/react-event-pooling for more information.
```

[공식 문서의 event pooling](https://reactjs.org/docs/events.html#event-pooling) 설명에 따르면 synthetic event 객체 자체가 재사용되면서 모든 속성이 비워지기 (nullified) 때문에 이를 비동기적인 방식으로 사용하는 것은 불가능하다는 것이다. 여기서 비동기적인 방식이 setMessage가 되므로 이를 고치려면 미리 const로 해당 값을 저장하고 이를 업데이트 해야한다. 코드는 아래와 같다.


```jsx
const msg = e.target.value;
setMessage(p => {
  return { ...p, message: msg };
}); 
```




