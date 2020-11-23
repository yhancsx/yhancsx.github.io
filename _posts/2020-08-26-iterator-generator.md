---
title: "Javascript Iterator and Generator"
category: js
tags:
  - js
  - study
  - front-end
  - javascript
  - es6
  - for...of
  - iterator
  - generator
---

객체의 순환을 조금더 편리하게 하기 위해서 es6에 iterator 및 for...of 키워드가 추가되었다.

```javascript
const obj = {
    a: 1,
    b: 2,
    c: 3
}
for(const key in obj) {
    console.log(`${key} : ${obj[key]}`); // => a : 1, b : 2, c : 3
}
```
```javascript
const array = [1,2,3,4,5]

for(const value in array) {
    console.log(value); // => 0, 1, 2, 3, 4  // value가 아닌 index 값이 나옴.
}
```
```javascript
const str = 'abcdef'

for(let i = 0; i < str.length; i ++) {
    console.log(str[i]); // => a, b, c, d, e, f
}
for(const c in str) {
    console.log(c); // => 0, 1, 2, 3, 4, 5  // character가 아닌 index 값이 나옴.
}
```
기존엔 index를 이용한 단순 for loop나, for...in 을 사용하여 객체, array, 문자열 등을 순회할 수 있었다.

for...in 구문은 value가 아닌 key 값을 반환한다는 것에서 불편함이 있었다.

C++, Java의 iterator와 같이 조금 더 다양한 순회 기능을 제공하기 위해 iterator, for...of 키워드 및 generator가 ES6에서 정의 되었다.


## Iterator
```javascript
const fibonacci = {
  maxStep: 10,
  [Symbol.iterator]() {
    let pre = 0, cur = 1, step = 0;
    const maxStep = this.maxStep
    return {
      next() {
        [pre, cur, step] = [cur, pre + cur, step+1];
        return { done: step >= maxStep, value: cur }
      }
    }
  }
}

for (var n of fibonacci) {
  console.log(n); // => 1, 2, 3, 5, 8, 13, ....
}

const it = fibonacci[Symbol.iterator]() // => iterator

console.log(it.next().value) // => 1
console.log(it.next().value) // => 2
console.log(it.next().value) // => 3
```

for...of 키워드를 사용하면 뒤에 오는 객체의 `[Symbol.iterator]`를 호출한다.

호출 결과로 반환된 객체는 `iterator protocol` 만족시켜야 한다.
- next() 함수를 포함하고 있어야 하며, 이 next()를 반복적으로 호출하여 iterator를 반복시킬 수 있다.
- next() 함수는 value 속성과 done 속성을 가진 객체를 반환해야 한다. 
- value는 현재 iterator가 가리키는 곳의 값이고, done은 iterator가 끝까지 도달했는지를 구분해 주는 boolean 값이다.

어떤 객체가 `[Symbole.iterator]` key를 가지며, 이 key의 value 호출 결과로 `iterable protocol`을 만족시키는 객체를 반환한다면 이 객체를 `iterable object` (~ `iterator`) 라고 한다.
 
## Generator
iterable object를 만들기 위해선 규약에 맞추어 next() 함수와 반환 객체 `{value, done}`을 생성해야 하는 번거로움이 있다.

이를 간단하게 해주기 위해 generator 기능이 정의되었고, `function*` 와 `yield` 키워드를 사용한다.

`function*`: generator 함수. generator 객체 반환.  
`yield`: 함수를 잠시 멈추었다가 다음에 호출되었을 때 그 이후부터 시작한다는 표시이다. 

```javascript
const fibonacci = {
  maxStep: 10,
  *[Symbol.iterator]() {
    let pre = 0, cur = 1, step = 0;
    while(step < this.maxStep) {
      [pre, cur, step] = [cur, pre + cur, step+1];
      yield cur;
    }
  }
}

for (var n of fibonacci) {
  console.log(n); // => 1, 2, 3, 5, 8, 13, ....
}

const it = fibonacci[Symbol.iterator]() // => iterator

console.log(it.next().value) // => 1
console.log(it.next().value) // => 2
console.log(it.next().value) // => 3
```

next() 함수를 직접 작성해서 반환할 필요가 없어졌다.

기존 iterator가 done 으로 종료 조건을 판단했다면, generator는 전체 함수가 종료되면 반복이 종료된다.

yield 키워드를 만나면 generator 함수는 잠시 멈추었다가 다음 next() 호출시에 이어서 실행된다.

#### Generator not as object method
```javascript
const fibonacci = function* (maxStep) {
  let pre = 0, cur = 1, step = 0;
  while(step < maxStep) {
    [pre, cur, step] = [cur, pre + cur, step+1];
    yield cur;
  }
}

for (const o of fibonacci(10)) { 
  console.log(o);  // => 1, 2, 3, 5, 8, ...
}

const it = fibonacci(10)
console.log(it.next().value) // => 1  // it.next() : {value: 1, done: false}
console.log(it.next().value) // => 2
console.log(it.next().value) // => 3
```
앞의 예제들은 객체를 순환하기위해 `[Symbol.iterator]` property를 정의하였지만, generator 함수는 직접 generator 객체를 반환하여 사용할 수도 있다.

for...of 키워드의 뒤에 geneator 객체가 바로 와도 된다.

#### yield*
```javascript
function* anotherGenerator(i) {
  yield i + 1;
  yield i + 2;
  yield i + 3;
}

function* generator(i) {
  yield i;
  yield* anotherGenerator(i);
  yield* [i + 10, i + 11];
}

var gen = generator(10);

console.log(gen.next().value); // 10
console.log(gen.next().value); // 11
console.log(gen.next().value); // 12
console.log(gen.next().value); // 13
console.log(gen.next().value); // 20
console.log(gen.next().value); // 21
```
`yield*` 키워드를 통해 generator 내부에서 또다른 generator를 순회할 수 있다.

## Deconstructing
```javascript
const fibonacci = {
  maxStep: 10,
  *[Symbol.iterator]() {
    let pre = 0, cur = 1, step = 0;
    while(step < this.maxStep) {
      [pre, cur, step] = [cur, pre + cur, step+1];
      yield cur;
    }
  }
}
console.log([...fibonacci]) // => 1, 2, 3, 5, 8, ...
```
```javascript
const fibonacci = function* (maxStep) {
  let pre = 0, cur = 1, step = 0;
  while(step < maxStep) {
    [pre, cur, step] = [cur, pre + cur, step+1];
    yield cur;
  }
}
console.log([...fibonacci(10)]) // => 1, 2, 3, 5, 8, ...
```
deconstructing을 이용해서 전체 값을 가져올수 있다. generator 객체 뿐 아니라 iterable object도 deconstructing이 된다는 것이 흥미롭다.

## Array.from()
```javascript
const fibonacci = function* (maxStep) {
  let pre = 0, cur = 1, step = 0;
  while(step < maxStep) {
    [pre, cur, step] = [cur, pre + cur, step+1];
    yield cur;
  }
}
console.log(Array.from(fibonacci(10))) // => Array [1, 2, 3, 5, 8, ...]
```

## Iterable Objects
- Map
- Set
- Array
- String
- TypedArray

이 객체들의 프로토타입 객체들은 모두 `[Symbol.iterator] 메소드를 가지고 있다.`

## To study: Generator and Promise/async/await
https://suhwan.dev/2018/04/18/JS-async-programming-with-promise-and-generator/

> Generators in JavaScript -- especially when combined with Promises -- are a very powerful tool for asynchronous programming as they mitigate 
-- if not entirely eliminate -- the problems with callbacks, such as Callback Hell and Inversion of Control. 
However, an even simpler solution to these problems can be achieved with async functions. 
[Link](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)

## References
- [Must Know About Frontend > ES6](https://github.com/baeharam/Must-Know-About-Frontend/blob/master/Notes/javascript/es6.md#%EB%B0%98%EB%B3%B5%EC%9E%90iterator--forof)
- [MDN > Iteration Protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)
- [MDN > Iteators and Generators](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Iterators_and_Generators)
- [MDN > for...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of)
- \[속깊은 Javascript\] 양성익, 루비페이퍼, 2016
