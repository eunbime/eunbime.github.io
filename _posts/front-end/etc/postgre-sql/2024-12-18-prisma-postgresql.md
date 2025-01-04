---
title: Prisma와 PostgreSQL을 활용한 ORM 환경 구축 - Docker와 Railway를 이용한 배포 가이드
date: 2024-12-18 10:00:00 +09:00
categories: [프론트엔드, PostgreSQL]
tags: [PostgreSQL, Prisma, Docker, Railway]
---

## 1. Prisma와 PostgreSQL

**Prisma**는 현대적인 ORM으로, 데이터베이스와 애플리케이션 간의 상호작용을 쉽게 만들어주는 도구이다. 일반적인 ORM과 달리 **타입 안전성**과 **개발자 경험**에 초점을 맞추어, 더 효율적이고 오류를 줄이는 코드를 작성할 수 있게 돕는다.

- ORM(Object-Relational Mapping): **객체 지향 프로그래밍 언어**를 사용하여 **데이터베이스의 데이터를 처리**할 수 있도록 돕는 기술이다. 즉, 데이터베이스의 테이블과 프로그래밍 언어의 객체 간에 **자동 변환**을 제공한다. SQL에 대한 추상화를 제공하여 객체 지향 프로그래밍 환경에서 데이터베이스 작업을 더 쉽고 간결하게 만들어준다.

**PostgreSQL**은 강력한 오픈 소스 관계형 데이터베이스로, 안정성과 성능에서 높은 평가를 받는다.

두 기술은 함께 사용하기에 이상적이며, Docker와 Railway를 활용하면 로컬 개발 환경뿐만 아니라 클라우드 배포도 쉽게 설정할 수 있다.

<br />

---

## 2. 로컬 환경에서 Prisma와 PostgreSQL 설정

### 2.1 Docker 설정

먼저, Docker를 이용해 PostgreSQL 컨테이너를 실행한다.

**`docker-compose.yml`** 파일을 생성하고 다음 내용을 추가한다:

```yaml
version: "3.8"

services:
  postgres:
    image: postgres:15
    container_name: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

1. **`POSTGRES_USER`**: PostgreSQL의 기본 사용자 이름.
2. **`POSTGRES_PASSWORD`**: 데이터베이스 접근 비밀번호.
3. **`POSTGRES_DB`**: 생성될 기본 데이터베이스 이름.

이후, 터미널에서 다음 명령어로 컨테이너를 실행한다.

```bash
docker-compose up -d
```

### 2.2 Prisma 설치 및 초기화

다음 명령어로 Prisma를 설치하고 초기화하여 프로젝트에 추가한다.

```bash
npm install prisma --save-dev
npx prisma init
```

위 명령어를 실행하면 프로젝트에 **`prisma/schema.prisma`** 파일이 생성된다. 이 파일에서 데이터베이스 연결 설정을 수정한다:

```
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

**`DATABASE_URL`** 환경 변수를 설정하기 위해 프로젝트 루트의 `.env` 파일을 수정한다.

- `docker-compose.yml` 파일에서 설정했던 POSTGRES_USER: postgres, POSTGRES_PASSWORD: password, POSTGRES_DB: mydb 정보를 사용한다.

```

DATABASE_URL="postgresql://postgres:password@localhost:5432/mydb"
```

### 2.3 Prisma 데이터 모델 작성

예시로 간단한 사용자(User) 모델을 정의한다:

```

model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  createdAt DateTime @default(now())
}
```

모델을 정의한 후 다음 명령어로 데이터베이스와 동기화한다.

```bash
npx prisma migrate dev --name init
npm prisma db push
```

또한 prisma studio를 활용하여 설정한 데이터베이스를 빠르게 확인할 수 있다.

```jsx
npx prisma studio
```

<br />

---

## 3. Railway를 이용한 배포

빠르고 간편한 배포를 위해 클라우드 서비스인 Railway를 사용한다.

### 3.1 Railway에서 PostgreSQL 배포

1. [Railway](https://railway.app/)에 접속해 프로젝트를 생성한다.
2. 데이터베이스 플러그인 중 PostgreSQL을 추가하면 Railway에서 제공하는 클라우드 기반 PostgreSQL URL이 생성된다.

### 3.2 Prisma에 연결

Railway의 데이터베이스 URL을 `.env` 파일에 업데이트한다

- Railway에서 제공하는 URL은 데이터베이스 사용자 이름, 비밀번호, 호스트 및 포트를 포함한다

```

DATABASE_URL="postgresql://<user>:<password>@<host>:<port>/<database>"
```

<br />

---

## 4. 정리

Prisma와 PostgreSQL을 활용하면 효율적이고 직관적인 데이터베이스 연동이 가능하다. Docker를 통해 로컬 환경을 손쉽게 구축할 수 있으며, Railway를 이용해 클라우드 배포까지 구현할 수 있다.

이 과정은 초기 설정에 시간을 절약하고, 타입 안전성과 생산성을 제공하여 개발 경험을 향상시켜준다.
