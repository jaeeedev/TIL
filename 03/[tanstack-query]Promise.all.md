## 여러개의 쿼리를 무효화하기

![image](https://user-images.githubusercontent.com/72128840/222672181-99877c73-2f3f-43a5-b6bb-8d25bc9369ba.png)

`invalidateQueries` 가 바로 안먹던 문제를 해결하고 블로그에 썼었는데 이번에는 다른 문제 ㅋㅋㅋ

![image](https://user-images.githubusercontent.com/72128840/222672501-bd258c1e-46e8-432f-8566-c32b77f15d47.png)

리스트에서 글로 들어오면 이 화면이 나온다.
하단의 질문 닫기를 클릭하면 질문이 닫히는 구조였고 나는 첫번째 사진의 리스트를 다시 갱신시키는 방법을 썼었음  
질문을 다시 여는 기능을 추가하면서 열기 버튼과 닫기 버튼이 상황에 맞게 토글되어야 했다.

## 문제점

`invalidateQueries`를 작성해놓은 `onSuccess` 콜백은 return문을 작성하면 변이가 완료될 때 까지 기다려주고 넣지 않으면 기다리지 않는다.  
 하지만 지금은 **여러개의 함수를 무효화**해야하는 상황이라 문제

## 해결 방법

```js
const { mutate: closeMutate } = useMutation({
  mutationFn: closeQuestion,
  onSuccess: () => {
    return Promise.all([
      queryClient.invalidateQueries({
        queryKey: ["questionList"],
        refetchType: "all",
      }),
      queryClient.invalidateQueries({
        queryKey: ["question", id],
      }),
    ]);
  },
});
```

모든 프로미스가 다 이행될때까지 기다리는 방법으로 `Promise.all`을 사용할 수 있다고 한다. `invalidateQueries`는 프로미스를 리턴하기 때문에 이 방법이 가능하다.

간단하게 묶기만 하면 되니 짱 좋음  
Promise는 async, await 문법이나 then 아니면 활용해볼 일이 잘 없었는데 이런 방법을 알게 되어서 좋다~

출처 | [react-query mutation onSuccess에서 여러 개의 invalidateQueries를 기다리는 방법](https://careerly.co.kr/qnas/1976?fa=qna-list)
