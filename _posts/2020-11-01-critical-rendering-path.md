---
title: "Browser Critical Rendering Path"
category: web
tags:
  - javascript
  - web
  - front-end
  - study
  - browser
  - rendering
  - html
  - css
  - dom
  - cssom
  - render tree
  - layer tree
  - performance
  - optimization
  - critical render path
---

브라우저가 html 파일을 렌더링 하는 과정을 학습해 보자.

TL;DR

1. HTML 파싱
2. DOM Tree 구축
3. CSSOM 구축
4. Render Tree 구축
5. Layout (Reflow)
6. Paint (Repaint)
7. Composite

## 1. Constructing the Object Model

#### Document Object Model (DOM)

![html_parsing_sequence]({{site.url}}{{site.baseurl}}/assets/images/critical_rendering_path/html_parsing_sequence.PNG)

- html 파싱.
- byte -> characters -> tokens (tags) -> nodes (element) -> DOM (tree)
- css, javascript 파일을 만나면 파싱 중단 후 파일 요청.


#### CSS Object Model (CSSOM)

![cssom_tree]({{site.url}}{{site.baseurl}}/assets/images/critical_rendering_path/cssom_tree.PNG)

![cssom_document_stylesheets]({{site.url}}{{site.baseurl}}/assets/images/critical_rendering_path/cssom_document_stylesheets.PNG)

- html 파싱 중 css 파일 가리키는 link 태그 만나면 css 파일 요청하여 CSSOM 구축.
- CSSOM 그려지기 전까지는 html 파싱 중지. (render blocking resource)
- DOM 과 동일한 과정.
- 부모의 속성을 이어 받음. (cascading)
- document.styleSheets 을 이용해 접근해보면 selector 계층 순서대로 적용되는 rule이 작성되어 있음.
- Chrome Dev Tools 에서 `Recalculate Style` 이 CSSOM 생성하는 단계.


## 2. Constructing Render Tree

![combine_to_rendertree]({{site.url}}{{site.baseurl}}/assets/images/critical_rendering_path/constructing_render_tree.PNG)

- DOM, CSSOM 구성이 완료되면(Render Ttree 구축 준비가 되면) `DOMContentLoaded` 이벤트 발생.
- Visual representaion of the document.
- header, meta, script 등 제외.
- `display: none` 제외. (`visibilitiy: hidden` 은 포함)
- 노드들에 적절한 cssom rule 적용.


#### [Render Layer Tree](https://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome)

![layertree]({{site.url}}{{site.baseurl}}/assets/images/critical_rendering_path/domtree_layertree.PNG)

- Render Tree 의 구셩요소.
- Render Object: DOM 노드 중 화면에 표시할 render tree 의 구성 요소.  
- Render Layer: 레이어 분리가 필요할 경우. (root, opacity, css position, transparency, overflow, etc.)
- Graphics Layer: GPU 가속이 필요할 경우. (transform, css animation, css filter, \<video>, \<canvas>, etc.)
- 크롬 개발자 도구의 layers 탭에서 layer 시각적으로 확인 가능.
- 크롬 개발자 도구 Rendering > Layer border 로 화면에 layer 표시 가능. 

## 3. Layout, Paint and Composite

<iframe width="700" height="394" src="https://www.youtube.com/embed/ZTnIxIA5KGw" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
![reflow_repaint](https://developers.google.com/web/fundamentals/performance/rendering/images/intro/frame-full.jpg)
#### Layout (Reflow)

- Geometry 정보 (위치, 사이즈) 결정
- DOM 구성요소가 추가/삭제 되거나 geometry 정보가 변경될 경우 reflow 발생.

#### Paint (Repaint)

- layout 단계에서 계산된 각 노드를 실제 화면의 픽셀로 변환하여 그리는 작업.
- color, visibility, text-decoration 등의 css 변경시 repaint 발생.

#### Composite 

- paint 된 여러개의 Render layer를 GPU를 사용하여 합성.

## References
- [Google Web Fundamentals > Critical Rendering Path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path)
- [Udacity Course > Website Performance Optimization](https://www.udacity.com/course/website-performance-optimization--ud884)
- [브라우저는 웹페이지를 어떻게 그리나요? - Critical Rendering Path](https://post.naver.com/viewer/postView.nhn?volumeNo=8431285&memberNo=34176766)
- [HTML5 Rocks > How Browsers Work: Behind the scenes of modern web browsers](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork)
- [HTML5 Rocks > Accelerated Rendering in Chrome](https://www.html5rocks.com/ko/tutorials/speed/layers/)
- [Naver D2 > 브라우저는 어떻게 동작하는가](https://d2.naver.com/helloworld/59361)
- [Naver D2 > 하드웨어 가속에 대한 이해와 적용](https://d2.naver.com/helloworld/2061385)
- [The Chromium Projects > GPU Accelerated Compositing in Chrome](https://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome)
- [JSConf EU 2015 > Ryan Seddon:  So how does the browser actually render a website](https://www.youtube.com/watch?v=SmE4OwHztCc&feature=emb_title)
- [Performance monitoring in CSS animations](https://medium.com/chegg/performance-monitoring-in-css-animations-f11a21d0054f)
