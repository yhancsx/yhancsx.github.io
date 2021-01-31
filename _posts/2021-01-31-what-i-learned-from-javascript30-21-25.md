---
title: "What I Learned from Javascript30 (21~25)"
category: web
tags:
  - javascript
  - web
  - front-end
  - study
  - window
  - navigator
  - geolocation
  - viewport
  - getBoundigClient
  - DOMRect
  - sticky
  - event
---


[Wes Bos](https://github.com/wesbos) 님의 Vanilla Javascript 강의인 [Javascript 30](https://javascript30.com)을 따라하며 새롭게 배운 내용들을 정리한 포스트입니다.

## 21. Geolocation

#### [Navigator Object](https://developer.mozilla.org/en-US/docs/Web/API/Navigator)
- `window.navigator`로 접근
- represents the state and the identity of the user agent.
- network, cookie, credential, device, geolocations 등의 정보들을 담고 있는 객체.

#### [Geolocation Object](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation)

- `navigator.geolocation` 로 접근
- Device의 현재 위치에 접근할 수 있는 API
- `geolocation.watchPosition()`
- Device의 위치가 변할 때마다 실행할 handler 등록 할수 있는 method.
- callback parameter로 `GeolocationPositoin` object 전달.

#### [GeolocationPosition Object](https://developer.mozilla.org/en-US/docs/Web/API/GeolocationPosition)
- Device의 현재 position과 timestamp정보를 가지고 있는 object
- position은 GeolocationCorordinates object를 통해 표현.

#### [GeolocationCorordinates](https://developer.mozilla.org/en-US/docs/Web/API/GeolocationCoordinates)
- `GeolocationPosition.coords`로 접근
- x,y 좌표, 고도 및 정확도 등으 담고 있는 객체.

```javascript
navigator.geolocation.watchPosition(
    geolocationPosition => {
        const { 
            latitude,   
            longitude, 
            altitude, 
            accuracy, 
            altitudeAccuracy, 
            heading
        } = geolocationPosition.coords;
        
        // ...
    }
)
```

## 22. Follow Along Links

#### Window scrollX, scrollY

- 문서가 현재 수평 / 수직으로 얼마나 스크롤 되었는지를 픽셀단위로 반환.

#### [Viewport](https://developer.mozilla.org/en-US/docs/Glossary/Viewport)
- 현재 browser window 상에서 보여지고 있는 (전체 document의) 부분 

#### getBoundingClientRect()

- [DOMRect](https://developer.mozilla.org/en-US/docs/Web/API/DOMRect) Object 반환.
```javascript
const { x, y, width, height, top, right, bottom, left } = element.getBoundingClientRect();
```
- viewport로부터의 위치
- x,y: 좌상단 꼭지점의 위치
- width, height: 너비, 높이
- top: y 값과 같음. (height이 음수일 땐 y + height)
- left: x 값과 같음. (width가 음수일 땐 x + width)
- bottom: y + height 값과 같음. (height이 음수일땐 y) 
- right: x + width 값과 같음. (width가 음수일 땐 x) 

## 23. Speach Synthesis

## 24. Sticky Nav

```javascript
const nav = document.querySelector('#nav');
const topOfNav = nav.offsetTop;

if(window.scrollY >= topOfNav) {
    // fix nav to top by 
    // position: fixed
} else {
    // back to normal
}
```

## 25. Event Capture, Propagation, Bubbling and Once(https://yhancsx.github.io/web/event-bubbling-capture-delegation/)

#### once
- event listener가 설정된 이후 최대 1번만 실행.
- 최초 1번 실행 뒤 listener 삭제됨.