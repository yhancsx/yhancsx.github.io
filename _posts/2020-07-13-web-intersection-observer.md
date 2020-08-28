---
title: "Web Intersection Observer API"  
category: web  
tags:
  - web
  - js
  - study
  - front-end
  - intersection observer
  - sticky
  - lazy loading
comments: true
---
지난 [css sticky](https://yhancsx.github.io/web/web-css-position/) 에서 detection을 위해 사용했던 custom hooks에 쓰이며
**lazy-loading**, **infinite-scroll** 을 가능하게 하는 Intersection Observer API에 대해 알아보자.

## Intersection Observer API
target 요소가 scroll 가능한 부모 요소 또는 browser viewport와 교차가 일어나는지 관찰하는 방법이다.

```javascript
const options = {
  root: document.querySelector('#scrollArea'), // null 이면 window 객체
  rootMargin: '0px',
  threshold: 1.0
}
const callback = (entries, observer) => {
    // entries: IntersectionObserverEntry[]
    // observer: Instance of IntersectionObserver, which is itself

    // callback 은 observer에 target이 연결되면 최초에 한번 실행된다.
}
const observer = new IntersectionObserver(callback, options);

const target = document.querySelector('#listItem');
observer.observe(target);
```
`root` : target의 visibility를 체크할 조상 엘리먼트. null이면 browser viewport로 할당된다.  
`rootMargin`: root rectangle을 조정할 margin 값. 일반 margin 속성처럼 `top right bottom left` 값으로 표기 가능하다.  
`threshold`: single number 또는 number array. callback을 실행시킬 임계점. **visible 영역의 비율**.  
```
현재 보이는 상태에서 아래로 스크롤
----------------   < threshold 1
               |
tareget element|   < threshold 0.5
               |
----------------   < threshold 0
``` 

## Intersection Observer Entry
#### Properties
- `isIntersecting`: 현재 target element가 root 내에서 보이고 있는지(intersection 인지) 여부. 이 값으로 교차하러 가는지 교차가 끝난지 구별 가능하다.  
- `boundingClientRect`: 현재 target element rectagle의 bounds 요소 (DOMRectReadOnly: `x`, `y`, `width`, `height`, `top`, `right`, `bottom`, `left`). Element.getBoundingClientRect() 를 사용한다.  
- `intersectionRect`: target element중 visible area bounds 요소.  
- `rootBounds`: root element의 bounds 요소.  
- `intersectionRatio`: 현재 intersection된 비율 (보이면 1, 안보이면 0).  

## React
```javascript
function App () {
  const ref = useRef();

  useEffect(() => {
    if (!ref?.current) return;
    const options = {
      root: null,
      rootMargin: '0px',
      threshold: 1.0,
    };
    const callback = (entries) => {
      const entry = entries[0];
      // ...
    };
    const observer = new IntersectionObserver(callback, options);
    observer.observe(ref.current);

    return () => observer && observer.disconnect();
  }, [ref]);
  
  <Component ref={ref} />
}
```


## Refereces
- [MDN > Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
- [MDN > Intersection Observer Entry](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry)


