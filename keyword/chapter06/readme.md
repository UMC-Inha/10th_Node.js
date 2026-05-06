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
    
    - 직관적이고 가독성 좋은 코드
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
        
    - 객체 지향적 접근 → 생산성 증가
    - 재사용, 유지보수 용이
    - DBMS 종속성 감소
    
    #### 단점
    
    - 잘못 구현될 경우 속도 저하
    - 복잡한 쿼리는 QueryDSL 등을 통해 직접 처리해야 함
    - ORM 프레임워크마다 개념과 사용법을 습득해야 함
    
    #### 프레임워크 종류
    
    - JAVA : Hibernate
    - Python : SQLAlchemy, Django ORM
    - JavaScript : Sequelize, TypeORM
    
- Prisma 문서 살펴보기
    - ex. Prisma의 Connection Pool 관리 방법
        
        Prisma는 DB 커넥션을 효율적으로 관리하기 위해 내부적으로 커넥션 풀을 활용한다
        
        1. Prisma Client 설정에서 pool 관련 옵션은 직접 제공하지 않음
            
            : 별도의 풀 설정 옵션을 노출하지 않고, DB 드라이버의 커넥션 풀에 의존하는 구조
            
        2. 환경변수로 연결 최대 개수 제한 가능
            
            : Prisma Client 생성 시 재사용을 고려하는 패턴 (싱글턴 or 전역 인스턴스 사용)을 권장해서 불필요한 커넥션 생성 방지
            
        3. 서버리스 환경에서는 커넥션 제한 주의
            
            : 커넥션이 폭주할 수 있으니 Prisma 공식 안내에 따라 커넥션 관리를 세심하게 해야 한다
            
    - ex. Prisma의 Migration 관리 방법
        1. prisma migrate 명령어
            
            : 스키마 변경사항을 반영 후, 명령어로 마이그레이션을 생성하고 DB에 적용
            
        2. 마이그레이션 파일 관리
            
            : 변경하상을 SQL 스크립트 형태로 버전별 관리함 → 협업 프로젝트에서 효과적이다
            
        3. 환경별 마이그레이션 분리
            
            : 개발, 스테이징, 프로덕션 등 환경에 따라서 마이그레이션 실행을 조절할 수 있다
            
        4. 롤백, 상태확인 기능
            
            : Prisma Migrate는 마이그레이션 상태를 점검하고 문제 발생 시 롤백도 지원한다
            
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