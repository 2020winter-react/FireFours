# 2020년 1월 12일

장소 - 용산역

## P407 static contextType의 원리

기존의 컴포넌트에서 Context API를 사용할 때 Provider와 Consumer를 이용해 복잡한 과정을 거쳐야 했다면 <br>
Hook에선 useContext를 이용해 훨씬 간단하게 Context를 사용할 수 있다. <br>
클래스형 컴포넌트에선 static contextType을 이용해 비슷한 효과를 볼 수 있다.
하지만 단순히 contextType에 지정해줬다고 해서 this.context를 이용해 사용할 수 있다는 점이 신기하였다.

이를 더 자세하게 알아보기 위해 Component를 정의한 코드를 찾아보았고 아래와 같다.

``` ts
// Base component for plain JS classes
// tslint:disable-next-line:no-empty-interface
interface Component<P = {}, S = {}, SS = any> extends ComponentLifecycle<P, S, SS> { }
class Component<P, S> {
    // tslint won't let me format the sample code in a way that vscode likes it :(
    /**
        * If set, `this.context` will be set at runtime to the current value of the given Context.
        *
        * Usage:
        *
        * ```ts
        * type MyContext = number
        * const Ctx = React.createContext<MyContext>(0)
        *
        * class Foo extends React.Component {
        *   static contextType = Ctx
        *   context!: React.ContextType<typeof Ctx>
        *   render () {
        *     return <>My context's value: {this.context}</>;
        *   }
        * }
        * ```
        *
        * @see https://reactjs.org/docs/context.html#classcontexttype
        */
    static contextType?: Context<any>;
```
위와 같이 Componet의 정의 속에 `static contextType?: Context<any>;`이 존재함을 알 수 있다.
인자로 아예 Context만을 받고 있기 대문에 Context 자체로 지정해주면 알아서 이를


## Context API
- static contexttype 의 정체
- export 했을 때 이름을 바꿀 수 있다.
- useContext에서도 actions 사용 가능
