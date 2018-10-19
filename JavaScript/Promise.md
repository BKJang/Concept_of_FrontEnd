# Promise

## 제어의 역전

콜백 지옥의 문제점은 제어의 역전이다. **믿음직하지 못한 어떤 비동기 프로그램에게 우리 프로그램의 호출 제어권을 넘겨주어 어떤 일이 발생할 수 있다는 것**이다.

예를 다시 들어보자.

```javascript
solutionUtil.trackPurchase(purchaseData, function CBF() {
  // 결제데이터를 넘긴 뒤
  // 비동기적인 처리가 완료되면 콜백실행
  assignCredit(); // 결제
  goThanksPage(); // 페이지 이동
});
```

저 유틸의 trackPurchase 라는 함수는

- purchaseData 와 콜백함수 두가지 인자를 받는다.
- 우리는 **알 수 없는 어떤 로직 진행 후 완료가되면 콜백함수를 실행**한다.

이 과정의 문제점은 콜백의 실행 제어권이 저 함수에게 넘어간다는 점이다.

<br/>

## 제어의 재역전

자 이제 저 제어권을 돌려받자. **콜백을 저 함수가 아니라 우리가 실행할 수 있게 만들자**는 개념이 promise 다.
먼저 예시를 들어보자.

1. 어떤 작업을 하려고 foo()라는 함수를 부른다.
2. foo()의 내용은 관심이 없지만 foo()가 끝나길 기다린다.
3. foo()가 끝나면 어떤 작업을 실행한다.

콜백 베이스의 코드라면 **foo()를 실행할 때 콜백을 넘겨준다.**

```javascript
foo(someData, bar(), barErr());
```

이렇게 하면 foo 가 자신이 끝나면 bar()를 **백그라운드**에서 실행한다. 하지만 자바스크립트에는 이벤트 방식도 존재한다.

```javascript
function foo(x) {
  // 어떤 일을 실행하고 이벤트를 반환
  return listner;
}

var evt = foo(42);

evt.on('completion', bar());
evt.on('failure', barErr());
```

이게 사실 프로미스가 실행되는 방식이다.
위의 코드를 잘 생각해보자. foo 가 비동기적으로 실행되고 그 결과를 받아 evt 에 저장한다.
**foo(42)가 완료되어 evt 에 할당되면 그걸 우리가 읽고 다음 단계를 실행**한다.

즉 제어권을 다시 역전시킨 것이다.

조금 더 간단하게는 이렇게 바뀔 수 있다.

```javascript
var evt = foo(42);

bar(evt);
barErr(evt);
```

<br/>

## 리스너를 프로미스로

이벤트 리스너가 아니라 프로미스로 조금만 바꿔보자.

```javascript
function foo(x) {
  // 어떤 일을 실행하고 프로미스를 반환

  return new Promise(function(resolve, reject) {});
}

var p = foo(42);

bar(p);
```

여기서 bar 의 내부를 보자면

```javascript
function bar(fooPromise) {
  fooPromise.then(
    function() {}, // resolve 시 작업
    function() {} // reject 시 작업
  );
}
```

이렇게 되어있을 것이다. 현대의 코드는 `.then()`과 `.catch()`를 많이 사용하지만 기본은 `resolve` 와 `reject` 다.

이처럼 프로미스는 단 한번만 결과값이 귀결된다. 이 값은 불변성을 가져 변하지 않는다.(믿음성)

**단 한번만 결과값이 도출**되기에 여러번 호출하지 않고, 값이 변하지 않기 때문에 그 결과로 다른 로직에 이용되도 같은 결과가 도출된다. 
또한 **에러가 발생하더라도 프로그램의 흐름은 멈추지 않는다.** 
그냥 **예외가 되었기에 버림콜백(reject)로 진행**한다.


<br/>

## Promis.resolve()

프로미스는 사실 별게 아니다. 콜백을 넘겨주는 위치를 달리했을 뿐이다. 
**`foo()`에 콜백을 넘기는 것이 아닌 `foo()`에서 프로미스를 받은 뒤 이 프로미스에 콜백을 준 것**이다.

프로미스가 나오게 된 가장 중요한 점은 믿음성이다.
근데 중요한 것은 이 받은 프로미스가 진짜 프로미스가 아니라면?
단지, `.then()`을 쓸 수 있는 object 라면 어떨까?

이를 흔히 **`Thenable`** 이라고 한다.

```javascript
var p = {
  then: function(foo, err) {
    foo(42);
    err('하이');
  }
};

p.then(
  function resolve(val) {
    console.log(val);
  },
  function rejected(err) {
    console.log(err);
  }
);
```

이 경우 p 처럼 then()이 있는 thenable 은 모두 프로미스처럼 보이게 동작할 수 있다.

이 경우 `Promise.resolve()`로 해결이 가능하다. Promise.resolve()를 거치면 이 값으로 이루어진 프라미스 객체가 반환된다.

이게 굉장히 중요하다. 프로미스가 등장한 이유가 믿음성인데 이 믿음성을 해결해준다.

```javascript
var p1 = new Promise(function(resolve, reject) {
  resolve(42);
});
var p2 = Promise.resolve(42);

console.log(p1 === p2); // true
```

자 이제 Thenable 한 값을 Promise.resolve()로 정규화시켜보자.

```javascript
var p = {
  then: function(foo, err) {
    foo(42);
    err('하이');
  }
};

Promise.resolve(p).then(
  function resolve(val) {
    console.log(val);
  },
  function rejected(err) {
    console.log(err); // 실행 안됌
  }
);
```

자 그럼 이제 할 일은 아래와 같다. 만일 foo()가 어떤 비동기 행위이며 반환값이 프로미스인지 확신하게끔 만들자.

```javascript
foo(42).then(function(v) { // bad
  console.log(v); 
});

Promise.resolve(foo(42)).then(function(v) { // good
  console.log(v); 
});
```

이러면 어떠한 경우에도 무리가 없다. 믿음이 확실해진다.

<br/>

## 익숙한 ajax

그럼 자주 사용하는 ajax 에 응용해보자.

가장 중요하게 기억해야 할 것은 기존의 콜백은 백그라운드에서 호출, 프로미스는 호출스택에서 호출한다는 점이다.

```javascript
function request(url) {
  return new Promise(function(resolve, reject) {
    // ajax의 콜백함수가 일반 함수가 아니라 방금 생성한 promise의 resolve가 된다.
    ajax(url, resolve);
  });
}

request('https://...')
  .then(function(response1) {
    return request('https://.../?q=' + response2);
  })
  .then(function(response2) {
    console.log(response2);
  });
```
