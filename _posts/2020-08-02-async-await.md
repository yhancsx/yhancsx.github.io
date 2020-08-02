---
title: "Javascript async await"  
category: js  
tags:
  - javascript
  - study
  - front-end
  - asynchronous
  - callback hell
  - promise
  - async await
---

ES6에 추가 되었던 Promise를 조금 더 직관적으로 사용하기 위해 ES8에 추가된 async/await 에 대해 알아보자. 

```javascript
getUserInfo()
.then(parseValue)
.then(auth)
.then(getFriends)
.catch(e => console.log(e));
```
Promise 를 사용하면 callback 지옥을 빠져나와 훨씬 깔끔한 `then()` `catch()` 형태로 여러개의 비동기 작업을 처리할 수 있었다. 

## Syntax
```javascript
async function fetchUser(){
    const userInfo = await getUserInfo();
    const parsedUserInfo = await parseValue(userInfo);
    const validUser = await auth(parsedUserInfo);
    if(validUser) {
        const friends = await getFriends(parsedUserInfo);
    }else {
        // ...
    }
}
```
async await을 사용한다면 동기작업에서 우리가 사용하던 한줄한줄 명령을 실행하는 코드 형태로 비동기 작업들을 처리할 수 있어 훨씬 더 직관적이다.

await 다음에 오는 함수는 Promise 객체가 반환 되어야 의도 했던대로 동작핟다.

## Return
async 함수의 리턴 값 또한 Promise 객체이다.
```javascript
const asyncFunction = async function() {return 'async function'}
const ret = asyncFunction()

ret (Promise)
├── __proto__: Promise
├── [[PromiseStatus]]: "fullfilled"
└── [[PromiseValue]]: 'async function'
```

## Error handling
```javascript
(async function () {
    console.log('start')
    const rej = await Promise.reject('reject');
    console.log(rej);
    console.log('end')
})()

> start
> Uncaught (in Promise) reject
```
async 함수 중간에 reject을 만나면 그 즉시 함수를 빠져나간다. 

```javascript
(async function () {
    try {
        console.log('start')
        const rej = await Promise.reject('reject');
        console.log(rej);
        console.log('end')
    } catch(e) {
        console.log(e)
    }
})()

> start
> reject
```
try catch 구문을 이용해 에러 처리를 할 수 있다.

## Parallel
```javascript
async function series() {
    await wait(500);
    await wait(500);
    return "done!";
}
```
두개의 비동기 작업을 직렬로 실행하기 때문에 완료되는데 데 1초 이상의 시간이 소요 될것이다.

```javascript
async function parallel() {
    const wait1 = wait(500);
    const wait2 = wait(500);
    await wait1;
    await wait2;
    return "done!";
}
```
동시에 실행해도 되는 여러개의 비동기 작업은 위와 같이 병렬로 처리해 줄 수 있다. 

## Etc
- async/await 구문은 마치 동기 함수 인것처럼 보이지만 실제로 메인스레들를 차단하지 않고 비동기로 실행된다.

## Reference
- [bono's blog > javascript async, await를 사용하여 비동기 javascript를 동기식으로 만들자](https://blueshw.github.io/2018/02/27/async-await/)
- [Captain Pangyo > 자바스크립트 async와 await](https://joshua1988.github.io/web-development/javascript/js-async-await/)
- [Javascript 와 Async Await](https://medium.com/@pks2974/javascript-%EC%99%80-async-await-111fdad3c20d)
- [Google Web Fundamentals > Async functions - making promises friendly](https://developers.google.com/web/fundamentals/primers/async-functions?hl=ko)