---
title: 스택(Stack)과 큐(Queue) - 올바른 괄호/주식가격/다리를 지나는 트럭/프로세스
date: 2024-12-08 07:40:00 +09:00
categories: [알고리즘, 풀이]
tags: [JavaScript, 알고리즘, 스택, 큐, 프로그래머스]
---

# 스택 (LIFO, pop & push)

## 1. 올바른 괄호

**스택(Stack)** 자료구조를 사용하여 괄호 문자열이 올바른지 판별하는 문제

### 풀이 전략

1. 괄호를 저장하기 위해 스택 사용
2. 문자열 순회
   - 여는 괄호인 경우 스택에 추가
   - 닫는 괄호인 경우
     - 스택이 비어 있다면, false 반환
     - 스택에 괄호가 있다면 가장 최근에 추가된 여는 괄호를 제거(pop)하여 짝을 맞춤
3. 최종 스택 상태 확인하여 값 반환

### 알고리즘 풀이

```jsx
function solution(s) {
  // 1. 스택 초기화
  const stack = [];

  // 2. 문자열 순회
  for (let char of s) {
    // 여는 괄호인 경우 스택에 push
    if (char === "(") {
      stack.push(char);

      // 닫는 괄호인 경우
    } else {
      // 스택이 비어 있다면, false 반환
      if (stack.length === 0) {
        return false;
      }
      // 스택에 괄호가 있다면 최근 여는 괄호 pop
      stack.pop();
    }
  }

  // 3. 최종 스택 상태 확인하여 값 반환
  return stack.length === 0;
}
```

<br/>

## 2. 주식가격

스택을 사용하여 가격이 떨어지는 시점을 효율적으로 계산하는 문제

### 풀이 전략

1. 주식 가격을 확인하면서 **현재 가격이 이전 가격보다 작다면 스택에 저장된 인덱스를 이용해 기간을 계산**한다.
2. 가격이 떨어지는 시점을 빠르게 계산하며, 모든 주식 가격에 대해 마지막까지 남아 있는 스택을 처리한다.

### 알고리즘 풀이

```jsx
function solution(prices) {
  const result = Array(n).fill(0); // 결과 배열
  const stack = []; // 가격의 인덱스를 저장할 스택
  const n = prices.length;

  for (let i = 0; i < n; i++) {
    // 현재 가격이 스택의 마지막 가격보다 낮다면
    while (stack.length > 0 && prices[stack[stack.length - 1]] > prices[i]) {
      const top = stack.pop(); // 스택에서 인덱스를 꺼낸다
      result[top] = i - top; // 현재 인덱스와의 차이를 결과로 저장
    }
    stack.push(i); // 현재 인덱스를 스택에 추가
  }

  // 스택에 남아있는 인덱스 처리 (끝까지 가격이 떨어지지 않은 경우)
  while (stack.length > 0) {
    const top = stack.pop();
    result[top] = n - top - 1; // 끝까지 유지된 기간 계산
  }

  return result;
}
```

<br />

---

# 큐(FIFO, shift & push)

## 1. 다리를 지나는 트럭

**큐(Queue)** 구조를 활용하여 다리 위를 건너는 트럭의 상태를 시뮬레이션하며 해결하는 문제

### 풀이 전략

1. 다리 상태를 큐로 표현
   - 다리는 `bridge_length` 만큼의 길이를 가지므로, 다리 위에 있는 트럭들을 큐에 저장한다.
   - 트럭이 다리에 올라오면 `bridge_length` 동안 큐에 머문다.
2. 다리 위 무게 관리
   - 다리 위의 트럭 총 무게를 추적하여, 추가적인 트럭이 올라올 수 있는지 판단한다.
3. 시뮬레이션 진행
   - 다리 위의 트럭들을 한 칸씩 전진
   - 다리를 지나간 트럭은 큐에서 제거
   - 다리의 현재 무게를 확인하여 다음 트럭이 올라갈 수 있으면 큐에 추가
4. 모든 트럭이 다리를 건넌 뒤 종료

### 알고리즘 풀이

```jsx
function solution(bridge_length, weight, truck_weights) {
  let time = 0; // 경과 시간
  let bridge = Array(bridge_length).fill(0); // 다리를 큐로 표현
  let currentWeight = 0; // 다리 위의 현재 무게

  while (truck_weights.length > 0 || currentWeight > 0) {
    time++;

    // 다리의 첫 번째 칸에서 트럭 제거
    currentWeight -= bridge.shift();

    // 다음 트럭을 다리에 올릴 수 있는지 확인
    if (
      truck_weights.length > 0 &&
      currentWeight + truck_weights[0] <= weight
    ) {
      const truck = truck_weights.shift();
      bridge.push(truck); // 트럭이 다리 위에 올라감
      currentWeight += truck;
    } else {
      // 다리가 꽉 차 있으면 0을 추가 (빈 칸 유지)
      bridge.push(0);
    }
  }

  return time;
}
```

<br/>

## 2. 프로세스

**큐(Queue)를 활용**하여 프로세스 우선순위에 따라 프로세스를 처리하고, 특정 프로세스의 실행 순서를 찾는 문제다. 큐와 우선순위를 고려한 시뮬레이션으로 해결하는 문제

### 풀이 전략

1. 큐 구조로 프로세스 관리
   - 각 프로세스의 중요도와 인덱스를 함께 큐에 저장
   - 배열 메서드 shift와 push를 활용하여 큐 동작 구현
2. 우선순위 확인
   - 큐의 맨 앞 프로세스를 꺼내고 남아있는 큐와 우선순위 비교(shift)
   - 높은 우선순위가 있다면 다시 큐의 맨뒤로 보냄(push)
3. 프로세스 실행
   - 높은 우선순위가 없다면 해당 프로세스를 실행
   - 실행된 프로세스가 목표 프로세스라면 현재 실행 순서 반환

### 알고리즘 풀이

```jsx
function solution(priorities, location) {
  // 1. 큐 구조로 프로세스 관리
  let queue = priorities.map((priority, index) => ({ priority, index })); //  // 큐에 인덱스와 중요도 저장
  let count = 0; // 실행 순서

  // 2. 우선순위 확인
  while (queue.length > 0) {
    // 큐에서 첫번째 프로세스 꺼냄: shift()
    let current = queue.shift();

    // 우선순위가 더 높은 프로세스가 있으면
    if (queue.some((process) => process.priority > current.priority)) {
      queue.push(current); // 큐의 맨뒤로 보냄(push)

      // 높은 우선 순위가 없다면
    } else {
      count++; // 현재 프로세스 실행

      // 실행된 프로세스가 목표 프로세스일 경우 실행 순서 반환
      if (current.index === location) {
        return count;
      }
    }
  }

  return count;
}
```
