---
title: "[Study] 리엑트 톺아보기 - Hooks"
category: react
tags:
  - javascript
  - web
  - front-end
  - study
  - react
  - rendering
  - dom
  - optimization
---

kgc4958 님께서 연재하신 [`리엑트 톺아보기`의 hooks 부분](https://goidle.github.io/react/in-depth-react-hooks_1/)을 공부하고 깨달은점, 생긴 궁금증들을 정리해 보자.


Hooks state update 및 렌더링 관련 아래 예제들의 실행 결과는 어떻게 될까?

1

```javascript
function FC() {
  const [a, setA] = useState(0) // aHook
  setA(pre => pre + 1) 
  setA(pre => pre + 1) 
  setA(pre => pre + 1) 

  console.log(state)
  // return ...
}
```

```javascript
// console
0 3 6 9 12 15 18 21....75
Error: Too many re-renders. React limits the number of renders to prevent an infinite loop.
```

연속적인 state 변경은 묶여서 업데이트 되므로 3의 배수만큼 state가 올라간다.

하지만 그 업데이트가 새로운 업데이트를 유발하므로 무한 업데이트에 빠지게 된다.

리엑트에서는 최대 업데이트 횟수를 25회로 제한하므로 75(3*25)까지 도달하였다.


2

```javascript
  // ...
  
  const [state, setState] = useState(0);
  useEffect(() => {
    setState(pr => pr + 1);
    setState(pr => pr + 1);
    setState(pr => pr + 1);
  }, []);
  
  console.log(state);
  
  // ...
```

```javascript
// console
0 3
```

최초 렌더링 시의 3번의 state update가 한데 묶여 실행되므로 컴포넌트가 한번만 재 호출된다. 


3

```javascript
  // ...
  
  const [state, setState] = useState(0);
  useEffect(() => {
    setState(0);
    setState(0);
    setState(0);
  }, []);
  
  console.log(state);
  
  // ...
```

```javascript
// console
0
```

업데이트 한 state 의 값이 원래의 값과 같으므로 최적화 과정을 거쳐 함수가 재 호출 되지 않는다.


4

```javascript
  // ...
  
  const [state, setState] = useState(0);
  if (state === 1) {
    setState(2);
    setState(3);
  }

  console.log(state);
  return <button onClick={() => setState(1)}>{state}</button>;
```

클릭시 컴포넌트 호출은 몇번이 될까?

```javascript
// console
1 3
```

버튼 클릭을 통한 업데이트로 컴포넌트가 재호출 된다. 이 시점에 추가로 발생하는 업데이트는 묶여서 한번만 실행한다.