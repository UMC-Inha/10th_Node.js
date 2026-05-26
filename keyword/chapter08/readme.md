- Swagger& OpenAPI
    
    # OpenAPI Specification (OAS)
    
    > Restful API를 기술하는 표준 명세 작성 방식
    > 
    
    서비스에서 제공하는 API의 구조와 기능& 요청/응답 정보를 일관된 형식으로 표현한다
    
    → API가 어떻게 동작하는지를, 사람과 컴퓨터 모두가 이해할 수 있도록 문서화하는 표준
    
    #### 특징
    
    - JSON이나 YAML 형식으로 작성됨
        
        → 코드나 문서 없이도 전체 구조를 이해하기 쉽다
        
    - 개발 언어에 구애받지 않는다
        
        → 다양한 환경에서 동일한 명세를 사용할 수 있다
        
    
    # Swagger
    
    > OAS 기반으로 동작하는 오픈소스 프레임워크
    > 
    
    2011년에 등장했으며, RESTful API 개발에서 중요한 역할을 해왔다
    
    ### OpenAPI와 Swagger의 관계
    
    예전에는 OpenAPI 2.0을 Swagger Specification 또는 Swagger 2.0으로 불렀다
    
    이 때에는 Swagger 자체가 API 명세 표준 역할을 했지만, 이후 OpenAPI로 이름이 변경되어서 현재는 다음과 같다
    
    - OpenAPI → RESTful API 디자인에 대한 정의
    - Swagger → OpenAPI를 활용하는 도구
    
    #### 장점
    
    1. **API 문서 자동 생성**
        
        : 문서를 따로 작성할 필요가 없다
        
    2. **테스트 가능**
        
        : 백엔드 개발 및 디버깅 과정에서 매우 유용하다
        
    3. **협업 효율 증가**
    
    ## Swagger Tool 종류
    
    : 동일한 API 명세를 공유할 수 있다
    
    #### Swagger Codegen
    
    > Swagger로 정의된 대로 클라이언트 • 서버 코드를 생성하는 CLI 툴
    > 
    
    API 명세만으로 다음과 같은 코드를 생성할 수 있다
    
    - Java Client
    - TypeScript Client
    - Node.js Server
    - Spring Server
    
    → 반복적인 코드 작성을 줄여준다
    
    #### Swagger UI
    
    > API 명세서를 HTML 형식으로 확인 가능한 툴
    > 
    
    Swagger에서 가장 많이 사용되는 도구이다
    
    - API 목록 확인
    - 요청 • 응답 구조 확인
    - 브라우저에서 직접 테스트
    
    → 백엔드 프로젝트에서 API 문서를 제공할 때 자주 사용된다
    
    #### Swagger Editor
    
    > 표준에 따른 API 설계서 • 명세서 작성을 위한 에디터
    > 
    
    JSON 또는 YAML 형식으로 API 문서를 작성할 수 있으며, 실시간 미리보기를 지원한다
    
    → API 설계 단계에서 많이 사용된다
    
- TSOA(TypeScript-first OpenAPI) (핵심적인 부분이라 한번 더 넣었어요!)
    
    ## TSOA
    
    **T**ype**S**cript **O**pen **A**PI
    
    > TypeScript 기반의 서버 프레임워크(Express, Koa, Hapi 등 )에서
    > 
    > 
    > API 문서(Swagger/OpenAPI)와 라우트를 자동 생성해주는 프레임워크
    > 
    
    TypeScript 코드에 작성한 타입 정보와 데코레이터를 분석해 API 문서와 서버 라우팅 코드를 자동으로 생성한다
    
    #### 특징
    
    - Swagger(OpenAPI) 문서 자동 생성
    - TypeScript 타입 기반 API 설계
    - Express/Koa/Hapi 지원
    - 라우트 코드 자동 생성
    - 데코레이터 기반 구조
    - 타입 안정성 제공
    - 유지보수 및 협업 효율 향상
    
    ---
    
    ## TSOA 사용 이유
    
    기존 방식에서는 다음 작업을 각각 따로 관리해야 했다
    
    - API 구현 코드 작성
    - Swagger(OpenAPI) 문서 작성
    
    이 과정에서 다음과 같은 문제가 자주 발생
    
    - 실제 API와 문서 내용이 달라지는 문제
    - 문서 수정 누락 가능성
    - API 변경 시 유지보수 비용 증가
    
    TSOA는 이러한 문제를 해결하기 위해 등장
    
    컨트롤러 코드에 타입과 데코레이터만 작성하면
    
    - API 라우트 자동 생성
    - Swagger(OpenAPI) 문서 자동 생성
    
    이 함께 이루어진다
    
    따라서 문서와 실제 코드 간 불일치 문제를 줄일 수 있으며, 유지보수도 훨씬 편해진다
    
    ## 데코레이터
    
    : TSOA는 데코레이터를 이용하여 API 정보를 정의한다.
    
    이를 통해 다음과 같은 정보를 선언할 수 있다.
    
    - API 경로(Route)
    - HTTP Method(GET, POST 등)
    - 요청 Body
    - Query Parameter
    - URL Parameter
    - 응답 타입(Response)
    
    즉, 코드 자체가 API 명세 역할까지 수행
    
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
    
    ---
    
    ## 사용 예시
    
    #### 기본 GET API
    
    ```tsx
    import {Controller,Get,Route,Tags }from"tsoa";
    
    @Route("users")
    @Tags("User")
    exportclassUserControllerextendsController {
    
    @Get("/")
    publicasync getUsers():Promise<string[]> {
    return ["kim","lee"];
      }
    }
    ```
    
    → `GET /users` API가 생성됨
    
    또한 Swagger(OpenAPI) 문서에도 자동으로 반영된다
    
    ---
    
    #### URL Parameter 사용 예시
    
    ```tsx
    import {Controller,Get,Path,Route }from"tsoa";
    
    @Route("users")
    exportclassUserControllerextendsController {
    
    @Get("{userId}")
    publicasync getUser(
    @Path()userId:number,
      ):Promise<string> {
    
    return`userId:${userId}`;
      }
    }
    ```
    
    → `GET /users/1`
    
    `@Path()` 데코레이터를 이용하여 URL 파라미터 값을 받을 수 있다
    
    ---
    
    ## Request Body 사용 예시
    
    ```tsx
    import {Body,Controller,Post,Route }from"tsoa";
    
    interfaceCreateUserRequest {
      name:string;
    }
    
    @Route("users")
    exportclassUserControllerextendsController {
    
    @Post("/")
    publicasync createUser(
    @Body()body:CreateUserRequest,
      ):Promise<string> {
    
    returnbody.name;
      }
    }
    ```
    
    `@Body()` 데코레이터를 사용하면 요청 Body 데이터를 타입 안전하게 받을 수 있다
    
    TypeScript 인터페이스 기반으로 Swagger 문서까지 자동 생성
    
- Type-Driven-Documentation
    
    ## Type-Driven-Documentation
    
    > 타입 정보를 기반으로 API 문서와 구조를 자동 생성하는 방식
    > 
    
    : TypeScript의 타입 정의를 기준으로, 문서와 코드 구조를 함께 관리하는 개발 방식이다
    
    ### 특징
    
    - API 변경사항 자동 반영
        
        → 타입 수정 시 요청/응답 구조와 문서 내용도 함께 변경
        
    - TypeScript 인터페이스/타입 정보 분석
        
        → Swagger 문서 작성, 요청/응답 구조 생성, Validation 정보 생성 등 자동화
        
        예시)
        
        ```tsx
        interface UserResponse {
          id: number;
          name: string;
        }
        ```
        
        위 정보를 기반으로 자동 생성한다
        
    
    ### 장점
    
    기존에는 API 코드 작성, 문서 작성, 요청/응답 타입 정의를 각각 따로 작성해야 했다
    
    → 실제 코드와 문서 내용이 달라지는 문제 발생
    
    Type-Driven-Documentation은 타입 정보를 기준으로 문서를 생성하기 때문에 
    
    - 코드와 문서 동기화
    - 타입 안정성 확보
    - 유지보수 효율 증가 : API 변경 시 수정 범위 ↓
    
    등의 장점이 있다
    
    ### 한계
    
    - TypeScript 의존성이 높음
    - 복잡한 타입 구조에서는 설정 어려움
    - 데코레이터 기반 문법 학습 필요

    # 시니어
    - **2.0**
    
    ## OpenAPI 2.0 (Swagger 2.0)
    
    > Swagger 시절 사용되던 API 명세 버전
    > 
    
    2010년 초 등장했으며, 당시 REST API 문서화 표준 역할을 했다
    
    ### 특징
    
    - JSON/YAML 기반 API 명세 지원
    - Swagger UI와 연동
    - 요청/응답 구조 정의
    - 기본적인 인증 방식 지원
    
    ### 한계
    
    #### 1. Request Body 표현 제한
    
    : Body 데이터를 표현할때 바디 파라미터 하나만 사용할 수 있다
    
    → 파일 업로드나 다양한 컨텐츠 타입 표현이 불편함
    
    #### 2. Content-Type 처리 부족
    
    consumes, produces 키워드를 사용
    
    ```yaml
    consumes:
      - application/json
    ```
    
    위처럼 제한적으로 표현 가능
    
- **3.0**
    
    ## OpenAPI 3.0
    
    > OpenAPI라는 이름으로 변경된 이후 첫 주요 버전
    > 
    
    OpenAPI 2.0의 한계를 크게 개선했다
    
    : 현재까지도 널리 사용되는 버전
    
    ### 특징
    
    #### 1. Request Body 구조 개선
    
    : requestBody 객체가 추가됨
    
    ```yaml
    requestBody:
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/User'
    ```
    
    → 다양한 요청 형식을 유연하게 표현할 수 있다
    
    #### 2. Content-Type 표현 개선
    
    : 기존 consumes, produces 대신
    
    ```yaml
    content:
      application/json:
    ```
    
    위와 같은 구조를 사용한다
    
    → JSON/XML/Multipart 등을 세부적으로 정의할 수 있다
    
    #### 3. Components 도입
    
    : 재사용 가능한 객체를 관리할 수 있음
    
    ```yaml
    components:
      schemas:
    ```
    
    → Schema, Security, Response 등을 중앙 관리할 수 있다
    
    #### 4. 파일 업로드 개선
    
    : Multipart/From-Data 지원이 자연스러움
    
    #### 5. Links 지원
    
    : HATEOAS 스타일 API 표현 가능
    
- **3.1**
    
    ## OpenAPI 3.1
    
    > JSON Schema 표준과 완전히 호환되도록 개선된 버전
    > 
    
    : JSON Schema 공식 표준을 그대로 지원하게 되었다
    
    ### 특징
    
    #### 1. JSON Schema 호환
    
    : 3.0에서는 일부만 지원했는데, 완전히 호환되었다
    
    → nullable 처리 개선, if/then/else 지원, const 지원
    
    #### 2. nullable 제거
    
    3.0 방식
    
    ```yaml
    nullable: true
    ```
    
    3.1 방식
    
    ```yaml
    type:
      - string
      - "null"
    ```
    
    #### 3. 웹 표준 강화
    
    : 웹 표준과의 호환성 증가, Validation 도구 연동 강화
    
    #### 4. examples 개선
    
    : 예제 데이터 표현 방식이 더 유연해짐
    

| 항목 | OpenAPI 2.0 | OpenAPI 3.0 | OpenAPI 3.1 |
| --- | --- | --- | --- |
| 명칭 | Swagger 2.0 | OpenAPI 3.0 | OpenAPI 3.1 |
| 발표 시기 | 2014 | 2017 | 2021 |
| Request Body | 제한적 | requestBody 지원 | 개선 유지 |
| Content-Type | consumes/produces | content 기반 | content 기반 |
| Schema 재사용 | 제한적 | components 지원 | 개선 유지 |
| JSON Schema 호환 | 부분 지원 | 부분 지원 | 완전 호환 |
| nullable 처리 | 미지원 | nullable: true | null 타입 사용 |
| 확장성 | 낮음 | 높음 | 매우 높음 |

- 웹