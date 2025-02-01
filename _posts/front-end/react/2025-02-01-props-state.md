---
title: Props와 State의 정의와 차이점
date: 2025-02-01 10:00:00 +09:00
categories: [프론트엔드, React]
tags: [React]
---

# **Props와 State의 정의와 차이점**

React에서 **`props`**(속성)와 **`state`**(상태)는 **컴포넌트의 데이터를 관리하는 두 가지 주요 방법**이다.

각각의 역할과 차이를 명확히 이해하면 **컴포넌트 간의 데이터 흐름을 효과적으로 설계할 수 있다.**

---

## **1. Props란?**

- Props (Properties)는 부모 컴포넌트에서 자식 컴포넌트로 전달하는 **읽기 전용 데이터**이다.
- **부모 → 자식 방향으로만 전달됨 (단방향 데이터 흐름)**
- **자식 컴포넌트에서 수정 불가능 (불변성)**
- **컴포넌트의 재사용성을 높이는 역할**
- **함수형 컴포넌트와 클래스형 컴포넌트에서 모두 사용 가능**

### **✅ 예제: Props 사용법**

```tsx
const Greeting = ({ name }: { name: string }) => {
  return <h1>Hello, {name}!</h1>;
};

// 부모 컴포넌트에서 자식 컴포넌트에 props 전달
const App = () => {
  return <Greeting name="Eunbi" />;
};
```

✔️ `App` 컴포넌트가 `Greeting` 컴포넌트에 `name="Eunbi"`라는 **props를 전달**하고 있음.

✔️ `Greeting` 컴포넌트는 **`name`을 직접 수정할 수 없음** (읽기 전용).

---

## **2. State란?**

**📌 State**는 **컴포넌트 내부에서 관리되는 동적인 데이터**이다.

- **컴포넌트 내부에서 선언되고 관리됨**
- **컴포넌트의 상태를 저장하고 업데이트할 수 있음**
- **사용자 입력, 네트워크 요청 등의 이벤트에 따라 변경 가능**
- **state가 변경되면 해당 컴포넌트가 다시 렌더링됨**

### **✅ 예제: State 사용법**

```tsx
import { useState } from "react";

const Counter = () => {
  const [count, setCount] = useState(0); // state 선언

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increase</button>
    </div>
  );
};
```

✔️ `useState(0)`을 사용해 **초기 state 값을 `0`으로 설정**.

✔️ `setCount(count + 1)`을 호출하면 **state가 변경되고 컴포넌트가 다시 렌더링됨**.

---

## **3. Props vs State 비교**

| 구분                        | **Props**                                | **State**                                |
| --------------------------- | ---------------------------------------- | ---------------------------------------- |
| **데이터 흐름**             | 부모 → 자식 (단방향)                     | 컴포넌트 내부에서 관리                   |
| **수정 가능 여부**          | **불변 (읽기 전용)**                     | **변경 가능 (setState, useState)**       |
| **컴포넌트 내부 선언 여부** | ❌ (부모에서 전달됨)                     | ✅ (컴포넌트 내부에서 선언)              |
| **재사용성**                | ✅ (컴포넌트 재사용 증가)                | ❌ (특정 컴포넌트에 종속됨)              |
| **렌더링 영향**             | props 변경 시 **자식 컴포넌트 재렌더링** | state 변경 시 **해당 컴포넌트 재렌더링** |

---

## **4. 언제 Props를 사용하고, 언제 State를 사용할까?**

✅ **Props를 사용해야 하는 경우**

- 부모 컴포넌트에서 **자식에게 데이터를 전달**할 때
- **컴포넌트가 동일한 데이터로 렌더링**될 때
- UI의 상태와 관계없는 **정적인 값**을 다룰 때

✅ **State를 사용해야 하는 경우**

- **컴포넌트 내부에서 변경되는 데이터**를 관리할 때
- **사용자 입력, 네트워크 요청 결과** 등을 저장할 때
- **시간에 따라 변하는 값 (예: 카운터, 토글 버튼 상태 등)**

---

## **5. Props와 State를 함께 사용하는 예제**

```tsx
import { useState } from "react";

// 자식 컴포넌트 (props 사용)
const Child = ({ count }: { count: number }) => {
  return <h1>Count: {count}</h1>;
};

// 부모 컴포넌트 (state 사용)
const Parent = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <Child count={count} /> {/* state를 props로 전달 */}
      <button onClick={() => setCount(count + 1)}>Increase</button>
    </div>
  );
};
```

✔️ `Parent` 컴포넌트에서 **state(`count`)를 관리**하고 있음.

✔️ `Child` 컴포넌트는 `count`를 **props로 전달받아 표시**할 뿐, 값을 변경하지 않음.

---

## **6. 결론**

✔️ **Props는 부모 → 자식 간 데이터 전달용이며, 읽기 전용이다.**

✔️ **State는 컴포넌트 내부에서 관리되는 동적인 데이터로, 변경 가능하다.**

✔️ **props와 state를 함께 활용하여 효율적인 데이터 관리를 해야 한다.** 🚀
