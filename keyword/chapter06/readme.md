- ORM
    
    ### ORM (Object Relational Mapping)
    
    > 객체와 DB의 테이블을 연결시켜서 RDB 테이블을 객체지향적으로 사용하게 해주는 도구
    > 
    > 
    > 객체지향 코드에서 DB를 직접 다루지 않고도 CRUD를 할 수 있도록 도와준다
    > 
    
    #### 기능
    
    - 객체와 테이블 매핑(연결)
        
        : 클래스와 테이블, 객체의 속성과 테이블의 컬럼을 연결
        
    - 데이터 조작 간소화
        
        : SQL문 없이 객체 조작만으로 DB 조작 가능
        
    - 트랜잭션 관리
    - 지연 로딩&캐싱
        
        : 필요한 시점에만 데이터를 불러와서 성능 최적화
        
    
    #### 장점
    
    - 객체 지향적인 코드로 인해서 직관적이고 가독성 좋은 코드
        
        : 비지니스 로직에 더 집중할 수 있게 도와준다
        
    - 재사용, 유지보수 용이
        
        : 객체들을 재활용할 수 있어서 디자인 패턴을 견고하게 다지는 데 유리하다
        
        - 반복 작업 감소
            
            ```sql
            INSERT INTO Products (Name, Price) VALUES ('Laptop', 1200);
            INSERT INTO Products (Name, Price) VALUES ('Phone', 800);
            ```
            
            ```jsx
            context.Products.Add(new Product { Name = "Laptop", Price = 1200 });
            context.Products.Add(new Product { Name = "Tablet", Price = 800 });
            context.SaveChanges();
            ```
            
        
    - DBMS에 대한 종속성 감소
        
        : 객체에 집중해 로직을 구현할 수 있다
        
        : 극단적으로 DBMS를 교체하는 거대한 작업에도, 비교적 적은 리스크와 시간이 소요된다
        
    
    #### 단점
    
    - 완벽하게 ORM으로만 서비스를 구현하기 어려움
        
        : 프로젝트가 복잡해질 경우, 난이도가 상승한다 
        
        : 잘못 구현된 경우에는 속도가 저하되고, 일관성이 무너진다
        
    
    #### 프레임워크 종류
    
    - JAVA : Hibernate
    - Python : SQLAlchemy, Django ORM
    - JavaScript : Sequelize, TypeORM
    
- Prisma 문서 살펴보기
    - ex. Prisma의 Connection Pool 관리 방법
        
        Prisma는 다른 ORM과 같이 커넥션을 효율적으로 관리하기 위해 내부적으로 connection pool 기능을 지원한다
        
        connection pool은 query engine에 의해 관리된다
        
        #### 커넥션 풀 생성
        
        - 명시적으로 $connection() 호출하기
        - Prisma Client api를 호출하기 → 내부적으로 $connection()을 호출함
        
        ### connection_limit
        
        : 커넥션 풀의 크기
        
        ```tsx
        connection_limit=10
        ```
        
        connection_limit 값을 명시하지 않으면, 기본적으로 공식을 활용해 사이즈가 정해진다
        
        **CPU core x 2 +1**
        
        ### pool_timeout
        
        : 쿼리가 커넥션 풀에서 커넥션을 얻기 위해 기다릴 수 있는 최대 시간
        
        - 기본값 = 10초
        - 의미
            - pool에 사용 가능한 커넥션이 없을 때, 쿼리 엔진이 FIFO 큐에 넣고 기다리는 최대 시간
        - 동작 흐름
            1. 쿼리 실행
            2. 남는 커넥션이 없음
            3. 쿼리 → FIFO 큐 대기
            4. pool_timeout 안에 커넥션을 못 얻으면 에러 발생
        
        ### queue 처리 규칙
        
        - FIFO
        - 오래 기다린 요청은 pool_timeout 적용
        
        #### 전체 동작 흐름
        
        1. pool 생성
            - pool 사이즈 결정 : connection_limit / 없으면 공식 활용
        2. connection 생성
            - 처음에는 몇 개의 커넥션만 생성
            - 필요할 때 점점 증가시킴 / connection_limit까지만
        3. 쿼리 실행
            - Prisma Client API 호출
            - 쿼리 엔진이 pool 확인
            - 사용 가능한 커넥션 즉시 할당 / 없으면 FIFO 큐에 대기
        
    - ex. Prisma의 Migration 관리 방법
        
        #### 스키마 마이그레이션 패턴
        
        크게 두 가지로 나뉜다
        
        - Model/Entity-first
        - Database-First
        
        : Prisma는 기본적으로 Model-first 방식에 초점을 맞추고 있다 (DB-first도 지원하긴 함)
        
        ### Model/Entity-first Migration
        
        > 코드로 스키마 구조를 정의하고, 마이그레이션 도구가 DB와 동기화하기 위한 SQL을 자동 생성하는 방식
        > 
        - 작동 방식
            1. 애플리케이션 코드에서 모델 정의
            2. 마이그레이션 도구가 변경사항 감지
            3. SQL 마이그레이션 파일 자동 생성
            4. DB에 적용
        
        ### Prisma의 스키마 상태 추적
        
        추적 요소
        
        1. Prisma Schema : 애플리케이션의 스키마 정의 파일
        2. Migrations History : prisma/migrations 폴더 내의 SQL 파일들
        3. Migrations Table : DB의 _prisma _migraiton 테이블
        4. Database Schema : 실제 DB의 현재 상태
        
        #### 명령어
        
        1. prisma migrate dev
            
            : 개발 환경에서 스키마 변경사항을 추적하고 적용하는 명령어
            
            - 개발 환경에서만 사용해야 한다
        2. —cerate-only 플래그
            
            : 마이그레이션 파일을 생성만 하고 적용하지 않을 때 사용
            
            - 데이터 손실이나 DB 확장 없이 컬럼들을 수정할 때 사용된다
            
            ```tsx
            npm prisma migrate dev --create-only
            ```
            
        3. prisma migrate diff
            
            : 스키마 차이를 비교하고 SQL을 생성하는 명령어
            
            - 수동 변경사항 동기화, 마이그레이션 SQL 미리보기 시 사용한다
            
            ```tsx
            npx prisma migrate diff \
              --from-schema-datamodel prisma/schema.prisma \
              --to-schema-datasource prisma/schema.prisma \
              --script
            ```
            
        4. prisma migrate deploy
            
            : 프로덕션 환경에서 마이그레이션을 적용하는 명령어
            
            - 정합성 검사를 수행하지 않아서 오류가 발생할 수 있다
            
            ```tsx
            npx prisma migrate deploy
            ```
            
        5. prisma db push
            
            : 마이그레이션 히스토리 없이 스키마를 직접 적용하는 명령어
            
            - 데이터 손실 가능성이 있으므로 프로덕션에서는 사용 금지
            
            ```tsx
            npx prisma db push
            ```
            
- ORM(Prisma)을 사용하여 좋은 점과 나쁜 점
    
    ### 좋은 점
    
    1. 직관적인 개발 경험
        
        : Prisma는 타입 안정성을 제공하는 ORM이라서 TypeScript와 잘 어울려, 개발자가 코드 작성 시 오류를 컴파일 단계에서 미리 잡아낼 수 있다
        
        → 버그 발생 가능성 감소, 코드 품질 향상
        
    2. Prisma Client 자동 생성
        
        : 스키마를 정의하면 자동으로 DB와 상호작용할 수 있는 Prisma Client가 생성된다
        
        → SQL문을 직접 작성할 필요가 없고, 다양한 DB 쿼리를 API 형태로 호출할 수 있어서 개발 생산성이 크게 올라간다
        
    3. 넓은 DB 지원
        
        : MySQL뿐만 아니라 PortageSQL, SQLite 등 중요 관계형 DB를 지원해서 다양한 프로젝트에 폭넓게 적용할 수 있다
        
        → 활용도 up
        
    
    ### 나쁜 점
    
    1. 커넥션 풀의 세밀한 제어에 한계가 있음
        
        : 대규모 트래픽 환경에서는 커넥션 관리가 어려울 수 있다
        
    2. 복잡한 쿼리 작성 어려움
        
        : 기본적으로 단순하고 명확한 작업에 최적화되어 있어서 매우 복잡하거나 고도화된 SQL 쿼리는 직접 작성하거나, 쿼리 빌더를 함께 사용해야 한다
        
    3. 오버헤드 (실행 성능)
        
         SQL을 직접 다룰 때보다, 추상화 계층이 추가되어서 성능이 조금 저하될 수 있다
        
    
- 다양한 ORM 라이브러리 살펴보기
    
    #### Hibernate
    
    > 가장 널리 사용되는 Java ORM 프레임워크
    > 
    - 복잡한 쿼리 작성과 다양한 DB를 지원한다
    - 캐싱, 트랜잭션 관리가 뛰어나다
    - 설정과 사용법이 복잡해서 진입 장벽이 높다
    
    대규모 Java 프로젝트에 적합
    
    #### Prisma
    
    > TypeScript 친화적인 ORM
    > 
    - 타입 안정성과 자동 생성 클라이언트 지원
    - 직관적인 API, 자동 마이그레이션 지원 → 빠른 개발에 적합하다
    - 복잡한 쿼리 작성이 어려울 수 있다
    
    #### SQLAlchemy
    
    > 파이썬에서 사용되는 ORM
    > 
    - ORM과 SQL 익스프레스 언어를 지원한다
    - 유연성과 확장성이 뛰어나다
    - 간단한 애플리케이션에는 무거울 수 있다
    
    #### Entity Framework
    
    > 마이크로소프트의 .NET 환경용 ORM
    > 
    - LINQ와 통합되어 있어서 쿼리 작성이 편리하다
    - Visual Studio와 연동이 잘 된다
    - 닷넷 환경에 한정되어 있고, 복잡한 쿼리는 성능이 저하될 수 있다
    
    #### TypeORM
    
    > JS와 TS 모두 지원하는 ORM
    > 
    - 데코레이터 기반이어서 사용법이 직관적이다
    - 다양한 DB를 지원한다
    
- 페이지네이션을 사용하는 다른 API 찾아보기
    - ex. https://docs.github.com/en/rest/using-the-rest-api/using-pagination-in-the-rest-api?apiVersion=2022-11-28
        
        ## Github REST API Pagination
        
        #### 페이지네이션 사용 이유
        
        이슈/PR/커밋 등의 데이터는 양이 아주 많음
        
        → 한번에 보낼 시 
        
        : 서버 부담 증가, 응답 속도 저하, 클라이언트 메모리 부담 증가
        
        따라서 기본적으로 30개 단위로 보낸다
        
        #### 동작 방식 - Link Header
        
        ```tsx
        link: <url?page=2>; rel="next",
              <url?page=1>; rel="first",
              <url?page=515>; rel="last"
        ```
        
        | next | 다음 페이지 |
        | --- | --- |
        | prev | 이전 페이지 |
        | first | 첫 페이지 |
        | last | 마지막 페이지 |
        
        #### 페이지 이동 방식
        
        1. 첫 요청
        2. 응답에서 link header 확인
        3. next URL 가져와서 다시 요청
        
        ### per_page
        
        : 한 페이지 크기 조절
        
        ```tsx
        ?per_page=2
        
        /issues?per_page=2&page=1
        ```
        
        → 한페이지에 2개만 반환한다 : Link Header에도 자동으로 반영
        
        #### 자동/수동 페이지네이션
        
        1. 자동 (Octokit)
            
            ```tsx
            const data = await octokit.paginate("GET /repos/{owner}/{repo}/issues")
            ```
            
            - next page를 계속 요청
            - 마지막 페이지까지 수집
            - 결과를 하나의 배열로 합친다
            
        2. 수동
            - 요청
            - 데이터 저장
            - link header 확인
            - next 있으면 url 갱신
            
    - ex. https://developers.notion.com/reference/intro#pagination
        
        ## Notion API Pagination
        
        > 커서 기반 페이지네이션으로 데이터를 단계적으로 가져오는 구조
        > 
        
        - 기본 응답 개수 : 10개
        - 최대 응답 개수 : 100개정도 (endpoint마다 다름)
        - 초과 데이터는 페이지네이션으로 나눠서 가져온다
        
        #### 응답 구조
        
        ```tsx
        {
          "has_more": true,
          "next_cursor": "33e19cb9-751f-4993-b74d-234d67d0d534",
          "results": [ ... ],
          "object": "list",
          "type": "page"
        }
        ```
        
        | has_more | 다음 페이지 존재 여부 |
        | --- | --- |
        | next_cursor | 다음 페이지 시작 위치 |
        | results | 실제 데이터 |
        | object | 항상 “list” |
        | type | 반환 데이터 타입 |
        
        #### 특징
        
        - 커서 기반이라 안정적이다 : 데이터 추가/삭제 시에도 문제X
        - stateless : 서버가 페이지 번호를 기억하지 않고, 커서만 있으면 이동이 가능하다
        - API 설계 단순 : next_cursor만 넘기면 끝이다
        
        ### Github vs Notion
        
        |  | Github | Notion |
        | --- | --- | --- |
        | 방식 | link header+page URL | cursor |
        | 이동 방식 | URL 따라 | cursor 값 전달 |
        | 종료 조건 | link 없음 | has_more = false |
        | 구조 | REST 전통 방식 | modern cursor 방식 |
        
        #### 동작 흐름
        
        1. 첫 요청
        2. results+next_cursor 받음
        3. next_cursor로 다시 요청
        4. has_more가 false가 될 때까지 반복

#### 시니어
- 한 번에 여러 번의 DB 작업을 연달아 처리할 때, 중간에 처리가 실패했는데 DB에는 중간까지만 값이 반영되어 있으면 문제가 있을 것 같습니다. 이를 방지하는 기술로는 Transaction이 있는데, Prisma를 이용해 Transaction을 관리하는 방법을 찾아 정리해주세요. 워크북의 실습 프로젝트에서도 적용할 수 있다면 적용해주세요.
    
    ### Prisma 트랜잭션 종류
    
    - Sequential 트랜잭션
    - Interactive 트랜잭션
    
    ### Sequential 트랜잭션
    
    > 여러개의 쿼리를 하나의 트랜잭션으로 수행
    > 
    
    ```tsx
    import { PrismaClient } from '@prisma/client';
    
    const prisma = new PrismaClient();
    
    // 결과값은 각 쿼리의 순서대로 배열에 담겨 반환
    const [posts, comments] = await prisma.$transaction([
      prisma.posts.findMany(),
      prisma.comments.findMany(),
    ]);
    ```
    
    : 여러 개의 쿼리를 배열[ ]로 전달받고, 각 쿼리를 순서대로 실행한다 → 여러 작업이 순차적으로 실행되어야 할 때 사용
    
    - Prisma의 모든 쿼리 메서드뿐만 아니라 Raw Query도 사용할 수 있음
    
    ### Interactive 트랜잭션
    
    > Prisma가 자체적으로 트랜잭션의 성공/실패 관리
    > 
    
    ```tsx
    import { PrismaClient } from '@prisma/client';
    
    const prisma = new PrismaClient();
    
    const result = await prisma.$transaction(async (tx) => {
      // 트랜잭션 내에서 사용자 생성
      const user = await tx.users.create({
        data: {
          email: 'testuser@gmail.com',
          password: 'aaaa4321',
        },
      });
    
      // 에러 -> 롤백
      throw new Error('트랜잭션 실패!');
      return user;
    });
    ```
    
    : 트랜잭션 내 모든 로직이 성공적으로 완료 or 에러발생 할 경우, 자체적으로 COMMIT / ROLLBACK을 실행해서 관리한다
    
    - 트랜잭션 진행 중에도 비지니스 로직 처리 가능 → 복잡한 쿼리 시나리오를 효과적으로 구현할 수 있음
    
    #### 사용 방법
    
    1. **$transaction({…])**
        
         : 기본적인 방법
        
        ```tsx
        const result = await prisma.$transaction([
          prisma.user.create({ data: { name: '나현' } }),
          prisma.post.create({ data: { title: '첫 글', authorId: 1 } }),
        ]);
        ```
        
        → 여러 쿼리를 배열로 넣음 : 하나라도 실패하면 전체 롤백
        
    
    1. **$transaction(async tx ⇒ {…})** 
        
        : 트랜잭션 내 로직이 필요한 경우 사용
        
        ```tsx
        await prisma.$transaction(async (tx) => {
          const user = await tx.user.create({
            data: { name: '나현' },
          });
        
          await tx.post.create({
            data: {
              title: '나현의 글',
              authorId: user.id,
            },
          });
        });
        ```
        
        tx : 트랜잭션 컨텍스트 안에서만 사용하는 Prisma 클라이언트
        
    
    1. **Interactive Transactions 설정** 
        
        : 장기 실행 쿼리에 사용
        
        ```tsx
        await prisma.$transaction(async (tx) => {
          // 시간이 오래 걸릴 수 있는 작업
        }, {
          maxWait: 10000,
          timeout: 10000,
          isolationLevel: 'Serializable',
        });
        ```
        
        - prisma.$transaction은 기본적으로 5초 제한(timeout)
        
        → 5초 이상 걸릴 수 있는 트랜잭션 필요 시, interactiveTransactions: true로 설정하고 사용해야 한다
        
        - isolationLevel : 트랜잭션 간 격리 수준 조절
        
        → 기본은 ‘ReadCommitted’이고, ‘Serializable’이 가장 엄격하다

- 우리는 흔히 DB 쿼리에서 N+1 문제를 마주할 수 있습니다. 예를 들어, 게시글을 조회하면서 각 게시글에 해당하는 댓글들을 개별적으로 쿼리한다면, 댓글을 조회하는 쿼리가 각 게시글마다 추가로 발생하게 되어 N+1 문제가 발생합니다. 이를 Prisma에서 N+1문제를 해결할 수 있는 방법을 정리해주세요.
    
    ### N+1문제
    
    > DB에서 데이터 조회 시 발생하는 비효율적인 쿼리 패턴
    > 
    
    **예시)**
    
    User과 Post가 있다 
    
    User은 여러 개의 Post를 작성할 수 있음 → 1 : N 관계
    
    모든 사용자, 그리고 각 사용자가 작성한 게시물들을 함께 조회할 때, 관계형 데이터를 미리 로드하지 않으면 다음과 같이 실행된다
    
    1. 사용자 목록 가져오기 (1)
        
        ```sql
        SELECT id, name, email FROM User;
        ```
        
        : 10명의 사용자가 조회됨 (예시)
        
    2. 각 사용자의 게시글 불러오기 (N)
        
        ```sql
        SELECT id, title, content, authorId FROM Post WHERE authorId = 1; -- 1번 사용자의 게시물
        SELECT id, title, content, authorId FROM Post WHERE authorId = 2;
        -- ... (이런 쿼리가 사용자 수만큼 반복)
        SELECT id, title, content, authorId FROM Post WHERE authorId = 10; -- 마지막 사용자의 게시물
        ```
        
        : 10명의 User이 조회되었으므로, 10번의 쿼리가 실행됨
        
    
    → 총 N+1번의 쿼리가 실행됨
    
    #### 문제
    
    - 네트워크 오버헤드 : 네트워크 지연 발생
    - DB 부하 : 부하 증가, 성능 저하
    
    ### 문제 해결 - Prisma
    
    > Prisma는 이런 문제를 해결하기 위해서 관계형 데이터를 한 번의 쿼리로 미리 가져오는 방식을 제공한다
    > 
    > 
    > : 즉시 로딩 (Eager Loading)
    > 
    
    #### include 사용
    
    **include**
    
    : 메인 모델을 조회하면서, 관련된 다른 모델의 데이터도 함께 가져오는 옵션
    
    → 한 번의 쿼리로 필요한 모든 데이터를 가져올 수 있다
    
    ```tsx
    const users = await prisma.user.findMany({
      include: {
        posts: true,
      },
    });
    
    console.log(users);
    ```
    
    - 네트워크 오버헤드 감소, DB부하 감소, 애플리케이션 성능 UP
    - 필요없는 필드까지 모두 가져올 수 있어서 데이터 전송량이 많아질 수 있음
    
    #### select 사용
    
    **select** 
    
    : 조회하려는 모델에서 특정 필드만 선택적으로 가져오는 옵션
    
    → 불필요한 데이터 전송을 줄인다
    
    include와 함께 사용할 시, 관계된 모델의 특정 필드만 가져올 수 있다
    
    ```tsx
    const users = await prisma.user.findMany({
      select: {
        id: true,
        name: true,
        posts: {
          select: {
            id: true,
            title: true,
          },
        },
      },
    });
    ```
    
    - 네트워크 전송량 / 메모리 사용량 최적화 가능
    - 민감한 정보가 실수로 노출되는 일을 방지할 수 있음
