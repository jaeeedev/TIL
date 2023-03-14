## 오늘 한것

![image](https://user-images.githubusercontent.com/72128840/224933682-de20d6e9-da49-4a7b-a757-ae76cbe57d9c.png)
깃허브 rest api 이용해서 요런 페이지를 만들었음

## border-radius 줬을 때 모서리만 지워지는 문제

프엔이라는 놈이 css 몰라서 이렇게 기록하고있는게 좀 찔리긴 하지만.. 처음겪은 일이라 기록

![image](https://user-images.githubusercontent.com/72128840/224927115-239ea6b2-b87b-4deb-bf1b-c28b3d211586.png)

윗부분 모서리가 살짝 지워져있는걸 볼 수 있는데 구글링해보니

![image](https://user-images.githubusercontent.com/72128840/224927681-5e4641dd-24ae-40f0-8ca8-f9ac4ad1fa64.png)
[출처](https://stackoverflow.com/questions/39420256/border-radius-disappears-with-overflow-rule)

```css
.comment-box {
  width: 100%;
  border-radius: 0.5rem;
  border: 1px solid #d0d7de;
  margin-bottom: 1rem;
}
.comment-info {
  display: flex;
  justify-content: space-between;
  padding: 0.5rem 1rem;
  background: #f6f8fa;
  color: #57606a;
  border-bottom: 1px solid #d0d7de;
  border-radius: 0.5rem 0.5rem 0 0; /* ✨✨ */
}
```

자식 요소에 배경색이 있을 때 이런 문제가 생긴다는듯  
해결 방법은 자식 요소에도 동일하게 `border-radius` 주면 됨  
저 답변처럼 `inherit` 해도 되는데 나는 윗부분만 둥글어야 해서 직접 지정함

![image](https://user-images.githubusercontent.com/72128840/224933371-d52ff8d0-574a-46e8-9335-0fac5794364e.png)
모서리 잘 나옴
