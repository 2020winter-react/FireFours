### 질문:

App.js에 렌더링된 CounterContainer 컴포넌트에서, number를 가리킬 때 state.counter.number로 가르킬 수 있는가?

```javascript
const mapStateToProps = state => ({
  number: state.counter.number
});
```

### 해결:

(1)

```javascript
const mapStateToProps = state => ({
  number: state.counter.number
});
```

mapStateToProps는 state를 파라미터로 받아오며, 이 값은 <b>'현재 스토어가 지니고 있는 상태'</b>를 가리킨다.

(2)
리듀서인 counter와 todos를 묶은 rootReducer를 사용해서 store를 생성했다.
react-redux에서 제공하는 Provider컴포넌트로 App 컴포넌트를 감싸면(이때 store를 props로 전달해주어야 함), 리액트 컴포넌트에서 스토어를 사용할 수 있다.

```javascript
const store = createStore(rootReducer);

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);
```

(3)
(2)를 통해 리액트 컴포넌트에서 스토어 사용이 가능해지므로, 현재 스토어가 지니고 있는 상태인 state에도 접근할 수 있고, 스토어 내의 리듀서에도 접근할 수 있다.
따라서 state.counter.number를 쓴 것이다.

<hr />

### 질문: ColorProvider를 쓸 때의 이점은?

### 해결:

```javascript
<ColorContext.Provider value={{ color: 'black' }}>
```

기존에는 위와 같은 식으로 코드를 썼는데, 이와 달리 ColorProvider를 쓰면 하단의 코드와 같다.

```javascript
//App.js

  return (
    <ColorProvider>
        <ColorBox />
    </ColorProvider>
  );
}
```

```javascript
//contexts/color.js

const ColorProvider = ({ children }) => {
  const [color, setColor] = useState("black");
  const [subColor, setSubColor] = useState("red");

  const value = {
    state: { color, subColor },
    actions: { setColor, setSubColor }
  };

  return (
    <ColorContext.Provider value={value}>{children}</ColorContext.Provider>
  );
};
```
ColorProvider를 썼을 때의 이점은 다음과 같다.
1) App.js에서 사용되는 코드가 깔끔해진다.
2) <ColorContext.Provider>에서는 변화시킬 value을 props로 직접 주었는데, ColorProvider에서는 변화시킬 값을 따로 관리할 수 있다. 이렇게 value를 props로 직접 전달하지 않고 따로 관리함으로써, context의 value에 상태값 뿐만 아니라 동적인 함수도 용이하게 전달해줄 수 있다. 또한 value를 state와 actions로 나누어 관리할 수 있다는 점도 ColorProvider의 장점이다. 나누어 관리하는 것은 필수사항은 아니지만 이후 편리한 context값 사용을 위해 해놓는 것이 좋다.

<hr />
