# 테스트 코드

```js
import { render, screen, fireEvent } from "@testing-library/react";

test("button has correct initial color", () => {
  render(<App />);

  const colorButton = screen.getByRole("button", {
    name: "Change to Midnight Blue",
  });

  expect(colorButton).toHaveStyle({
    backgroundColor: "MediumVioletRed",
  });

  fireEvent.click(colorButton);

  expect(colorButton).toHaveStyle({ backgroundColor: "MidnightBlue" });
  expect(colorButton).toHaveTextContent("Change to Medium Violet Red");
});
```

버튼 컴포넌트를 테스트한다고 치면 어떤 클래스네임을 가진 버튼을 선택하는 방식이 아니라 어떤 **텍스트**를 가진 버튼(`getByRole`로 기능을 확인)인지를 체크하여 동작시킨다.

```
최초 색상,텍스트 확인 - 클릭(`fireEvent`) - 변경된 색상, 텍스트 확인
```

이런 구조라서 시나리오를 쓰듯 테스트 할 수 있다.

## 현재 습득한 수준

- `render`로 컴포넌트를 렌더하고 `screen`을 통해 원하는 요소를 탐색
- `fireEvent`, `user-event`를 통해 이벤트를 발생시킬 수 있음
- jest의 `toHaveStyle`, `toBeEnabled`, `not`등의 매처를 활용

## fireEvent와 user-event의 차이

- fireEvent는 DOM의 이벤트를 디스패치
- user-event는 모든 상호작용을 시뮬레이션(좀 더 사용자 관점), promise를 반환

## user-event

```js
import userEvent from "@testing-library/user-event";

test("...", async () => {
  const user = userEvent.setup();

  await user.click(element);
});
```

`user-event`를 사용하기 위해서는 `setup` 메서드를 사용해 유저 객체를 만들고
객체 내부의 이벤트 메서드를 사용해야 한다.  
그리고 이 이벤트들은 promise를 반환하기 때문에 await을 꼭 써주어야 한다(당연히 테스트의 콜백 함수도 async 함수여야 함)

## getBy와 queryBy의 차이점

`getBy`는 일치하는 요소가 없거나 여러개인 경우 **오류**를 발생시킨다.
반면에 `queryBy`는 일치하는 요소가 없으면 **null**을 반환한다. 특정 이벤트(호버)가 발생했을 때만 나타나는 팝업같은 요소를 테스트할 때 사용한다.
