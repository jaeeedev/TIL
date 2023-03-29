# 최적화

## 폰트 최적화

### unicode-range

![image](https://user-images.githubusercontent.com/72128840/228398698-0bb84c6e-6dc2-40be-aaf3-e53d2b2f9271.png)

유니코드로 지정된 글자에만 웹 폰트를 적용  
한글과 영어에 적용되는 폰트를 다르게 할 때 사용할 수 있을듯

### FOIT, FOUT

`FOIT`는 폰트가 적용될 때 까지 빈 화면 -> 폰트가 적용되어 렌더링  
`FOUT`는 폴백 폰트 적용된 상태로 렌더링 -> 폰트가 적용되어 변경

`FOUT`가 UX관점에서 좀 더 낫다  
`<link rel=preload>`와 `font-display:optional`을 조합해 사용하기

### 필요한 글리프만 사용하기

![image](https://user-images.githubusercontent.com/72128840/228452613-ee5cc156-ea91-49de-962c-a455c9546670.png)

잘 쓰이지 않는 문자들은 폰트 파일에서 제외하여 용량을 줄일 수 있다.  
내가 사용한 프리텐다드는 다행히도 서브셋 파일을 자체적으로 제공하고 있어서 간편하게 용량 최적화를 할 수 있었다.  
서브셋 파일은 모든 글리프를 지원하는 원본 파일 대비 3분의 1 정도인 가벼운 용량을 가지고 있다.

## 이미지 최적화

### 정보 제공

- 이미지에 너비와 높이 설정하여 layout shift 방지
- 이미지에 aspect-ratio 속성 제공

### 이미지 지연 로딩

이미지 지연 로딩에는 두가지 방법이 있다.

- **`intersection Observer API` 이용하기**

```jsx
<img src={...} alt="지연 로딩" data-src="url" src="fallback-src" ref={imgRef} />

imgRef.current.src = imgRef.current.dataset.src;
```

대략적인 흐름

상대적으로 가벼운 폴백 이미지를 먼저 띄우고 data-src같은 커스텀 속성에 지연로딩할 이미지의 경로를 저장해둔다. 뷰포트에 이미지 요소가 들어오면 경로를 변경하여 이미지를 로드한다.

```jsx
const baseUrl = "https://picsum.photos/400/";
const fallbackUrl = "https://picsum.photos/10/";
const imgRef = useRef([]);
const observerRef = useRef();

const getObserver = useCallback(() => {
  observerRef.current = new IntersectionObserver((entries, observer) => {
    entries.forEach((entry) => {
      const { target } = entry;
      if (!entry.isIntersecting) return;

      target.src = target.dataset.src;

      observer.unobserve(target);
      // 뷰포트에서 나가는 시점까지 관찰할 필요 x
    });
  });

  return observerRef.current;
}, []);

useEffect(() => {
  imgRef.current.forEach((img) => getObserver().observe(img));
}, [imgRef.current]);

return (
  <ImgWrapper>
    {[...new Array(40)].map((_, i) => (
      <img
        ref={(el) => (imgRef.current[i] = el)}
        key={i}
        src={`${fallbackUrl}${300 + i}`}
        data-src={`${baseUrl}${300 + i}`}
        width="400"
        height="400"
      />
    ))}
  </ImgWrapper>
);
```

적용한 코드

![image](https://user-images.githubusercontent.com/72128840/228434570-f494c7a8-b1fd-4780-9b67-23d50e63696c.png)

이렇게 미리 로드된 이미지가

![image](https://user-images.githubusercontent.com/72128840/228434643-882eacfe-ee7e-4804-8400-288d48b05e7a.png)

뷰포트 내로 들어오면 제대로 된 이미지로 교체된다.

[참고](https://velog.io/@godud2604/%EC%9D%B4%EB%AF%B8%EC%A7%80-%EC%B5%9C%EC%A0%81%ED%99%94-Lazy-Load-Intersection-Observer-API)

- **img 태그의 `loading` 속성 사용하기**

이 방법은 훨씬 간단하다.

```jsx
const ImgTest = () => {
  const baseUrl = "https://picsum.photos/400/";
  return (
    <ImgWrapper>
      {[...new Array(40)].map((_, i) => (
        <img key={i} src={`${baseUrl}${300 + i}`} width="400" height="400" />
      ))}
    </ImgWrapper>
  );
};
```

40장의 이미지를 화면에 띄우도록 했다.

![image](https://user-images.githubusercontent.com/72128840/228423781-1fb1e94c-a8ae-46d2-ad95-8af353332336.png)

그냥 이미지 태그를 렌더링하면 한번에 다 불러온다. 용량이 크고 많을수록 오래 걸릴 것

-> 이미지 태그에 `loading="lazy"` 속성을 추가한다.

![image](https://user-images.githubusercontent.com/72128840/228425160-259a5241-df93-479d-afb9-2738a5cb233c.png)  
초기 로드

![image](https://user-images.githubusercontent.com/72128840/228425406-a891e151-c731-413f-9e63-0a6e3925f560.png)  
화면을 내릴 때 마다 다음 이미지를 로드한다.

`loading`의 속성
`auto`: 기본 동작 (사용할 수는 있으나 사용 지양 권장)
`lazy`: 뷰포트에서 이미지의 위치에 도달할 때 까지 리소스 로드를 지연
`eager`: 페이지의 위치와 관계없이 리소스를 즉시 로드

[참고](https://web.dev/i18n/ko/browser-level-image-lazy-loading/)

### caniuse?

**intersection Observer** 의 경우
![image](https://user-images.githubusercontent.com/72128840/228433740-79254df8-da3f-4434-aa45-faff3381edbe.png)

**loading = lazy**
![image](https://user-images.githubusercontent.com/72128840/228434141-ea01ee65-85ed-4b34-a028-b811860bbe47.png)

현재로서는 `loading` 속성보다 조~~~금 더 지원을 안정적으로 하고 있는 것 같다. 읽어보니 `Firefox only supports lazy loading for images` 라고 하는데 이미지를 최적화 할 목적이라면 문제없이 사용할 수 있는것  
firefox는 iframe 관련해서 loading 속성을 실험적 기능으로 사용할 수 있다고 하는듯하다

### 후기

나는 원래 화면 가장 아래에 투명한 하나의 관찰 요소(타겟)를 만들어놓고 그 요소가 보이면 원하는 함수를 실행시키는 식으로 사용해왔다. 무한스크롤 같은 경우는 그렇게 처리해왔는데 이번에는 여러가지 개별 img 태그들이 타겟이었기 때문에 그 방법이 적합하지 않았다. observe와 unobserve를 사용하는 방법을 드디어 알게되었다 ㅋ\_ㅋ
