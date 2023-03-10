## 신택스 하이라이팅 적용 문제

gpt api를 이해하기 위해서 미리 만들어진 보일러플레이트(강아지 이름 지어주는) 받고 거기서 커스텀 하기 시작함

![image](https://user-images.githubusercontent.com/72128840/224191477-976b95cb-5c29-4d6b-9848-dd34daa67a95.png)

**SyntaxError: Unexpected token 'export' react syntax highlighter**  
만들어진 코드는 nextjs로 작성되어있었음  
이렇게 신택스 하이라이팅 모듈이 임포트되지 않았다.

```js
import { oneLight } from "react-syntax-highlighter/dist/esm/styles/prism";
```

기존 코드는 `react-syntax-highlighter/dist/esm/...` 으로 임포트를 했고

```js
import { oneLight } from "react-syntax-highlighter/dist/cjs/styles/prism";
```

`react-syntax-highlighter/dist/cjs/...`로 고쳤더니 제대로 동작했다. cjs는 `commonJs`고 esm은 `ECMAScript` 모듈이다.

![image](https://user-images.githubusercontent.com/72128840/224255553-d3bd153d-e469-414a-85f9-25ce2912900f.png)

마크다운이 적용되었다.

챗봇 형식으로 고쳤다.

![image](https://user-images.githubusercontent.com/72128840/224255838-533db18e-a65d-4d59-a916-e1ae4a24d9c6.png)

```js
const [chat, setChat] = useState([]);

//   {id:1, text: "", author: "you" | "bot"} 이런 데이터가 들어감
```

원래는 대화 하나만 되게 했다가 대화 내용을 전부 남기는 방식으로 바꿨는데 그 과정에서 타이핑 효과를 뺐다. (다시 추가해야함)
그리고 고민되는게 코드를 반환해주면 복사 버튼을 올려서 코드 복사가 되게 하고 싶은데 신택스 하이라이팅이 먹힐 때가 있고 안먹힐 때가 있다.  
안된 상태에서는 텍스트를 뽑기가 수월하고 하이라이팅이 된 상태에는 코드들이 조각조각 잘려있는 상태에 개행문자가 잔뜩 붙어있어서 또 뽑기가 애매하다...
너무 예외가 많아서 억지로 구현하면 안될 것 같은 느낌..?? 방법이 있을텐데 잘 모르겠다
