Google Web Fundamentals 의 Ciritical Rendering Path 를 찬찬히 읽으며 공부한 것들을 기록해보자.

## Critical Rendering Path
![html_parsing_sequence]({{site.url}}{{site.baseurl}}/assets/images/critical_rendering_path/html_parsing_sequence.PNG){: width="300 height="300" }
- html parsing
- byte -> characters  -> tokens (tags) -> nodes (element) -> object model (tree)
- dom tree construction (document object model)
- cssom tree construction (css object model) 
    - render blocking resource (the browser won't render any processed content until the CSSOM is constructed)
    - cascade propererties (부모 속성 물려받음) 
- dom과 cssom은 independent data structure
- combine into render tree
    - header, meta, script 등 제외
    - `display: none` 제외 (`visibility: hidden` 은 포함)
    - 남은 노드들에 적절한 cssom rules 적용
- layout (reflow)
    - 위치, 사이즈 결정
    - font-size, window size 등 변경시 reflow
- paint
    - 픽셀 색칠
    
<iframe width="700" height="394" src="https://www.youtube.com/embed/ZTnIxIA5KGw" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Render Tree
![transition-domtree_layertree-function]({{site.url}}{{site.baseurl}}/assets/images/critical_rendering_path/domtree_layertree.PNG){: width="300 height="300" }

- Render Object Tree (= render tree)
    - visual  output
    - geometric info
    - can layout and paint
- Render Layer Tree (필요할경우 매핑)
    - position nodes, transparency, overflow, etc.
- Graphics Layer (필요할 경우 매핑)
    - css: transform, animation, filter,
    - html :\<video>, \<canvas>
- Composite


## Meta tag
`<meta name="viewport" content="width:device-width, initial-scale=1"/>`  
viewport의 크기는 device width와 같아야 하며, 초기 로딩시 scale 1배를 적용한다.  
이 태그가 없으면 기본 크기인 980px 적용.

## rendering block css
#### css
- css는 렌더링 블락요소. cssom이 모두 구성되기 전까지는 html 파싱을 재개하지 않음.
- media query로 필요한 파일만 로드하도록.
```html
<link href="style.css" rel="stylesheet">
<link href="print.css" rel="stylesheet" media="print"> 
<link href="other.css" rel="stylesheet" media="(min-width: 40em)">
<!--portrait: 세로 모드, landscape: 가로 모드-->
<link href="portrait.css" rel="stylesheet" media="orientation:portrait"> 
```
- cssom tree

#### script
- js 또한 렌더링 블락요소.
- \<script /> 태그는 body 가장 아래에. HTML parse 중단하지 않고 렌더링 부터 빠르게
- defer / async 사용


## Optimization
- dom 수정은 한번에
- preventing layout thrashing
- `transform` rather than `left` or `top`


## References
- [Google Web Fundamentals > Critical Rendering Path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path)
- [브라우저는 웹페이지를 어떻게 그리나요? - Critical Rendering Path](https://post.naver.com/viewer/postView.nhn?volumeNo=8431285&memberNo=34176766)
- [How Browsers Work: Behind the scenes of modern web browser](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/#Render_tree_construction)
- [Udacity Course > Website Performance Optimization](https://www.udacity.com/course/website-performance-optimization--ud884)
- [JSConf EU 2015 > Ryan Seddon:  So how does the browser actually render a website](https://www.youtube.com/watch?v=SmE4OwHztCc&feature=emb_title)
- [Naver D2 > 하드웨어 가속에 대한 이해와 적용](https://d2.naver.com/helloworld/2061385)
- [Performance monitoring in CSS animations](https://medium.com/chegg/performance-monitoring-in-css-animations-f11a21d0054f)
