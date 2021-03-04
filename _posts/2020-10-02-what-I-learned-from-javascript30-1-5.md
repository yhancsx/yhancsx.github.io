---
title: "What I Learned from Javascript30 (1~5)"
category: web  
tags:
  - web
  - front-end
  - html
  - css
  - javascript
  - transition
  - query selector
  - data attribute 
---

[Wes Bos](https://github.com/wesbos) 님의 Vanilla Javascript 강의인 [Javascript 30](https://javascript30.com)을 따라하며 새롭게 배운 내용들을 정리한 포스트입니다.
 
## 1. Drum Kit

#### Document querySelector
```javascript
document.querySelector([css selector])  => Element
document.querySelectorAll([css selector]) => NodeList
```
- getElementId, getElementsByClassName 등에 비해 css selector 사용할 수 있다는 장점.    
- getElementsByClassName 이 반환하는 HTMLCollection 은 실시간(live), NodeList는 정적(non-live)

#### Html data attribute
```html
<div data-key="63">
...
</div>
```

```javascript
const element = document.querySelector(`div[data-key="63`);
console.log(element.dataset.key) // => 63
```

#### Element class list manipulation
```javascript
const hadler = (e) => {
  e.target.classList.add('some-class')
  e.target.classList.remove('some-class')
  e.target.classList.toggle('some-class')
}
``` 


#### Transition-end event
```javascript
element.addEventListener("transitionend", handler);
```


## 2. Clock

#### Transition timing function
![transition-timing-function]({{site.url}}{{site.baseurl}}/assets/images/javascript30/transition_timing_function.png){: width="300 height="300" }

#### Element style manipulation
```javascript
element.setAttribute("style", "color: #000")
element.style = "color:#000";
element.style.setProperty("color", "#000")
element.style.color = "#000"
```

#### Transform origin
```css
.transform{
  transform: rotate(90deg);
  transform-origin: 100%;
}
```
- rotate할 피벗 지점.  
- 0%: 왼쪽 끝, 100%: 오른쪽 끝, 50%(default): 중앙 

## 3. CSS Variables

#### CSS variables
```css
:root {
  --base: #cecece;
  --spacing: 10px;
  --blur: 10px;
}

img {
  padding: var(--spacing);
  background: var(--base);
  filter: blur(var(--blur));
}
```

### Html input type range, color
```html
<input type="range" min="10" max="200" value="10" />
<input type="color"  value="#cecece" />
```
![input-color]({{site.url}}{{site.baseurl}}/assets/images/javascript30/input_range.png){: width="200 height="200" }
![input-range]({{site.url}}{{site.baseurl}}/assets/images/javascript30/input_color.png){: width="200 height="200" }

## 4. Array Cardio 1


## 5. Flex Panels Image Gallery

#### Transition event property name
```javascript
function handler(e) {
  console.log(e.propertyName) // => ex) transform, font-size, width, flex-grow, ...
}
element.addEventListener("transitionend", handler);
```