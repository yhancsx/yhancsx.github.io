---
title: "[기술서적 리뷰] 이펙티브 타입스크립트 - 2. 타입스크립트의 타입 시스템 (아이템 13 ~ 18)"
category: js
tags:
  - javascript
  - study
  - typescript
  - interface
  - union
  - intersection
  - extends
  - mapped types
  - array
  - tuple
  - generic
---

*DAN VANDERKAM*님의 [이펙티브 타입스크립트](https://book.naver.com/bookdb/book_detail.nhn?bid=20611649)을 읽고 요악하는 포스트입니다.

## 아이템 13. 타입과 인터페이스의 차이점 알기

```typescript
type TState = {
  name: string;
  capital: string;
}
interface IState {
  name: string;
  capital: string;
}
```

객체의 타입을 정의하는 방법은 두 가지가 있습니다.

```typescript
const wyoming: TState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000 // 오류
};

type TDict = { [key: string]: string };
interface IDict {
  [key: string]: string;
}

type TFn = (x: number) => string;
interface IFn {
  (x: number): string;
}

type TPair<T> = {
  first: T;
  second: T;
}
interface IPair<T> {
  first: T;
  second: T;
}

interface IStateWithPop extends TState {
  population: number;
}
type TStateWithPop = IState & { population: number; };

class StateT implements TState {
  name: string = '';
  capital: string = '';
}
class StateI implements IState {
  name: string = '';
  capital: string = '';
}

```
잉여 속성 체크, 인덱스 시그니처, 함수 타입 정의, 제네릭, 확장, 클래스 구현 등 대부분의 기능들에서는 두 가지 모두로 작성할 수 있고 동일하게 동작합니다. 

하지만 차이점을 분명하게 알고 프로젝트 내에서 일관된 타입 정의 방법을 사용하는 것이 좋습니다.

#### 타입 키워드로 만들 수 있지만 인터페이스로는 표현할 수 없는 문법들이 있습니다.

#### 유니온 타입

```typescript
type AorB = 'a' | 'b';
```

유니온 타입은 있지만 유니온 인터페이스라는 개념은 없습니다. 

#### 유니온 타입의 확장

```typescript
type Input = { /* ... */ };
type Output = { /* ... */ };
type NamedVariable = (Input | Output) & { name: string };
```

유니온 타입에 name 속성을 붙여 타입을 확장하는 것이 필요할 수도 있지만, 인터페이스로는 표현할 수 없습니다.

#### 배열, 튜플 타입

```typescript
type Pair = [number, number];
type StringList = string[];
type NamedNums = [string, ...number[]];

interface Tuple {
  0: number;
  1: number;
  length: 2;
}
const t: Tuple = [10, 20];  // OK
```

인터페이스로도 동작하는 것처럼 보이지만 `concat`과 같은 기본 메서드들을 이용할 수 없습니다. 

#### 매핑된 타입 (Mapped Types)

```typescript
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
```

#### 조건부 타입 (Conditional Types)

```typescript
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}
type Example1 = Dog extends Animal ? number : string;
```

#### 반면, 인터페이스에는 있지만 타입에는 없는 몇가지 기능이 있습니다.

#### 선언 병합 (Declaration Merging)

```typescript
interface IState {
  name: string;
  capital: string;
}
interface IState {
  population: number;
}
const wyoming: IState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000
};  // 정상
```

이 예제처럼 속성을 확장하는 것을 `속성 병합`이라고 부르며 주로 타입 선언 파일(d.ts)에서 사용됩니다.

예를 들면, Array 인터페이스는 lib.es5.d.ts에 주로 정의되어 있지만 ES2015에 추가된 `.find()` 같은 문법은 lib.es2015.d.ts에 선언되어 있습니다. 

결과적으로 typescript는 이들을 병합해 전체 매서드를 가지는 하나의 Array 타입을 생성합니다.

#### 타입과 인터페이스 중 어느 것을 사용해야 할까요?  

복잡한 타입이라면 타입을 사용하면 됩니다. 그러나 두 가지 방법으로 모두 표현할 수 있는 간단한 객체 타입이라면 프로젝트의 일관성과 선언 병합의 관점에서 고려해야 합니다.

아직 스타일이 확립되지 않은 프로젝트라면, 향후 선언 병합의 가능성을 생각해 봐야 합니다. 어떤 API에 대한 타입 선언을 작성해야 한다면 API가 변경될 때 사용자가 선언 병합을 통해 새로운 필드를 병합할 수 있어 유용하기 때문입니다.  
-> 조금더 구체적인 예시?

그러나 3rd 라이브러리가 아닌 내부적으로 사용되는 타입에 선언 병합이 사용되는 것은 잘못된 설계일 확률이 높습니다.

## 아이템 14. 타입 연산과 제네릭 사용으로 반복 줄이기

> Don't repeat yourself (DRY)

#### 기본적인 중복 제거

```typescript
interface Point2D {
  x: number;
  y: number;
}
function distance(a: Point2D, b: Point2D) { /* ... */ }
```

```typescript
type HTTPFunction = (url: string, options: Options) => Promise<Response>;
const get: HTTPFunction = (url, options) => { return Promise.resolve(new Response()); };
const post: HTTPFunction = (url, options) => { return Promise.resolve(new Response()); };
```

```typescript
interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate extends Person {
  birth: Date;
}
```

명명된 타입, 함수 시그니처 분리, 확장 등 기본적인 문법으로 중복을 제거할 수 있습니다.


#### 객체의 일부 속성만 포함하는 타입

```typescript
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}
// Bad
interface TopNavState {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
}
// Good
type TopNavState = {
  [k in 'userId' | 'pageTitle' | 'recentFiles']: State[k]
};
// Best
type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;
```

#### 태그된 유니온의 태그 타입

```typescript
interface SaveAction {
  type: 'save';
  // ...
}
interface LoadAction {
  type: 'load';
  // ...
}
type Action = SaveAction | LoadAction;
// Bad
type ActionType = 'save' | 'load';
// Good
type ActionType = Action['type'];
```

#### 객체 내 모든 속성이 optional한 타입
```typescript
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}
// Bad
interface OptionsUpdate {
  width?: number;
  height?: number;
  color?: string;
  label?: string;
}
// Good
type OptionsUpdate = {[k in keyof Options]?: Options[k]};
// Best
type OptionsUpdate = Partial<Options>
```

#### 객체 값 타입

```typescript
const INIT_OPTIONS = {
  width: 640,
  height: 480,
  color: '#00FF00',
  label: 'VGA',
};
// Bad
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}
// Good
type Options = typeof INIT_OPTIONS;
```

'타입 공간'의 typeof 를 사용하여 정의된 객체의 값 형태에 해당하는 타입을 추출할 수 있습니다.

#### 함수 리턴 타입

```typescript
function getUserInfo(userId: string) {
  // ...

  return {
    userId,
    name,
    age,
    height,
    weight,
    favoriteColor,
  };
}
type UserInfo = ReturnType<typeof getUserInfo>;
```

ReturnType은 함수인 getUserInfo가 아니라 타입인 typeof getUserInfo에 적용되었습니다.

#### 제네릭 매개변수 제한

```typescript
interface Name {
  first: string;
  last: string;
}
type DancingDuo<T extends Name> = [T, T];

const couple2: DancingDuo<{first: string}> = [ // 오류
  {first: 'Sonny'},
  {first: 'Cher'}
];
```
> Type '{ first: string; }' does not satisfy the constraint 'Name

extends로 매개변수에 전달 될 수 있는 타입을 제한할 수 있습니다.

```typescript
type Pick<T, K extends keyof T> = {
  [k in K]: T[k]
}
type FirstMiddle = Pick<Name, 'first' | 'middle'>; // 오류
```

Pick 타입의 두번째 파라미터로 전달될 수 있는 타입은 keyof T 타입으로 제한됩니다.

## 아이템 15. 동적 데이터에 인덱스 시그니처 사용하기

```typescript
type Rocket = {[property: string]: string};
const rocket: Rocket = {
  name: 'Falcon 9',
  variant: 'v1.0',
  thrust: '4,940 kN',
};
```

이렇게 선언한 인덱스 시그니처는 문제점이 있습니다.

- 잘못된 키를 포함해도 인식할 수 없습니다. (`name` vs `Name`)
- 빈 객체 (`{}`)도 할당될 수 있습니다.
- 다른 타입의 값을 가질 수 없습니다. (`number`, `boolean` 등)
- 자동완성 등의 언어서비스를 사용할 수 없습니다.

외부 데이터 값을 받아 오는 경우(ex. 엑셀 값 읽기) 같이 정말로 객체 내에 어떤 필드가 존재할지 모르는 경우 외에는 조금 더 정확한 시그니처를 사용해야 합니다.

```typescript
// Bad
interface Row1 { [column: string]: number }
// Good
interface Row2 { a: number; b?: number; c?: number; d?: number }
// Good
type Row3 =
    | { a: number; }
    | { a: number; b: number; }
    | { a: number; b: number; c: number;  }
    | { a: number; b: number; c: number; d: number };
```

```typescript
type Vec3D = Record<'x' | 'y' | 'z', number>;
type Vec3D = {[k in 'x' | 'y' | 'z']: number};
type ABC = {[k in 'a' | 'b' | 'c']: k extends 'b' ? string : number};
```

## 아이템 16. number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

javascript의 객체에는 이상한점이 몇가지 있습니다.

```typescript
const x = {};
x[[1,2,3]] = 2;
console.log(x) // {'1,2,3': 2}
```
```typescript
const x = {1: 2, 3: 4};
console.log(x) // {'1': 2, '3': 4}
```

객체의 key로는 문자열만 사용가능합니다. (es6이후로는 symbol도 가능합니다.)

```typescript
console.log(typeof []) // 'object'
const x = [1, 2, 3]
console.log(x['1']) // 2
console.log(Object.keys(x)) // ['0', '1', '2']
```

javascript에서 배열은 객체이며 숫자 인덱스가 사용가능한것 처럼 보여도, 내부적으로는 문자열로 변환되어 사용됩니다.

typescript에서는 이러한 혼란을 바로잡기 위해 객체에 숫자 키를 허용하고 문자열 키와 구분합니다.

```typescript
interface Array<T> {
  [n: number]: T;
}
```

```typescript
const xs = [1, 2, 3];
const x0 = xs[0];  // 정상
const x1 = xs['1']; // 오류
```

런타임에는 소용이 없는 가상의 정의 이지만, 타입 체크 시점에 오류를 잡을 수 있어 유용합니다.

```typescript
const xs: Array<number> = [1, 2, 3];
const keys = Object.keys(xs);
for (const key in xs) {
  console.log(typeof key);  // 'string'
  console.log(xs[key]);  // 1, 2, 3
}
```

런타임에서는 여전히 key가 문자열로 변환됩니다.

이는 혼란을 불어일으킬 수 있습니다. 또한 push, concat등 내부 프로퍼티나 메소드를 사용할 때 오류를 발생시킬 수 있습니다.

인덱스 시그니처로 number를 사용할 일은 많지 않기 때문에 배열 타입을 선언 할 때는 인덱스 시그니처 대신 Array 또는 튜플 타입을 사용하는것이 좋습니다.

```typescript
interface ArrayLike<T> {
  readonly length: number;
  readonly [n: number]: T;
}
```

arguments, HTMLCollection 과 같은 유사 배열 객체(Array-like object)를 사용하고 싶다면 lib.es5.d.ts 정의한 ArrayLike 타입을 사용하면 됩니다. 그러나 여전히 런타임에서는 키가 string이라는 점은 주의해야 합니다.

## 아이템 17. 변경 관련된 오류 방지를 위해 readonly 사용하기

javascript에서는 `const` 변수를 선언하여도 객체와 배열의 값을 추가하거나 변경할 수 있습니다. 

```typescript
const arr: readonly number[] = [1, 2, 3];
arr.push(3); // 오류
```

> Property 'push' does not exist on type 'readonly number[]'.

`readonly` 키워드를 통해서 배열을 변경할 수 없다는 선언을 해줄 수 있습니다.

readonly number[] 타입은 number[] 타입과 몇가지 차이점이 있습니다.

- 배열의 요소를 읽을 수 있지만, 쓸 수 없습니다.
- length를 읽을 수 있지만 바꿀 수는 없습니다.
- 배열을 변경하는 pop, push등의 메소드를 호출할 수 없습니다.

```typescript
const arr: number[] = [1, 2, 3];
const arr2: readonly number[] = arr; // 정상
const arr3: number[] = arr2; // 오류
```

number[] 타입은 readonly number[] 타입의 서브타입이 됩니다. (구조적 관점에서 number[] 타입이 메소드등 기능이 더 많습니다.)

```typescript
function arraySum(arr: readonly number[]) { /* ... */ }
```

매개변수를 readonly로 선언하면 다음과 같은 일이 일어납니다.

- typescript는 함수내에서 매개변수의 변경이 일어나는지 체크합니다.
- 호출하는 쪽에서 함수가 매개변수를 변경하지 않는다는 보장을 받게 됩니다.
- 호출하는 쪽에서 함수에 readonly 배열을 매개변수로 넣을 수도 있습니다.

함수형 프로그래밍 관점에선 암묵적으로 함수내에서 매개변수를 변경하지 않는다고 가정하지만 암묵적인 방법보단 명시적인 방법을 사용하는것이 모두에게 좋습니다.

readonly는 얕게 (shallow) 동작한다는 것에 유의해야 합니다.

```typescript
const dates: readonly Date[] = [new Date()];
dates.push(new Date()); // 오류
dates[0].setFullYear(2037);  // 정상
```

객체의 readonly 배열이 있다면 배열의 변경은 불가능하지만 배열 안의 객체의 변경은 가능합니다.

```typescript
interface Outer {
  inner: {
    x: number;
  }
}
const o: Readonly<Outer> = { inner: { x: 0 }};
o.inner = { x: 1 }; // 오류
o.inner.x = 1;  // 정상
```

객체의 속성 변경을 방지하려면 Readonly 제네릭을 사용하면 됩니다. 얕게 동작한다는 점은 동일합니다.

```typescript
type Readonly<T> = { readonly [P in keyof T]: T[P]; }
```

Readonly 제네릭은 객체 인덱스 시그니처에 readonly 키워드를 선언해준 것과 동일합니다.

```typescript
let obj: {readonly [key: string]: number} = {};
obj.hi = 45; // 오류
obj = {...obj, hi: 12};  // 정상
```

객체 속성에 접근하여 값을 변경하는 것은 안되지만, 객체 자체를 재할당 하는 것은 가능합니다.

## 아이템 18. 매핑된 타입을 사용하여 값을 동기화하기

Scatter Polot을 그리는 컴포넌트를 설계해봅시다.

```typescript
interface ScatterProps {
  // The data
  xs: number[];
  ys: number[];

  // Display
  xRange: [number, number];
  yRange: [number, number];
  color: string;

  // Events
  onClick: (x: number, y: number, index: number) => void;
}
```

데이터나 디스플레이 속성이 변경되면 다시 그려야 하지만, 이벤트 핸들러가 변경되면 다시 그릴 필요가 없습니다.

```typescript
function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== 'onClick') return true;
    }
  }
  return false;
}
```

업데이트 여부를 판단하는 함수가 잘동작하도록 구현하였습니다. 하지만, 컴포넌트에 새로운 속성이 추가되었을 때 개발자가 함수 수정을 놓쳐 불필요한 리렌더링이 발생할 수 있습니다.

```typescript
const REQUIRES_UPDATE: {[k in keyof ScatterProps]: boolean} = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
};

function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      return true;
    }
  }
  return false;
}
```

매핑된 타입 `[k in keyof ScatterProps]`은 타입 체커에게 `REQUIRES_UPDATE`가 `ScatterProps`와 동일한 속성을 가져야 한다는 정보를 제공합니다.

추후 새로운 속성이 추가되거나 이름이 변경 되었을 때 `REQUIRES_UPDATE`에도 수정이 필요하다는 오류가 발생하여 개발자가 인지할 수 있습니다.