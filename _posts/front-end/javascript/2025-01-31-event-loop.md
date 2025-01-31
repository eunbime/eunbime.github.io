---
title: 자바스크립트의 이벤트 루프
date: 2025-01-31 10:00:00 +09:00
categories: [프론트엔드, JavaScript]
tags: [JavaScript]
---

# 자바스크립트 이벤트 루프(Event Loop)

자바스크립트는 **싱글 스레드(Single Thread)** 기반의 언어이지만, **이벤트 루프(Event Loop)**를 통해 비동기 작업을 처리하며 동시성을 지원한다.

이번 글에서는 **이벤트 루프의 개념과 동작 방식**을 정리한다.

---

## 1. 이벤트 루프란?

- 이벤트 루프(Event Loop)는 **자바스크립트에서 비동기 작업을 관리하는 핵심 메커니즘**이다.

자바스크립트는 싱글 스레드로 동작하기 때문에, **동기적인 코드 실행만으로는 긴 시간이 걸리는 작업(예: 네트워크 요청, 파일 읽기) 동안 프로그램이 멈추게 된다.**

이를 해결하기 위해 **이벤트 루프가 콜 스택(Call Stack)과 태스크 큐(Task Queue)를 조정하여 비동기 코드를 효율적으로 실행한다.**

---

## 2. 이벤트 루프의 동작 방식

이벤트 루프는 **콜 스택(Call Stack), 웹 API(Web API), 태스크 큐(Task Queue), 마이크로태스크 큐(Microtask Queue)** 등의 개념을 기반으로 동작한다.

### 2-1. 콜 스택(Call Stack)

콜 스택은 **현재 실행 중인 함수들을 저장하는 스택(Stack) 자료구조**이다.

함수가 호출되면 **콜 스택에 추가(push)** 되고, 실행이 완료되면 **콜 스택에서 제거(pop)** 된다.

```
function first() {
  console.log("First");
}

function second() {
  console.log("Second");
}

first();
second();

```

**실행 순서:**

1. `first()` 실행 → 콜 스택에 추가
2. `console.log("First")` 실행 → 콜 스택에서 제거
3. `second()` 실행 → 콜 스택에 추가
4. `console.log("Second")` 실행 → 콜 스택에서 제거

**출력 결과:**

```
First
Second

```

---

### 2-2. 웹 API(Web API)

비동기 함수(예: `setTimeout`, `fetch`)는 콜 스택에서 실행되면 **웹 API로 전달**되어 처리된다.

```
console.log("Start");

setTimeout(() => {
  console.log("Timeout");
}, 1000);

console.log("End");

```

**실행 순서:**

1. `console.log("Start")` 실행 → 출력 후 콜 스택에서 제거
2. `setTimeout()` 실행 → **웹 API에서 타이머 시작 후 콜 스택에서 제거**
3. `console.log("End")` 실행 → 출력 후 콜 스택에서 제거
4. **1초 후**, `setTimeout()`의 콜백 함수가 **태스크 큐(Task Queue)로 이동**
5. 이벤트 루프가 **콜 스택이 비어 있음을 확인한 후, 태스크 큐에서 콜백을 실행**

**출력 결과:**

```
Start
End
Timeout

```

---

### 2-3. 태스크 큐(Task Queue)

태스크 큐(또는 콜백 큐)는 **비동기 작업이 완료된 후 실행될 콜백 함수들이 대기하는 곳**이다.

이벤트 루프는 **콜 스택이 비었을 때** 태스크 큐에서 대기 중인 함수를 실행한다.

**예제:**

```
console.log("Start");

setTimeout(() => console.log("Task Queue"), 0);

console.log("End");

```

**출력 결과:**

```
Start
End
Task Queue

```

`setTimeout`의 지연 시간이 `0ms`여도 태스크 큐에 들어가기 때문에 **콜 스택이 비어야 실행될 수 있다.**

---

### 2-4. 마이크로태스크 큐(Microtask Queue)

마이크로태스크 큐는 **태스크 큐보다 우선적으로 실행되는 비동기 작업 큐**이다.

주로 **`Promise.then()`, `MutationObserver`, `queueMicrotask()`** 등이 포함된다.

```
console.log("Start");

setTimeout(() => console.log("Task Queue"), 0);

Promise.resolve().then(() => console.log("Microtask"));

console.log("End");

```

**출력 결과:**

```
Start
End
Microtask
Task Queue

```

**실행 순서:**

1. `console.log("Start")` 실행 → 출력 후 콜 스택에서 제거
2. `setTimeout()` 실행 → 웹 API에서 타이머 시작 후 콜 스택에서 제거
3. `Promise.resolve().then()` 실행 → **마이크로태스크 큐에 추가**
4. `console.log("End")` 실행 → 출력 후 콜 스택에서 제거
5. 이벤트 루프가 **마이크로태스크 우선 실행** → `"Microtask"` 출력
6. **콜 스택이 비어있음을 확인한 후, 태스크 큐 실행** → `"Task Queue"` 출력

마이크로태스크는 태스크 큐보다 먼저 실행되므로 `"Microtask"`가 `"Task Queue"`보다 먼저 출력된다.

---

## 3. 이벤트 루프 실행 과정 요약

1. **콜 스택이 비어있으면** → **마이크로태스크 큐 실행**
2. **마이크로태스크 큐가 비어있으면** → **태스크 큐 실행**
3. 위 과정을 반복하여 **비동기 작업을 실행**

---

## 4. 이벤트 루프의 실행 순서 실습

다음 코드를 실행해보자.

```
console.log("Start");

setTimeout(() => console.log("setTimeout"), 0);

Promise.resolve().then(() => console.log("Promise"));

console.log("End");

```

**출력 결과:**

```
Start
End
Promise
setTimeout

```

**실행 과정:**

1. `console.log("Start")` → `"Start"` 출력
2. `setTimeout()` → **웹 API에서 처리 후 태스크 큐로 이동**
3. `Promise.resolve().then()` → **마이크로태스크 큐에 추가**
4. `console.log("End")` → `"End"` 출력
5. **마이크로태스크 큐 실행** → `"Promise"` 출력
6. **태스크 큐 실행** → `"setTimeout"` 출력

---

## 5. 결론

- **이벤트 루프는 자바스크립트의 동시성을 지원하는 핵심 메커니즘**이다.
- **콜 스택이 비어있을 때 태스크 큐에서 대기 중인 콜백 함수를 실행**한다.
- **마이크로태스크 큐가 태스크 큐보다 우선 실행**된다.
- **비동기 처리를 이해하면 코드 실행 순서를 예측하고 성능을 최적화할 수 있다.**
