---
title: '[기술서적 리뷰] 이펙티브 타입스크립트 - 3. 타입 추론 (아이템 19 ~ 23)'
category: js
tags:
  - javascript
  - study
  - typescript
---

*DAN VANDERKAM*님의 [이펙티브 타입스크립트](https://book.naver.com/bookdb/book_detail.nhn?bid=20611649)을 읽고 요악하는 포스트입니다.

## 아이템 19. 추론 가능한 타입을 사용해 장황한 코드 방지하기

```typescript
let x: number = 12; // Bad
let x = 12; // Good

// Bad
const person: {
  name: string;
  born: {
    where: string;
    when: string;
  };
  // ...
} = {
  name: 'yhan',
  born: {
    where: 'Busan, South Korea',
    when: '25, 12, 1993',
  },
  // ...
};

// Good
const person = {
  name: 'yhan',
  born: {
    where: 'Busan, South Korea',
    when: '25, 12, 1993',
  },
  // ...
};
```

```typescript
function square(nums: number[]) {
  return nums.map((x) => x * x);
}
const squares = square([1, 2, 3, 4]); // number[]
```

IDE가 추론가능한 타입에 대해서는 명시적 타입 구문이 필요하지 않습니다.

함수 및 메서드 시그니처에는 타입구문을 포함하고, 함수 내에서 생성된 지역변수에는 타입구문을 넣치 않는 것이 좋습니다. 타입구문을 생략하여 불필요한 코드들을 줄이고 로직에 집중할 수 있도록 하는 것이 좋습니다.

```typescript
// Good
const elmo: Product = {
  name: 'Tickle Me Elmo',
  id: '048188 627152',
  price: 28.99,
};
```

객체 리터럴을 정의할 때는 타입을 명시해주면 오타 등의 오류를 잡는데 유용합니다 (잉여 속성 체크). 사용하는 곳이아닌, 선언하는 곳에서 오류를 발견할 수 있습니다.

```typescript
interface Vector2D {
  x: number;
  y: number;
}
function add(a: Vector2D, b: Vector2D): Vector2D {
  return { x: a.x + b.x, y: a.y + b.y };
}
```

함수를 작성할 때도 반환타입을 구체적으로 명시함으로써 사용자가 타입명세를 확인할 때 혼동하지 않게 해주는 것이 좋습니다.

eslint의 `no-inferrable-types` 규칙을 사용하면 작성도니 타입구문이 정말로 필요한지 확인할 수 있습니다.

## 아이템 20. 다른 타입에는 다른 변수 사용하기

> 변수의 값은 바뀔 수 있지만 그 타입은 보통 바뀌지 않는다.

**Bad**

```typescript
let id: string | number = '12-34-56';
fetchProduct(id);

id = 123456;
fetchProductBySerialNumber(id);
```

**Good**

```typescript
const id = '12-34-56';
fetchProduct(id);

const serial = 123456;
fetchProductBySerialNumber(serial);
```

다른 타입에는 별도의 변수를 사용하는 것이 좋습니다.

- 서로관련없는 두 값을 분리합니다.
- 변수명을 구체적으로 지을 수 있습니다.
- 타입 추론을 향상시키고, 타입 구문이 불필요해집니다
- let 대신 const를 사용할 수 있습니다.

## 아이템 21. 타입 넓히기

타입스크립트가 작성된 코드를 체크하는 정적 분석 시점에, 변수는 `가능한` 값들의 집합을 가집니다.

타입명시 없이 변수를 초기화할 때 타입 체커는 지정된 값을 가지고 할당가능한 값들의 집합을 유추합니다. 이러한 과정을 `넓히기(widening)`라고 합니다.

이 과정을 이해한다면 오류의 원인을 파악하고 타입 구문을 더 효과적으로 사용할 수 있을 것입니다.

```typescript
interface Vector3 {
  x: number;
  y: number;
  z: number;
}
function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis];
}

let x = 'x';
let vec = { x: 10, y: 20, z: 30 };
getComponent(vec, x); // 오류
```

> Argument of type 'string' is not assignable to parameter of type '"x" | "y" | "z"'.

런타임에서는 문제 없지만 타입체커는 오류를 발생시킵니다.

변수 x는 할당시기에 string으로 추론되어 `"x" | "y" | "z"` 타입에 할당이 불가능 합니다.

x에 `RegExp`, `Array`, `number`등 다양한 값을 할당할 수 있지만 타입스크립트는 string으로 추론했습니다.

```typescript
const mixed = ['x', 1]; // (string|number)[]

/**
 * ('x' | 1)[]
 * ['x', 1]
 * [string, number]
 * (string|number)[]
 * [any, any]
 * ...
 * /
```

추론할 수 있는 후보가 상당히 많지만, 타입 스크립트는 명확성과 유연성 사이에서 균형을 유지하려고 합니다.

이러한 타입스크립트의 넓히기 방법을 제어할 수 있는 몇가지 방법이 있습니다.

- let 대신 const를 사용하면 더 좁은 타입이 됩니다.

```typescript
const x = 'x';
let vec = { x: 10, y: 20, z: 30 };
getComponent(vec, x); // 성공
```

- 객체는 한번에 만드는 것이 좋습니다.

```typescript
const v = {
  x: 1,
};
v.x = 3; // 성공
v.x = '3'; // 오류
v.y = 4; // 오류
v.name = 'Pythagoras'; // 오류
```

- 명시적 타입구문 사용

```typescript
const v: { x: 1 | 3 | 5 } = {
  x: 1,
};
```

- 추가적인 문맥 제공 (아이템 26)

- const 단언문 사용

```typescript
const v1 = {
  x: 1,
  y: 2,
}; // { x: number; y: number; }

const v2 = {
  x: 1 as const,
  y: 2,
}; // { x: 1; y: number; }

const v3 = {
  x: 1,
  y: 2,
} as const; // { readonly x: 1; readonly y: 2; }

const a1 = [1, 2, 3]; // number[]
const a2 = [1, 2, 3] as const; // readonly [1, 2, 3]
```

`as const` 를 사용하면 타입스크립트는 최대한 좁은 타입으로 추론합니다.

## 아이템 22. 타입 좁히기

#### null 체크

```typescript
const el = document.getElementById('foo'); // HTMLElement | null
if (el) {
  el; // HTMLElement
  el.innerHTML = 'Party Time'.blink();
} else {
  el; // null
  alert('No element #foo');
}
```

#### instanceof

```typescript
function contains(text: string, search: string | RegExp) {
  if (search instanceof RegExp) {
    search; // RegExp
    return !!search.exec(text);
  }
  search; // string
  return text.includes(search);
}
```

#### typeof

null 값 조심하기

```typescript
const el = document.getElementById('foo'); // HTMLElement | null
if (typeof el === 'object') {
  el; // HTMLElement | null
}
```

#### in

```typescript
interface A {
  a: number;
}
interface B {
  b: number;
}
function pickAB(ab: A | B) {
  if ('a' in ab) {
    ab; // A
  } else {
    ab; // B
  }
}
```

#### 내장함수

```typescript
function contains(text: string, terms: string | string[]) {
  const termList = Array.isArray(terms) ? terms : [terms];
  termList; // string[]
}
```

#### 태그된 유니온

```typescript
interface UploadEvent {
  type: 'upload';
  filename: string;
  contents: string;
}
interface DownloadEvent {
  type: 'download';
  filename: string;
}
type AppEvent = UploadEvent | DownloadEvent;

function handleEvent(e: AppEvent) {
  switch (e.type) {
    case 'download':
      e; // DownloadEvent
      break;
    case 'upload':
      e; // UploadEvent
      break;
  }
}
```

#### 커스텀 타입 가드

```typescript
function isInputElement(el: HTMLElement): el is HTMLInputElement {
  return 'value' in el;
}

function getElementContent(el: HTMLElement) {
  if (isInputElement(el)) {
    el; // HTMLInputElement
    return el.value;
  }
  el; // HTMLElement
  return el.textContent;
}
```

## 아이템 23. 한꺼번에 객체 생성하기

객체를 생성할때는 여러 속성을 포함해 한꺼번에 생성해야 타입 추론에 유리합니다.

```typescript
// Bad
const pt = {};
pt.x = 3; // 오류 ~ Property 'x' does not exist on type '{}'
pt.y = 4; // 오류 ~ Property 'y' does not exist on type '{}'

// Good
const pt = {
  x: 3,
  y: 4,
};
```

```typescript
interface Point {
  x: number;
  y: number;
}
const pt = { x: 3, y: 4 };
const id = { name: 'Pythagoras' };

// Bad
const namedPoint = {};
Object.assign(namedPoint, pt, id);
namedPoint.name; // 오류 ~ Property 'name' does not exist on type '{}'

// Good
const namedPoint = { ...pt, ...id }; // {name: string; x: number; y: number;}
```

spread 연산자를 사용하면 객체를 한꺼번에 생성하며 타입 추론도 정확히 할 수 있습니다.

```typescript
declare let hasMiddle: boolean;
const firstLast = { first: 'Harry', last: 'Truman' };
// {middle?: string; first: string; last: string;}
const president = { ...firstLast, ...(hasMiddle ? { middle: 'S' } : {}) }; 
const president2 = { ...firstLast, ...(hasMiddle && { middle: 'S' }) }; 
```

spread 연산자로 optional한 속성 추가도 할 수 있습니다.


```typescript
// {first: string; last: string;} | {middle: string; gender: string; first: string; last: string;}
const president3 = { ...firstLast, ...(hasMiddle && { middle: 'S', gender: 'M'}) }; 
president3.middle // 오류
```

하지만 한번에 여러 속성을 optional하게 추가하면 유니온 타입이 만들어집니다. middle과 gender는 항상 함께 정의되기 때문입니다.

```typescript
function addOptional<T extends object, U extends object>(a: T, b: U | null): T & Partial<U> {
  return {...a, ...b};
}

const president4 = addOptional(firstLast, hasMiddle ? {middle: 'S'} : null);
president.middle  //  string | undefined
```

유니온 대신 optional 필드를 사용하고 싶다면 이런 헬퍼 함수를 사용할 수도 있습니다.