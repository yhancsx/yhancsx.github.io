---
title: "Javascript Tree Shaking"
category: js
tags:
  - web
  - study
  - front-end
  - javascript
  - webpack
  - es6
  - optimization
  - tree shaking
  - bundle
  - babel
---
front-end 최적화 방안 중 하나인 tree shaking에 대해 공부해 보자.

#### Prerequisites

[webpack bundle analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer) 또는 [source-map-explore](https://www.npmjs.com/package/source-map-explorer) 를 사용하면 build 후 bundle에 포함되는 라이브러리들을 살펴볼 수 있다.

## Tree Shaking

rollup.js에서 처음 제시되었던 개념으로, 사용되지 않는 component 및 dependency 들을 최종 bundle에서 제외시키는 최적화 기능을 말한다.

단, CommonJS (`require()`, `module.exports`) 형태로 export 된 library는 tree-shaking이 되지 않는다. ES Module에서만 tree-shaking을 할 수 있다.

## 내부 components

```javascript
import components from "components";
import * as components from "components";
```

이 형태의 import 는 기본적으로 tree-shaking이 되지 않는다.

```javascript
import { conponent } from "components";
```

형태로 import 해주거나

`package.json` 에서 [`sideEffects: false`](https://webpack.js.org/guides/tree-shaking/) 설정을 해주면 사용하지 않는 component들은 bundle 에 포함되지 않는다.

**components/index.js**
```javascript
export { default as componentA } from 'components/componentA';
export { default as componentB } from 'components/componentB';
```

각 폴더의 index 파일에서 해당 폴더 안의 모든 components들을 모아서 re-export 해주는 경우가 있다.

**container/container.js**
```javascript
import { componentA } from 'components`
```

위 방식으로 componentA를 사용하며 다른 어디서도 componentB를 사용하지 않았더라도 최종 번들에는 componentB가 포함된다.

이를 방지하기 위해서는 `sideEffects: false` 설정을 해주어야 한다.

## Dependencies

ES6 Module 형태로 작성되었으면서 해당 라이브러리의 package.json에 `sideEffects:false` 설정이 되어 있으면 tree-shaking이 기본적으로 된다.

#### antd

`package.json` 에서 dist 폴더 및 style 파일들을 제외하고는 `sideEffects: false` 설정이 되어있으며 대부분의 component가 export 되는 es 폴더는 es6+ 문법으로 작성되어 있다. 따라서 build 하면 tree-shaking이 되어서 import 된다.

```javascript
import * as antd from "antd";
import { Button } from "antd";
```

어떤 형태로 import 하든 사용하는 component만 bundle에 포함된다.

[babel-plugin-import](https://github.com/ant-design/babel-plugin-import)를 사용하면 lib 폴더의 컴포넌트를 cherry-pick 해서 import 하게 된다.

#### lodash

```
import * as _ from "lodash"
...
_.sortBy(arr)
...
또는

import { sortBy } from "lodash"
```

위 형태의 import 모두 tree-shaking이 잘 안된다.

![lodash not_tree shaking](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/tree-shaking/lodash_not_tree_shaking.png){: width="400 height="400"}

**`lodash/index.js;`**
```javascript
module.exports = require("./lodash");
```

이는 lodash에서 각 요소들을 commonJS 형태로 export 하고 있기 때문이다.

```
import sortBy from "lodash/sortBy"
```

형태로 import 하거나

[`babel-plugin-lodash`](https://github.com/lodash/babel-plugin-lodash) 를 설치해야한다.

이 플러그 인을 사용하면 `import _ from "lodash'` 형태로 사용하여도 tree shaking 이 된다.

![lodash tree shaking](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/tree-shaking/lodash_tree_shaking.png){: width="400 height="400"}

#### d3

`import * as d3 from 'd3'` 는 모든 d3 모듈을 다 불러온다.

![d3 not_tree shaking](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/tree-shaking/d3_not_tree_shaking.png){: width="400 height="400"}

`import { foramt } from 'd3-format`을 사용하면 사용되는 모듈만을 불러온다.

![d3 tree shaking](https://github.com/yhancsx/yhancsx.github.io/raw/master/assets/images/tree-shaking/d3_tree_shaking.png){: width="400 height="400"}

[`babel-plugin-transform-d3-imports`](https://www.npmjs.com/package/babel-plugin-transform-d3-imports)
d3의 각 모듈에서도 내부 폴더까지 cherry-picking 해주는 babel-plugin을 사용하면 여기서 더 줄일 수 있을것 같지만
사용한 결과 크기상에 변화가 거의없다 parsed size 기준 30KB 정도. (3.4MB -> 3.37MB)

- [Huns.me > Webpack 4의 Tree Shaking에 대한 이해](https://huns.me/development/2265)
- [webpack > Tree Shaking](https://webpack.js.org/guides/tree-shaking/)
- [webpack > Side Effects](https://webpack.js.org/configuration/optimization/#optimizationsideeffects)
- [Lukas Bombach >How to write a tree-shakable component library](https://dev.to/lukasbombach/how-to-write-a-tree-shakable-component-library-4ied)
- [Ant design > babel-plugin-import](https://github.com/ant-design/babel-plugin-import)
- [Google Web Fundamentals > Reduce JavaScript Payloads with Tree Shaking](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/tree-shaking)
- [Toast UI > 트리 쉐이킹으로 자바스크립트 페이로드 줄이기](https://ui.toast.com/weekly-pick/ko_20180716/)
