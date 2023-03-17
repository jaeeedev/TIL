## 에러 핸들링

에러를 어떻게 잘 다룰수있을지 고민중임  
모든 컴포넌트에서 산발적으로 하나하나씩 에러 처리를 하면 죽도록 힘들겠지....

### 직접 에러 던질때 status code 붙이는 법

```js
const error = new Error();
error.name = "테스트 오류";

error.response = {
  status: 401,
  data: {
    detail: "커스텀 에러",
  },
};

throw error;
```

이렇게하고 에러바운더리에서 `error.response.status`로 꺼내 쓰면 됨

### react-query와 사용하기

react-query 를 쓰는 경우 queryClient 인스턴스 생성할 때 옵션에 `suspense:true`를 주면 에러 바운더리 쓸 수 있음

```js
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 0,
      suspense: true, // ✨✨
      staleTime: 9000000,
      cacheTime: 12000000,
      refetchOnWindowFocus: false,
      refetchOnMount: false,
    },
  },
});
```

### 에러바운더리를 선택적으로 쓰고 싶은 경우

useQuery나 useMutation 할 때 속성을 주면 됨

```js
useQuery({
  queryKey: ["todos"],
  queryFn: fetchTodos,
  useErrorBoundary: (error) => error.response?.status >= 500,
});
```

이런 경우에 500번대 이상 오류(서버 오류)들만 에러바운더리를 사용하고 나머지 오류들은 해당 컴포넌트 내에서 처리한다.
해당 요청에서 에러바운더리르 쓰기 싫다면 아예 `false`주면 됨

```js
const defaultHandlers = {
  401: () => {
    console.log("error 401");

    return <UnauthFallback />;
  },
  403: () => {},
  500: () => {},
  default: () => {
    console.log("error default");
    return (
      <div>
        <h2>서비스를 이용할 수 없습니다.</h2>
        <p>잠시 후 다시 이용해주세요.</p>
      </div>
    );
  },
};
```

객체를 하나 파서 어떤 상황마다 일어날 코드들을 작성했음  
실제로는 더 복잡하고 같은 에러 코드라도 상황이 달라서 분기를 더 나눠야겠지만 일단 status code를 기준으로 나누었다.  
작성해놓은 상태 코드에 존재하지 않는 오류가 발생한다면 디폴트로 지정해놓은 폴백이 나타남

```js
const useError = () => {
  const handleError = useCallback((error) => {
    const httpStatus = error.response.status;

    switch (true) {
      case !!defaultHandlers[httpStatus]:
        return defaultHandlers[httpStatus]();
      default:
        return defaultHandlers["default"]();
    }
  }, []);

  return { handleError };
};
```

그리고 에러를 처리하는 커스텀 훅을 생성함  
있으면 실행해라 라는 느낌으로 !! 붙여서 boolean화 시켜주어야 잘 동작함

```js
import { ErrorBoundary } from "react-error-boundary";
import useError from "../../components/error/useError";

const IssueFallback = ({ error }) => {
  const { handleError } = useError();

  const fallback = handleError(error);
  return fallback;
};

const Issue = () => {
  return (
    <ErrorBoundary FallbackComponent={IssueFallback}>
      <IssueList />
    </ErrorBoundary>
  );
};
```

에러바운더리의 fallback 컴포넌트에 에러를 넘겨주면(react-error-boundary에서 제공하는 에러 바운더리에서 폴백 컴포넌트 지정하면 알아서 에러가 넘어감)

![image](https://user-images.githubusercontent.com/72128840/225818516-44b2e51a-7cef-46e5-b31f-7a9da4e1267d.png)

에러 폴백 화면이 잘 나타난다

### Error responses

| 상태 코드 | 이름                  | 설명                                                              |
| --------- | --------------------- | ----------------------------------------------------------------- |
| 400       | Bad Request           | 잘못된 문법으로 요청하여 서버가 이해할 수 없음                    |
| 401       | Unauthorized          | 사용자가 인증되지 않은 경우, 클라이언트가 누구인지 모름           |
| 403       | Forbidden             | 사용자가 콘텐츠에 접근할 권리가 없는 경우, 클라이언트가 누군지 앎 |
| 404       | Not Found             | 요청받은 리소스가 없음. 엔드포인트를 제대로 지정해도 발생가능     |
| 408       | Request Timeout       | 서버가 사용되지 않는 오래된 연결을 끊으려고 할 때 발생            |
| 500       | Internal Server Error | 서버가 처리 방법을 모르는 상황이 발생                             |

에러바운더리는 선언적!! 이라고 하는데 if문이나 삼항연산자로 에러가 있으면 이 컴포넌트 보여주고 없으면 안보여주고 이렇게 하나하나 통제하는 흐름이 아니라 무엇을 구현할지에만 집중할 수 있어서 선언적이라고 한다.

[화해 블로그 에러핸들링](https://blog.hwahae.co.kr/all/tech/7867)  
[리액트쿼리, 리액트 에러 바운더리](https://www.datoybi.com/error-handling-with-react-query/)  
[리액트 쿼리 오류 처리](https://tkdodo.eu/blog/react-query-error-handling)
