---
title: "Front-end Interview Cheatsheet"  
category: interview  
tags:
  - javascript
  - web
  - interview
  - study
  - front-end
---

frontend 관련 중요한 개념들을 cheatsheet 형태로 정리해 보자.

## [javascript prototype 이란?](https://yhancsx.github.io/js/js-prototype/)
- 생성자 함수로 생성한 객체들이 프로퍼티와 메소드를 공유하기 위해 사용하는 객체.
- 객체지향과 상속을 구현하는 javascript의 방식.
- 함수가 정의될때 prototype 객체가 생성된다.
- prototype link, chain을 통해 상속을 구현할 수 있다.
- class는 인스턴스를 생성 할 수 있는 추상적 개념이지만, prototype은 그 자체로도 객체라는 것이 큰 차이점이다.
- 프로토타입을 사용하면 객체들에 담을 속성/메소드를 한곳에서 관리하여 메모리 절약을 할 수 있다.

## [hoisting 이란?](https://yhancsx.github.io/js/js-scope-hoisting-closure/)
- ‘끌어올린다’라는 뜻으로 변수 및 함수 선언문이 스코프 내의 최상단으로 끌어올려지는 현상을 말한다.
- 정확히는, 어떤 스코프(Lexical Environment)가 형성됨과 동시에 그 안에서 정의된 변수들이 함께 생성되기 때문이다.
- var, function 선언문은 undefined로 초기화 된다.
- let, const도 호이스팅이 되지만 접근이 불가능하다. (Undeclared) 스코프가 생성될 당시 Temporal Dead Zone(TDZ)에 있기 때문이다.

## [closure 란?](https://yhancsx.github.io/js/js-scope-hoisting-closure/)
- 만들어진 환경(스코프)를 기억하여 그 환경 밖에서 실행될 때도 그 환경에 접근할 수 있는 방법.
- private 환경 제공
- 함수 팩토리

## [var, let, const 차이](https://yhancsx.github.io/js/js-scope-hoisting-closure/)
- 호이스팅 차이점
- var 는 함수 스코프, let, const는 블록 스코프
- let, const 는 재선언 불가능, const는 재할당 불가능.

## [객체 생성 과정](https://doitnow-man.tistory.com/132)
```javascript
function Person(name, age) {
    this.name = name,
    this.age = age
}
const obj = new Person('yhan', 28)
```
- 빈 객체 생성 
- 객체 property(name, age) 정의
- 객체 property에 인자 값 할당
- obj 변수에 객체 할당.
- 참고: [함수생성과정](https://medium.com/@kkak10/javascript-function%EC%9D%B4-%EC%83%9D%EC%84%B1%EB%90%98%EB%8A%94-%EA%B3%BC%EC%A0%95-15b74ecb7164)

## [this bind](https://yhancsx.github.io/js/js-this/)
- 일반: window 객체
- 객체 메소드: 해당 객체
- new: 생성된 인스턴스
- call/apply/bind: 전달된 객체
- arrow: lexical this

## [arrow function 차이점](https://yhancsx.github.io/js/js-arrow-function/)
- 문법
- this lexical 바인딩
- arguments 객체 전달 x
- new 사용 x

## [1급 객체](https://medium.com/@soeunlee/javascript%EC%97%90%EC%84%9C-%EC%99%9C-%ED%95%A8%EC%88%98%EA%B0%80-1%EA%B8%89-%EA%B0%9D%EC%B2%B4%EC%9D%BC%EA%B9%8C%EC%9A%94-cc6bd2a9ecac)
- js 에서는 함수 1급 객체
- 함수 리턴: 클로저 함
- 함수 인자: 콜백 함수
- 변수에 할당: 함수 표현식, 객체 메소드

## [브라우저 렌더링 과정](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction?hl=ko)
- html parsing > dom tree 생성
- css parsing > css tree 생성
- 두개 합쳐서 render tree 생성
- layout(reflow): element 위치 (x,y, width, height) 결정
- paint(repaint): 컬러, 컨텐츠등 작성

## [repaint, reflow in React](https://velopert.com/3236)
- virtual DOM에 변경사항 모음.
- Reconciliation 알고리즘으로 변경 필요한 부분만 batch 처리
    1) element type 다르면 자식노드 포함 모두 버림
    2) element type 같으면: 변경된 attribute 만 수정 / 자식노드들에 대해선 key로 식별.

## [이벤트 버블링, 캡처링, 위임](https://yhancsx.github.io/web/event-bubbling-capture-delegation/)
- 이벤트 전달 3단계: root -> 캡처링 -> target -> 버블링 -> root
- default는 버블링 단계에서 이벤트 핸들링.
- `useCapture: true` 설정으로 캡처링 단계에서 핸들링. (react에서는 `onClickCapture()`)
- 이벤트 위임: 자식 리스트 엘리먼트들의 이벤트를 부모 엘리먼트가 묶어서 처리하는 방식. 

## [es6문법](https://github.com/baeharam/Must-Know-About-Frontend/blob/master/Notes/javascript/es6.md)
- arrow function
- class
- import/export
- string template
- deconstructing
- parameter default value, rest, spread
- let, const
- [iterator/genterator/for...of](https://yhancsx.github.io/js/iterator-generator/)
- Symbol
- Map/Set/WeekMap/WeekSet
- Promise

## [CORS 의미 및 해결방법](https://evan-moon.github.io/2020/05/21/about-cors/)
- Cross-Origin Resource Sharing
- 브라우저는 다른 출처 (Protocol(HTTP/HTTPS), Host(Domain), Port가 다른 곳)에서의 request를 제한하는데 (Same Origin Policy), CORS 규칙을 지킨 요청은 예외적으로 허용한다. 
- 브라우저에 구현되어 있기 때문에, 서버는 정상 응답을 하지만 브라우저가 그 응답을 버린다.
- 서버에서 response 헤더의 `Access-Control-Allow-Origin` 값에 `이 리소스를 접근하는 것이 허용된 출처`를 명시하여야 한다. 
- Preflight Request 방식: 본 요청을 보내기전에 예비 요청(OPTIONS 메소드 사용)을 한번 보내 이 요청을 보내는것이 안전한지 확인.
- Simple Request 방식: 예비 요청없이 본 요청을 바로 보낸 후 CORS 위반 여부 확인.
- Credentialed Request 방식: request 헤더에 추가 인증 정보(`credentials`: `same-origin` || `include` || `omit`)를 담아 보안을 강화하는 방법.
- 해결방법: 
    - 서버 측에서 `Access-Control-Allow-Origin` 세팅.
    - 클라이언트 측에서 `webpack-dev-server`의 proxy 기능 사용. 웹팩 개발 서버에서 요청을 받아 프록싱함. / CRA 사용 시 package.json proxy 값 설정.
    - img,script,style 태그의 src는 SOP 정책에서 제외됨.
    
## 프론트엔드 성능최적화
- [tree shaking](https://yhancsx.github.io/js/tree-shaking/)
- [code splitting](https://reactjs.org/docs/code-splitting.html)
- [Server-side Rendering](https://yhancsx.github.io/web/server-side-rendering/) ([Exercise](https://github.com/yhancsx/ssr-example))
- React.memo(): props shallow comparison 
- [Image Lazy loading (Intersection Observer API)](https://yhancsx.github.io/web/web-intersection-observer/)

## React class/functional component 차이 및 사용
1. class
    - stateful
    - life cycle.
    - render method 사용.
2. functional
    - stateless
    - hooks 사용해서 state 관리.

## [javascript event loop](https://yhancsx.github.io/js/js-event-loop/)
- js 엔진은 싱글스레드. 
- 비동기 작업(ajax, setTimeout, UI rendering 등)은 web API 사용.
- 비동기 작업 호출 시 일어나는 일.
    1. call stack에서 web API로 콜백 함수와 같이 비동기 작업 전달.
    2. 작업 완료시 콜백 함수가 task queue 에 쌓임
    3. event loop가 call stack 비었는지 확인하고 콜백 함수 가져와 실행.
- micro task queue: task queue 보다 우선 순위 높음. (Promise)

## [undefined, null, undeclared](https://github.com/yangshun/front-end-interview-handbook/blob/master/contents/en/javascript-questions.md#whats-the-difference-between-a-variable-that-is-null-undefined-or-undeclared-how-would-you-go-about-checking-for-any-of-these-states)
- undefined: 변수에 값 할당되지 않은 상태
- null: 명시적으로 null 할당된 상태
- undeclared: TDZ. Reference Error 발생.

## [콜백지옥, promise](https://yhancsx.github.io/js/js-promise/), [async, await](https://yhancsx.github.io/js/async-await/)
- data fetch / user interaction 등의 비동기 작업을 많이 처리하는 js는 콜백함수 사용시 가독성이 떨어지고 디버깅을 어렵게 한다.
- Promise 객체를 사용하여 `then()`, `catch()` depth 1단계로 비동기 작업을 처리할 수 있다. -
- `then()`, `catch()` 또한 Promise 객체를 반환한다.
- async/await: Promise 객체를 한줄 한줄 선언적으로 사용할 수 있는 방법.
- async 함수의 리턴 값 또한 Promise 객체이다.

## typeof 연산자 결과 종류
- number, string, bigint, boolean, undefined, symbol

## client parameter 위변조 방지 방법

## [ssr, csr](https://yhancsx.github.io/web/server-side-rendering/)
- server-side rendering (universal rendering)
- 초기 화면은 서버에서 rendering 하여 전달. 
- 이후 js 코드 삽입(hydration)하여 SPA 처럼 동작.

## symbol