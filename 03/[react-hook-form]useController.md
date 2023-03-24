## useController

### 원하던 구현사항

```
- 타이핑 마다 유효성(rule) 검사
- 필드에 문제가 있는 내용이 들어오면 오류 메시지, 유효하면 ✔️ 표시
```

`useForm`의 `register`를 사용하면 폼이 submit 되는 단계에서 rule검사를 실행함  
타이핑 될 때 마다 실행하고 싶기 때문에 다른 방법 필요  
필드의 변화를 관찰시키는 방법인 `onChange`나 `watch`를 사용하면 전체 폼이 리렌더링 됨 -> 왜???

`watch`는 폼의 루트 수준에서 실행되어 필드가 변경되면 폼 전체가 리렌더링된다고 함.  
`useWatch`를 사용하라는데 찾아보니 `useController` 사용하는게 낫대서 `useController`로 변경

### useController 세팅

```js
// Form.jsx

<Form onSubmit={handleSubmit(onSubmit)} onKeyDown={getCapsLockState}>
  <h2 className="title">회원가입</h2>
  {capsLockState && (
    <div className="capslock-message">❗CapsLock이 켜져있습니다.</div>
  )}

  <Input name="id" control={control} placeholder="아이디" />
  <Input name="password" control={control} placeholder="비밀번호" />
  <Input name="password-check" control={control} placeholder="비밀번호 확인" />

  <div className="button-area">
    <button className="register-submit">Sign up</button>
    <Link to="/login">이미 계정이 있으세요?</Link>
  </div>
</Form>;

// Input.jsx
import { useController } from "react-hook-form";

const Input = ({ name, control, rules, placeholder }) => {
  const {
    field,
    fieldState,
    formState: { isDirty, isTouched, error, isValid, isValidating },
  } = useController({
    name,
    control,
    rules,
  });

  return (
    <StyledInput
      placeholder={placeholder || ""}
      value={field.value}
      rule={field.rule}
      name={field.name}
      inputRef={field.ref}
      onChange={(e) => {
        field.onChange(e.target.value);
      }}
    />
  );
};
```

개별 input으로 컴포넌트를 분리하여 관리하기 때문에 특정 폼의 일부라는것을 어떻게 알아차리나 했는데
`control`을 prop으로 내려준 다음 `useController`의 인수로 넣어주면 연결할 수 있다.
`onChange`를 연결시켜줘야 변화가 감지됨. 이제 작성 중인 필드만 리렌더링된다.

### 문제

> Warning: A component is changing an uncontrolled input to be controlled.

useController 로 Input들을 따로 분리하고 나니 폼을 입력할 때 이런 오류가 발생함.  
defaultValue나 폼의 defaultValues를 지정하지 않아서 발생하는 문제라고 함.

```js
const {
  control,
  handleSubmit,
  setError,
  formState: { errors },
} = useForm({
  defaultValues: {
    id: "",
    password: "",
    "password-check": "",
  },
});
```

폼에다 `defaultValues` 넣어주니 해결됨

### 캡스락 감지하는법

```js
const getCapsLockState = useCallback((e) => {
  setCapsLockState(e.getModifierState("CapsLock"));
}, []);
```

키보드 관련 이벤트 받아서 `e.getModifierState("CapsLock")` 값을 보면 됨  
리턴값은 boolean이고 true일 때 특정한 처리를 해주면 된다.
