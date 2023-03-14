# github oauth

외부 사이트에서 깃허브 api를 연동하여 깃허브의 기능을 사용할 수 있게 만들고 싶었다. 단순히 나 혼자 쓰는게 아니라 다른 사람도 기능을 사용할 수 있어야 해서(이슈의 comment 기능) 다른 사용자의 액세스 토큰이 필요했음.

## 처음 생각한 흐름

토큰이 동적으로 변해도 되는가? 이 방식으로 할수있나? 너무 불분명하고 생각보다 깃허브api에 대해 작성된 블로그 글들이 많이 없어서 삽질 많이했음..  
안되겠어서 utterances 오픈소스 뜯어봤다 거기선 octokit은 안썼지만 요청을 보낼때 헤더에 토큰을 넣는게 보였고 원래 의도랑 비슷한거같아서 계획 짜고 진행

인가 단계(리다이렉트) -> 동의 후 코드 받기 -> 토큰 받기 -> 토큰으로 octokit 인스턴스 찍어서 원하는 api 사용하기

## 문제 발생

토큰 잘 확인되고 octokit 인스턴스도 나오는데 private 레포가 안보임
레포를 전부 가져오도록 요청해도 **퍼블릭 레포만 나옴**

## 원인

토큰 내려주는 코드에서 콘솔 찍어보니

```js
access_token=토큰&scope=&token_type=bearer
```

이렇게 `scope`가 비어있었음

나는 깃허브 설정에서 토큰 발급받을 때 개별적으로 scope를 조정한 적이 있어서 그걸로 해결되는 줄 알았다. (지금 생각해보면 인증 할때마다 토큰이 바뀌는데 왜 그럴거라고 생각했는지...)
그렇지 않고 명시적으로 범위를 지정해 주어야 함.  
scope는 **앱의 권한**과 관련이 있다.  
기본적으로 아무것도 없으면 공개 정보(프로필, 퍼블릭 레포)만 볼 수 있고 `repo`를 주면 코드, 커밋 상태, 프라이빗 레포지토리 등의 정보에 접근이 가능

```jsx
const Main = () => {
  const link = `https://github.com/login/oauth/authorize?client_id=${CLIENT_ID}&redirect_uri=${"http://localhost:3000/callback"}?&scope=${"repo,user"}`;

  return (
    <div>
      <h1>oauth</h1>
      <a href={link}>테스트</a>
    </div>
  );
};
```

[참고](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/scopes-for-oauth-apps)

## 추가

인가 후에 리다이렉트되는 페이지에서 토큰을 요청하게 되는데 이 페이지로 다시 돌아오면 또 요청이 가서 bad request가 뜬다. 상태에 토큰을 세팅하든 브라우저 스토리지에 저장하든 bad request로 덮어씌워져버리면 오류가 나기 때문에 이 페이지에 다시 들어오지 못하게 해야 함.

```js
const navigate = useNavigate();

useEffect(() => {
  const getToken = async () => {
    try {
      const result = await axios.post("api 주소", {
        code,
      });

      const { token } = result.data;

      localStorage.setItem("gh-token", token);
      navigate("/test", { replace: true }); // ✨✨ 뒤로가기 막아서 배드리퀘스트 방지
    } catch (error) {
      console.log(error);
    }
  };
  getToken();
}, [code, navigate]);
```

리액트 라우터의 `useNavigate` 옵션 중 `replace`를 true로 주면 그 페이지에서 벗어난 다음 뒤로가기를 눌러도 이 페이지로 다시 돌아올 수 없다. 가장 간단한 방법인듯
