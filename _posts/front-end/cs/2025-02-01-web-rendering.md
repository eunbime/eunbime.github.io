---
title: 웹 브라우저의 렌더링 방식
date: 2025-02-01 10:00:00 +09:00
categories: [프론트엔드, CS]
tags: [CS]
---

웹 브라우저는 HTML, CSS, JavaScript 등의 리소스를 받아서 화면에 표시하는 과정을 거친다.

이 과정은 **HTML 파싱 → CSS 파싱 → 렌더 트리 생성 → 레이아웃 계산 → 페인팅** 순으로 진행된다.

---

## **1. 렌더링 과정 개요**

브라우저는 웹 페이지를 렌더링할 때 **다섯 단계**를 거친다.

1. **HTML 파싱** → **DOM 트리 생성**
2. **CSS 파싱** → **CSSOM 트리 생성**
3. **렌더 트리(Render Tree) 생성**
4. **레이아웃(Layout) 계산**
5. **페인팅(Painting) 및 합성(Compositing)**

---

## **2. 브라우저 렌더링 과정 상세 설명**

### ✅ **1) HTML 파싱 → DOM 생성**

- HTML 문서를 해석하여 **DOM(Document Object Model) 트리**를 만든다.
- 브라우저는 HTML을 위에서 아래로 한 줄씩 읽으며 **태그를 노드(Node)로 변환**한다.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>브라우저 렌더링</title>
  </head>
  <body>
    <h1>Hello World</h1>
    <p>브라우저 렌더링 방식</p>
  </body>
</html>
```

👉 **위 HTML을 브라우저가 해석하면 다음과 같은 DOM 트리가 생성됨**

```
Document
 ├── html
 │    ├── head
 │    │    └── title
 │    ├── body
 │         ├── h1
 │         ├── p

```

✔ **DOM 트리는 HTML 요소들의 계층 구조를 표현한 트리 구조이다.**

---

### ✅ **2) CSS 파싱 → CSSOM 생성**

- CSS 파일을 다운로드하고, **CSS 규칙을 해석하여 CSSOM(CSS Object Model) 트리**를 만든다.
- CSS는 **선택자 우선순위에 따라 해석**된다.

```css
h1 {
  color: blue;
  font-size: 24px;
}

p {
  color: gray;
  font-size: 16px;
}
```

👉 **위 CSS를 브라우저가 해석하면 다음과 같은 CSSOM 트리가 생성됨**

```
CSSOM
 ├── h1 { color: blue; font-size: 24px; }
 ├── p { color: gray; font-size: 16px; }

```

✔ **CSSOM 트리는 CSS 스타일을 정의하는 트리 구조이다.**

---

### ✅ **3) 렌더 트리(Render Tree) 생성**

- 렌더 트리는 **DOM과 CSSOM을 결합한 트리**이다.
- 화면에 표시될 요소들만 포함되며, `display: none` 같은 요소는 렌더 트리에 포함되지 않는다.

👉 **렌더 트리 생성 과정**

1. DOM 트리와 CSSOM 트리를 결합하여 화면에 표시할 요소만 남긴다.
2. 각 요소에 적용될 스타일 정보를 매칭한다.

예를 들어, 위에서 만든 DOM과 CSSOM을 기반으로 한 렌더 트리는 다음과 같다.

```
Render Tree
 ├── h1 { color: blue; font-size: 24px; }
 ├── p { color: gray; font-size: 16px; }

```

✔ **렌더 트리는 실제 화면에 그려질 요소들만 포함한다.**

---

### ✅ **4) 레이아웃(Layout) 계산**

- 각 요소의 **크기(width, height), 위치(x, y 좌표)** 를 계산한다.
- 부모 요소의 크기나 스타일에 따라 자식 요소의 크기가 결정된다.

예를 들어, `div` 내부에 있는 `h1`과 `p`의 위치를 계산하는 과정이다.

```
Layout 계산
 ├── h1 { x: 10px, y: 20px, width: 200px, height: 40px }
 ├── p { x: 10px, y: 70px, width: 300px, height: 30px }

```

✔ **이 과정에서 요소의 크기와 위치가 결정된다.**

---

### ✅ **5) 페인팅(Painting) 및 합성(Compositing)**

- **페인팅(Painting)**: 계산된 레이아웃을 실제 픽셀로 변환하여 화면에 그림을 그린다.
- **합성(Compositing)**: 여러 개의 레이어로 나누어 최적화된 방식으로 화면을 업데이트한다.

📌 **페인팅 최적화 기법**

- **레이어 분리**: `will-change`, `transform: translateZ(0)` 등을 사용하여 특정 요소를 GPU에서 직접 렌더링하도록 유도.
- **CSS 애니메이션 최적화**: `transform`과 `opacity`를 사용하면 레이아웃 재계산(Layout) 없이 애니메이션을 실행할 수 있다.

✔ **페인팅 과정이 최적화되지 않으면 화면 깜빡임(Flickering)이나 성능 저하가 발생할 수 있다.**

---

## **3. 렌더링 최적화 방법**

브라우저의 렌더링 과정을 최적화하려면 **불필요한 연산을 최소화**하는 것이 중요하다.

| 최적화 기법               | 설명                                          |
| ------------------------- | --------------------------------------------- |
| **레이아웃 최소화**       | `position: absolute` 사용, 크기 변경 최소화   |
| **CSS 애니메이션 최적화** | `transform`과 `opacity`만 변경                |
| **이미지 최적화**         | `WebP`, `lazy loading` 활용                   |
| **리플로우 최소화**       | DOM 조작 최소화, `requestAnimationFrame` 사용 |
| **코드 스플리팅**         | 필요한 CSS, JS만 로드                         |
| **GPU 가속**              | `will-change`, `translateZ(0)` 사용           |

---

## **4. 결론**

- 브라우저는 HTML, CSS를 파싱하여 **렌더 트리를 만들고, 레이아웃을 계산한 후 페인팅**하는 과정을 거친다.
- `DOM`, `CSSOM`, `Render Tree`, `Layout`, `Painting` 단계를 이해하면 **렌더링 성능 최적화**가 가능하다.
- `리플로우(Reflow)`, `리페인트(Repaint)`를 최소화하면 **더 빠른 렌더링이 가능**하다.

🚀 **브라우저 렌더링을 이해하면 최적화된 웹 애플리케이션을 만들 수 있다!**
