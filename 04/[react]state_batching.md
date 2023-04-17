# state

리액트 새 공식문서로 혼란스러웠던 부분 학습

## 기억할 것

- 상태는 렌더링 당시의 스냅샷이다.
- 리액트는 상태 업데이트 전 이벤트 핸들러의 모든 코드가 실행될 때 까지 기다린다.
- useState는 기존 렌더링 내에서 변수를 변경하는것이 아니라 새로운 렌더링을 요청한다.
- state 변수를 설정하면 다음 렌더링이 큐에 들어감(일단 기억해두삼)

## 배칭!!(일괄 처리)

```jsx
const [num, setNum] = useState(0);

const batching = () => {
  setNum(num + 1);
  setNum(num + 1);
  setNum(num + 1);
};
```

이 코드는

```jsx
setNum(0 + 1);
setNum(0 + 1);
setNum(0 + 1);
```

이렇게 바꿔쓸 수 있음  
`batching`이 실행될 당시의 `num`의 스냅샷이 0이기 때문에 이렇게 동작한다.

```jsx
const [num, setNum] = useState(0);
const batching = () => {
  setNum(num + 1);
  setNum(num + 5);
  setNum(num + 2);
};
```

이렇게 했을 경우 한번 클릭하면 결과는 2가 된다

> 이는 음식점에서 주문을 받는 웨이터를 생각해 볼 수 있습니다. 웨이터는 첫번째 요리를 말하자마자 주방으로 달려가지 않습니다! 대신 주문이 끝날 때까지 기다렸다가 주문을 변경하고, 심지어 테이블에 있는 다른 사람의 주문도 받습니다.

이 설명을 보면 바로 이해가 감

## 배칭을 왜 하나?

리렌더링을 줄이고 리액트가 빠르게 실행되도록 만들어준다. 대신 이벤트 핸들러 내 코드들이 완료될 때 까지 UI 업데이트는 안됨

## 연속해서 변수를 업데이트 하고싶으면 어캄?

함수형 업데이트, state를 갈아끼우는 방법이 아니라 state 값으로 무언가를 해라. 라고 지시하는것

```jsx
setNumber((n) => n + 1);
setNumber((n) => n + 1);
setNumber((n) => n + 1);
```

이게 큐에 하나씩 추가됨, 그리고 다음 렌더링에서 react는 큐를 순회함.아래처럼 차곡차곡 업데이트된다.

| _queued update_ | _n_ | _returns_ |
| --------------- | --- | --------- |
| n => n + 1      | 0   | 0 + 1 = 1 |
| n => n + 1      | 1   | 1 + 1 = 2 |
| n => n + 1      | 2   | 2 + 1 = 3 |

```jsx
const batching = () => {
  setNum(num + 5);
  setNum((n) => n + 1);
};
```

이렇게 섞여있는 경우에는?  렌더링이 큐에 쌓인다는걸 기억하기

| _queued update_  | _n_        | _returns_ |
| ---------------- | ---------- | --------- |
| “replace with 5” | 0 (unused) | 5         |
| n => n + 1       | 5          | 5 + 1 = 6 |

setState(x)가 setState(n => x)랑 동일하게 동작하고 있는것임  
이렇게 보면 더 명확하게 차이점을 알 수 있네.. prevState가 뭐였든지간에 x로 갈아끼운다.

```jsx
const batching = () => {
  setNum(num + 5);
  setNum((n) => n + 1);
  setNum(42);
};
```

그러면 이런 경우에도 전혀 헷갈리지 않음
큐의 마지막에 있는 42로 갈아끼운다

내가 이해한건

```jsx

function render1() {
    ...
    setState(...)
}

function render2() {
    ...
}

```

실행시키는 함수별로 렌더링이 나뉘고 개별 렌더링 안에서는 상태를 변화시키는게 아니라 다음 렌더링을 요청한다(=큐에 넣는다) 현재 스냅샷은 절대 건드리지 않음
그래서 아래같이 바꾸고 바로 콘솔로 확인해도 업데이트 전 값이 반환되는것

```jsx
const [count, setCount] = useState(0);

setCount(1);
console.log(count); // 0
```
