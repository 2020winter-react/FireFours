## 12월 30일 내용


### Q1. getSnapshotBeforeUpdate이 가지는 의미와 사용법

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


### 







