---
title: "[기술서적 리뷰] 이펙티브 타입스크립트 - 2. 타입스크립트의 타입 시스템 (아이템 6 ~ 12)"
category: js
tags:
  - javascript
  - study
  - typescript
  - union
  - Intersection
  - vscode
  - typeof
  - instanceof
---

*DAN VANDERKAM*님의 [이펙티브 타입스크립트](https://book.naver.com/bookdb/book_detail.nhn?bid=20611649)을 읽고 요악하는 포스트입니다.

## 아이템 6. IDE를 사용하여 타입 시스템 검색하기

typescript를 설치하면 두가지 기능을 이용할 수 있습니다.

- typescript 컴파일러(tsc)
- 단독으로 실행할 수 있는 typescript 서버(tsserver)

typescript 서버는 `언어 서비스`를 제공한다는 점에서 중요합니다.

- 코드 자동 완성
- 명세(specification) 검사
- 검색
- 리팩터링

이외에도 언어 서비스에는 많은 기능이 포함되어 있으며 [VS Code](https://code.visualstudio.com/docs/languages/typescript)를 사용한다면 별다른 설정없이 사용할 수 있습니다.

## 아이템 7. 타입이 값들의 집합이라고 생각하기

런타임에 실행되는 자바스크립트에서 모든 변수에는 `42`, `null`, `undefined`, `(x,y) => x + y`, `{name: 'yhan', gender: 'M'}` 등 다양한 종류의 값을 할당할 수 있습니다.

그러나 코드가 실행되기 전 typescript가 오류를 체크하는 순간에는 `타입`을 가지고 있습니다. 타입은 `할당 가능한 값들의 집합`이라고 생각할 수 있습니다.

```typescript
const n: number = 3.75; // 정상
const n: number = "hello"; // 오류
const s: string = 42; // 오류
const x: never = 12; // 오류
```

예를 들면, 모든 숫자값의 집합을 number 타입, 모든 문자열 집합을 string 타입, 아무 값도 포함하지 않는 공집합은 never 타입입니다.

```typescript
type A = "A";
```

공집합 다음으로 작은 집합은 한가지 값만 포함하는 타입으로, 유닛(unit) 타입 또는 리터럴(literal) 타입이라고 불립니다.

```typescript
type AB = "A" | "B";
```

여러개를 유니온(union) 타입으로 묶어서 사용할 수도 있습니다.

```typescript
const c: AB = "C"; // 오류
```

> Type '"C"' is not assignable to type 'AB'

`A is assignable to B`의 의미는 집합의 관점에서 `A는 B의 원소` 또는 `A는 B의 부분 집합` 이라는 의미입니다. 타입 체커의 주요 역할은 하나의 집합이 다른 집합의 부분 집합인지 검사하는 것이라고 볼 수 있습니다.

```typescript
interface Identified {
  id: string;
}
```

위 인터페이스가 값들의 집합이라고 생각해보면 string 형태의 id 속성을 가지고 있는 모든 객체는 있다면 Identified 타입입니다. (구조적 타이핑)

```typescript
interface Person {
  name: string;
}
interface Lifespan {
  birth: Date;
  death?: Date;
}
type PersonSpan = Person & Lifespan;
```

언뜻 보기에 Person과 Lifespan의 &연산 (Intersection, 교집합)은 공통으로 가지는 속성이 없기에 공집합(never)라고 생각하기 쉽습니다.

```typescript
// Person에 속하는 값 집합
// {name: 'yhan'}, {name: 'yhan', birth: '1993-12-25'}, {name: 'yhan', gender: 'M'},
// {name: 'yhan', birth: '1993-12-25', death: '2093-12-25'}, ...

// Lifespan 속하는 값 집합
// {birth: '1993-12-25'}, {name: 'yhan', birth: '1993-12-25'}, {birth: '1993-12-25', gender: 'M'},
// {name: 'yhan', birth: '1993-12-25', death: '2093-12-25'}, ...

// 두 집합의 교집합 == PersonSpan
// {name: 'yhan', birth: '1993-12-25'}, {name: 'yhan', birth: '1993-12-25', death: '2093-12-25'}, ...
```

그러나 타입은 값의 집합이라는 관점에서 보면 Person과 Lifespan의 속성을 모두 가지는 값이 인터섹션 타입에 속하게 됩니다. 당연히 이외 더 많은 속성을 가지는 값들도 PersonSpan 타입에 속하게 됩니다.

```typescript
type K = keyof (Person | Lifespan); // never
```

두 인터페이스의 유니온 타입에 속하는 값은 어떠한 공통된 키도 없기 때문에 유니온에 대한 keyof는 공집합입니다. ([참고](https://github.com/Microsoft/TypeScript/issues/12948))

```typescript
interface Person {
  name: string;
}
interface PersonSpan extends Person {
  birth: Date;
  death?: Date;
}
```

일반적으로 PersonSpan 타입을 선언하는 방법은 extends 키워드를 사용하는 것입니다. 타입이 값의 집합이라는 관점에서 extends의 의미는 `PersonSpan은 Person의 부분집합` 이라는 의미로 생각할 수 있습니다.

```typescript
interface Vector1D {
  x: number;
}
interface Vector2D extends Vector1D {
  y: number;
}
interface Vector3D extends Vector2D {
  z: number;
}
```

![subtype]({{site.url}}{{site.baseurl}}/assets/images/efts/efts_subtype.png){: width="400" height="400"}

어떤 집합이 다른 집합의 부분집합이라는 의미로 `서브타입 (subtype)`이라는 용어를 사용합니다. 클래스 관점에서는 상속 관계로 그려지지만 벤 다이어그램으로 그리는 것이 부분 집합의 개념에서는 더욱 적절합니다.

- A는 B에 할당가능하다.
- A는 B의 부분집합이다.
- A는 B를 상속받았다.

typescript에서 위 3가지 문장은 모두 같은 의미를 가집니다.

타입을 집합이라는 관점에서 보면 배열과 튜플의 관계 역시 명확합니다.

```typescript
const list = [1, 2]; // 타입: number[]
const tuple: [number, number] = list; // 오류
```

number[] 타입은 [number, number] 타입의 부분집합이 아니기 때문에 할당할 수 없습니다.  (그 반대는 가능)

```typescript
const triple: [number, number, number] = [1, 2, 3];
const double: [number, number] = triple; // 오류
```

> Source has 3 element(s) but target allows only 2 (TS v4)  
> Type '3' is not assignable to type '2' (TS v3)

트리플은 구조적 타이핑 관점에서 튜플로 할당 가능할 것처럼 보이지만 오류가 발생합니다.

```typescript
type len2 = typeof double["length"]; // 2
type len3 = typeof triple["length"]; // 3
```

typescript가 더블과 트리플 타입을 각각 길이에 맞는 length를 갖도록 모델링하여 할당할 수 없는 것입니다.

## 아이템 8. 타입 공간과 값 공간의 심벌 구분하기

`값 공간`과 `타입 공간`을 구분하여 생각해야 합니다. 타입공간에 존재하는 코드들은 javascript로 변환되며 모두 제거됩니다.

```typescript
// 타입 공간
interface Cylinder {
  radius: number;
  height: number;
}

// 값 공간
const Cylinder = (radius: number, height: number) => ({ radius, height });
```

같은 심벌 (Cylinder)라도 정의 방식에 따라 다른 공간에 존재할 수 있습니다. 상황에 따라 타입으로 쓰일 수도 있고 값으로 쓰일 수도 있습니다.

```typescript
function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape.radius; // 오류
  }
}
```

> Property 'radius' does not exist on type '{}'

instanceof는 javascript 런타임 연산자(값 공간)이므로 인터페이스가 아닌 함수 Cylinder를 참조합니다.

```typescript
type T1 = "string literal";
type T2 = 123;
const v1 = "string literal";
const v2 = 123;
```

type이나 interface 다음에 나오는 심벌은 타입 공간, const와 같은 변수 선언에 쓰이는 심벌은 값 공간에 존재한다고 할 수 있습니다.

타입 공간에 존재하는 심벌들은 javascript로 변환 시 모두 제거되기 때문에 [typescript playground](https://www.typescriptlang.org/play)에서 transpile되는 코드를 확인하며 구분해 볼 수 있습니다.

class, enum은 상황에 따라 타입과 값 두가지 모두로 사용가능합니다.

```typescript
class Cylinder {
  radius = 1;
  height = 1;
}

function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape; // 정상. 타입: Cylinder
    shape.radius; // 정상. 타입: number
  }
}
```

instanceof 구문에서는 값으로 사용되었지만, if block안에서는 타입으로 추론되었습니다.

`typeof` 연산자는 타입에서 쓰일 때와 값에서 쓰일 때 다른 기능을 합니다.

```typescript
interface Person {
  first: string;
  last: string;
}
const p: Person = { first: "Jane", last: "Jacobs" };

type T = typeof p; // 타입: Person
const v1 = typeof p; // 값: "object"
```

타입 관점에서 typeof는 값을 읽어 타입을 반환하지만, 값의 관점에서는 javascript 런타임 타입을 가리키는 문자열을 반환합니다.

```typescript
type T = typeof Cylinder; // 타입: typeof Cylinder
const v = typeof Cylinder; // 값: "function"
```

T가 Cylinder 타입이 아닌 typeof Cylinder 타입인 것은 Cylinder는 클래스의 인스턴스 타입을 가리키기 때문입니다.

```typescript
const cylinder = new Cylinder();
type T = typeof cylinder; // 타입: Cylinder
type T2 = InstanceType<typeof Cylinder>; // 타입: Cylinder
```

`InstanceType` 제네릭을 사용해 인스턴스 타입을 추출해 낼 수 있습니다.

```typescript
const first: Person['first'] = p['first'];
```

타입 공간에서 속성 접근자를 사용하여 객체 속성의 타입을 추출해 낼 수 있습니다.

그 외 타입 공간에서 값 공간과는 다른 의미를 가지는 코드 패턴이 있습니다.

- `this`: 다형성 this의 typescript 타입
- `&`, `|`: 인터섹션, 유니온 타입
- `as const`: 리터럴 표현식의 추론된 타입 변경
- `extends`: 서브타입 또는 제네릭 타입의 한정자
- `in`: 매핑된 타입

## 아이템 9. 타입 단언보다는 타입 선언을 이용하기

- 선언: Declarations
- 단언: Assertions

```typescript
interface Person {
  name: string;
}

const alice: Person = { name: "Alice" }; // 타입 선언
const bob = { name: "Alice" } as Person; // 타입 단언
```

타입 단언보다 타입 선언을 사용하는 것이 좋습니다.

```typescript
const alice: Person = {}; // 오류
const bob = {} as Person; // 정상

const alice2: Person = {
  name: "Alice",
  occupation: "Typescript developer", // 오류
};

const bob2 = {
  name: "Alice",
  occupation: "Javascript developer", // 정상
} as Person;
```

타입 선언은 할당되는 값이 typescript에서 제공하는 타입 체크, 타입 추론을 만족하는지 검사하지만 표시하지만 타입 단언은 타입 체커에게 _"내가 무슨 일을 하는지 알고 있으니"_ 오류를 무시하라고 말하는 것입니다.

```typescript
// Bad
const people = ["alice", "bob", "jane"].map((name) => ({ name } as Person));

// Good
const people2 = ["alice", "bob", "jane"].map((name) => {
  const person: Person = { name };
  return person;
});

// Best
const people3 = ["alice", "bob", "jane"].map((name): Person => ({ name }));
const people4: Person[] = ["alice", "bob", "jane"].map((name): Person => ({ name }));
```

위 처럼 타입 단언을 사용한 케이스(Bad)는 속성을 잘못 삽입할 때와 같은 오류를 검출 할 수 없으니 선언을 사용해야 합니다.

```typescript
document.querySelector("#myButton").addEventListener("click", (e) => {
  e.currentTarget; // EventTarget
  const button = e.currentTarget as HTMLButtonElement; // HTMLButtonElement
  return button;
});

const el = document.querySelector("#myInput"); // Element | null
const checked = (el as HTMLInputElement).checked;
```

DOM API에 접근하는 등 typescript의 타입 체커가 추론한 타입보다 개발자가 판단하는 타입이 더 정확한 경우에는 타입 단언이 필요합니다.

```typescript
const el = document.querySelector("#myInput")!; // Element
```

접미사 `!`를 이용해 null이 아님을 알려주는 문법도 타입 단언의 일종입니다.

```typescript
const body = document.body;
const el = body as Person; // 오류
const el2 = body as unknown as Person; // 정상
```

> Conversion of type 'HTMLElement' to type 'Person' may be a mistake because ...

타입 단언은 항상 사용할 수 있는 것은 아니며, A가 B의 부분집합인 경우에만 사용할 수 있습니다.
`Element` 타입은 `Element | null` 타입의 부분집합이기 때문에 변환이 가능했습니다.

모든 타입은 (unknown의 부분 집합이기 때문에) unknown으로 변환 가능하며 unknown 단언은 또 다른 타입으로의 변환을 가능하게 합니다.

## 아이템 10. 객체 래퍼 타입 피하기

javascript에는 객체 외에 string, number, boolean, null, undefined, symbol, bigint 7개의 원시(primitive) 타입이 있습니다.

이들은 불변이며 메서드를 가지지 않는다는 점에서 객체와 차이가 있습니다.

```typescript
const str = "primitive";
const char3 = str.charAt(3);
```

그런데 원시 타입들 또한 메서드를 가지고 있는 것처럼 보입니다.

하지만 사실은 javascript 내부적으로 String 객체로 변환하는 작업이 일어나기 때문입니다.

- String 객체로 변환 (래핑)
- 메서드 호출
- 변환한 객체 폐기

```typescript
"hello" === new String("hello"); // false
new String("hello") === new String("hello"); // false
new String("hello") == new String("hello"); // false
```

String은 객체이기 때문에 reference가 같은 자신과만 동일합니다.

```typescript
const str = "hello";
str.language = "English";
console.log(str.language); // undefined
```

메서드, 프로퍼티로 접근할 때 생성되는 래퍼 객체는 작업 완료후 폐기 되기 때문에 변경사항을 저장할 수 없습니다.

Number, Boolean, Symobl, BigInt 같은 다른 래퍼 객체들도 동일한 성격을 가지고 있습니다.

```typescript
function getStringLen(foo: String) {
  return foo.length;
}

getStringLen("hello"); // 정상
```

String 타입을 매개변수로 받는 함수에 string 타입을 전달하여도 문제 없이 보일 수 있지만,

```typescript
function isGreeting(phrase: String) {
  return ["hello", "good day"].includes(phrase); // 오류
}
```

> Argument of type 'String' is not assignable to parameter of type 'string'. ...

string을 매개변수로 받는 함수에 String을 전달한다면 오류가 발생합니다. <ins>string은 String에 할당할 수 있지만 String은 string에 할당할 수 없습니다.</ins>

굳이 String(래퍼) 타입을 선언할 필요 없이 string(원시) 타입을 사용하는 것이 좋습니다.

## 아이템 11. 잉여 속성 체크의 한계 인지하기

```typescript
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}

const room: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present", // 오류
};
```

> Type '{ numDoors: number; ceilingHeightFt: number; elephant: string; }' is not assignable to type 'Room'.

아이템4에서 다룬 구조적 타이핑 관점에서 생각해보면 오류가 발생하지 않아야 합니다.

```typescript
const tmpRoom = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};

const room: Room = tmpRoom; // 정상
```

임시 변수를 사용하면 오류가 발생하지 않습니다. tmpRoom의 타입은 Room 타입의 부분집합이므로 할당 가능하며 타입체커도 통과합니다.

두 예제의 차이점은 `잉여 속성 체크` 단계 입니다. 첫 번째 예제에서는 구조적 타입 시스템에서 발생할 수 있는 실수나 오류를 잡을 수 있도록 `잉여 속성 체크`가 수행되었습니다. 이는 `할당 가능 검사`와는 별개로 진행됩니다.

typescript는 런타임에 에러가 발생할 상황을 미리 감지해 오류를 표시하는 것 뿐만 아니라 의도와 다르게 작성된 코드를 찾으려고 합니다.

```typescript
interface Options {
  title: string;
  darkMode?: boolean;
}

function createWindow(options: Options) {
  if (options.darkMode) {
    // ...
  }
}

createWindow({ title: "Spider Solitaire", darkmode: true }); // 오류
```

> Argument of type '{ title: string; darkmode: boolean; }' is not assignable to parameter of type 'Options'.

이 코드를 실행하면 런타임엔 아무 에러도 발생하지 않지만 의도한 대로 동작하지 않을 수 있습니다. (darkmode vs darkMode)

```typescript
const o1: Options = document; // 정상
const o1: Options = new HTMLButtonElement(); // 정상
```

두 인스턴스 모두 title 송석을 가지고 있기 때문에 할당문은 정상입니다.

이와 같이 순수한 구조적 타입 체커만으로는 엄청나게 넓은 범위를 가진 Options에 대해 적절한 오류를 찾아주지 못할 수 있습니다.

`잉여 속성 체크`를 이용하면 typescript 시스템의 구조적 본질을 해치지 않으면서도 객체 리터럴에 알 수 없는 속성을 허용하지 않음으로써 의도와 다르게 작성된 코드를 방지할 수 있습니다.

중요한점은 `객체 리터럴`에서만 작동한다는 것입니다. `document`, `new HTMLButtonElement` 등은 객체 리터럴이 아니기 때문에 `잉여 속성 체크`가 동작하지 않았습니다.

```typescript
interface Options {
  darkMode?: boolean;
  [otherOptions: string]: unknown;
}

const o: Options = { darkmode: true }; // 정상
```

`잉여 속성 체크`를 원하지 않는 상황이라면 인덱스 시그니처를 사용하여 타입 체커가 추가적인 속성을 예상하도록 할 수 있습니다.

```typescript
interface LineChartOptions {
  logscale?: boolean;
  invertedYAxis?: boolean;
}
const opts = { logScale: true };
const o: LineChartOptions = opts; // 오류
```

> Type '{ logScale: boolean; }' has no properties in common with type 'LineChartOptions'.

optional한 속성만 가지는 약한(week) 타입에는 `공통 속성 체크`라는 비슷한 검사가 수행됩니다.

구조적 관점에서 LineChartOptions 타입은 모든 객체를 포함 할 수 있습니다. 이런 타입에 대해서는 할당 가능성 보단 작성자의 의도와 다른 오타를 검사하는 것이 효과적입니다.

`잉여 속성 체크`와는 다르게 임시변수와 관련 없이 관련된 할당문마다 수행됩니다.

## 아이템 12. 함수 표현식에 타입 적용하기

- 문장식: Statement
- 표현식: Expression

```typescript
function rollDice1(sides: number): number {
  return 0;
} // 문장식
const rollDice2 = function (sides: number): number {
  return 0;
}; // 표현식
const rollDice3 = (sides: number): number => {
  return 0;
}; // 표현식
```

typescript에서는 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 재사용하기 위해 함수 표현식을 사용하는 것이 좋습니다.

```typescript
type DiceRollFn = (sides: number) => number;
const rollDice: DiceRollFn = (sides) => {
  return 0;
};
```

rollDice 함수의 매개변수와 반환 값의 타입을 추론할 수 있습니다.

```typescript
function add(a: number, b: number) {
  return a + b;
}
function sub(a: number, b: number) {
  return a - b;
}
function mul(a: number, b: number) {
  return a * b;
}
function div(a: number, b: number) {
  return a / b;
}

type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

함수 문장식과는 달리 함수 표현식은 시그니처를 하나의 타입으로 통합하여 재사용할 수 있습니다.

```typescript
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error("Request failed: " + response.status);
  }
  return response;
}
```

함수의 매개변수, 반환 타입을 일일히 지정해 줄 수도 있지만,

```typescript
// lib.dom.ds.ts
declare function fetch(input: RequestInfo, init?: RequestInit): Promise<Response>;

const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error("Request failed: " + response.status);
  }
  return response;
};
```

시그니처가 일치하는 다른 함수가 있다면 타입을 가져와 함수표현식에 적용하면 더 간결한 코드를 작성할 수 있습니다.
