## 에러 바운더리

![image](https://user-images.githubusercontent.com/72128840/223039248-2de073fa-33de-40f9-b63c-88ee77f4927e.png)

리액트 앱에서 에러가 나면 앱이 완전히 멈춘다.  
그렇기 때문에 오류가 나타나도 부분 경고만 띄워주고 앱은 계속 동작하도록 해야한다.

### 리액트 공식 홈페이지에 있는 에러 바운더리 예제

```js
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 다음 렌더링에서 폴백 UI가 보이도록 상태를 업데이트 합니다.
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 에러 리포팅 서비스에 에러를 기록할 수도 있습니다.
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 폴백 UI를 커스텀하여 렌더링할 수 있습니다.
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

클래스 컴포넌트를 거의 안써봐서 클래스 컴포넌트로만 구현할 수 있다는 걸 듣고 좀 당황  
하지만 이해하는 데 어렵지 않다.

### 한계점?

> **Note**  
> 에러 경계는 다음과 같은 에러는 포착하지 않습니다.
>
> - 이벤트 핸들러 (더 알아보기)
> - 비동기적 코드 (예: setTimeout 혹은 requestAnimationFrame 콜백)
> - 서버 사이드 렌더링
> - 자식에서가 아닌 에러 경계 자체에서 발생하는 에러

에러 바운더리는 편리한 기능이지만 이러한 경우에는 에러를 포착하지 않는다.
`try-catch` 구문을 이용해서 잡아주어야 한다.

### 리액트 쿼리(탠스택 쿼리)와 함께 사용하기

```js
const { data: { data } = [], isLoading } = useQuery({
  queryKey: ["issues"],
  queryFn: getIssues,
  useErrorBoundary: true, // ✨✨
  retry: 0,
});
```

`useErrorBoundary` 속성을 true로 주게되면 쿼리를 요청하다가 발생한 오류를 에러 바운더리에 넘길 수 있다.
_useErrorBoundary 안써도 폴백 보인다고 처리 잘 되고 있다고 생각하면 안됨 해보면 메시지 다르게 나옴_

```js
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, errorMessage: "오류가 발생했습니다." };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, errorMessage: error.message };
  }

  componentDidCatch(error, errorInfo) {}

  render() {
    if (this.state.hasError) {
      return <ErrorToast>{this.state.errorMessage}</ErrorToast>;
    }

    return this.props.children;
  }
}

export default ErrorBoundary;

const ErrorToast = styled.div`
  background: #fee2e2;
  color: #b91c1c;
  border-radius: 0.3rem;
  padding: 1rem;
  max-width: 500px;
  margin: 0 auto;
  animation: toast 0.4s ease;

  @keyframes toast {
    0% {
      opacity: 0;
      transform: translateY(10px);
    }
    100% {
      opacity: 1;
      transform: translateY(0);
    }
  }
`;
```

나는 에러의 메시지를 받아서 그 메시지를 화면에 띄우도록 코드를 조금 변경했다.

![image](https://user-images.githubusercontent.com/72128840/223039461-4f13ffbf-f744-4804-8d82-b7ed5c40a9de.png)

`Not Found`나 `Cannot read properties of undefined (reading 'map')` 이런 메시지는
코드를 짠 사람이 보면 이해할 수 있을지 몰라도 사용자가 보면 당황스러운 메시지일 것이다. (만들다 만 느낌..?)

![image](https://user-images.githubusercontent.com/72128840/223039916-25043fed-98b7-4057-9246-0e8c54c50a7a.png)

`try-catch` 문구로 에러를 발생시켜서 더 명료한 에러 메시지를 주면 좋을 것 같다.

```js
const getIssues = async () => {
  try {
    const result = await octokit.request("GET /repos/{owner}/{repo}/issues", {
      owner: "없는 이름",
      repo: "틀린 주소",
    });

    return result;
  } catch (error) {
    throw new Error("해당하는 내용이 없습니다."); // ✨✨
  }
};
```

`getDerivedStateFromError`와 `componentDidCatch`가 헷갈렸는데 공식 문서의 말대로라면  
UI 폴백을 보여주는 목적에는 `getDerivedStateFromError`를 사용하고 에러를 리포팅할 목적에 `componentDidCatch`를 쓰는 것 같다.  
componentDidCatch 단계에서 에러 메시지를 세팅해도 전혀 문제가 없지만 에러를 기록하는 역할은 아니라 getDerivedStateFromError에 작성했다.
