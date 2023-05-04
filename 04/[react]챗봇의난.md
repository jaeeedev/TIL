# chatbot

지금 현재 챗봇은 아니고 그런 구조의 뭔가를 연습하고 나중에 가져다 붙일 생각이었음 타이핑 이펙트, 스크롤 등등 손이 끔찍하게 많이 간다.

## 번거로워

일단 카드 컴포넌트 만들고 텍스트 쌓음 -> 텍스트가 삐져나감 -> max-height와 스크롤 추가 -> 타이핑 효과 적용 -> 답변 타이핑 중에는 질문 막기(input disable) -> 타이핑 끝나면 input disabled 풀고 다시 input창에 포커스 이동시키기 -> 포커스 안됨 -> setTimeout(or flushSync)으로 동작순서 변경 -> opacity만 0으로 바꿔서 뒷 배경이 안눌림 -> z-index 추가 -> 텍스트 쌓일 때 스크롤도 같이 내려줘야 함 -> 관련 코드 추가 -> **문제사항**

```jsx
const scrollToBottom = useCallback(() => {
  chatListRef.current.scrollTop = chatListRef.current.scrollHeight;
}, []);
```

원래 useEffect로 isTyping이 변했을 때 높이를 변경하려고 했는데 이벤트 핸들러로 처리할 수 있는 부분은 핸들러로 하는것이 좋다던 공식 문서의 조언을 떠올리고 변경했다.

**문제 발생**

```jsx
useEffect(() => {
  setIsTyping(true);
  scrollToBottom(); // 스크롤 내리기
  if (typingText.length === chat.data.length) {
    setIsTyping(false);
    setTimeout(() => {
      focusInput();
    }, 0);
  }
}, [typingText]);
```

타이핑이 되고있을 때 스크롤 코드를 추가해 줬다. 그러다보니 한 글자 한 글자 생성될 때마다 스크롤이 내려간다. 그래서 결과가 타이핑될 동안 위의 내용을 다시보려고 스크롤을 올려도 다시 내려간다는 뜻이다. 이 문제는 따로 해결을 해야한다...  
타이핑 효과가 완료됐을 때 스크롤을 내리는 방법도 해봤지만 그건 더 안좋음 유저가 집중하고 있을 때라면 텍스트가 잘릴 때 그냥 스크롤을 내려버릴테니

## 그래서 resizeObserver

```jsx
useEffect(() => {
  const ro = new ResizeObserver((entries) => {
    for (let entry of entries) {
      scrollToBottom();
    }
  });

  if (textRef.current) {
    ro.observe(textRef.current);
  }

  return () => {
    ro.disconnect();
  };
}, []);
```

한 글자마다 스크롤을 내리지 않고 줄바꿈이 발생할 때 마다 스크롤을 내려준다. 줄이 바뀌면 요소의 사이즈가 늘어나니까 그때 resizeObserver가 변화를 감지해서 실행시킴

-> 스크롤을 띄워놨을 때는 내리지 않도록 하는 코드 추가 필요

챗봇 컴포넌트가 반응형이 아니고 width는 완전 고정이라 가능한 방법인 것 같음

## 개선사항 고민

- input에 최대 작성가능한 텍스트 수를 정하지 않았는데 어느정도가 좋은지 모르겠음
- 타이핑 효과중일 때 input 을 막기보다는 form의 submit을 막는게 더 사용성 측면에서 좋겠다는 생각.. 추가로 질문할 내용이 기다리다가 휘발되는것보다는 적어놓고 기다리는게 낫지 않나? (그렇다면 그냥 타이핑 효과를 빼는게 최고인듯한데 챗봇에서는 왠지모르게 필수적인 느낌이 있다)
