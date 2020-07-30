

```
$ npx create-react-app webpack-study
$ cd webpack-study
$ yarn eject 
```
create-react-app (CRA) 를 이용해 react 프로젝트를 만들고,
yarn eject를 통해 추출한 config 폴더 및 webpack.config.js 파일을 이해해 보자.

## Webpack  // 모듈?
기존의 웹에서 javascript 개발 방식은 기능별/페이지별로 자바스크립트 파일을 분리하고 HTML의 \<script> 태그로 로드하는 것이었다.
하지만 브라우저에서 여러개의 파일을 로딩하며 치르는 네트워크비용, 파일간의 스코프 침범 등의 단점이 있었다.  
더욱이 현대의 프론트엔드는 역할과 기능이 다양해지며 react, angular 등의 프레임워크가 등장하였고, 더이상 몇개의 js 파일로 작성하는 것이 불가능해졌다.
이를 해결하기 위해 프로젝트 안에 포함된 js, css, img 파일들을 모두 엮어 하나의 자바스크립트 파일로 만들어 주는 번들러로서 webpack 이 등장하였다.

#### build
2.chunk.js 파일에는 react, react-dom 등 node_modules 관련 코드들이,
main.chunk.js 파일에는 내가 작성한 컴포넌트 코드들이 들어간다.

## Loaders
webpack 은 js 뿐아니라 이미지, 폰트, css 등 모든 파일을 모듈로 관리한다. 하지만 webpack은 javascript 밖에 모르므로 다른 파일들을 이해할 수 있게끔 해주어야한다.
이 역할을 하는 것이 loader이다. loader는 {test, use} 형태의 객체이다.


```javascript
rules: [
        ...
        {
            test: /\.(scss|sass)$/,
            use: ['style-loader', 'css-loader', 'sass-loader'] 
        }
        ...
    ]
```
위와 같이 loader들을 webpack에 plugin으로 추가하여 번들링한다.  
use에 나열된 loader은 역순으로 적용 된다는 것이 중요.
 
#### babel-loader
우리는 기본적으로 ES6 이상의 문법을 이용하여 코드를 작성한다.
하지만 브라우저에 따라 높은 버전의 자바스크립트를 이해할수 없는 경우가 있다.
이를 해결하기 위해 ES5 문법으로 변환해주는 것이 babel 이다.

##### babel-presets-react-app
**`package.json`**
```json
...
    "babel": {
      "preset": [
        "react-app"  
      ]
    }
...
```

ES6 문법의 각 구성요소를 ES5로 변환하려면 그에 해당되는 babel plugin을 설정해주어야 한다.  
예를 들어 `const arrow = () => {}` 와 같은 화살표 함수를 변화해주려면 `@babel/plugin-transform-arrow-functions` 플러그인을 설치하고 설정해 주어야한다.  
문법 하나하나에 대해 매번 설치하고 설정해주는 것이 귀찮으니 plugin들을 포함한 번들이 만들어졌는데 그것이 `preset`이다.  
cra로 만들어진 프로젝트는 기본적으로 react에서 사용되는 plugin들을 담은 `babel-preset-react-app`을 포함하고 있다.  
그 외에 babel 공식 preset으로는 `@babel/preset-env`,`@babel/preset-flow`,`@babel/preset-react`,`@babel/preset-typescript` 등이 있다.  
babel은 package.json 파일 대신 .babelrc 파일을 별도로 생성하여 설정 할 수 있다.

#### style-loader
> "style" loader turns CSS into JS modules that inject \<style> tags.

css-loader를 통해 묶인 css 파일을 DOM에 \<style> 태그 형태로 삽입해준다.
style-loader 대신 `mini-css-extract-plugin` plugin을 사용하면 \<style> 태그 대신 \<link> 태그로 삽입할수 있다.

#### css-loader
> "css" loader resolves paths in CSS and adds assets as dependencies.

@import, url() 등을 해석하여 import/require 와 같은 js 형태로 묶어준다. 

#### postcss-loader
>"postcss" loader applies autoprefixer to our CSS.
```css
.box-wrap {
    box-sizing: border-box;
    -webkit-box-sizing: border-box;  // chrome, safari prefix
    -moz-box-sizing: border-box;     // firefox
    -o-box-sizing: border-box;       // old opera versions
    -ms-box-sizing: border-box;      // IE, Microsoft Edge
}
```
js plugin 들을 적용하여 css로의 변환을 도와준다. js를 변환해주는 babel과 개념이 비슷하다.
autoprefixer 플러그인을 사용하면 기존에 크로스 브라우저 호완을 위해 붙여야 했던 접두사를 자동으로 붙여준다.
그외에 color() modifier, 변수사용, @import, nesting, for loop 사용등을 가능하게 하는 plugin 등이 있다.

sass, less 등은 css 로 변환되는 `스타일시트 언어` 또는 `css preprocessor` 이지만 PostCSS는 js기반의 플러그인을 이용하여 css로 변환은 자동화하는 개발 도구에 가깝다.
 
#### sass-loader
sass(scss) 파일을 css로 변환해준다.


- [김정환 블로그 > 웹팩의 기본 개념](http://jeonghwan-kim.github.io/js/2017/05/15/webpack.html)
- [TOAST UI > 번들러](https://ui.toast.com/fe-guide/ko_BUNDLER/)
- [pop8682.log > babel-preset-env는 무엇이고 왜 필요한가](https://velog.io/@pop8682/%EB%B2%88%EC%97%AD-%EC%99%9C-babel-preset%EC%9D%B4-%ED%95%84%EC%9A%94%ED%95%98%EA%B3%A0-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%9C%EA%B0%80-yhk03drm7q)
- [HyeonSeok Yang > PostCSS 소개](https://medium.com/@FourwingsY/postcss-%EC%86%8C%EA%B0%9C-727310aa6505) 