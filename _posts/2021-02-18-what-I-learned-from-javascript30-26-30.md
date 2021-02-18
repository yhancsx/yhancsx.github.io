---
title: "What I Learned from Javascript30 (26~30)"
category: web
tags:
  - javascript
  - web
  - front-end
  - study
  - browser
  - html
  - mouse event
  - video element
  - attribute
  - form
---

[Wes Bos](https://github.com/wesbos) 님의 Vanilla Javascript 강의인 [Javascript 30](https://javascript30.com)을 따라하며 새롭게 배운 내용들을 정리한 포스트입니다.

## 26. Stripe Follow Along Dropdown

## 27. Click and Drag to Scroll1

#### Mouse Event PageX, PageY
```javascript
const { pageX, pageY } = MouseEvent;
```

전체 page 내에서의 마우스 위치. viewport가 아닌 전체 page 기준 offset 값.

## 28. Video Speed Controller UI

#### video playback Rate
```html
<video src="..." />
```
```javascript
const video = document.querySelector(video);
const playbackRate = 1; // 정상속도

video.playbackRate = playbackRate;
```

#### setAttribute()

```javascript
video.setAttribute('playbackRate', playbackRate)
```
setAttribute는 모든 값을 string으로 변환하기 때문에 정상적으로 동작하지 않을 수 있다.

## 29. Countdown Clock

#### name attribute

name attribute를 사용하면 element에 쉽게 접근할 수 있다.

```html
<form name="customForm" id="custom">
  <input type="text" name="customInput" />
</form>
```

```javascript
const form = document.customForm
const input = form.customInput
const value = form.value
```

## 30. What a Mole Game

#### [Event isTrusted](https://developer.mozilla.org/en-US/docs/Web/API/Event/isTrusted)

```javascript
Event
{
  ...
  isTrusted: true
  ...
}
```
이벤트가 사용자 행위에 의해 발생했으면 true, 스크립트나 dispatchEvent 등의 기타 요인으로 인해 발생했으면 false.
