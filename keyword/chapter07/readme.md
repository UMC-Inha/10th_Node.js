- 미들웨어
    
    ## 미들웨어
    
    > Express에서 요청(Request)과 응답(Response) 사이에서 실행되는 함수
    > 
    
    클라이언트의 요청을 처리하기 전 또는 응답을 보내기 전에 중간에서 동작하며, 요청과 응답 객체를 조작하거나 추가 기능을 수행한다.
    
    `next()` 함수를 호출하여 다음 미들웨어로 제어를 넘김
    
    #### 역할
    
    - 요청 데이터 처리
    - 응답 데이터 가공
    - 에러 처리
    - 인증/인가
    - 로깅
    - 압축 처리
    - CORS 설정
    - …
    
    → 이와 같은 공통 기능을 수행한다
    
    #### 실행 흐름
    
    클라이언트 요청 → 미들웨어 1 → 미들웨어 2 → 라우터(컨트롤러) → 에러 미들웨어 → 클라이언트 응답
    
    #### 예시
    
    - **JSON 파싱 미들웨어**
        
        ```tsx
        app.use(express.json());
        ```
        
        → 클라이언트의 JSON 요청 데이터를 파싱해서 req.body에 저장
        
    - **CORS 미들웨어**
        
        ```tsx
        app.use(cors());
        ```
        
        → 다른 도메인에서 API 요청이 가능하도록 설정
        
    - **압축 미들웨어**
        
        ```tsx
        app.use(
          compression({
            threshold: 512,
          }),
        );
        ```
        
        응답 데이터를 gzip/brotli 방식으로 압축하여 네트워크 사용량을 줄임
        
        `threshold: 512`는 512byte 이하의 작은 응답은 압축하지 않도록 설정한 것
        
    - **에러 처리 미들웨어**
        
        ```tsx
        app.use(errorMiddleware);
        ```
        
        → 애플리케이션에서 발생한 에러를 공통 형식으로 처리
        
    
    #### 특징
    
    - 요청과 응답 사이에서 동작
    - 순서대로 실행됨
    - next( )를 호출해야 다음 미들웨어로 이동함
    - 공통 기능을 재사용 가능하게 함
    
- HTTP 상태 코드
    
    ## HTTP 상태 코드
    
    > 특정 HTTP 요청이 어떻게 처리되었는지 알려주는 응답 코드
    > 
    
    #### 구분
    
    : 맨 앞자리 숫자에 따라 구분된다
    
    |  | 의미 | 설명 |
    | --- | --- | --- |
    | 100번대 | Informational | 처리 중 |
    | 200번대 | Success | 성공 |
    | 300번대 | Redirection | 추가 작업 필요 |
    | 400번대 | Client Error | 클라이언트 오류 |
    | 500번대 | Server Error | 서버 오류 |
    
    ### 1xx : Informational
    
    요청이 수신되어 처리 중
    
    | 코드 | 의미 |
    | --- | --- |
    | 100 Continue | 요청 계속 진행 |
    | 101 Switching Protocols | 프로토콜 전환 |
    | 102 Processing | 처리 중 |
    | 103 Early Hints | 사전 정보 제공 |
    
    ### 3xx : Redirection
    
    요청을 완료하기 위해 추가 동작이 필요
    
    | 코드 | 의미 |
    | --- | --- |
    | 301 Moved Permanently | 영구 이동 |
    | 302 Found | 임시 이동 |
    | 304 Not Modified | 캐시 사용 |
    | … | … |
    | 307 Temporary Redirect | 임시 리다이렉트 |
    | 308 Permanent Redirect | 영구 리다이렉트 |
    
    ### 5xx : Server Error
    
    서버가 정상적인 요청을 처리하지 못함
    
    | 코드 | 의미 |
    | --- | --- |
    | 500 Internal Server Error | 서버 내부 오류 |
    | 501 Not Implemented | 기능 미지원 |
    | 502 Bad Gateway | 게이트웨이 오류 |
    | 503 Service Unavailable | 서버 사용 불가 |
    | 504 Gateway Timeout | 서버 응답 시간 초과 |
    | … | … |
    
    ### 2xx : Success
    
    요청이 정상적으로 처리됨
    
    | 코드 | 의미 |
    | --- | --- |
    | 200 OK | 요청 성공 |
    | 201 Created | 리소스 생성 성공 |
    | 202 Accepted | 요청 접수 완료 |
    | 204 No Content | 성공했지만 반환 데이터 없음 |
    | … | … |
    
    ### 4xx : Client Error
    
    클라이언트의 잘못된 요청으로 인해 서버가 요청을 처리할 수 없음
    
    | 코드 | 의미 |
    | --- | --- |
    | 400 Bad Request | 잘못된 요청 |
    | 401 Unauthorized | 인증 필요 |
    | 403 Forbidden | 접근 금지 |
    | 404 Not Found | 리소스를 찾을 수 없음 |
    | 405 Method Not Allowed | 허용되지 않은 HTTP Method |
    | 409 Conflict | 요청 충돌 |
    | … | … |
    | 429 Too Many Requests | 요청 과다 |
    
    #### 자주 사용하는 상태 코드
    
    - 200 OK
        - 주로 GET 요청 성공 시 사용
    - 201 Created
        - 새로운 리소스 생성에 성공했을 때 사용 (회원가입, 게시글 작성 등)
    - 204 No Content
        - 요청은 성공했지만 반환할 데이터가 없을 때 사용 (삭제 요청 처리 시)
    - 400 Bad Request
        - 클라이언트 요청 값이 잘못되었을 때 사용 (필수값 누락, 잘못된 데이터 형식, 유효성 검사 실패 등)
    - 401 Unauthorized
        - 인증이 필요할 때 사용 (로그인 안 된 사용자, 토근 없음, JWT 만료 등)
    - 403 Forbidden
        - 인증은 되었지만 권한이 없는 경우 사용 (일반 사용자의 관리자 페이지 접근)
    - 404 Not Found
        - 요청한 리소스를 찾을 수 없을 때 - 가장 자주 보이는 상태 코드 중 하나
    - 409 Conflict
        - 현재 서버 상태와 요청이 충돌할 때 사용 (이미 존재하는 이메일로 회원가입 등)
    - 500 Internal Server Error
        - 서버 내부 오류 발생 시 사용 - 서버 개발 중 가장 자주 보게 되는 상태코드 중 하나
    
    #### 효과적인 설계 방법
    
    1. HTTP 상태코드는 표준에 맞게 사용하기
        
        → 의미에 맞게 정확히 사용해야 한다
        
    2. 에러 코드는 서비스별로 일관성 있게 정리하기
        
        → 같은 상황에서는 항상 동일한 상태코드와 응답 형식을 사용해야 한다 : 응답 구조 통일 시 협업하기 쉬워짐
        
        ```json
        {
          "success": false,
          "message": "존재하지 않는 사용자입니다."
        }
        ```
        
    3. 응답 포맷 통일하기
        
        → 성공/실패 여부와 데이터를 일정한 형식으로 반환하는 것이 좋다
        
        ```json
        {
          "success": true,
          "data": {
            "userId": 1,
            "name": "홍길동"
          }
        }
        ```
        
    
- 에러 핸들링(Error Handling)
    
    ## 에러 핸들링
    
    > 웹 애플리케이션 개발 중 오류가 발생할 때, 올바른 응답을 보내는 것
    > 
    
    : 에러 메시지를 표시하거나 사용자를 적절한 페이지로 리다이렉트하는 것을 말한다
    
    오류 상황이 발생하더라도 프로그램이 안정적으로 동작하도록 처리하는 과정이며, 서비스의 안정성과 사용자 경험을 향상시키는데 매우 중요하다
    
    주로
    
    - 에러 메시지 출력
    - 로그인 페이지로 리다이렉트
    - 관리자에게 오류 로그 전송
    - 사용자에게 안내 메시지 제공
    
    등의 방식으로 처리할 수 있음
    
    **에러 처리는 크게 에러 핸들링, 예외 핸들링 두 가지로 나뉜다**
    
    #### 에러 핸들링
    
    언어의 문법 오류나 통신 장애 등으로 발생하는, 컴퓨터가 코드를 실행하는 과정 자체에서 발생하는 에러 상황에 대한 처리 
    
    **특징**
    
    - 프로그램 실행 자체가 실패할 수 있음
    - 시스템 환경이나 외부 요인의 영향을 많이 받음
    - 개발자의 코드 실수로 발생하기도
    
    **발생 원인**
    
    - 잘못된 문법 작성
    - 서버 다운
    - DB 연결 실패
    - 네트워크 통신 장애
    - 파일 접근 실패
    - …
    
    #### 예외 핸들링
    
    : 정상적인 서비스 진행을 방해하는 코드가 실행될 경우를 막기 위해 개발자가 의도적으로 발생시키는 예외 상황에 대한 처리
    
    **특징**
    
    - 프로그램 자체는 정상 실행 가능
    - 서비스 정책을 위한 검증 과정
    
    **예시**
    
    - 회원가입 이메일에는 ‘@’가 들어가야 한다
    - 비밀번호는 10자 이상이어야 한다
    - 상품 재고보다 많은 수량을 주문할 수 없다
    - 로그인 없이는 게시글 작성이 불가하다
    
- Tsoa
    
    ## TSOA
    
    **T**ype**S**cript **O**pen **A**PI
    
    > TypeScript 기반의 Express, Koa, Hapi 등의 서버 프레임워크에서 API 문서(Swagger/OpenAPI)와 라우트를 자동 생성해주는 프레임워크
    > 
    
    → TS 코드에 작성한 타입과 데코레이터를 기반으로, API 명세서와 서버 라우팅 코드를 자동으로 만들어주는 도구
    
    #### 특징
    
    - Swagger(OpenAPI) 문서 자동 생성
    - TypeScript 타입 기반 API 설계
    - Express/Koa/Hapi 지원
    - 라우트 자동 생성
    - 데코레이터 기반 구조
    - 타입 안정성 제공
    - 유지보수성 향상
    
    ---
    
    #### 사용 이유
    
    기존 방식에서는 API 코드 작성과 Swagger 문서 작성을 따로 해야 했는데
    
    - 실제 API와 문서 내용 불일치
    - 문서 수정 시 누락 발생 가능
    - 유지보수 까다로움
    
    등의 문제가 있었다
    
     TSOA를 사용하면 컨트롤러 코드만 작성해도 API 문서와 라우트가 자동으로 생성된다
    
    : 문서와 실제 코드의 불일치 문제를 줄일 수 있음
    
    #### 데코레이터
    
    : TSOA는 데코레이터를 이용해 API 정보를 정의한다
    
    이를 통해 API 경로, HTTP 메서드, 요청 Body, 쿼리 파라미터, 응답 타입 등의 정보를 선언할 수 있음
    
    | 데코레이터 | 설명 |
    | --- | --- |
    | @Route() | 기본 API 경로 지정 |
    | @Get()/@Post()/@Put()/@Delete() | GET/POST/PUT/DELETE 요청 정리 |
    | @Body() | 요청 Body 값 받기 |
    | @Query() | 쿼리 파라미터 받기 |
    | @Patch() | URL 파라미터 받기 |
    | @Tags() | Swagger 그룹 지정 |
    | @SuccessResponse() | 성공 응답 정의 |
    | … | … |
    
    **사용 예시**
    
    Controller
    
    ```tsx
    import { Controller, Get, Route, Tags } from "tsoa";
    
    @Route("users")
    @Tags("User")
    export class UserController extends Controller {
    
      @Get("/")
      public async getUsers(): Promise<string[]> {
        return ["kim", "lee"];
      }
    }
    ```
    
    → GET /users API를 생성함
    
    URL 파라미터 예시
    
    ```tsx
    import { Controller, Get, Path, Route } from "tsoa";
    
    @Route("users")
    export class UserController extends Controller {
    
      @Get("{userId}")
      public async getUser(
        @Path() userId: number,
      ): Promise<string> {
    
        return `userId: ${userId}`;
      }
    }
    ```
    
    → GET /users/1
    
    @Path를 이용해 URL 파라미터를 받을 수 있다
    
    Body 파라미터 예시
    
    ```tsx
    import { Body, Controller, Post, Route } from "tsoa";
    
    interface CreateUserRequest {
      name: string;
    }
    
    @Route("users")
    export class UserController extends Controller {
    
      @Post("/")
      public async createUser(
        @Body() body: CreateUserRequest,
      ): Promise<string> {
    
        return body.name;
      }
    }
    ```
    
    @Body 를 이용해 요청 바디 데이터를 받을 수 있다


서버에서 내려주는 응답의 크기가 크면 네트워크 대역폭을 많이 사용하고, 전송 속도 역시 느려질 수 있습니다. 이를 위해 일반적으로는 gzip 등의 방식의 압축을 통해 응답을 압축하여 제공하기도 하는데요, gzip, brotli 등의 압축 방식들에 대해 정리

# Gzip

> 번들링 이후 단계에서 만들어진 파일들을 압축하는 알고리즘
> 

서버에서 클라이언트로 데이터를 전송할 때 데이터의 크기를 줄이며 네트워크 전송 시간을 단축하는데 목적을 두고 있음

#### 특징

- 웹에서 파일을 압축/전송하기 위한 파일 포맷이자 소프트웨어
- 1992년 GNU 프로젝트에서 개발됨
- LZ77 알고리즘과 Deflate 방식을 사용
- 대부분의 브라우저와 서버에서 지원
- 무손실 압축이기 때문에, 압축/해제 과정에서 데이터 손실이 없음
    - 데이터를 압축하고 해제하는 과정에서 데이터의 손실이 없기 때문에 사용자는 빠른 경험을 받을 수 있음

# Brotli

> Google에서 개발한 압축 알고리즘
> 

Gzip보다 더 우수한 압축률을 제공하며, 특히 웹 리소스 압축에 최적화되어 있음

#### 특징

- 구글에서 개발
- 압축 알고리즘: LZ77 + Huffman 부호화 + 사전(dictionary)
    - 사전 기반 압축을 사용하기 때문에 웹에서 자주 사용되는 문자열 패턴을 더 효율적으로 압출할 수 있다
- 텍스트 기반 파일에서 특히 효율적

|  | Gzip | Brotli |
| --- | --- | --- |
| 출시 | 1992년 | 2015년 |
| 압축 알고리즘 | LZ77 + Huffman | LZ77 + Huffman + 사전 압축 |
| 압축 속도 | 빠름 | 느림 |
| 압축 해제 속도 | 빠름 | 빠름 |
| 파일 확장자 | .gz | .br |
| 표준 지원 | 모든 브라우저 | HTTP/2, HTTP/3, 대부분 브라우저 |
| 최대 압축 레벨 | 9 | 11 |

---

#### 장점

- 네트워크 트래픽 감소
- 페이지 로딩 속도 향상
- 대역폭 절약
- API 응답 속도 개선

#### 단점

- CPU 사용량 증가
- 작은 파일은 압출 효율이 낮음
- 이미지, 영상 같이 이미 압축된 파일은 효과가 적음

### Brotli가 더 좋은데 Gzip도 같이 사용하는 이유

1. **브라우저 호환성**
    
    : 구형 브라우저에서는 Brotli를 지원하지 않을 수도 있음
    
2. **압축 속도 차이**
    
    : Brotli가 압축률은 높지만, 압축 속도는 Gzip이 더 빠르다
    
    →  실시간 API 응답, 서버 부하가 높은 환경에서는 Gzip이 더 효율적일 수 있다
    
3. 서버 부하 문제
    
    : 압축은 CPU 자원을 사용하는데, Brotli의 높은 압축 레벨은 트래픽이 많은 서버에서는 부담이 될 수 있다