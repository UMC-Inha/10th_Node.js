- 환경 변수
    
    ### **환경 변수**
    
    : 코드 밖에서 설정하는 설정값
    
    > 
    > 
    > 
    > 코드에 데이터베이스 비밀번호나 API 키와 같은 정보를 올려 버리면,
    > 
    > →  깃허브에 업로드될 경우 보안 취약
    > 
    > → 개발/테스트/운영 환경마다 값이 달라질 때마다 코드를 계속 수정해야 함
    > 
    > → 환경별 설정 관리 어려움
    > 
    
    Node.js + Typescript 환경에서는 일반적으로 .env 파일을 사용해서 환경 변수를 관리함
    
    - DB 연결 정보
    - API 키
    - JWT secret
    - 포트 번호
    - 외부 서비스 URL
    
    와 같은 정보들을 저장한다
    
    예시) .env
    
    ```tsx
    DB_HOST=localhost
    DB_USER=root
    DB_PASSWORD=1234
    JWT_SECRET=mysecret
    PORT=3000
    ```
    
    - **dotenv** : 환경 변수 로드 라이브러리
        
        dotenv의 .config() 함수를 사용하면 process.env 객체를 통해 .env 파일에 접근할 수 있음
        
        ```tsx
        import dotenv from "dotenv";
        
        dotenv.config();
        ```
        
        이 코드를 실행하면 .env에 정의된 값들이 process.env에 저장된다
        
        환경변수 사용법
        
        ```tsx
        const port = process.env.PORT;
        const dbPassword = process.env.DB_PASSWORD;
        ```
        
        Typescript에서는 process.env 값이 기본적으로 string | undefined 이다
        
        → 바로 사용하면 에러 발생할 수 있음
        
        ```tsx
        const port: number = process.env.PORT; // 에러
        
        // 타입 단언
        const port = process.env.PORT as string; 
        // 숫자 변환
        const port = Number(process.env.PORT); 
        
        // 환경 변수 타입 정의
        declare namespace NodeJS {
          interface ProcessEnv {
            PORT: string;
            DB_PASSWORD: string;
          }
        }
        ```
        
- CORS
    
    > 
    > 
    > 
    > **C**ross-**O**rigin **R**esource **S**haring : 교차 출처 리소스 공유
    > 
    > : 서로 다른 출처 간의 리소스 요청을 허용하기 위한 브라우저 보안 정책
    > 
    
    예시) 
    
    - 브라우저에서 동영상 플랫폼의 동영상을 가져올때
    - 국가 날씨 데이터베이스의 날씨를 표현하는 앱
    
    - **Origin(출처) 구성 요소**
        - 프로토콜 (http, https)
        - 도메인 (example.com)
        - 포트 (3000 등)
    
    → 셋 중 하나라도 다르면 다른 출처이다
    
    ```
    https://www.myshop.com/example/
    
    https://www.myshop.com/example2/ <- 같은 출처 (경로는 출처에 포함되지 않음)
    http://www.myshop.com/example/ <- 다른 출처 (프로토콜)
    https://en.myshop.com/example/ <- 다른 출처 (도메인)
    https://www.myshop.com:8080/example/ <- 다른 출처 (포트)
    ```
    
    - **SOP** (**S**ame-**O**rigin **P**olicy : 동일 출처 정책)
        
        : 서로 다른 출처 간의 요청/응답을 제한하는 정책 (브라우저 기본 보안 정책)
        
        - 사용자 인증 정보 탈취 방지
        - 악성 사이트 데이터 접근 차단
        
        → 안전하지만, SOP 정책만 사용하면 실제 서비스에서는 제한적임
        
    
    - **CORS 정책 생긴 이유**
        
        이전에는 프론트엔드 + 백엔드가 같은 서버에서 동작했기 때문에
        
        다른 출처로부터의 요청 : 의심스러움
        
        요즘에는 클라이언트와 API가 다른 도메인/포트에서 동작하는 경우가 많다
        
        이 경우 SOP 정책은 요청/응답을 차단해버리기 때문에 COPS 정책이 필요하게 되었다
        
    - **동작 방식**
        1. 브라우저가 요청
        2. 서버가 응답 헤더에 허용 정보를 포함
        3. 브라우저가 해당 정보를 보고 허용/차단 결정
        
        - 응답 헤더
            - Access-Control-Allow-Origin
            
            ```
            Access-Control-Allow-Origin: http://localhost:3000 <- 이 출처만 허용
            Access-Control-Allow-Origin: * <- 모든 출처 허용
            ```
            
            * 로 설정하면 보안이 취약해지기 때문에 허용할 출처를 적는 방식이 더 좋다
            
            - Preflight Request : 사전 요청
                
                특정 요청은 바로 보내지 않고 먼저 확인한다
                
            
            ```
            OPTIONS /api
            
            서버 응답
            Access-Control-Allow-Methods: GET, POST
            Access-Control-Allow-Headers: Content-Type
            
            요청 해도 되는지 물어보는 과정이다
            ```
            
    
    - **CORS 에러**
        
        : 서버가 허용하지 않은 출처일 때 브라우저가 차단해서 에러가 나는 것
        
        해결
        
        - 서버에서 헤더 설정
        - 프록시 서버 사용
            
            애플리케이션이 리소스를 직접 사용하지 않고, 프록시 서버를 이용해 리소스로의 요청을 전달하는 방법 : 프록시 서버를 통해 요청을 우회하는 방법이다
            
            → 브라우저 입장에서는 같은 출처로 보인다
            
        
    
- DB Connection, DB Connection Pool
    
    ### **DB Connection**
    
    > 애플리케이션과 데이터베이스 서버 간의 통신 열결
    > 
    
    아래 정보가 필요함
    
    - DB 주소 (host)
    - 포트
    - 사용자 계정
    - 비밀번호
    - DB 이름
    
    → 이 정보들을 기반으로 COnnectuon 객체가 생성되며, 이를 통해 SQL 요청을 보내고 결과를 받아 온다
    
    **문제점**
    
    Connection을 매번 생성/해제하게 되면 시간 소요(생성 과정), 서버 부하 증가, 응답 속도 저하 등의 문제가 발생한다
    
    ### **DB Connection Pool**
    
    > Connection 개체의 반복적인 생성과 해제를 줄이고 효율적인 DB 연결을 위한 재사용 구조
    > 
    
    1. 애플리케이션 시작 시에 미리 일정 수의 Connection 개체 생성
    2. Pool에 보관
    3. 이후 요청이 오면  Pool에서 Connection 개체 꺼내서 사용
    4. 작업 종료 시 Pool에 반환
    
    **장점**
    
    1. 성능 향상
        
        Pool에 연결을 유지하고, 요청 발생 시 재사용
        
        → 빠른 응답, 에플리케이션 성능 즐가
        
    2. 자원 관리
        
        연결 생성/유지 시 필요한 자원을 최적화
        
        : 불필요한 연결 X, 연결 재사용 
        
        → 메모리, CPU의 효율적인 관리 가능
        
    3. 동시성 관리
        
        동시에 여러 연결을 처리할 수 있는 연결 제공 
        
        → 사용자들이 동시에 접속해도 안정적으로 처리할 수 있음
        
    4. 과부하 방지
        
        연결의 개수를 제한하고, 개수를 초과하는 요청이 들어올 시 대기하게 만듦
        
        → 서버의 과부하 방지
        
    5. 커넥션 오버헤드 감소
        
        반복적인 DB 연결/해제 작업에 따른 추가적인 비용과 부하가 감소함
        
    
    **단점**
    
    1. 메모리 사용 증가
        
        Connection을 미리 생성
        
        → 리소스 점유
        
    2. 설정/관리 복잡
        
        다양한 환경에서 최적의 성능을 발휘하기 위해서는 관리와 모니터링 필요, 초기 설정에 시간을 써야 함
        
    3. 커넥션 누수
        
        애플리케이션게서 올바르지 않은 연결을 반환하거나 예외 발생 시 반환되지 않으면 Pool 고갈됨
        
        → DB 연결 불가능 사태가 발생한다
        
    
    - **Pool 크기**
        - 클때
            
            DB 서버 과부하
            
            컨텍스트 스위칭 증가
            
            → 오히려 성능 저하 가능
            
        - 작을때
            
            요청 대기 시간 증가
            
            응답 속도가 느려짐
            
        
        커넥션 풀을 크게 설정했을 때와 작게 설정했을 때의 장단점이 다르므로, 
        
        최적의 성능을 위해서는 커넥션 풀의 크기를 적절하게 조절해야 함
        
    
- 비동기 (async, await)
    
    ### **비동기?**
    
    > 
    > 
    > 
    > Asynchronous, 
    > 
    > 작업 실행 시, 특정 코드의 로직이 완료되지 않았더라도 다음 코드를 먼저 실행하는 방식
    > 
    
    → 시간이 오래 걸리는 작업을 기다리지 않고 다른 작업을 먼저 처리할 수 있음
    
    JavaScript는 싱글 스레드 기반이기 때문에 한 번에 하나의 작업만 처리 가능
    
    : 모든 작업을 동기적으로 처리하면, 오래 걸리는 작업 때문에 전체 프로그램이 멈춤
    
    → 비동기 처리로 >기다리는 시간<을 낭비하지 않도록 함
    
    **Promise** 
    
    : 비동기 함수의 결과를 담고 있는 객체
    
    상태
    
    - 대기(Pending): 비동기 함수 시작 전
    - 성공(Fulfilled): 비동기 함수가 성공적으로 완료
    - 실패(Rejected): 비동기 함수가 실패
    
    대기에서 상태가 바뀔때 then(), catch()를 사용해서 각각 성공/실패 상태의 Promise를 처리할 수 있음
    
    ```tsx
    fetch("/api")
      .then(res => res.json()) // 성공 처리
      .catch(err => console.error(err)); // 실패 처리
    ```
    
    문제점
    
    :  then() 체인을 길게 이어나가면 코드 가독성 떨어지고, 에러가 어디서 났는지 확인이 어려움
    
    ## **async / await**
    
    : ES2017부터 도입된 JavaScript의 비동기 처리 방식 중 하나이며, 
    
      비동기 코드를 동기 코드처럼 작성할 수 있어서 가독성이 좋아지고 에러 처리가 간단해짐
    
    #### **async**
    
    : 함수의 앞에 붙여서, 해당 함수가 Promise를 반환하는 비동기 함수임을 나타낸다
    
    ```tsx
    async function example() {
      return 1;
    }
    ```
    
    async 함수는 항상 Promise 객체를 반환
    
    → 반환 값이 Promise 생성 함수가 아니어도, 반환되는 값을 Promise 객체에 담음
    
    #### **await**
    
    : Promise가 처리될 때까지 (비동기 함수의 실행 결과를)  기다리는 키워드
    
      Promise 객체가 완료될때까지 함수 내부 코드 실행을 일시중지 (이벤트 루프는 계속 동작함)
    
    async 함수 안에서만 사용할 수 있는 문법
    
    ```tsx
    async function test() {
      console.log("1");
    
      const res = await fetch("/api");
    
      console.log("2");
    }
    ```
    
    “1” 출력 → fetch 요청 보냄 → 응답 기다림 → 응답 도착 후 “2” 출력
    
    fetch에서 네트워크 에러 발생 시 await 이후의 코드는 실행되지 않으며 catch 블록으로 제어가 넘어가서 에러를 처리할 수 있다
    
    - **에러 처리 : try-catch**
        
        
- try/catch/finally
    
    : 예외 처리
    
    ### **try…catch 문**
    
    실행할 코드블럭을 표시하고, 예외가 발생할 경우의 응답을 지정함
    
    **기본 구조**
    
    ```tsx
    try {
      tryStatements
    } catch (e) {
      catchStatements
    } finally {
      finallyStatements
    }
    ```
    
    - **tryStatements** : try 블록에서 실행할 구문
    - **catchStatements** : try 블록에서 예외 발생 시 실행할 구문
    - **e** (exeption var) : 발생한 예외 객체를 가지는 식별자
        
        try-block에서 예외가 발생하면 e가 예외 값을 가짐 → 이 식별자를 이용해서, 발생한 예외에 대한 정보를 얻을 수 있음 (에러 메시지, 스택 정보)
        
    - **finallyStatements** : 구문에서 제어 흐름이 빠져나가기 전에 실행되는 구문. 예외 여부와 관계없이 항상 실행
    
    ### catch 활용 방식
    
    1. **무조건적 catch 블록**
        
        try-block에서 예외가 발생하면 catch-block이 실행됨
        
        ```jsx
        try {
            ...
        }
        catch (e) {
            ...
        }
        ```
        
    2. **조건적 catch 블록**
        
        if…else문과 결합해서 생성
        
        ```jsx
        try {
            ...
        } catch (e) {
            if (...) {
            
            } else if (...) {
            
            } else {
                // 지정되지 않은 예외를 처리하기 위한 상태
            }
        }
        ```
        
    3. **중첩 catch 블록**
        
        ```jsx
        try {
            try {
                ...
            } 
            catch {
                ...
            }
            finally {
                ...
            }
        } 
        catch(e) {
            ...
        }
        ```
        
        : 내부 블록에서 예외를 포착한다면, 외부 catch 블록은 작동하지 않음
        
    
    ### finally 블록
    
    : 예외 발생 여부와 상관없이 무조건 실행되는 블록
    
    try 블록 → catch 블록 → return, throw, break, continue 실행 → finally 블록
    
    목적 : 리소스 정리, DB 연결 반환, 파일 닫기 등
    
    **finally 블록에서의 return**
    
    → finally 블록에서 return을 하면 try/catch의 return을 덮어 버린다
    
    ```tsx
    function test1() {
      try {
        return 1;
      } finally {
        return 2;
      }
    }
    
    console.log(test1()); // 2
    ```
    
- Interface(인터페이스)
    
    ## Interface
    
    > 객체의 구조를 정의하기 위한 타입
    > 
    
    → 일반적으로 일관된 타입을 위해 사용됨 : 변수, 함수, 클래스에 사용 가능
    
    특징
    
    - 값이 아닌 형태만 정의
    - 타입 체크 시 사용됨
    - 코드의 일관성. 안정성 유지
    
    #### 변수의 타입으로 사용
    
    인터페이스를 타입으로 선언한 변수는 해당 인터페이스를 준수해야 한다
    
    ```tsx
    // 인터페이스 정의
    interface Todo {
    	id: number;
      content: string;
      completed: boolean;
    }
    
    let study: Todo;
    study = {
    	id: 1, 
    	content: typescript, 
    	completed: false
    };
    ```
    
    #### 함수의 타입으로 사용
    
    인터페이스를 타입으로 선언한 함수는 선언된 파라미터와 타입을 준수해야 한다
    
    ```tsx
    interface Func {
        (num: number): number;
    }
    
    const SquareFunc: Func = function (num: number) {
        return num * num;
    }
    ```
    
    #### 하이브리드 타입
    
    함수이자 객체인 하이브리드 타입을 정의할 수 있다
    
    ```tsx
    interface Func2 {
    	 (a: number): string;   // 함수
    	 b: boolean;            // 객체
    }
    	 
    const func: Func2 = (a) => "hello";
    func.b = true;
    ```
    
    #### 클래스에서 사용
    
    클래스 선언 시 implements의 뒤에 인터페이스를 선언하면 해당 클래스는 지정된 인터페이스를 반드시 구현해야 한다
    
    ```tsx
    interface Animal {
      name: string;
      move(): void;
    }
    
    class Dog implements Animal {
      name: string;
    
      constructor(name: string) {
        this.name = name;
      }
    
      move() {
        console.log("move");
      }
    }
    ```
    
    인터페이스는 클래스와 유사하나(프로퍼티와 메소드 보유), 
    
    직접 인스턴스를 생성할 수 없다는 점이 다르다
    
    |  | Interface | Class |
    | --- | --- | --- |
    | 역할 | 구조 정의 | 실제 구현 |
    | 인스턴스 생성 | 불가능 | 가능 |
    | 구현 포함 | 불가능. 형태만 정의 | 가능 |
    
    #### 선택적 프로퍼티 & 읽기 전용 프로퍼티
    
    **선택적**
    
    ```tsx
    interface User {
      name: string;
      age?: number;    // 선택적 속성 (없어도 됨)
    }
    ```
    
    → 인터페이스에 속하지 않는 프로퍼티의 사용을 방지하며 사용 가능한 속성을 기술
    
    **읽기 전용**
    
    ```tsx
    interface User {
      readonly id: number;
    }
    ```
    
    → 해당 프로퍼티는 읽기 전용이 되어서 한번 값이 설정되면 변경 불가
    
    #### 인터페이스 확장 & 병합
    
    **확장**
    
    ```tsx
    interface Person {
      name: string;
    }
    
    interface Student extends Person {
      grade: number;
    }
    ```
    
    ㄴ 확장 : 코드 재사용 + 구조 확장이 가능하다
    
    - **확장 시 프로퍼티 재정의**
        
        확장 시 기존 프로퍼티 타입을 재정의 할 수 있지만, 기존 타입보다 좁은 타입으로만 재정의 가능
        
        예시)
        
        ```tsx
        interface Animal {
        	 name: string;
        	 color: string;
        	}
        
        interface Dog extends Animal {
        	 name: "leo";              // "leo" 로 타입 고정
        	 breed: string
        	}
        	
        	
        interface Dog extends Animal {
        	 name: number;             // 오류 발생
        	}
        ```
        
    - **다중 확장**
        
        여러 개의 인터페이스 동시 확장 가능
        
        예시)
        
        ```tsx
        interface DogCat extends Dog, Cat {}
        ```
        
    
    **병합**
    
    ```tsx
    interface User {
      name: string;
    }
    
    interface User {
      age: number;
    }
    
    // 결과
    interface User {
      name: string;
      age: number;
    }
    ```
    
    ㄴ 병합 : 자동으로 합쳐진다
    
- Type Assertion(as 키워드)
    
    ## Type Assertion
    
    > 특정 값의 타입을 강제로 지정해주는 키워드 : (오브젝트) as (타입)
    > 
    
    Typescript에만 존재하는 키워드이며, JavaScript에는 없는 기능이다
    
    ```tsx
    const num = 10;
    const value = num as string; // 컴파일러에게 강제로 알려줌
    ```
    
    #### **사용 예시**
    
    1. **DOM (D**ocument **O**bject **M**odel**)**
    
    ```tsx
    const input = document.getElementById("name") as HTMLInputElement;
    input.value = "hello";
    ```
    
    DOM은 HTML을 JS에서 다룰 수 있게 만든 구조인데,
    
    getElementById는 타입이 너무 넓어서 as HtmlInputElement로 좁혀준다
    
    1. **union/any 타입 처리**
    
    ```tsx
    function printLength(value: unknown) {
      console.log((value as string).length);
    }
    ```
    
    → .length를 사용해야 할 때, unknown은 바로 사용할 수 없으므로
    
         type assertion을 이용해서 문자열이라고 강제로 지정하고 쓸 수 있다
    
    1. **Union 타입 좁힐 때**
    
    ```tsx
    function printLength(value: string | number) {
      console.log((value as string).length);
    }
    
    // 타입 가드 사용 (더 안전)
    if (typeof value === "string") {
      console.log(value.length);
    }
    ```
    
    1. **API 응답 처리 : 서버 응답 타입을 확신할 때**
    
    ```tsx
    const data = fetchSomething() as User;
    ```
    
    → 런타임 검증이 없어 위험할 수 있음
    
    ### 단점
    
    - 버그 숨김
    
    ```tsx
    const value = 10 as string;
    ```
    
        : 컴파일은 통과하지만 논리 오류가 발생할 수 있다
    
    - 타입 안정성 낮음
    
        : TypeScript의 타입 검사를 우회하기 때문이다
    
    - 유지 관리에 영향
    
        : 과도하게 사용하면 코드의 구조나 형태가 변경될 시 그에 영향받는 모든 type 단언도 변경돤 내용에 맞게 업데이트 해야한다
    
    → 최후의 수단으로 최소 사용하기