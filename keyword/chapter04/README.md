- Node.js 에는 두가지 모듈 방식이 존재합니디. ESM, CommonJS 방식에 대해 비교해서 정리해주세요
    
    # 모듈 방식
    
    ```json
    // package.jcon
    {
      "type": "module",
    }
    ```
    
    # CJS (common js)
    
    Node.js가 처음 탄생했을 때부터 채택한 기본 모듈 시스템
    
    - require() 외부 모듈(파일)을 가져올 때 사용
    - module.exports / exports 코드를 밖으로 내보낼 때 사용
    
    ```jsx
    // math.js
    const add = (a, b) => a + b;
    module.exports = { add };
    
    // app.js
    const math = require('./math.js');
    console.log(math.add(2, 3)); // 5
    ```
    
    - 장점
        - cjs 방식을 완벽하게 지원해 호환성 좋음
        - 동적인 로딩 지원한다.
    - 단점
        - 모듈을 불러올 때 동기적으로 작동하기에, 서버 사이드에서는 문제가 적지만, 브라우저 환경에서는 로딩이 완료될 때까지 시행이 멈춰 성능 저하 가능
        - 어떤 모듈을 쓰지 않는지 동적 시간에 알기 때문에 빌드에 사용하지 않는 코드를 제거하는 트리 쉐이킹이 어렵다.
        - 자바스크립트 공식 표준인 ESModule이 등장하여 이제는 레거시가 될 가능성
    
    # ESM
    
    자바스크립트 언어 자체에 탑재된 공식 표준 모듈 시스템
    
    모듈 간 의존성을 정적으로 분석하고, 비동기적으로 로드함.
    
    - import
    - export
    
    ```jsx
    // math.js
    export const add = (a, b) => a + b;
    
    // index.js
    import { add } from './math.js';
    console.log(add(1, 2));
    ```
    
    장점
    
    - 공식 표준, 트리 셰이킹 최적화 굳
    
    단점
    
    - node.js에서 사용하려면 package.json 설정 필요, 기존 CJS 라이브러리 불러올 때 가끔 문제 생길 수 있음
    
    ```jsx
    // named import 오류
    import { shuffle } from 'lodash';
    // -> CJS 라이브러리는 module.exports로 전체 객체 통으로 내보내는 구조 대다수
    // 내부 구조가 잘 구조화되지 않다면 네이밍 오류 발생
    
    // 안전한 방법
    import _ from 'lodash';
    const { shuffle } = _;
    ```
    
    # 한눈에 보기
    
    |  | **CommonJS (CJS)** | **ES Modules (ESM)** |
    | --- | --- | --- |
    | **등장 배경** | Node.js 초기부터 사용된 기본 방식 | ES6(2015)에서 도입된 공식 표준  |
    | **사용 문법** | require() module.exports | import, export |
    | **로드 방식** | 동기적(Synchronous) 로드 | 비동기적(Asynchronous) 로드 |
    | **분석 시점** | 런타임(실행 시) 결정 | 빌드 타임(정적 분석) 결정 |
    | **최적화** | 트리 쉐이킹(Tree Shaking) 어려움 | 트리 쉐이킹을 통한 코드 최적화 용이 |
    | **사용 환경** | 주로 Node.js (서버 사이드) | 브라우저 및 최신 Node.js |
    
- ES6의 주요 문법
    
    # let, const / var 차이
    
    var 방식은 스코프가 모호해 예기치 못한 오류 발생 가능
    
    - const : 재할당 불가 상수
    - let : 재할당 가능 변수
    
    ### var 쓰면 varbo
    
    ```jsx
    var name ='Kim';
    var name ='Lee'; // 가능
    
    let age =20;
    let age =30; // 에러
    ```
    
    ```jsx
    if (true) {
      var hello = 'hi';
      let bye = 'see ya';
    }
    console.log(hello); // 'hi' 출력됨
    console.log(bye);   // 에러! (bye is not defined)
    
    ```
    
    # 구조분해할당
    
    객체, 배열 값을 뽑아낼 때 하나씩 대입하지 않고 한 번에 추출하는 문법
    
    ```jsx
    const user = { name: 'Kim', age: 25 };
    
    // 기존 방식
    const name = user.name;
    
    // 구조분해할당
    const { name, age } = user; 
    const colors = ['red', 'blue'];
    const [first, second] = colors; // first = 'red'
    ```
    
    # async, await
    
    비동기 코드를 동기 코드처럼 작성하는 문법
    
    ```jsx
    async function fetchData() {
      const response = await fetch('url'); // 완료될 때까지 대기
      const data = await response.json();
      console.log(data);
    }
    ```
    
    # 옵셔널 체이닝, nullish 문법
    
    데이터 없을 때 발생하는 에러를 방지하기 위한 문법
    
    - 옵셔널 체이닝 (?) - 앞의 값이 없으면 에러를 내지 않고 undefined 반환
    
    ```jsx
    const city = user?.address?.city; // address가 없어도 에러 안 남
    ```
    
    - nullish (??) - null, undefined일 때 기본값 사용
    
    ```jsx
    const nickname = user.name ?? '익명'; // name이 없으면 '익명' 사용
    ```
    
- TypeScript 문법
    
    ### Typescript를 사용하는 이유(JS와 비교), 기본타입(string, number, any, unknown, never)에 대해 정리
    
    컴파일 타임에 어떤 에러가 발생할 수 있을지 미리 잡기 위함이다.
    
    개발을 위한 것.
    
    자동완성이나 문서화, 리팩토링 등등 개발 과정에 있어서 편리함
    
    타입
    
    - string, number, boolean
    - any - 타입 검사를 아예 꺼버리기 때문에 남발하면 타입스크립트를 사용하는 이유가 사라짐.
    - unknown → any보단 안전하다.
        - any처럼 모든 값을 담을 수 있지만, 사용하기 전 타입 검사를 강제함으로써
        - 유연 + 안전성을 동시에 챙긴다.
    - void : 아무것도 반환하지 않는 함수
    - never
        - 에러를 반환하는 함수, 혹은 무한루프를 갖는 함수의 반환타입
        
        ```tsx
        function fail(message: string): never {
          throw new Error(message); // 함수가 결과값을 반환하지 않고 종료
        }
        
        function infiniteLoop(): never {
          while (true) {
            // 무한 루프
          }
        }
        ```
        
        - switch case 문에서 체크하지 않은 타입을 찾을 때 사용
        
        ```tsx
        type Animal = "Dog" | "Cat" | "Bird";
        
        function getSound(animal: Animal) {
          switch (animal) {
            case "Dog":
              return "멍멍";
            case "Cat":
              return "야옹";
            default:
              // "Type 'string' is not assignable to type 'never'"
              const _exhaustiveCheck: never = animal;
              return _exhaustiveCheck;
          }
        }
        
        ```
        
    
    ### type, interface의 차이를 비교, class와 함께 사용할때 어떻게 달라질까
    
    차이점은 크게 두 가지이다.
    
    - 런타임 시 존재 여부
    - 역할
    
    타입스크립트는 컴파일 하면 타입 자체가 아예 사라지기 때문에, 관련된 interface, type, generic 등이 전부 삭제된다. 변수 뒤 : string 도 모두 삭제된다.
    
    인터페이스는 객체의 설계를 정의하는 용도로, 컴파일 후에 자바스크립트 파일에서 사라지고, 오직 타입 체크를 위해 존재.
    
    클래스는 데이터와 로직을 담는다. 컴파일 후에도 코드로 남고, 생성자로 인스턴스를 생성할 수 있다.
    
    또한 인터페이스는 다중 상속이 가능해서 extends를 통해 아래 처럼 다중상속이 가능하다.
    
    ```jsx
    interface C extends A, B { ... }
    ```
    
    즉, 여러 설계도를 합쳐서 새로운 설계도를 만드는 것이다.
    
    다만 타입스크립트의 클래스는 단일 상속만 가능하다.
    
    실제 인터페이스는 어디에 쓰이나
    
    서비스 계층 간 의존성 분리를 위해 사용된다.
    
    ```tsx
    // 적어도 이 기능은 있어야함
    interface NotificationService {
      sendEmail(to: string, message: string): Promise<void>;
      sendSMS(to: string, message: string): Promise<void>;
    }
    
    // 실제 구현은 나중에
    class EmailService implements NotificationService { 
      // 구현
    }
    
    ```
    
    보통 클래스 dto를 통해 입력갑 검증을 하는데, Interface는 컴파일 후에 사라지기 때문이다.
    
- 프로젝트 아키텍쳐가 무엇인지, 왜 중요한지 설명해주세요
    
    # 레이어드 아키텍쳐가 무엇인지, 각 계층(Controller / Service / Repository)의 역할을 설명
    
    레이어드 아키텍쳐란?
    
    소프트웨어를 구성하는 요소들을 비슷한 관심사로 층을 나누어 쌓아 올린 구조.
    
    각 계층은 바로 아래 계층에만 의존하고, 이를 통해 코드 가독성과 유지보수성을 높인다.
    
    ![image.png](attachment:a7342c27-597f-48f6-91c2-882c127d248a:image.png)
    
    컨트롤러
    
    - 어플리케이션 맨 앞단에서 사용자 요청을 받고 응답을 반환하는 창구
    - http 요청을 해석하고, 비즈니스 로직을 호출한 뒤 적절한 형식(json 등)으로 사용자에게 전달한다.
    
    서비스 (비즈니스 로직)
    
    - 핵심 비즈니스 로직을 수행하는 곳
    - 데이터 유효성 검사하거나, 여러 repository를 조합하여 기능을 처리하는 곳
    
    레포지토리
    
    - 데이터베이스와 통신을 담당하는 곳
    - 디비에 접근해 sql 실행하거나 엔티티 객체를 관리
    - 로직은 거의 포함하지 않고, 데이터 출입에만 집중
    
    계층을 나누는 이점
    
    - 코드의 재사용성이 높아진다.
    - 테스트 용이성
        - DB를 직접 연결하지 않아도, 서비스 계층만 떼어서, 논리적으로 맞는지 테스트하기에 좋음
    - 유지보수 편의성
        - mysql → mongodb 전환 시 계층 분리와 의존성이 나눠져있다면 repository 계층만 수정하면 됨
    
    # 아키텍쳐
    
    아키텍쳐?
    
    - 어떤 대상의 구성과 동작 원리, 구성 요소간의 관계 및 시스템 외부 환경과의 관계를 설명하는 설명서
    - 시스템의 설계 및 동작하는 방식
    
    ![image.png](attachment:26bf0edb-66c0-4fb1-9322-45e384875c04:image.png)
    
    기능도 중요하지만, 구조도 중요하다.
    
    → 초기에는 개발 비용이 많이 들지만, 장기적으로 보았을 때 유지보수에 드는 비용이 증가하기 때문
    
    좋은 구조를 만드는 방법에는 여러가지가 있다.
    
    - 패러다임
    - 설계원칙 SOLID
    - 응집원칙
    - 결합원칙 등…
    
    → 모두 지키기 어려움
    
    아키텍쳐 패턴은 이러한 원칙들을 쉽게 지킬 수 있도록 도와줌. 따라하는 것만으로도 원칙 일부를 지키게 해준다.
    
    헥사고날 아키테쳐
    
    ![image.png](attachment:4dbf3666-27a1-440e-b9d6-313808eb1469:image.png)
    
    기존 레이어드 아키텍쳐를 헥사고날 아키텍쳐로 바꿔보자.
    
    아래와 같은 경우, 특정 클래스에 의존하게된다.
    
    ```tsx
    class UserService {
      constructor(private repository: UserRepository) {} // 특정 클래스 의존
    
      async save(data) {
        return this.repository.save(data); // 리포지토리 함수 직접 호출
      }
    }
    ```
    
    헥사고날 아키텍쳐로 변환하면, 서비스와 레포지토리 사이에 벽(port)을 세워 서비스에서는 레포지토리가 어떤 것인지 모르고, 설명서만 보고 호출할 수 있게 된다.
    
    ```tsx
    interface IUserRepository {
      save(data): Promise<void>;
    }
    
    // 서비스는 이 인터페이스만 바라보기 때문에
    // 진짜 리포지토리가 누군지 모름
    class UserService {
      constructor(private repository: IUserRepository) {} // 인터페이스 의존
    }
    ```
    
    이제 헥사고날 파일 구조까지 완전히 분리해보자.
    
    우선 외부 기술로부터 독립된 애플리케이션의 순수 비즈니스 영역이다.
    
    하나는 서비스가 DB에 저장하기 위해 밖으로 요청하는 포트,
    
    ```tsx
    // src/application/ports/out/user-repository.port.ts
    export interface UserRepositoryPort {
      save(user: any): Promise<void>;
    }
    ```
    
    하나는 컨트롤러가 서비스에 요청할 기능을 인터페이스로 정의한다.
    
    ```tsx
    // src/application/ports/in/create-user.use-case.ts
    export interface CreateUserUseCase {
      execute(name:string, email:string): Promise<void>;
    }
    ```
    
    이렇게 안, 밖과 연결할 수 있는 포트를 만들었다면 이 포트를 사용하는 순수 비즈니스 로직만 담겨있는 서비스 코드를 작성한다.
    
    ```tsx
    // src/application/services/user.service.ts
    export class UserService implements CreateUserUseCase {
      constructor(private readonly userRepository: UserRepositoryPort) {} // 아웃포트 주입
    
      async execute(name: string, email: string): Promise<void> {
        console.log('비즈니스 로직 실행 중...');
        const user = { name, email, createdAt: new Date() };
        await this.userRepository.save(user); // 실제 저장은 어댑터가 알아서 함
      }
    }
    
    ```
    
    즉 외부와 연결할 포트, 그리고 그 포트를 이용하는 서비스 코드로 이뤄졌다.
    
    이제 실제 DB와 통신하거나 HTTP 요청을 받는 영역을 구현해보자.
    
    이 부분은 위에서 정의한 interface를 구현하는 클래스를 각자의 외부 기술로 구현하면 된다.
    
    ```tsx
    // src/infrastructure/adapters/persistence/typeorm-user.repository.ts
    export class TypeOrmUserRepository implements UserRepositoryPort {
      async save(user: any): Promise<void> {
        console.log('TypeORM을 사용하여 DB에 저장 완료!');
        // return await this.orm.save(user);
      }
    }
    ```
    
    ```tsx
    // src/infrastructure/adapters/web/user.controller.ts
    export class UserController {
      constructor(private readonly createUserUseCase: CreateUserUseCase) {}
    
      async handleRequest(req: any, res: any) {
        const { name, email } = req.body;
        await this.createUserUseCase.execute(name, email);
        res.status(201).send('User Created');
      }
    }
    ```
    
    # 디자인 패턴
    
    node.js에서 express와 같이 유용하게 사용할 수 있는 패턴들 위주로 정리해습니다.
    
    ### 미들웨어를 통한 책임 연쇄 패턴
    
    미들웨어 : 요청과 응답 사이에서 작업을 수행하고 next()로 다음 순서에 넘기는 방식
    
    ```tsx
    const express = require('express');
    const app = express();
    
    // 로깅 미들웨어 (모든 요청에서 실행)
    const logger = (req, res, next) => {
      console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
      next(); // 다음 미들웨어나 라우터로 제어권을 넘김
    };
    
    app.use(logger);
    
    app.get('/', (req, res) => {
      res.send('Hello World');
    });
    ```
    
    ### 전략 패턴 (Strategy) - passport.js 예시
    
    실행 중에 알고리즘(행위)을 동적으로 교체할 수 있도록 설계된 디자인 패턴이다.
    
    게임을 만들 때 캐릭텨가 어떤 무기를 들고 있냐에 따라 공격 방식이 달라진다.
    
    즉 function이 자주 바뀔 수 있다.
    
    전략패턴에서는 “공격하기”라는 행동을 정의하고, 구체적인 전략들은 그것을 상속받아 디테일한 것을 구현하는 방식이다.
    
    전략 패턴을 쓰지 않았다면, attack 함수에 아래와 같이 분기를 해야한다.
    
    ```tsx
    if (무기 == "검") {
      ...
    }
    else if (무기 == "활") {
      ...
    }
    ...
    ```
    
    즉, 원래 attack 함수는 “공격을 실행한다”는 명령만 내리면 되는데 위와 같이 if-else로 모든 공격 방법을 다 알고있어야 하게 된다.
    
    → 객체지향원칙의 개방폐쇄원칙 위반!
    
    아래처럼 새로운 무기가 추가되더라도 이 코드는 수정할 필요 없어야한다.
    
    ```tsx
    void attack() {
        currentWeapon->doAttack();
    }
    ```
    
    ![image.png](attachment:4b12b147-f526-4a25-9e08-10851bcc1b94:image.png)
    
    인증 로직 등을 구현할 때 독립된 전략으로 캡슐화해서 필요에 따라 교체하여 사용할 수 있어야한다.
    
    만약에 전략패턴을 사용하지 않는다면 위의 예시와 마찬가지로 passport.authenticate 함수를 다음과 같이 만들어야한다.
    
    ```tsx
    passport.authenticate = (type) => {
      if (type === 'local') {
        // 로컬 로그인 로직...
      } else if (type === 'google') {
        // 구글 로그인 로직...
      } else if (type === 'facebook') {
        // 페이스북 로그인 로직...
      }
      // 새로운 인증 방식이 나올 때마다 이 코드를 계속 고쳐야 함
      // (OCP 위반)
    };
    
    ```
    
    아래 방식은 전략 파턴을 사용하여 개방-패쇄 원칙을 준수한 코드이다.
    
    ```tsx
    const passport = require('passport');
    const LocalStrategy = require('passport-local').Strategy;
    
    // '로컬 로그인'이라는 전략을 정의
    // 여기서 use는 전략 등록 함수이다.
    passport.use(new LocalStrategy(
      (username, password, done) => {
        // 실제 DB에서 사용자 확인 로직 (전략 내용)
        User.findOne({ username }, (err, user) => {
          if (err) return done(err);
          if (!user) return done(null, false);
          return done(null, user);
        });
      }
    ));
    
    // 라우트에서는 어떤 전략을 쓸지만 결정
    app.post('/login', passport.authenticate('local'), (req, res) => {
      res.send('로그인 성공!');
    });
    
    ```
    
    전략 패턴을 사용해서 Nestjs의 Strategy Passport 전략을 구현해보자.
    
    먼저, 모든 전략이 반드시 가지고 있어야하는 메서드를 정의한다. 위의 예시에서는 attack이다.
    
    ```tsx
    class AuthStrategy {
    	validate(data) {
    		throw new Error("validate 메소드 구현해야함");
    	}
    }
    ```
    
    이제 구체적인 전략 클래스를 만들어보자.
    
    ```tsx
    // 1. 로컬 로그인 전략
    class LocalStrategy extends AuthStrategy {
      validate({ username, password }) {
        console.log(`DB에서 ${username}을 찾고 비밀번호를 검증합니다...`);
        // 성공 시 유저 객체 반환
        return { id: 1, name: username, method: 'local' };
      }
    }
    
    // 2. 구글 로그인 전략
    class GoogleStrategy extends AuthStrategy {
      validate({ accessToken }) {
        console.log(`구글 서버에 토큰 ${accessToken}을 보내 유저 정보를 가져옵니다...`);
        return { id: 99, name: '구글유저', method: 'google' };
      }
    }
    ```
    
    전략을 보관하고 실행하는 메니저 클래스를 만든다. 위의 예시에서는 포켓몬 클래스가 될 것이다. 내가 어떤 인증 방식을 사용할 것인지 등록할 수 있어야하고, 실제 인증을 실행하는 함수도 가지고있어야 한다. 이때 인증은 위에서 만든 AuthStrategy에 위임하는 방식이다.
    
    ```tsx
    class AuthManager {
      constructor() {
        this.strategies = new Map(); // 전략 보관함
      }
    
      // NestJS의 passport.use() 같은 역할
      use(name, strategy) {
        this.strategies.set(name, strategy);
      }
    
      // 실제 인증 실행
      authenticate(name, data) {
        const strategy = this.strategies.get(name);
        if (!strategy) {
          throw new Error(`${name} 전략을 찾을 수 없습니다.`);
        }
        return strategy.validate(data); // 위임(Delegation)
      }
    }
    ```
    
    위와 같이 구현했으면 실제 코드에서는 아래처럼 편리하게 사용할 수 있게 된다.
    
    ```tsx
    const authManager = new AuthManager();
    
    // 전략
    authManager.use('local', new LocalStrategy());
    authManager.use('google', new GoogleStrategy());
    
    // 실행
    const loginType = 'local'; // 'google'로 바꾸면 바로 동작이 변함
    const userData = authManager.authenticate(loginType, { username: 'gemini', password: '123' });
    
    console.log('로그인 성공 결과:', userData);
    ```
    
    ### 싱글톤 패턴
    
    노드는 모듈을 처음 불러올 때 그 결과(인스턴스)를 캐싱한다.
    
    따라서 이 인스턴스가 메모리에 저장되어있는데, 웬만하면 다른 파일에서도 메모리에 저장된 기존 객체를 사용하는 것이 좋을것이다.
    
    단순하게 아래처럼 사용하면 두 객체는 메모리에 매번 새로운 객체가 생성된다.
    
    ```tsx
    export class UserService { ... }
    
    // A.js와 B.js에서 각각
    const service = new UserService(); // A와 B는 서로 다른 객체를 가짐
    ```
    
    싱글톤으로 만들기 위해서는 인스턴스를 export해야한다.
    
    ```tsx
    // UserService.js
    class UserService { ... }
    
    export const userService = new UserService(); // 딱 한 번만 실행됨
    
    // A.js와 B.js에서 각각
    import { userService } from './UserService.js'; // A와 B는 동일한 객체를 공유
    ```
    
    Express + tsyringe를 사용해서 싱글톤 다루기
    
    ```tsx
    import { singleton, container } from "tsyringe";
    
    @singleton() // 싱글톤으로 지정
    class DatabaseService {
      constructor() { console.log("DB 연결됨"); }
    }
    
    @singleton()
    class UserService {
      constructor(private db: DatabaseService) {} // 의존성 자동 주입
    }
    
    // 컨테이너를 통해 인스턴스를 가져옴
    const service1 = container.resolve(UserService);
    const service2 = container.resolve(UserService);
    
    console.log(service1 === service2); // true (싱글톤)
    ```
    
- DTO가 무엇인지, DTO 없이 req.body 만 사용하면 어떤 문제가 있을지 조사해주세요
    
    # DTO란
    
    각 레이어 사이에서 데이터 주고받기 위해 사용되는 데이터 전송 객체(Data Transfer Object)이다.
    
    req.body를 그대로 비즈니스 로직이나 DB에 넣으면 여러 문제점 발생
    
    dto 없이 엔티티로 주고받는다면 어떤 문제가 생길까?
    
    - 과도한 데이터 전송
        - 화면에는 이름만 필요한데 엔티티 모든 필드 다 보낸다면 그만큼 네트워크 비용 증가
    - DB에서 컬럼이 바뀌기만 해도 프론트에 전달되는 json의 키 값이 바뀌어버림. API 규격은 DTO는 프론트엔드와의 약속이기에, 그 스펙을 강제시킬 필요가 있다.
    - 검증
        - body 내용을 각 서비스에서 별도로 검증하면 코드가 복잡해짐.
        - 각 dto별로 검증 로직을 미리 갖추어놓으면 재사용하기에도 좋음
    
    # DAO (Data Access Object)
    
    DB에 접속해서 데이터를 넣고 빼는 기능을 담당하는 객체이다.
    
    비즈니스 로직과 DB로직을 분리하기 위해 사용된다.
    
    요즘에는 Nestjs-Typeorm을 통해 DAO 대신 Repository 패턴을 사용.
    
    ```tsx
    // user.dao.js (또는 user.repository.js)
    const db = require('./db-config');
    
    class UserDao {
      async save(userData) {
        // 실제 DB insert 쿼리
        const result = await db.execute('INSERT INTO users ...', [userData.email, ...]);
        return result;
      }
    
      async findById(id) {
        return await db.execute('SELECT * FROM users WHERE id = ?', [id]);
      }
    }
    
    module.exports = new UserDao();
    
    ```