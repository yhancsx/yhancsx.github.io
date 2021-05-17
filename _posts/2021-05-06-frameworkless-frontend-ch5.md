---
title: "Frameworkless Frontend Development - 5. HTTP 요청"
category: web
tags:
  - javascript
  - web
  - front-end
  - study
  - frameworkless
  - vanillajs
  - fetch
  - XMLHttpRequest
  - axios
  - request
  - response
  - HTTP
  - Rest API
---

*Francesco Strazzullo*님의 [프레임워크 없는 프론트 엔드 개발](https://book.naver.com/bookdb/book_detail.nhn?bid=17758920)(원제 Frameworkless Frontend Development)을 읽고 요악하는 포스트입니다.

## AJAX의 탄생

199년 이전에는 서버에서 데이터를 가져와야 할때 마다 전체 페이지를 다시 로드해야 했다. 1999년 아웃룩, 지메일과 구글 지도 같은 어플리케이션들은 페이지를 다시 로드하지 않고 필요한 데이터만 서버에서 로드하는 새로운 기술을 사용하기 시작했다. 이 기술은 Asynchronous Javascript And XML (AJAX)로 명명되었다.

AJAX의 핵심은 XMLHttpRequest 객체이다. 이 객체를 사용하여 HTTP 요청으로 서버에서 데이터를 가져올 수 있다. 2006년에 이객체의 표준 규격 초안이 작성되었다. AJAX의 X는 XML을 나타내지만 현재는 XML 대신 좀더 친숙한 JSON 형식이 사용된다.

## Todo 앱 Rest 서버
```javascript
let todos = []

app.get('/api/todos', (req, res) => {
  res.send(todos)
})

app.post('/api/todos', (req, res) => {
  // ...

  res.status(201)
  res.send(newTodo)
})

app.patch('/api/todos/:id', (req, res) => {
  // ...

  res.send(newTodo)
})

app.delete('/api/todos/:id', (req, res) => {
  // ...

  res.status(204)
  res.send()
})
```
Express로 아주 간단한 Rest 서버를 구축하였다.

## 코드 예제

XMLHttpRequest, Fetch, axios 세가지 기술을 사용해 HTTP 어플리케이션을 작성하며 장단점을 분석해본다.

#### 기본 구조 

**index.js**
```javascript
const onListClick = async () => {
  const result = await todos.list()
  printResult('list todos', result)
}

const onAddClick = async () => {
  const result = await todos.create('A simple todo Element')
  printResult('add todo', result)
}

const onUpdateClick = async () => {
  const list = await todos.list()

  const { id } = list[0]
  const newTodo = {
    id,
    completed: true
  }

  const result = await todos.update(newTodo)
  printResult('update todo', result)
}

const onDeleteClick = async () => {
  const list = await todos.list()
  const { id } = list[0]

  const result = await todos.delete(id)
  printResult('delete todo', result)
}
```

HTTP 클라이언트를 사용하는 부분을 todos 모델 객체로 한단계 래핑했다. 역할 별로 구현이 잘 나누어져 있어 읽기 쉽고 todos 객체를 mocking하여 테스트 하기가 쉽다. 

**todos.js**
```javascript
const list = () => http.get(BASE_URL)

const create = text => {
  const todo = {
    text,
    completed: false
  }

  return http.post(
    BASE_URL,
    todo,
    HEADERS
  )
}

const update = newTodo => {
  const url = `${BASE_URL}/${newTodo.id}`
  return http.patch(
    url,
    newTodo,
    HEADERS
  )
}

const deleteTodo = id => {
  const url = `${BASE_URL}/${id}`
  return http.delete(
    url,
    HEADERS
  )
}
```

HTTP 클라이언트를 객체 형태로 정의하고 각 메소드별로 필요한 파라미터를 주입받고 있다.

이제 HTTP 클라이언트의 실제 구현을 살펴보자.

#### XMLHttpRequest - Chapter5/00

XMLHttpRequest 객체는 W3C가 비동기 HTTP 요청의 표준 방법을 정의한 첫번째 시도였다.

**http.js**
```javascript
// ...

const request = params => {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest()

    const {
      method = 'GET',
      url,
      headers = {},
      body
    } = params

    xhr.open(method, url)

    setHeaders(xhr, headers)

    xhr.send(JSON.stringify(body))

    xhr.onerror = () => {
      reject(new Error('HTTP Error'))
    }

    xhr.ontimeout = () => {
      reject(new Error('Timeout Error'))
    }

    xhr.onload = () => resolve(parseResponse(xhr))
  })
}

const get = async (url, headers) => {
  const response = await request({
    url,
    headers,
    method: 'GET'
  })

  return response.data
}

const post = async (url, body, headers) => {
  const response = await request({
    url,
    headers,
    method: 'POST',
    body
  })
  return response.data
}

const put = async (url, body, headers) => {
  const response = await request({
    url,
    headers,
    method: 'PUT',
    body
  })
  return response.data
}

const patch = async (url, body, headers) => {
  const response = await request({
    url,
    headers,
    method: 'PATCH',
    body
  })
  return response.data
}

const deleteRequest = async (url, headers) => {
  const response = await request({
    url,
    headers,
    method: 'DELETE'
  })
  return response.data
}

// ...
```

핵심은 `request` 메서드이다. `xhr 생성 -> 초기화 -> 설정(헤더) -> 전송 -> 대기 -> 콜백` 의 흐름을 가지며 `onload`, `onerror`, `ontimeout`등의 콜백들로 요청에 대한 응답 처리를 한다.

`get`, `post`, `put`, `patch`, `delete` 메서드는 코드를 더 읽기쉽게 `request` 메서드를 래핑하여 Promise 객체를 리턴한다.

#### Fetch - Chapter5/01

Fetch는 원격 리소스에 접근하고자 만들어진 새로운 API이다. Request, Response와 같은 네트워크 객체들에 대한 표준 정의를 제공하는 것이 목적이었다.

이런 방식으로 ServiceWorker와 Cache 같은 다른 API와 상호작용할 수 있다.

**http.js**
```javascript
const parseResponse = async response => {
  const { status } = response
  let data
  if (status !== 204) {
    data = await response.json()
  }

  return {
    status,
    data
  }
}

const request = async params => {
  const {
    method = 'GET',
    url,
    headers = {},
    body
  } = params

  const config = {
    method,
    headers: new window.Headers(headers)
  }

  if (body) {
    config.body = JSON.stringify(body)
  }

  const response = await window.fetch(url, config)

  return parseResponse(response)
}

const get = async (url, headers) => {
  const response = await request({
    url,
    headers,
    method: 'GET'
  })

  return response.data
}

const post = async (url, body, headers) => {
  const response = await request({
    url,
    headers,
    method: 'POST',
    body
  })
  return response.data
}

// ...
```

XMLHttpRequest 와 동일한 API를 가지도록 작성하였다.

window 객체에 포함된 `fetch` 메서드를 사용하면 된다. 이는 Promise 객체를 반환하기 때문에 훨씬 더 읽기 쉽다. 

Promise 형태인 response 객체를 데이터 형태에 따라 `text()`, `blob()`, `json()` 같은 메서드로 resolve 하면된다.


#### Axios - Chapter5/02

axios 오픈소스 라이브러리이다. 다른 방식과의 큰 차이는 브라우저와 Node.js에서 바로 사용할 수 있다는 것이다.

```javascript
const request = async params => {
  const {
    method = 'GET',
    url,
    headers = {},
    body
  } = params

  const config = {
    url,
    method,
    headers,
    data: body
  }

  return axios(config)
}

const get = async (url, headers) => {
  const response = await request({
    url,
    headers,
    method: 'GET'
  })

  return response.data
}

const post = async (url, body, headers) => {
  const response = await request({
    url,
    headers,
    method: 'POST',
    body
  })
  return response.data
}

// ...
```

axios API는 Promise를 기반으로 하고 있어 Fetch API와 유사하다. `request` 메서드도 훨씬 간략해졌다.

#### 아키텍처 검토

![http_client_interface]({{site.url}}{{site.baseurl}}/assets/images/frameworkless_frontend/
http_client_interface.jpeg){: width="300 height="300" }  

서로 다른 HTTP 클라이언트 들을 같은 API를 가지도록 구현하였다. 최소한의 노력으로 HTTP에 사용되는 라이브러리를 변경할 수 있다.

> 구현이 아닌 인터페이스로 프로그래밍 하라. - 갱 오브 포 [[GoF의 디자인 패턴](http://www.yes24.com/Product/Goods/17525598)]


## 적합한 HTTP API를 선택하는 방법

`딱 맞는` 프레임워크랑 존재하지 않는다. `적합한` 프레임워크를 선택할 수 있지만 이는 `적합한` 컨텍스트에서만 유효하다.

#### 호환성

Fetch API는 최신브라우저에서만 동작한다. axios는 IE11 미만 버전은 지원하지 않는다.

#### 휴대성

Fetch API와 XMLHttpRequest는 브라우저 환경에서만 도작한다. Node.js나 리액트 네이티브 같은 다른 환경에서는 axios가 `적합한` 솔루션이다.

#### 발전성

Fetch API의 가장 중요한 기능중 하나는 Request, Response 같은 네트워크 관련 객체의 표준 정의를 제공한다. ServiceWorker, Cache API와 연동할 필요가 있는 경우 적합하다.

#### 보안

axios는 cross-site request나 XSRF에 대한 보호 시스템이 내장되어 있다. XMLHttpRequest, Fetch API는 이런 종류의 보안 시스템을 구현해야 한다면 axios의 [단위 테스트](https://github.com/axios/axios/blob/master/test/specs/xsrf.spec.js)를 살펴보는 것을 추천한다. 

#### 학습 곡선

axios나 Fetch API는 초보자가 의미를 이해하기 더 쉬울 것이다. XMLHttpRequest는 콜백 작업이 익숙하지 않은 개발자에게 이상하게 보일 수 있다.


