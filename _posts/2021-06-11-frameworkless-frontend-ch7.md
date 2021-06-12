---
title: "Frameworkless Frontend Development - 7. 상태 관리"
category: web
tags:
  - javascript
  - web
  - front-end
  - study
  - frameworkless
  - vanillajs
  - state management
  - MVC
  - Reactive Programming
  - RxJS
  - Flux
  - Redux
  - store
  - Event Bus
---

*Francesco Strazzullo*님의 [프레임워크 없는 프론트 엔드 개발](https://book.naver.com/bookdb/book_detail.nhn?bid=17758920)(원제 Frameworkless Frontend Development)을 읽고 요악하는 포스트입니다.

웹 뿐만아니라 모든 종류의 클라이언트 앱에서 구성 요소들을 연결하는 데이터 관리 방법을 상태 관리(state management)라고 한다. MVC 패턴은 1970년대에 등장했지만 React가 떠오르며 상태 관리라는 용어가 등장하기 시작했다. 오늘날엔 Vuex, NgRx, MobX, Redux 같은 프론트엔드 전용 상태 관리 라이브러리들이 있다.

3가지 상태 관리 전략을 구축하고 장단점을 비교해 보자.

## 1. MVC 패턴 - Chapter7/00

상태 관리 라이브러리의 첫번째 버전은 MVC (model-view-controller) 패턴이다.

![mvc_pattern]({{site.url}}{{site.baseurl}}/assets/images/frameworkless_frontend/mvc_pattern.png)

1. Controller는 Model에서 초기 상태를 가져온다.
2. Controller는 View를 호출해 초기 상태를 렌더링한다.
3. 사용자가 어떤 동작을 수행한다.
4. Controller는 올바른 메서드(`model.addItem`)와 사용자의 동작을 매핑한다.
5. Model이 상태를 업데이트 한다.
6. Controller는 Model에서 새로운 상태를 얻는다.
7. Controller는 View를 호출해 새로운 상태를 렌더링한다. 

**index.js**
```javascript
// ...
const model = modelFactory()

const events = {
  addItem: text => {
    model.addItem(text)
    render(model.getState())
  },
  updateItem: (index, text) => {
    model.updateItem(index, text)
    render(model.getState())
  },
  deleteItem: (index) => {
    model.deleteItem(index)
    render(model.getState())
  },
  toggleItemCompleted: (index) => {
    model.toggleItemCompleted(index)
    render(model.getState())
  },
  completeAll: () => {
    model.completeAll()
    render(model.getState())
  },
  clearCompleted: () => {
    model.clearCompleted()
    render(model.getState())
  },
  changeFilter: filter => {
    model.changeFilter(filter)
    render(model.getState())
  }
}

const render = (state) => {
  window.requestAnimationFrame(() => {
    const main = document.querySelector('#root')

    const newMain = registry.renderRoot(
      main,
      state,
      events)

    applyDiff(document.body, main, newMain)
  })
}
//...
```

상태 관리 로직은 컨트롤러인 `events` 객체에 있으며, 이 객체는 메서드를 DOM 핸들러에 연결하기 위해 View 함수에 전달된다. `events` 객체 내에서는 외부의 `model` 객체 API 사용하여 상태를 관리한다.

렌더링에 사용할 실제 데이터는 `model` 객체의 `getState` 메서드에서 반환된다.

**model.js**
```javascript
const cloneDeep = x => {
  return JSON.parse(JSON.stringify(x))
}

const INITIAL_STATE = {
  todos: [],
  currentFilter: 'All'
}

export default (initalState = INITIAL_STATE) => {
  const state = cloneDeep(initalState)

  const getState = () => {
    return Object.freeze(cloneDeep(state))
  }

  const addItem = text => {
    if (!text) {
      return
    }

    state.todos.push({
      text,
      completed: false
    })
  }

  const updateItem = (index, text) => {
    if (!text) {
      return
    }

    if (index < 0) {
      return
    }

    if (!state.todos[index]) {
      return
    }

    state.todos[index].text = text
  }

  // ...
}
```

`model` 객체에서 추출한 값은 불변(immutable)이다. `getState` 메서드가 호출될 때 마다 복사본을 생성(`cloneDeep`)하고 값을 고정(`freeze`)한다.

데이터가 불변 형태라면 사용자는 공개 API를 통해 데이터를 조작할 수 밖에 없다. 이런 방법으로 비지니스 로직이 Model 객체에 완전히 포함되 있으면 관리하고 테스트하기 용이하다.

#### Observable Model - Chapter7/01

MVC 패턴은 일반적으로 잘 작동하지만 다음과 같은 문제점이 있다.
1. 상태 변경 후에 렌더링을 수동으로 호출하는 방법은 오류가 발생하기 쉽다.
2. 동작이 상태를 변경하지 않을 때(ex. 빈 항목을 리스트에 추가할 때)에도 render 메서드가 호출된다.

이는 [obeserver pattern](https://en.wikipedia.org/wiki/Observer_pattern)을 기반으로하는 이번 버전에서 해결된다.

**model.js**
```javascript
export default (initalState = INITIAL_STATE) => {
  const state = cloneDeep(initalState)
  let listeners = []

  const addChangeListener = listener => {
    listeners.push(listener)

    listener(freeze(state))

    return () => {
      listeners = listeners.filter(l => l !== listener)
    }
  }

  const invokeListeners = () => {
    const data = freeze(state)
    listeners.forEach(l => l(data))
  }

  const addItem = text => {
    if (!text) {
      return
    }

    state.todos.push({
      text,
      completed: false
    })

    invokeListeners()
  }

  // ...
}
```

**index.js**
```javascript
// ...

const {
  addChangeListener,
  ...events
} = model

const render = (state) => {
  window.requestAnimationFrame(() => {
    const main = document.querySelector('#root')

    const newMain = registry.renderRoot(
      main,
      state,
      events)

    applyDiff(document.body, main, newMain)
  })
}

addChangeListener(render)
```

Controller 부분의 로직이 단순화 되었다.

`model` 객체로 `render` 함수를 삽입(`addChangeListener`)하여 `model` 내 메서드가 호출될 때마다 re-render(`invokeListeners`) 되도록 한다.

이 방식은 Model의 공개 API를 수정하지 않고 Controller에 새 기능을 추가하기 유용하다.

**Chapter7/01.1 > index.js**
```javascript
addChangeListener(state => {
  Promise.resolve().then(() => {
    window
      .localStorage
      .setItem('state', JSON.stringify(state))
  })
})

addChangeListener(state => {
  console.log(
    `Current State (${(new Date()).getTime()})`,
    state
  )
})
```

이와 같이 기존 상태 관리 외 다양한 리스너를 연결할 수 있다.

옵저버블 모델 없이 동일한 기능을 구현하기는 어려울 뿐만 아니라 유지 관리하기도 힘들다. Controller가 Model과 밀접하게 결합된다면 이 패턴을 고려하는 것이 좋다.

## 2. Reactive Programming (RP)

Reactive Programming(반응형 프로그래밍)은 frontend 커뮤니티에서 꽤 오랫동안 사용된 용어다. RP는 앵귤러 프레임워크가 RxJS를 기반오르 하고 있다고 발표하면서 인기를 끌기 시작했다. 

Reactive Programming이란 어플리케이션이 Model 변경, HTTP 요청, 사용자 동작, 탐색 등과 같은 이벤트를 방출할 수 있는 Observable로 동작하도록 구현하는 것을 의미한다.

> 자신의 코드에서 observable을 사용하고 있다면 이미 반응형 패러다임으로 작업하고 있는 것이다.

몇가지 다른 방식으로 반응형 상태 관리 라이브러리를 작성해 본다.

#### Reactive Model - Chapter7/02

위에서 반응형 상태 관리의 간단한 버전을 작성했었다. 하지만 다양한 Model 객체를 갖고 있는 복잡한 어플리케이션에서는 Observable을 생성할 수 있는 쉬운 방법이 필요하다.

**model.js**
```javascript
export default (initalState = INITIAL_STATE) => {
  const state = cloneDeep(initalState)

  // ...

  const model = {
    addItem,
    updateItem,
    deleteItem,
    toggleItemCompleted,
    completeAll,
    clearCompleted,
    changeFilter
  }

  return observableFactory(model, () => state)
}
```
**observable.js**
```javascript
export default (model, stateGetter) => {
  let listeners = []

  const addChangeListener = cb => {
    listeners.push(cb)
    cb(freeze(stateGetter()))
    return () => {
      listeners = listeners
        .filter(element => element !== cb)
    }
  }

  const invokeListeners = () => {
    const data = freeze(stateGetter())
    listeners.forEach(l => l(data))
  }

  const wrapAction = originalAction => {
    return (...args) => {
      const value = originalAction(...args)
      invokeListeners()
      return value
    }
  }

  const baseProxy = {
    addChangeListener
  }

  return Object
    .keys(model)
    .filter(key => {
      return typeof model[key] === 'function'
    })
    .reduce((proxy, key) => {
      const action = model[key]
      return {
        ...proxy,
        [key]: wrapAction(action)
      }
    }, baseProxy)
}
```

Observable Factory의 기능은 매우 단순하다. `model` 객체를 받아 원본 메서드를 실행하고 리스너를 호출하는 proxy를 생성한다. proxy로 상태를 전달하고자 간단한 getter (`stateGetter`) 함수를 사용해 `model`에서 변경이 수행될 때마다 현재 상태를 가져온다.

외부에서 볼 때 이전에 작성한 Observable Model과 이번에 작성한 Model은 동일한 공개 API를 갖는다. 처음엔 간단한 Observable Model을 생성하고, 둘 이상의 Model 객체가 필요할때 Observable Factory를 작성할 수 있다. 3장에서 소개한 YAGNI 원칙의 또다른 예시이다.

#### Native Proxy - Chapter7/02.2

Javascript는 `Proxy` 객체를 통해 proxy를 생성할 수 있는 방법을 제공한다. 

**Chapter7/02.1 > index.js**
```javascript
const base = {
  foo: 'bar'
}

const handler = {
  get: (target, name) => {
    console.log(`Getting ${name}`)
    return target[name]
  },
  set: (target, name, value) => {
    console.log(`Setting ${name} to ${value}`)
    target[name] = value
    return true
  }
}

const proxy = new Proxy(base, handler)

proxy.foo = 'baz'
console.log(`Logging ${proxy.foo}`)
```

**result**
```shell
Setting foo to baz
Getting foo
Loggin baz
```

기본 객체를 래핑하는 proxy를 생성하려면 `trap` 집합으로 구성된 핸들러가 필요하다. `trap`은 기본 객체의 기본 작업을 래핑하는 방법이다. 위에선 모든 속성의 getter와 setter를 덮어 썼다. setter는 작업 성공을 나타내는 boolean 값을 반환해야 한다.

**observable.js**
```javascript
export default (initialState) => {
  let listeners = []

  const proxy = new Proxy(cloneDeep(initialState), {
    set: (target, name, value) => {
      target[name] = value
      listeners.forEach(l => l(freeze(proxy)))
      return true
    }
  })
  
  // ...

  return proxy
}
```
**model.js**
```javascript
export default (initialState = INITIAL_STATE) => {
  const state = observableFactory(initialState)

  const addItem = text => {
    if (!text) {
      return
    }

    state.todos = [...state.todos, {
      text,
      completed: false
    }]
  }

  const updateItem = (index, text) => {
    if (!text) {
      return
    }

    if (index < 0) {
      return
    }

    if (!state.todos[index]) {
      return
    }

    state.todos = state.todos.map((todo, i) => {
      if (i === index) {
        todo.text = text
      }
      return todo
    })
  }

  // ...
}
```

Proxy 객체를 통해 setter를 래핑했기 때문에 `model` 객체의 state 업데이트 방식도 setter를 이용하도록(todos 배열을 매번 덮어쓰도록) 수정이 필요하다.  

> Proxy 객체를 사용할 때는 객체 속성을 수정하는 대신 교체해야 한다.

## 3. Event Bus (EB)

Event Bus(이벤트 버스)는 Event-Driven Architecture(EDA)를 구현하는 하나의 방법이다. EDA로 작업할 때 모든 상태 변경은 시스템에서 전달된 이벤트로 나타난다.

```javascript
const event = {
  type: 'ITEM_ADDED',
  payload: 'Buy Milk'
}
```
이벤트는 발생한 상황을 식별할 수 있는 type과 의미있는 정보를 담고있는 payload로 구성된다.

![event_bus_pattern]({{site.url}}{{site.baseurl}}/assets/images/frameworkless_frontend/event_bus_pattern.png)

Event Bus 패턴의 기본 개념은 어플리케이션을 구성하는 모든 '노드'들을 연결하는 단일 객체가 모든 이벤트를 처리한다는 것이다. 이벤트가 처리되면 결과가 연결된 모든 노드로 전송된다.

Event Bus 동작 방식을 분석해보자.

1. view는 초기 상태를 렌더링한다.
2. 사용자가 폼을 작성하고 엔터키를 누른다.
3. DOM 이벤트가 View에 의해 캡처된다.
4. View는 ITEM_ADDED 이벤트를 생성하고 버스로 보낸다.
5. Bus는 새로운 상태를 생성하는 이벤트를 처리한다.
6. 새로운 상태가 Controller로 전송된다.
7. Controller가 View를 호출해 새로운 상태를 렌더링한다.
8. 시스템이 사용자 입력을 받을 준비가 됐다.

5번에서 Bus는 아키텍처적 요소이기 때문에 서비스 도메인에 관련된 코드(상태 변경)를 포함해서는 안된다. 따라서 Model을 Event Bus에 믹스(mix)해야 한다.

#### Frameworkless - Chapter7/03

**eventBus.js**
```javascript
export default (model) => {
  let listeners = []
  let state = model()

  const subscribe = listener => {
    listeners.push(listener)

    return () => {
      listeners = listeners.filter(l => l !== listener)
    }
  }

  const invokeSubscribers = () => {
    const data = freeze(state)
    listeners.forEach(l => l(data))
  }

  const dispatch = event => {
    const newState = model(state, event)

    if (!newState) {
      throw new Error('model should always return a value')
    }

    if (newState === state) {
      return
    }

    state = newState

    invokeSubscribers()
  }
  return {
    subscribe,
    dispatch,
    getState: () => freeze(state)
  }
}
```

Event Bus 내에서 `model`은 이전 상태와 이벤트를 받아 새로운 상태를 반환하는 순수 함수이다. 이벤트가 발생할 때(dispatch 함수가 호출 될 때)마다 Subscriber가 실행되지만 state 변경이 없다면 건너뛸 수 있다.

**model.js**
```javascript
const INITIAL_STATE = {
  todos: [],
  currentFilter: 'All'
}

const addItem = (state, event) => {
  const text = event.payload
  if (!text) {
    return state
  }

  return {
    ...state,
    todos: [...state.todos, {
      text,
      completed: false
    }]
  }
}

const updateItem = (state, event) => {
  const { text, index } = event.payload
  if (!text) {
    return state
  }

  if (index < 0) {
    return state
  }

  if (!state.todos[index]) {
    return state
  }

  return {
    ...state,
    todos: state.todos.map((todo, i) => {
      if (i === index) {
        todo.text = text
      }
      return todo
    })
  }
}

// ...

const methods = {
  ITEM_ADDED: addItem,
  ITEM_UPDATED: updateItem,
  // ...
}

export default (initalState = INITIAL_STATE) => {
  return (prevState, event) => {
    if (!prevState) {
      return cloneDeep(initalState)
    }

    const currentModifier = methods[event.type]

    if (!currentModifier) {
      return prevState
    }

    return currentModifier(prevState, event)
  }
}
```

이벤트 타입에 맞는 메서드를 선택하고자 간단한 매핑 객체를 사용했다.

**Chapter7/03.1 > model.js**
```javascript
export default (initalState = INITIAL_STATE) => {
  return (prevState, event) => {
    if (!event) {
      return cloneDeep(initalState)
    }

    const {
      todos,
      currentFilter
    } = prevState

    const newTodos = todosModifers(todos, event)
    const newCurrentFilter = filterModifers(currentFilter, event)

    if (newTodos === todos && newCurrentFilter === currentFilter) {
      return prevState
    }

    return {
      todos: newTodos,
      currentFilter: newCurrentFilter
    }
  }
}
```

실제 어플리케이션에서 모델함수는 여러개의 작은 서브모듈로 분리되어야 한다. 위 예제는 todos, filters 두개의 모듈로 분리된 보습을 보여준다.

**index.js**
```javascript
const render = (state) => {
  window.requestAnimationFrame(() => {
    const main = document.querySelector('#root')

    const newMain = registry.renderRoot(
      main,
      state,
      eventBus.dispatch)

    applyDiff(document.body, main, newMain)
  })
}
```

이전 버젼의 controller와 차이는 view에 이벤트를 제공하지 않는다는 점이다. 대신 event bus의 dispatch 메서드를 전달한다.

**app.js**
```javascript
// ...

const addEvents = (targetElement, dispatch) => {
  targetElement
    .querySelector('.new-todo')
    .addEventListener('keypress', e => {
      if (e.key === 'Enter') {
        const event = eventCreators
          .addItem(e.target.value)
        dispatch(event)
        e.target.value = ''
      }
    })
}

export default (targetElement, state, dispatch) => {
  const newApp = targetElement.cloneNode(true)

  newApp.innerHTML = ''
  newApp.appendChild(getTemplate())

  addEvents(newApp, dispatch)

  return newApp
}
```

**eventCreator.js**
```javascript
export default {
  addItem: text => ({
    type: EVENT_TYPES.ITEM_ADDED,
    payload: text
  }),
  // ...
}
```

view에서는 `eventCreators` 객체를 사용하여 이벤트 객체를 생성한 뒤 disaptch 메서드로 전달한다.

#### Redux - Chapter7/04

Redux는 2015년에 react 유럽 컨퍼런스에서 처음 발표되었다. 페이스북의 Flux 아키텍처와 유사한 라이브러리 중 하나이다.

Redux를 사용하는 방식은 위에서 살펴본 frameworkless 방식과 매우 유사하지만 Flux 패턴 이후에 만들어졌기 때문에 아키텍처의 구성요소를 정의하는 용어가 다르다.

- Event Bus -> Store
- Event -> Action
- Model -> Reducer

**index.js**
```javascript
const INITIAL_STATE = {
  todos: [],
  currentFilter: 'All'
}

const {
  createStore
} = Redux

const store = createStore(
  reducer,
  INITIAL_STATE,
  window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
)

const render = () => {
  window.requestAnimationFrame(() => {
    const main = document.querySelector('#root')

    const newMain = registry.renderRoot(
      main,
      store.getState(),
      store.dispatch)

    applyDiff(document.body, main, newMain)
  })
}
```

Event Bus 대신 Redux의 store를 사용해도 controller 부분은 거의 차이가 없다. reducer는 이전에 구현한 Event Bus의 Model과 코드가 동일하다.

frameworkless Event Bus 대신 Redux를 이용하는 주요 이점 중 하나는 넓은 생테계에서 나오는 많은 도구와 플러그인이다. 대표적으로는 예를 들면 Redux DevTools로 시스템의 모든 작업(action)을 기록하고 state 변경을 확인해 볼 수 있다.

## 상태관리 전략 비교

단순성, 일관성, 확장성 3가지 관점에서 MVC, Reactive Programming, Event Bus 3가지 상태관리 전략을 비교해보자.

#### MVC

구현하기 매우 간단하면서 비지니스 로젝에 대한 테스트 가능성, 관심사의 분리와 같은 이점을 제공한다.

문제는 엄격한 패턴이 아니라는 것이다. 각 요소간의 정의와 관계가 불분명해질 수 있다. MVC 패턴의 회색 영역을 각자 정의하고 채워야 한다. 이로 인한 확장성 문제도 있다. 어플리케이션이 커질수록 회색영역이 증가하고 일관성이 해결되지 않으면 코드를 읽기 어려워진다.

#### Reactive Programming

기본 아이디어는 어플리케이션이 Observable하다는 것이다. 사용자 입력, timer, HTTP 요청 등 모든 front-end 어플리케이션의 요소들을 Observable하게 변환해주는 RxJS 같은 라이브러리도 있다. 이 방식은 동일한 타입의 객체로 작업하기 때문에 뛰어난 일관성을 보장해준다.

하지만 모든 Observable을 래핑하는것이 간단하지 않다. RxJS 같은 서드파티 라이브러리를 사용할 수 있지만 여전히 간단하지는 않다. 

> 쉬운 아키텍처와 단순한 아키텍처는 동일한 의미가 아니다. 우리의 목표는 쉬운 아키텍처가 아니라 요구사항을 충족시키는 단순한 아키텍처의 구현이다.

추상화로 작업하면 [어플리케이션이 누설](https://en.wikipedia.org/wiki/Leaky_abstraction)되기 때문에 규모가 커질수록 문제가 될 수 있다.

#### Event Bus

'모든 상태 변경은 event에 의해 발생한다' 라는 엄격한 규칙을 기반으로 한다. 이 규칙 덕분에 어플리케이션의 복잡성을 어플리케이션의 크기에 비례하도록 유지할 수 있다. 사용하기 쉽고 구축도 쉽다. 또한 패턴 뒤에 숨겨져 있는 추상화가 Reactive Programming 만큼 강하지 않기 때문에 단순하기도 하다.

가장 큰 문제는 다변성이다. event를 생성하고, bus로 event를 발송하고, state를 업데이트한 model을 작성한다음 새 state를 리스너에 전송해야 한다. 이런 과정들을 항상 거쳐야 하기 때문에 개발자는 더 작거나 간단한 도메인 관리를 위해 다른 전략 (MVC)를 함께 사용하게 되고 결과적으로 일관성이 떨어진다.
