# 테이블

테이블을 써본적이 없다

![image](https://user-images.githubusercontent.com/72128840/232651841-6471a1eb-1625-4e5b-a7a2-89039cf182da.png)

어드민 페이지를 만들어야 하는데 div로 틀어막다가 너무 복잡해져서
다시 table을 공부하기 시작..
처음엔 라이브러리를 쓰려고 했는데 이것도 테이블을 모르면 사용하기가 난감해지는,,  
그리고 지금은 동적인 데이터를 받지 않기때문에 아직은 필요하지 않겠다고 결론내림

![image](https://user-images.githubusercontent.com/72128840/232670896-bfa528cc-6322-4765-8cd9-3379afb6a064.png)
이런 형태의 테이블을 구현해야 했다.

## 겪은 문제

구글링해서 나오는 테이블 예제 코드는 다 간단하게

```html
<table>
  <tr>
    행행
  </tr>
  <tr>
    행행
  </tr>
</table>
```

테이블 안에 row를 구분지어서 결과를 보여주는 예제들이었다.  
그래서 이렇게 시도해보니 오류가 남

```
<tr> cannot appear as a child of <table>. Add a <tbody>, <thead> or <tfoot> to your code to match the DOM tree generated by the browser.
```

리액트 관련 오류인데 리액트에서는 thead, tbody 태그를 생략할 수 없다고 한다  
없어도 화면에 나오긴 하는데 콘솔 오류나면 거슬리니까 tbody로 감싸줌

`tr`은 행 역할을 하고 `tr`을 쌓아주면
| |
|-|
|행행|
|행행|

이런 두줄짜리 테이블이 나올 수 있음

내가 고민했던게 4행까지는 두 섹션(두 열)로 나뉘어야 한다는건데
이걸 어떻게하나... 구역잡아서 grid나 flex로 나눠야되나 생각했는데
이렇게 스타일링으로 해결보겠다는 사고를 하니까 해결이 더 오래걸림

```html
<tr>
  <th>타이틀</th>
  <td>테스트</td>
  <th>타이틀</th>
  <td>테스트</td>
</tr>
```

이렇게 하니까 한 행 안에서 두개의 섹션으로 분리할 수 있었다.
`th`는 데이터의 제목이 들어가고 `td` 는 데이터가 들어가는 태그

특정 칸을 늘리고 싶으면 `colSpan={늘릴 칸 수}` 프로퍼티 주기