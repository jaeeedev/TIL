# 쿠키

내가만든쿠키 ~~~~~

쿠키는 서버가 생성하고 응답 헤더에 Set-Cookie로 값을 넣어 전달하면 브라우저는 알아서 쿠키를 저장

## 쿠키의 목적

- **세션 관리**
- 개인화
- 트래킹

쿠키는 모든 요청마다 함께 전송되기 때문에 성능 저하의 원인이 될 수 있다. 단순한 정보 저장 목적이라면 로컬 스토리지, 세션 스토리지, indexedDB를 사용하는 것이 좋다.

## 쿠키의 구성

`키-값` 쌍으로 이루어져 있음, 특수문자 사용 불가

### expires

쿠키가 파기되는 시간, Date 형태  
지정하지 않으면 세션 쿠키(브라우저 끄면 사라짐)  
유효 일자는 반드시 GMT 포맷이어야 하며 `date.toUTCString`으로 쉽게 변환 가능

```js
let date = new Date(Date.now() + 86400e3);
//86400e3은 하루를 밀리초 단위로 변환한 수
date = date.toUTCString();
document.cookie = "user=John; expires=" + date;
```

값을 과거로 주면 쿠키가 삭제됨

### max-age

쿠키가 만료될 때 까지의 시간
Expires와 Max-Age가 둘 다 지정된 경우 Max-Age가 더 우선됨. 음수가 되면 쿠키가 즉시 만료

```js
// 1시간 뒤 쿠키가 삭제됨
document.cookie = "user=John; max-age=3600";

// 쿠키를 즉시 삭제
document.cookie = "user=John; max-age=0";
```

### path

말 그대로 경로
경로를 지정하면 그 외 경로에서는 쿠키를 볼 수 없음
특별한 경우가 아니라면 `path=/` 일케 루트로 설정해놓고 모든 페이지에서 접근 가능하게 할 것

### domain

쿠키에 접근 가능한 도메인을 지정
설정하지 않으면 쿠키를 설정한 도메인에서만 접근 가능
서브 도메인에서도 접근하게 하고 싶다면

```js
// 서브 도메인(*.site.com) 어디서든 접속 되도록
document.cookie = "user=John; domain=site.com";
```

#### 서브 도메인

메인 도메인에서 분리하여 보조적인 역할을 수행할 수 있는 도메인

- m.naver.com
- blog.naver.com
- store.naver.com

이런거처럼 \*\*\*/naver.com 형태로 앞만 바꿔서 서브 도메인을 쓸 수 있음

### secure

이 옵션 있을 경우 HTTPS로 통신하는 경우에만 쿠키가 전송

```js
document.cookie = "user=John; secure";
```

## 브라우저에서 쿠키 조회하기

```js
document.cookie;
```

간단한데 별안간 복잡한 값들이 다 나옴

## 브라우저에서 쿠키 쓰기

```js
document.cookie = "user=jaeee";
```

직접 값을 쓸 수 있는데 모든 값이 다 뒤집히는것이 아니라 이런 케이스에선 user의 값만 지정한 값으로 바뀐다 이건 좋은듯

## 쿠키 라이브러리

쿠키는 꼴이 지저분하게 생겼다.

```js
document.cookie;

//'pixelRatio=1.25; _ga=GA1.2.******; _ym_uid=********; _ym_d=*****; __stripe_mid=40187ffb-64a6-41f7-8754-*****; _gid=GA1.2.********; _ym_isad=2; user=John'
```

그래서 원하는 키값으로 정보를 뽑으려면 정규표현식을 써서 뽑아내야 하는데 쉽지 않다.

```js
function getCookie(name) {
  let matches = document.cookie.match(
    new RegExp(
      "(?:^|; )" +
        name.replace(/([\.$?*|{}\(\)\[\]\\\/\+^])/g, "\\$1") +
        "=([^;]*)"
    )
  );
  return matches ? decodeURIComponent(matches[1]) : undefined;
}
```

이런 엄청난 코드가 필요함
`react-cookie` 라이브러리같은거 쓰면 간단하게 세팅하고 받아오기 가능
