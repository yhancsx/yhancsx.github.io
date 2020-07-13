---
title: "Web css position, sticky"  
category: web  
tags:
  - web
  - css
  - position
  - sticky
comments: true
---

## Position
- `static`: 요소를 일반적인 문서 흐름에 따라 배치. top, right, bottom, left, z-index 속성이 영향 주지 않음.
- `relative`: 요소를 일반적인 문서 흐름에 따라 배치하고 **자기 자신**을 기준으로 offset(`top`, `right`, `left`, `bottom`)의 값에 따라 위치 조정. 
다른 요소에는 영향 주지 않음(레이아웃에서 차지하는 공간은 static일 때와 같음).
- `absolute`: 요소를 일반적인 문서 흐름에서 제거하고, 레이아웃에서 공간도 배정하지 않음. 대신 가장 가까운 위치 지정(`static 속성이 아닌`) 부모 요소에 대해 상대적으로 offset 따라 배치. 
그런 부모가 없다면 가장 위의 태그(`body`) 기준. 
- `fixed`: 요소를 이반적인 문서 흐름에서 제거하고 페이지 레이아웃에 공간 배정하지 않음. viewport (현재 사용자에게 보이는 화면)의 초기 컨테이너 블록을 기준으로 offset 위치에 배정. 스크롤을 내려도 변하지 않음.

## Sticky
`position:sticky`
```javascript
스크롤 블록
└── 부모 블록
   └── sticky 블록
```
- sticky 모드에서 사용할 offset 지정 필수.
- sticky 모드가 아닐때는 `position:static` 처럼 동작한다. (offset 옵션 작동x)
- sticky 모드가 발동되는 임계점은 현재 `스크롤 블록` 상에서의 위치가 offset 을 지날때 이다.
- sticky 모드가 발동되면 **가장 가까운** `스크롤 매커니즘이 있는 부모` (`overflow: hidden, scroll, auto, overlay`)를 기준으로 offset 위치에 배치 된다.
- viewport 기준으로 위치가 고정는 것이 아닌 스크롤 블록 기준이라는점, 스크롤 블록에 padding 이 있다면 이 padding 안쪽을 기준으로 배치된다는점에서 `position: fixed`와는 조금 다름. 
- 실제 스크롤이 되지 않더라도 달라붙으므로 `overflow: hidden`의 부모가 껴 있으면 동작하지 않는것처럼 보일수 있음.
- sticky 모드가 끝나는 지점은 자신의 `부모 블록`이 `스크롤 블록`을 를 벗어날 때 이다.
- 중간에 끼어있는 써드파티 라이브러리의 컴포넌트가 `overflow:hidden` 속성을 가지고 있다면 `overflow:visible`로 변경하여 sticky 속성을 사용할 수 있다.

[CODEPEN DEMO](https://codepen.io/naradesign/pen/GeBxqe)


#### Detect Sticky trigger

- [Stack Overflow > IntersectionObserver](https://stackoverflow.com/questions/16302483/event-to-detect-when-positionsticky-is-triggered) 
- [react-use > useIntersection](https://github.com/streamich/react-use/blob/master/docs/useIntersection.md)

```javascript
export default function() {
    const ref = useRef(null)
    const intersection = useIntersection(ref, {root: null, rootMargin: '0px', threshold: 1,});
    const isSticky = intersection && intersection.intersectionRatio < 1

    return (
        <StickyBlock ref={ref}>
            ....
        </StickyBlock>
    ); 
}
```
   
 
## Reference
- [MDN > css position](https://developer.mozilla.org/ko/docs/Web/CSS/position)
- [WEBISFREE > Position sticky](https://webisfree.com/2018-11-16/[css]-%EC%8A%A4%ED%81%AC%EB%A1%A4%ED%95%B4%EB%8F%84-%EC%97%98%EB%A6%AC%EB%A8%BC%ED%8A%B8%EA%B0%80-%EC%9B%80%EC%A7%81%EC%9D%B4%EC%A7%80-%EC%95%8A%EB%8A%94-%EB%B0%A9%EB%B2%95-position-sticky)
- [W3S > Stick Element](https://www.w3schools.com/howto/howto_css_sticky_element.asp)