---
title: 스택(Stack)과 큐(Queue)에 대해 알아보기
date: 2024-12-05 07:40:00 +09:00
categories: [알고리즘, 개념]
tags: [JavaScript, 알고리즘, 스택, 큐]
---

스택과 큐는 데이터를 저장하고 처리하는 방법을 정의하는 가장 기본적인 자료구조이다. 둘 다 선형 자료구조지만, 데이터를 삽입하고 삭제하는 규칙이 다르다.

스택과 큐에 대한 정의와 차이점에 대해 자세히 알아보자.

---

## 스택(Stack) - LIFO(**Last In First Out)**

### 정의

스택은 **후입선출(LIFO, Last In First Out)** 원칙을 따르는 자료구조이다.

가장 나중에 들어온 데이터가 가장 먼저 나간다.

<img src='https://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Data_stack.svg/1920px-Data_stack.svg.png'>

### 주요 연산

| **연산**         | **설명**                                       | **JavaScript 구현**       |
| ---------------- | ---------------------------------------------- | ------------------------- |
| **`push(item)`** | 스택의 맨 위에 데이터를 추가                   | `stack.push(item)`        |
| **`pop()`**      | 스택의 맨 위 데이터를 제거하고 반환            | `stack.pop()`             |
| **`peek()`**     | 스택의 맨 위 데이터를 확인하지만 제거하지 않음 | `stack[stack.length - 1]` |
| **`isEmpty()`**  | 스택이 비어 있는지 확인                        | `stack.length === 0`      |

### 스택의 활용

1. **괄호 검사**: 문자열의 괄호 짝을 올바르게 확인.
2. **DFS(깊이 우선 탐색)**: 스택을 활용해 구현 가능.
3. **백트래킹**: 스택을 사용하여 경로를 저장하고 되돌아가는 작업.

### [예제] 스택 - 괄호 검사

```jsx
function isValidParentheses(s) {
  const stack = [];
  for (const char of s) {
    if (char === "(") {
      stack.push(char);
    } else {
      if (stack.length === 0) return false;
      stack.pop();
    }
  }
  return stack.length === 0;
}

// 예제 실행
console.log(isValidParentheses("()()")); // true
console.log(isValidParentheses("(())")); // true
console.log(isValidParentheses("(()")); // false
```

---

## 큐(Queue) - FIFO(First In First Out)

### 정의

큐는 **선입선출(FIFO, First In First Out)** 원칙을 따르는 자료구조이다.

가장 먼저 들어온 데이터가 가장 먼저 나간다.

<img src='https://upload.wikimedia.org/wikipedia/commons/thumb/5/52/Data_Queue.svg/2560px-Data_Queue.svg.png'>

### 주요 연산

| **연산**            | **설명**                                     | **JavaScript 구현**  |
| ------------------- | -------------------------------------------- | -------------------- |
| **`enqueue(item)`** | 큐의 맨 뒤에 데이터를 추가                   | `queue.push(item)`   |
| **`dequeue()`**     | 큐의 맨 앞 데이터를 제거하고 반환            | `queue.shift()`      |
| **`peek()`**        | 큐의 맨 앞 데이터를 확인하지만 제거하지 않음 | `queue[0]`           |
| **`isEmpty()`**     | 큐가 비어 있는지 확인                        | `queue.length === 0` |

### 큐의 활용

1. **BFS(너비 우선 탐색)**: 큐를 활용해 구현 가능.
2. **데이터의 순차 처리**: 예를 들어, 프린터 작업 큐.
3. **캐시(Cache)**: 페이지 교체 알고리즘(예: FIFO) 구현.

### [예제] 큐 - BFS

```jsx
function bfs(graph, start) {
  const visited = [];
  const queue = [start];

  while (queue.length > 0) {
    const node = queue.shift();
    if (!visited.includes(node)) {
      visited.push(node);
      queue.push(...graph[node]);
    }
  }
  return visited;
}

// 예제 실행
const graph = {
  1: [2, 3],
  2: [4, 5],
  3: [],
  4: [],
  5: []
};

console.log(bfs(graph, 1)); // [1, 2, 3, 4, 5]
```

---

## 스택과 큐의 비교

| **특징**        | **스택**                     | **큐**                       |
| --------------- | ---------------------------- | ---------------------------- |
| **데이터 처리** | 후입선출(LIFO)               | 선입선출(FIFO)               |
| **삽입 위치**   | 데이터의 끝(top)             | 데이터의 끝(rear)            |
| **삭제 위치**   | 데이터의 끝(top)             | 데이터의 앞(front)           |
| **응용 사례**   | **DFS**, 괄호 검사, 백트래킹 | **BFS**, 작업 스케줄링, 캐시 |

---

## 정리

- **스택**: 후입선출(LIFO) 원칙으로, **DFS**나 백트래킹 문제 해결에 적합하다.
- **큐**: 선입선출(FIFO) 원칙으로, **BFS**나 순차적인 데이터 처리 문제 해결에 적합하다.
