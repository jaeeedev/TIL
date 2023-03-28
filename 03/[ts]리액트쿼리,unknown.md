## reacy-query

버전3 강의라서 버전3 내용임 크게다른건없겠지만

### Promise<void>

```ts
async function setAppointmentUser(
  appointment: Appointment,
  userId: number | undefined
): Promise<void> {
  if (!userId) return;
  const patchOp = appointment.userId ? "replace" : "add";
  const patchData = [{ op: patchOp, path: "/userId", value: userId }];
  await axiosInstance.patch(`/appointment/${appointment.id}`, {
    data: patchData,
  });
}
```

여기서 setAppointmentUser의 리턴값 타입이 `Promise<void>`인게 이해가 안가서 찾아보니 비동기 요청 이후 아무것도 리턴하지 않으면 `Promise<void>`를 줘야 한다고 한다. `() => void`로 타입을 정의하면 비동기 함수뿐만이 아닌 모든 리턴값이 없는 함수를 의미하기 때문에 좀 더 좁은 의미를 갖도록 Promise를 붙이는 것

### UseMutationFunction

```ts
export function useReserveAppointment(): UseMutateFunction<
  void,
  unknown,
  Appointment,
  unknown
> {
  const { user } = useUser();
  const toast = useCustomToast();

  const { mutate } = useMutation((appointment: Appointment) =>
    setAppointmentUser(appointment, user?.id)
  );

  return mutate;
}
```

useMutation을 실행하는 커스텀 훅
`UseMutateFunction` 타입은 `TData`, `TError`, `TVariables`, `TContext` 네가지 제네릭을 가진다.

> `TData`는 queryFn(useMutation에 전달되는 함수)이 반환하는 데이터(없으니까 void)  
> `TError`는 에러의 타입  
> `TVariables`는 queryFn에 전달되는 인수의 타입  
> `TContext`는 onMutate가 반환하는 context의 타입

### unknown 타입

위의 코드들에서 어떤 값이 올지 미리 확신할 수 없을 때 `unknown`을 사용했다.  
`any`보다는 `unknown`을 사용하는게 좋다는건 알고 있는데 자세한 차이점은 모르던 상태

unknown 타입은 선언해두고 아무런 타입의 값을 할당할 수 있지만 **이미 타입이 지정된 변수에 할당할 수는 없다.**(any 타입의 변수에는 할당 가능)

```ts
let test: unknown;

test = true; // 가능
test = "hi"; // 가능

let number: number = test; // 불가능
let any: any = test; // 가능
```

또한 할당된 값을 모르는 상태여서 프로퍼티나 메서드를 참조할 수 없다.

```ts
let test: unknown;

test.propFn(); // 불가능
```

any는 어떤 동작을 하더라도 컴파일 단계에서 오류를 잡지 않으니 런타임 오류를 만나게 되는데 unknown은 좀 더 깐깐하고 타입 좁히기를 곁들여서 안전하게 코드를 작성할 수 있는 듯 하다.
