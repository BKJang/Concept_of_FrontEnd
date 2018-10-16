# Async-Await

## Async-Await에 도달하기 위해서

1. Iterator
2. Generator
3. Promise
4. Async-Await

</br>

## for-of

우리가 흔히 사용하고 많이 사용하는 `for`문이 있다. 
그러나 어디서부터 어디까지 지정을 하는것이 아닌 전체를 순회한다고 하면 `for-in`이나 `foreach`를 사용한다.

`for-in`은 **각각의 `key`를 받아와서 그 키를 가지고 `value`에 접근**하며 `foreach`는 **중간에 나올 수 없다는 단점**을 가지고 있다.

**이러하여 생긴것이 `for-of` 이다.**

**ECMA2015**에 새로운 문법으로 추가가 되었으며 이것은 배열은 물론 `object`까지 순회가 가능하다.(물론 key가 있어야 가능)

**for-of문은 열거가 가능한 객체라면 무엇이든 순회가 가능하다.**

열거가 가능한 객체란 **배열(Array)는 물론, TypedArray, 문자열(String), Map, Set, DOM Collection 등**을 말한다.

```javascript
/* array */
let iterable = [10, 20, 30];

for (let value of iterable) {
    console.log(value); // 10 20 30
}

/* string */
let iterable = "boo";

for (let value of iterable) {
    console.log(value); // "b" "o" "o"
}

/* Typed Array */
let iterable = new Uint8Array([0x00, 0xff]);

for (let value of iterable) {
    console.log(value); // 0 255
}

/* Map */
let iterable = new Map([["a", 1], ["b", 2], ["c", 3]]);

for (let entry of iterable) {
    console.log(entry); // [a, 1] [b, 2] [c, 3]
}

for (let [key, value] of iterable) {
    console.log(value); // 1 2 3
}

/* Set */
let iterable = new Set([1, 1, 2, 2, 3, 3]);

for (let value of iterable) {
    console.log(value); // 1 2 3
}

/* DOM Collection */
let articleParagraphs = document.querySelectorAll("article > p");

for (let paragraph of articleParagraphs) {
    paragraph.classList.add("read");
}
```

이렇게 순회를 할 수 있는 이유는 `iterator` 덕분이다.

`Collection`들은 내부적으로 `Iterable Protocol`을 구현한다.

</br>

## Iterable Protocol

어떤 객체가 순회가 되기 위해서는 `iteration`동작에 대해 정의가 되어 있어야 한다.

`Iterable Protocol`에 의하면 **`Iterator`는 `next()`함수를 구현해야하고 이 함수는 결과값으로 `{value: value, doen: boolean}` 같은 객체를 반환**해야 한다고 한다.


```javascript
function makeRangeIterator(start, end) {
    var nextIndex = start;
    var n = start;

    var rangeIterator = {
        next: function() {
            var result;
            if (nextIndex < end) {
                result = { value: nextIndex, done: false };
            } else if (nextIndex == end) {
                result = { value: n, done: true };
            } else {
                result = { done: true };
            }

            nextIndex++;
            n++;
            
            return result;
        }
    }

    return rangeIterator;
}

// run iterator
const it = makeRangeIterator(1, 4);
let result = it.next();// 처음 한번 실행
while (!result.done) {
    console.log(result.value); // 1 2 3
    result = it.next();
}
```

`Iterator protocol`을 이용하여 열거가능한 객체를 만들 수 있다.

`[Symbol.iterator]()` 와 `next()`를 이용하면 쉽게 정의할 수 있다.

`for-of`는 `[Symbol.iterator]()`를 호출하여 반복을 실행하게 된다.

```javascript
const iterable = {
    [Symbol.iterator]() {
        return {
            i: 0,
            next() {
                if (this.i < 3) {
                    return { value: this.i++, done: false };
                }

                return { value: undefined, done: true };
            }
        };
    }
};

for (var value of iterable) {
    console.log(value); // 0 1 2
}
```
</br>

## Generator

`Iterator`와 비슷한 이녀석은 일종의 **코루틴(Co-Routine)**인데, 다른 언어에서는 곧 잘 사용하는 개념이다.

> 코루틴(Co-Routine) : 어느 작업을 하다가 중간에 멈추고 멈춘 부분부터 다시 시작이 가능한

`Generator`는 함수 **실행도중에 잠시 멈췄다가 다시 실행할 수 있는** 독특한 함수이다. `Generator`는 `function*`키워드를 사용해서 생성하며,

**`Generator`를 호출하면 실행되는 것이 아니라 `Iterator`객체가 반환된다.**

따라서 **Iterator에서 구현한 `next()` 함수를 호출하면 Generator가 실행되면서 `yield`를 만날 때까지 실행**되고, 이때 컨텍스트는 저장된 상태로 남아 있게 된다.

```javascript
function* idMaker(){
    var index = 0;
    while(index < 3)
        yield index++;
}

const gen = idMaker(); // iterator객체가 반환

console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
```

`Generator`를 사용하면 `Iterable Protocol`을 구현하는 것보다 좀 더 쉽게 `Iterator`를 사용할 수 있다.

그렇다면 `Generator`는 어떻게 구현이 되어있을까?

```javascript
// ES6
function* foo(){
    yield bar();
}

// ES5 Compiled
"use strict";

var _marked = /*#__PURE__*/ regeneratorRuntime.mark(foo);

function foo() {
    return regeneratorRuntime.wrap(
        function foo$(_context) {
            while (1) {
                switch ((_context.prev = _context.next)) {
                    case 0:
                        _context.next = 2;
                        return bar();
                    case 2:
                    case "end":
                        return _context.stop();
                }
            }
        },
        _marked, this
    );
}
```

`Genrator`는 결국 `iterable Protocel`를 구현하는 객체이다. 그러나 프로토콜과 관련된 어느것도 보이지 않는다. 

대신 `regeneratorRuntime`이것이 보인다.

**`babel`에서는 `regeneratorRuntime`라이브러리를 사용해서 구현**을 했다.

이 라이브러리는 2013년 `Node.js v0.11.2`에서 `generator syntax`를 지원하기 위해 만들어 졌으며, `Babel`에서는 이 라이브러리를 사용하여 `generator`를 구현하고 있다.
실제 코드를 들여다보면 `Symbol`과 `Iterator`를 이용해서 `Iterable Protocol`을 구현하고 있다.

</br>

## Async-await

`Async-await`는 `ECMA-262`에서 초안으로 처음 등장했으며, `ECMAScript 2017`에서 표준으로 정의되었다.(`ES8`에서 표준이 되었다.)

먼저 `Babel`에서 `Async-await`를 돌리게 되면 `ES5`버전으로 나온다. 그렇다면 `ES5`로 구현이 가능하다는 것이다. 실제로 `Babel`로 돌리게 되면 아래와 같은 결과가 나온다.

```javascript
// ES7
async function foo() {
    await bar();
}

// ES5 complied
let foo = (() => {
   var _ref = _asyncToGenerator(function*() {
       yield bar();
   });

    return function foo() {
       return _ref.apply(this, arguments);
    };
})();

function _asyncToGenerator(fn) {
    return function() {
        var gen = fn.apply(this, arguments);

        return new Promise(function(resolve, reject) {
            function step(key, arg) {
                try {
                    var info = gen[key](arg);
                    var value = info.value;
                } catch (error) {
                    reject(error);
                    return;
                }

                if (info.done) {
                    resolve(value);
                } else {
                    return Promise.resolve(value).then(
                        function(value) {
                            step("next", value);
                        },
                        function(err) {
                            step("throw", err);
                        }
                    );
                }
            }
            return step("next");
        });
    };
}
```

위에 컴파일됨 구문을 보게 되면 **`async`를 `generator`로 바꾸고 `await`는 `yield`로 바꾸었다.**

`Generator`는 `yield`를 만날 때까지 실행된다. 이때 `Context`는 저장된 상태로 남아있게 된다.

즉, **`Generator`로 비동기 로직이 끝날때마다 적절히 `next()`를 적절하게 호출해주면 `Async-Await`함수가 만들어 지는 것**이다.

그러나 `bar()`함수의 작업이 종료되는 시점은 `bar()`함수밖에 모른다. 그렇다고 `next()`를 `bar()`함수 내에서 직접 실행하게 된다면 의존성이 생기게 된다. 

그렇다면 어떻게 의존성을 분리 할 수 있을까?

</br>

### `Promise`

---

`Babel`은 `promise`와 `재귀함수`를 이용하여 `next()`를 대신 호출해주는 함수를 만드는데, 그게 바로 **`_asyncToGenerator`**이다.

`fn.apply`를 실행하여 인자로 넘어온 `Generator`를 실행하여 `iterator`객체를 **클로저로 저장해둔다.** 
나머지는 클로저에 저장한 `iterator`를 실행시키고, 반환된 `promise`객체를 재귀함수를 통해 반복실행한다.

정리하자면 `Generator`는 비동기적 패턴을 `yield`를 통해 동기적인 모습으로 바꾸어주고, `Promise`는 `Generator`로 만든 `iterator`를 반복해서 실행해주는 역할을 한다. `await keyword`에 사용하는 함수가 항상 `promise`를 반환해야하는 이유가 여기에 있다.
</br>

> Promise => **프로미스는 자바스크립트 비동기 처리에 사용되는 객체다.** 여기서 자바스크립트의 비동기 처리란 특정 코드의 실행이 완료될 때까지 기다리지 않고 다음 코드를 먼저 수행하는 자바스크립트의 특성을 의미한다

`Promise`로 구현

```js
const makeRequest = () => {
    return getData()
        .then(data => {
            if (data.needMoreRequest) {
                return makeMoreRequest(data)
                    .then(moreData => {
                        console.log(moreData);
                        return moreData;
                    }).catch((error) => {
                      console.error('Error while makeMoreRequest', error);
                    });
            } else {
                console.log(data);
                return data;
            }
        }).catch((error) => {
          console.error('Error while getData', error);
        });
}
```

* `async/await`로 구현

```js
const makeRequest = async () => { 
    try {
      const data = await getData();

      if (data.needMoreRequest) {
          try {
            const moreData = await makeMoreRequest(data);
            console.log(moreData);
            return moreData;
          } catch (error) {
            console.error('Error while makeMoreRequest', error);
          }
      } else {
          console.log(data);
          return data;
      }
    } catch (error) {
      console.error('Error while getData', error);
    }
}
```

</br>

## 참고

- [Generator](https://meetup.toast.com/posts/73)
- [iterator](https://medium.com/@la.place/javascript-iterator-b16ca3c51af2)
- [Aync-Await](
https://medium.com/@la.place/async-await%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EA%B5%AC%ED%98%84%ED%95%98%EB%8A%94%EA%B0%80-fa08a3157647)
