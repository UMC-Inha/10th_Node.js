
# 7주차 미션 README

이번 7주차 미션에서는 기존 Express 프로젝트에 Tsoa를 적용하고, API 응답 구조와 에러 처리 방식을 통일했다.

## 1. 작업 내용

- Tsoa 기반 Controller 적용
- `tsoa spec-and-routes`로 routes 자동 생성
- 성공 응답 wrapper 적용
- `AppError`, `DuplicateUserEmailError` 생성
- 전역 에러 핸들러 적용
- `morgan`, `cookie-parser` 미들웨어 적용
- Postman으로 성공/실패 응답 테스트

---

## 2. Tsoa 적용

기존에는 `index.ts`에서 직접 라우트를 등록했다.

```ts
app.post("/api/v1/users/signup", handleUserSignUp);
````

Tsoa 적용 후에는 Controller에서 데코레이터로 API를 정의했다.

```ts
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

이후 자동 생성된 routes를 `index.ts`에 연결했다.

```ts
const router = express.Router();

RegisterRoutes(router);

app.use("/api/v1", router);
```

---

## 3. 성공 응답 통일

```ts
export interface ApiResponse<T> {
  resultType: "SUCCESS";
  error: null;
  success: T;
}

export const success = <T>(data: T): ApiResponse<T> => ({
  resultType: "SUCCESS",
  error: null,
  success: data,
});
```

성공 응답 예시:

```json
{
  "resultType": "SUCCESS",
  "error": null,
  "success": {
    "userId": 1,
    "preferences": ["한식", "중식"]
  }
}
```

---

## 4. 실패 응답 통일

기본 `Error` 대신 커스텀 Error를 만들었다.

```ts
export class AppError extends Error {
  public readonly errorCode: string;
  public readonly statusCode: number;
  public readonly data?: unknown;
}
```

이메일 중복 에러는 별도 클래스로 분리했다.

```ts
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

Service에서는 중복 이메일 발생 시 해당 에러를 던지도록 수정했다.

```ts
if (joinUserId === null) {
  throw new DuplicateUserEmailError(data);
}
```

---

## 5. 전역 에러 핸들러

```ts
app.use((err: AppError, req: Request, res: Response, next: NextFunction) => {
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

실패 응답 예시:

```json
{
  "resultType": "FAIL",
  "error": {
    "errorCode": "U001",
    "reason": "이미 존재하는 이메일입니다.",
    "data": {}
  },
  "success": null
}
```

---

## 6. 미들웨어 적용

```ts
app.use(morgan("dev"));
app.use(cookieParser());
```

* `morgan`: 요청 메서드, URL, 상태 코드, 응답 시간을 터미널에 출력
* `cookie-parser`: 쿠키 값을 `req.cookies`로 쉽게 읽을 수 있게 처리

인증 미들웨어에서는 쿠키에 `username`이 있는지 확인했다.

```ts
const { username } = req.cookies;

if (username) {
  next();
} else {
  res.status(401).send("로그인이 필요합니다.");
}
```

---

## 7. 테스트 결과

### 회원가입 성공

```json
{
  "resultType": "SUCCESS",
  "error": null,
  "success": {
    "userId": 1,
    "preferences": ["한식", "중식"]
  }
}
```

### 중복 이메일 실패

```json
{
  "resultType": "FAIL",
  "error": {
    "errorCode": "U001",
    "reason": "이미 존재하는 이메일입니다."
  },
  "success": null
}
```

---

## 8. 정리

이번 미션을 통해 API는 기능 구현뿐만 아니라 응답 구조와 에러 처리 방식도 일관되게 관리해야 한다는 것을 알 수 있었다.

Tsoa를 통해 API 경로와 요청/응답 타입을 Controller에서 명확하게 표현할 수 있었고, `success()` wrapper와 `AppError`를 통해 성공/실패 응답 구조를 통일할 수 있었다.