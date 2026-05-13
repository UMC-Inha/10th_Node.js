# 1. 미들웨어

## 1-1. 미들웨어란?

Express에서 미들웨어는 **요청(Request)이 들어오고 응답(Response)이 나가기 전까지 중간에서 실행되는 함수**이다.

Express 공식 문서에서는 미들웨어가 `req`, `res`, `next`에 접근할 수 있고, 코드를 실행하거나 요청/응답 객체를 수정하거나, 요청-응답 사이클을 종료하거나, 다음 미들웨어를 호출할 수 있다고 설명한다. 

쉽게 말하면 미들웨어는 다음과 같은 역할을 한다.

```
클라이언트 요청
→ 미들웨어 1
→ 미들웨어 2
→ 라우터/컨트롤러
→ 응답
```

즉, 미들웨어는 “요청과 응답 사이에 끼워 넣는 공통 처리 함수”라고 이해하면 된다.

---

## 1-2. 미들웨어의 기본 구조

```tsx
const myLogger = (req, res, next) => {
  console.log("요청이 들어왔습니다.");

  // 다음 미들웨어 또는 라우터로 넘김
  next();
};

app.use(myLogger);
```

### 코드 설명

```tsx
const myLogger = (req, res, next) => {
```

- `req`: 클라이언트가 보낸 요청 정보
- `res`: 서버가 클라이언트에게 보낼 응답 객체
- `next`: 다음 미들웨어로 넘어가기 위한 함수

```tsx
next();
```

- 현재 미들웨어에서 처리가 끝났으니 다음 단계로 넘어가라는 의미이다.
- `next()`를 호출하지 않고 응답도 보내지 않으면 요청이 중간에서 멈출 수 있다.

---

## 1-3. 미들웨어 실행 흐름

```tsx
app.use((req, res, next) => {
  console.log("1번 미들웨어 실행");
  next();
});

app.use((req, res, next) => {
  console.log("2번 미들웨어 실행");
  next();
});

app.get("/", (req, res) => {
  res.send("Hello UMC!");
});
```

요청이 `/`로 들어오면 실행 순서는 다음과 같다.

```
1번 미들웨어 실행
→ 2번 미들웨어 실행
→ GET / 라우터 실행
→ 응답 반환
```

미들웨어는 **등록한 순서대로 실행**된다.

그래서 `express.json()`처럼 요청 body를 파싱하는 미들웨어는 라우터보다 먼저 등록해야 한다.

---

## 1-4. 자주 사용하는 미들웨어

| 미들웨어 | 역할 | 사용 이유 |
| --- | --- | --- |
| `express.json()` | JSON 요청 body 파싱 | `req.body`로 JSON 데이터 사용 |
| `express.urlencoded()` | form 데이터 파싱 | HTML form 요청 처리 |
| `morgan` | 요청/응답 로그 기록 | API 요청 디버깅 |
| `cookie-parser` | 쿠키 문자열 파싱 | `req.cookies`로 쿠키 사용 |
| 인증 미들웨어 | 로그인 여부 검사 | 보호된 라우트 접근 제한 |
| 에러 미들웨어 | 오류 응답 통일 | 서버 오류 처리 일관화 |

`morgan`은 Node.js용 HTTP request logger middleware로 소개되어 있고, 요청 로그를 보기 좋게 확인할 때 사용한다. 

`cookie-parser`는 요청의 Cookie header를 파싱해서 `req.cookies` 객체에 넣어주는 미들웨어이다. 

---

## 1-5. express.json()과 express.urlencoded()

```tsx
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
```

### express.json()

```tsx
app.use(express.json());
```

클라이언트가 다음처럼 JSON을 보냈을 때:

```json
{
  "email": "test@example.com",
  "name": "찬찬"
}
```

서버에서 이렇게 사용할 수 있게 해준다.

```tsx
console.log(req.body.email);
console.log(req.body.name);
```

### express.urlencoded()

```tsx
app.use(express.urlencoded({ extended: false }));
```

HTML form처럼 `application/x-www-form-urlencoded` 형식으로 들어오는 데이터를 파싱한다.

예를 들어 로그인 폼에서 다음 값이 전송되면:

```
email=test@example.com&password=1234
```

Express에서 `req.body.email`, `req.body.password`로 접근할 수 있게 된다.

---

## 1-6. morgan 예시

```tsx
import morgan from "morgan";

app.use(morgan("dev"));
```

요청이 들어오면 터미널에 다음과 비슷한 로그가 찍힌다.

```
GET /api/v1/users/1/reviews 200 12.345 ms - 512
```

### 의미

| 항목 | 의미 |
| --- | --- |
| `GET` | HTTP 메서드 |
| `/api/v1/users/1/reviews` | 요청 URL |
| `200` | 응답 상태 코드 |
| `12.345 ms` | 응답 시간 |
| `512` | 응답 크기 |

스터디에서 설명할 때는 이렇게 말하면 된다.

> morgan은 API 요청이 들어왔을 때 어떤 메서드와 URL로 요청이 왔고, 몇 번 상태 코드로 응답했으며, 응답 시간이 얼마나 걸렸는지 로그로 보여주는 미들웨어입니다. 그래서 개발 중 API가 제대로 호출되는지 확인할 때 유용합니다.
> 

---

## 1-7. cookie-parser 예시

```tsx
import cookieParser from "cookie-parser";

app.use(cookieParser());
```

쿠키를 생성하는 라우터:

```tsx
app.get("/set-cookie", (req, res) => {
  res.cookie("username", "UMC10th", { maxAge: 60 * 60 * 1000 });
  res.send("쿠키가 생성되었습니다.");
});
```

쿠키를 읽는 라우터:

```tsx
app.get("/get-cookie", (req, res) => {
  const username = req.cookies.username;

  if (!username) {
    return res.send("쿠키가 없습니다.");
  }

  res.send(`현재 사용자: ${username}`);
});
```

`cookie-parser`를 사용하지 않으면 쿠키는 단순한 문자열 형태로 들어오기 때문에 직접 파싱해야 한다.

하지만 `cookie-parser`를 사용하면 `req.cookies.username`처럼 객체 형태로 바로 사용할 수 있다.

---

## 1-8. 인증 미들웨어 예시

```tsx
const isLogin = (req, res, next) => {
  const { username } = req.cookies;

  if (!username) {
    return res.status(401).json({
      message: "로그인이 필요합니다.",
    });
  }

  next();
};

app.get("/mypage", isLogin, (req, res) => {
  res.send("마이페이지입니다.");
});
```

### 흐름

```
GET /mypage 요청
→ isLogin 미들웨어 실행
→ 쿠키에 username 있는지 확인
→ 있으면 next()
→ 없으면 401 응답
```

### 스터디 설명 멘트

> 미들웨어는 공통 로직을 라우터 앞에 끼워 넣을 수 있다는 점이 핵심입니다. 예를 들어 마이페이지처럼 로그인한 사용자만 접근해야 하는 라우트가 있다면, 라우터마다 로그인 확인 코드를 반복해서 쓰지 않고 `isLogin` 미들웨어로 분리할 수 있습니다.
> 

---

## 1-9. 미들웨어 장점과 주의점

| 구분 | 내용 |
| --- | --- |
| 장점 | 공통 로직을 재사용할 수 있다. |
| 장점 | 라우터 코드가 깔끔해진다. |
| 장점 | 인증, 로깅, 파싱, 에러 처리 같은 기능을 분리할 수 있다. |
| 주의점 | `next()`를 빼먹으면 요청이 멈출 수 있다. |
| 주의점 | 등록 순서가 중요하다. |
| 주의점 | 너무 많은 기능을 하나의 미들웨어에 넣으면 오히려 복잡해진다. |

---
# 2.  HTTP 상태 코드

## 2-1. HTTP 상태 코드란?

HTTP 상태 코드는 서버가 클라이언트의 요청을 처리한 결과를 숫자로 표현한 것이다.

MDN에서는 HTTP 응답 상태 코드를 요청이 성공적으로 완료되었는지 알려주는 값으로 설명하며, 크게 100번대 정보 응답, 200번대 성공, 300번대 리다이렉션, 400번대 클라이언트 오류, 500번대 서버 오류로 구분한다.

---

## 2-2. 상태 코드 큰 분류

| 범위 | 이름 | 의미 |
| --- | --- | --- |
| 100번대 | Informational | 요청을 계속 처리 중 |
| 200번대 | Success | 요청 성공 |
| 300번대 | Redirection | 다른 위치로 이동 필요 |
| 400번대 | Client Error | 클라이언트 요청 문제 |
| 500번대 | Server Error | 서버 내부 문제 |

---

## 2-3. 백엔드 API에서 자주 쓰는 상태 코드

| 상태 코드 | 이름 | 사용 상황 |
| --- | --- | --- |
| 200 | OK | 조회, 수정, 삭제 성공 |
| 201 | Created | 회원가입, 리뷰 생성, 미션 생성 성공 |
| 400 | Bad Request | 요청 형식이 잘못됨 |
| 401 | Unauthorized | 인증이 필요함 |
| 403 | Forbidden | 권한이 없음 |
| 404 | Not Found | 리소스를 찾을 수 없음 |
| 409 | Conflict | 현재 리소스 상태와 충돌 |
| 500 | Internal Server Error | 서버 내부 오류 |

`200 OK`는 요청이 성공했음을 의미한다. 

`400 Bad Request`는 요청 문법이나 메시지 형식이 잘못되었을 때 사용할 수 있다.

`409 Conflict`는 요청이 현재 리소스의 상태와 충돌할 때 사용되며, 이메일 중복 같은 상황에 사용할 수 있다. 

---

## 2-4. 상황별 상태 코드 예시

### 회원가입 성공

```tsx
res.status(201).json({
  resultType: "SUCCESS",
  error: null,
  success: {
    userId: 1,
  },
});
```

### 이미 존재하는 이메일

```tsx
res.status(409).json({
  resultType: "FAIL",
  error: {
    errorCode: "U001",
    reason: "이미 존재하는 이메일입니다.",
    data: null,
  },
  success: null,
});
```

### 존재하지 않는 사용자

```tsx
res.status(404).json({
  resultType: "FAIL",
  error: {
    errorCode: "U002",
    reason: "존재하지 않는 사용자입니다.",
    data: null,
  },
  success: null,
});
```

### 요청 body 누락

```tsx
res.status(400).json({
  resultType: "FAIL",
  error: {
    errorCode: "C001",
    reason: "필수 요청 값이 누락되었습니다.",
    data: null,
  },
  success: null,
});
```

---

## 2-5. 상태 코드를 잘 써야 하는 이유

상태 코드는 프론트엔드와 백엔드 사이의 약속이다.

예를 들어 모든 실패를 `200 OK`로 보내면 프론트엔드는 요청이 성공한 것인지 실패한 것인지 응답 body를 일일이 확인해야 한다.

반대로 상태 코드를 의미 있게 사용하면 프론트엔드에서 다음처럼 분기 처리할 수 있다.

```tsx
if (response.status === 401) {
  alert("로그인이 필요합니다.");
}

if (response.status === 409) {
  alert("이미 존재하는 이메일입니다.");
}
```

---
# 3. 에러 핸들링

## 3-1. 에러 핸들링이란?

에러 핸들링은 서버에서 오류가 발생했을 때, 그 오류를 잡아서 정해진 형식으로 클라이언트에게 응답하는 과정이다.

Express 공식 문서에서는 에러 핸들링을 동기/비동기 코드에서 발생한 에러를 Express가 잡고 처리하는 방식이라고 설명한다. Express에는 기본 에러 핸들러가 있지만, 실제 서비스에서는 응답 형식을 통일하기 위해 직접 에러 핸들링 미들웨어를 작성하는 경우가 많다.

---

## 3-2. 일반 미들웨어와 에러 미들웨어 차이

일반 미들웨어:

```tsx
app.use((req, res, next) => {
  console.log("일반 미들웨어");
  next();
});
```

에러 미들웨어:

```tsx
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ message: "서버 오류" });
});
```

Express에서 에러 처리 미들웨어는 반드시 인자를 4개 가져야 한다.

즉, `(err, req, res, next)` 형태여야 Express가 에러 핸들러로 인식한다.

---

## 3-3. 왜 에러 핸들링을 통일해야 할까?

기존 방식:

```json
{
  "message": "이미 존재하는 이메일입니다."
}
```

다른 API의 실패 응답:

```json
{
  "error": "존재하지 않는 가게입니다."
}
```

또 다른 API의 실패 응답:

```json
{
  "reason": "잘못된 요청입니다."
}
```

이렇게 API마다 실패 응답 모양이 다르면 프론트엔드에서 처리하기 어렵다.

그래서 이번 워크북에서는 성공/실패 응답을 다음처럼 통일하는 방향으로 진행한다.

성공 응답:

```json
{
  "resultType": "SUCCESS",
  "error": null,
  "success": {
    "userId": 1
  }
}
```

실패 응답:

```json
{
  "resultType": "FAIL",
  "error": {
    "errorCode": "U001",
    "reason": "이미 존재하는 이메일입니다.",
    "data": null
  },
  "success": null
}
```

워크북에서도 모든 오류가 `unknown`으로 내려가면 프론트엔드가 구체적인 안내를 하기 어렵기 때문에, `U001` 같은 에러 코드를 따로 두는 이유를 설명하고 있다.

---

## 3-4. AppError 만들기

기본 `Error` 객체는 `message`만 담기 쉽다.

하지만 API에서는 `statusCode`, `errorCode`, `data` 같은 정보도 필요하다.

그래서 공통 에러 클래스를 만든다.

```tsx
export class AppError extends Error {
  public readonly errorCode: string;
  public readonly statusCode: number;
  public readonly data?: unknown;

  constructor(params: {
    errorCode: string;
    message: string;
    statusCode: number;
    data?: unknown;
  }) {
    super(params.message);

    this.errorCode = params.errorCode;
    this.statusCode = params.statusCode;
    this.data = params.data ?? null;
  }
}
```

### 코드 설명

```tsx
extends Error
```

- 기본 Error 클래스를 상속한다.
- 기존 Error처럼 `message`를 사용할 수 있다.

```tsx
errorCode
```

- 프론트엔드가 오류 종류를 구분하기 위한 코드이다.
- 예: `U001`, `S001`, `M001`

```tsx
statusCode
```

- HTTP 상태 코드이다.
- 예: `400`, `404`, `409`, `500`

```tsx
data
```

- 오류와 관련된 추가 정보를 담는다.
- 예: 중복된 이메일 값, 잘못 들어온 요청 body 등

---

## 3-5. 구체적인 커스텀 에러 만들기

```tsx
import { AppError } from "./app.error.js";

export class DuplicateUserEmailError extends AppError {
  constructor(data?: unknown) {
    super({
      errorCode: "U001",
      message: "이미 존재하는 이메일입니다.",
      statusCode: 409,
      data,
    });
  }
}
```

이렇게 만들면 서비스 계층에서 다음처럼 사용할 수 있다.

```tsx
if (joinUserId === null) {
  throw new DuplicateUserEmailError(data);
}
```

기존 방식:

```tsx
throw new Error("이미 존재하는 이메일입니다.");
```

개선된 방식:

```tsx
throw new DuplicateUserEmailError(data);
```

### 차이점

| 방식 | 담을 수 있는 정보 |
| --- | --- |
| `new Error()` | message 중심 |
| `new AppError()` | message, errorCode, statusCode, data |
| `DuplicateUserEmailError` | 특정 오류 상황을 명확하게 표현 |

---

## 3-6. 전역 에러 핸들러

```tsx
app.use((err: AppError, req: Request, res: Response, next: NextFunction) => {
  if (res.headersSent) {
    return next(err);
  }

  res.status(err.statusCode || 500).json({
    resultType: "FAIL",
    error: {
      errorCode: err.errorCode || "UNKNOWN",
      reason: err.message || "서버 오류가 발생했습니다.",
      data: err.data || null,
    },
    success: null,
  });
});
```

### 코드 설명

```tsx
app.use((err, req, res, next) => {})
```

- Express의 전역 에러 처리 미들웨어이다.
- 라우터나 서비스에서 던진 에러가 여기로 모인다.

```tsx
if (res.headersSent) {
  return next(err);
}
```

- 이미 응답이 나간 상태라면, 다시 응답을 보내면 오류가 생길 수 있다.
- 이 경우 다음 에러 핸들러로 넘긴다.

```tsx
res.status(err.statusCode || 500)
```

- 커스텀 에러에 상태 코드가 있으면 그걸 사용한다.
- 없으면 기본값으로 500을 사용한다.

---

## 3-7. 에러 핸들링 흐름

```
Controller
→ Service
→ Error 발생
→ throw new AppError()
→ Express 전역 에러 미들웨어
→ 통일된 실패 응답 반환
```

예시:

```
회원가입 요청
→ userSignUp Service 실행
→ 이미 존재하는 이메일 확인
→ DuplicateUserEmailError 발생
→ 전역 에러 핸들러 실행
→ 409 + U001 응답 반환
```

---

# 4. Tsoa

## 4-1. Tsoa란?

Tsoa는 TypeScript 기반으로 API 컨트롤러를 작성하고, 이를 바탕으로 라우트와 OpenAPI 문서를 생성할 수 있게 도와주는 도구이다.

Tsoa 공식 문서의 getting started 흐름도 TypeScript 설정, 모델 정의, 컨트롤러 정의, Express 서버 생성, routes 파일 생성으로 구성되어 있다.

Tsoa를 적용하면 DTO 단계의 validation, Swagger 문서화, `@Route`, `@Get`, `@Post` 같은 데코레이터 기반 라우팅으로 API 구조를 파악하기 쉬워진다.

---

## 4-2. 기존 Express 라우팅 방식

```tsx
app.post("/api/v1/users/signup", handleUserSignUp);
```

기존 방식에서는 `index.ts` 또는 router 파일에서 URL과 handler를 직접 연결했다.

컨트롤러는 보통 이렇게 작성했다.

```tsx
export const handleUserSignUp = async (req, res, next) => {
  const user = await userSignUp(req.body);

  res.status(200).json({
    result: user,
  });
};
```

### 기존 방식의 특징

| 특징 | 설명 |
| --- | --- |
| 장점 | 구조가 단순하다. |
| 장점 | Express만 알아도 이해하기 쉽다. |
| 단점 | 라우트가 많아지면 `index.ts`가 복잡해진다. |
| 단점 | API 문서화와 validation을 별도로 관리해야 한다. |
| 단점 | 컨트롤러의 요청/응답 타입이 흐려질 수 있다. |

---

## 4-3. Tsoa 적용 방식

Tsoa를 사용하면 컨트롤러를 클래스 기반으로 작성하고, 데코레이터로 라우트를 선언한다.

```tsx
import { Body, Controller, Post, Route, Tags } from "tsoa";

@Route("users")
@Tags("Users")
export class UserController extends Controller {
  @Post("signup")
  public async handleUserSignUp(
    @Body() body: UserSignUpRequest
  ): Promise<ApiResponse<UserSignUpResponse>> {
    const user = await userSignUp(body);

    return success(user);
  }
}
```

### 코드 설명

```tsx
@Route("users")
```

- 이 컨트롤러의 기본 경로를 의미한다.
- 최종 경로는 `/api/v1/users`처럼 사용된다.

```tsx
@Tags("Users")
```

- Swagger/OpenAPI 문서에서 API를 그룹화할 때 사용한다.

```tsx
@Post("signup")
```

- `POST /users/signup` 엔드포인트를 의미한다.

```tsx
@Body() body: UserSignUpRequest
```

- 요청 body를 `UserSignUpRequest` 타입으로 받는다.

```tsx
Promise<ApiResponse<UserSignUpResponse>>
```

- 이 API가 어떤 응답 타입을 반환하는지 명확하게 보여준다.

---

## 4-4. tsoa.json 설정

```json
{
  "entryFile": "src/index.ts",
  "noImplicitAdditionalProperties": "throw-on-extras",
  "controllerPathGlobs": ["src/**/*.controller.ts"],
  "spec": {
    "outputDirectory": "dist",
    "specVersion": 3
  },
  "routes": {
    "routesDir": "src/generated"
  }
}
```

### 주요 설정 설명

| 옵션 | 의미 |
| --- | --- |
| `entryFile` | Express 서버 진입 파일 |
| `controllerPathGlobs` | Tsoa가 찾을 controller 파일 경로 |
| `noImplicitAdditionalProperties` | DTO에 없는 추가 필드 처리 방식 |
| `spec.outputDirectory` | OpenAPI spec 생성 위치 |
| `routes.routesDir` | 자동 생성 routes.ts 위치 |

`noImplicitAdditionalProperties: "throw-on-extras"`는 DTO에 정의되지 않은 값이 들어왔을 때 에러를 발생시키는 설정이다.

이 설정을 사용하면 클라이언트가 예상하지 않은 필드를 보냈을 때 더 엄격하게 검증할 수 있다.

---

## 4-5. routes 자동 생성

```bash
npx tsoa spec-and-routes
```

이 명령어를 실행하면 Tsoa가 컨트롤러를 읽고 `routes.ts`를 생성한다.

Tsoa의 routes 문서에서도 생성된 `routes.ts` 파일을 사용하는 방법을 설명하고 있다. 

생성된 routes를 Express에 연결한다.

```tsx
import { RegisterRoutes } from "./generated/routes.js";

const router = express.Router();

RegisterRoutes(router);

app.use("/api/v1", router);
```

### 흐름

```
Controller 작성
→ npx tsoa spec-and-routes
→ generated/routes.ts 생성
→ RegisterRoutes(router)
→ app.use("/api/v1", router)
→ API 사용 가능
```

---

## 4-6. package.json scripts

```json
{
  "scripts": {
    "start": "tsoa spec-and-routes && tsx src/index.ts",
    "dev": "tsoa spec-and-routes && nodemon --exec tsx src/index.ts"
  }
}
```

### 의미

서버를 실행할 때마다 Tsoa routes를 먼저 생성한 뒤 서버를 켠다.

```
tsoa spec-and-routes
→ generated/routes.ts 생성
→ tsx src/index.ts 실행
```

이렇게 하면 컨트롤러를 수정한 뒤 routes 생성 명령어를 깜빡하는 실수를 줄일 수 있다.

---

## 4-7. Tsoa 장점과 주의점

| 구분 | 내용 |
| --- | --- |
| 장점 | 데코레이터 기반으로 API 구조를 한눈에 볼 수 있다. |
| 장점 | TypeScript 타입과 DTO를 API 문서화에 활용할 수 있다. |
| 장점 | routes 파일을 자동 생성할 수 있다. |
| 장점 | OpenAPI/Swagger 문서화와 연결하기 좋다. |
| 주의점 | 설정 파일이 필요하다. |
| 주의점 | 컨트롤러 작성 방식이 기존 Express 함수형 방식과 다르다. |
| 주의점 | 컨트롤러 수정 후 routes 재생성이 필요하다. |
| 주의점 | generated 파일 관리 방식을 팀에서 정해야 한다. |

---

## 4-8. Tsoa와 Express 방식 비교

| 비교 항목 | Express 직접 라우팅 | Tsoa |
| --- | --- | --- |
| 라우트 등록 | `app.get`, `app.post` 직접 작성 | 데코레이터 기반 |
| 컨트롤러 형태 | 함수형 컨트롤러 | 클래스 기반 컨트롤러 |
| 문서화 | 별도 작성 필요 | OpenAPI 생성 가능 |
| 타입 활용 | 직접 관리 | DTO/응답 타입 활용 |
| 러닝 커브 | 낮음 | 상대적으로 있음 |
| 규모가 커질 때 | 라우트 관리가 복잡해질 수 있음 | 구조화에 유리 |