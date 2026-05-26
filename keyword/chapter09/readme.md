# OAuth 2.0

> Open Authorization
> 
> 
> 사용자의 로그인 정보(로그인 자격 증명, 패스워드 등)를 노출하지 않고, 다른 애플리케이션이 제한된 권한으로 사용자 정보에 접그할 수 있도록 하는 개방형 표준 권한 부여 프로토콜
> 

: 신뢰할 수 있는 외부 어플리케이션에 정보를 입력해 해당 어플리케이션이 인증 과정을 처리해주는 방식

OAuth 사용 전에는 인증 방식의 표준이 없었기 떄문에 ID-PW를 사용하는 기본 인증을 사용

→ 보안 취약!

: 제각각인 인증 방식을 표준화해서 이를 사용하는 어플리케이션끼리는 별도의 인증 없이 통합해 사용할 수 있게 되었다

---

### 역할

- **리소스 소유자 - Resource Owner**
    
    : 보호되는 리소스에 대한 접근 권한을 부여하는 엔티티
    
    클라리언트를 인증하는 역할 수행
    
- **클라이언트 - Client**
    
    : 리소스에 접근하려는 third-party 서비스
    
    Google 로그인, 카카오 로그인 등 기능을 제공하는 서비스
    
- **권한 서버 - Authorization Server**
    
    : 클라이언트가 리소스 소유자의 권한을 얻을 수 있도록 도와주는 서버
    
    사용자 인증, 권한 부여 및 토큰 발급 관리
    
- **리소스 서버 - Resource Server**
    
    : 보호되는 리소스를 호스팅하는 서버
    
    클라이언트에게 리소스에 접근할 권한 부여와 실제 데이터 제공
    

### 인증 과정

<img width="696" height="591" alt="Image" src="https://github.com/user-attachments/assets/ddc97fff-31ab-45b5-b173-1ff078ac643b" />

1. 클라이언트가 사용자에게 로그인 요청
2. 사용자가 권한 서버에서 로그인 수행
3. 권한 서버가 사용자 인증 완료 후 Authorization Code 발급
4. 클라이언트가 Authorization Code로 Access Token 요청
5. 권한 서버가 Access Token 발급
6. 클라이언트가 Access Token을 이용해 리소스 서버에 사용자 정보 요청
7. 리소스 서버가 사용자 정보 제공

#### Access Token, Refresh Token

**Access Token**

: 리소스 서버에 접근할 때 사용하는 토큰

유효 시간이 짧으며, 사용자 정보 요청 시 사용된다

**Refresh Token**

: Access Token 만료 시 재발급받기 위한 토큰

유효시간이 비교적 길다

### 장점

- 사용자 정보 노출 위험 감소
- 통합 로그인 기능 제공 가능
- 서비스 간 인증 표준화
- 모바일/웹 환경에서 활용하기 쉽다

---

## 1.0과의 차이점

- **토큰 갱신**
    
    : 1.0에서는 토큰을 갱신하는 것이 어려웠다 (요청 토큰(Request Token)과 접근 토큰(Access Token) 사용)
    
- **인증 방식**
    
    : 1.0에서는 서명된 요청을 이용해 사용자 인증을 하지만, 2.0에서는 보안 토큰을 사용해 인증한다
    
- **범용성**
    
    : 2.0이 더 범용적이다. 모바일 기기와 같은 다양한 플랫폼에서 사용할 수 있다. 1.0은 웹 기반 어플리케이션에서 주로 사용된다
    
- **승인 방식**
    
    : 1.0은 2단계 승인 절차를 사용하지만, 2.0은 보다 단순한 승인 절차 사용
    

- JWT
    
    # JWT
    
    > JSON Web Token
    > 
    > 
    > 인증에 필요한 정보들을 암호화시킨 JSON 토큰
    > 
    
    : JSON 객체를 사용해 사용자 정보를 안전하게 전달하기 위한 토큰 기반 인증 방식
    
    로그인 이후 서버가 발급한 토큰을 HTTP 헤더 등에 포함해 사용자 인증을 처리하는 방식이다
    
    ---
    
    ### 특징
    
    - 토큰 기반 인증 방식 사용
    - Stateless 방식 지원
    - 서버가 세션 정보를 저장하지 않음
    - 디지털 서명을 통해 위변조 검증 가능
    
    ## 구조
    
    JWT는 `.` 을 기준으로 세 부분으로 구성
    
    `Header.Payload.Signature`
    
    ### Header
    
    : 토큰의 타입과 암호화 알고리즘 정보를 저장
    
    예시:
    
    ```
    {
      "alg":"HS256",
      "typ":"JWT"
    }
    ```
    
    ### Payload
    
    : 사용자 정보와 권한 정보 저장
    
    이를 Claim이라고 부름
    
    예시:
    
    ```
    {
      "userId":1,
      "role":"USER"
    }
    ```
    
    Claim 종류
    
    - Registered Claim
    : 미리 정의된 정보 (`iss`, `exp`, `sub` 등)
    - Public Claim
    : 사용자 정의 데이터
    - Private Claim
    : 서버와 클라이언트 간 약속된 정보
    
    ### Signature
    
    : Header와 Payload를 비밀키로 암호화한 값
    
    토큰 변조 여부를 검증하는 역할 수행
    
    ---
    
    ### 인증 과정
    
    1. 사용자가 로그인 요청
    2. 서버가 사용자 정보 검증
    3. 서버가 JWT 발급
    4. 클라이언트가 JWT 저장
    5. 요청 시 Authorization Header에 JWT 포함
    6. 서버가 JWT 검증 후 사용자 인증 처리
    
    ---
    
    ### 장점
    
    - 서버가 세션을 저장하지 않아도 됨
    - 확장성과 성능에 유리
    - 모바일/웹 환경에서 사용하기 편리
    - 다양한 서버 간 인증 공유 가능
    - 로그인 상태 유지에 효율적
    
    ## Session 방식과 JWT 방식 차이
    
    | Session | JWT |
    | --- | --- |
    | 서버에 세션 저장 | 클라이언트가 토큰 저장 |
    | Stateful 방식 | Stateless 방식 |
    | 서버 메모리 사용 | 서버 부담 감소 |
    | 서버 확장성 제한 | 확장성 우수 |
    
- Bearer Token
    
    # Bearer Token
    
    > HTTP 인증 방식 중 하나로, 토큰을 소유한 사용자에게 접근 권한을 부여하는 인증 방식
    > 
    
    : 일반적으로 JWT나 OAuth의 Access Token을 Authorization Header에 포함하여 사용
    
    서버는 전달받은 토큰을 검증하여 사용자의 인증 및 권한을 확인
    
    ---
    
    ### 특징
    
    - 토큰 기반 인증 방식 사용
    - 주로 JWT 또는 OAuth Access Token과 함께 사용
    - HTTP Authorization Header 사용
    - Stateless 인증 방식에 적합
    - 모바일 및 REST API 환경에서 많이 사용
    
    ### 구조
    
    Bearer Token은 다음과 같은 형식으로 전달된다.
    
    ```
    Authorization: Bearer <ACCESS_TOKEN>
    ```
    
    - `Authorization`
        
        : 인증 정보를 전달하는 HTTP 헤더
        
    - `Bearer`
        
        : “토큰 소유자(Bearer)에게 권한 부여”를 의미하는 인증 타입
        
    - `<ACCESS_TOKEN>`
        
        : 서버가 발급한 실제 인증 토큰
        
    
    ### 인증 과정
    
    1. 사용자가 로그인 요청
    2. 서버가 사용자 인증 수행
    3. 서버가 Access Token 발급
    4. 클라이언트가 토큰 저장
    5. 요청 시 Authorization Header에 Bearer Token 포함
    6. 서버가 토큰 검증 후 사용자 인증 처리
    
    ### 장점
    
    - 세션 저장 없이 인증 가능
    - 서버 확장성에 유리
    - REST API와 잘 어울림
    - 모바일/SPA 환경에서 사용하기 편리
    - 다양한 인증 시스템과 연동 가능
    
    ---
    
    ## JWT와의 관계
    
    Bearer Token은 “전달 방식”이고, JWT는 “토큰 형식”임
    
    즉:
    
    - Bearer Token → Authorization Header 사용 방식
    - JWT → 토큰 내부 데이터 구조
    
    따라서 JWT를 Bearer 방식으로 전달하는 경우가 많다
    
    예시:
    
    ```
    Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
    ```
    
    ---
    
    ## OAuth와의 관계
    
    OAuth 2.0 에서는 Access Token을 사용할 때 Bearer 방식을 주로 사용한다
    
    예:
    
    - Google OAuth 로그인
    - 카카오 로그인
    - GitHub OAuth 인증
    
    등에서 발급받은 Access Token을 Bearer Token 형태로 API 요청에 포함한다