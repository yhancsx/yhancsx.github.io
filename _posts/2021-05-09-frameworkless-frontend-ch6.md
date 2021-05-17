---
title: "Frameworkless Frontend Development - 6. 라우팅"
category: web
tags:
  - javascript
  - web
  - front-end
  - study
  - frameworkless
  - vanillajs
  - router
  - 
---

*Francesco Strazzullo*님의 [프레임워크 없는 프론트 엔드 개발](https://book.naver.com/bookdb/book_detail.nhn?bid=17758920)(원제 Frameworkless Frontend Development)을 읽고 요악하는 포스트입니다.

## Single Page Application (SPA)

SPA는 하나의 HTML 페이지로 실행되는 웹 어플리케이션이다. 사용자가 다른 페이지로 이동할 때 새로운 HTML 파일을 받아 오지 않고 동적으로 뷰를 다시 그린다. 페이지간 탐색 시 지연을 최소화 해 더 나은 사용자 경험을 제공할 수 있다.

AngularJ, Ember와 같은 프레임워크는 SPA 방식의 웹 어플리케이션 발전에 큰 기여를 했다. 이들은 라우팅 시스템을 통해 경로를 정의할 수 있는 방법을 제공한다.

라우팅 시스템은 최소 두가지 핵심 요소를 가진다.
1. 어플리케이션의 경로(path) 목록을 저장하는 레지스트리. 
   - 가장 간단한 형태로는 URL을 DOM Component에 매칭시킨다.

2. 현재 URL의 listener.
   - URL이 변경되면 DOM을 현재 URL과 매칭되는 Component로 교체한다.

## 코드 예제

라우팅 시스템을 Fragment Identifiers, History API, Navigo 세가지 방법으로 작성해본다.

### Fragment Identifiers (FI)

모든 URL은 Fragment Identifiers라고 불리는 해시(#)로 시작한다. FI의 목적은 웹페이지의 특정 섹션을 식별하는 것이다. `www.example.com/#foo`라면 `id="foo"`인 HTML Element를 식별한다.

FI가 포함된 URL을 탐색할 때 브라우저는 이 Element가 Viewport 맨 위로 오도록 페이지를 스크롤한다.

#### 첫번째 예제 - Chapter6/00

**index.html**
```html
<header>
  <a href="#/">Go To Index</a>
  <a href="#/list">Go To List</a>
  <a href="#/dummy">Dummy Page</a>
</header>
<main>

</main>
```

엥커(a) tag 의 href 속성을 이용하면 URL을 변경할 수 있다.

**pages.js**
```javascript
export default container => {

  const home = () => {
    container
      .textContent = 'This is Home page'
  }

  const list = () => {
    container
      .textContent = 'This is List Page'
  }

  const notFound = () => {
    container
      .textContent = 'Page Not Found!'
  }

  return {
    home,
    list,
    notFound
  }
}
```

component는 전달받는 container의 textContent를 업데이트 하는 간단항 방식으로 작성되었다.

**index.js**
```javascript
import createRouter from './router.js'
import createPages from './pages.js'

const container = document.querySelector('main')

const pages = createPages(container)

const router = createRouter()

router
  .addRoute('#/', pages.home)
  .addRoute('#/list', pages.list)
  .setNotFound(pages.notFound)
  .start()
```

`addRoute`는 path에 따른 component를 연결하고, `setNotFound`는 등록되지 않은 모든 fragment에 대한 default component를 정의한다. `start`는 라우터를 초기화하고 URL 변경을 감지하기 시작한다.

**router.js**
```javascript
export default () => {
  const routes = []
  let notFound = () => {}

  const router = {}

  const checkRoutes = () => {
    const currentRoute = routes.find(route => {
      return route.fragment === window.location.hash
    })

    if (!currentRoute) {
      notFound()
      return
    }

    currentRoute.component()
  }

  router.addRoute = (fragment, component) => {
    routes.push({
      fragment,
      component
    })

    return router
  }

  router.setNotFound = cb => {
    notFound = cb
    return router
  }

  router.start = () => {
    window
      .addEventListener('hashchange', checkRoutes)

    if (!window.location.hash) {
      window.location.hash = '#/'
    }

    checkRoutes()
  }

  return router
}
```

현재 FI는 location 객체의 `hash`에 저장된다. `hashchange` 이벤트를 통해 fragement가 변경될 때마다 리스너를 실행시킬 수 있다.

`checkRoutes` 메서드는 현재 fragment와 일치하는 path를 찾아 발견되면 해당 component로 메인 콘텐츠를 대체한다. 발견되지 않으면 `notFound` component가 호출된다.

#### 프로그래밍 방식으로 라우팅 - Chapter6/00.1

path를 변경하기 위해 a tag를 클릭하는 경우 말고 프로그래밍 방식이 필요할 때도 있다. 예를 들면 로그인에 성공한 사용자를 메인 페이지로 리다이렉션 할 수 있다.

**index.html**
```html
<header>
  <button data-navigate="/">
    Go To Index
  </button>
  <button data-navigate="/list">
    Go To List
  </button>
  <button data-navigate="/dummy">
    Dummy Page
  </button>
</header>
<main>
</main>
```

이를 위해 헤더의 링크를 button tag로 바꾸었다.

**index.js**
```javascript
// ...

document
  .body
  .addEventListener('click', e => {
    const { target } = e
    if (target.matches(NAV_BTN_SELECTOR)) {
      const { navigate } = target.dataset
      router.navigate(navigate)
    }
  })
```

**router.js**
```javascript
// ... 

router.navigate = fragment => {
  window.location.hash = fragment
}

// ...
```

프로그래밍 방식으로 라우팅 되도록 `navigate` 메서드를 추가하였다. fragment를 가져와 location 객체의 hash 속성을 변경한다.

#### Path Parameter - Chapter6/00.2

`http://example.com/order/1` 에서 order 모델의 ID를 얻을 수 있다. 1은 id라는 path parameter이다. 일반적으로 `http://example.com/order/:id` 와 같이 URL에 parameter가 포함돼 있음을 나타낸다.

**pages.js**
```javascript
// ...

const detail = (params) => {
  const { id } = params
  container
    .textContent = `This is Detail Page with Id ${id}`
}

const anotherDetail = (params) => {
  const { id, anotherId } = params
  container
  .textContent = `
    This is Detail Page with Id ${id} 
    and AnotherId ${anotherId}`
}

// ...
```

인수를 사용해 동적으로 렌더링하도록 component를 수정한다. 이 인수는 path parameter로 채워진다.

**index.js**
```javascript
router
  // ...
  .addRoute('#/list/:id', pages.detail)
  .addRoute('#/list/:id/:anotherId', pages.anotherDetail)
```

일반적인 방식과 동일한 형태로 URL과 component를 연결한다. :id, :anotherId 이 컴포넌트로 전달될 path parameter이다.

**router.js**
```javascript
const ROUTE_PARAMETER_REGEXP = /:(\w+)/g
const URL_FRAGMENT_REGEXP = '([^\\/]+)'

// ...

router.addRoute = (fragment, component) => {
  const params = []

  const parsedFragment = fragment
    .replace(
      ROUTE_PARAMETER_REGEXP,
      (match, paramName) => {
        params.push(paramName)
        return URL_FRAGMENT_REGEXP
      })
    .replace(/\//g, '\\/')

  console.log(`^${parsedFragment}$`)

  routes.push({
    testRegExp: new RegExp(`^${parsedFragment}$`),
    component,
    params
  })

  return router
}

// ...
```

현재 URL에서 id, anotherId와 같은 path parameter를 추출해내기 위해 정규표현식을 사용한다.

첫번째 `replace` 메서드는 정규식 `:(\w+)`을 사용하여 fragment에서 매개변수 이름을(id, anotherId) 추출하고 (`params`) 새로운 정규식(`([^\\/]+)`)으로 치환한다. 두번째 메서드는 `/` 기호를 매칭하기 위해 `\`기호를 삽입해 준다. 마지막으로 `^`과 `$` 사이에 새 fragment를 끼워 넣는다.


> `#/list/:id/:anotherId` -> `^#\/list\/([^\\/]+)\/([^\\/]+)$`

현재 URL에서 path parameter를 추출해내는 정규식이 완성되었다.

**router.js**
```javascript
const extractUrlParams = (route, windowHash) => {
  const params = {}

  if (route.params.length === 0) {
    return params
  }

  const matches = windowHash
    .match(route.testRegExp)

  matches.shift()

  matches.forEach((paramValue, index) => {
    const paramName = route.params[index]
    params[paramName] = paramValue
  })

  return params
}

// ...

const checkRoutes = () => {
  const { hash } = window.location

  const currentRoute = routes.find(route => {
    const { testRegExp } = route
    return testRegExp.test(hash)
  })

  if (!currentRoute) {
    notFound()
    return
  }

  const urlParams = extractUrlParams(
    currentRoute,
    window.location.hash
  )

  currentRoute.component(urlParams)
}
```

`extractUrlParams` 함수에서는 현재 windowHash에 위에서 만든 정규식을 매칭시키고 결과물을 파싱하여 path paramter를 구한다.

String 객체의 `match` 메서드는 리턴값의 첫번째 요소로 일치하는 전체 문자열을 반환한다. 우리는 캡처된 그룹 정보만 필요하므로 `shift()` 메소드로 버린 후 나머지만 사용한다.

path parameter를 관리할 때 발생하는 상황을 요약하자면

1. `#/list/:id/:anotherId` fragment가 `addRoute` 메서드로 전달된다.
2. `addRoute` 메서드는 path parameter 이름을 추출하고 새 정규식으로 fragment를 변환한다.
3. 사용자가 `#/list/1/2` 같은 URL로 탐색할때 `checkRoutes` 메서드는 정규식을 사용해 올바른 path와 component를 선택한다.
4. `extractUrlParam` 메서드는 현재 URL에서 실제 매개변수 `{id: 1, anotherId:2}` 를 추출한다.
5. Component에서 이 매개변수를 받아 DOM을 업데이트 한다.
  
### History API - Chapter6/01.1

History API 를 통해 사용자 탐색 히스토리를 조작할 수 있다.

- `back()`: 이전 페이지로 이동한다.
- `forward()`: 다음 페이지로 이동한다.
- `go(index)`: history에서 특정 index의 페이지로 이동한다.
- `pushState(state, title, URL)`: history stack에 데이터를 푸시하고 URL로 이동힌다.
- `replaceState(state, title, URL)`: history stack에서 가장 최근 데이터를 바꾸고 URL로 이동한다.

History API를 사용하는 경우엔 Fragment Identifiers를 기반으로 경로를 지정할필요가 없다. `www.example.com//list/1/2` 같은 실제 URL을 활용한다.

**router.js**
```javascript
const checkRoutes = () => {
  const { pathname } = window.location
  if (lastPathname === pathname) {
    return
  }

  lastPathname = pathname

  const currentRoute = routes.find(route => {
    const { testRegExp } = route
    return testRegExp.test(pathname)
  })

  if (!currentRoute) {
    notFound()
    return
  }

  const urlParams = extractUrlParams(currentRoute, pathname)

  currentRoute.callback(urlParams)
}

// ...

router.navigate = path => {
  window.history.pushState(null, null, path)
}

router.start = () => {
  checkRoutes()
  window.setInterval(checkRoutes, TICKTIME)

  document
    .body
    .addEventListener('click', e => {
      const { target } = e
      if (target.matches(NAV_A_SELECTOR)) { // const NAV_A_SELECTOR = 'a[data-navigation]'
        e.preventDefault()
        router.navigate(target.href)
      }
    })

  return router
}
```
**index.html**
```html
<header>
    <a data-navigation href="/">Go To Index</a>
    <a data-navigation href="/list">Go To List</a>
    <a data-navigation href="/list/1">Go To Detail With Id 1</a>
    <a data-navigation href="/list/2">Go To Detail With Id 2</a>
    <a data-navigation href="/list/1/2">Go To Another Detail</a>
    <a data-navigation href="/dummy">Dummy Page</a>
</header>
```

window.location.hash 대신 window.location.pathName을 사용하면 된다. `navigate` 메서드는 History API를 사용하도록 변경됐다.

다만, `hashchange`와 달리 path가 변경되었을 때 알림을 받을 수 있는 이벤트가 없어 setInterval을 사용하였다.

a tag에 data-navigation을 넣고 `preventDefault()` 메서드로 다른 페이지(`/list/index.html` 등)가 열리는 default 동작을 막아 주었다.

### Navigo - Chapter6/02

마지막 구현은 간단한 오폰소스 라이브러리 [Navigo](https://github.com/krasimir/navigo)를 이용한다.

다른 방법들과 동일한 API를 유지하면서 라우터 자체의 내부 코드만 변경했다.

**router.js**
```javascript
// ...
const navigoRouter = new window.Navigo()

router.addRoute = (path, callback) => {
  navigoRouter.on(path, callback)
  return router
}

router.setNotFound = cb => {
  navigoRouter.notFound(cb)
  return router
}

router.navigate = path => {
  navigoRouter.navigate(path)
}

router.start = () => {
  navigoRouter.resolve()
  return router
}
// ...
```


## 올바른 라우터를 선택하는 방법

세가지 구현 간에 의미있는 차이는 없다. History API는 IE9 이하에서 지원되지 않지만 큰 문제는 아니다. 프레임워크 없이 시작해서 복잡한 기능이 필요할 경우에만 3rd party 라이브러리를 도입하는 것을 추천한다.

라우팅은 SPA에 있어서 신경계와 같다. 프로젝트에 react-router를 사용하는 경우 SPA 라우팅 시스템을 변경하기 어려워 react를 제거하기가 매우 어렵다. 하지만 라우팅 시스템이 독립적이라면 framework를 변경하기 쉽다.