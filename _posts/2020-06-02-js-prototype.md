---
title: "Javascript Prototype 이란?"  
category: js  
tags:
  - javascript
  - study
  - front-end
  - prototype
comments: true
---

자바스크립트는 프로토타입 기반 언어라고 한다.  
프로토타입이란 무엇이고 다른 java, python 등과 같은 클래스기반 언어와의 차이점이 무엇일까?

### Prototype vs Class

- Class
    - 기본적으로 class 기반의 언어는 `클래스`와 `인스턴스` 라는 개념을 가지고 있다. 
    - `클래스`는 어떠한 객체군을 특정짓는 속성(method, field)들을 정의하는 추상적인 개념이다. Person 이라는 클래스에 사람이 가질 수 있는 속성들을 정의할 수 있다 (age, gender 등). 
    - `인스턴스`는 `클래스`에 의해 만들어진 실체라고 볼 수 있다. Person 클래스에 의해 만들어진 `yhan`이 인스턴스인 것이다. `yhan`은 부모 class인 Person이 가지는 속성과 동일한 속성을 가진다.
    
- Prototype
    - 반면 javascript는 클래스와 인스턴스의 차이를 두지 않는다. 단지 `객체`만을 가질 뿐이다.
    - prototype(원형) 객체에 해당 객체가 가질 속성들을 정의 할 수있다. 
    - 또 다른 객체는 이 prototype 이나 다른 객체의 속성을 이어받아 생성될 수 있다.
    
### Javascript Prototype
##### Why Prototype?
```javascript
function Person() {
    this.eyes = 2;
    this.nose = 1;
    this.eat = function (){
        console.log('먹는다')
    }
}
```
javascript에서는 함수를 이용하여 객체 원형을 정의할 수 있다. 이러한 함수는 객체를 생성하는데 사용되므로 생성자 함수라고 불린다.  
this 키워드를 이용하여 프로퍼티를 명시할 수 있으며, 함수가 일반 함수인지 생성자 함수인지 구분을 위해, 생성자 함수의 첫 문자는 대문자로 표기한다.

```javascript
var kim = new Person();
var bae = new Person();
```
**new** 키워드를 사용하여 생성자 함수 Person으로부터 두개의 객체를 생성하면 eyes, nose라는 변수가 2개씩 메모리에 할당된다.

```javascript
function Person() {}

Person.prototype.eyes = 2;
Person.prototype.nose = 1;

var kim  = new Person();
var park = new Person();
``` 
prototype안에 eyes, nose 속성을 담아 메모리 사용량을 절반으로 줄일 수 있다.

위와 같이 동일한 프로퍼티를 가지는 객체를 여러개 생성하고, 메모리를 절약하기 위해 prototype 개념이 필요하다.

### Prototype Object
```javascript
var yhan = {};
var obj = new Object();
```
객체는 언제나 함수로부터 생성된다.  
우리가 많이쓰는 첫째줄과 같은 방식은, 사실은 두번째 줄과 같은 코드이다.
> Object와 마찬가지로 Function, Array도 모두 함수로 정의되어 있다.

함수가 정의될때는
1. 해당 함수에 constructor(생성자) 자격 부여 (-> new 키워드를 통해 객체 생성 가능)
2. 해당 함수의 Prototype Object 생성 및 연결

이 두가지 일이 일어난다.
 
```javascript
function Person() {}
// 또는 var Person = function (){}

Person.prototype (Person Prototype Object):
├── constructor: function Person()
└── __proto__: Object

Person.prototype.constructor === Person // => true
```
생성된 함수의 prototype이라는 속성을 통해 Prototype Object에 접근할 수 있으며 이 Prototype Object는 기본적으로 constructor 및 __proto__를 가지고있다.  
여기서 constructor는 다시 Person 함수를 가리키고 있고, `__proto__`는 뒤에서 설명할 Prototype Link이다.

```javascript
function Person() {}

Person.prototype.eyes = 2;
Person.prototype.nose = 1;

Person.prototype (Person Prototype Object):
├── eyes: 2
├── nose: 1
├── constructor: function Person()
└── __proto__: Object
```
이와같이 Person Prototype Object의 속성들을 마음대로 추가할수 있다.

### Prototype Link and Chain
```javascript
function Person() {}

Person.prototype.eyes = 2;
Person.prototype.nose = 1;

var kim = new Person();
console.log(kim.eyes) // => 2
kim.__proto__ === Person.prtotype // => true
```
위 예제에서 kim에는 eyes라는 속성이 없는데도 kim.eyes가 2를 반환 하였다.  
이는 `__proto__` 속성을 이용한 Prototype Link 때문에 가능한 것이다.  
prototype은 함수만 가지고 있는 속성이지만, `__proto__`은 모든 객체가 가지고 있는 속성이다.  
`__proto__` 는 객체가 생성될때, 조상이었던 생성자 함수의 Prototype Object를 가리킨다.  

kim 객체가 eyes를 직접 가지고 있지 않기 때문에, eyes를 찾을때까지 `__proto__` 를 계속해서 타고 올라가며 eyes를 찾는다.  
더이상 검사할 `__proto__`가 없을 때가 되면 undefined를 반환한다.  
이를 Prototype Chain이라고 한다.

```javascript
function Person() {}
var kim = Object.create(Person)

kim.__proto__ === Person.prototype // => false
kim.__proto__ === Person // => true
```
ES6에서 Object.create라는 문법이 도입되었다
생성된 객체의 프로토타입은 이 메소드의 첫번째 인수로 지정된다.
new 연산자와는 조금 다른 방식이다.

### Summary
**면접에서 js prototype이 뭐냐고 물어볼때 어떻게 답해야 하는가**  
생성자 함수로 생성한 객체들이 프로퍼티와 메서드를 공유하기 위해 사용하는 객체(Prototype Object)이다.  
생성된 객체들은 Prototype link(`__proto__`)를 통해 생성자 함수의 Prototype Object를 참조할 수 있으며, 이를 통해 상속을 구현할 수 있다.

### Reference
- [Javascript 객체모델의세부사항](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/%EA%B0%9D%EC%B2%B4_%EB%AA%A8%EB%8D%B8%EC%9D%98_%EC%84%B8%EB%B6%80%EC%82%AC%ED%95%AD)
- [Javascript 프로토타입 이해하기](https://medium.com/@bluesh55/javascript-prototype-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-f8e67c286b67)
- [Javascript 상속과 프로토타입](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain)
- [Javascript 생성자 함수와 프로토타입](https://doitnow-man.tistory.com/132)