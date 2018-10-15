# Functional setState

```javascript

onChangeEvent({target}){
    this.setState({
        value : target.value
    })
}

```

제목처럼 `setState()`는 **Functional Programming** 이다.

<br/>

## Functional Programming

[함수형 프로그래밍](https://github.com/BKJang/Concept_of_FrontEnd/blob/master/JavaScript/%ED%95%A8%EC%88%98%ED%98%95%20%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D.md)에 있어서 가장 중요한 것은 2가지로 요약할 수 있다.

1. 순수함수(Pure Function)
    1. 순수 함수는 **같은 입력 값을 넣었을 때 항상 같은 출력 값을 반환**한다.
    2. 유용한 순수 함수는 **최소 한 개의 매개변수**를 가진다.
    3. 유용한 순수 함수는 **반드시 무엇인가를 반환**한다.
2. 불변성(Immutable)


<br/>

## Component 기반의 UI 라이브러리

먼저 React는 **Component 기반**의 UI 라이브러리이다.

React의 Component는 **보통 자체 상태(State)가 포함 된 독립적이고 재사용 가능한 코드 조각(pieces of code)**이다.

이것은 어플리케이션의 UI를 구성하는 리액트 엘리먼트(element)를 반환한다.

**로컬 상태를 포함하는 컴포넌트에는 state라는 속성(property)이 있다.**

this.props 및 this.state는 비동기적(asynchronously)으로 업데이트 될 수 있으므로 다음 상태(state)를 계산하는 데는 이 값을 의존해서는 안된다.

<br/>

## setState()
  
- 업데이트 할 상태의 부분을 포함하는 **객체**를 `setState`의 인자로 전달한다.
- 전달한 객체에는 상태의 키에 해당하는 값을 포함

```javascript

this.setState({ key : value})

```

위에서부터 보게 되면 `setState()`의 인자로는 `Object`를 보내게 된다.

인자로 `String`, `int`를 보내게 되면 에러가 발생한다.
`setState()` **인자로 `key`와 `value`가 넘어가고 `key`로 비교를 해서 있다면 덮어씌우고 없다면 새로 생성하는 역할**을 하기 때문이다.

이러한 형태는 `Map`형태와 유사하다.

- `setState()`는 그 객체를 상태에 `Merge`하여 상태를 업데이트하거나 새로 설정
- `setState()`는 비동기적으로 업데이트가 이루어질 수 있다.
  
<br/>

### 1. 객체 대신 함수를 넘길 수 있다.

---

```javascript

this.setState(
    function (state, props) {
        return { score: state.score - 1 };
    }
);

```

- 함수를 넘길 수 있다는 것은 함수형 프로그래밍의 특성 중 하나이다.

<br/>

### 2. 함수를 인자로 넘기는 이유

---

- state 업데이트가 비동기로 실행이 된다.
- 현재 상태의 값을 안정적으로 가져올 수 있다.

```javascript
submit(){
    this.setState(
        function(prevState, props){
            return {
                showForm: !prevState.showForm
            }
        }
    );
}
```

<br/>

### 3. setState()가 호출이 되면

---

- 전달한 객체를 **현재 상태로 합치고 Merge**
- 새로운 리액트 엘리먼트 트리를 만들고(Render), **이전 트리와 비교하고 변경된 부분을 DOM에 적용**

<br/>

## React의 상태 업데이트

리액트는 여러 `setState()` 호출을 성능 향상을 위해 단일 배치 방식으로 작업을 할 수도 있다.

여러 setState() 호출은 setState()를 한 번 이상 단일 함수 내에서 호출하는 것을 의미 할 수 있다.

```javascript
//constructor
state = {score : 0}; // multiple setState() calls

increaseScoreBy3 () {
    this.setState({score : this.state.score + 1});
    this.setState({score : this.state.score + 1});
    this.setState({score : this.state.score + 1});
}
```
setState()에 전달하는 것은 **일반 객체**이다.

여러 setState() 호출을 만나면 전달 된 모든 객체를 추출하여 **배치 작업**을 수행하고 이를 `Merge`하여 단일 객체를 만들고 그 단일 객체를 사용하여 `setState()`를 진행

> * 해당되는 객체들을 전부 가져와서 복사를 한다.
> * 같은 프로퍼티 즉 **같은 key를 가지고 있다면 맨마지막 key의 값을 넣는다.**

<br/>

```javascript
const singleObject =
    Object.assign(
        {},
        objectFromSetState1,
        objectFromSetState2,
        objectFromSetState3 // 3개가 같다면 3번이 우선
    );
```

위와 같은 패턴을 `Object Composition`이라고 한다.

`Javascript`에서 만약 3개의 객체가 동일한 키를 가지고 있다면 `Object.assign()`에 마지막으로 전달 된 객체의 `key`값이 우선이 된다.

```javascript

const me = {name : "Justice"},
      you = {name : "Your name"},
      we = Object.assign({}, me, you);

we.name === "Your name"; // true
console.log(we); // {name : "Your name"}

```

즉, 위의 increaseScoreBy3 함수를 사용하면 리액트가 **setState() 호출 순서로 상태를 즉시 업데이트하지 않기 때문에 함수의 최종 결과는 3 대신 1이 된다.**

리액트는 이처럼 먼저 모든 객체를 하나로 결합한 후에 계산한다.

`{score : this.state.score + 1}`

새로 만든 객체로 setState를 한 번만 수행했다.

`User.setState ({score : this.state.score +1}`

중요한 것은 +3이 아니라 +1이 된다는 점 이유는 위에 나와있지만 간단한게 여러 `setState`가 이루어지면 모든 **객체를 단일 객체로 만들고 그 단일 객체를 setState()한다.**

<br/>

## 함수를 인자로 넘기는 setState()

```javascript
// outside your component class
function increaseScore (prestate, props) {
  return {score : state.score + 1}
}
class User{
  ...
// inside your component class
  handleIncreaseScore () {
    this.setState(increaseScore)
  }
  ...
}
```

위처럼 컴포넌트 클래스 외부에서 상태 업데이트 로직을 관리하면 **컴포넌트 클래스는 더이상 상태 업데이트 방법을 고려하지 않는다.** 단순히 원하는 업데이트 유형을 선언한다.

많은 상태(state)가 있는 복잡한 컴포넌트에 대해 생각해보자. 

**각 상태를 다른 액션으로 업데이트 할 것**이고 때로는, **각각의 업데이트 함수에는 많은 코드가 포함**되어 있을 수 있다. 
이 모든 로직이 컴포넌트 클래스 내에 있는 것이다.

그러나 더 이상은 그럴 필요가 없게 된다.

```javascript
import {increaseScore} from "../stateChanges";

class User{
  ...
  // inside your component class
  handleIncreaseScore () {
    this.setState( increaseScore)
  }
  ...
}
```

다음 상태를 계산하기 위해 추가 인자를 전달할 수도 있다.

```javascript
// outside your component class
function increaseScore (plus) {
  return function(prestate, props){
        return {score : state.score + plus};
      }
    }
}
class User{
  ...
// inside your component class
  handleIncreaseScore () {
    this.setState(increaseScore(1))
  }
  ...
}
```

<br/>

#### Reference

- https://www.vobour.com/%ED%95%A8%EC%88%98%ED%98%95-setstate%EA%B0%80-%EB%A6%AC%EC%95%A1%ED%8A%B8-react-%EC%9D%98-%EB%AF%B8%EB%9E%98%EC%9D%B4%EB%8B%A4-functiona
- https://www.vobour.com/-setstate-%EB%A9%94%EC%8F%98%EB%93%9C-%ED%8C%8C%EB%9D%BC%EB%AF%B8%ED%84%B0%EB%A1%9C-%EA%B0%9D%EC%B2%B4-%EB%8C%80%EC%8B%A0-%ED%95%A8%EC%88%98-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-using
- https://meetup.toast.com/posts/110