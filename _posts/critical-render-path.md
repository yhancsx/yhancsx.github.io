## Critical Rendering Path
- html parsing
- byte -> characters  -> tokens (tags) -> nodes (element) -> object model (tree)
- dom tree construction (document object model)
- cssom treeconstruction (css object model) 
    - render blocking resource (the browser won't render any processed content until the CSSOM is constructed)
    - cascade propererties (부모 속성 물려받음) 
- dom과 cssom은 independent data structure
- combine into render tree
    - header, meta, script 등 제외
    - `display: none` 제외 (`visibility: hidden` 은 포함)
    - 남은 노드들에 적절한 cssom rules 적용
- layout (reflow)
    - 위치, 사이즈 결정
- paint
    - 픽셀 색칠

## Render Tree

- Render Object Tree (= render tree)
- Render Layer Tree ( 필요할경우 매핑)
- Graphics Layer (필요할 경우 매핑)
    - css: transform, animation, filter,
    - html :\<video>, \<canvas>
    -       
- Composite
    
![transition-domtree_layertree-function]({{site.url}}{{site.baseurl}}/assets/images/critical_rendering_path/domtree_layertree.PNG){: width="300 height="300" }



- [Google Web Fundamentals > Critical Rendering Path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path)
- [브라우저는 웹페이지를 어떻게 그리나요? - Critical Rendering Path](https://post.naver.com/viewer/postView.nhn?volumeNo=8431285&memberNo=34176766)
- [Udacity Course > Website Performance Optimization](https://www.udacity.com/course/website-performance-optimization--ud884)

