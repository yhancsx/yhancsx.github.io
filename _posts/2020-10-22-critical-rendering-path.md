---
title: "Critical Rendering Path"
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
published: false
---

TL;DR

1. HTML 파싱
2. DOM 구축
3. CSSOM 구축
4. Render Tree 구축
5. Layout (Reflow)
6. Paint (Repaint)

## Constructing the Object Model

#### Document Object Model (DOM)

![html_parsing_sequence]({{site.url}}{{site.baseurl}}/assets/images/critical_rendering_path/html_parsing_sequence.PNG){: width="300 height="300" }

- html 파싱.
- byte -> characters -> tokens (tags) -> nodes (element) -> DOM (tree)
- css, javascript 파일을 만나면 파싱 중단 후 파일 요청.
- html 파싱 완료되면 `DOMContentLoaded` 이벤트 발생.

#### CSS Object Model (CSSOM)

![cssom_tree]({{site.url}}{{site.baseurl}}/assets/images/critical_rendering_path/cssom_tree.PNG){: width="300 height="300" }
![cssom_document_stylesheets]({{site.url}}{{site.baseurl}}/assets/images/critical_rendering_path/cssom_document_stylesheets.PNG){: width="300 height="300" }

- html 파싱 중 css 파일 가리키는 link 태그 만나면 css 파일 요청하여 CSSOM 구축.
- CSSOM 그려지기 전까지는 html 파싱 중지. (render blocking resource)
- DOM 과 동일한 과정.
- 부모의 속성을 이어 받음. (cascading)
- document.styleSheets 을 이용해 접근해보면 selector 계층 순서대로 적용되는 rule이 작성되어 있음.
- Chrome Dev Tools 에서 "Recalculate Style" 이 CSSOM 생성하는 단계.

## Constructing Render Tree

![combine_to_rendertree]({{site.url}}{{site.baseurl}}/assets/images/critical_rendering_path/constructing_render_tree.PNG){: width="300 height="300" }

- Visual representaion of the document.
- header, meta, script 등 제외.
- `display: none` 제외. (`visibilitiy: hidden` 은 포함)
- 노드들에 적절한 cssom rule 적용.

![layertree]({{site.url}}{{site.baseurl}}/assets/images/critical_rendering_path/dommtree_layertree.PNG){: width="300 height="300" }

#### Layer Tree (TBD)

- Render Object: DOM 노드중 화면에 표시할 것들 매핑.
- Render Layer Tree: 필요할 경우(z-index, position, transparency, overflow, etc.)
- Graphics Layer: 필요할 경우 (transform, animation, filter, \<video>, \<canvas>, etc.)

## Layout and Paint

#### Layout (Reflow)

- Geometry 정보 (위치, 사이즈) 결정
- <iframe width="700" height="394" src="https://www.youtube.com/embed/ZTnIxIA5KGw" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

#### Paint (Repaint)
