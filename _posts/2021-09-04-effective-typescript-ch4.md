---
title: '[기술서적 리뷰] 이펙티브 타입스크립트 - 4. 타입 설계'
category: js
tags:
  - javascript
  - study
  - typescript
---

*DAN VANDERKAM*님의 [이펙티브 타입스크립트](https://book.naver.com/bookdb/book_detail.nhn?bid=20611649)을 읽고 요악하는 포스트입니다.

> 누가 순서도를 보여주면서 테이블을 감춘면 나는 여전히 갸우뚱할 것이다. 하지만 테이블을 보여준다면 순서도는 별로 필요하지 않다. 보지 않더라도 명백할 것이기 때문이다 - <<맨먼스 미신(The Mythical Man Month)>> (인사이트, 2015)

연산이 이루어지는 데이터나 데이터 타입을 알 수 없다면 코드를 이해하기가 매우 어렵습니다. 데이터 타입을 명확히 알 수 있어 코드를 이해하기 쉽다는 점은 타입 시스템의 큰 장점 중 하나입니다.

## 아이템 28. 유효한 상태만 표현하는 타입을 지향하기

웹 어플리케이션에서 페이지 상태 데이터를 설계해 봅시다.

```typescript
interface State {
  pageText: string;
  isLoading: boolean;
  error?: string;
}
```

타입 `State`는 로딩중(isLoading: true)이면서 error 값이 설정되는 무효한 상태를 허용하고 있습니다.

```typescript
function renderPage(state: State) {
  if (state.error) {
    return `Error! Unable to load ${currentPage}: ${state.error}`;
  } else if (state.isLoading) {
    return `Loading ${currentPage}...`;
  }
  return `<h1>${currentPage}</h1>\n${state.pageText}`;
}
```

정보가 부족하기 때문에 `renderPge`의 분기문들이 `state` 객체의 모든 값들을 고려해야 하고 이를 빠트리기 쉽습니다.

```typescript
async function changePage(state: State, newPage: string) {
  state.isLoading = true;
  try {
    // ...
    state.isLoading = false;
    state.pageText = text;
  } catch (e) {
    state.error = '' + e;
  }
}
```

오류 발생시에 isLoading을 true로 설정하는 로직이 빠져 있지만 이를 발견하지 못할 수 있습니다.

```typescript
interface RequestPending {
  state: 'pending';
}
interface RequestError {
  state: 'error';
  error: string;
}
interface RequestSuccess {
  state: 'ok';
  pageText: string;
}
type RequestState = RequestPending | RequestError | RequestSuccess;
```

태그된 유니온 방법을 사용하여 유효한 정보들만 표현하는 타입을 선언해주는 것이 좋습니다.

```typescript
function renderPage(requestState: RequestState) {
  switch (requestState.state) {
    case 'pending':
      return `Loading...`;
    case 'error':
      return `Error! Unable to load: ${requestState.error}`;
    case 'ok':
      return `${requestState.pageText}`;
  }
}

async function changePage(requestState: RequestState) {
  requestState = { state: 'pending' };
  try {
    const response = await fetch(/* ... */);
    if (!response.ok) {
      throw new Error(`Unable to load: ${response.statusText}`);
    }
    const pageText = await response.text();
    requestState = { state: 'ok', pageText };
  } catch (e) {
    requestState = { state: 'error', error: '' + e };
  }
}
```

이제 각 상태별로 어떤 값에 접근할 수 있는지 명확해 졌고, 필요한 값이 누락되는 것을 방지할 수 있습니다.

## 아이템 29. 사용할 때는 너그럽게, 생성할 때는 엄격하게

함수의 시그니처를 정의할 때, 매개변수의 타입은 범위가 넓어도 되지만 반환되는 결과의 타입은 최대한 구체적이어야 합니다.

## 아이템 30. 문서에 타입 정보를 쓰지 않기

<<클린코드>>에서도 설명하듯 주석은 코드와 동기화되기 어려워 잘못된 정보를 나타내는 경우가 많습니다.

타입 구문은 typescript 컴파일러가 체크해서 적절한 오류를 표시해 주기 때문에 구현체와의 동기화가 어긋나지 않습니다.

```typescript
/** Does not modify nums */
function sort(nums: number[]) {
  /* ... */
}

function sort(nums: readonly number[]) {
  /* ... */
}
```

함수의 리턴 타입에 관한 내용, 파라미터는 변경되지 않는다는 내용 등 대체 가능한 주석들은 타입 구문을 통해 표현하는 것이 좋습니다.

## 아이템 31. 타입 주변에 null 값 배치하기

```typescript
function extent(nums: number[]) {
  let min, max;
  for (const num of nums) {
    if (!min) {
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num); // 오류. Argument of type 'number | undefined' is not assignable to parameter of type 'number'
    }
  }
  return [min, max];
}

const [min, max] = extent([0, 1, 2]);
const span = max - min; // 오류. Object is possibly 'undefined'.
```

값들을 다른 변수로 관리하다보니 min에 대해서만 체크하는 실수를 범했습니다. 또한 nums가 `[]`이나 `[0]`일 경우 extend의 리턴 값은 `[undefined, undefined]`입니다. 리턴 타입이 `(number \| undefined)[]`이기 때문에 함수를 사용하는 곳마다 타입 오류를 발생시킵니다.

```typescript
function extent(nums: number[]) {
  let result: [number, number] | null = null;
  for (const num of nums) {
    if (!result) {
      result = [num, num];
    } else {
      result = [Math.min(num, result[0]), Math.max(num, result[1])];
    }
  }
  return result;
}

const [min, max] = extent([0, 1, 2])!;
const span = max - min; // 정상

const range = extent([0, 1, 2]);
if (range) {
  const [min2, max2] = range;
  const span = max2 - min2; // 정상
}
```

min과 max를 한 객체로 묶고, 반환 타입이 null인 경우를 추가해 주면 훨씬 다루기 쉽습니다. 모든 값을 검사하지 못한 실수(min에 대해서만 체크)를 줄일 수 있고 반환타입이 `[number, number] | null`이 되어 null 아님 단언(`!`) 또는 null 체크 구문을 사용하면됩니다.

## 아이템 32. 유니온의 인터페이스보다는 인터페이스의 유니온 사용하기

```typescript
interface Layer {
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

아이템 28에서 설명한 것과 같이 유효하지 못한 상태를 허용하고 있습니다.

```typescript
interface FillLayer {
  type: 'fill';
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  type: 'line';
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  type: 'paint';
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

태그된 유니온을 이용하여 유효한 상태만을 표현하는것이 좋습니다.

```typescript
// Bad
interface Person {
  name: string;
  // 동시에 제공되거나 동시에 undefined이거나
  placeOfBirth?: string;
  dateOfBirth?: Date;
}

// Good
interface Person {
  name: string;
  birth?: {
    place: string;
    date: Date;
  };
}

// Best
interface Name {
  name: string;
}
interface PersonWithBirth extends Name {
  placeOfBirth: string;
  dateOfBirth: Date;
}
type Person = Name | PersonWithBirth;
```

값들이 동시에 제공되거나 동시에 undefined되는 경우엔 두 값을 하나로 모으는 방법(아이템 31)도 있지만 타입 상속(확장)과 유니온을 이용하는 것도 좋은 방법입니다.

## 아이템 33. string 타입보다 더 구체적인 타입 사용하기

string 타입을 적용하기 전에 그보다 더 좁은(정확한) 타입이 있는지 검토해 보아야 합니다.

```typescript
interface Album {
  artist: string;
  title: string;
  releaseDate: string; // YYYY-MM-DD
  recordingType: string; // "live" or "studio"
}
```

string 타입을 남용하고 주석을 통해 타입 정보를 제공하고 있습니다. 관리되기도 어렵고, 사용하는 측에서 이 주석을 찾아볼지는 불확실합니다.

```typescript
type RecordingType = 'studio' | 'live';

interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordingType;
}
```

타입의 범위를 좁히는 것은 3가지 장점을 지닙니다.

#### 1. 다른곳으로 전달되더라도 타입 정보가 유지됩니다.

#### 2. 타입을 명시적으로 정의하고 설명하는 주석을 붙여 넣으면 찾아보기 쉽습니다.

![type_description]({{site.url}}{{site.baseurl}}/assets/images/efts/type_description.png){: width="400" height="400"}

#### 3. keyof 연산자로 더욱 세밀하게 객체 속성 체크가 가능해집니다.

```typescript
function pluck<T>(record: T[], key: string): any[] {
  return record.map((r) => r[key]); // 오류
}
```

> Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'unknown'.

일반 string 타입의 키로는 객체에 접근하기에는 범위가 너무 넓습니다.

```typescript
function pluck<T, K extends keyof T>(record: T[], key: K): T[K][] {
  return record.map((r) => r[key]); // 정상
}

pluck(albums, 'releaseDate'); // 타입: Date[]
```

key 타입을 keyof를 사용한 제네릭으로 가져오면 객체 접근이 정확할 뿐만 아니라 반환 타입도 명확해집니다.

## 아이템 34. 부정확한 타입보다는 미완성 타입을 사용하기

일반적으로 타입이 구체적일수록 버그를 더 많이 잡을 수 있지만 주의를 기울여야 합니다. 실수가 발생하기 쉽고 잘못된 타입은 없는 것보다 못할 수 있기 때문입니다.

```typescript
12,
  'red',
  ['+', 10, 5], // 15
  ['case', ['>', 20, 10], 'red', 'blue'], // red
  ['/', 20, 2], // 10
  ['rgb', 255, 0, 127]; // #FF007f
```

위 모든 형태를 만족시키는 타입을 작성한다고 해봅시다.

```typescript
const tests: Expression[] = [
  10,
  'red',
  true, // 오류. boolean 불가
  ['+', 10, 5],
  ['case', ['>', 20, 10], 'red', 'blue', 'green'], // 오류. 값이 한개 더 있음
  ['**', 2, 31], // 오류. ** 연산자는 지원 불가능
  ['rgb', 255, 128, 64],
  ['rgb', 255, 0, 127, 0], // 오류. 값이 한개 더 있음
];
```

```typescript
type Expression1 = any;
type Expression2 = number | string | any[];
```

이 타입들은 너무 제너럴합니다. boolean 값 외에는 오류를 잡을 수 없습니다.

```typescript
type FnName = '+' | '-' | '*' | '/' | '>' | '<' | 'case' | 'rgb';
type CallExpression = [FnName, ...any[]];
type Expression3 = number | string | CallExpression;
```

배열 첫요소에 문자열 리터럴 타입을 사용해볼 수 있습니다. 올바르지 않은 연산자는 잡아낼 수 있지만 배열의 길이에 대한 오류를 잡아낼 수 없습니다.

```typescript
type Expression4 = number | string | CallExpression;
type CallExpression = MathCall | CaseCall | RGBCall;

interface MathCall {
  0: '+' | '-' | '/' | '*' | '>' | '<';
  1: Expression4;
  2: Expression4;
  length: 3;
}
interface CaseCall {
  0: 'case';
  1: Expression4;
  2: Expression4;
  3: Expression4;
  length: 4 | 6 | 8 | 10 | 12 | 14 | 16; // etc.
}
interface RGBCall {
  0: 'rgb';
  1: Expression4;
  2: Expression4;
  3: Expression4;
  length: 4;
}
```

배열의 길이가 정확한지 확인하기 위해 모든 종류의 타입을 작성할 수 있지만 재귀적으로 동작하기 때문에 좋은 방법은 아닙니다.

```typescript
const tests: Expression4[] = [
  10,
  'red',
  true, // Type 'boolean' is not assignable to type 'Expression4'.
  ['+', 10, 5],
  ['case', ['>', 20, 10], 'red', 'blue', 'green'], // Type '[">", number, number]' is not assignable to type 'string'
  ['**', 2, 31], // Type 'number' is not assignable to type 'string'.
  ['rgb', 255, 128, 64],
  ['rgb', 255, 0, 127, 0], // Type 'number' is not assignable to type 'string'.
];
```

유효하지 않은 값들은 잡아냈지만 오류의 메시지가 이해하기 힘듭니다.

타입 정보가 정밀해졌지만 결과적으로 개선되었다고 보기는 어렵습니다. typescript 개발에서 언어 서비스의 올바른 사용도 타입 체크 못지않게 중요한 부분입니다. 너무 복잡한 경우들을 가지는 상황에서는 정확한 타입을 작성하려하기 보다 테스트 코드를 추가하여 놓치는 부분이 없는지 확인하는 것이 더 좋을 수 있습니다.

## 아이템 35. 데이터가 아닌, API와 명세를 보고 타입 만들기

잘 설계된 타입은 typescript 사용을 즐겁게 해주는 반면, 잘못 설계된 타입은 더 큰 문제를 불러옵니다.

이런 상황에서 타입을 직접 작성하지 않고 자동으로 생성할 수 있는 방안이 있다면 사용하는것이 좋습니다.

1. [DefinitelyTyped](https://definitelytyped.org/)에 저장된 타입은 다운받아 사용가능합니다

2. [Graphql Code Generator](https://github.com/dotansimha/graphql-code-generator)  
   typescript와 비슷한 타입 시스템을 사용하는 GraphQL API는 쿼리를 typescript 타입으로 자동 추출해주는 도구를 사용하는것이 좋습니다.

3. [quickType](https://quicktype.io/) 같은 도구로 데이터로부터 타입을 생성할 수 있지만 완벽하게 일치하지 않을 수 있습니다.

4. 브라우저 DOM API에 대한 타입 선언은 IDE에 포함되어 있기 때문에 잘 사용하면 됩니다.

## 아이템 36. 해당 분야의 용어로 타입 이름 짓기

> There are only two hard things in Computer Science: cache invalidation and naming things. -- Phil Karlton

값 공간에서의 변수 이름을 정확하게 짓는 것이 중요하다는 것은 모두가 알고 있습니다. 타입 구문도 남들과 공유하고 협업해야 하는 코드의 일부이기 때문에 가독성을 위해, 또는 오해의 여지를 줄이기 위해 정확한 이름을 지어주는 것이 좋습니다.

한 개념에 한 단어를 사용하고, 불용어(-data, -info)는 피하고, 해당 도메인의 용어들을 가져오는 등 좋은 변수명을 위한 일반적인 기법들을 사용하면 좋습니다.

## 아이템 37. 공식 명칭에는 상표(brand) 붙이기

```typescript
interface Vector2D {
  x: number;
  y: number;
}
function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm({ x: 3, y: 4 }); // 정상
const vec3D = { x: 3, y: 4, z: 1 };
calculateNorm(vec3D); // 정상
```

구조적 타이핑 특성 때문에 가끔 잘못된 코드에서 오류를 잡아낼 수 없는 문제가 있었습니다.

3차원 벡터를 허용하지 않기 위해 typescript 상의 상표(brand)를 붙이는 방법을 사용할 수 있습니다.

```typescript
interface Vector2D {
  _brand: '2d';
  x: number;
  y: number;
}
function vec2D(x: number, y: number): Vector2D {
  return { x, y, _brand: '2d' };
}
function calculateNorm(p: Vector2D) {
  /**/
}

const vec3D = { x: 3, y: 4, z: 1 };
calculateNorm(vec3D); // 오류
```

> Argument of type '{ x: number; y: number; z: number; }' is not assignable to parameter of type 'Vector2D'.

vec3D에 악의적으로 \_brand 속성을 추가하는 것은 막을 수 없지만 실수를 방지하기에는 충분합니다.

number, string 같은 원시타입에서도 사용할 수 있습니다.

```typescript
type AbsolutePath = string & { _brand: 'abs' };
function listAbsolutePath(path: AbsolutePath) {
  // ...
}
function isAbsolutePath(path: string): path is AbsolutePath {
  return path.startsWith('/');
}
function f(path: string) {
  if (isAbsolutePath(path)) {
    listAbsolutePath(path);
  }
  listAbsolutePath(path); // 오류
}
```

string과 객체의 intersection 타입은 값 공간에서 존재할 수 없지만 타입 공간에서는 상표 기법으로서 유용하게 사용됩니다.

타입 시스템에서는 string이 절대 경로인지(`/`로 시작하는지) 판단하기 어렵습니다. 하지만 상표기법과 사용자정의 타입가드(user-defined type guard) 를 사용하여 타입을 좁힐 수 있습니다.

```typescript
type SortedList<T> = T[] & { _brand: 'sorted' };

function isSorted<T>(xs: T[]): xs is SortedList<T> {
  for (let i = 1; i < xs.length; i++) {
    if (xs[i] > xs[i - 1]) {
      return false;
    }
  }
  return true;
}

type Meters = number & {_brand: 'meters'};
type Seconds = number & {_brand: 'seconds'};

const meters = (m: number) => m as Meters;
const seconds = (s: number) => s as Seconds;

const oneKm = meters(1000);  // 타입: Meters
const oneMin = seconds(60);  // 타입: Seconds
```

타입 시스템 내에서 표현할 수 없는 속성들을 모델링하는데 유용하게 사용할 수 있습니다.
