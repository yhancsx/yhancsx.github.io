---
title: "Javascript data type and comparison"  
category: js  
tags:
  - javascript
  - study
  - front-end
  - data type
  - primitive
  - object
  - test
  - toBe
  - toEqual
  - shallow compare
published: false
---

2020년 8월 20일에 본 토스 front-end 코딩 테스트에 toEqual() 과 toBe()를 구현하는 문제가 나왔다.  

둘의 차이점은 무엇인지 살펴보며 javascript의 data type과 몇가지 비교 연산자들, 그리고 react의 shallow comparison을 파헤쳐보자.

## Primitive data type
자바스크립트에는 primitive, object 두가지의 데이터 타입이 있다.  

```javascript
console.log(3) // => number
```
primitive data type은 object가 아니고 메소드가 없는 타입이며 string, number, bigint, boolean, undefined, symbol 6가지 종류가 있다.  

```javascript
console.log(typeof null) // => object
```
null은 primitve data type으로 보지만 typeof 연산자는 object를 반환한다.

```javascript
const obj = {}
const arr = []
const fn = function () {}

console.log(typeof obj) // => object
console.log(typeof arr) // => object
console.log(typeof fn) // => function
```
그 외는 모두 Object를 기초로하는 객체(object) 타입이며 Object, Function, Array, Set, Map 등이 있다.

```javascript
console.log(typeof Number(3)) // => number
```

이외에도 null, undefined 이외의 primitive type을 래핑한 primitive wrapper obejct가 있다.  wrapper의 valueof() 는 primitive type을 리턴한다.



## toBe()
native 객체는

## toEqual()

## React and Shallow compare
전후 비교 확인

## Reference
- [MDN > Primitive](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)