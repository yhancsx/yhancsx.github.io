---
title: "What I Learned from Javascript30 (16~20)"
category: web
tags:
  - javascript
  - web
  - front-end
  - study
  - browser
  - html
  - offset
  - text shadow
  - navigator
  - mediaDevices
  - srcObject
  - requestAnimationFrame
  - audio
  - video
  - SpeechRecognition
---

[Wes Bos](https://github.com/wesbos) 님의 Vanilla Javascript 강의인 [Javascript 30](https://javascript30.com)을 따라하며 새롭게 배운 내용들을 정리한 포스트입니다.

## 16. Mouse Move Shadow

#### offset Left, Top, Width, Height
```javascript
const { offsetLeft,  offsetTop } = element 
```
Integer representing the offset to the [left, top] in pixels from the closest _relatively positioned_ parent element. 
부모 노드로 부터의 오프셋 위치.

```javascript
const { offsetWidth,  offsetHeight } = element 
```
Returns the [height,width] of an element, including vertical padding and borders, as an integer.  
element의 width, height 값.

#### text-shadow
```javascript
text-shadow: 1px 1px 2px black; 
```
`offset-x | offset-y | blur-radius | color`

## 17. Sort Without Articles

## 18. Adding Up Times with Reduce

## 19. Web cam

```javascript
 navigator.mediaDevices
    .getUserMedia({ video: true, audio: false })
    .then((localMediaStream) => {
      video.srcObject = localMediaStream;
      video.onloadedmetadata = function (e) {
        video.play();
      };
    }
```
[reference](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia)

#### [Navigator](https://developer.mozilla.org/ko/docs/Web/API/Navigator)
- Represents the **state** and the **identity** of the user agent. window 객체안에 포함 됨.

#### navigator.mediaDevices
- 카메라, 마이크, 화면 공유와 같이 현재 연결된 미디어 입력 장치에 접근할 수 있는 `MediaDevices` 객체 반환.
- 미디어 데이터 제공하는 하드웨어에 접근할 수 있는 방법.

#### MediaDevices.getUserMedia()
- 사용자에게 권한을 요청한 후, 시스템의 카메라와 오디오 각각 활성화.
- 장치의 입력 데이터를 비디오/오디오 트랙으로 포함한 `MediaStream` 객체 반환.
- Promise 형태로 반환.

#### HTMLMediaElement.srcObject
- MediaStream, MediaSource, Blob, File 와 같은 미디어 객체를 서빙하기 위한 attribute.
- `srcObject`를 지원하지 않는 old browser (e.g., IE)는 `video.src = URL.createObjectURL(mediaStream)` 형태로 대체.
 
#### [HTMLVideoElement.videoHeight(videoWidth)](https://developer.mozilla.org/en-US/docs/Web/API/HTMLVideoElement/videoWidth#About_intrinsic_width_and_height)
- 현재 video 화면의 resolution(해상도) height(width) 값. 

#### [requestAnimationFrame()](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)
```javascript
function paintToCanvas() {
  const width = video.videoWidth;
  const height = video.videoHeight;

  canvas.width = width;
  canvas.height = height;
  ctx.drawImage(video, 0, 0, width, height);

  id = requestAnimationFrame(paintToCanvas);
}

...
cancelAnimationFrame(id)
...
```
- 기본적으로는 브라우저 프레임 속도 (60fps, 프레임당 16ms), 보통은 모니터 주사율에 맞추어 (W3C 표준) 콜백 함수를 실행.
> This will request that your animation function be called before the browser performs the next repaint.
- 재귀적으로 호출 하여 animation 함수 실행.
- callback 함수에 전달되는 parameter는 [time origin](https://developer.mozilla.org/en-US/docs/Web/API/DOMHighResTimeStamp#The_time_origin)(the beginning of the current document's lifetime.) 으로부터 경과된 시간. 애니메이션 시 이동할 위치 계산에 사용. 
- setInterval, setTimeout 은 최소시간만을 보장하지만 requestAnimationFrame 프레임 속도에 맞는 일정한 간격으로 실행가능. 
- for loop는 메인스레드를 차지 하지만 requestAnimationFrame은 비동기적으로 작동.
- 가장 마지막에 호출된 requestAnimationFrame의 반환값을 cancelAnimationFrame에 넘겨주어 정지 가능.

- [TOAST UI > 성능 최적화](https://ui.toast.com/fe-guide/ko_PERFORMANCE/)
- [JavaScript window.requestAnimationFrame 튜토리얼](https://blog.eunsatio.io/develop/JavaScript-window.requestAnimationFrame-%ED%8A%9C%ED%86%A0%EB%A6%AC%EC%96%BC)

#### [audio/video events](https://www.w3schools.com/tags/av_event_loadstart.asp)
아래 순서로 발생
1. loadstart
2. durationchange
3. loadedmetadata
4. loadeddata
5. progress
6. canplay
7. canplaythrough

#### [HTMLCanvasElement.toDataURL](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toDataURL)
```javascript
const image = canvas.toDataURL("image/jpeg");

const link = document.querySelector("a");
link.href = image;

const img = document.querySelector("img");
img.src = image;
```
현재 이미지를 Data URI (text based representation of picture) 형태로 변환
\<a> 나 \<img> 에 담아 사용가능.

#### HTML <a> download Attribute
```javascript
<a href="http://some.link" download="filename"/>
```
링크 클릭시 href의 타겟이 [filename] 이름의 파일로 다운로드됨.

#### Node.insertBefore()
```javascript
let insertedNode = parentNode.insertBefore(newNode, referenceNode);
```
- insertedNode: The node being inserted (the same as newNode)
- parentNode: The parent of the newly inserted node.
- newNode: The node to be inserted.
- referenceNode: The node before which newNode is inserted. If this is null, then newNode is inserted at the end of parentNode's child nodes.
  - parentNode 의 child가 아니면 에러 반환.
  
## 20. Speech Detection

#### [SpeechRecognition](https://developer.mozilla.org/en-US/docs/Web/API/SpeechRecognition)

[Github Examples](https://github.com/yhancsx/Javascript30/blob/master/20%20-%20Speech%20Detection/index.js)