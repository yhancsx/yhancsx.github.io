---
title: "Frameworkless Frontend Development - 4. 웹 구성 요소"
category: web
tags:
  - javascript
  - web
  - front-end
  - study
  - frameworkless
  - vanillajs
  - component
  - custom element
  - html template
  - shodow dom
  - HTMLElement
---

*Francesco Strazzullo*님의 [프레임워크 없는 프론트 엔드 개발](https://book.naver.com/bookdb/book_detail.nhn?bid=17758920)(원제 Frameworkless Frontend Development)을 읽고 요악하는 포스트입니다.

 > 웹 구성요소: web component

## API

[웹 구성요소](https://developer.mozilla.org/en-US/docs/Web/Web_Components)는 다음 3가지 기술로 구현 가능하다
- [HTML 템플릿](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template): \<template> 태그는 렌더링 되지 않지만 javascript로 동적인 content를 생성할 경우 `스탬프`로 사용하기 유용하다.
- [Custom Element](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements): \<app-claender>와 같은 custom html tag를 작성할 수 있다.
- [Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM)

> 3가지 기술 모두 IE에서는 지원하지 않는다.

#### Custom Element - Chapter4/00

Custom Element API를 사용할 때는 대시(-)로 구분된 두 단어 이상의 태그를 사용해야 한다. 한 단어 태그는 W3C에서만 사용할 수 있다.

**HelloWorld.js**
```javascript
export default class HelloWorld extends HTMLElement {
  connectedCallback () {
    window.requestAnimationFrame(() => {
      this.innerHTML = '<div>Hello World!</div>'
    })
  }
}
```

HTML Element class를 확장해서 만들 수 있다.

- `connectedCallback`: custom element의 life cycle 메소드 중 하나로, 구성 요소가 DOM에 연결될 때 호출된다. (~ React의 `componentDidMount`)
- `disconnectedCallback`: component가 DOM에서 작세죌 때 호출되는 메소드.

새롭게 생성한 component를 사용하려면 브라우저 component 레지스트리에 추가해야 한다.

**index.js**
```javascript
window
  .customElements
  .define('hello-world', HelloWorld)
```

레지스트리에 추가함으로써 tag 이름과 custom class는 연결되었고, html tag로 사용할 수 있다.

**index.html**
```html
<hello-world>
```

#### 속성 관리 - Chapter4/00.1

표준 element에는 attribute를 설정할 수 있는 방법이 3가지가 있다.
- `<input type="text" value="fameworkless">`
- `input.value = "frameworkless"`
- `input.setAttribute("value", "frameworkless)`

사용자 인풋을 통해 입력 받은 값은 attribute를 통해 접근 하는 값과 동기화 된다.

custom element도 같은 방식으로 attribute를 관리할 수 있어야 한다.

**HelloWorld.js**
```javascript
export default class HelloWorld extends HTMLElement {
  get color () {
    return this.getAttribute('color') || 'black'
  }

  set color (value) {
    this.setAttribute('color', value)
  }

  // ...
```

getter, setter로 getAttribute, setAttribute를 래핑하면 마크업에서도 속성을 사용할 수 있다.

**index.html**
```html
<hello-world color="red"></hello-world>
<hello-world color="green"></hello-world>
```

#### attributeChangedCallback - Chapter4/00.2

setAttribute로 속성을 변경해도 화면에는 아무일도 일어나지 않는다. HTMLElement의 attribute 값은 변경되었지만 DOM이 아직 업데이트 되지 않은 탓이다.

또 다른 메소드인 `attributeChangeCallback`를 사용하여 속성이 변경될 때 DOM을 업데이트 되도록 해주어야 한다. 이 메서드는 `observedAttributes` 배열에 나열된 attribute 만 트리거한다.

**HelloWorld.js**
```javascript
export default class HelloWorld extends HTMLElement {
  static get observedAttributes () {
    return ['color']
  }

  // ...

  attributeChangedCallback (name, oldValue, newValue) {
    if (!this.div) {
      return
    }

    if (name === 'color') {
      this.div.style.color = newValue
    }
  }

connectedCallback () {
  window.requestAnimationFrame(() => {
    this.div = document.createElement('div')
    this.div.textContent = 'Hello World!'
    this.div.style.color = this.color
    this.appendChild(this.div)
  })
}
```
- name: 변경된 속성의 이름
- oldValue: 속성의 이전 값
- newValue: 속성의 새로운 값

#### Virtual DOM 통합 - Chapter4/00.3

**HelloWorld.js**
```javascript
const createDomElement = color => {
  const div = document.createElement('div')
  div.textContent = 'Hello World!'
  div.style.color = color
  return div
}

export default class HelloWorld extends HTMLElement {
  // ...

  attributeChangedCallback (name, oldValue, newValue) {
    if (!this.hasChildNodes()) {
      return
    }

    applyDiff(
      this,
      this.firstElementChild,
      createDomElement(newValue)
    )
  }

  // ...
}
```

2장에서 구현했던 applyDiff 알고리즘을 사용하여 attribute 변경시 DOM을 업데이트할 수 있다.

#### 사용자 정의 이벤트 - Chapter4/00.4

**index.html**
```html
<github-avatar user="francesco-strazzullo"></github-avatar>
```

**GithubAvatar.js**
```javascript
export default class GitHubAvatar extends HTMLElement {
  constructor () {
    super()
    this.url = LOADING_IMAGE
  }

  // ...

  render () {
    window.requestAnimationFrame(() => {
      this.innerHTML = ''
      const img = document.createElement('img')
      img.src = this.url
      this.appendChild(img)
    })
  }

  async loadNewAvatar () {
    const { user } = this
    if (!user) {
      return
    }
    try {
      this.url = await getGitHubAvatarUrl(user)
    } catch (e) {
      this.url = ERROR_IMAGE
    }

    this.render()
  }

  connectedCallback () {
    this.render()
    this.loadNewAvatar()
  }
}
```

constructor를 통해 Loading Image를 먼저 표시한 후 API를 이용해 user attribute에 맞는 유저의 이미지를 불러와 렌더링한다.

하지만 component 외부에서 이 API 요청 결과에 반응할 수 있어야 한다. (표준 DOM Element와 동일하게 동작해야 한다.) 

이는 3장에서 사용했던 사용자 정의 이벤트를 통해 구현할 수 있다.

**Chapter 4/00.5 > GithubAvatar.js**
```javascript
export const EVENTS = {
  AVATAR_LOAD_COMPLETE,
  AVATAR_LOAD_ERROR
}

export default class GitHubAvatar extends HTMLElement {
  // ...

  onLoadAvatarComplete () {
    const event = new CustomEvent(AVATAR_LOAD_COMPLETE, {
      detail: {
        avatar: this.url
      }
    })

    this.dispatchEvent(event)
  }

  onLoadAvatarError (error) {
    const event = new CustomEvent(AVATAR_LOAD_ERROR, {
      detail: {
        error
      }
    })

    this.dispatchEvent(event)
  }

  async loadNewAvatar () {
    const { user } = this
    if (!user) {
      return
    }
    try {
      this.url = await getGitHubAvatarUrl(user)
      this.onLoadAvatarComplete()
    } catch (e) {
      this.url = ERROR_IMAGE
      this.onLoadAvatarError(e)
    }

    this.render()
  }

  // ...
}
```

**Chapter 4/00.5 > index.js**
```javascript
avatar
  .addEventListener(
    EVENTS.AVATAR_LOAD_COMPLETE,
    e => { console.log('Avatar Loaded', e.detail.avatar) }
  )
```

외부에서 다음과 같이 이벤트 핸들러를 연결할 수 있다.

## Todo 앱에 사용 - Chapter 4/01

**Application.js**
```javascript
export default class App extends HTMLElement {
  constructor () {
    super()
    this.state = {
      todos: [],
      filter: 'All'
    }

    this.template = document
      .getElementById('todo-app')
  }

  deleteItem (index) {
    this.state.todos.splice(index, 1)
    this.syncAttributes()
  }

  addItem (text) {
    this.state.todos.push({
      text,
      completed: false
    })
    this.syncAttributes()
  }

  syncAttributes () {
    this.list.todos = this.state.todos
    this.footer.todos = this.state.todos
    this.footer.filter = this.state.filter
  }

  connectedCallback () {
    window.requestAnimationFrame(() => {
      // ...

      this.list = this.querySelector('todomvc-list')
      this.list.addEventListener(
        EVENTS.DELETE_ITEM,
        e => {
          this.deleteItem(e.detail.index)
        }
      )

      this.syncAttributes()
    })
  }

  // ...
}
```
App 컴포넌트의 state로 todos를 등록하고 add, delete 이벤트 발생에 따라 list, footer component의 attribute와 sync를 맞추어 준다.

**List.js**
```javascript
export default class List extends HTMLElement {
  static get observedAttributes () {
    return [
      'todos'
    ]
  }

  onDeleteClick (index) {
    const event = new CustomEvent(
      EVENTS.DELETE_ITEM,
      {
        detail: {
          index
        }
      }
    )

    this.dispatchEvent(event)
  }

  updateList () {
    this.list.innerHTML = ''

    this.todos
      .map((t,i)=>this.getTodoElement(t,i))
      .forEach(element => {
        this.list.appendChild(element)
      })
  }

  connectedCallback () {
    // ...

    this.list.addEventListener('click', e => {
      if (e.target.matches('button.destroy')) {
        this.onDeleteClick(e.target.dataset.index)
      }
    })

    this.updateList()
  }

  attributeChangedCallback () {
    this.updateList()
  }
}
```

List 컴포넌트에서 delete 버튼 클릭을 감지하여 `EVENTS.DELETE_ITEM` 이벤트를 발생시킨다.  

이 이벤트는 App 컴포넌트에서 감지되어 todos state 변화를 일으키며, 이는 List 컴포넌트의 attribute로 전달된다.

변경된 attribute는 `attributeChangedCallback` 함수 호출을 발생시켜 최종적으로 리스트가 re-rendering 된다.

## 웹 구성 요소와 렌더링 함수

ch2, ch3에서 분석한 렌더링 함수 접근 방식과 비교해 보자.

#### 코드스타일

- 렌더링 함수는 함수형으로 작성할 수 있다.
- 웹 구성 요소는 HTMLElement를 확장해야 하므로 클래스 작업이 필요하다.

#### 테스트 가능성

- 렌더링 함수를 테스트 하려면 jest와 같은 JSDOM이 통합된 test runner가 있으면 된다.
- Custom Element는 아직까지 JSDOM에서 지원되지 않는다. Puppeteer와 같은 도구를 사용해야 하지만 복잡할 수 있다.

#### 휴대성
웹 구성 요소는 휴대성이 좋아야 한다. 기존 DOM 요소와 동일하게 동작시킬 수 있으므로 다른 어플리케이션에서 가져다 사용하기 좋다.

#### 커뮤니티
Component Class는 대부분의 프레임워크에서 DOM UI를 작성하는 표준 방법이다.