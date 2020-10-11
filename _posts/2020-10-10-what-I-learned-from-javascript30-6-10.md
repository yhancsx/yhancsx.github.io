---
title: "What I Learned from Javascript30 (6~10)"
category: web  
tags:
  - web
  - front-end
  - html
  - css
  - javascript
  - regexp
---

[Wes Bos](https://github.com/wesbos) 님의 Vanilla Javascript 강의인 [Javascript 30](https://javascript30.com)을 따라하며 새롭게 배운 내용들을 정리한 포스트입니다.
 
## 6. Type Ahead

#### RegExp
```javascript
const re = /ab+c/i; // literal pattern
const re = new RegExp('ab+c', 'i'); // constructor with string pattern 
const re = new RegExp(/ab+c/, 'i'); // constructor with literal pattern (ES6 에서 추가)
```
위 3가지 모두 같은 regex expression object 생성.

- `i` : 대소문자 무시
- `g` : 글로벌 매치 (첫 매치에서 멈추지 않음)  

```javascript
"ABC".match(/\w/i) // => ["A", index: 0, input: "ABC", groups: undefined]
[..."ABC".matchAll(/\w/gi)] 
// => [
    ["A", index: 0, input: "ABC", groups: undefined],
    ["B", index: 1, input: "ABC", groups: undefined],
    ["C", index: 2, input: "ABC", groups: undefined],
]
"ABC".replace(/\w/i,"a") // =>aBC
```
`match()`, `matchAll()`, `replace()`, 함수에서 사용 가능

## 7. Array Cardio 2
```javascript
const people = [
  { name: "Wes", year: 1988 },
  { name: "Kait", year: 1986 },
  { name: "Irv", year: 1970 },
  { name: "Lux", year: 2015 },
];
```

#### Array.prototype.some 
```javascript
people.some(({ year }) => new Date().getFullYear() - year >= 19); // => true
```

배열 안의 어느 한 요소라도 주어진 함수를 통과하는지 확인

#### Array.prototype.every 
```javascript
people.every(({ year }) => new Date().getFullYear() - year >= 19); // => true
```
배열 안의 모든 요소가 주어진 함수를 통과하는지 확인

```javascript
const comments = [
  { text: "Love this!", id: 523423 },
  { text: "Super good", id: 823423 },
  { text: "You are the best", id: 2039842 },
  { text: "Ramen is my fav food ever", id: 123523 },
  { text: "Nice Nice Nice!", id: 542328 },
];
```
#### Array.prototype.find 
```javascript
comments.find(({ id }) => id === 823423); // => { text: "Super good", id: 823423 }
```
주어진 함수를 만족하는 배열안의 첫 원소 리턴


#### Array.prototype.findIndex 
```javascript
const findIndexByID = comments.findIndex(({ id }) => id === 823423); // => 1
```
주어진 함수를 만족하는 배열안의 첫 원소의 인덱

#### Array.prototype.splice
```javascript
array.splice(startIndex, deleteCount, item1, item2, ...)
```
```javascript
comments.splice(findIndexByID, 1);

console.log(comments) // => [
  { text: "Love this!", id: 523423 },
  { text: "You are the best", id: 2039842 },
  { text: "Ramen is my fav food ever", id: 123523 },
  { text: "Nice Nice Nice!", id: 542328 },
]
```
기존 배열 요소를 삭제하거나 대체하거나 추가.

## 8. HTML5 Canvas
[Github Examples](https://github.com/yhancsx/Javascript30/tree/master/08%20-%20Fun%20with%20HTML5%20Canvas/index.js)

## 9. Dev Tools Tricks

#### Break on Element Change

![Set BreakPoint]({{site.url}}{{site.baseurl}}/assets/images/javascript30/element_break_on_change.png){: width="400 height="400" }
![Break in JS File]({{site.url}}{{site.baseurl}}/assets/images/javascript30/element_break_in_js.png){: width="400 height="400" }

#### Console Logging Level
```javascript
console.log();
console.info();
console.warn();
console.error();
```
#### Assert
```javascript
console.assert([condition], [logging])
```
#### Showing DOM Element
```javascript
const p = document.querySelector("p");
console.log(p) // => <p>innerText</p>
console.dir(p) // => Show all attributes of DOM element as object format
```

#### Grouping
```javascript
console.groupCollapsed(`${dog.name}`);
console.log(`This is ${dog.name}`);
console.log(`${dog.name} is ${dog.age} years old`);
console.log(`${dog.name} is ${dog.age * 7} dog years old`);
console.groupEnd(`${dog.name}`);
```
![Set Console Group]({{site.url}}{{site.baseurl}}/assets/images/javascript30/console_group.png){: width="400 height="400" }

#### Counting
```javascript
console.count('yhan') // => yhan : 1
console.count('yhan') // => yhan : 2
```

#### Timing
```javascript
console.time("fetching data");
setTimeout(() => console.timeEnd("fetching data"), 1000);
// => fetching data: 1005.619140625 ms
```

#### Clearing
```javscript
console.clear()
```

## 10. Shitkey and Checkbox

#### MouseEvent shiftKey property
Mouse Event 에는 같이 눌린 키보드 정보를 가져올 수 있다.
```
Mouse Event
{
  ...
  altKey: false,
  metaKey: false,
  shiftKey: false,
  ctrkKey: false,
  ...
}
```
Keyboard Event 등에도 똑같이 들어 있다.