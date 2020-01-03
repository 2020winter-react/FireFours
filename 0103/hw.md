# React.memo (리액트 성능 최적화)

##### React.memo는 고차 컴포넌트(Higher Order Component)입니다. 함수 컴포넌트가 동일한 props로 동일한 결과를 렌더링한다면, React.memo를 호출하고 결과를 메모이징(memoizing) 하도록 래핑하여 성능향상을 누릴 수 있습니다. 다시 말해, React는 컴포넌트를 렌더링하지 않고 마지막으로 렌더링된 결과를 재사용합니다. props가 갖는 복잡한 객체에 대하여 얕은 비교만을 수행하는 것이 기본 동작입니다. 다른 비교 동작을 원한다면, 두 번째 인자로 별도의 비교 함수를 제공하면 됩니다. 아래 참조
```jsx
function MyComponent(props) {
  /* props를 사용하여 렌더링 */
}
function areEqual(prevProps, nextProps) {
  /*
  nextProp가 prevProps와 동일한 값을 가지면 true를 반환하고, 그렇지 않다면 false를 반환
  */
}
export default React.memo(MyComponent, areEqual);
```
```jsx
React.memo(Component, [areEqual(prevProps, nextProps)]);
```
```jsx
function moviePropsAreEqual(prevMovie, nextMovie) {
  return (
    prevMovie.title === nextMovie.title &&
    prevMovie.releaseDate === nextMovie.releaseDate
  );
}

const MemoizedMovie2 = React.memo(Movie, moviePropsAreEqual);
```
##### 특정 컴포넌트의 props를 비교를 하는 함수를 만들어 React.memo의 두번째 매개변수에 적용하면 리렌더링을 하기 전에 해당 props값의 변화 유무를 확인합니다.

##### 하지만, 특정 props를 비교하는 areEqual 함수를 잘못 사용하면 버그가 발생할 수 있습니다. 예를 들어, 함수형 업데이트를 하지 않은 상태로 props의 특정 요소만 비교한다면 정작 props를 참조해야 하는 메소드에서 최신 props 배열이나 객체(갱신된 값이 반영된)를 참조하지 않으므로 심각한 문제를 야기할 수 있습니다.

```javascript
  // Test for A's keys different from B.
  for (let i = 0; i < keysA.length; i++) {
    if (
      !hasOwnProperty.call(objB, keysA[i]) ||
      !is(objA[keysA[i]], objB[keysA[i]])
    ) {
      return false;
    }
  }
 ```
 ##### 위에 제시된 코드는 React.memo의 얕은 복사를 보여주는 코드입니다. if문을 통해 알 수 있듯이 React.memo는 주어진 key(props)에 대해 얕은 복사를 하고 있고 만약 key 속에 객체나 배열이 있을 경우 반영을 못 할 것으로 예상할 수 있습니다.
 
 ###### 참조 https://ko.reactjs.org/docs/react-api.html#reactmemo       https://ui.toast.com/weekly-pick/ko_20190731/
 
