---
title: "[번역] Demystifying SSR, CSR, universal and static rendering with animations"
category: web  
tags:
  - web
  - front-end
  - translate
  - server side rendering
  - client side rendering
  - performance
---

Franck Abgrall 님의 [Demystifying SSR, CSR, universal and static rendering with animations](https://dev.to/kefranabg/demystifying-ssr-csr-universal-and-static-rendering-with-animations-m7d?fbclid=IwAR31N68HLXa8lDnC3iOa7wsuQ4cDZBlKaUOgG_Fk7DovME2VYlG3ABtcczM
) 영상을 정리한 글 입니다.

## 1. Full Server Side Rendering
- SPA 가 대두되기 전 주로 사용. (ex. django + template language)

![fss1]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/full_server_side_rendering_1.PNG){: width="400 height="400"}
![fss2]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/full_server_side_rendering_2.PNG){: width="400 height="400"}  
- 브라우저 요청시 서버에서 데이터 fetch 후 html 파일 완성하여 response.
- 새로운 페이지 넘어가더라도 그에 해당하는 html 파일 만들기 위해 위 작업 반복.

## 2. Client Side Rendering
- React, Angular, Vue

![csr1]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/client_side_rendering_1.PNG){: width="400 height="400"}
![csr2]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/client_side_rendering_2.PNG){: width="400 height="400"}
- 브라우저 요청시 script, style tag 포함된 빈 index.html 반환.
- 브라우서에서 다시 script, style 태그 파싱.
- 그동안 브라우저는 빈 화면.

![csr3]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/client_side_rendering_3.PNG){: width="400 height="400"}
![csr4]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/client_side_rendering_4.PNG){: width="400 height="400"}
- 브라우저에서 직접 데이터 파싱하여 렌더링.
- 페이지 전환시 새 페이지 레이아웃 신속하게 렌더링.
- 새 페이지에서 필요한 데이터 브라우저에서 다시 파싱.

## 3. Universal Rendering
- Next.js, Nuxt.js, Angular Universal

![ur1]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/universal_rendering_1.PNG){: width="400 height="400"}
![ur2]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/universal_rendering_2.PNG){: width="400 height="400"}
- 브라우저 최초 요청시 서버에서 데이터 fetch 후 html로 렌더링하여 response.
- 페이지를 볼수는 있지만 아직 자바스크립트 파일 파싱전이라서 인터렉션은 불가능.
- 자바스크립트 파일 파싱.(= hydration)
- 최초 서버에서 렌더링 될때 생성된 state 동기화 작업 포함.

![ur3]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/universal_rendering_3.PNG){: width="400 height="400"}
![ur4]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/universal_rendering_4.PNG){: width="400 height="400"}
- 자바스크립트 파싱 완료후엔 SPA 앱처럼 작동.
- 브라우저에서 직접 데이터 파싱하여 렌더링.

## 4. Static Rendering
- Gatsby, VuePress, Gridsome, Next.js, Hugo, Jekyll

![ur4]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/static_rendering_1.PNG){: width="400 height="400"}
![ur4]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/static_rendering_2.PNG){: width="400 height="400"}
- 서버에서 모든 페이지 및 데이터에 대해 pre-render
- 이 후엔 서버 및 데이터 api 서버 불필요.

![ur4]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/static_rendering_3.PNG){: width="400 height="400"}
![ur4]({{site.url}}{{site.baseurl}}/assets/images/server_side_rendering/static_rendering_4.PNG){: width="400 height="400"}
- pre-render 된 static 페이지 CDN 서버에 저장하여 호스팅.


## Pros and Cons

|                           | Server Side Rendering | Client Side Rendering | Universal Rendering | Static Rendering |  
| :-------------------------| --------------------- | --------------------- | ------------------- | ---------------: |  
| SEO | O | X | O | O |
| Full page reload when navigating | O | X| X | Tool by Tool |
| Require Server Hosting | O | X | O | X |
| Initial load | Fast | Slow | Fast | Super Fast |
| Seamless user experience on page navigation | Bad | Good | Good| Good |
| Integrates with websites based on frequent/real-time updates| Good | Good | Good | Bad