---
title: "React (Vue)에 Eslint Prettier 적용하기 (+Typescript)"
category: js
tags:
  - javascript
  - front-end
  - study
  - eslint
  - prettier
  - react
  - vue
  - vscode
---

Javascirpt, Typescript를 주로 사용하는 프론트엔드 개발자를 위한 생산성 도구로 ESLint와 Preitter가 있다. 이들에 대해 공부해보고 VSCode에 적용해보자.

## 패키지 설치
```shell
$ npm i --save-dev prettier eslint eslint-config-prettier eslint-plugin-prettier eslint-plugin-react(eslint-plugin-vue) --save-dev

$ yarn add --dev prettier eslint eslint-config-prettier eslint-plugin-prettier eslint-plugin-react(eslint-plugin-vue)
```

## ESlint
잘못된 코드 스타일로 인해 에러가 나지 않게 코드 문법을 잡아주는 문법 검사기.
- 문법 통일성
- 사용하지 않는 변수, 함수, 모듈
- 문장 뒤 세미콜론, 콤마 붙이기
- 등등..

## Pritter
코드 스타일 정리.
- 한줄당 최대 코드 수
- 작은 따옴표, 큰따옴표
- tab대신 space 사용
- 등등...

## eslint-plugin-prettier
- ESlint 규칙에 prettier의 코드 스타일 기능 삽입해준다.
- prettier 항목이 ESlint 알림을 통해 표시된다.

**.eslintrc.js**
```javascript
{
    "extends": ["plugin:prettier/recommended"],
}
```
는 아래와 같다.
```javascript
{
  "extends": ["prettier"],
  "plugins": ["prettier"],
  "rules": {
    "prettier/prettier": "error",
    "arrow-body-style": "off",
    "prefer-arrow-callback": "off"
  }
}
```

`.eslintrc.js`안에 `"prettier/prettier": ["error", { "singleQuote": true, "parser": "flow" }]`와 같은 형태로 prettier 설정을 넣어줄 수도 있다. 

하지만 몇몇 IDE의 prettier는 (prettier-vscode 포함) `.prettierrc.js`는 읽지만 `.eslintrc.js` 내부 prettier 설정은 읽지 못한다. 그래서 eslint에 의한 error 표시는 뜨지만 prettier에 의해 auto-fix가 되진 않는다. 추가하고 싶은 rule은 `.prettierrc.js`파일에 작성하자. ([Link](https://github.com/prettier/eslint-plugin-prettier#options))

## eslint-config-prettier
ESlint 규칙 중 prettier 와 충돌할 수 있는 기능들 (코드 스타일 관련) 비활성시키는 역할을 한다.

**.eslintrc.js**
```javascript
{
    "extends": ["prettier"],
}
```

`eslint-plugin-prettier`의 `plugin:prettier/recommended` 이 자동으로 생성해주므로 따로 설정해줄 필요는 없다.


> "extends" 는 가장 마지막에 오는 설정이 다른 설정들을 덮어쓴다.

## eslint-plugin-react
- React 문법을 위한 공식 ESlint plugin.

## eslint-plugin-vue
- VueJS 를 위한 공식 ESlint plugin. 
- .vue 파일의 \<template>과 \<script>에 ESlint 적용 가능하다.

## ESLint/Prettier 설정

**eslintrc.js**
```javascript
module.exports = {
  root: true,
  parserOptions: {
    parser: 'babel-eslint',
  },
  env: {
    browser: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:prettier/recommended',
    'plugin:react/recommended',
    // 'plugin:vue/essential',
  ],
  rules: {
    'max-len': [
      'warn',
      {
        code: 150,
        tabWidth: 2,
        comments: 150,
        ignoreComments: false,
        ignoreTrailingComments: true,
        ignoreUrls: true,
        ignoreStrings: true,
        ignoreTemplateLiterals: true,
        ignoreRegExpLiterals: true,
      },
    ],
  },
};
```
**prettier.js**
```javascript
module.exports = {
  printWidth: 150,
  singleQuote: true,
  trailingComma: 'all',
  useTabs: false,
  htmlWhitespaceSensitivity: 'ignore',
  tabWidth: 2,
  semi: true,
};
```
[`"htmlWhitespaceSensitivity"](https://prettier.io/docs/en/options.html#html-whitespace-sensitivity): html 태그 내부의 whitespace를 중요하게 여길지 무시할지를 결정한다. 의도치 않게 화면구성이 변할 수 있다.


## VSCODE 적용
![ESLint Plugin]({{site.url}}{{site.baseurl}}/assets/images/vscode_eslint_prettier/eslint.PNG)
- ESlint Plugin 설치.
![Prettier Plugin]({{site.url}}{{site.baseurl}}/assets/images/vscode_eslint_prettier/prettier.PNG)
- Prettier Plugin 설치.		
        
**setting.json**
```json
{
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
  },
  "[vue]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
  },
  "editor.formatOnSave": true,
  "vetur.experimental.templateInterpolationService": true,
  "eslint.alwaysShowStatus": true,
}	
```

auto-format 기능을 prettier가 아닌 eslint로 사용하려면 다음과 같이 설정하면 된다.
```json
{
  "editor.formatOnSave": false,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
}
```

이때는 
`.eslintrc.js` 에서 `"prettier/prettier": ["error", {"singleQuote": true, "parser": "flow"}]` 와 같은 설정으로 auto-format이 가능하다.


## Typescript
```shell
npm i --save-dev typescript eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin

yarn add --dev typescript eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

TSLint 프로젝트가 ESLint로 통합되었다. https://github.com/palantir/tslint/issues/4534

## @typescript-eslint/parser
ESLint의 default parser를 대체하기 위한 Typescript용 파서

## @typescript-eslint/eslint-plugin
Typescript 관련 Lint 규칙 모음


## ESLint/Prettier 설정

Prettier와 함께 사용하기 위해선 똑같이 `eslint-config-prettier`, `eslint-plugin-prettier` 가 필요하다.

**eslintrc.js**
```javascript
module.exports = {
  parser: "@typescript-eslint/parser",
  env: {
    browser: true,
  },
  extends: [
    'eslint:recommended', // eslint basic
    'plugin:@typescript-eslint/recommended', // @typescript-eslint/eslint-plugin
    'prettier/@typescript-eslint',  // eslint의 typescript formatting 기능 제거 (eslint-config-prettier)          
    'plugin:prettier/recommended',  // eslint-plugin-prettier, eslint-config-prettier
    'plugin:react/recommended', // react 추천 규칙
    // 'plugin:vue/essential' vue 추천 규칙
  ],
  rules: {
    'max-len': [
      'warn',
      {
        code: 150,
        tabWidth: 2,
        comments: 150,
        ignoreComments: false,
        ignoreTrailingComments: true,
        ignoreUrls: true,
        ignoreStrings: true,
        ignoreTemplateLiterals: true,
        ignoreRegExpLiterals: true,
      },
    ],
  },
};
```

## References
- [eslint-plugin-prettier](https://github.com/prettier/eslint-plugin-prettier)
- [ESLint Plugin TypeScript](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/eslint-plugin)
- [prettier와 eslint설정하기(+타입스크립트 설정)](https://simsimjae.medium.com/prettier%EC%99%80-eslint%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%84%A4%EC%A0%95-110dc8ab94b6)
- [Vue.js 개발 생산성을 높여주는 도구 3가지](https://joshua1988.github.io/web-development/vuejs/boost-productivity/)
- [VSCode에서 @vue/cli4, Prettier 그리고 Eslint 통합환경 구성하기](https://webruden.tistory.com/179)
- [TypeScript ESLint + Prettier 함께 사용하기(w/ VSCode)](https://pravusid.kr/typescript/2020/07/19/typescript-eslint-prettier.html)














