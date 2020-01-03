# Shallow copy and deep copy
##### :얕은 복사와 깊은 복사

###### 목차
- 기본 개념


###### 기본 개념

자바스크립트에서의 복사는 크게 두가지가 있다. 객체내부에 있는 객체들의 "참조"만 복사하는 shallow 복사와 immutable 한 "값"까지 복사하는 deep한 복사가 있다. 

이는 평소 우리가 코딩을 배울 때 함수의 값 복사에서 사용하는 call by ref와 call by value의 개념을 생각해보면 이해가 쉽다. call by ref는 함수에서 변수를 읽어올 때 호출시 대입되는 변수의 참조값을 읽어와 변수에 직접 접근하여 값을 바꾼다. 반면 call by value 는 파라미터로 들어오는 변수가 저장하고 있는 '값' 즉 데이터만 읽어오게 된다. 

자바스크립트의 얕은복사와 깊은복사도 이 개념에 비교해서 볼 수 있다.
 - shallow한 복사는 복사를 할 때는 call by ref 와 같이 복사하는 대상 내부의 객체들의 참조만을 가져오게 되어 복사한 객체에서 내부 객체의 값을 변경하면 복사하기 전의 기존 객체도 영향을 받게 된다. 
 - deep한 복사는 call by value와 같이 내부 객체의 immutable한 값만을 복사해오기 때문에 복사된 값을 바꾸더라도 복사되기 전의 객체에는 영향이 없다.

코드를 보며 알아보자

- 깊은 복사
    ```JavaScript
    let valueDeep1={obj:{a:10}};
    let valueDeep2={...valueDeep1,obj:{...value.obj}};
    console.log(`valueDeep1: ${valueDeep1}`);
    valueDeep2.obj.a=20
    console.log(`valueDeep1: ${valueDeep1}`);//바뀌지 않는다.{obj:{a:10}};
    ```
    위의 출력 값은 둘 다 바뀌기 전 값인"10"이 출력된다. 그 이유는 가장 바깥의 객체를... 이용해 복사해준 후 내부의 객체또한 ... 연산자를 이용해 한번 더 복사해주었기 때문에 immutable한 "값" 자체가 복사되었기 때문이다. 

 
- 얕은 복사

    ```JavaScript
        let valueDeep1={obj:{a:10}};
        let valueDeep2={...valueDeep1};
        console.log(`valueDeep1: ${valueDeep1}`);
        valueDeep2.obj.a=20
        console.log(`valueDeep1: ${valueDeep1}`);//바뀌게 된다. valueDeep1:{a:20};
    ```

    위의 출력 값중 첫 번째는 "{obj:{a:10}}" 가 출력되고 두 번 째 값은 "{obj:{a:20}}"으로 원본 객체 내부의 obj.a값이 바뀌어서 출력이 될 것이다. 그 이유는 spread (...)연산을 이용해 객체 내부에 객체가있는,{{}} 구조의 객체를 복사할 경우 가장 바깥의 단계만 복사가 되는 얕은 복사가 되기 때문이다. 






