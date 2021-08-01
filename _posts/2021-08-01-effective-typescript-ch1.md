---
title: "[기술서적 리뷰] 이펙티브 타입스크립트 - 1. 타입스크립트 알아보기"
category: js
tags:
  - javascript
  - study
  - typescript
  - any
  - duck typing
---

*DAN VANDERKAM*님의 [이펙티브 타입스크립트](https://book.naver.com/bookdb/book_detail.nhn?bid=20611649)을 읽고 요악하는 포스트입니다.

## 아이템 1. 타입스크립트와 자바스크립트의 관계 이해하기

typescript는 javascript의 상위 집합(superset)입니다.

![superset]({{site.url}}{{site.baseurl}}/assets/images/efts/efts_superset.png){: width="400" height="400"}

모든 javascript 프로그램이 typescript이지만, 그 반대는 성립하지 않습니다.

```typescript
function greet(who: string) {
  console.log("Hello, ", who);
}
```

이는 유효한 typescript 프로그램이지만, javascript를 구동하는 node와 같은 프로그램으로 실행하면 오류를 출력합니다.

> SyntaxError: Unexpected token :

typescript의 목표 중 하나는 런타임에 오류를 발생시킬 코드를 미리 찾아내는것 입니다. typescript가 **정적** 타입 시스템이라는 것은 바로 이런 특징을 말하는것 입니다.

```javascript
const states = [
  { name: "Alabama", capital: "Montogomery" },
  { name: "Alask", capital: "Juneau" },
  { name: "Alabama", capital: "Phoenix" },
];

for (const state of states) {
  console.log(state.capitol);
}

// undefined
// undefined
// undefined
```

위 코드는 어떠한 오류도 없이 실행됩니다.

그러나 typescript 타입 체커는 타입 구문 없이도 오류를 찾아 냅니다.

> Property 'capitol' does not exist on type '{ name: string; capital: string; }'. Did you mean 'capital'?

```javascript
const states = [
  { name: "Alabama", capital: "Montogomery" },
  { name: "Alask", capitol: "Juneau" },
  { name: "Alabama", capital: "Phoenix" },
];

for (const state of states) {
  console.log(state.capital);
}

// Montogomery
// undefined
// Phoenix
```

위와 같이 딱 한번만 `capitol` 이라고 오타를 썼다면 typescript도 오류를 찾아내지 못합니다.

그런데 타입 구문을 추가하면 오류를 찾을 수 있습니다.

```typescript
type State = { name: string; capital: string };

const states: State[] = [
  { name: "Alabama", capital: "Montogomery" },
  { name: "Alask", capitol: "Juneau" },
  { name: "Alabama", capital: "Phoenix" },
];
```

> Type '{ name: string; capitol: string; }' is not assignable to type 'State'.

typescript의 타입 시스템은 javascript의 런타임 동작을 **모델링**합니다.

```typescript
const x = 2 + "3"; // "23"
const y = "2" + 3; // "23"
```

이는 javascript에서 동작하는 코드이기 때문에 typescript의 타입 체커도 정상으로 인식합니다.

반면, 정상 동작하는 코드인데 오류를 표시하기도 합니다.

```typescript
const a = null + 7; // Operator '+' cannot be applied to types 'null' and '7'.
const b = [] + 7; // Operator '+' cannot be applied to types 'undefined[]' and 'number'
window.alert("hello", "typescript"); // Expected 0-1 arguments, but got 2.
```

typescript의 타입 시스템은 javascript의 런타임 동작을 **모델링**하는 것이 기본 원칙이지만, 의도치 않은 이상한 코드가 오류로 이어질 수도 있습니다.

작성한 프로그램이 타입체크를 통과하더라도 여전히 런타임에 오류가 발생할 수 있습니다.

```typescript
const names = ["alice", "bob"];
console.log(names[2].toUpperCase());
```

> Cannot read property 'toUpperCase' of undefined

any 타입을 사용할 때도 예상치 못한 오류가 발생합니다.

타입 시스템이 정적 타입의 정확성을 보장해 줄 것 같지만, 그렇지 않은 경우도 있습니다.

## 아이템 2. 타입스크립트 설정 이해하기

```typescript
function add(a, b) {
  return a + b;
}
add(10, null);
```

위 코드가 오류없이 타입 체커를 통과할 수 있을지 여부는 typescript 설정을 알기 전까지는 대답할 수 없습니다.

`tsc --init`을 실행하여 간단히 설정 파일 _tsconfig.json_ 을 생성할 수 있습니다.

#### noImplicitAny

```typescript
function add(a, b) {
  return a + b;
}
```

noImplicitAny가 해제되어 있을때 typescript가 추론한 타입을 보면 (add에 마우스를 올려보면 알 수 있습니다)

> function add(a: any, b: any): any  
> Parameter 'a' implicitly has an 'any' type, but a better type may be inferred from usage.

any를 코드에 넣지 않았지만, any 타입으로 간주되기 때문에 **implicit**이라는 단어를 사용합니다.

같은 코드임에도 noImplicitAny가 설정되어 있다면 오류가 됩니다.

any 타입이라고 선언해 주거나 더 분명한 타입을 사용하면 해결할 수 있습니다.

```typescript
function add(a: number, b: number) {
  return a + b;
}
```

새 프로젝트를 시작한다면 처음부터 noImplicitAny를 설정하여 코드를 작성할때마다 타입을 명시하도록 해야 합니다. noImplicitAny 설정 해제는 javascript로 되어있는 기존 코드를 typescript로 전환하는 상황에서만 필요합니다.

#### strictNullCheckes

strictNullChecke는 null과 undefined가 모든타입에서 허용되는지 확인하는 설정입니다.

```typescript
const x: number = null;
```

위 코드는 strictNullChecks가 해제되었을 때 유효합니다.

```typescript
const x: number = null;
```

> Type 'null' is not assignable to type 'number'.

strictNullChecks가 설정되었다면 오류가 됩니다.

```typescript
const x: number | null = null;
```

null을 허용하려고 한다면 의도를 명시적으로 드러내야 합니다.

noImplicitAny와 마찬가지로 프로젝트 상황에 따라 설정 여부를 결정해야 합니다. 해제 상태로 개발하기로 했다면, "_undefined is not an object_" 라는 런타임 오류를 많이 마주하게 될것입니다.

#### strict

noImplicitAny, strictNullCheckes 를 포함해 언어에 의미적으로 영향을 미치는 설정들 모두를 체크하고 싶다면 `strict: true` 설정을 하면 됩니다.

> The strict flag enables a wide range of type checking behavior that results in stronger guarantees of program correctness. ([ref](https://www.typescriptlang.org/tsconfig#strict))

## 아이템 3. 코드 생성과 타입이 관계없음을 이해하기

typescript compiler는 두가지 역할을 수행합니다.

- 최신 typescript/javascript를 브라우저에서 동작할 수 있도록 구버전의 javascript로 트랜스파일(transpile)합니다.
- 코드의 타입 오류를 체크합니다.

위 두가지 역할은 완벽히 독립적입니다. typescript 코드가 javascript로 변환될 때 코드 내의 타입에는 영향을 주지 않습니다. 생성된 javascript의 실행 시점에도 타입은 영향을 미치지 않습니다.

#### 타입 오류가 있는 코드도 컴파일이 가능합니다.

```typescript
let x = "hello";
x = 1234;
```

> Type 'number' is not assignable to type 'string'

위 오류는 타입 체크 단계에서 오류를 발생시키지만 정상적으로 컴파일이 가능합니다.

*tsconfig.json*의 noEmitOnError를 설정하여 오류가 있을때 컴파일을 멈출 수 있습니다.

#### 런타임에는 타입체크가 불가능합니다.

javascript로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 제거됩니다.

**from**

```typescript
function add(a: number, b: number): number {
  return a + b;
}
```

**to**

```javascript
function add(a, b) {
  return a + b;
}
```

#### 타입 연산은 런타임에 영항을 주지 않습니다.

**from**

```typescript
function asNumber(val: number | string): number {
  return val as number;
}
```

**to**

```javascript
function asNumber(val) {
  return val;
}
```

타입 체커를 통과하는 코드를 작성하였지만 javascript로 변환되면 아무 영향도 미치지 못합니다.

```typescript
function asNumber(val: nubmer | string): number {
  return typeof val === "string" ? Number(val) : val;
}
```

런타임에서 타입을 체크하고 값을 변환하는 javascript 연산을 사용해야 합니다.

#### 런타임 타입은 선언된 타입과 다를 수 있습니다.

```typescript
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      turnOn();
      break;
    case false:
      turnOff();
      break;
    default:
      console.log("실행이 될까요?");
  }
}
```

typescript는 일반적으로 죽은(dead) 코드를 찾아내지만, 여기서는 찾아내지 못합니다.

`: boolean`은 타입 선언문이기 때문에 javascript로 변환되면 제거됩니다. `value`로 `"on"`과 같은 값이 들어올 수도 있습니다.

#### 타입스크립트 타입으로는 함수를 오버로드할 수 없습니다.

```typescript
function add(a: number, b: number): number {
  return a + b;
}
function add(a: string, b: string): string {
  return a + b;
}
```

> Duplicate function implementation

typescript가 함수 오버로딩 기능을 지원하기는 하지만, 온전히 타입 수준에서만 동작합니다. 하나의 함수에 대해 여러개의 선언문을 작성할 수 있지만, 구현체는 하나입니다.

```typescript
function add(a: number, b: number): number;
function add(a: string, b: string): number;
function add(a, b) {
  return a + b;
}
```

#### 타입스크립트 타입은 런타임 성능에 영향을 주지 않습니다.

javascript로 컴파일되는 과정에서 모든 타입과 타입 연산자는 제거되기 때문에, 런타임 성능에 영향을 주지 않습니다.

런타임 오버헤드 대신 **빌드타임** 오버헤드가 있습니다. 오버헤드가 커지면 빌드도구에서 타입 체크를 건너 뛰고 트랜스파일만 가능하게 할 수 있습니다.

## 아이템 4. 구조적 타이핑에 익숙해지기

javascript는 본질적으로 덕 타이핑(duck typing) 기반입니다.

> Duck Typing은 한 객체(Object)가 특정한 목적에 적합한지 판별하는 데 관한 것이다. 일반적인 타이핑의 경우, 객체의 적합성은 오로지 객체의 타입의 따라 정해진다. Duck Typing에서는 실제 객체의 타입보다는, 객체의 (알맞은 의미를 가진) 메서드와 속성들의 존재 여부로 적합성을 판단한다. ([ref](https://medium.com/spacewalk-blog/javascript-duck-typing-5dfbb7d26769))

```typescript
interface Vector2D {
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x ** 2 + v.y ** 2);
}
```

```typescript
interface NamedVector {
  name: string;
  x: number;
  y: number;
}

const v: namedVector = { x: 3, y: 4, name: "Zee" };
calculateLength(v);
```

NamedVector는 number 타입의 x, y 속성이 있기 때문에 calculateLength 함수로 호출 가능합니다.

NamedVector의 구조가 Vector2D와 호환되기 때문에 NamedVector와 Vector2D의 관계를 전혀 선언하지 않아도 됩니다. 여기서 구조적 타이핑(structural typing)이라는 용어가 사용됩니다.

```typescript
interface Vector3D {
  x: number;
  y: number;
  z: number;
}

function normalize(v: Vector3D) {
  const len = calculateLength(v);
  return {
    x: v.x / len,
    y: v.y / len,
    z: v.z / len,
  };
}
```

이는 구조적 타이핑 때문에 발생하는 문제 중 하나 입니다. Vector3D로 calculateLength를 호출하여도 x, y 값이 있어 Vector2D와 호환되기 때문에 타입 체커가 문제로 인식하지 못합니다. typescript는 타입에 대해 봉인(sealed)되어 있지 않고 열려(open)있습니다.

```typescript
function calculateLengthL1(v: Vector3D) {
  let length = 0;
  for (const axis of Object.keys(v)) {
    const coord = v[axis];
    length += Maht.abs(coord);
  }
  return length;
}
```

> Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'Vector3D'.

axis 는 Vector3D 타입의 키 중 하나이기 때문에 x, y, z 뿐일 것 같지만, typescript는 타입에 대해 열려 있으므로 v로 `{ x: 3, y: 4, z: 1, address: '512-312' }` 등이 전달 될 수 있습니다.

## 아이템 5. any 타입 지양하기

typescript의 타입 시스템은 1) 코드에 타입을 조금씩 추가할 수 있기 때문에 점진적(gradual)이며, 언제든지 타입 체커를 해제할 수 있기 때문에 선택적(optional)입니다.

이 기능들의 핵심은 any 타입 입니다.

부득이하게 any를 사용하더라도 위험성을 알고 있어야 합니다.

#### any 타입에는 타입 안정성이 없습니다.

```typescript
let age: number;
age = "12" as any;
age += 1; // "123"
```

`as any`를 사용하여 number 타입에 string 타입을 할당할 수 있게 됩니다.

#### any는 함수 시그니처를 무시해 버립니다.

함수는 작성된 시그니처에 따라 적절한 입력을 받아들이고 약속된 타입의 결과를 반환해야 합니다.

```typescript
function calculateAge(birthDate: Date): number {
  // ...
}
let birthDate: any = "1990-01-19";
calculateAge(birthDate);
```

any 타입 사용으로 위 코드에서 오류를 발견할 수 없습니다.

#### any 타입에는 언어 서비스가 적용되지 않습니다.

any 타입을 사용할 땐 IDE 자동완성 기능이나, 이름 변경(rename symbol) 기능을 사용할 수 없습니다.

이는 개발자의 생산정 저하로 이어집니다.

#### any 타입은 코드 리팩토링 때 버그를 감춥니다.

```typescript
interface ComponentProps {
  onSelectItem: (item: any) => void;
}

function SomeComponent(props: ComponentProps) {
  /*..*/
}

function handleSelectItem(item: any) {
  selectedId = item.id;
}

renderSelector({ onSelectItem: handleSelectItem });
```

위 컴포넌트의 onSelectItem 콜백에 전달되는 파라미터에 아이템 객체에서 필요한 부분만 전달하도록 개선하는 경우를 생각해 보겠습니다.

```typescript
interface ComponentProps {
  onSelectItem: (id: number) => void;
}
```

타입 체크는 통과되었지만 handleSelectItem는 any 파라미터를 전달 받습니다. 그래서 id를 전달받아도 문제가 없다고 나옵니다. 이는 런타임 오류로 이어집니다.

#### any는 타입 설계를 감춰버립니다.

어플리케이션 상태 같은 복잡한 객체를 정의할 때 수많은 속성의 타입을 일일히 작성하는 것이 번거로오 any 타입으로 간단히 끝내버릴 유혹을 받을 수 도 있습니다.

이는 객체의 설계를 감춰버려 동료들이 코드를 검토할 때 상태가 어떻게 변경되는지 이리저리 다니며 찾아야 합니다.

#### any는 타입시스템의 신뢰도를 떨어뜨립니다.

타입 체커의 역할은 사람의 실수를 잡아주고 코드의 신뢰도를 높여주는 것입니다. any 타입을 사용하여 런타임에 오류가 발생하면 타입 체커를 신뢰할 수 없게 됩니다.
