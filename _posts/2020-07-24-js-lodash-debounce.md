---
title: "Lodash debounce 이해하기"  
category: js  
tags:
  - js
  - web
  - study
  - front-end
  - lodash
  - debounce
  - this
  - arguments
---
[`you don't need`](https://github.com/you-dont-need/You-Dont-Need-Lodash-Underscore) 시리즈에 구현된 lodash의 debounce 함수를 파헤쳐 보자.

```javascript
function debounce(func, wait, immediate) {
  var timeout;
  return function() {
  	var context = this, args = arguments;
  	clearTimeout(timeout);
  	timeout = setTimeout(function() {
  		timeout = null;
  		console.log(context, args); // 1
  		if (!immediate) func.apply(context, args);
  	}, wait);
  	if (immediate && !timeout) func.apply(context, args);
  };
}
const onChange = (e) => { console.log(e) } //2
const element = document.getElementById('element_id') // <div id="element_id>element</div>
element.addEventListener('click', debounce(onChange))

// 1: <div id="element_id>element</div>, Argument { MouseEvent, callee, Symbol }
// 2: MouseEvnet 
```
- 첫번째 출력:  
element click 시 debounce 함수에서 리턴된 내부 함수가  호출된다.  
`addEventListener`의 콜백함수에서 `this`는 리스너에 바인딩되된 element 요소를 가리킨다. [(참고)](https://yhancsx.github.io/js/js-arrow-function/#addeventlistener-%ED%95%A8%EC%88%98%EC%9D%98-%EC%BD%9C%EB%B0%B1-%ED%95%A8%EC%88%98)  
또한 args는 리턴된 내부함수가 받아온 arguments를 가리킨다.  
- `func.apply(context, args)` :   
wait시간이 만족되었다면 debounce 함수의 첫번째 인자로 전달받은 함수를 element를 바인딩하고 argument를 parameter로 전달하여 실행한다. 두번째 `console.log` 가 출력된다.
- `clearTimeout(timeout)` :   
지정된 setTimeout 함수의 실행을 중지한다.  
- `if (immediate && !timeout) func.apply(context, args)`:   
등록된 timeout이 없으며, immediate===true 이면 즉시 실행한다.


#### Arrow Function
```javascript
function debounce(func, wait, immediate) {
  var timeout;
  return function() {
  	clearTimeout(timeout);
  	timeout = setTimeout(()=> {
  		timeout = null;
        console.log(this, arguments); // 1
  		if (!immediate) func.apply(this, arguments);
  	}, wait);
  	if (immediate && !timeout) func.apply(this, arguments);
  };
}
```

화살표 함수를 이용해서 this, arguments 값을 복사하지 않고도 쓸 수 있다.

## References
- [you don't need > You Don`t Need Lodash Underscore > _.debounce](https://github.com/you-dont-need/You-Dont-Need-Lodash-Underscore#_debounce)






