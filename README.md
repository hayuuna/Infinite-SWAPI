# Infinite SWAPI

💡 **서버: Star Wars API**

- 무한 스크롤에 알맞은 포맷으로 데이터 반환
- React Query 라이브러리 설치
- Query Client 만들고 App.js에 provider 추가
- App.js에 개발자 도구 추가

<br />

### 1. 무한 스크롤

- 사용자가 페이지의 특정 지점을 누르거나 버튼을 클릭하면 많은 데이터가 로딩됨
- React Query는 데이터와 페이지 번호를 추적하게 됨
- 사용자가 스크롤시 데이터를 불러옴
- 모든 데이터를 한번에 가져오는 것 보다 효율적

**useInfiniteQuery**

- 다음 쿼리가 뭘지 추적함
- 다음 쿼리가 데이터의 일부로 반환됨
- 객체를 가지게 됨 → 데이터 배열을 가진 결과라는 프로퍼티를 가짐, 데이터의 다음 페이지로 가려면 어느 쿼리를 사용할지, 이전 페이지로 가려면 어느 쿼리를 쓸지 알려줌(React Query 페이지 관리 방법을 지시하는데 유용)

**Pagination**

- 현재 페이지를 컴포넌트 상태에서 추적함
- 사용자가 새 페이지를 열기 위해 버튼 클릭 시 업데이트 된 상태가 쿼리 키를 업데이트 하고 쿼리 키가 데이터를 업데이터 함
- 블록 데이터: 데이터의 배열

| useQuery | useInfiniteQuery |
| --- | --- |
| 데이터는 쿼리 함수에서 반환된 데이터 | 객체는 두 개의 프로퍼티를 가짐
  1. pages: 데이터 페이지 객체의 배열인 페이지(페이지에 있는 각 요소가 하나의 useQuery에서 받는 데이터에 해당)
  2. pageParams: 각 페이지의 매개변수가 기록되어 있음 |
- 반환 객체에서 반환된 데이터 프로퍼티의 형태가 다름
- 모든 쿼리는 페이지 배열에 고유한 요소를 가지고 있고 그 요소는 해당 쿼리에 대한 데이터에 해당 → 페이지가 진행되면서 쿼리도 바뀜
- pageParams는 검색된 쿼리의 키를 추적
    - 흔하게 사용되지 않음

**useInfiniteQuery 구문**

```jsx
useInfiniteQuery("sw-people", ({ pageParam = defaultUrl }) => fetchUrl(pageParam))
```

- pageParams는 쿼리 함수에 전달되는 매개변수
    - useInfiniteQuery에 쿼리 키는 sw-people가 되고, 쿼리 함수가 실행되는 동안 매개변수, 객체를 구조 분해한 pageParam을 사용함
    - 첫번째 Url로 정의한 Url을 기본값으로 설정
    - defaultUrl을 기본값을로 하는 pageParam을 사용해서 해당 pageParam에서 fetchUrl을 실행함
    - fetchUrl이 pageParam을 사용해서 적절한 페이지를 찾아 가져옴
    - React Qurey가 pageParam의 현재 값을 유지함(컴포넌트 상태 값의 일부가 아님)
- useInfiniteQuery에 옵션을 사용하는 방식으로 작업 실행
    - getNextPageParam: 다음 페이지로 가는 방식을 정의하는 함수, (lastPage, allPage) 마지막 페이지의 데이터나 모든 페이지의 데이터에서 가져옴, 마지막 페이지의 데이터를 사용하면 다음 페이지의 URL이 무엇인지 알려줌
        - pageParam을 업데이트해 줌
        - 데이터의 모든 페이지를 사용할 수 있음
        - 데이터의 마지막 페이지만 사용
- 반환 객체의 프로퍼티
    - **fetchNextPage**
        - 사용자가 더 많은 데이터를 요청할 때 호출하는 함수, 더 많은 데이터를 요청하는 버튼을 클릭하거나 스크린에서 데이터가 소진되는 지점을 누르는 경우
    - **hasNextPage**
        - getNextPageParam 함수의 반환값을 기반으로 함, 이 프로퍼티를 useInfiniteQuery에 전달해서 마지막 쿼리의 데이터를 어떻게 사용할지 지시
        - undefined인 경우 더이상 데이터가 없음
        - useInfiniteQuery에서 반환 객체와 함께 반환된 경우 hasNextPage는 거짓임
    - **isFetchingNextPage**
        - useQuery에는 없음
        - useInfiniteQuery는 다음페이지를 가져오는지 일반적인 페칭인지 구별할 수 있음(다음 페이지 페칭과 현재 페이지든 다음이든 상관없는 일반적인 페칭을 구별하는 것이 유용)
        

**useInfiniteQuery의 흐름**

1. 컴포넌트 마운트

```jsx
const { data } = useInfiniteQuery(...)

// data: undefined
```

- useInfiniteQuery이 반환한 객체의 data 프로퍼티가 정의되어 있지 않음: **쿼리를 만들지 않았기 때문**

2. 쿼리 함수를 사용해서 첫 페이지를 가져옴

```jsx
useInfiniteQuery({ pageParam = defaultUrl } => ... )

// pageParam: default
```

- **쿼리 함수**는 useInfiniteQuery의 첫번째 인수, **pageParam**을 인수로 받음
- 첫 pageParam은 이 요소에서 기본값으로 정의한 것

```jsx
data.pages[0]: { ... }
```

- 해당 pageParam을 사용해서 첫번째 페이지를 가져오고 반환 객체 데이터의 페이지 프로퍼티를 설정
- 인덱스가 0인 배열의 첫번째 요소를 설정
- { … }는 쿼리 함수가 반환하는 값

3. 데이터 반환된 후 React Query가 getNextPageParam을 실행

```jsx
getNextPageParam: (lastPage, allPage) => ...
```

- lastPage와 lastPage를 사용해 pageParam을 업데이트 하는 함수

```jsx
pageParam: "http://swapi.dev/api/species/?page=2"
```

- 데이터는 첫 페치에서 받아 getNextPageParam에 전달하면 다음 페이지에서 pageParam을 업데이트함

4. hasNextPage
- 존재함 → React Query가 hasNextPage의 값을 결정하는 방식이 pageParam이 정의되어 있는지 아닌지에 따르기 때문
- pageParam이 정의되어 있어 hasNextPage가 존재

5. 사용자가 페이지에서 스크롤 또는 버튼을 클릭할 때 fetchNextPage 함수를 트리거 하는 행동을 함
- fetchNextPage 함수는 useInfiniteQuery가 반환하는 객체의 속성
- 구조분해 하지 않음
- fetchNextPage을 호출하여 사용자가 더 많은 데이터를 요청할 때 함 →  React Query가 쿼리 함수 실행
- pageParam이 뭐든지(현재는 2페이지를 요청)
- 사용해서 다음 요소를 업데이트 하거나 데이터의 프로퍼티인 페이지 배열에 다음 요소 추가
- 새 데이터를 가지고 getNextPageParam을 실행 nextPageParam을 설정

→ 두 페이지의 데이터만 있다고 가정했을 때 getNextPageParam을 실행하면 nextPageParam는 정의되지 않음

```jsx
pageParam: undefined
```

- 받은 새 데이터를 보면 더이상 페이지가 없다고 나옴(마지막 데이터를 getNextPageParam함수에 줄 때 )
→ pageParam이 undefined이기 때문에 hasNextPage는 거짓이 됨
→ hasNextPage이 거짓: 작업 완료(더이상 수집할 데이터가 없음)

<br />

### 2. useInfiniteQuery 호출

```jsx
import { useInfiniteQuery } from "react-query";
```

불러온 useInfiniteQuery를 함수형 컴포넌트에서 호출함

```jsx
const {
    data,
    fetchNextPage,
    hasNextPage,
    isLoading,
    isFetching,
    isError,
    error,
  } = useInfiniteQuery(
    "sw-species",
    ({ pageParam = initialUrl }) => fetchUrl(pageParam),
    {
      getNextPageParam: (lastPage) => lastPage.next || undefined,
    }
  );

if (isLoading) return <div className="loading">Loading...</div>;
if (isError) return <div>Error! {error.toString()}</div>;
```

- 구조분해
- 반환된 객체에서 필요한 것은 data
- 페이지를 계속 로드할 때 데이터 페이지가 포함됨
- **fetchNextPage**: 더 많은 데이터가 필요할 때 어느 함수를 실행할지를 지시하는 역할
- **hasNextPage**: 수집할 데이터가 더 있는지 결정하는 불리언임
- 첫번째 인수: 쿼리키
- 두번째 인수: 쿼리 함수(객체 매개변수를 받고 프로퍼티 중 하나로 pageParam을 가지고 있음)
- pageParam은 fetchNextPage이 어떻게 보일지 결정하고 다음 페이지가 있는지 결정함
- fetchUrl: Url인 pageParam을 가져와서 json을 반환해 줌
- pageParam은 기본값을 줌
- useInfiniteQuery를 처음 실행할 땐 pageParam이 설정돼있지 않고 기본 값이 initialUrl
- getNextPageParam에 옵션을 줌, 이 옵션은 lastPage를 가진 함수(필요시 allPage를 두번째 인자로 넣을 수 있음)
- lastPage를 가져와 pageParam을 lastPage.next로 작성하고 fetchNextPage를 실행하면 next 속성이 무엇인지에 따라 마지막 페이지에 도착한 다음 pageParam을 사용하게 됨
- hasNextPage: (lastPage) => lastPage.next 함수가 undefined를 반환하는지에 아닌지에 따라 결정이 됨
- lastPage.next가 거짓인 경우 undefined로 반환

```jsx
const initialUrl = "https://swapi.dev/api/people/";
```

![image](https://github.com/hayuuna/base-blog-em/assets/144312023/ffb4c1d2-f7b0-4e4c-874d-f9440acd7c3c)


- 반환될 데이터
- next 속성을 가짐 → 다음 페이지가 무엇인지, 결과의 다음페이지로 가는데 필요한 Url이 무엇인지 알려줌

<br />

### 3. InfiniteScroll 컴포넌트

[공식문서](https://www.npmjs.com/package/react-infinite-scroller)

- infinite Scroller를 사용하면 useInfiniteQuery와 호환이 잘 됨
- 두 개의 속성이 존재
    - **loadMore={fetchNextPage}**: 데이터가 더 필요할 때 불러와 useInfiniteQuery에서 나온 fetchNextPage 함숫값을 이용함
    - **hasMore=**{**hasNextPage}**: useInfiniteQuery에서 나온 객체를 해체한 값을 이용함
- 스스로 페이지 끝에 도달했음을 인식하고 fetchNextPage를 불러오는 기능
- 데이터 속성에서 데이터에 접근할 수 있음 → useInfiniteQuery 컴포넌트에서 나온 객체 이용, 배열을 이용해 그 페이지 배열의 맵을 만들어 데이터를 표시할 수 있게 해 줌

```jsx
import InfiniteScroll from "react-infinite-scroller";
```

```jsx
{isFetching && <div className="loading">Loading...</div>}
      <InfiniteScroll loadMore={fetchNextPage} hasMore={hasNextPage}>
        {data.pages.map((pageData) => {
          return pageData.results.map((species) => {
            return (
              <Species
                key={species.name}
                name={species.name}
                language={species.language}
                averageLifespan={species.average_lifespan}
              />
            );
          });
        })}
      </InfiniteScroll>
```

<br />

### 4. 양방향 스크롤

- 데이터의 중간부터 시작할 때 유용함
- 시작점 이후 뿐 아니라 이전의 데이터도 가져와야 함
- 모든 next 메서드들과 프로퍼티와 동일한 (fetchNextPage, hasNextPage, getNextPageParam 함수와 같은) previos를 사용하는 똑같은 함수들이 존재 → 이전 페이지에 대해서도 동일한 기능 수행 가능: 둘다 수행 시 양방향 스크롤이 됨(시작점 이전, 이후 데이터를 모두 불러옴)
