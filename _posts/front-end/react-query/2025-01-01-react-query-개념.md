---
title: React Query - 서버 상태 관리를 위한 강력한 데이터 패칭 라이브러리
date: 2025-01-01 07:20:30 +09:00
categories: [프론트엔드, React Query]
tags: [React, React Query]
---

### 1. React Query란?

**React Query**는 React 애플리케이션에서 서버 상태(server state)를 효율적으로 관리하기 위한 라이브러리이다.

데이터 패칭, 캐싱, 동기화, 갱신과 같은 복잡한 작업을 간단한 API로 처리할 수 있다.

주요 사용 사례:

- REST API 또는 GraphQL에서 데이터를 가져오기.
- 서버와 클라이언트 간 상태 동기화.
- 데이터가 오래되었거나 변경된 경우 자동으로 업데이트.

React Query를 사용하면 상태 관리와 데이터 패칭을 분리하여 더욱 **유지보수성**과 **가독성**이 높은 코드를 작성할 수 있다.

<br />

---

### 2. 주요 특징

1. **데이터 캐싱**
   - 데이터를 클라이언트에 캐싱하여, 동일한 요청에 대해 네트워크 호출을 줄인다.
2. **자동 리페치**
   - 데이터가 오래되었거나 특정 트리거 이벤트가 발생하면 자동으로 데이터를 다시 가져온다.
3. **로딩 및 오류 상태 관리**
   - 데이터를 가져오는 동안 로딩 상태와 오류를 자동으로 관리한다.
4. **Query Keys**
   - 데이터를 고유하게 식별하는 키를 기반으로 캐싱과 동기화 전략을 정의할 수 있다.
5. **Mutation 지원**
   - 데이터 생성, 업데이트, 삭제 작업을 위한 Mutation API를 제공한다.
6. **배경에서 데이터 동기화**
   - 사용자가 화면을 보는 동안 데이터가 최신 상태로 유지되도록 배경에서 자동으로 동기화한다.
7. **Devtools 지원**
   - React Query Devtools를 사용하여 쿼리 상태를 시각적으로 확인하고 디버깅할 수 있다.

<br />

---

### 3. 설치

React Query는 NPM 또는 Yarn을 사용해 설치할 수 있다.

```bash
npm install @tanstack/react-query
or
yarn add @tanstack/react-query
```

React Query Devtools도 함께 설치하려면 다음 명령어를 사용한다.

```bash
npm install @tanstack/react-query-devtools
```

<br />

---

### 4. 기본 사용법

### 4.1 QueryClient와 QueryClientProvider 설정

React Query를 사용하려면 `QueryClient`와 `QueryClientProvider`로 애플리케이션을 감싸야 한다.

```tsx
import React from "react";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <h1>React Query Example</h1>
    </QueryClientProvider>
  );
}

export default App;
```

<br />

---

### 4.2 데이터 패칭

`useQuery` 훅을 사용하여 데이터를 가져온다.

```tsx
import React from "react";
import { useQuery } from "@tanstack/react-query";

const fetchPosts = async () => {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  if (!response.ok) throw new Error("Network response was not ok");
  return response.json();
};

function Posts() {
  // useQuery 훅을 사용하여 데이터를 가져온다.
  const { data, isLoading, error } = useQuery(["posts"], fetchPosts);

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <ul>
      {data.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

export default Posts;
```

<br />

---

### 5. 주요 API

### 5.1 `useQuery`

`useQuery`는 데이터를 가져오고 상태를 관리하는 데 사용된다.

```tsx
const { data, error, isLoading } = useQuery(
  ["queryKey"], // 캐싱과 데이터 식별을 위한 고유 키
  fetchFunction, // 데이터를 가져오는 비동기 함수
  options // 선택적으로 제공되는 설정 객체
);
```

옵션 예제:

```tsx
useQuery(["posts"], fetchPosts, {
  staleTime: 10000, // 데이터가 신선한 상태로 간주되는 시간 (ms)
  cacheTime: 300000, // 데이터가 캐시에 유지되는 시간 (ms)
  retry: 3 // 요청 실패 시 재시도 횟수
});
```

<br />

---

### 5.2 `useMutation`

`useMutation`은 데이터를 생성, 업데이트, 삭제하는 데 사용된다.

```tsx
import { useMutation } from "@tanstack/react-query";

const createPost = async (newPost) => {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
    method: "POST",
    body: JSON.stringify(newPost),
    headers: {
      "Content-Type": "application/json"
    }
  });
  if (!response.ok) throw new Error("Failed to create post");
  return response.json();
};

function CreatePost() {
  const mutation = useMutation(createPost, {
    onSuccess: () => {
      console.log("Post created successfully!");
    }
  });

  const handleCreate = () => {
    mutation.mutate({ title: "New Post", body: "This is a new post" });
  };

  return (
    <div>
      <button onClick={handleCreate}>Create Post</button>
      {mutation.isLoading && <p>Creating post...</p>}
      {mutation.isError && <p>Error: {mutation.error.message}</p>}
    </div>
  );
}

export default CreatePost;
```

<br />

---

### 5.3 Query Keys

React Query는 **Query Key**를 사용하여 데이터를 캐싱하고 식별한다.

- Query Key는 배열 또는 문자열로 정의할 수 있다.
- Query Key를 활용해 캐싱, 무효화, 갱신 등의 작업을 제어할 수 있다.

```tsx
useQuery(["posts", userId], fetchPostsByUser);
```

<br />

---

### 6. React Query Devtools

React Query Devtools는 쿼리 상태를 시각적으로 확인할 수 있는 도구이다.

### 설치 및 사용

```tsx
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";

<QueryClientProvider client={queryClient}>
  <App />
  <ReactQueryDevtools initialIsOpen={false} />
</QueryClientProvider>;
```

<br />

---

### 7. 장점과 단점

### 장점

1. 서버 상태 관리에 특화된 기능 제공.
2. 데이터 패칭과 상태 관리를 간단히 처리.
3. 강력한 캐싱 및 동기화 메커니즘.
4. 네트워크 오류 및 로딩 상태 자동 관리.
5. Devtools를 통한 디버깅 지원.

### 단점

1. 클라이언트 상태 관리에는 적합하지 않음.
2. 초기 설정(Provider 등)이 필요함.
3. 간단한 데이터 패칭에는 과할 수 있음. (SWR 사용 추천)

<br />

---

### 8. React Query vs SWR

| **특징**                  | **React Query**                          | **SWR**                             |
| ------------------------- | ---------------------------------------- | ----------------------------------- |
| **주요 목적**             | CRUD 작업 및 복잡한 데이터 관리          | 읽기 전용 데이터와 단순한 상태 관리 |
| **규모**                  | 대규모 프로젝트                          | 소규모 프로젝트                     |
| **서버 데이터 변경 빈도** | 데이터 변경 및 동기화가 빈번             | 데이터 변경이 적고, 캐싱이 중요     |
| **API 인터랙션**          | 서버와의 빈번한 상호작용 (Mutation 지원) | 단순한 데이터 패칭                  |
| **주요 예시**             | E-commerce, 프로젝트 관리 툴             | 블로그, 뉴스 피드                   |

React Query는 대규모 애플리케이션에서 복잡한 데이터 로직을 다룰 때 유리하고, SWR은 가볍고 빠른 데이터 조회가 필요한 프로젝트에서 유리하다. 프로젝트의 요구사항과 규모를 고려하여 적합한 라이브러리를 선택하면 된다.

<br />

---

### 9. 정리

React Query는 **서버 상태 관리**를 최적화하여 React 애플리케이션에서 데이터 패칭, 캐싱, 동기화를 간단히 처리할 수 있도록 돕는다.

특히, 데이터 갱신이나 복잡한 상태 관리가 필요한 프로젝트에 적합하다.

간단한 데이터 패칭이 주된 작업이라면 **SWR**을, 더 복잡한 상태 관리와 서버 동기화가 필요하다면 **React Query**를 사용하는 것이 좋다.
