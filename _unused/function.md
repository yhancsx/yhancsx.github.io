```javascript
function a(){}
a.__proto__.constructor === Function
a.__proto__ === Function.prototype
a.prototype.__proto__ === Obejct.prototype


Function.prototype.__proto__ === Object.prototype
// => Function = new Object(~~~)

Object.prototype.__proto__ === null

```
a, Function 의 prototype 객체 모두 객체이기 때문에 __proto__가 Object.prototype 이다.


```javascript
Object.__proto__ === Function.prototype
Function.__proto__ === Function.prototype

Function.prototype = {
    call: ~~,
    bind: ~~,
    apply: ~~,
    ...
}
```
Object, Function 오프젝트 모두 함수로 정되어 있다.