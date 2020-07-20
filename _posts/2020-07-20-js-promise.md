---
title: "Javascript Promise"  
category: js  
tags:
  - javascript
  - study
  - front-end
  - promise
  - callback hell
  - asynchronous
---

## Callback hell
```javascript
async(1, function({
    async(2, function({
        async(3, function({
            async(4, funciton({
                // do something
            )}
        )}
    )}
})
```
javascript에서는 비동기 작업시에 콜백함수를 같이 넘겨주는 형태로 코드를 자주 작성하는데,
이것이 여러개가 중첩되면 가독성이 떨어지고 디버깅을 어렵게하는 `콜백 지옥`에 빠지게 된다.  

```javascript
try {
  setTimeout(() => { 
    throw new Error('Error!'); 
  }, 1000);
} catch (e) {
  console.log(e);
}
```
에러처리에서도 문제가 있다. 
setTimeout의 콜백 함수가 Web API, Task Queue, Event Loop를 거쳐 
다시 Call Stack으로 돌아 왔을때는 try, catch 구문이 더이상 남아있지 않아 의도한대로 에러를 캐치할 수 없다. 

이러한 문제들을 해결하기 위해서 ES6에서 Promise 패턴이 제안되었다.

## Promise
#### Syntax
```javascript
function doSomething() {
    return new Promise((resolve, reject) =>  {
        setTimeout(() => resolve('success1'), 1000);
    })
}

doSomething()
.then(val => {
    console.log(val);
    return 'success2';
}).then(val => {
    console.log(val);
    return 'success3';
}).then(val => {
    console.log(val);
    throw new Error('error1');
}).catch(e => {
    console.log(e);
})

> success1
> success2
> success3
> Error: error1
```
Promise 생성자 함수를 이용하여 새로운 Promise 객체를 만들 수 있으며, 비동기 작업을 수행할 콜백함수의 parameter로 resolve, reject 함수를 전달받는다.
성공시 resolve, 실패시 reject 를 통해 값을 전달 할 수 있다.

`doSomething()` 함수를 호출하면 Promise 객체가 생성되어 비동기 작업을 수행하고, 완료된 뒤 `resolve()`를 통해 넘겨준 결과값을 `then()`을 통해 접근할 수 있다.  
`then()` 또한 새로운 Promise 객체를 반환하며, return 값을 다음 `then()`에서 파라미터로 받을 수 있다. 이를 Promise chaining이라고 한다.

reject를 통해 전달된 값이나, promise chaining 도중 에러가 발생했을 때는 `catch()`를 통해 처리할 수 있다. `then()`함수의 두번째 파라미터로 처리할 수도 있지만 `catch()`를 사용하는 것이 훨씬 직관적이다.

단순한 값을 반환하는 대신 Promise를 반환하는 함수들을 전달할 수 도 있다.
```javascript
getUserInfo()
.then(parseValue)
.then(auth)
.then(getFriends)
.catch(e => console.log(e));

function parseValue() {
    return new Promise((param) => {
        // ...
    })
}
```
`getUserInfo`, `parseValue`, `auth`,  `display` 함수 모두 프로미스 객체를 리턴하는 함수인 것이다.

#### Status
Promise 객체는 생성되고 종료될 떄까지의 라이프 사이클동안 3가지의 상태 값을 가진다. 
```javascript
const p = new Promise(resolve => {
        setTimeout(() => resolve(3), 1000)
        })

p (Promise)
├── __proto__: Promise
├── [[PromiseStatus]]: "pending" -> "resolved"
└── [[PromiseValue]]: undefined -> 3
```
- `pending`: Promise 객체가 생성되면 최초 Pending(대기) 상태가 된다.
- `fulfilled`: Promise 객체가 resolve 된 된 상태로, `then()`을 통해 결과값을 받아올 수 있다.
- `rejected`: Promise 객체가 reject 되거나 에러가 발생한 상태로, `catch()`를 통해 결과값이나 에러값을 받아올 수 있다.

`settled`: Promise 객체가 resolve 또는 reject 된 상태를 뜻하는 용어.

#### finally
```javascript
function doSomething() {
    return new Promise((resolve) => {
        setTimeout(() => resolve('then'), 1000);
    })
}

doSomething()
.then(val => {
    console.log(val);
    return 'finally';
})
.finally(() => {
    return 'then aggain';   
})
.then(val => {
    console.log(val);
})

> then 
> finally
```
```javascript
Promise
.reject(3)
.finally(()=>{})
.catch(val => console.log(val)) // => 3
```
ES9(ES2018)에서 추가되었다.  
try-catch 문의 finally 처럼 Promise 객체가 `settled` 되었을 때 사용할 수 있다.  
하지만 콜백함수의 parameter로는 아무 값도 받아올 수 없으며 이전 Promise 객체의 리턴 값은 다음 `.then()` 또는 `catch()`에 전달된다.

## Static Methods
Javascript에서는 생성자 함수도 객체이므로 메소드를 가지고 있다.

#### Promise.resolve(), Promise.reject()
```javascript
const resolvedPromise = Promise.resove(3);
resolvedPromise.then(val => console.log(val) // => 3

const rejectedPromise = Promise.reject(4);
rejectedPromise.catch(val => console.log(val) // => 4
```
원하는 값을 Promise 객체로 만들고 싶을 때 사용할 수 있다.

#### Promise.all()
```javascript
Promise.all([
    new Promise(resolve => setTimeout(() => resolve(3), 3000)),
    new Promise(resolve => setTimeout(() => resolve(2), 2000)),
    new Promise(resolve => setTimeout(() => resolve(1), 1000)),
])
.then(val => {
    console.log(val) // => [ 3, 2, 1 ]
})
```
Promise 객체 여러개를 담은 배열을 전달받아 병렬로 실행시킨다.  
모든 Promise가 성공된 후 처음 전달한 순서대로 결과 값을 받아올 수 있는 새 Promise 객체를 반환한다.  
전달된 Promise 중 하나라도 실패하면 가장 먼저 실패한 Promise가 reject한 값을 즉시 반환한다.
 
#### Promise.allSettled()
```javascript
Promise.allSettled([
    new Promise(resolve => setTimeout(() => resolve(3), 3000)),
    new Promise(resolve => setTimeout(() => resolve(2), 2000)),
    new Promise((resolve,reject) => setTimeout(() => reject(1), 1000)),
])
.then(results => {
    results.forEach(result => console.log(result))
})

> { status: 'fulfilled', value: 3 }
> { status: 'fulfilled', value: 2 }
> { status: 'rejected', value: 1 }
```
ES11(ES2020)에서 추가 되었다.  
`Promise.all()`은 reject되는 Promise 객체가 하나라도 있을 경우 해당 결과값만 즉각 리턴했지만 이 함수는 모두에 대한 값을 배열로 반환 받을 수 있다.  

#### Promise.race()
```javascript
Promise.race([
    new Promise(resolve => setTimeout(() => resolve(3), 3000)),
    new Promise(resolve => setTimeout(() => resolve(2), 2000)),
    new Promise((resolve, reject) => setTimeout(() => reject(1), 1000)),
])
.then(val => {
    console.log(val) // not called
})
.then(val => {
    console.log(val) // => 1
})
```
가장 먼저 **settled** 된 Promise 객체가 처리한 결과를 반환(resolve or reject) 하는 새로운 Promise 객체를 리턴한다.

#### Promise.any()
```javascript
Promise.any([
    new Promise(resolve => setTimeout(() => resolve(3), 3000)),
    new Promise(resolve => setTimeout(() => resolve(2), 2000)),
    new Promise(resolve => setTimeout(() => resolve(1), 1000)),
])
.then(val => {
    console.log(val) // => 1
})
```
현재 stg-3 단계이지만 크롬, 파이어폭스 등에 구현되어 있다.  
가장 먼저 **resolve** 된 Promise 객체가 처리한 결과를 resolve 하는 새로운 Promise 객체를 리턴한다.  
`Promise.all` 의 반대이다. 

- [Captain Pangyo > 자바스크립트 Promise 쉽게 이해하기](https://joshua1988.github.io/web-development/javascript/promise-for-beginners/)
- [PoiemaWeb > Promise](https://poiemaweb.com/es6-promise)
- [Must Know About Frontend > es9~es10](https://github.com/baeharam/Must-Know-About-Frontend/blob/master/Notes/javascript/es9-es10.md)
- [Huiseoul Engineering > 자바스크립트는 어떻게 작동하는가: 이벤트 루프와 비동기 프로그래밍의 부상, async/await을 이용한 코딩 팁 다섯 가지](https://engineering.huiseoul.com/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%91%EB%8F%99%ED%95%98%EB%8A%94%EA%B0%80-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84%EC%99%80-%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%EC%9D%98-%EB%B6%80%EC%83%81-async-await%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%BD%94%EB%94%A9-%ED%8C%81-%EB%8B%A4%EC%84%AF-%EA%B0%80%EC%A7%80-df65ffb4e7e)
- [MDN > Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [MDN > Using Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)