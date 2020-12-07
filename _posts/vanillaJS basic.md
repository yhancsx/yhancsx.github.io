https://stackoverflow.com/questions/33276224/why-would-textcontent-not-trigger-a-reflow

- element.innerText : visibility, text-transform 등 고려 -> reflow 발생
- element.textContent: css 상관없이 포함한 텍스트 반환


https://developers.google.com/web/fundamentals/performance/rendering/avoid-large-complex-layouts-and-layout-thrashing#use_flexbox_over_older_layout_models
- float 대비 flex box 가 성능 4배 향상.


- inline css 는 최초 render tree 구축 후 additional reflow 발생.