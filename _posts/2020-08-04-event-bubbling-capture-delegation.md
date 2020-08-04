---
title: "Web event propagation"
category: web
tags:
  - web
  - study
  - front-end
  - javascript
  - event
  - react
---

사용자와의 인터렉션이 많아진 현대 web에서는 거의 모든 개발이 event 기반이라고 할 수 있다.
이벤트를 등록하는 방법, 전달 방식들을 파헤쳐 보자.

## Event Register

```javascript
const handleEvent = (e) => {
  // doSomething
};
<div onClick={handleEvent}>// ...</div>;
```

react에서 개발할 때는 주로 jsx 태그의 `onClick` property (html은 `onclick`)에 event를 inline으로 등록하여 원하는 로직을 구현하게 된다.

```javascript
const target = document.getElementById('target')[0]
target.addEventListener('click', handleEvent)
<div id='target'>
  // ...
</div>
```

html과 vanillaJs를 이용하여 개발할 때는 주로 `EventTarget` 객체의 `addEventListener` 메서드를 이용한다.

## target vs currentTarget

element에서 event 가 발생하면 리스너에 등록된 함수는 `Event` 객체를 parameter로 받는다.

이 객체안에는 2개의 노드 정보(target, currentTarget)가 포함되어 있다.

- Event.target: event가 발생한 element.
- Event.currentTarget: listener가 등록되어 event를 처리하고 있는 현재 element.

## Event Propagation

![event propagation](https://www.w3.org/TR/DOM-Level-3-Events/images/eventflow.svg)
[출처 w3.org](https://www.w3.org/TR/DOM-Level-3-Events/#event-flow)

Event 가 발생하면

1. 가장 상위 노드부터 target 노드까지 찾아나간다. (Event Capture Phase)
2. target 노드
3. 가장 상위 노드까지 이벤트를 전달한다. (Event Bubbling Phase)

## Event Bubbling

```html
<div class="one">
  <div class="two">
    <div class="three"></div>
  </div>
</div>
```

```javascript
const handleEvent = (e) => console.log(e.currentTarget.className)
const divs = document.querySelectorAll("div");
divs.forEach((div) => {
  div.addEventListener("click", handleEvent);
});

> three
> two
> one
```

default 옵션은 event bubbling이다. 발생한 지점으로부터 위로 전달된다.

## Event Capture

```html
<div class="one">
  <div class="two">
    <div class="three"></div>
  </div>
</div>
```

```javascript
const handleEvent = (e) => console.log(e.currentTarget.className)
const divs = document.querySelectorAll("div");
divs.forEach((div) => {
  div.addEventListener("click", handleEvent, true);
});

> one
> two
> three
```

반대로 DOM tree의 가장 상단(`\<body>`)에서부터 event 발생 target 까지 내려오며 listener를 동작시키고 싶으면 event capture를 사용하면 된다.

addEventListener의 3번째 parameter(_useCapture_) 가 true이면 event capture 모드로 작동한다.

#### useCapture

> **useCapture**: A Boolean indicating whether events of this type will be dispatched
> to the registered listener before being dispatched to any EventTarget beneath it in the DOM tree.
> Events that are bubbling upward through the tree will not trigger a listener designated to use capture.
> ([MDN](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener))

해석이 어렵다...

`현재 발생 된 event가 DOM tree상 아래에 있는 target에 dispatch 되기 전에 이 리스너에 dispatch될지를 나타낸다. 위로 bubbling 되고 있는 event는 이 lisenter를 trigger 시키지 않는다.`

위에서부터 아래로 찾아오는 capture phase 에서 listener를 trigger 시킬지 결정한다.

#### alternately capture

한 event가 전달되는 과정에서 useCapture 값이 섞여 있으면 어떻게 동작할까?

```javascript
<div class="root"> false ↑
  <div class="one"> true ↓
      <div class="two"> false ↑
          <div class="three"/> true ↓
      </div>
  </div>
</div>

const handleEvent = (e) => console.log(e.currentTarget.className)
const divs = document.querySelectorAll("div");
divs.forEach((div, index) => {
  div.addEventListener(
    "click",
    handleEvent,
    index % 2 === 1
  );
});

> one
> three
> two
> root
```

- .three 가 클릭 된 후
- root에서 아래방향으로 전달되며 useCapture가 true인 `.one`의 listener를 동작시킨다.
- 발생된 element인 `.three`를 찍고, `.two` 및 `.root`에 차례로 bubbling된다.

## Event.stopPropagation

```javascript
<div class="root"> ↑ stop
  <div class="one"> ↓
      <div class="two"> ↑ stop
          <div class="three"/> ↓
      </div>
  </div>
</div>

var divs = document.querySelectorAll("div");
divs.forEach((div, index) => {
  div.addEventListener(
    "click",
    (e) => {
      handleEvent(e),
      index % 2 === 0 && e.stopPropagation()
    },
    index % 2 === 1
  );
});

> one
> three
> two
```

event를 처리한 후 더이상 전파되는 것을 원치 않는다면 Event.stopPropagation을 사용하면 된다.

현재 listener 이후의 event 전달을 중단한다.

## Event Delegation

event가 부모 또는 자식 노드로 전달되는 속성 떄문에 이벤트를 위임 시킬 수 있다.

```javascript
<div onClick={(e) => console.log(e.target.className)}>
  <div className="one" />
  <div className="two" />
  <div className="three" />
  <div className="four" />
</div>
```

하위 노드들에서 발생할 수 있는 event를 묶어 부모 노드에서 처리해주면 작성해야 하는 코드도 줄어들고, 자식 노드 동적으로 추가 때마다 listener를 또다시 달지 않아도 된다.

## addEventListener vs onclick

event를 등록시킬 때 addEventListener와 onclick에 어떤 차이점이 있을지 알아보았다.

#### addEventlistener

- 이론적으로 한개 element에 대해 무한개의 event를 등록 시킬 수 있다.
- 사실상 client-side의 메모리나 성능 이슈로 개수 제한이 된다.
- 3번째 parameter로 event capture 모드를 동작시킬 수 있다.

#### onclick

- 원하는 element에 inline 형태로로 event를 등록시킨다. 간편하고 직관적이다.
- 하나의 event listener만 등록시킬 수 있다.
- event capture 설정이 불가능하다.

## React

react에서 사용하는 onClick()은 bubbling 이다. event capture를 사용하려면 onClckCapture()를 사용하여야 한다.

react에서 event 발생시 전달되는 Event 객체는 한번 래핑된 이벤트이다.  
원래의 Event 객체와 비슷한 인터페이스를 가지고 있지만 stopPropagation 등은 조금 다르게 동작한다고 하는데 추후 to study list이다.

## References

- [이벤트 버블링, 이벤트 캡처 그리고 이벤트 위임까지](https://joshua1988.github.io/web-development/javascript/event-propagation-delegation/)
- [MDN > EventTarget.addEventListener()](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
- [w3.org > eventflow](https://www.w3.org/TR/DOM-Level-3-Events/#event-flow)
- [addEventLinstener vs onclick](https://stackoverflow.com/questions/6348494/addeventlistener-vs-onclick)
- [tapjoy korea > react의 이벤트 핸들러](https://medium.com/tapjoykorea/%EB%A6%AC%EC%95%A1%ED%8A%B8-react-%EC%9D%98-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%ED%95%B8%EB%93%A4%EB%9F%AC-event-handler-syntheticevent-nativeevent-3a0da35e9e3f)
