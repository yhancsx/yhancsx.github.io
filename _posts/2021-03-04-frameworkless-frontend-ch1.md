---
title: "Frameworkless Frontend Development - 1. 프레임워크 이야기 "
category: web
tags:
  - javascript
  - web
  - front-end
  - study
  - frameworkless
  - vanillajs
  - jquery
  - angular
  - angularjs
  - react
---

*Francesco Strazzullo*님의 [프레임워크 없는 프론트 엔드 개발](https://book.naver.com/bookdb/book_detail.nhn?bid=17758920)(원제 Frameworkless Frontend Development)을 읽고 요악하는 포스트입니다.


## Framework?
> A supporting structure around which something can be built (무언가를 만들 수 있는 지지 구조)- Cambridge Dictionary

소프트웨어에서도 프레임워크는 서비스, 컴포넌트 같은 요소를 사용해 어플리케이션을 구축하는데 필요한 구조를 제공한다.

#### Framework vs Library
Lodash.js, Moment.js 등은 라이브러리이다. 프레임워크와 라이브러리의 차이점은 무엇일까?

> 프레임워크는 코드를 호출한다. 코느는 라이브러리를 호출한다.
내가 작성한 함수가 호출당하면 프레임워크, 내가 호출하면 라이브러리라고 할 수 있다.

#### Framework 방식
프레임워크는 코드를 어떻게 작성해야 하는지를 강제한다.

react 공식문서에는 자신을 라이브러리라고 소개하고 있지만 프레임워크 방식(제약사항)이 있다.

아래는 react와 react-pose를 이용해 상자를 애니메이션하는 코드.

```javascript
const Box = posed.div({
  hidden: { opacity: 0 },
  visible: { opacity: 1 },
  transition: {
    ease: 'linear',
    duration: 500
  }
})

class PoseExample extends Component {
  constructor (props) {
    super(props)
    this.state = {
      isVisible: true
    }

    this.toggle = this.toggle.bind(this)
  }

  toggle () {
    this.setState({
      isVisible: !this.state.isVisible
    })
  }

  render () {
    const { isVisible } = this.state
    const pose = isVisible ? 'visible' : 'hidden'
    return (
      <div>
        <Box className='box' pose={pose} />
        <button onClick={this.toggle}>Toggle</button>
      </div>
    )
  }
}
```
직접 element를 애니메이션하지 않고 애니메이션을 정의한다음 상태를 변경한다.
react에서는 이런 선언적 프로그래밍 패턴을 유도한다.

[Web Animation API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API)를 이용해 똑같은 애니메이션을 구현한 방법이다.
```javascript
const animationTiming = {
  duration: 500,
  ease: 'linear',
  fill: 'forwards'
}

const showKeyframes = [
  { opacity: 0 },
  { opacity: 1 }
]

const hideKeyframes = [
  ...showKeyframes
].reverse()

class PosedExample extends Component {
  // ...

  componentDidUpdate (prevProps, prevState) {
    const { isVisible } = this.state
    if (prevState.isVisible !== isVisible) {
      const animation = isVisible ? showKeyframes : hideKeyframes
      this.div.animate(animation, animationTiming)
    }
  }
  render () {
    return (
      <div>
        <div ref={div => { this.div = div }} className='box' />
        <button onClick={this.toggle}>Toggle</button>
      </div>
    )
  }
}
```
직접 element에 애니메이션을 일으키는 명령형 패턴을 사용했다. react 개발자에게 이런 방법은 어색하게 느껴지는데, 이 느낌이 react는 라이브러리가 아닌 프레임워크라고 할 수 있는 이유이다.

react 핵심 멤버 및 커뮤니티에서 표준으로 삼고 제약하기 때문이다. 즉, 선언적 패턴은 react 방식의 일부이다.
> 작업을 처리할 때 `프레임워크 방식`을 사용하고 있다면 프레임워크라고 볼 수 있다.


## History of Javascript Frameworks

#### jQuery
모든 자바스크립트 프레임워크의 모체이다. `var element = $(".my-class")` 와 같은 selector 구문으로 프론트엔드 개발에 편의성을 제공했다.

#### AngularJS
SPA(Single Page Application)을 주류로 만드는데 큰 역할을 했다. 가장 주목할 만한 기능은 모델과 뷰를 연결하는 양방향 데이터 바인딩.

#### React
현재 가장 인기있는 프레임워크(or 라이브러리). DOM을 직접 수정하는 대신 state를 변경하면 react가 알아서 나머지 작업을 수행한다.

기술적으로 보면 프레임워크가 아닌 렌더링 라이브러리이다. Redux, Mobx 등을 통해 이 간극을 메울 수 있다.

#### Angular
AngularJS의 새로운 버전으로 시작되어 Angular2로 불리었지만 둘사이의 완전히 다른 접근 방식 때문에 이름이 변경되었다.

Typescript를 표준으로 삼고 있다.


## 기술 부채

모든 프레임워크는 기술 부채를 가지고 있다. 의존성이 생기고 미래에 코드 변경이 어려울 수 있다. 프레임워크 아키텍처 자체에 이미 비용을 포함하고 있다.

내외부 여러가지 요인으로 인해 소프트웨어, 아키텍처에 변경이 필요할 때가 있다. 프레임워크는 이런 상황에서 장애물이 된다.

금융 세계에서 부채가 항상 나쁜 것이 아니듯 기술 부채도 반드시 나쁜것만은 아니다. 합리적인 이유로 빠른 솔루션을 사용한다면 기술 부채가 아닌 기술 투자가 될 수 있다.
