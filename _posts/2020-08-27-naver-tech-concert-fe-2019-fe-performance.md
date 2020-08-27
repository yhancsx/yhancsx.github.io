---
title: "[Naver Tech Concert 2019] Front-end Performance Analyst"
category: web  
tags:
  - javascript
  - web
  - performance
  - study
  - front-end
  - optimization
  - profiling
---

Naver Apollo 팀의 [손찬욱](http://sculove.github.io/blog/)님이 2019년 Naver Tech Concert Front-end 에서 발표하신 [`오늘부터 나도 FE 성능 분석가'](https://www.youtube.com/watch?v=cpE1dwJgS4c) 발표를 정리한 글입니다.

## Preliminary Questions.
- 성능개선은 빠르면 빠를수록 좋다.
- 로딩속도는 수치적으로 빠르면 빠를수록 좋다.
- 특정 부분의 성능을 개선하면 개선한 만큼 성능 향상이 된다.
- 서비스 성능이슈는 개발자의 무지에서 나온다.
- 성능 전문가는 서비스의 성능개선 포인트를 찾고 개선할 수 있다.
- 서비스 성능 개선은 전문성을 갖춘 영역이기에 아무나 할 수 없다.

## Intro. FE 에서 성능이란?
- 서버: 얼마나 많은 요청을 처리할수 있는가. TPS (transition per second)
- 클라이언트: 사용자가 입력한것을 얼마나 빠르게 반응 할 수 있느냐. **LAI (Loading and Interaction)**
    - Loading: 초기 로딩 속도. 얼마나 페이지를 빨리 볼 수 있는가.
    - Interaction: 스크롤 및 키보드 입력 반응속도, 매끄러운 애니메이션.

_어느 부분이 느리다고 인지했다고 그 부분을 바로 들여다 보면 안된다. 시스템의 전체상태를 고려하고 계획을 짜야한다._

### 1. 대상 선정하기 (숲을 보기)
- 서비스에서 가장 많이 사용하는 화면은 무엇인가.
- 서비스에서 사용자에게 가치있는 화면은 무엇인가.

### 2. 개선 프로세스
- 측정(Measure): 어디가 느린지
- 분석 (Analytic): bottleneck 찾기
- 최적화 (Optimize): 해당 부분 튜닝 후 반영
- 위 과정 반복.

### 3. 언제까지?
- 목표에 도달할 떄 까지.
- 목표는? "초기로딩 속도가 3초 이상 걸리면 대부분의 사용자가 떠난다?" 
- 네이버는 모바일 1.5초, PC 2초  
![google metric](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_goolge_metric.PNG){: width="400 height="400"}  
- 구글은 세분화된 측정지표. load는 FMP (First Meaningful Paint)기준 1초. 네이버는 전체로딩 기준 

## Part1. 초기로딩속도 개선하기. 
![waterfall](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_waterfall.PNG){: width="600 height="500" .align-center}
  
결론은 Waterfall 차트를 어떻게 개선하느냐가 포인트.
> 높이를 줄이고, 폭을 줄이고, 간격을 땡긴다.  
> 마지막으로 총체적 점검하기

### 1. 높이줄이가: Request 수 줄이기
![request height](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_reduce_request_height.PNG){: width="600 height="500" .align-center}
- js, css 번들링
- css Sprite 기법으로 여러개 icon, 이미지 파일들 하나의 request로 묶기
- DATA URI 기법으로 html 안에 이미지 포함시키기.
- 가장 효과적인 방법: 초기 로딩시 불필요한 자원은 삭제하거나 lazy loading.
    - 실수로 요청한 자원들
    - 초기 로딩 시 필요없는 JS  
    ![image lazy loading](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_image_lzay_loading.PNG){: width="400 height="400"}  
    - [Viewport 바깥에 있는 이미지](https://web.dev/virtualize-long-lists-react-window/?utm_source=lighthouse&utm_medium=devtools)

    > 전체 웹페이지에서 통계 봤을 때 전체 웹페이지 1.6Mb 중 800KM가 이미지. 50%정도. 국내는 70~80% 됨.
- 브라우저는 동시에 연결할 수 있는 개수가 정해져있다. 이를 넘는 요청은 브라우저에서 대기 시키기 때문에 성능저하 효과가 나타남.


### 2. Request 하나의 폭 줄이기.
![request width](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_reduce_request_width.PNG){: width="400 height="500"}
![request information](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_request_information.PNG){: width="400 height="400"}
  
중요한 것은 initial connection, waiting (TTFB), content download
- Initial Connection: HTTP 프로토콜의 영향을 받음. HTTP2를 통해 스트리밍으로 request를 쏴야.(multiplex stream)
- TTFB (Time to First Byte): 서버에서 응답에 걸린시간. 서버 튜닝이 필요한 부분.
- **Content Download**: 네트워크 속도가 낮거나 컨텐츠의 크기가 큰 경우
    - js, css 압축: minify, obcuscation, gzip. 난독화(minify,obcusaction): 용량 50% 감소, 압축(gzip): 용량 70% 감소. -> 총 15% 정도로 감소.
    > webpack bundle analyzer에서 stat size: 원래크기. parsed size: minify 후 크기, gzippled: gzip 후 크기
    - HTTP2에 request의 헤더를 압축하는 기술이 있지만, 컨텐츠를 압축하는 것과는 무관하다.
    - 가장 효과가 큰것은 이미지 사이즈 줄이기. (화면에 500x300으로 보여지는데 실제 이미지는 2048x1362 인경우가 있다.)
    - 이미지 메타데이터 (시간, 위치 등) 지우기. 
    - 이미지는 받아온 후 브라우저에 그릴 수 있도록 RGB 로 변화하는 decode 비용도 크다.
    - 디자이너들은 레티나(device-ratio: 2) 이상을 요구한다. (500x300 의 사이즈에 1000x600 이미지를 사용하여 선명하게 보여줄 수 있다.) **눈딱 감고 2배까지만 허용하자.**
    
### 3. 간격 땡기기.
![reduce gab](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_reduce_gab.PNG){: width="400 height="400"}
![browser render sequence](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_brower_render_sequence.PNG){: width="400 height="400"}
- 브라우저 렌더링 과정 
- \<head>에 포함된 자원은 병렬 다운로드 및 실행
- \<head>에 포함된 자원을 모두 실행 할 때 까지 <body> 태그가 실행되지 않음.
- \<body> 태그가 실행되면 화면을 그리기 시작.
- DOM 구성이 완료되면(HTML, CSS 파싱이 끝나고 render tree 구성할 준비가 되면) DOMContentLoaded 이벤트 발생
- html 상에 필요한 모든 자원 로딩 완료되면 load 이벤트 발생 ([참고: TOAST UI 성능 최적화](https://ui.toast.com/fe-guide/ko_PERFORMANCE/#%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%EA%B8%B0%EC%A4%80%EC%9D%98-%EC%84%B1%EB%8A%A5-%EC%B8%A1%EC%A0%95))
![html head body](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_html_head_body.PNG){: width="500 height="500"}  
- \<body> 영역에서 로딩하는 js, css 는 render block 요소.
- **\<head> 태그에는 css와 필수 js만 넣고, js는 \<body> 마지막에 넣기.**
![async defer](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_script_async_defer.PNG){: width="500 height="500"}  
- 불가피할 때는 async, defer 태그 사용  
![preload](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_preload.PNG){: width="500 height="500"}  
- 원래는 css가 불려진 다음 css 에서 사용하는 폰트, 이미지 로딩. `rel="preload"`속성을 이용해서 폰트, 이미지 빠르게 로딩.
- HTTP2 sever push 기능으로 html 로드 동시에 css, js, png 로딩

### 4. 총체적 점검.
이전 과정중 오류에 빠질 수 있는것들 점검.

![fp fmp tti](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_fp_fmp_tti.PNG){: width="600 height="500" .align-center}
- FP (First Paint): <head> 태그 종료 후
- FMP (First Meaningful Paint): **Hero 엘리먼트** 가 보이는 시기. 구글의 기준
- TTI (Time to Interactive) : 사용자 액션으로 서비스를 이용할 수 있는 시기.
- 마냥 절대적인 로드 속도를 줄이기 보다는 위 지표들도 신경을 써야함.
- Hero 엘리먼트는 서비스 오너 / 기획자가 결정을 해주어야 함.

## Part2. 인터렉션 속도 개선하기. 
대부분 Casy by Case. 
But, 원리는 **브라우저 Main Thread 괴롭히지 말기**.
- Javascript가 DOM을 건들면 Main Thread에 의해 Rendering Pipeline 동작. 

![render pipeline](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_render_pipeline.PNG){: width="600 height="500" .align-center}

### 1. pipeline을 발생하지 않게 하는것이 포인트. 특히 layout과 paint.
![reflow attributes](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/naver_fe_perfomance/naver_fe_performance_reflow_attributes.PNG){: width="600 height="500" .align-center}
- 이들이 발생하지 않는 attribute를 건드리지 않기.
- ex. top: layout 부터 발생, transform: composite만 발생.

### 2. main thread (CPU)가 아닌 GPU 도움 받기
- composite: 그린 layer 들의 위상은 변경해서 겹치는 작업. 비용이 적음.
- 그 layer들을 구성하는데 GPU를 이용함. GPU의 도움을 받기위해 layer를 만듬. GPU의 가속을 주기위해가 아님.
    - 브라우저가 규칙에 따라 레이어 구성
    - 명시적으로 레이어 구성 (translate3d, scale3d, matrixd3d, will-change, ...)
    - Side Effect: layer 초기 구성작업은 CPU(main thread)가 진행. 이후 layer 비트맵 정보를 GPU로 복사하기 때문에 초기 CPU 부하가 크고 메모리가 2배로 듬. -> 꼭 필요한 부분만 레이어로 만든다.

### 3. 60fps, 1 frame 16ms 보장하기.
- render pipeline을 16ms 내에 끝내야 한다.
- requesetAnimationFrame으로 16ms 보장.
- DOM 건드리지 않는 JS 코드 실행시간 중이기 -> 서비스 로직 부분이라 개발자가 잘 알고 있다.

## QnA
- 리소스 사이즈는 250kb 정도 이하로, waterfall 차트의 균형을 맞추는 정도로 튜닝.
- <head>에 스크립트를 defer로 넣는것과 <body> 맨마지막에 넣는것 차이 x
- SPA도 페이지 이동시 네트워크 탭 보고 튜닝하면 된다.

## Results
- ~~성능개선은 빠르면 빠를수록 좋다~~  
-> 목표를 정해야 한다.
- ~~로딩속도는 수치적으로 빠르면 빠를수록 좋다.~~  
-> 절대적 수치보다는 Hero 컨텐츠 로딩 속도에 포커스
- ~~특정 부분의 성능을 개선하면 개선한 만큼 성능 향상이 된다.~~  
-> 전체 관점에서 가장 많이 사용하는 페이지를 살펴보고 워터폴 차트는 전체 폭이 줄어들 수 있도록 해야한다.
- ~~서비스 성능이슈는 개발자의 무지에서 나온다.~~  
-> 바빠서 신경을 못쓴다. 별도로 최적화 시간을 가져야.
- ~~성능 전문가는 서비스의 성능개선 포인트를 찾고 개선할 수 있다.~~  
-> 개선사항을 누구보다 잘 알 수 있는 사람은 서비스 개발자이다.
- ~~서비스 성능 개선은 전문성을 갖춘 영역이기에 아무나 할 수 없다.~~  
-> 이 강의만으로 충분. 노력과 관심이 중요하다.