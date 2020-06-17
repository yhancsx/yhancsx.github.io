---
title: "Javascript Scope, Hoisting and Closure"  
category: js  
tags:
  - javascript
  - study
  - front-end
  - scope
  - hoisting
  - closure
comments: true
---
Javascript 에서 까다롭게 느껴지는 Scope 와 Hoisting 그리고 이를 사용한 Closure 개념까지 탐구해 보자.
 
### Scope

javascript는 기본적으로 함수 레벨 또는 블록레벨 스코프 규칙을 따른다.  
`var` 변수 또는 함수 선언식으로 만들어진 함수는 함수레벨 스코프를 갖는다.  
`let`, `const` 변수는 블록레벨 스코프를 갖는다.

```javascript
function run() {
  var foo = "Foo";
  let bar = "Bar";

  console.log(foo, bar);  // => Foo, Bar

  {
    let baz = "Bazz";
    console.log(baz); // => Bazz
  }

  console.log(baz);  // => ReferenceError
}

run();
```
baz 변수는 자신이 포함된 블록의 scope(Lexical Environment)에 정의되어 있어 외부 스코프에서 접근할 수 없기 때문에 ReferenceError가 발생한다.

### Hoisting

호이스팅이란 '끌어올린다'라는 뜻으로 변수 및 함수 선언문이 스코프 내의 최상단으로 끌어올려지는 현상을 말한다.  
정확히는, 어떤 스코프가 형성됨(Lexical Environment가 형성)과 동시에 그 안에서 정의된 변수들이 함께 생성되기 때문이다.

##### var, Function Declaration
```javascript
console.log(a); // => undefined
var a = 2;

var b = new func();
function func() {
    this.a = 1;
}
console.log(b.a) // => 1
```
var 변수는 호이스팅됨과 동시에 undefined로 초기화된다(`var a`). 후에 해당 부분 코드가 실행될 때 설정한 값이 할당된다(`a = 2`).

##### Function Expression
```javascript
func();
var func = function() {} // => TypeError
```
함수 표현식은 해당 변수의 호이스팅 영향을 받는다.  
func가 undefined로 초기화 되어있으므로 func()를 실행하는 것은 TypeError를 뿜는다.

##### let, const
```javascript
let a = 1;
{
    console.log(a); // => Reference Error: a is not defined;
    let a = 2;
}
```
let, const 변수도 호이스팅이 되지만 undefined로 초기화 하지 않고 접근이 불가능하다는 차이점이 있다.  
이런 접근 불가 영역을 TDZ(Temporal Dead Zone) 이라고 부른다.  
호이스팅 때문에 출력값이 1이 아닌 에러를 반환 하는 것이다.  

let, const 는 스코프, 호이스팅 이외에도 재선언이 불가능 하다는 var와의 차이점이 있다. 

### Closure 
클로저 함수란, `만들어진 환경(스코프)을 기억하여 그 환경 밖에서 실행될 때도 그 환경에 접근할 수 있는 함수` 라고 설명 할 수 있다.
```javascript
function getClosure() {
  var text = "foo";
  return function() {
    return text;
  };
}

var closure = getClosure();
console.log(closure()); // => foo
```
closure 변수가 getClosure 내부 환경을 기억하고 있기 때문에 getClosure 함수 실행이 종료된 후에도 foo를 출력할 수 있다.  
원래는 getClosure함수가 수행된 후, 해당 함수의 환경을 GC가 회수해야 하는데 closure가 참조 하고 있기 때문에 그렇지 않은 것이다.

### Why Closure? 

그렇다면 이런 클로저 함수는 어디에 쓰이는 걸까?

##### 은닉화(private)
javascript는 java와 달리 객체안에 private기능을 제공하지 않는다.

```javascript
function Hello(name) {
  this._name = name;
}

Hello.prototype.say = function() {
  console.log('Hello, ' + this._name);
}

var hello1 = new Hello('yhan');
hello1.say() // => Hello, yhan
hello.__name = 'stupid'
hello1.say() // => Hello, stupid
```
네이밍 컨벤션을 이용해 `__name`이 프라이빗 변수라는 의미를 전달해 주어도, this를 통해 접근 가능하기 때문에 보안상 아무 소용이 없다.

클로저를 이용해 이 문제를 해결할 수 있다.
```javascript
function hello(name) {
  var _name = name;
  return function() {
    console.log('Hello, ' + _name);
  };
}

var hello1 = hello('yhan');
hello1() // => Hello, yhan
```
다른 인터페이스를 제공하는 것이 아니라면, 외부에서 name에 접근해 값을 바꿀 방법이 전혀 없게 되었다.

```javascript
function counter() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  };   
};

var counter1 = counter()

console.log(counter1.value()); // => 0
counter1.increment();
counter1.increment();
console.log(counter1.value()); // => 2
counter1.decrement();
console.log(counter1.value()); // => 1
```
이렇게 private method를 흉내내는 것도 가능하다.

##### 함수 팩토리 (Function Factory)
```javascript
function powerOfNFactory(power){
    return function generatedFunction(subject){
        return Math.pow(subject, power);
   }
}

const powerof2 = powerOfNFactory(2);
const powerof3 = powerOfNFactory(3);

console.log(powerof2(3)); // => 9
console.log(powerof3(2)); // => 8

```
powerOfNFactory는 특정한 값을 인자로 가지는 함수(generatedFunciton)들을 찍어는 함수 공장이라고 볼 수 있다.  
power 값을 받아 다른 인스턴스들과 완전히 독립된 상태로 기억하며, 추후 입력되는 subject 값에 따라 적절한 결과를 반환해 줄 수 있다.


### Closure Example

클로저 관련 대표적인 예제이다.
```javascript
function count() {
    for (var i = 1; i < 5; i += 1) {
        setTimeout(function() {
            console.log(i);
        }, i*100);
    }
}
count(); // => 5 5 5 5 5 
```
setTimeout의 설정 시간이 모두 지난 후 count 함수 스코프의 i 값에 접근 하는 시점은 이미 i=5로 변경된 후이다.
  
```javascript
function count() {
    for (var i = 1; i < 5; i += 1) {
        (function(j) {
            setTimeout(function() {
                console.log(j);
            }, j*100);
        })(i)  
    }
}
count(); // => 1 2 3 4 5
```
IIFE(Immediately Invoked Function Expression) 을 통해서 내부 setTimeout에 걸린 익명 함수를 클로저로 만들 수 있다.

```javascript
function count() {
    for (let i = 1; i < 5; i += 1) {
        setTimeout(function() {
            console.log(i);
        }, i*100);
    }
}
count(); // => 1 2 3 4 5
```
블록스코프를 사용하는 let변수를 이용하여 원하는 결과를 얻어낼 수 도 있다.  
for 문의 매 반복마다 새로운 스코프와 i가 생성된다. 
따라서 setTimeout의 콜백함수가 i를 참조하기 위해 상위 스코프를 검색할 때 각자 다른 i를 참조하는 것이다.

### References 
- [DailyEngineering > 클로저](https://hyunseob.github.io/2016/08/30/javascript-closure/)
- [MDN > 클로저](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Closures)
- [Must Know About Frontend > 클로저](https://github.com/baeharam/Must-Know-About-Frontend/blob/master/Notes/javascript/closure.md)
- [TOAST Meetup > 자바스크립트의 스코프와 클로저](https://meetup.toast.com/posts/86)
- [JavasScript Closures, Memoization and Factories](https://itnext.io/why-you-need-to-understand-javascript-closures-53efa66ae11a)
- [ES6 Class는 단지 Prototype의 상속의 문법설탕일 뿐인가](https://gomugom.github.io/is-class-only-a-syntactic-sugar/)