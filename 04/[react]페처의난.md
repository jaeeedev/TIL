## fetcher

어떻게 하면 귀찮지 않게 api 요청을 날릴 수 있는가

```ts
type AnyOBJ = { [key: string]: any };
const BASE_URL = "/";

export const restFetcher = async ({
  method,
  path,
  body,
  params,
}: {
  method: "GET" | "POST" | "PUT" | "DELETE" | "PATCH";
  path: string;
  body?: AnyOBJ;
  params?: AnyOBJ;
}) => {
  try {
    let url = `${BASE_URL}${path}`;
    const fetchOptions: RequestInit = {
      method,
      headers: {
        "Content-Type": "application/json",
        "Access-Control-Allow-Origin": BASE_URL,
      },
    };
    if (params) {
      const searchParams = new URLSearchParams(params);
      url += "?" + searchParams.toString();
      // params가 들어온 경우 url 뒤에 파라미터를 붙여서 요청에 보내주기
    }
    if (body) fetchOptions.body = JSON.stringify(body);
    const res = await fetch(url, fetchOptions);
    const json = await res.json();
    return json;
  } catch (err) {
    console.error(err);
  }
};
```

강의 듣다가 본 페처  
method랑 body param 까지 다 때려넣어서 범용적이다

```ts
export const baseUrl = "http://localhost:포트번호";

const config: AxiosRequestConfig = { baseURL: baseUrl };
export const axiosInstance = axios.create(config);

async function getUser(user: User | null, signal: AbortSignal): Promise<User> {
  if (!user) return null;
  const { data }: AxiosResponse<{ user: User }> = await axiosInstance.get(
    `/user/${user.id}`,
    {
      headers: getJWTHeader(user),
      signal,
    }
  );

  return data.user;
}
```

또다른 강의에서 본 페처...  
axios 인스턴스를 찍어서 원하는 속성을 미리 지정해놓고 쓰는 방법이 있음

```ts
import axios from "axios";

const fetcher = (url: string) =>
  axios
    .get(url, {
      withCredentials: true,
    })
    .then((res) => res.data);

export default fetcher;
```

또 다른 강의에서 본 페처, 제일 단순함  
get만 할거면 그냥 url만 받기도..  
이 강의는 get과 post 요청만 해서 페처를 따로 구분해 만들어도 무리가 없다.

**고민해본 결과**

```js
const axiosInstance = axios.create({
  baseURL:
    process.env === "development" ? "http://localhost:5000" : "서버 포트번호",
  timeout: 1000,
  headers: { 헤더: "넣어주기" },
});

const customFetcher = async (url, ...rest) => {
  const { method = "GET", body } = rest;
  // method의 기본값은 GET
  // url, options: {method, body} 해도 될지도..
  // (options, ...rest) 요렇게 해서 옵션으로 만들면

  const axiosOptions = {
    method,
    url,
  };

  if (body) {
    // body가 옵션에 들어온 경우 axiosOptions를 업데이트해 body를 보내줍시다

    //axios(config) 형태로 보낼때는 data에 담네

    axiosOptions.data = body;
  }

  const response = await axiosInstance(axiosOptions);

  return response.data;
};
```

여기에 try catch구문도 넣어야할텐데

- 위 코드대로 사용을 했더니 의도와 다르게 객체를 잘 찾지 못해서
  살짝 수정해줬다.

```jsx
export const fetcher = async (url, ...rest) => {
  const { method = "GET", body } = rest[0];

  const axiosOptions = {
    method,
    url,
  };

  if (method) {
    axiosOptions.method = method;
  }

  if (body) {
    axiosOptions.data = body;
  }

  try {
    const response = await axiosInstance(axiosOptions);
    return response.data;
  } catch (err) {
    console.log(err);
  }
};
```

`rest[0]`으로 해야되더라
