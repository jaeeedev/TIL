# ag-grid

어제는 html테이블을 공부해서 폼을 하나 만들었는데
동적인 데이터를 다룰 경우를 고려해서 테이블 라이브러리도 공부

리액트 쿼리를 써봤으니 같은곳에서 나온 리액트 테이블을 써보자 싶어서 들어갔는데 공식문서가 너무,,, 어려웠음 😂 ag-grid 선택

## install

```
npm install --save ag-grid-community
npm install --save ag-grid-react
```

## 기본 세팅

```jsx
import "ag-grid-community/styles/ag-grid.css"; // 코어 css, 꼭 필요
import "ag-grid-community/styles/ag-theme-alpine.css"; // 테마 적용
```

css 입히기, theme는 따로 import하지않고 직접 스타일링해도 된다.  
테이블의 width와 height는 기본 100%, 100%으로 부모 요소를 꽉 채우기 때문에 고정된 사이즈의 테이블을 만든다면 따로 css를 지정해줄 것

```jsx
const [rowData, setRowData] = useState();
...

useEffect(() => {
  fetch("https://jsonplaceholder.typicode.com/posts")
    .then((result) => result.json())
    .then((rowData) => setRowData(rowData));
}, []);

...

<AgGridReact
  rowData={rowData}
  ...
/>;
```

데이터를 받아온 후 `setRowData`로 저장한다. 그리고 rowData를 <AgGridReact/> 컴포넌트의 rowData 프로퍼티로 전달한다.

```jsx
const [columnDefs, setColumnDefs] = useState([
  { field: "title", filter: true },
  { field: "body", filter: true },
]);

const defaultColDef = useMemo(() => ({
  sortable: true,
}));

...
 <AgGridReact
          rowData={rowData}
          columnDefs={columnDefs}
          defaultColDef={defaultColDef}
          animateRows={true}
        />
```

`useState`와 `useMemo`를 사용하지 않는 경우 렌더링이 될 때 변수가 새로 만들어지면서 테이블의 설정이 변경되는 등 예상치 못한 문제가 발생할 수 있음

### 전체 코드

```jsx
import "ag-grid-community/styles/ag-grid.css";
import "ag-grid-community/styles/ag-theme-alpine.css";

const Table = () => {
  const gridRef = useRef();
  const [rowData, setRowData] = useState();

  const [columnDefs, setColumnDefs] = useState([
    { field: "title", filter: true },
    { field: "body", filter: true },
  ]);

  // DefaultColDef sets props common to all Columns
  const defaultColDef = useMemo(() => ({
    sortable: true,
  }));

  // Example of consuming Grid Event
  const cellClickedListener = useCallback((event) => {
    console.log("cellClicked", event);
  }, []);

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/posts")
      .then((result) => result.json())
      .then((rowData) => setRowData(rowData));
  }, []);

  // Example using Grid's API
  const buttonListener = useCallback((e) => {
    gridRef.current.api.deselectAll();
  }, []);

  return (
    <div>
      <button onClick={buttonListener}>click</button>

      <Wrapper className="ag-theme-alpine">
        <AgGridReact
          ref={gridRef}
          rowData={rowData}
          columnDefs={columnDefs}
          defaultColDef={defaultColDef}
          animateRows={true}
          rowSelection="multiple"
          onCellClicked={cellClickedListener}
        />
      </Wrapper>
    </div>
  );
};
export default Table;
```

## column 움직임(드래그) 막기

```jsx
const defaultColDef = useMemo(
  () => ({
    sortable: true,
    suppressMovable: true,
  }),
  []
);
```

column 옵션으로 `suppressMovable: true` 추가
위 코드는 전체 컬럼이 움직이지 않게 하는 예시
특정 컬럼만 움직이지 않게 하려면 상태 개별 옵션에 추가

## 컬럼 리사이즈

기본 상태에서 컬럼의 넓이가 고정되어있기 때문에 테이블에 맞게 채워주는 것이 좋음.

**sizeColumnsToFit 사용하기**

```jsx
<AgGridReact
...
  onGridReady={(e) => {
    ref={gridRef}
    e.api.sizeColumnsToFit();
    // ref를 설정한 경우 gridRef.current.api.sizeColumnsToFit() 으로도 호출할 수 있음
  }}
/>
```

자동으로 테이블에 맞춰 컬럼들이 늘어나도록 사이즈를 조절해준다.
테이블이 렌더링된 시점에 실행되는 이벤트 핸들러이기 때문에 컬럼의 넓이가 완전히 고정되어 브라우저가 resize되는 경우에 넓이가 변하지 않는다. (`window.resize` 이벤트를 통해 보완 가능)

**flex 사용하기**
css의 `flex-grow`와 같은 효과를 낼 수 있음. 개별 컬럼 옵션에 적용한다.

```jsx
const [columnDefs, setColumnDefs] = useState([
  { headerCheckboxSelection: true, checkboxSelection: true, width: 50 },
  { field: "title", filter: true, flex: 1 },
  { field: "body", filter: true, flex: 1 },
]);
```

모든 컬럼의 너비가 같아야 한다면 defaultColDef에 추가하여 한번에 적용시킬 수 있다. 브라우저가 resize 되어도 컬럼의 너비가 변경된다.

## 컬럼 체크박스

```jsx
const [columnDefs, setColumnDefs] = useState([
  { headerCheckboxSelection: true, checkboxSelection: true, width: 50 },
  { field: "title", filter: true, flex: 1 },
  { field: "body", filter: true, flex: 1 },
]);
```

`headerCheckboxSelection: true` 는 헤더에 체크박스(전체선택)추가 `checkboxSelection`는 개별 row에 체크박스 추가

## 체크박스 이벤트 추가하기

## n개씩 보기(pagination)

페이지네이션으로 몇개씩 보이도록 할 수 있다

```jsx
// <AgGridReact pagination={true} paginationPageSize={20} />
```

`pagination={true}`와 `paginationPageSize={개수}` 설정

![image](https://user-images.githubusercontent.com/72128840/233017549-b641c974-4102-4346-9d49-1ea6a6437156.png)

테이블에 페이지네이션이 적용됨
불러온 데이터를 기준으로 구현하기 때문에 요청을 나눠서 하는 식으로 페이지네이션을 구현한 경우에는 다른 방법을 사용해야 할듯..?

![image](https://user-images.githubusercontent.com/72128840/233022657-a8383742-3741-4d7c-9353-793b79467cd6.png)

이런 드롭다운을 추가해서 보이는 개수를 변경하려면

```jsx
const [itemsPerPage, setItemsPerPage] = useState(20);

const changePerPage = useCallback((e) => {
  setItemsPerPage(Number(e.target.value));
}, []);

...

<select onChange={changePerPage} defaultValue={itemsPerPage}>
        <option value="10">10개씩</option>
        <option value="20">20개씩</option>
        <option value="30">30개씩</option>
        <option value="40">40개씩</option>
      </select>
      <Wrapper className="ag-theme-alpine">
        <AgGridReact
         ...
          paginationPageSize={itemsPerPage}
        />
```

상태를 이용하면 됨  
체크 여부나 정렬된 상태까지 저장이 된 채로 리렌더링된다.  
해당 페이지네이션 기능은 가장 상위의 row 개수만 셀 수 있어서 그룹화 된 row의 내부 개수까지는 세지 않는다

to be continue...
