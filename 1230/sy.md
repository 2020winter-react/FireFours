## 12월 30일 내용


### (7장) Q1. getSnapshotBeforeUpdate이 가지는 의미와 사용법

예시로 스크롤 위치를 위해 쓰인다는 말이 있지만 구체적으로 어떤 방식으로 사용되는지 이해가 안됐으나 
React 공식 문서에서 그 사용법을 알 수 있었다.

``` javascript
class ScrollingList extends React.Component {
  constructor(props) {
    super(props);
    this.listRef = React.createRef();
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // 어떤 값이 추가됐다면 그때의 스크롤 위치를 snapshot으로 반환
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current;
      // snapshot 값으로 Return
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // snapshot을 통해 받아온 스크롤 위치로 바꿔준다. 
    // (위 getSnapshot.. 함수에서 넘어온 스크롤 값이 snapshot에 저장되어 있다)
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}
```

getSnapshotBeforeUpdate 와 componentDidUpdate는 둘 다 render가 이뤄진 후라는 점은 같으나
**React DOM 및 refs 업데이트** 의 전후라는 차이가 있다. 
state의 변화가 반영되어 있지만 실제 화면에 뿌려지기 전후라고 이해할 수 있다.
위 예시에선 DOM이 업데이트 되면서 스크롤이 최상단으로 가기 때문에 그 전에 그 위치를 저장하고 업데이트 이후 다시 그 위치로 변경시켜주는 것이다.
그 외에도 DOM 변화와 관련된 업데이트에 대해서 사용할 수 있을 것이다.


### (4장) 이벤트핸들링 함수의 다른 방안

기존의 target.name을 활용한 onChange 함수에서 더 많은 인자를 받을 수 있는 함수로 변환해볼 수 있다.


``` javascript
  const [data, setData] = useState({user:'cho', pw: 123});
  
  // key를 받아 해당 key값의 state를 변경
  const onChange = key => e => {
    setData({...data, [key]: e.target.value});
  };
  
  const onChange2 = e => {
    setData({...data, [e.target.name]: e.target.value});
  };
  
  return (<>
    <input
    type="text"
    placeholder="유저명"
    value={data.user}
    onChange={onChange("user")} // {(event) => (onChnage("user"))(event)} 도 똑같이 동작한다.
  />
   <input
    type="text"
    name="pw"
    placeholder="패스워드"
    value={data.pw}
    onChange={onChange2}
  />)
```


### (8장) useCallback 함수에 대한 심층 이해

 useCallback 함수는 useMemo 함수와 비슷하게 어떤 state의 변화가 생겼을 때에만 함수가 생성되도록 하는 Hook이다.
 이해가 힘들었던 것은 어떤 것은 처음 렌더링 될 때만 함수를 생성하고, 어떤 함수는 state 변화에 따라 함수를 생성하는데
 이 차이가 무엇인지 와닿지 않았다. 이 그 두 함수는 아래와 같다.

``` javascript
  const onChange = useCallback(e => {
    setNumber(e.target.value);
    console.log('number:', number);
  }, []); // 컴포넌트가 처음 렌더링 될 때만 함수 생성

  const onInsert = useCallback(() => {
    const nextList = list.concat(parseInt(number));
    setList(nextList);
    setNumber('');
    inputEl.current.focus();
  }, [number, list]); // number 혹은 list 가 바뀌었을 때만 함수 생성

  return (
    <div>
      <input value={number} onChange={onChange} ref={inputEl} />
      <button onClick={onInsert}>등록</button>
      <ul>
        {list.map((value, index) => (
          <li key={index}>{value}</li>
        ))}
      </ul>
      <div>
        <b>평균값:</b> {avg}
      </div>
    </div>
  );

```

이를 더 잘 이해하기 위해 몇가지 실험을 하였다. 
먼저 `console.log('number:', number);` 를 통해 onChnage 속에서 number state를 보면 어떤 값을 입력하더라도 값이 없다고 나온다.
**즉 onChange가 처음 생겼을 때 number 상태 그대로를 가지고 있다** 반면 onInsert는 실행할 때마다 list, number의 값이 바뀌었다.
그리고 onChnage 두번째 인자로 `[number]`를 넣으면 number가 계속 변한다.

두번째로 onInsert의 두번째 인자를 모두 삭제하면 아래와 같이 된다.

```
  const onInsert = useCallback(() => {
    const nextList = list.concat(parseInt(number));
    setList(nextList);
    setNumber('');
    inputEl.current.focus();
  }, []); // 컴포넌트가 처음 렌더링 될 때만 함수 생성

```

위 상태에선 "Warning: Received NaN for the `children` attribute. If this is expected, cast the value to a string." 란 오류가 뜬다. 이 말인 즉슨 Nan 을 li 에서 받아 경고창을 보냈다. 즉 위 함수에서 parseInt(number)의 number가 Nan이였기 때문이다. number의 처음상태가 빈값이라 그 이후에도 실행시켜도 빈값이 반환된 것이다. 

> 정리하자면, 어떤 state의 값을 이용하여 새로운 값으로 변환하는 것이라면 state 변화에 따라 함수를 생성하고, static 하게 단순한 함수 실행이라면 처음 렌더링될때만 함수가 생성하도록 한다.





