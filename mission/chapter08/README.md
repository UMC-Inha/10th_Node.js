# 8주차 미션 README

## 1. 미션 목표

8주차 미션의 목표는 기존에 구현한 API에 Swagger 문서화를 적용하는 것이다.

이번 주차에서는 Swagger, OpenAPI, TSOA(TypeScript-first OpenAPI)를 학습하고, 프론트엔드 개발자가 API를 쉽게 이해하고 사용할 수 있도록 API 명세를 제공하는 방법을 실습하였다.

미션에서는 기존 API에 대해 다음 내용을 확인하고 정리하였다.

- Swagger UI 설정
- OpenAPI 명세 파일 생성
- TSOA 기반 API 문서화 구조 이해
- 요청 Body, 응답 형식, 실패 응답 문서화 방식 확인
- 프론트엔드 협업 관점에서 API 문서화의 필요성 정리

---

## 2. 미션 핵심 개념

### 2-1. Swagger & OpenAPI

Swagger는 API 문서를 브라우저에서 확인하고 테스트할 수 있게 도와주는 도구이다.

OpenAPI는 REST API의 요청 방식, 파라미터, 응답 구조 등을 표준 형식으로 표현하기 위한 명세이다.

즉, OpenAPI는 API 문서를 작성하는 표준이고, Swagger UI는 그 명세를 보기 좋게 보여주는 도구라고 이해할 수 있다.

---

### 2-2. TSOA

TSOA는 TypeScript 코드 기반으로 OpenAPI 명세를 생성해주는 도구이다.

컨트롤러, DTO, 데코레이터를 기반으로 Swagger 문서를 생성할 수 있으며, TypeScript의 타입 정보를 활용하기 때문에 API 문서와 실제 코드의 불일치를 줄일 수 있다.

---

### 2-3. Type-Driven Documentation

Type-Driven Documentation은 타입 정보를 기반으로 문서를 만드는 방식이다.

예를 들어 회원가입 요청 DTO에 `email`, `name`, `preferences` 같은 필드가 정의되어 있으면, 이를 기반으로 Swagger 문서에서 요청 Body 구조를 확인할 수 있다.

이 방식은 문서를 따로 수동으로 작성하는 것보다 유지보수에 유리하다.

---

## 3. 구현 및 적용 내용

### 3-1. Swagger UI 패키지 설치

Swagger UI를 Express 서버에서 확인하기 위해 다음 패키지를 설치하였다.

```bash
npm install swagger-ui-express
npm install --save-dev @types/swagger-ui-express
```

추가로 서버 실행 중 필요한 패키지를 확인하여 다음 패키지도 설치하였다.

```bash
npm install morgan cookie-parser
npm install --save-dev @types/morgan @types/cookie-parser
```

TSOA 관련 실행을 위해 다음 패키지도 확인하였다.

```bash
npm install tsoa @tsoa/runtime
```

---

### 3-2. Swagger UI 연결

`src/index.ts`에서 TSOA가 생성한 Swagger 명세 파일을 읽어 `/docs` 경로에 Swagger UI를 연결하였다.

```ts
import swaggerUi from "swagger-ui-express";
import fs from "fs";
import path from "path";

const swaggerPath = path.resolve("dist/swagger.json");

if (fs.existsSync(swaggerPath)) {
  const swaggerFile = JSON.parse(fs.readFileSync(swaggerPath, "utf8"));
  app.use("/docs", swaggerUi.serve, swaggerUi.setup(swaggerFile));
}
```

위 설정을 통해 서버 실행 후 다음 주소에서 Swagger 문서를 확인할 수 있다.

```text
http://localhost:3000/docs
```

---

### 3-3. CORS 및 API 호출 확인

프론트엔드에서 API를 호출하는 상황을 확인하기 위해 `test.html`을 작성하고 Live Server로 실행하였다.

회원가입 API를 호출하여 성공 응답과 실패 응답을 확인하였다.

```js
await callAPI("/api/v1/users/signup", userData);
```

이를 통해 브라우저에서 백엔드 API를 직접 호출할 때 CORS 설정과 표준 응답 구조가 중요하다는 점을 확인하였다.

---

## 4. TSOA 문서화 적용 과정

TSOA에서는 일반적으로 다음과 같은 클래스 기반 컨트롤러 구조를 사용한다.

```ts
import {
  Body,
  Controller,
  Post,
  Route,
  Tags,
  Response as TsoaResponse,
} from "tsoa";

@Route("users")
@Tags("Users")
export class UserController extends Controller {
  /**
   * 회원가입 API
   *
   * 사용자의 기본 정보와 선호 카테고리를 입력받아 새로운 사용자를 생성한다.
   *
   * @summary 회원가입 API
   */
  @Post("signup")
  @TsoaResponse<ApiResponse<UserSignUpResponse>>(200, "회원가입 성공")
  @TsoaResponse<ApiResponse<null>>(400, "요청 값 검증 실패")
  @TsoaResponse<ApiResponse<null>>(409, "이미 존재하는 이메일")
  public async signUp(
    @Body() body: UserSignUpRequest
  ): Promise<ApiResponse<UserSignUpResponse>> {
    const result = await userSignUp(body);
    return success(result);
  }
}
```

하지만 현재 프로젝트의 기존 컨트롤러 구조는 다음과 같이 Express 핸들러 함수 기반으로 작성되어 있었다.

```ts
export const handleUserSignUp = async (req, res, next) => {
  // 회원가입 처리 로직
};
```

따라서 이번 실습에서는 기존 Express 라우터 구조를 유지하면서 Swagger UI 연결과 문서 확인을 중심으로 진행하였다.

---

## 5. 실행 및 확인

### 5-1. 서버 실행

```bash
npm run dev
```

서버가 정상 실행되면 다음과 같은 로그를 확인할 수 있다.

```text
[server]: Server is running at http://localhost:3000
```

---

### 5-2. Swagger UI 확인

브라우저에서 다음 주소로 접속하였다.

```text
http://localhost:3000/docs
```

Swagger UI를 통해 API 문서 화면을 확인하였다.

---

### 5-3. 회원가입 API 호출 확인

Live Server에서 `test.html`을 실행하고 회원가입 API를 호출하였다.

확인한 내용은 다음과 같다.

- 회원가입 성공 응답 확인
- 중복 이메일 실패 응답 확인
- CORS 설정 확인
- 표준 응답 구조 확인

---

## 6. 트러블슈팅

### 이슈 1. Prisma DATABASE_URL 형식 오류

#### 문제

Prisma 명령어 실행 중 다음 오류가 발생하였다.

```bash
Error: P1013
The provided database string is invalid.
datasource.url in prisma.config.ts is invalid:
must start with the protocol mysql://
```

#### 원인

DataGrip에서는 다음과 같은 JDBC URL을 사용한다.

```text
jdbc:mysql://localhost:3306/umc_week8
```

하지만 Prisma에서는 JDBC 형식이 아니라 다음과 같은 MySQL URL 형식을 사용해야 한다.

```env
DATABASE_URL="mysql://root:비밀번호@127.0.0.1:3306/umc_week8"
```

#### 해결

`.env`의 `DATABASE_URL`을 Prisma 형식에 맞게 수정하였다.

---

### 이슈 2. 빈 DB에서 prisma db pull 실행 시 P4001 발생

#### 문제

DB 연결을 수정한 뒤 `prisma db pull`을 실행했지만 다음 오류가 발생하였다.

```bash
P4001 The introspected database was empty
```

#### 원인

`prisma db pull`은 이미 DB에 존재하는 테이블을 Prisma schema로 가져오는 명령어이다.  
하지만 새로 만든 `umc_week8` DB에는 아직 테이블이 없었기 때문에 가져올 모델이 없어 오류가 발생하였다.

#### 해결

빈 DB에는 `db pull`이 아니라 migration을 적용해야 한다.

```bash
npx prisma migrate dev --schema=./prisma/schema.prisma
```

---

### 이슈 3. food_category 외래키 오류

#### 문제

회원가입 API 호출 시 다음과 같은 오류가 발생하였다.

```bash
Foreign key constraint violated on the fields: (`food_category_id`)
```

#### 원인

회원가입 요청 Body에는 다음 값이 포함되어 있었다.

```js
preferences: [1]
```

하지만 DB의 `food_category` 테이블에 `id = 1`인 데이터가 없어 `user_favor_category` 생성 시 외래키 제약 조건 오류가 발생하였다.

#### 해결

DataGrip에서 기본 음식 카테고리 데이터를 추가하였다.

```sql
INSERT INTO food_category (name)
VALUES
('한식'),
('중식'),
('일식'),
('양식'),
('치킨');
```

---

### 이슈 4. PR 충돌 발생

#### 문제

8주차 실습 PR을 생성하는 과정에서 다음 파일들에 conflict가 발생하였다.

```text
.gitignore
package-lock.json
package.json
src/db.config.ts
src/index.ts
src/modules/users/controllers/user.controller.ts
src/modules/users/dtos/user.dto.ts
src/modules/users/services/user.service.ts
tsconfig.json
```

#### 원인

PR 대상 브랜치인 `UMC-Inha:chanchan/main`과 내 작업 브랜치의 동일 파일 수정 사항이 충돌하였다.

#### 해결

로컬에서 원본 브랜치를 가져와 merge 후 충돌을 해결하였다.

```bash
git remote add upstream https://github.com/UMC-Inha/10th_Node.js_Practice_Mission.git
git fetch upstream
git merge upstream/chanchan/main
```

충돌 해결 후 서버 실행을 확인하고 커밋하였다.

```bash
npm run dev
git add .
git commit -m "fix: 8주차 PR 충돌 해결"
git push origin feature/chapter-08
```

---

### 이슈 5. 잘못된 PR 브랜치 선택

#### 문제

GitHub에서 PR을 생성했을 때 다음과 같이 잘못된 브랜치로 PR이 열렸다.

```text
from inhadissolve:main
```

#### 원인

실제 제출해야 하는 브랜치는 `feature/chapter-08`인데, 실수로 `main` 브랜치에서 PR을 열었다.

#### 해결

잘못 열린 PR은 닫고, 새 PR을 다음 기준으로 생성하였다.

```text
base repository: UMC-Inha/10th_Node.js_Practice_Mission
base branch: chanchan/main

head repository: inhadissolve/10th_Node.js_Practice_Mission
compare branch: feature/chapter-08
```

---

## 7. 미션 정리

이번 미션을 통해 API 구현만큼 API 문서화도 중요하다는 것을 알 수 있었다.

프론트엔드 개발자는 Swagger 문서를 통해 API 주소, 요청 Body, 응답 구조, 실패 케이스를 확인할 수 있다. 따라서 Swagger와 OpenAPI는 백엔드와 프론트엔드 간 협업에서 중요한 역할을 한다.

또한 TSOA는 TypeScript 타입과 컨트롤러 정보를 기반으로 OpenAPI 명세를 생성할 수 있어 문서와 코드의 불일치를 줄일 수 있다. 다만 프로젝트 구조가 Express 핸들러 함수 기반인지, TSOA 클래스 컨트롤러 기반인지에 따라 적용 방식이 달라질 수 있다는 점도 확인하였다.

이번 실습에서는 기존 Express 구조를 유지하면서 Swagger UI 연결과 API 호출 흐름을 확인하였고, 이후 TSOA를 더 안정적으로 적용하기 위해서는 컨트롤러를 클래스 기반 구조로 정리하는 과정이 필요하다고 느꼈다.