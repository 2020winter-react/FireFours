## P.298 useState의 함수형 업데이트

useState에서 새로운 상태 값을 넣어 상태를 업데이트 시켜줄 수도 있지만, 아예 업데이트는 함수를 넣어줄 수도 있다.<br>
이를 함수형 업데이트라 부르며 여러 장점이 존재한다. 특히 책에서 소개하는 장점은 이 함수형 업데이트를 통해 성능을 크게 올릴 수 있다는 점이었다.

단순히 `()=>` 를 추가하는 것으로 어떻게 크게 성능을 올릴 수 있었을까? 

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

만약 함수형 업데이트가 아닌 
```jsx
const onRemove = useCallback(id => {
  setTodos(todos.filter(todo => todo.id !== id));
}, []);
```
만약 함수형 업데이트를 하지 않고 두번째 인자를

“setState 를 함수형으로 사용하기” - “setState 를 함수형으로 사용하기” by 김토성 https://link.medium.com/yjBS2DnuW2
