### react-virtualized에서 List 컴포넌트의 height (p.307)
height는 렌더링된 할 일 리스트들의 높이를 모두 합친, 전체 리스트의 높이였다. (57 * 9 = 513) 
즉, rowRenderer를 통해 만든 각 TodoListItem들의 높이를 모두 합친 길이이다. 


### rowRenderer함수에서 style의 역할 (p.308)
컴포넌트 최적화를 위해 react-virtualized에서 제공하는 List 컴포넌트를 사용했는데, 여기에서 rowRenderer 함수를 제공한다. rowRenderer 함수는 List 컴포넌트에서 각 TodoListItem을 렌더링하는 데 사용된다.

```javascript
const TodoList = ({ todos, onRemove, onToggle }) => {
	const rowRenderer = useCallback(
		({ index, key, style }) => {
			const todo = todos[index];
			return (
				<TodoListItem
					todo={todo}
					key={key}
					onRemove={onRemove}
					onToggle={onToggle}
					style={style}
				/>
			);
		},
		[onRemove, onToggle, todos],
	);

	return (
		<List
			(...)
			rowRenderer={rowRenderer}
		/>
	);
};
```
이 rowRenderer 함수는 다양한 파라미터를 받아올 수 있는데, 그 중 style props를 받아와 TodoListItem에도 사용할 수 있다. TodoListItem 컴포넌트의 최상단의 태그에, props로 받아온 style을 적용시켜주면, 투두리스트가 잘 렌더링된다.

궁금했던 점은, 여기서 style이 어떤 역할을 하는지에 대해서였다. style을 지우고 리스트를 스크롤해보니, 아래의 다음 투두리스트들이 렌더링되지 않고, 기존의 위의 투두리스트들만 보여졌다.

고민하고 찾아본 결과, style은 행의 positioning을 위해 행에 적용되는 객체였다. 행의 사이즈와 위치를 정의해주기 때문에, 렌더링되는 행 요소에 반드시 넣어야 한다. 그래서 렌더링되는 행 요소인 TodoListItem 요소에 적용된 것이었다.

react virtualized의 리스트 컴포넌트에서 스크롤이 잘 작동하게 하기 위해서는, 위와 같이 style을 통한 sizing&positioning이 필요하다. 

