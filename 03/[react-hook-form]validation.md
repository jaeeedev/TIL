# react-hook-form

valid체크 과정에서 원하는대로 동작하지 않아서 다시 useForm에 register 조합으로 돌아왔고
거기서 원하던대로 구현한것 일단 기록

## 타이핑마다 valid check 수행하게 하기

```js
const {
  register,
  handleSubmit,
  setError,
  formState: { errors, dirtyFields },
} = useForm({
  mode: "onChange",
  criteriaMode: "all",
});
```

`useForm`에 아무런 인자를 전달하지 않으면 기본적으로 submit을 해야 valid check를 수행한다.  
`useForm`에 `mode` 옵션을 주면 변경할 수 있음
react-hook-form에서 오류가 발생하면 최초로 발생한 오류 하나만 잡히고 그 오류가 해결되면 다음 오류가 잡힌다.  
한번에 모든 오류를 표시하고 싶어서 `criteriaMode`를 `all` 로 주었다.

## valid, invalid 한 경우마다 색상으로 표시하기

![image](https://user-images.githubusercontent.com/72128840/227456726-3513e3be-b602-47de-b9f7-1cd46c98a23a.png)

스타일드 컴포넌트를 사용하고 있기 때문에 prop의 값을 통한 조건부 스타일링을 활용하였다.
기본 색상은 검정색이지만 해당 속성의 오류가 잡히면 붉은색, 문제가 없다면 초록색을 입혀주는 방식이다.
이 논리대로 구현하면 기본 값 또한 초록색이 되기 때문에 필드가 변화되었는지를 감지하여 색상을 변화시켜 주는
로직을 추가하는 것이 좋다. 그래서 사용자가 변화시킨 필드 정보가 객체로 담기는 `dirtyFields` (`formState`에서 반환함)를 체크해 주었다. dirtyFields 객체에 password가 있다면 사용자가 password를 건드렸다는 뜻이고 에러 객체에도 password가 존재하지 않으면 초록색을 입혀 유효하다는 것을 알린다.

```js
<Input
  type="password"
  /.../
  errorStatus={errors?.password ? "active" : null}
  validStatus={!!dirtyFields?.password && !errors?.password ? "active" : null}
/>


const Input = styled.input`
  ...

  border-bottom: 1px solid
    ${({ errorStatus, validStatus }) => {
      if (errorStatus) {
        return "crimson";
      }
      if (validStatus) {
        return "#10b981";
      }
    }};
`;
```

## 문제

콘솔 찍어보니 password-check는 validate prop을 줬기 때문에 타이핑을 할 때 마다 추가적으로 validate의 콜백들로 valid check를 한다. 하지만 password는 그렇지 않기 때문에 password를 변경해서 발생하는 에러를 잡지 못한다.

![image](https://user-images.githubusercontent.com/72128840/227455249-7aa9d1b8-5729-42fc-96e1-2c51d428036b.png)

`비밀번호 확인` 필드에서 먼저 123 을 입력하고 `비밀번호` 필드에 123을 입력하였다.

> 동작 순서
>
> 1. 비밀번호 확인 필드에 123 을 작성한다. validation의 콜백 함수들이 실행된다. 비밀번호 필드와 값이 다르므로 오류 메시지가 출력된다.
> 2. 비밀번호 필드에 동일하게 123을 작성한다. 하지만 비밀번호 필드 자체의 valid check만 수행되어 비밀번호 확인 필드에서는 아무런 업데이트도 이루어지지 않는다.

한참 고민하다 다시 공식문서의 register 항목을 보았다. 가장 마지막에 있는 deps가 눈에 띄었다.

![image](https://user-images.githubusercontent.com/72128840/227458624-4dd78f31-e9a5-4076-9e0d-c4747f92c620.png)

의존성 배열을 둔다면 리액트 훅 처럼 서로 연관되어 valid check를 다시 수행시킬 수 있을 것이라고 기대했다.

```js
<Input
          type="password"
          {...register("password", {
            required: { value: true, message: "비밀번호를 입력해주세요." },
            maxLength: { value: 20, message: "20자 이내로 입력해주세요." },
            maxLengthminLength: { value: 8, message: "8자 이상 입력해주세요." },
            pattern: {
              value: passwordValid,
              message:
                "비밀번호는 영어 소문자,숫자,특수문자의 조합이어야 합니다.",
            },
            deps: ["password-check"], // ✨✨
          })}
```

![image](https://user-images.githubusercontent.com/72128840/227459322-b59e9ee0-20bb-4b93-8c52-ef7f9c2ede43.png)

이번에는 비밀번호 필드를 나중에 채워도 valid check가 수행되어 업데이트가 잘 이루어졌다.  
개념을 혼동하면 안되는게 해당 필드!!가 업데이트 됐을 때 의존성 배열 속 필드의 valid check가 다시 이뤄지는것!!!

## Input 코드

```js
<Input
  type="password"
  {...register("password", {
    required: { value: true, message: "비밀번호를 입력해주세요." },
    maxLength: { value: 20, message: "20자 이내로 입력해주세요." },
    maxLengthminLength: { value: 8, message: "8자 이상 입력해주세요." },
    pattern: {
      value: passwordValid,
      message: "비밀번호는 영어 소문자,숫자,특수문자의 조합이어야 합니다.",
    },
    deps: ["password-check"],
  })}
  id="password"
  className="password"
  placeholder="password"
  errorStatus={errors?.password ? "active" : null}
  validStatus={!!dirtyFields?.password && !errors?.password ? "active" : null}
/>;
{
  errors?.password && (
    <p className="error-message">{errors?.password?.message}</p>
  );
}
<Input
  type="password"
  {...register("password-check", {
    required: { value: true, message: "비밀번호를 입력해 주세요." },
    validate: {
      sameCheck: (value, formValues) => {
        if (value !== formValues.password) {
          return "비밀번호가 일치하지 않습니다.";
        }
      },
    },
  })}
  id="password-check"
  className="password-check"
  placeholder="password-check"
  errorStatus={errors?.["password-check"] ? "active" : null}
  validStatus={
    !!dirtyFields?.["password-check"] && !errors?.["password-check"]
      ? "active"
      : null
  }
/>;
{
  errors?.["password-check"] && (
    <p className="error-message">{errors?.["password-check"]?.message}</p>
  );
}
```

input 코드가 너무 비대해져서 다시 나누는걸 고민해 봐야지
