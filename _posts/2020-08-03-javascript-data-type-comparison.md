---
title: "Javascript data types and comparison"  
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
  - react
  - shallow compare
---

2020년 8월 2일에 본 토스 front-end 코딩 테스트에 toEqual() 과 toBe()를 구현하는 문제가 나왔다.  

둘의 차이점은 무엇인지 살펴보며 javascript의 data type과 몇가지 비교 연산자들, 그리고 react의 shallow comparison을 파헤쳐보자.

## Data type
자바스크립트에는 primitive, object 두가지의 데이터 타입이 있다.  

#### Primitive
primitive data type은 object가 아니며 메소드가 없는 타입으로 string, number, bigint, boolean, undefined, symbol, null 7가지 종류가 있다.  

```javascript
console.log(typeof 3); // => number
console.log(typeof '3'); // => string
console.log(typeof 3n); // => bigint
console.log(typeof true); // => boolean
console.log(typeof undefined); // => undefined
console.log(typeof Symbol('s')); // => symbol

```
typeof 연산자로 타입 확인이 가능하다.

```javascript
console.log(typeof null) // => object
```
null은 primitve data type이지만 typeof 연산자는 object를 반환하는 특수한 경우이다.

#### Object
```javascript
const obj = {}
const arr = []
const map = new Map()
console.log(typeof obj) // => object
console.log(typeof arr) // => object
console.log(typeof map) // => object
```
new 키워드와 constructor를 통해 만들 수 있는 인스턴스는 object 타입이다.

Object, Array, Set, Map, WeekMap, WeekSet 등이 있다.

배열을 생성하는 `const arr = []` 구문도 내부적으로는 `const arr = new Array()`와 같다.

#### Function
```javascript
const fn = function () {}

console.log(typeof fn) // => function
```
javascript에서는 함수 또한 객체(Object)이지만 typeof 연산자는 call이 가능한 객체에 `'function'`을 반환한다.

#### Primitive wrapper object
null, undefined 이외의 primitive type을 래핑한 primitive wrapper obejct가 있다. 

```javascript
console.log(typeof Number(3)) // => number
console.log(typeof new Number(3)) // => object
```
생성자로 호출한다면 Number 객체를 반환하고 아니라면 primitive value를 반환한다.

```javascript
const str = 'str'
const Str = new String('str');

console.log(str == Str) // => true
console.log(str === Str) // => false
console.log(str === Str.valueOf()) // => true
```
wrapper의 valueOf() 메소드는 primitive value를 반환한다.

```javascript
console.log(str instanceof String) // => false
console.log(Str instanceof String) // => true

console.log(str.constructor === String); // => true
console.log(Str.constructor === String); // => true
```
`str`은 String의 instance가 아닌데 constructor에 접근 가능하다.

이는 `str`(primitive type)에서 메서드에 접근할때 내부적으로 wrapper object로의 형변환이 일어난 후에 접근하기 때문이다.

## toBe()
exactly same object 인지 판단한다. native 객체는 값과 타입을 비교하고 object는 reference(메모리상 주소)를 비교한다.
`===` 연산자와 같다.

## toEqual()
object의 값(property)들이 같은지를 (deep equality) 판단한다.

## Object.is() vs `===`
> This is also not the same as being equal according to the === operator. 
> The === operator treats the number values -0 and +0 as equal and treats Number.NaN as not equal to NaN.
> [(MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)

## React Shallow comparison
react의 최적화 방안 중 한가지가 class 컴포넌트의 PureComponent 또는 function 컴포넌트의 React.memo()를 사용하는 것이다.

react에서는 상위 컴포넌트가 re-rendering 되면 하위 컴포넌트들도 다시 호출되어 re-rendering을 수행하는데, 이 컴포넌트들을 사용한다면 이 자식 컴포넌트들의 불필요햔 rerendering을 줄일 수 있다.

이때 react는 이전 props를 메모해놓고 이후 props와 비교할때 각 props의 reference 만을 비교하는 _shallow compare_ 을 수행하여 re-rendering 여부를 결정한다.

props 및 state를 immutable하게 관리해야하는 이유이다. 


## Reference
- [MDN > Primitive](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)
- [MDN > Jvascript Data Type and Data Structure](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures)
- [MDN > Object.is()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)
- [Jest Docs> toEqaul](https://jestjs.io/docs/en/expect#toequalvalue)
- [Jest Github> toEqual](https://github.com/jasmine/jasmine/blob/8cd4873e4859e0f0177c1cbd9910e923bac60092/src/core/matchers/matchersUtil.js#L149)
- [React > PureComponent](https://reactjs.org/docs/react-api.html#reactpurecomponent)
- [Stack Overflow > Jasmine JavaScript Testing - toBe vs toEqual](https://stackoverflow.com/questions/22413009/jasmine-javascript-testing-tobe-vs-toequal)
- \[속깊은 Javascript\] 양성익, 루비페이퍼, 2016
