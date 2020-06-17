
##### this
| case           | function() {} | () => {}      |
|----------------|---------------|---------------|
|객체 메소드        | 객체 | window | 
|일반 함수, 내부 함수, 콜백 함수 | window | 상위 scope this|
|생성자 함수| 인스턴스|x



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
```javascript
let f = function() {
    return function () {
        console.log(this)
    }
}
f()()  //= > whindow
```