---
title: React의 useEffect 실행 순서 이해하기
date: 2025-01-31 10:00:00 +09:00
categories: [프론트엔드, React]
tags: [React]
---

# React useEffect의 실행 순서 이해하기

useEffect는 React의 핵심 Hook 중 하나로, 그 실행 순서를 정확히 이해하는 것이 중요하다. 실행 순서와 각 상황별 동작 방식을 자세히 알아보자.

## 기본적인 실행 순서

```jsx
function ExampleComponent() {
  useEffect(() => {
    console.log("첫 번째 useEffect 실행");
    return () => {
      console.log("첫 번째 useEffect cleanup");
    };
  });

  useEffect(() => {
    console.log("두 번째 useEffect 실행");
    return () => {
      console.log("두 번째 useEffect cleanup");
    };
  });

  console.log("렌더링");
  return <div>예시 컴포넌트</div>;
}
```

실행 결과:

1. '렌더링'
2. '첫 번째 useEffect 실행'
3. '두 번째 useEffect 실행'

## 의존성 배열에 따른 실행 예시

```jsx
function DependencyExample() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("");

  // 마운트 시 한 번만 실행
  useEffect(() => {
    console.log("마운트 시에만 실행");
    return () => {
      console.log("언마운트 시에만 실행");
    };
  }, []);

  // count가 변경될 때마다 실행
  useEffect(() => {
    console.log("count 변경 시 실행:", count);
    return () => {
      console.log("count effect cleanup");
    };
  }, [count]);

  // 매 렌더링마다 실행
  useEffect(() => {
    console.log("매 렌더링마다 실행");
    return () => {
      console.log("매 렌더링 전 cleanup");
    };
  });

  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>카운트 증가</button>
      <input value={name} onChange={(e) => setName(e.target.value)} />
    </div>
  );
}
```

## Cleanup 함수의 실행 순서

```jsx
function CleanupExample() {
  const [visible, setVisible] = useState(true);

  useEffect(() => {
    console.log("Timer Effect 시작");
    const timer = setInterval(() => {
      console.log("타이머 실행 중...");
    }, 1000);

    return () => {
      console.log("Timer Cleanup");
      clearInterval(timer);
    };
  }, []);

  return (
    <div>
      <button onClick={() => setVisible(!visible)}>토글</button>
      {visible && <ChildComponent />}
    </div>
  );
}
```

## 실제 활용 예시

```jsx
function DataFetchingComponent() {
  const [data, setData] = useState(null);
  const [id, setId] = useState(1);

  useEffect(() => {
    // 이전 데이터 요청 취소
    let isCancelled = false;

    async function fetchData() {
      try {
        const response = await fetch(`https://api.example.com/data/${id}`);
        const result = await response.json();

        if (!isCancelled) {
          setData(result);
        }
      } catch (error) {
        if (!isCancelled) {
          console.error("에러 발생:", error);
        }
      }
    }

    fetchData();

    // cleanup: 새로운 요청 전에 이전 요청 취소
    return () => {
      isCancelled = true;
    };
  }, [id]); // id가 변경될 때마다 실행

  return (
    <div>
      <button onClick={() => setId(id + 1)}>다음 데이터</button>
      {data && <div>{JSON.stringify(data)}</div>}
    </div>
  );
}
```

## 결론

1. 렌더링이 완료된 후 useEffect가 실행된다.
2. 여러 useEffect는 작성된 순서대로 실행된다.
3. cleanup 함수는 다음 effect 실행 전 또는 언마운트 시 실행된다.
4. 의존성 배열에 따라 실행 시점이 결정된다.
