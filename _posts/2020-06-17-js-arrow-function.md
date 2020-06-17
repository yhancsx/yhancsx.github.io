---
title: "Javascript Arrow Function"  
category: js  
tags:
  - javascript
  - study
  - front-end
  - function
  - es6
  - arrow function
  - scope
  - this
comments: true
---

ES6(2015)에 도입 된 화살표함수(arrow function)와 기존 함수의 차이점을 탐구해 보자.

## Syntax
```javascript
(param1, param2, …, paramN) => { statements }
(param1, param2, …, paramN) => expression // 다음과 동일함:  => { return expression; }

// parameter 하나뿐인 경우 괄호는 선택사항:
(singleParam) => { statements }
singleParam => { statements }

// parameter 없는 함수는 괄호가 필요
() => { statements }

// 객체 리터럴 표현 반환
params => ({foo: bar})

// ...rest, default parameter 지원
(param1, param2, ...rest) => { statements }
(param1 = defaultValue1, param2, …, paramN = defaultValueN) => { statements }

// destructuring 지원
var f = ([a, b] = [1, 2], {x: c} = {x: a + b}) => a + b + c;
f();  // 6
```
javascript 엔진이 화살표 함수를 만나게되면 Function 오브제트의 prototype에 연결된 constructor 메서드로 함수를 생성한다.  
이는 es5에서 function 키워드로 생성하는 방법과 같다.

## this
```javascript
var sam = {
  _name: "sam",
  _friend: 'bob',
  printFriends() {
    setTimeout(function(){
        console.log(this._name + " knows " + this._friend);
    })
  } 
}
sam.printFriends(); // => undefined knows undefined

var elith = {
  _name: "elith",
  _friend: 'jane',
  printFriends() {
    var that = this;
    setTimeout(function(){
        console.log(that._name + " knows " + that._friend);
    })
  } 
}
elith.printFriends(); // => elith knows jane

var yhan = {
  _name: "yhan",
  _friend: 'kate',
  printFriends() {
    setTimeout(()=> {
        console.log(this._name + " knows " + this._friend);
    })
  } 
}
yhan.printFriends(); // => yhan knows kate
```
일반 function 은 만들어진 상황에 따라 다른 this 바인딩이 일어난다.
- 생성자 함수: 생성된 인스턴스
- 객체 메소드: 해당 객체
- 일반/내부/콜백 함수: window(브라우자) or undefined(strict mode)

sam에서 일어나는 문제를 해결하기 위해 elith와 같은 방법을 사용했었는데, 이제는 arrow function 을 이용하면 된다.

arrow function은 this를 새로 정의하지 않고, 자신을 감싸는 바로 바깥 코드의 함수(혹은 class)의 동일한 this를 참조한다.  
즉, 렉시컬 스코프를 가진다.  
> 이는 this를 클로저 값으로 처리하는 것과 같은 효과이다.


## Etc
#### new
```javascript
var Foo = () => {};

var foo = new Foo // => TypeError: Foo is not a constructor at ~~ 

console.log(Foo.prototype); // => undefined
console.log(Foo.__proto__ === Function.prototype) // => true
```
일반 함수는 prototype 객체의 constructor 메소드를 호출하는 new 키워드를 사용하여 생성자 함수로 이용할 수 있다.  
하지만 화살표 함수는 prototype 객체가 연결되어 있지 않아 생성자 함수로써 사용할 수 없다.  
다만, 2번째 출력에서 암시하듯 Foo 또한 Function 오브젝트로 만든 함수인것은 맞다. 특별한 함수인것이다.

#### arguments
```javascript
var es5 = function() {
    console.log(arguments);
}
var es6 = () => {
    console.log(arguments);
}

es5() // => Arguments
es6() // => ReferenceError: arguments is not defined
```
일반 함수와는 다르게 arguments 프로퍼티를 사용할 수 없다.

#### 객체 메소드, 프로토타입 메소드
```javascript
var obj = {
  name: 'yhan',
  say: () => console.log(this.name),
  hello: function(){
    console.log(this.name)
  }
}
obj.say(); // => undefined
obj.hello(); // => yhan
```
객체 메소드(프로토타입 메소드)로 화살표 함수를 사용하는 것은 피해야 한다.
this가 객체가 아닌 전역 객체(window)를 가리키기 때문이다.

```javascript
var obj = {
  name: 'yhan',
  say() {
    console.log(this.name)
  }
}
```
ES6의 축약 메소드 표현을 사용하자.

#### addEventListener 함수의 콜백 함수
```javascript
var button = document.getElementById('myButton');

// Bad
button.addEventListener('click', () => {
  console.log(this === window); // => true
  this.innerHTML = 'Clicked button';
});

// Good
button.addEventListener('click', function() {
  console.log(this === button); // => true
  this.innerHTML = 'Clicked button';
});
```
addEventListener에서 지정하는 이벤트 핸들러는 콜백함수이지만 this는 리스너에 바인딩되는 요소를 가리킨다.
arrow function으로 정의하면 전역객체인 window를 가리킨다.

정확한 원리는 공부해 봐야겠지만 객체 내부 메소드와 this 바인딩이 같다고 생각하면 쉬울 것 같다.

#### prototype, yield
화살표 함수에서는 prototype 속성을 사용할 수 없다.
화살표 함수에서는 본문에 yield 키워드를 사용할 수 없다.

## References
- [Must know About Fontent > ES6](https://github.com/baeharam/Must-Know-About-Frontend/blob/master/Notes/javascript/es6.md)  
- [MDN > 화살표함수](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/%EC%95%A0%EB%A1%9C%EC%9A%B0_%ED%8E%91%EC%85%98)  
- [PoiemaWeb > 화살표함수](https://poiemaweb.com/es6-arrow-function)
- \[ECMAScript6, 두고두고보는 자바스크립트 표준 레퍼런스\] 김영보, 루비페이퍼, 2017