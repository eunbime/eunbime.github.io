---
title: RESTful API 이해하기
date: 2025-01-31 10:00:00 +09:00
categories: [프론트엔드, CS]
tags: [REST]
---

## REST API란?

REST(Representational State Transfer)는 웹 서비스의 아키텍처 스타일로, 다음과 같은 특징을 가집니다.

- 자원(Resource) 중심의 설계
- 상태를 가지지 않음(Stateless)
- HTTP 프로토콜 기반
- URI를 통한 자원 식별

## REST 6가지 원칙

### 1. Client-Server (클라이언트-서버 구조)

```
클라이언트와 서버의 관심사를 분리하여 독립적 진화 가능

```

### 2. Stateless (무상태)

```
각 요청은 이전 요청과 독립적으로 처리되어야 함

```

### 3. Cacheable (캐시 가능)

```
응답은 캐시 가능 여부를 명시해야 함

```

### 4. Uniform Interface (인터페이스 일관성)

```
URI로 지정한 리소스에 대한 조작을 통일되고 한정적인 인터페이스로 수행

```

### 5. Layered System (계층화)

```
중간 계층을 둘 수 있으며, 클라이언트는 서버와 직접 통신하는지 알 필요가 없음

```

### 6. Code on Demand (선택사항)

```
서버가 클라이언트에 실행 가능한 코드를 전송하여 기능을 확장할 수 있음

```

## RESTful API란?

RESTful API는 REST 아키텍처의 제약조건을 준수하며 REST API를 제공하는 웹 서비스를 의미합니다.

### HTTP 메서드의 올바른 사용

```tsx
// RESTful API 예시
GET    /users           // 사용자 목록 조회
GET    /users/:id       // 특정 사용자 조회
POST   /users           // 새 사용자 생성
PUT    /users/:id       // 사용자 정보 전체 수정
PATCH  /users/:id       // 사용자 정보 일부 수정
DELETE /users/:id       // 사용자 삭제

```

## REST API vs RESTful API

### REST API

```tsx
// REST API 예시 (덜 RESTful한 방식)
GET    /getAllUsers
POST   /createUser
PUT    /updateUser
DELETE /deleteUser/:id

```

### RESTful API

```tsx
// RESTful API 예시 (리소스 중심)
GET    /users
POST   /users
PUT    /users/:id
DELETE /users/:id

```

## 실제 구현 예시

### Express를 사용한 RESTful API 구현

```tsx
import express from "express";
const router = express.Router();

// 사용자 관련 엔드포인트
router.get("/users", async (req, res) => {
  try {
    const users = await User.find();
    res.status(200).json(users);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});

router.get("/users/:id", async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: "User not found" });
    }
    res.status(200).json(user);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});

router.post("/users", async (req, res) => {
  try {
    const newUser = await User.create(req.body);
    res.status(201).json(newUser);
  } catch (error) {
    res.status(400).json({ error: "Invalid request" });
  }
});
```

## 모범 사례

### 1. 적절한 HTTP 상태 코드 사용

```tsx
// 상태 코드 사용 예시
200: OK
201: Created
204: No Content
400: Bad Request
401: Unauthorized
403: Forbidden
404: Not Found
500: Internal Server Error

```

### 2. 버전 관리

```
/api/v1/users
/api/v2/users
```

### 3. 페이지네이션 구현

```tsx
// 쿼리 파라미터를 통한 페이지네이션
GET /api/users?page=1&limit=10

// 응답 예시
{
  "data": [...],
  "pagination": {
    "currentPage": 1,
    "totalPages": 5,
    "totalItems": 48,
    "itemsPerPage": 10
  }
}

```

### 4. 에러 처리

```tsx
// 에러 응답 형식
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "사용자를 찾을 수 없습니다.",
    "details": {
      "userId": "123"
    }
  }
}

```

### 5. HATEOAS 구현

```tsx
// HATEOAS 응답 예시
{
  "id": 1,
  "name": "John Doe",
  "_links": {
    "self": { "href": "/api/users/1" },
    "profile": { "href": "/api/users/1/profile" },
    "orders": { "href": "/api/users/1/orders" }
  }
}

```

## 실제 프로젝트 구조 예시

```
src/
├── controllers/
│   ├── userController.ts
│   └── postController.ts
├── routes/
│   ├── userRoutes.ts
│   └── postRoutes.ts
├── models/
│   ├── User.ts
│   └── Post.ts
├── middleware/
│   ├── auth.ts
│   └── errorHandler.ts
└── app.ts

```

## API 문서화 예시 (Swagger)

```yaml
/users:
  get:
    summary: 사용자 목록 조회
    parameters:
      - name: page
        in: query
        type: integer
        description: 페이지 번호
    responses:
      200:
        description: 성공
        schema:
          type: array
          items:
            $ref: "#/definitions/User"
```

## 보안 고려사항

### 1. 인증과 인가

```tsx
// JWT를 사용한 인증 예시
app.use("/api", authenticate);

function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) {
    return res.status(401).json({ error: "Authentication required" });
  }
  // JWT 검증 로직
}
```

### 2. 속도 제한

```tsx
import rateLimit from "express-rate-limit";

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15분
  max: 100 // IP당 최대 요청 수
});

app.use("/api", limiter);
```

## 마무리

RESTful API를 설계할 때는 다음 사항을 고려해야 합니다.

1. 리소스 중심의 URL 설계
2. 적절한 HTTP 메서드 사용
3. 상태 코드의 올바른 사용
4. 에러 처리
5. 보안
6. 문서화
