### 실습
1. 진행 중, 진행 완료한 미션

```sql
SELECT 
	m.mission_id,
	m.title,
	s.store_name,
	m.reward_point,
	m.target_amount,
	um.status,
	um.received_at
FROM user_mission um
JOIN mission m on um.mission_id = m.mission_id
JOIN store s ON m.store_id = s.store_id
WHERE um.user_id = :(유저 아이디)
AND um.status = 'RECEIVED' #진행 완료 미션 : RECEIVED -> COMPLETED로 변경
ORDER BY um.received_at DESC
LIMIT 10 OFFSET ((미션 개수)-1)*10;
```

2. 리뷰 작성 쿼리

```sql
INSERT INTO review (
	user_id,
	store_id,
	rating,
	content,
	created_at
)
VALUES (
	:(유저 아이디)
	:(가게 아이디)
	:rating,
	:content,
	NOW()
);
```

3. 홈 화면 쿼리

```sql
#완료한 미션 n/10
SELECT 
    COUNT(*) % 10 AS current_count
FROM user_mission
WHERE user_id = :userId
  AND status = 'COMPLETED';
  

#포인트 잔액
SELECT point
FROM user
WHERE user_id = (유저 아이디);

#선택한 지역에서 할 수 있는 미션
SELECT
	m.mission_id,
	m.title,
	m.description,
	m.reward_point,
	m.target_amount,
	s.store_name,
	s.district
FROM mission m
JOIN store s ON m.store_id = s.store_id
LEFT JOIN user_mission um
	ON um.mission_id = m.mission_id
	AND um.user_id = (유저 아이디)
WHERE s.district = (지역)
AND (um.status = IS NULL OR um.status !=)
ORDER BY m.created_at DESC
LIMIT 10 OFFSET (미션 개수 -1)*10;
```

4. 마이페이지 쿼리

```sql
SELECT
	name,
	email,
	phone,
	phone_verified,
	point
FROM user
WHERE user_id = (유저 아이디);
```


## 시니어 미션
### 미션 1(내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함))에서 정렬 기준을 1순위는 포인트로 2순위는 최신순으로 하여 Cursor기반 페이지네이션을 구현해보세요
    - 설명
        
        ```sql
        SELECT m.mission_id,
        	m.title,
        	s.store_name,
        	m.reward_point,
        	m.target_amount,
        	um.status,
        	um.received_at
        FROM user_mission um
        JOIN mission m on um.mission_id = m.mission_id
        JOIN store s ON m.store_id = s.store_id
        WHERE um.user_id = userId
        AND um.status = 'RECEIVED'
        AND ( 
        	m.reward_point < :lastRewardPoint 
        OR ( m.reward_point = :lastRewardPoint AND um.received_at < :lastReceivedAt ) 
        OR ( m.reward_point = :lastRewardPoint AND um.received_at = :lastReceivedAt AND um.usermission_id < :lastUserMissionId
        )
        ORDER BY 
        	m.reward_point DESC,
        	um.received_at DESC
        LIMIT 10;
        ```
        

- [x]  SQL Injection에 대해 조
###  SQL Injection에 대해 조사하고 어떠할 때 일어나고 어떻게 막을 수 있는 지를 적어주세요
    - 설명
        - **SQL 인젝션 (SQLi) 이란?**
            
            웹 애플리케이션에 직접적인 영향을 미치는 대표적인 보안 위협 중 하나이다
            
            비교적 오래된 공격 기법이지만, 여전히 많은 서비스에서 발생하고 있다
            
            주로 사용자 입력값을 제대로 검증하지 않고 SQL 쿼리에 그대로 포함시키는 경우에 발생한다
            
            특히, 취약한 코드 패턴이 포함된 공유 코드나, 래거시 시스템을 통해서 쉽게 확산된다
            
            공격자는 SQL 구문을 조작해서 DB에 악성 쿼리를 실행시키고, 이를 통해 다음과 같은 행위를 시도할 수 있다
            
            - 무단 로그인 (인증 우회)
            - 데이터 조회 및 유출
            - 데이터 수정, 삭제
            - DB 구조 파악
        
        - **유형**
            - **대역 내 SQLi**
                
                가장 일반적이고 널리 사용되는 방식 : 단순하고 효율성이 높다
                
                같은 통신 채널 (HTTP응답)을 통해 결과를 직접 확인할 수 있는 공격이다
                
                - **오류 기반 SQLi**
                    
                    DB가 반환하는 오류 메시지를 이용하는 방식이다
                    
                    공격자 :  의도적으로 오류가 발생하도록 SQL 주입
                    
                    → 노출되는 오류 메시지를 통해 DB 구조나 정보를 추출
                    
                - **조합 기반 SQLi**
                    
                    UNION 연산자를 이용해서 기존 쿼리 결과에 추가 데이터를 결합하는 방식이다
                    
                    공격자 : 정상적인 쿼리에 자신의 쿼리를 결합
                    
                    → 결과가 HTTP 응답에 포함되는 것을 통해 DB의 정보를 확인
                    
            - **추론 기반 (블라인드) SQLi**
                
                서버가 오류 메시지나 결과 데이터를 직접 반환하지 않을 때 사용하는 방식이다. 속도는 느리지만 매우 강력하다!
                
                응답 결과 대신 서버의 동작 변화를 분석해서 DB 정보를 추론한다
                
                - **부울 기반 SQLi**
                    
                    조건식의 참/거짓에 따라 서버 응답이 어떻게 달라지는지 관찰하는 방식이다
                    
                    공격자 : 특정 조건을 포함한 쿼리 전송
                    
                    → 응답 결과의 차이를 통해 해당 조건이 참인지 거짓인지 파악하며 데이터를 하나씩 추출
                    
                - **시간 기반 SQLi**
                    
                    DB의 지연 함수를 이용하는 방식이다
                    
                    조건이 참일 경우 일정시간 지연되도록 쿼리를 구성하고, 응답 시간이 길어지는지를 통해 조건의 참/거짓을 판단한다
                    
                    → 응답 내용이 없어도 공격 가능함
                    
            - **대역 외 SQLi**
                
                일반적인 HTTP 응답을 통해서 데이터를 확인할 수 없을 때 사용하는 방식이다. DM가 외부와 통신할 수 있는 기능(DNS, HTTP 요청 등)을 이용해서 데이터를 외부 서버로 전송하게 만드는 공격이다
                
                - 특정 DB 기능이 켜져 있는 경우에만 작동
                - 사용 빈도는 낮지만 강력함
                - 다른 SQLi 방식이 어려울 때 대안으로 사용됨
                
                공격자 : DB가 외부 서버로 요청을 보내도록 유도, 요청에 데이터를 포함시켜 유출
                
                → 데이터를 직접 받는것이 아닌, DB가 외부로 데이터를 보내게 유도하는 구조
                
        
        - **막는 법**
            1. 입력값 제한 및 소독
                
                사용자가 입력할 수 있는 값의 범위를 제한하거나 위험한 문자열을 제거, 변환하는 방법이다
                
                → 입력 길이 제한, 특정 문자만 허용, 비정상 입력 거부
                
                예) 비밀번호 생성 시 :  최소 길이, 특수문자 포함
                
            2. 웹 애플리케이션 방화벽
                
                웹 애플리케이션 앞단에서 요청을 검사해서 악성 트래픽을 차단하는 보안 시스템이다
                
                → SQLi 패턴을 기반으로 탐지, 의심스러운 요청 자동 차단
                
                → 이 방법으로 알려진 SQLi 공격에 효과적인 방어 가능
                
            3. SQL 실행 범위 및 접근 제어
                
                SQL 쿼리 실행 범위와 접근 권한을 제한해서 비정상적인 접근을 차단하는 방법이다
                
                → 특정 IP에서만 접근 허용, 내/외부 사용자 권한 분리, 관리자 기능은 별도로 보호
                
            4. 안전한 URL 및 HTTPS 사용
                
                웹사이트에서 데이터를 주고받을때 암호화를 적용해서 중간에 데이터가 탈취되거나 조작되는 것을 방지하는 방법이다
                
                → HTTPS, SSL/TLS

### 다양한 JOIN 방법들에 대해 찾아보고, 각 방식에 대해 비교하여 간단히 정리해주세요.
    - 설명
        
        
        | 종류 | 핵심 |
        | --- | --- |
        | INNER JOIN | 교집합 |
        | LEFT JOIN | 왼쪽 기준, 없으면 NULL |
        | RIGHT JOIN | 오른쪽 기준, 없으면 NULL |
        | FULL OUTER JOIN | 합집합 |
        | CROSS JOIN | 곱집합 |
        | SELF JOIN | 자기자신과 조인 |
        - INNER JOIN
            
            : 두 테이블에서 조건이 일치하는 데이터만 조회
            
            가장 많이 사용되며, 매칭이 안되면 제외하기 때문에 NOT NULL이다
            
            ```sql
            SELECT *
            FROM A
            INNER JOIN B ON A.id = B.id;
            ```
            
        - LEFT JOIN
            
            : 왼쪽 테이블 기준으로 모든 데이터, 매칭 안되면 NULL
            
            왼쪽은 무조건 다 조회되며, 오른쪽이 없으면 NULL 처리된다
            
            ```sql
            SELECT *
            FROM A
            LEFT JOIN B ON A.id = B.id;
            ```
            
        - RIGHT JOIN
            
            : LEFT JOIN의 반대 개념으로, 오른쪽 테이블 기준으로 한다
            
            RIGHT JOIN보다는 LEFT JOIN을 주로 사용한다
            
            ```sql
            SELECT *
            FROM A
            RIGHT JOIN B ON A.id = B.id;
            ```
            
        - FULL OUTER JOIN
            
            : 양쪽 테이블의 모든 데이터를 포함
            
            매칭이 되면 합쳐서 출력하며, 한쪽만 있으면 NULL로 채운다
            
            ```sql
            SELECT *
            FROM A
            FULL OUTER JOIN B ON A.id = B.id;
            ```
            
        - CROSS JOIN
            
            : 모든 경우의 수
            
            만약 A에 3개, B에 5개가 있으면 결과는 15개가 나온다
            
            ```sql
            SELECT *
            FROM A
            CROSS JOIN B;
            ```
            
        - SELF JOIN
            
            : 한 테이블을 자기 자신과 조인한다
            
            계층 구조를 표현할 때 사용한다

```sql
SELECT A.name, B.name
FROM employee A
JOIN employee B ON A.manager_id = B.id;
```

.