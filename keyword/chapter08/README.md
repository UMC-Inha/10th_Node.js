# 8주차 키워드 정리

# 1. Swagger & OpenAPI

## 1-1. 개념

Swagger는 REST API를 문서화하고, 브라우저에서 API를 직접 테스트할 수 있게 해주는 도구이다.  
OpenAPI는 REST API를 일정한 형식으로 설명하기 위한 표준 명세이다.

쉽게 말하면, OpenAPI는 API 문서를 작성하는 규칙이고, Swagger는 그 문서를 보기 좋게 보여주는 도구라고 이해할 수 있다.

| 구분 | 설명 |
|---|---|
| OpenAPI | API 명세를 작성하기 위한 표준 형식 |
| Swagger | OpenAPI 명세를 화면에서 확인하고 테스트할 수 있게 해주는 도구 |
| Swagger UI | API 목록, 요청값, 응답값을 브라우저에서 보여주는 화면 |
| swagger.json | OpenAPI 형식으로 생성된 API 명세 파일 |

---

## 1-2. 왜 필요한가?

백엔드에서 API를 만들면 프론트엔드 개발자는 다음 정보를 알아야 한다.

| 필요한 정보 | 예시 |
|---|---|
| API 주소 | `/api/v1/users/signup` |
| HTTP Method | `POST` |
| 요청 Body | `email`, `name`, `preferences` |
| Query Parameter | `cursor`, `limit` |
| Path Parameter | `/stores/{storeId}` |
| 성공 응답 | 회원가입 성공 데이터 |
| 실패 응답 | 중복 이메일, 잘못된 요청 등 |

Swagger가 없다면 프론트엔드 개발자는 API를 사용할 때마다 백엔드 개발자에게 직접 물어봐야 한다.

예를 들면 다음과 같은 질문이 계속 생긴다.

```txt
이 API 주소가 뭐예요?
POST인가요 GET인가요?
Body에는 어떤 값을 보내야 하나요?
성공하면 어떤 응답이 오나요?
실패하면 에러 응답은 어떤 구조인가요?
````

Swagger를 사용하면 이런 정보를 문서에서 바로 확인할 수 있기 때문에 협업이 훨씬 쉬워진다.

---

## 1-3. Swagger & OpenAPI 관계

```txt
TypeScript 코드
   ↓
TSOA
   ↓
OpenAPI 명세 생성
   ↓
swagger.json
   ↓
Swagger UI
   ↓
브라우저에서 API 문서 확인
```

즉, 이번 주차에서는 TSOA를 통해 OpenAPI 명세 파일인 `swagger.json`을 만들고, Swagger UI를 통해 그 명세를 화면으로 확인한다.

---

## 1-4. Swagger UI 설정 코드 예시

```ts
import express from "express";
import swaggerUi from "swagger-ui-express";
import path from "path";
import fs from "fs";

const app = express();

app.use(express.json());

// TSOA가 생성한 swagger.json 파일을 읽어온다.
// npx tsoa spec 명령어를 실행하면 dist/swagger.json 파일이 생성된다.
const swaggerFile = JSON.parse(
  fs.readFileSync(path.resolve("dist/swagger.json"), "utf8")
);

// /docs 경로에서 Swagger UI를 볼 수 있도록 설정한다.
// 사용자는 http://localhost:3000/docs 로 접속해서 API 문서를 확인할 수 있다.
app.use("/docs", swaggerUi.serve, swaggerUi.setup(swaggerFile));

app.listen(3000, () => {
  console.log("Server is running at http://localhost:3000");
  console.log("Swagger docs: http://localhost:3000/docs");
});
```

---

## 1-5. 코드 설명

| 코드                                  | 의미                                     |
| ----------------------------------- | -------------------------------------- |
| `swagger-ui-express`                | Express 서버에서 Swagger UI를 보여주기 위한 라이브러리 |
| `fs.readFileSync()`                 | 생성된 `swagger.json` 파일을 읽어옴             |
| `path.resolve("dist/swagger.json")` | `dist` 폴더 안의 Swagger 명세 파일 경로 지정       |
| `app.use("/docs", ...)`             | `/docs` 경로에 Swagger UI 연결              |

---

# 2. TSOA(TypeScript-first OpenAPI)

## 2-1. 개념

TSOA는 TypeScript-first OpenAPI 도구이다.
TypeScript 코드에 작성된 컨트롤러, 데코레이터, DTO 타입을 기반으로 OpenAPI 명세를 자동 생성해준다.

기존 Swagger 방식에서는 API 문서를 만들기 위해 긴 주석이나 JSON 구조를 직접 작성해야 했다.
하지만 TSOA를 사용하면 실제 TypeScript 코드와 타입을 기반으로 문서를 만들 수 있다.

---

## 2-2. 왜 필요한가?

API 문서를 사람이 직접 작성하면 실제 코드와 문서가 달라질 수 있다.

예를 들어 실제 회원가입 API에서는 `phoneNumber`라는 필드를 받는데, 문서에는 예전 이름인 `phone`이라고 적혀 있다면 프론트엔드 개발자는 잘못된 요청을 보낼 수 있다.

TSOA를 사용하면 TypeScript 타입을 기준으로 문서를 만들기 때문에 이런 불일치를 줄일 수 있다.

---

## 2-3. TSOA의 핵심 흐름

```txt
1. Controller 작성
2. DTO 타입 작성
3. TSOA 데코레이터 추가
4. npx tsoa spec 실행
5. dist/swagger.json 생성
6. Swagger UI에서 API 문서 확인
```

---

## 2-4. 주요 데코레이터 정리

| 데코레이터         | 역할                  | 예시                                            |
| ------------- | ------------------- | --------------------------------------------- |
| `@Route()`    | 컨트롤러의 기본 경로 지정      | `@Route("users")`                             |
| `@Tags()`     | Swagger에서 API 그룹 지정 | `@Tags("Users")`                              |
| `@Get()`      | GET 요청 API 정의       | `@Get("{userId}")`                            |
| `@Post()`     | POST 요청 API 정의      | `@Post("signup")`                             |
| `@Patch()`    | PATCH 요청 API 정의     | `@Patch("{userId}")`                          |
| `@Delete()`   | DELETE 요청 API 정의    | `@Delete("{userId}")`                         |
| `@Body()`     | 요청 Body 데이터 받기      | `@Body() body: UserSignUpRequest`             |
| `@Query()`    | Query Parameter 받기  | `@Query() cursor?: number`                    |
| `@Path()`     | Path Parameter 받기   | `@Path() userId: number`                      |
| `@Response()` | 응답 상태 코드 문서화        | `@Response<ApiResponse<null>>(409, "중복 이메일")` |

---

## 2-5. 회원가입 API 예시 코드

```ts
import {
  Body,
  Controller,
  Post,
  Route,
  Tags,
  Response as TsoaResponse,
} from "tsoa";

import { userSignUp } from "../services/user.service";
import { success } from "../../../common/response";
import {
  UserSignUpRequest,
  UserSignUpResponse,
  ApiResponse,
} from "../dtos/user.dto";

// @Route는 이 컨트롤러의 기본 API 경로를 지정한다.
// tsoa.json의 basePath가 /api/v1이라면 최종 경로는 /api/v1/users가 된다.
@Route("users")

// @Tags는 Swagger UI에서 API를 그룹화하는 이름이다.
// Swagger 화면에서 Users라는 그룹 아래에 이 API들이 표시된다.
@Tags("Users")
export class UserController extends Controller {
  /**
   * 회원가입 API
   *
   * 사용자의 이메일, 이름, 성별, 생년월일, 주소, 전화번호, 선호 카테고리를 입력받아
   * 새로운 사용자를 생성한다.
   *
   * @summary 회원가입을 처리하는 API입니다.
   */

  // POST /users/signup 요청을 처리한다.
  @Post("signup")

  // 성공 응답을 Swagger 문서에 표시한다.
  @TsoaResponse<ApiResponse<UserSignUpResponse>>(200, "회원가입 성공")

  // 잘못된 요청 형식에 대한 실패 응답을 문서화한다.
  @TsoaResponse<ApiResponse<null>>(400, "잘못된 요청 형식")

  // 이미 가입된 이메일에 대한 실패 응답을 문서화한다.
  @TsoaResponse<ApiResponse<null>>(409, "이미 존재하는 이메일")
  public async handleUserSignUp(
    // @Body는 HTTP 요청 Body에 담긴 JSON 데이터를 받는다.
    @Body() body: UserSignUpRequest
  ): Promise<ApiResponse<UserSignUpResponse>> {
    const user = await userSignUp(body);

    // 7주차에서 만든 표준 응답 형식으로 성공 응답을 반환한다.
    return success(user);
  }
}
```

---

## 2-6. DTO 예시 코드

```ts
// API 표준 응답 형식
export interface ApiResponse<T> {
  /** 요청 처리 결과 타입 */
  resultType: "SUCCESS" | "FAIL";

  /** 실패 시 에러 정보, 성공 시 null */
  error: ErrorResponse | null;

  /** 성공 시 응답 데이터, 실패 시 null */
  success: T | null;
}

// API 실패 응답 형식
export interface ErrorResponse {
  /** 에러 코드 */
  errorCode: string;

  /** 에러 메시지 */
  reason: string;

  /** 추가 에러 데이터 */
  data?: unknown;
}

// 회원가입 요청 DTO
export interface UserSignUpRequest {
  /** 유저 이메일. 로그인 시 사용된다. */
  email: string;

  /** 유저 이름 */
  name: string;

  /** 성별 */
  gender: string;

  /** 생년월일. YYYY-MM-DD 형식 */
  birth: string;

  /** 기본 주소 */
  address: string;

  /** 상세 주소 */
  detailAddress: string;

  /** 전화번호 */
  phoneNumber: string;

  /** 선호 음식 카테고리 ID 배열 */
  preferences: number[];
}

// 회원가입 성공 응답 DTO
export interface UserSignUpResponse {
  /** 생성된 유저 ID */
  id: number;

  /** 유저 이메일 */
  email: string;

  /** 유저 이름 */
  name: string;
}
```

---

## 2-7. Response 이름 충돌 해결

Express와 TSOA는 모두 `Response`라는 이름을 사용할 수 있다.
그래서 둘을 같은 파일에서 함께 import하면 이름이 충돌할 수 있다.

이때는 `as`를 사용해 별칭을 붙인다.

```ts
// TSOA의 Response 데코레이터는 TsoaResponse라는 이름으로 사용한다.
import { Response as TsoaResponse } from "tsoa";

// Express의 Response 타입은 ExpressResponse라는 이름으로 사용한다.
import { Response as ExpressResponse } from "express";
```

사용할 때는 다음처럼 작성한다.

```ts
@TsoaResponse<ApiResponse<null>>(409, "이미 존재하는 이메일")
```

---

## 2-8. tsoa.json 예시

```json
{
  "entryFile": "src/index.ts",
  "noImplicitAdditionalProperties": "throw-on-extras",
  "controllerPathGlobs": ["src/modules/**/*.controller.ts"],
  "spec": {
    "outputDirectory": "dist",
    "specVersion": 3,
    "basePath": "/api/v1"
  },
  "routes": {
    "routesDir": "src/generated"
  }
}
```

---

## 2-9. tsoa.json 항목 설명

| 항목                    | 의미                     |
| --------------------- | ---------------------- |
| `entryFile`           | 서버의 진입 파일              |
| `controllerPathGlobs` | TSOA가 읽을 컨트롤러 파일 경로    |
| `outputDirectory`     | `swagger.json`이 생성될 위치 |
| `specVersion`         | OpenAPI 명세 버전          |
| `basePath`            | API의 공통 기본 경로          |
| `routesDir`           | TSOA가 라우트 파일을 생성할 위치   |

---

## 2-10. Swagger 명세 생성 명령어

```bash
npx tsoa spec
```

이 명령어를 실행하면 다음 파일이 생성된다.

```txt
dist/swagger.json
```

이후 서버를 실행하고 다음 주소로 접속한다.

```txt
http://localhost:3000/docs
```

---

# 3. Type-Driven Documentation

## 3-1. 개념

Type-Driven Documentation은 타입을 기반으로 문서를 만드는 방식이다.
즉, TypeScript의 `interface`, `type`, `class` 같은 타입 정보를 API 문서의 기준으로 사용하는 방식이다.

이번 주차에서 TSOA가 바로 이 방식을 사용한다.

---

## 3-2. 기존 문서화 방식과 비교

| 구분         | 기존 Swagger 주석 방식      | Type-Driven Documentation |
| ---------- | --------------------- | ------------------------- |
| 문서 작성 기준   | 사람이 직접 작성한 주석         | TypeScript 타입             |
| 요청 Body 설명 | JSON 구조를 직접 작성        | DTO 타입 기반 자동 반영           |
| 응답 구조 설명   | 주석으로 직접 작성            | 응답 타입 기반 반영               |
| 유지보수       | 코드가 바뀌면 문서도 따로 수정해야 함 | 타입 변경이 문서에 반영되기 쉬움        |
| 실수 가능성     | 필드명, 타입 불일치 가능성 높음    | 실제 타입을 기준으로 하므로 불일치 감소    |
| 가독성        | 주석이 길어지면 복잡함          | DTO와 컨트롤러 중심으로 정리 가능      |

---

## 3-3. 예시 코드

아래처럼 리뷰 생성 요청 DTO를 작성했다고 가정한다.

```ts
export interface CreateReviewRequest {
  /** 리뷰 내용 */
  content: string;

  /** 리뷰 평점 */
  rating: number;

  /** 방문한 가게 ID */
  storeId: number;
}
```

TSOA는 이 타입을 읽어서 Swagger 문서에 요청 Body 구조를 표시할 수 있다.

Swagger에서는 대략 다음과 같은 요청 형식을 확인할 수 있다.

```json
{
  "content": "음식이 맛있어요",
  "rating": 5,
  "storeId": 1
}
```

즉, 문서를 따로 새로 쓰는 것이 아니라 실제 코드의 타입이 문서의 기준이 된다.

---

## 3-4. DTO 주석이 중요한 이유

타입만 작성하면 필드의 이름과 자료형은 알 수 있지만, 그 필드가 어떤 의미인지는 부족할 수 있다.
그래서 DTO에 주석을 달아주는 것이 중요하다.

```ts
export interface StoreReviewRequest {
  /** 리뷰를 작성할 가게 ID */
  storeId: number;

  /** 리뷰 내용 */
  content: string;

  /** 리뷰 평점. 1점부터 5점까지 입력 가능 */
  score: number;
}
```

이렇게 작성하면 프론트엔드 개발자가 Swagger 문서를 볼 때 각 필드의 의미를 더 쉽게 이해할 수 있다.

---

## 3-5. 실패 응답 문서화가 중요한 이유

성공 응답만 문서화하면 프론트엔드 개발자는 에러 상황을 예상하기 어렵다.
따라서 실패할 수 있는 경우도 `@Response`를 사용해 명시해야 한다.

```ts
@Post("signup")
@TsoaResponse<ApiResponse<UserSignUpResponse>>(200, "회원가입 성공")
@TsoaResponse<ApiResponse<null>>(400, "요청 값 검증 실패")
@TsoaResponse<ApiResponse<null>>(409, "이미 존재하는 이메일")
public async handleUserSignUp(
  @Body() body: UserSignUpRequest
): Promise<ApiResponse<UserSignUpResponse>> {
  const user = await userSignUp(body);
  return success(user);
}
```

이렇게 작성하면 Swagger 문서에서 성공 케이스뿐 아니라 실패 케이스도 함께 확인할 수 있다.

---

## 3-6. Type-Driven Documentation의 장점

| 장점             | 설명                                     |
| -------------- | -------------------------------------- |
| 코드와 문서의 일치성 증가 | 실제 TypeScript 타입을 기반으로 문서를 만들기 때문      |
| 협업 효율 향상       | 프론트엔드 개발자가 Swagger를 보고 요청/응답 구조를 파악 가능 |
| 유지보수 편리        | DTO를 수정하면 문서도 함께 갱신하기 쉬움               |
| 실수 감소          | 손으로 JSON 구조를 반복 작성하지 않아도 됨             |
| API 이해도 향상     | 컨트롤러, DTO, 응답 타입이 하나의 흐름으로 연결됨         |

---

## 3-7. 주의할 점

Type-Driven Documentation을 사용한다고 해서 모든 문서가 자동으로 완벽해지는 것은 아니다.

| 주의점           | 설명                                  |
| ------------- | ----------------------------------- |
| DTO 주석 작성 필요  | 필드 설명은 개발자가 직접 적어야 한다               |
| 실패 응답 명시 필요   | `@Response`로 실패 케이스를 직접 적어야 한다      |
| API 설명 작성 필요  | `/** ... */` 주석으로 API 설명을 보완해야 한다   |
| 명세 재생성 필요     | 코드 수정 후 `npx tsoa spec`을 다시 실행해야 한다 |
| Swagger 확인 필요 | `/docs`에서 문서가 의도대로 나오는지 확인해야 한다     |

---


# 4. 세 키워드의 관계 정리

세 키워드는 따로 떨어진 개념이 아니라 하나의 흐름으로 연결된다.

```txt
TypeScript 코드와 DTO 작성
        ↓
TSOA가 타입과 데코레이터를 분석
        ↓
OpenAPI 명세 생성
        ↓
swagger.json 생성
        ↓
Swagger UI에서 API 문서 확인
```

| 단계           | 사용되는 개념                   |
| ------------ | ------------------------- |
| API 컨트롤러 작성  | TSOA                      |
| 요청/응답 DTO 작성 | Type-Driven Documentation |
| API 명세 생성    | OpenAPI                   |
| API 문서 화면 확인 | Swagger UI                |

---

# 5. 최종 요약

8주차의 핵심은 API를 단순히 구현하는 것에서 끝나는 것이 아니라, 프론트엔드 개발자가 이해하고 사용할 수 있도록 문서화하는 것이다.

Swagger는 API 문서를 시각적으로 확인하고 테스트할 수 있게 해주는 도구이다.
OpenAPI는 API 명세를 작성하기 위한 표준 형식이다.
TSOA는 TypeScript 코드와 데코레이터를 기반으로 OpenAPI 문서를 생성해주는 도구이다.
Type-Driven Documentation은 실제 코드의 타입 정보를 문서의 기준으로 삼는 방식이다.

따라서 이번 주차는 TypeScript 타입과 TSOA 데코레이터를 활용해 API 문서를 만들고, Swagger UI를 통해 프론트엔드와 협업 가능한 API 명세를 제공하는 방법을 배우는 주차라고 정리할 수 있다.

```