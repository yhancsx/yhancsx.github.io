---
title: "Frameworkless Frontend Development - 3. DOM 이벤트 관리"
category: web
tags:
  - javascript
  - web
  - front-end
  - study
  - frameworkless$$
  - vanillajs
  - event
  - event capture
  - event bubbling
  - custom event
  - template
  - event delegation
---

*Francesco Strazzullo*님의 [프레임워크 없는 프론트 엔드 개발](https://book.naver.com/bookdb/book_detail.nhn?bid=17758920)(원제 Frameworkless Frontend Development)을 읽고 요악하는 포스트입니다.

2장에서 만든 렌더링 엔진은 DOM element 대신 문자열 형태로 동작하던 로직이 있었다. 이 방법에는 addEventListener를 붙이기가 쉽지 않다.
불완전한 렌더링 엔진을 사용한 이유는 가독성과 단순성 때문이다. 가장 중요한 기능에 초점을 맞춰 개발하고 새로운 요구가 생기면 이에 따라 아키텍처를 발전시켜 나간다. 

> YAGNI(You aren't gonna need it; 정말 필요하다고 간주할 때까지 기능을 추가하지 마라) - XP (eXtreme Programming) 원칙.  
> 당신이 필요하다고 생각할 때가 아니라 실제로 필요할 때 구현하라 - XP 창시자 Ron Jeffreis.

아무도 유지 관리하지 않는 또 다른 프레임워크를 작성하지 말아야 한다.


## DOM 이벤트 API

#### 핸들러 연결
on* attribute (onclick, onblur, onmouseover)는 한번에 하나의 핸들러만 연결 할 수 있기 때문에 addEventListener를 사용하는 것이 좋다. 

#### Event 객체
![event_interface]({{site.url}}{{site.baseurl}}/assets/images/frameworkless_frontend/event_interface_inherit.png{: width="300 height="300" }

이벤트가 발생했을 때 연결된 핸들러로 Event 객체가 전달된다. 포인터 좌표, 이벤트 타입, 트리거한 target 요소 등의 정보가 들어있다. Event 객체에서 파생된 MouseEvent, InputEvent, KeyboardEvent등이 사용된다.

#### [Event Life Cycle](https://yhancsx.github.io/web/event-bubbling-capture-delegation/)
Capture Phase -> Target Phase -> Bubbling Phase


#### 사용자 정의 이벤트 사용 - Chapter3/00.4

**index.js**
```javascript
input.addEventListener('input', () => {
  const { length } = input.value
  
  if (length === 5) {
    const time = (new Date()).getTime()
    const event = new CustomEvent(EVENT_NAME, {
      detail: {
        time
      }
    })

    input.dispatchEvent(event)
  }
})

input.addEventListener(EVENT_NAME, e => {
  console.log('handling custom event...', e.detail)
})

```
`CustomEvent` 생성자 함수와 `dispatchEvent`를 이용하여 사용자 정의 이벤트를 커스텀하여 사용할 수 있다.


## 렌더링 엔진에 이벤트 추가

#### Template Element - Chapter3/01
기존 문자열로 생성되어 innerHTML를 통해 삽입되던 로직을 대체해야 한다. `createElement`, `appendChild`등을 이용해 필요한 컴포넌트를 만들 수 있지만 읽고 유지하기 힘들다.

[template 태그](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template)를 이용해서 필요한 마크텁을 유지하는 방법이 있다. 이름에서 알 수 있듯이 렌더링 엔진의 스탬프로 사용할 수 있는 보이지 않는 태그이다.

**index.html**
```html
 <template id="todo-item">
    <li>
      <div class="view">
        <input class="toggle" type="checkbox">
        <label></label>
        <button class="destroy"></button>
      </div>
      <input class="edit">
    </li>
</template>
```

**todos.js**
```javascript
const template = document.getElementById('todo-item')

template.content.firstElementChild.cloneNode(true);
```
> The HTMLTemplateElement has a `content` property, which is a read-only `DocumentFragment` containing the DOM subtree which the template represents. Note that directly using the value of the content could lead to unexpected behavior - [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template#attributes)

`content` 속성 안에 렌더링 할 dom tree를 `DocumentFragment` 인스턴스로 담고 있지만 이는 다양한 event 들의 타겟이 되기 적합하지 않다. `firstElementChild` 속성으로 접근하여 `HTMLDivElement` 인스턴스를 복사해 사용하기 권장하고 있다.

#### 기본 이벤트 처리 아키텍처 - Chapter3/01.2
초기상태 -> 렌더링 -> 이벤트 -> 새로운 상태 -> 렌더링

**index.js**
```javascript
const events = {
  deleteItem: (index) => {
    state.todos.splice(index, 1)
    render()
  },
  addItem: text => {
    state.todos.push({
      text,
      completed: false
    })
    render()
  }
}

const render = () => {
  window.requestAnimationFrame(() => {
    const main = document.querySelector('#root')

    const newMain = registry.renderRoot(
      main,
      state,
      events
    )

    applyDiff(document.body, main, newMain)
  })
}
```

events는 상태를 수정하고 새로운 렌더링을 호출하는 간단한 함수로 구현하였다.

**app.js**
```javascript
const addEvents = (targetElement, events) => {
  targetElement
    .querySelector('.new-todo')
    .addEventListener('keypress', e => {
      if (e.key === 'Enter') {
        events.addItem(e.target.value)
        e.target.value = ''
      }
    })
}
```
appView 함수에서 새로운 todo item을 추가하기 위해 핸들러를 연결하였다.

**todos.js**
```javascript
const getTodoElement = (todo, index, events) => {
  const {
    text,
    completed
  } = todo

  const element = createNewTodoNode()

  element.querySelector('input.edit').value = text
  element.querySelector('label').textContent = text

  if (completed) {
    element.classList.add('completed')
    element
      .querySelector('input.toggle')
      .checked = true
  }

  const handler = e => events.deleteItem(index)

  element
    .querySelector('button.destroy')
    .addEventListener('click', handler)

  return element
}

export default (targetElement, { todos }, events) => {
  const newTodoList = targetElement.cloneNode(true)

  newTodoList.innerHTML = ''

  todos
    .map((todo, index) => getTodoElement(todo, index, events))
    .forEach(element => {
      newTodoList.appendChild(element)
    })

  return newTodoList
}
```

todosView 함수에서는 모든 todo item element에 deleteItem 핸들러를 연결하였다.


## Event Delegation (이벤트 위임) - Chapter3/01.4
위 예제와 같이 모든 element에 이벤트를 연결 하는 방법은 이벤트 위임 으로 성능과 메모리 사용성을 개선시킬 수 있다. 

```javascript
export default (targetElement, state, events) => {
  const { todos } = state
  const { deleteItem } = events
  const newTodoList = targetElement.cloneNode(true)

  newTodoList.innerHTML = ''

  todos
    .map((todo, index) => getTodoElement(todo, index))
    .forEach(element => {
      newTodoList.appendChild(element)
    })

  newTodoList.addEventListener('click', e => {
    if (e.target.matches('button.destroy')) {
      deleteItem(e.target.dataset.index)
    }
  })

  return newTodoList
}
```
newTodoList 자체에 하나의 이벤트 핸들러만 연결하였다.

> [Element.matches()](https://developer.mozilla.org/en-US/docs/Web/API/Element/matches): 요소가 `실제` 이벤트 대상인지 확인하는데 사용된다. 