## arrow function

##### Prototype
```javascript
let es6 = () => 123;
let es5_1 = function () {return 123}
let es5_2 = new Function()
function es5_3 () {return 123}
console.log(es6.__proto__ === es5_1.__proto__) // => True
console.log(es6.__proto__ === es5_2.__proto__) // => True
console.log(es6.__proto__ === es5_3.__proto__) // => True
```


##### this
| case           | function() {} | () => {}      |
|----------------|---------------|---------------|
|객체 메소드        | 객체 | window | 
|일반 함수, 내부 함수, 콜백 함수 | window | 상위 scope this|
|생성자 함수| 인스턴스|x


this가 일반 함수와 달리 함수 스코프가 아닌 렉시컬 스코프(바로 바깥의 함수(혹은 class)의 this값)에 바인딩된다. 
```javascript
var bob = {
  _name: "Bob",
  _friends: ['john','sam'],
  printFriends() {
    this._friends.forEach(f => 
      console.log(this._name + " knows " + f));
  }
}
bob.printFriends() // => Bob knows ~~

var sam = {
  _name: "Sam",
  _friends: ['bob','kate'],
  printFriends() {
    this._friends.forEach(function(f) {
      console.log(this._name + " knows " + f);
    })
  }
}
sam.printFriends(); // => undefined knows ~~

```
객체 메소드로 사용시 this는 window에 바인딩.
```javascript
var obj = {
  name: 'obj name',
  print: function p() { console.log(this.name); }
};
obj.print(); // obj name

var obj = {
  name: 'obj name',
  print: ()=> { console.log(this.name); }
};
obj.print(); // undefined
```

```javascript
var adder = {
  base : 1,
    
  add : function() {
    var f = () => {console.log(this.base)};
    f();
  },
  remove: () => {
    var f = () => console.log(this.base);
    f();
  }
}
adder.add() // => 1
adder.remove() // => undefined
```
##### new 키워드 사용 불가
prototype 속성이 없음.
```javascript
var Foo = () => {};
console.log(Foo.prototype); // undefined
```

```javascript
let f = function() {
    return function () {
        console.log(this)
    }
}
f()()  //= > whindow
```

https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/%EC%95%A0%EB%A1%9C%EC%9A%B0_%ED%8E%91%EC%85%98
https://poiemaweb.com/js-this