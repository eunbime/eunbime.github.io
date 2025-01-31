---
title: React의 컴포넌트 라이프사이클
date: 2025-01-31 10:00:00 +09:00
categories: [프론트엔드, React]
tags: [React]
---

# React 컴포넌트 라이프사이클

React 컴포넌트의 라이프사이클은 컴포넌트가 생성되고 사라지기까지의 전체적인 과정을 의미한다. 이를 이해하는 것은 React 애플리케이션을 효율적으로 개발하는 데 매우 중요하다.

## 라이프사이클의 3단계

React 컴포넌트의 라이프사이클은 크게 3단계로 구분된다.

### 1. 마운트 (Mount)

컴포넌트가 처음으로 DOM에 렌더링되는 단계이다. 이 시점에서는 주로 다음과 같은 작업들이 수행된다.

- 초기 상태 설정
- API 호출
- 이벤트 리스너 등록

```jsx
useEffect(() => {
  // 초기화 작업 수행
  console.log("컴포넌트가 마운트되었습니다");
}, []); // 빈 의존성 배열
```

### 2. 업데이트 (Update)

컴포넌트의 props나 state가 변경되어 리렌더링이 발생하는 단계이다. 다음과 같은 경우에 발생한다.

- props 변경
- setState() 호출
- forceUpdate() 사용

```jsx
useEffect(() => {
  console.log("데이터가 업데이트되었습니다");
}, [data]); // data가 변경될 때마다 실행
```

### 3. 언마운트 (Unmount)

컴포넌트가 DOM에서 제거되는 단계이다. 주로 다음과 같은 정리 작업이 수행된다.

- 이벤트 리스너 제거
- API 구독 해제
- 타이머 정리

```jsx
useEffect(() => {
  return () => {
    // 정리 작업 수행
    console.log("컴포넌트가 언마운트됩니다");
  };
}, []);
```

## 실제 활용 예시

실제 프로젝트에서는 다음과 같이 라이프사이클을 활용할 수 있다.

```jsx
function DataFetchingComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    // 마운트: API 데이터 가져오기
    const fetchData = async () => {
      const response = await fetch("<https://api.example.com/data>");
      const result = await response.json();
      setData(result);
    };

    fetchData();

    // 언마운트: 정리 작업
    return () => {
      console.log("데이터 구독 해제");
    };
  }, []);

  return <div>{data ? <p>{data.message}</p> : <p>로딩 중...</p>}</div>;
}
```

## 결론

라이프사이클을 잘 이해하고 활용하면 다음과 같은 이점이 있다.

- 효율적인 리소스 관리
- 메모리 누수 방지
- 최적화된 렌더링 성능
