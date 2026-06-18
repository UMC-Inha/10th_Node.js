# 1. OAuth 2.0

## 1-1. 개념

OAuth 2.0은 사용자가 Google, Kakao, Naver, GitHub 같은 외부 서비스 계정으로 다른 서비스에 로그인할 수 있게 해주는 인증/인가 표준이다.

예를 들어 사용자가 우리 서비스에서 “Google로 로그인” 버튼을 누르면, 우리 서비스가 사용자의 Google 비밀번호를 직접 받는 것이 아니다. 대신 Google이 사용자를 인증하고, 우리 서버는 Google로부터 사용자의 이메일, 이름, 프로필 정보 등을 전달받는다.

즉, OAuth 2.0은 다음과 같은 상황을 가능하게 한다.

```text
사용자 → Google에 로그인
Google → 우리 서버에 사용자 정보 전달
우리 서버 → 사용자 정보 기반으로 로그인 처리
```

---

## 1-2. 왜 필요한가?

사용자가 모든 웹사이트마다 새로운 이메일과 비밀번호를 만들어야 한다면 불편하다. 또한 우리 서버가 직접 비밀번호를 저장하고 관리해야 하므로 보안 부담도 커진다.

OAuth 2.0을 사용하면 사용자는 기존 Google 계정으로 쉽게 로그인할 수 있고, 우리 서비스는 사용자의 비밀번호를 직접 다루지 않아도 된다.

| 장점        | 설명                                               |
| --------- | ------------------------------------------------ |
| 사용자 편의성   | 별도 회원가입 없이 Google 계정으로 로그인 가능                    |
| 보안 부담 감소  | 우리 서버가 Google 비밀번호를 직접 저장하지 않음                   |
| 사용자 정보 활용 | 이메일, 이름, 프로필 이미지 등을 받아올 수 있음                     |
| 표준화       | Google, Kakao, Naver 등 대부분의 소셜 로그인이 OAuth 흐름을 따름 |

---

## 1-3. OAuth 2.0 주요 용어

| 용어                   | 의미                           |
| -------------------- | ---------------------------- |
| Resource Owner       | 사용자                          |
| Client               | 우리 서비스 또는 우리 서버              |
| Authorization Server | Google 로그인과 권한 동의를 처리하는 서버   |
| Resource Server      | Google 사용자 정보를 제공하는 서버       |
| Authorization Code   | Google 로그인 성공 후 발급되는 일회용 코드  |
| Access Token         | Google API를 호출하기 위해 사용하는 토큰  |
| Redirect URI         | Google 로그인 후 다시 돌아올 우리 서버 주소 |
| Scope                | 우리 서비스가 요청하는 사용자 정보 범위       |

---

## 1-4. OAuth 2.0 흐름

Google 로그인 기준으로 OAuth 2.0 흐름은 다음과 같다.

```text
1. 사용자가 /oauth2/login/google 접속
2. 서버가 사용자를 Google 로그인 화면으로 이동시킴
3. 사용자가 Google 로그인 및 권한 동의
4. Google이 Authorization Code를 Redirect URI로 전달
5. 우리 서버가 Authorization Code를 Google Access Token으로 교환
6. 우리 서버가 Google Access Token으로 사용자 프로필 정보 조회
7. 이메일 기준으로 우리 DB에서 사용자 조회
8. 사용자가 없으면 회원가입 처리
9. 우리 서버의 Access Token / Refresh Token 발급
```

이때 중요한 점은 Google의 Access Token과 우리 서버의 Access Token이 다르다는 것이다.

| 구분                  | 의미                              |
| ------------------- | ------------------------------- |
| Google Access Token | Google API에서 사용자 정보를 가져오기 위한 토큰 |
| 우리 서버의 Access Token | 우리 서비스 API를 인증하기 위한 JWT         |

---

## 1-5. Authorization Code가 필요한 이유

OAuth 2.0에서는 Google 로그인 성공 후 바로 Access Token을 전달하지 않고, 먼저 Authorization Code를 전달한다.

예시:

```text
http://localhost:3000/oauth2/callback/google?code=abc123
```

이 code는 일회용 코드이다. 우리 서버는 이 code를 Google 서버에 보내서 실제 Access Token으로 교환한다.

왜 바로 Access Token을 주지 않을까?

Authorization Code는 브라우저 주소창을 통해 전달되므로 노출될 위험이 있다. 그래서 이 code는 한 번만 사용할 수 있는 중간 단계로 두고, 실제 Access Token 교환은 서버와 Google 서버 사이에서 이루어지도록 한다.

---

## 1-6. Google 로그인 라우트 예시

```ts
app.get(
  "/oauth2/login/google",
  passport.authenticate("google", { session: false })
);

app.get(
  "/oauth2/callback/google",
  passport.authenticate("google", {
    session: false,
    failureRedirect: "/login-failed",
  }),
  (req, res) => {
    res.status(200).json({
      success: true,
      tokens: req.user,
    });
  }
);
```

### 코드 설명

| 코드                                | 설명                                          |
| --------------------------------- | ------------------------------------------- |
| `/oauth2/login/google`            | 사용자를 Google 로그인 화면으로 이동시키는 경로               |
| `/oauth2/callback/google`         | Google 로그인 성공 후 돌아오는 경로                     |
| `passport.authenticate("google")` | Passport의 Google Strategy 실행                |
| `session: false`                  | 세션 방식이 아니라 JWT 방식을 사용할 것이므로 세션 비활성화         |
| `req.user`                        | Google 로그인 성공 후 Strategy에서 넘겨준 사용자 또는 토큰 정보 |


---

# 2. JWT

## 2-1. 개념

JWT는 JSON Web Token의 약자이다.
사용자 정보를 JSON 형태로 담고, 서버의 비밀 키로 서명한 토큰이다.

로그인에 성공한 사용자에게 JWT를 발급하면, 이후 사용자는 API 요청마다 이 토큰을 보내 자신이 로그인한 사용자임을 증명할 수 있다.

JWT는 다음과 같은 형태를 가진다.

```text
xxxxx.yyyyy.zzzzz
```

즉, 점(`.`)을 기준으로 3부분으로 나뉜다.

```text
Header.Payload.Signature
```

---

## 2-2. JWT 구조

| 구성 요소     | 설명                |
| --------- | ----------------- |
| Header    | 토큰 타입과 서명 알고리즘 정보 |
| Payload   | 사용자 식별 정보         |
| Signature | 토큰 위조 여부를 검증하는 서명 |

예를 들어 Payload에는 다음과 같은 정보가 들어갈 수 있다.

```json
{
  "id": 1,
  "email": "user@example.com"
}
```

하지만 Payload에는 비밀번호 같은 민감한 정보를 넣으면 안 된다.
Header와 Payload는 암호화된 것이 아니라 Base64Url로 인코딩된 것이기 때문에 누구나 디코딩해서 내용을 볼 수 있다.

JWT에서 중요한 보안 요소는 Signature이다.
서버는 비밀 키를 이용해 Signature를 검증하고, 이 토큰이 위조되지 않았는지 확인한다.

---

## 2-3. JWT를 사용하는 이유

기존 세션 방식에서는 서버가 사용자의 로그인 상태를 기억해야 한다.
반면 JWT 방식에서는 클라이언트가 토큰을 가지고 있고, 서버는 요청이 들어올 때마다 토큰의 유효성을 검증한다.

| 구분           | 세션 방식          | JWT 방식            |
| ------------ | -------------- | ----------------- |
| 로그인 상태 저장 위치 | 서버             | 클라이언트             |
| 서버 상태 관리     | 필요함            | 상대적으로 적음          |
| 확장성          | 서버가 세션을 공유해야 함 | 토큰 검증만 하면 됨       |
| 로그아웃 처리      | 세션 삭제로 비교적 간단  | 토큰 만료/블랙리스트 처리 필요 |
| 보안 관리        | 세션 저장소 보호 중요   | 토큰 탈취 방지 중요       |

JWT 방식은 서버가 로그인 상태를 직접 저장하지 않아도 되기 때문에 확장성 측면에서 유리하다.
하지만 토큰이 탈취되면 만료 전까지 악용될 수 있으므로 Access Token과 Refresh Token을 나누어 사용한다.

---

## 2-4. Access Token과 Refresh Token

JWT를 사용할 때는 보통 토큰을 2개로 나눈다.

| 토큰            | 수명 | 용도               |
| ------------- | -- | ---------------- |
| Access Token  | 짧음 | 일반 API 요청 인증     |
| Refresh Token | 김  | Access Token 재발급 |

Access Token은 API 요청에 사용되는 짧은 수명의 토큰이다.
Refresh Token은 Access Token이 만료되었을 때 새 Access Token을 발급받기 위해 사용하는 긴 수명의 토큰이다.

이렇게 나누는 이유는 보안과 편의성을 동시에 잡기 위해서이다.

```text
Access Token이 너무 길면 탈취 시 위험함
Access Token이 너무 짧으면 사용자가 자주 로그인해야 함
따라서 Access Token은 짧게, Refresh Token은 길게 사용
```

---

## 2-5. Access / Refresh Token 흐름

```text
1. 사용자가 로그인한다.
2. 서버가 Access Token과 Refresh Token을 발급한다.
3. 클라이언트는 API 요청 시 Access Token을 보낸다.
4. 서버는 Access Token을 검증한다.
5. Access Token이 유효하면 요청을 처리한다.
6. Access Token이 만료되면 401 Unauthorized를 응답한다.
7. 클라이언트는 Refresh Token으로 새 Access Token을 요청한다.
8. 서버는 Refresh Token을 검증하고 새 Access Token을 발급한다.
```

---

## 2-6. JWT 생성 코드 예시

```ts
import jwt from "jsonwebtoken";

export const generateAccessToken = (user: { id: number; email: string }) => {
  return jwt.sign(
    {
      id: user.id,
      email: user.email,
    },
    process.env.JWT_SECRET!,
    {
      expiresIn: "1h",
    }
  );
};

export const generateRefreshToken = (user: { id: number }) => {
  return jwt.sign(
    {
      id: user.id,
    },
    process.env.JWT_SECRET!,
    {
      expiresIn: "14d",
    }
  );
};
```

### 코드 설명

| 코드                       | 설명                  |
| ------------------------ | ------------------- |
| `jwt.sign()`             | JWT를 생성하는 함수        |
| `{ id, email }`          | Payload에 담을 사용자 정보  |
| `process.env.JWT_SECRET` | 서명에 사용할 비밀 키        |
| `expiresIn: "1h"`        | Access Token 유효 기간  |
| `expiresIn: "14d"`       | Refresh Token 유효 기간 |

---

## 2-7. 주의할 점

JWT를 사용할 때 주의할 점은 다음과 같다.

| 주의점                      | 설명                          |
| ------------------------ | --------------------------- |
| Payload에 민감 정보 넣지 않기     | 비밀번호, 주민번호 등은 절대 넣으면 안 됨    |
| JWT_SECRET 안전하게 관리       | `.env`에 저장하고 GitHub에 올리지 않기 |
| Access Token 유효 기간 짧게 설정 | 탈취 시 피해 줄이기                 |
| Refresh Token은 더 안전하게 관리 | DB 저장, 쿠키 보안 옵션 등 고려        |
| 만료 처리 필요                 | 만료된 토큰 요청 시 401 응답 필요       |



---

# 3. Bearer Token

## 3-1. 개념

Bearer Token은 HTTP Authorization 헤더에 토큰을 담아 보내는 인증 방식이다.

형식은 다음과 같다.

```text
Authorization: Bearer <token>
```

여기서 Bearer는 “이 토큰을 가진 사람”이라는 의미로 이해할 수 있다.
즉, 서버는 요청에 포함된 토큰을 검증하고, 유효한 토큰이라면 해당 사용자의 요청으로 처리한다.

---

## 3-2. 왜 필요한가?

JWT를 발급받았다고 해서 서버가 자동으로 로그인 상태를 아는 것은 아니다.
클라이언트가 API 요청을 보낼 때 JWT를 함께 보내야 서버가 사용자를 인증할 수 있다.

이때 가장 일반적으로 사용하는 방식이 Authorization 헤더에 Bearer Token을 담는 방식이다.

```text
클라이언트가 API 요청
    ↓
Authorization 헤더에 Access Token 포함
    ↓
서버가 Access Token 검증
    ↓
사용자 인증 후 요청 처리
```

---

## 3-3. 요청 예시

Postman이나 프론트엔드에서 요청할 때 Header에 다음 값을 추가한다.

```text
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6...
```

프론트엔드 fetch 예시는 다음과 같다.

```js
const response = await fetch("http://localhost:3000/mypage", {
  method: "GET",
  headers: {
    Authorization: `Bearer ${accessToken}`,
  },
});
```

---

## 3-4. JWT Strategy 예시

`passport-jwt`를 사용하면 Authorization 헤더의 Bearer Token을 추출하고 검증할 수 있다.

```ts
import { Strategy as JwtStrategy, ExtractJwt } from "passport-jwt";
import { prisma } from "./db.config.js";

export const jwtStrategy = new JwtStrategy(
  {
    jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
    secretOrKey: process.env.JWT_SECRET!,
  },
  async (payload, done) => {
    try {
      const user = await prisma.user.findFirst({
        where: { id: payload.id },
      });

      if (!user) {
        return done(null, false);
      }

      return done(null, user);
    } catch (err) {
      return done(err, false);
    }
  }
);
```

### 코드 설명

| 코드                                         | 설명                                 |
| ------------------------------------------ | ---------------------------------- |
| `ExtractJwt.fromAuthHeaderAsBearerToken()` | Authorization 헤더에서 Bearer Token 추출 |
| `secretOrKey`                              | JWT 서명 검증에 사용할 비밀 키                |
| `payload.id`                               | JWT Payload에 담긴 사용자 ID             |
| `done(null, user)`                         | 인증 성공                              |
| `done(null, false)`                        | 인증 실패                              |

---

## 3-5. 보호된 라우트 예시

```ts
const isLogin = passport.authenticate("jwt", { session: false });

app.get("/mypage", isLogin, (req, res) => {
  res.status(200).json({
    message: "인증 성공",
    user: req.user,
  });
});
```

위 코드에서 `/mypage`는 JWT가 있어야 접근할 수 있는 보호된 라우트이다.

토큰 없이 요청하면 인증에 실패한다.
올바른 Access Token을 Bearer Token 형식으로 보내면 인증에 성공한다.

---

## 3-6. 토큰이 없는 경우와 있는 경우

| 요청 상태               | 결과               |
| ------------------- | ---------------- |
| Authorization 헤더 없음 | 401 Unauthorized |
| 잘못된 토큰              | 401 Unauthorized |
| 만료된 토큰              | 401 Unauthorized |
| 올바른 Bearer Token    | 요청 성공            |


---

# 4. 세 키워드의 관계 정리

OAuth 2.0, JWT, Bearer Token은 각각 따로 떨어진 개념이 아니라 로그인 흐름 안에서 연결된다.

```text
OAuth 2.0
    ↓
Google 계정으로 사용자 인증
    ↓
우리 서버가 사용자 정보 확보
    ↓
JWT 발급
    ↓
클라이언트가 JWT 저장
    ↓
Bearer Token 형식으로 API 요청
    ↓
서버가 JWT 검증 후 요청 처리
```

| 단계                    | 사용되는 개념      |
| --------------------- | ------------ |
| Google 로그인 화면 이동      | OAuth 2.0    |
| Authorization Code 발급 | OAuth 2.0    |
| Google 사용자 정보 조회      | OAuth 2.0    |
| 우리 서버의 로그인 토큰 발급      | JWT          |
| API 요청 시 토큰 전달        | Bearer Token |
| 서버에서 토큰 검증            | JWT          |

---
