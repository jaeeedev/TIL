# React

## 리액트란 무엇인가

컴포넌트로 사용자 인터페이스를 구성할 수 있는 라이브러리

**장점**

- 컴포넌트 단위로 코드를 관리하여 재사용성이 좋고 유지보수가 편리
- 가상 돔을 이용해 충분히 빠른 속도

**단점**

- 다른 프레임워크(뷰, 스벨트 등)에 비해 러닝커브가 높음

## 설치

CRA(create react app)를 사용합니다.  
기존에 웹팩, 바벨 설정과 라이브러리 의존성 관리로 인해 개발을 시작하는 데 어려움이 컸는데 CRA를 사용하면 간편하게 설정이 완료된 프로젝트 폴더를 생성할 수 있습니다.
(전자정부 프레임워크 front 예제 폴더가 CRA로 구성되어 있음)
(현재 권장되는 방법은 아님)

```
npx create-react-app <프로젝트명>
```

`node.js`가 설치되어 있어야 합니다.

## jsx

```jsx
// App.js

const App = () => {
  const name = "Josh Perez";

  return <h1>Hello, {name}</h1>;
};
export default App;
```

한 컴포넌트가 return하는 부분부터 jsx를 작성합니다. 기존에 html을 작성하던 것과 같이 html을 작성하고 자바스크립트 변수는 `{}` 안에 넣어주면 됩니다.

... 쭈루룩 ppt 설명

## Props

리액트는 state와 props 알면 거의 다 끝난거

리액트는

```jsx
const Parent = () => {
  return <Child />;
};
```

이런 구조를 가질 수 있는데 Parent 컴포넌트가 Child 컴포넌트에게 어떤 상태나 변수를 공유하고 싶을 수 있습니다. 그럴 때 props를 사용합니다.

```jsx
export default function Profile() {
  return (
    <Avatar person={{ name: "Lin Lanying", imageId: "1bX5QH6" }} size={100} />
  );
}

function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}
```

이 예제에서는 상위 컴포넌트(Profile)에서 `person`과 `size` props를 내려주고 있습니다.  
Avatar 컴포넌트에서 `person`과 `size`를 인수로 받아 컴포넌트 내부에서 사용합니다.

`function Avatar({ person, size })` 여기서 `{ person, size }`는 구조분해할당 문법입니다. `props`객체를 분해하여 프로퍼티를 바로 사용할 수 있습니다. 아래의 예제를 보면 금방 이해할 수 있습니다.

```jsx
function Avatar(props) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={props.person.name}
      width={props.size}
      height={props.size}
    />
  );
}
```

props는 자식 컴포넌트에서 부모 컴포넌트로는 전달할 수 없고, 위에서 아래로 전달하더라도 한번에 여러 컴포넌트를 건너뛰어 전달하는 것은 불가능합니다. 이런 불편한 점 때문에 복잡한 프로젝트의 경우 전역 상태 관리 라이브러리(redux, recoil) 혹은 context API를 고려할 수 있습니다.

## 이벤트 처리하기

리액트에서 이벤트를 처리하는 방법은 html에서 이벤트를 처리하는 방법과 유사합니다.

```jsx
<Component onClick={onClickFn} />
```

카멜케이스로 이벤트를 작성해주면 됩니다. 모든 이벤트 리스너는 on으로 시작합니다.
만약 인자를 전달해줘야하는 경우 아래처럼 작성합니다.

```jsx
const param = "something";

<Component onSubmit={() => submitForm(param)} />;
```

`<Component onSubmit={submitForm(param)} />` 이렇게 선언하면 안되는 이유는 이벤트가 실행되는 시점이 아니라 컴포넌트 렌더링과 동시에 즉시 함수가 호출되는 문제가 있기 때문입니다.

## 조건부 렌더링

조건부로 컴포넌트를 렌더링할 수 있습니다.  
예를 들어 로딩 상태일때는 로딩 스피너를 화면에 띄우고, 유저 토큰이 없을 경우에는 마이페이지 컴포넌트를 렌더링하지 않는 방식으로 활용할 수 있습니다

if문을 이용할 수도 있고
&&문이나 삼항연산자 사용 가능

```jsx
if (!token) return <p>인증받지 않은 사용자입니다.</p>;
```

```jsx
token ? <MyPage /> : <WrongAccess />;
```

## 리스트와 키

데이터 받아서 화면에 쭉 리스팅할때 map, filter같은 배열 메서드를 사용하는데 이때는 꼭 key를 지정해주어야 합니다. 그리고 key는 인덱스가 아니라 꼭 고유한 값으로 지정해야 합니다.
리액트는 key에 변화가 발생했을 경우에 다른 컴포넌트는 유지하고 그 컴포넌트만 변경하는데 인덱스를 사용하게 되면 중간에 새로운 요소를 끼워넣던지 하는 과정에서 모든 요소의 인덱스가 새로 지정되고 전부 리렌더링되는 문제가 발생!!  
그래서 키는 컴포넌트를 리셋하는 용도로도 사용할 수 있습니다.

## state 끌어올리기

상태는 `useState`가 선언된 부분에서 생성되기 때문에 한 상태를 다른 컴포넌트와 공유해야 하는 상황이라면 공통의 부모 컴포넌트에 useState로 상태를 선언해야 합니다. 그리고 자식 컴포넌트에 상태를 props로 넘깁니다.

## children

이건 어떤 경우에 사용하냐면  
Card나 모달 컴포넌트같이 이미 스타일이 지정되어있고 여기저기 갖다쓸수있는 컴포넌트가 있을 수 있다. padding이나 레이아웃이 대충 지정되어 있는데 어떤 내용(자식)이 들어올지는 모르는 경우에 children이라고 대충 선언해놓을 수 있다.

```jsx
const Card = ({ children }) => {
  return <div>{children}</div>;
};
```

이렇게 사용  
children은 단순한 텍스트일수도 있고 컴포넌트일수도 있음

## Hook

리액트는 원래 클래스 컴포넌트를 먼저 제공했고, 그 다음에 함수 컴포넌트가 등장  
클래스 컴포넌트가 가지고 있었던 react lifecylcle method라는게 있었는데 예를 들어서
컴포넌트가 화면에 나타났을때, 업데이트 되었을 때, 화면에서 사라졌을 때마다 특정한 동작들을 실행시킬 수 있도록 한 메서드
이걸 함수 컴포넌트에서는 사용할 수 없었는데 비슷한 기능을 하도록, 그리고 리액트의 다양한 기능들을 사용할 수 있도록 훅이 등장

필수적으로 알아야 하는건 `useState`, `useEffect`
최적화에 필요한건 `useCallback`, `useMemo`

훅은 무조건 최상위 레벨에서 실행해야 함. if문 내부, 함수 내부 등등 하위 레벨에서 사용 불가(`useState` 같은 훅 자체를 의미하고 `[state,setState]`같은 리턴값은 내부에서 사용할 수 있음)
커스텀 훅을 생성하여 사용할 수도 있음. 함수명은 무조건 `use`로 시작해야 함

## 상태

어쩌면 리액트의 전부....

```
컴포넌트가 기억해야 하는 정보 = 리액트의 메모리 = 상태
```

상태가 변경되면 컴포넌트가 리렌더링 됩니다. 리렌더링이 일어나면 기본적으로 모든 함수와 변수들이 새로 만들어집니다. 하지만 상태와 훅들, useCallback과 useMemo로 캐싱한 함수, 변수는 새로 만들어지지 않습니다.

## useState

```jsx
const [count, setCount] = useState(0);
```

`count`는 상태
`setCount`는 상태를 변경하는 함수
`useState(0)`에서 0은 초기값

**알고 가기**

리액트는 렌더링 최적화를 위해서 setState를 비동기적으로 실행시킵니다.  
그리고 state batching 이라고 하는 상태 일괄 업데이트를 합니다.

```jsx
const [count, setCount] = useState(0);

const update = () => {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);

  console.log(count); // 0
};

console.log(count); // 1
```

그래서 update 함수 내부의 가장 마지막줄에 console.log(count)가 작성되었지만
실제로는 가장 먼저 호출됩니다. 그 당시 state(`count`)의 스냅샷은 0 이었기 때문에
콘솔에는 0이 출력됩니다.

```jsx
const update = () => {
  setCount(0 + 1);
  setCount(0 + 1);
  setCount(0 + 1);

  console.log(count); // 0
};

console.log(count); // 1
```

count를 0으로 치환해보면 결과를 예상하기 더 쉬워집니다.
update 함수가 호출된 다음 count를 확인해보면 3이 아니라 1인 것을 확인할 수 있습니다.

## useEffect

```jsx
useEffect(() => {}, []);
```

이름에 effect가 들어있듯이 사이드이펙트가 발생할 수 있는 코드들을 작성  
대표적으로 서버에 요청할때, setTimeout 사용할때 (setInterval은 리액트에서 사용할 때 문제가 있기 때문에 다른 방법을 사용해야됩니다)

```jsx
useEffect(() => {},
return () => {};
[]);
```

이렇게 함수를 리턴하기도 하는데 이건 `클린업 함수`라고 부름  
컴포넌트가 언마운트(화면에서 사라지기 직전) 될때 함수를 실행시키고 싶을때 사용

마지막에 있는 []는 `의존성 배열(deps)`이라고 부름  
이 배열을 아예 빼버리면 상태가 업데이트 될 때 마다 콜백 함수가 실행됨  
빈 값으로 두면 화면이 렌더링될 때 최초 1회 실행됨  
안에 의존중인 상태(함수에서 사용하고있는 상태)를 넣어주면 그 상태가 변할때마다 실행

의존성은 적을수록 좋습니다  
그리고 useEffect를 많이 사용할수록 의존성에 대한 복잡도가 증가하기 때문에
useEffect가 꼭 필요한지 아닌지를 판단해서 사용하는것이 좋습니다.

예를 들어 fetching을 하는 경우에는 `리액트 쿼리`,`swr`같은 라이브러리를 사용하면 useEffect를 제거할 수 있습니다.
또한 상태가 변했을 때 어떤 함수를 실행시키기 보다는 이벤트 함수로 처리할 수 있다면 그렇게 하는것이 더 좋습니다.

useEffect는 깊게 생각하면 생각할수록 어려우니까 단순히

## useRef

ref는 막 필수!!는 아닌데 나는 자주 쓰게되는것 같아서 설명

```jsx
import { useRef } from "react";

const Component = () => {
  const ref = useRef(null);

  return (
    <div>
      <input ref={ref} />
    </div>
  );
};
```

useRef를 선언하게되면 화면이 렌더링되면서 ref 객체를 만듭니다.  
그리고 괄호 안에 넣었던 값으로 ref.current의 값을 초기화합니다.

```jsx
const ref = useRef(0);

useEffect(() => {
  console.log(ref.current); // 0
}, []);
```

`useRef`는 렌더링과 관계 없이 변수를 사용하고 싶거나(앞서 리렌더링이 일어나면
변수들이 모두 새로 만들어진다고 언급했습니다.) DOM에 직접 접근하고 싶은 경우 사용합니다.

예를 들어 input 컴포넌트의 값을 받아와서 써야하는 경우가 생깁니다.  
그럴 경우 useRef를 사용해 DOM에서 값을 받아올 수 있습니다.

```jsx
const ref = useRef(null);
const getText = () => {
  console.log(ref.current.value);
};

return (
  <>
    <input ref={ref} />
    <button onClick={getText}>값 확인하기</button>
  </>
);
```

## 커스텀 훅

컴포넌트 내에서 반복되는 로직을 함수로 분리할 수 있습니다.  
꼭 함수의 이름을 `use`로 시작해야 합니다. ex) useCount, useInput
