- Node.js 에는 두가지 모듈 방식이 존재합니디. ESM, CommonJS 방식에 대해 비교해서 정리해주세요
    
    > 
    > 
    > 
    > **모듈이란?**
    > 
    > 애플리케이션의 크기가 커지면 유지보수, 재사용을 위해서 여러 파일로 분리하는 것이 효율적인데, 이때 분리된 파일을 모듈이라고 부른다
    > 
    > **모듈의 특징**
    > 
    > - 파일 스코프를 가짐
    > - 캡슐화 - 선택적 공개 가능
    > - 공개된 자산을 자신의 스코프로 불러들여 재사용이 가능하다
    
    - **E**CMA**S**cript **M**odule ****(**ESM**)
        
        : ECMAScript 2015(ES6)에서 도입된 JavaScript의 공식 모듈 시스템
        
        import, export 사용
        
        - 브라우저, Node.js 둘 다 사용 가능
        - 비동기적으로 동작함
            
            → 지연 로딩 가능
            
        - 모듈을 정적으로 분석할 수 있음 (코드 실행 전에 모듈 간의 의존성 관계를 미리 분석함)
            
            → Tree Shaking 같은 최적화 기법 가능
            
        - 엄격한 모듈 스코프를 가짐
            
            → 전역 스코프 오염 방지
            
        
        **가져오기** - import
        
        ```jsx
        import name from './file.js' // default export 값 가져옴
        import {name} from './file.js' // named export 값 가져옴
        import * as name from './file.js' // 모든 내보내기를 하나로 묶어서 가져옴
        ```
        
        ```jsx
        import multiply from './file';
        
        console.log(multiply(2,4)); //8
        ```
        
        **내보내기** - export
        
        ```jsx
        export default value; // 모듈의 대표 값 내보냄 (모듈 당 하나)
        export const name = value; // 이름이 지정된 값 내보냄 (여러개 가능)
        ```
        
        ```jsx
        export cosnt add = (a, b) => a+b;
        export default function multiply(a, b) {
        	return a*b;
        }
        ```
        
    - **C**ommon**JS** (**CJS**)
        
        : 브라우저 외의 환경에서 JavaScript를 사용할 때, 공통의 API 사양을 만들기 위해 시작됨
        
        - 주로 Node.js에서 사용됨
        - 동기적으로 동작함
            
            → 필요한 모듈을 즉시 가져와 사용 가능
            
        - require문은 동적으로 모듈을 로드함 (실행 전에는 require문에서 무엇을 가져올지 알 수 없음)
            
            → Tree Shaking 불가
            
        - 모듈을 파일 단위로 정의함
        
        require, module.exports 사용
        
        **가져오기** - require
        
        ```jsx
        const moddule = require('./file.js'); // 내보낸 객체 전체를 변수에 할당
        const {name} = require('./file.js'); // 객체에서 원하는 속성만 분해해서 가져옴
        ```
        
        ```jsx
        const {add} = require('./file');
        console.log(add(1,2)); //3
        ```
        
        **내보내기** - module.export
        
        ```jsx
        const MyFunction = () => 'Hello CJS';
        
        module.exports = {
        MyFunction
        };
        ```
        
    
    | 비교 표 | **ESM** | **CJS** |
    | --- | --- | --- |
    | **주요 문법** | import/export | require()/module.exports |
    | **방식** | 비동기적 | 동기적 |
    | **default** | 공식적으로 존재 | module.exports가 대체 |
    | **환경** | 브라우저, Node.js(최신) | Node.js(전통적) |
    | **실행 시점** | 컴파일 시(코드 실행 전 모듈 구조 확정) | 런타임(코드 실행 중 require 호출 시) |
- ES6의 주요 문법
    
    ECMAScript는 ECMA(국제 기구) 에서 만든 표준 규격이다
    
    ES6은 ECMAScript 2015로도 알려져 있으며, ES 표준의 가장 최신 버전이다
    
    → 새로운 언어 기능이 포함된 주요 업데이트
    
    → ES5(2009년) 이후 첫 업데이트
    
    - let, const / var 차이
        - **스코프**
            - var
                
                ES6 업데이트 전에는 var을 사용하는 것이 대부분이었다.
                
                함수 단위 스코프 : 함수 내에서 선언된 변수는 함수 내에서만 유효함
                
            - let, const
                
                ES6 업데이트 후 생긴 변수 선언 방식
                
                블록 단위 스코프 : 블록 { }  내에서 선언된 변수는 외부에서 참조 불가
                
                ```jsx
                if (true) {
                	var a =1;
                	let b= 2;
                }
                
                console.log(a); // 1 출력
                console.log(b); // 에러
                ```
                
        - **재할당, 중복 선언**
            
            
            |  | var | let | const |
            | --- | --- | --- | --- |
            | 재할당 | 가능 | 가능 | 불가능 |
            | 중복 선언 | 가능 | 불가능 | 불가능 |
        - **호이스팅**
            
            : 선언한 변수, 함수가 맨 위로 올라가 작동하는 매커니즘
            
            변수, 함수가 작성된 위치와 상관없이 선언 단계에서 메모리에 저장되기 때문에 발생
            
            - **var**에서의 호이스팅
                
                변수 선언 시 : 메모리에 변수 선언 + undefined 값으로 초기화가 동시에 이루어짐 → 호이스팅으로 인한 문제 발생 
                
            - **let/const**에서의 호이스팅
                
                TDZ(Temporal Dead Zone | 변수 선언 ↔ 초기화 사이에 일시적으로 변수 값을 참조할 수 없는 구간)가 존재해서 호이스팅이 발생하지만, 값을 참조할 수 없기 때문에 동작하지 않는 것처럼 보임
                
        
    - 구조분해할당 (Destructuring assignment)
        
        : 배열이나 객체에서 값을 꺼내서 변수에 바로 넣는 문법
        
        구조분해할당의 좌변 : 값을 담을 변수/우변 : 값을 넣을 변수
        
        - **배열 분해**
            
            할당할 값이 없으면 undefined로 취급
            
            → 할당하려는 변수의 개수가 분해하려는 배열의 길이보다 커도 에러 발생 X
            
            → =을 이용해 할당할 값이 없을 때 기본값 설정 가능
            
            ```jsx
            const arr = [1,2,3];
            
            const [a,b,c] = arr; // 순서대로 담음
            console.log(a); // 1
            
            const [d, , e] = arr; // 일부만 꺼내기
            console.log(e); // 3
            
            const [a = 10, b = 20] = [1];
            console.log(a); // 1 (a에 1이 할당되었으므로 기본값 a=10 무시)
            console.log(b); // 20
            
            const [a, ...rest] = [1,2,3,4]; // rest
            console.log(a); // 1
            console.log(rest); // [2,3,4]
            ```
            
        - **객체 분해**
            
            → 키 이름 기준으로 할당하기 때문에 순서가 중요하지 않음
            
            → 프로퍼티가 없는 경우를 대비해 =을 사용해 기본값 설정 가능
            
            ```jsx
            const user = {
            	name: "gildong",
            	age: 50
            };
            
            const { name, age } = user; // 배열과 달리 키 이름 기준으로 가져옴
            const { name: username } = user; // {객체 프로퍼티: 목표 변수}
            
            concole.log(username); // gildong
            ```
            
        - **중첩 구조 분해**
            
            객체/배열이 다른 객체/배열을 포함하는 경우에, 중첩 배열이나 객체의 정보를 추출하는 것
            
            ```jsx
            // 배열
            const arr = [1, [2, 3]];
            
            const [a, [b, c]] = arr;
            console.log(b); // 2
            
            // 객체
            const user = {
              name: "kim",
              address: {
                city: "seoul"
              }
            };
            
            const { address: { city } } = user;
            
            console.log(city); // seoul
            ```
            
    - async, await
        
        : promise와 then 체인을 쉽게 사용할 수 있도록 도와주는 문법
        
        → 비동기 코드를 동기 코드처럼 작성할 수 있다
        
        - async
            
            : 함수 선언 앞에 async를 붙이면 해당 함수는 자동으로 promise를 반환함
            
            async 함수의 결과값을 외부에서 사용할 경우 .then을 통해 처리
            
            ```jsx
            
            ```
            
        - await
            
            : Promise가 해결될때까지 기다린 후 결과값을 반환함
            
            Promise.then()을 대신하고, async 함수 내에서 사용 가능
            
            ```jsx
            
            ```
            
    - 옵셔널 체이닝, nullish 문법
        - **옵셔널 체이닝 (?.)**
            
            : 일반 마침표(.)와 동일하지만, 객체의 프로퍼티에 접근할 때, 해당 객체가 null/undefined인 경우
            
            → 에러 발생 없이 undefined를 반환함
            
            → 에러를 발생시키지 않도록 감추는 역할
            
            ```jsx
            var user = {
            	name: 'kim',
            	age: 20
            }
            
            console.log(user.email); // 에러
            console.log(user?.email); // undefined
            ```
            
            객체에서 존재하지 않는 프로퍼티에 안전하게 접근할 수 있도록 도와준다
            
        - **nullish 문법 (??)**
            
            좌측의 값이 null/undefined일때, 우측의 값을 반환함
            
            → 데이터가 늦게 도착하거나, 아직 값이 설정되지 않았을 때 기본값을 제공하는 역할
            
            ```jsx
            var user;
            
            console.log(user ?? '로딩중'); // 로딩중
            ```
            
            기본값을 제공하여 사용자에게 안정적인 데이터를 보여줄 수 있다
            
- TypeScript 문법
    - Typescript를 사용하는 이유를 JS와 비교해서 설명하고, 기본타입(string, number, any, unknown, never)에 대해 정리해주세요
        - **TypeScript를 사용하는 이유**
            
            JS는 동적 타입 언어로, 변수에 어떤 타입의 값이든 자유롭게 할당할 수 있어서, 선언 시에 타입을 지정하지 않기 때문에, 동작하면서 형변환이 되어 있으면 예상치 못한 오류를 발생시킬 수 있다
            
            JS는 런타임에 오류가 드러나기 때문에, 디버깅이 어려워질 수 있다
            
            TS는 정적 타입 언어로, 변수나 함수에 타입을 명시할 수 있다
            
             TS를 사용하면 컴파일 단계에서 에러를 알려주기 때문에 에러를 미리 방지할 수 있다
            
        - **TS의 장점**
            1. 타입 안정성 확보
            2. 가독성 향상
            3. 개발 생산성 증가
            4. 유지보수 용이성
            
        - **기본 타입**
            - string
                
                : 문자열
                
                ```tsx
                let name: strinf = "kim";
                ```
                
            - number
                
                : 숫자 (정수, 실수 구분 X)
                
                ```tsx
                let age: number = 100;
                ```
                
            - any
                
                : 모든 타입 (JS와 동일하게 동작하며, 타입 검사X)
                
                타입 안정성이 사라지므로 사용을 최소화하는 것이 좋다
                
                ```tsx
                let value: any = 10;
                value = "hello";
                value = true;
                ```
                
            - unknown
                
                : 타입을 알 수 없을때
                
                any와 비슷하지만, 타입 검사를 수행하기 때문에 더 안전하다
                
                ```tsx
                let value: unknown = "hello";
                
                value.toUpperCase(); // 에러 발생
                ---
                if (typeof value === "string") {
                  value.toUpperCase(); // 정상 동작
                }
                
                ```
                
            - never
                
                : 값이 절대 반환되지 않는 경우
                
                항상 에러를 발생시키거나, 무한 루프에 빠지는 함수에서 사용한다
                
                함수가 정상적으로 종료되지 않음을 명시적으로 표현할 때 사용된다
                
                ```tsx
                function throwError(): never {
                  throw new Error("error");
                }
                ---
                function infiniteLoop(): never {
                  while (true) {}
                }
                ```
                
        
    - (+시니어) type, interface의 차이를 비교하고, class와 함께 사용할때 어떻게 달라지는지 설명해주세요
        
        TypeScript에서 type과 interface는 객체의 형태를 정의하는 데 사용되지만, 사용 방식과 확장성에서 차이가 있다
        
        - **기본 정의 방식** : 거의 동일하다
            - type
                
                ```tsx
                type User = {
                  name: string;
                  age: number;
                };
                ```
                
            - interface
                
                ```tsx
                interface User {
                  name: string;
                  age: number;
                }
                ```
                
        - **확장** : & vs extends
            - type : &(intersection)
                
                ```tsx
                type Person = {
                  name: string;
                };
                
                type User = Person & {
                  age: number;
                };
                ```
                
            - interface : extends
                
                ```tsx
                interface Person {
                  name: string;
                }
                
                interface User extends Person {
                  age: number;
                }
                ```
                
        - **선언 병합** : interface만 가능 (type은 에러 발생)
            
            ```tsx
            interface User {
              name: string;
            }
            
            interface User {
              age: number;
            }
            // 자동으로 합쳐짐
            
            // 결과
            interface User {
              name: string;
              age: number;
            }
            ```
            
        - **표현 범위** : type이 더 넓음
            
            type만 가능한 것들
            
            ```tsx
            type ID = string | number; // 유니온 타입
            
            type Point = [number, number]; // 튜플
            
            type Func = (a: number) => number; // 함수 타입
            ```
            
        - **class와 함께 사용할 때 type vs interface**
            
            둘 다 implements로 구현할 수 있다
            
            - 확장 방식
                - interface : class 상속 구조와 유사 (extends)
                    
                    ```tsx
                    interface Animal {
                      name: string;
                    }
                    
                    interface Dog extends Animal {
                      breed: string;
                    }
                    
                    class MyDog implements Dog {
                      name = "dog";
                      breed = "poodle";
                    }
                    ```
                    
                - type : 조합 중심 (&)
                    
                    기능적으론 같지만 상속보다는 타입을 합치는 느낌
                    
                    ```tsx
                    type Animal = {
                      name: string;
                    };
                    
                    type Dog = Animal & {
                      breed: string;
                    };
                    
                    class MyDog implements Dog {
                      name = "dog";
                      breed = "poodle";
                    }
                    ```
                    
            - 선언 병합 가능 여부
                
                interface는 자동으로 합쳐지지만, type는 같은 이름으로 재선언이 불가능해서 병합할 수 없다 → 대규모 프로젝트에서 확장성 차이 발생
                
            - 정리
                - interface + class
                    
                    확장이 자연스러움
                    
                    선언 병합 가능 → 확장성 굿
                    
                    클래스 구조 정의용
                    
                - type + class
                    
                    기본적 구조 구현 가능
                    
                    복잡한 타입 조합에 강함
                    
                    class 설계용으로는 제한적
                    
                
                → type은 타입 유연성이 필요한 경우에 쓴다다
                
        
- 프로젝트 아키텍쳐가 무엇인지, 왜 중요한지 설명해주세요
    - 레이어드 아키텍쳐가 무엇인지, 각 계층(Controller / Service / Repository)의 역할을 설명해주세요
        - 레이어드 아키텍처
            
            > 소프트웨어를 여러 개의 계층으로 분리해서 설계하는 방법
            > 
            > 
            > 각 계층이 서로 독립적으로 동작하며 상위 계층에서 하위 계층으로의 의존만 존재하는 형태
            > 
            
            구조 흐름 : Controller → Service → Repository → Database
            
            요청을 위에서 아래로 내려가고, 결과는 아래에서 위로 올라옴
            
            각 계층이 하나의 책임만 가지게 해서 유지보수성, 재사용성, 테스트가 용이하지만, 계층이 많아지면 구조가 복잡해질 수 있다
            
        - 각 계층의 역할
            - Controller
                
                : 요청과 응답을 담당하는 계층
                
                - 클라이언트의 HTTP 요청을 받는다
                - 요청 데이터(파라미터, 바디 등)을 처리한다
                - 서비스를 호출한다
                - 결과를 응답 형태(JSON 등)로 반환한다
                
                → 비지니스 로직을 넣으면 안됨
                
                ```tsx
                @Controller()
                export class UserController {
                  @Get("/users")
                  getUsers() {
                    return this.userService.getUsers();
                  }
                }
                ```
                
            - Service
                
                : 비지니스 로직을 처리하는 핵심 계층
                
                - 실제 로직을 수행한다
                - 여러 리포지토리를 조합할 수 있다
                - 트랜잭션을 처리한다
                
                → 핵심 로직이 해당 계층에 존재
                
                ```tsx
                @Injectable()
                export class UserService {
                  getUsers() {
                    return this.userRepository.findAll();
                  }
                }
                ```
                
            - Repository
                
                : 데이터 접근을 담당하는 계층
                
                - DB와 직접 통신한다
                - CRUD를 처리한다
                
                → SQL/ORM 처리
                
                ```tsx
                @EntityRepository(User)
                export class UserRepository {
                  findAll() {
                    return this.repo.find();
                  }
                }
                ```
                
    - (+ 시니어) 프로젝트 아키텍쳐와 디자인 패턴에 대해 조사해서 설명해주세요
        
        소프트웨어 개발 시에 어떤 구조로 시스템을 설계할 것인지, 각 기능을 어떤 방식으로 구현할 것인지를 고려해야 하는데, 이 때 등장하는 개념이 프로젝트 아키텍처와 디자인 패턴이다
        
        - 프로젝트 아키텍처
            
            : 애플리케이션 전체적인 구조를 설계하는 것
            
            - 역할
                - 구조적 분리 : 기능을 역할별로 분리해서 각 부분이 하나의 책임만 지도록 한다
                - 확장성, 유지보수성 확보 : 기능이 추가 또는 변경될 때, 전체 시스템에 미치는 영향을 최소화할 수 있도록 한다
                - 팀 협업 효율 증가 : 구조가 명확하면, 각자의 역할이 분명해지기 때문이다
                
            - 예시
                - 레이어드 아키텍처 : Controller/Service/Repository
                - MVC 패턴 : Model/View/Controller
                - 클린 아키텍처 : 의존성 방향 관리
                
        - 디자인 패턴
            
            : 개발 과정에서 반복적으로 발생하는 문제를 해결하기 위한 검증된 코드 설계 방식
            
            특정 상황에서 어떻게 하면 좋을지 알려주는 역할
            
            - 역할
                - 재사용 가능한 해결 방법 : 이미 많은 개발자들이 사용하고 검증했기 때문에 비슷한 문제를 만났을 때 빠르게 적용할 수 있다
                - 코드 유연성 향상 : 패턴을 사용하면 코드 간 결합도를 낮추고, 변경에 유연하게 대응할 수 있는 구조를 만들 수 있다
                - 의사소통 도구 : 구현 방식에 대한 설명 없이도 구조를 이해할 수 있다
            
            - 종류 : 크게 생성, 구조, 행위 패턴으로 분류됨
                - 생성 패턴
                    
                    → 싱글톤 패턴, 팩토리 메서드 패턴, 추상 팩토리 패턴, 빌더 패턴, 프로토타입 패턴
                    
                - 구조 패턴
                    
                    → 어댑터 패턴, 브릿지 패턴, 컴포지트 패턴, 데코레이터 패턴, 퍼사드 패턴, 플라이웨이트 패턴, 프록시 패턴
                    
                - 행위 패턴
                    
                    → 옵저버 패턴, 전략 패턴, 커맨드 패턴, 상태 패턴, 책임 연쇄 패턴, 방문자 패턴, 인터프리터 패턴, 메멘토 패턴, 중재자 패턴, 템플릿 메서드 패턴, 이터레이터 패턴
                    
            
    
- DTO가 무엇인지, DTO 없이 req.body 만 사용하면 어떤 문제가 있을지 조사해주세요
    - **DTO (Data Transfer Object)**
        
        : 계층 간 데이터 전달을 위해 사용하는 객체
        
        주로 Controller에서 받은 요청 데이터를 저으이된 형태로 검증하고 전달하는 역할을 한다
        
        → 요청 데이터를 정해진 구조로 포장해서 넘기는 것
        
        - **DTO 사용 이유**
            1. 데이터 구조 명확화 : 어떤 값이 들어오는지 한눈에 알 수 있음
            2. 유효성 검사 가능 : class-validator
                
                ```tsx
                export class CreateUserDto {
                  @IsString()
                  name: string;
                
                  @IsNumber()
                  age: number;
                }
                ```
                
            3. 계층 간 역할 분리 : Controller ↔ Service 사이를 깔끔하게 분리함
            
    - **DTO 없이 req.body만 쓴다면?**
        
        ```tsx
        // 예시
        @Post()
        create(@Body() body: any) {
          return this.userService.create(body);
        }
        ```
        
        1. 타입 안정성 X :  잘못된 데이터가 들어와도 막지 못한다
        2. 유효성 검사 불가 : 값이 빠지거나, 이상한 값이 들어와도 그대로 들어간다
            
            ex) 나이 : 음수
            
        3. 보안 문제 : 클라이언트가 원하지 않는 값까지 넣을 수 있다
        4. 유지보수 어려움 : 어떤 값이 오는지 코드만 보고는 모른다
        
        → DTO는 단순히 데이터를 담는 객체가 아니라, 데이터의 구조를 정의하고 검증하며 안전하게 전달하기 위한 핵심 요소이다. DTO를 사용하지 않을 경우 위와 같은 문제가 발생할 수 있다