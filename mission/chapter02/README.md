# 2주차 미션
1. 1주차 때 설계한 데이터베이스를 토대로 아래의 화면에 대한 쿼리를 작성

![내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)](attachment:fb2c37cf-9eb2-4e18-b21c-af42c1dd2cc0:Untitled.png)

내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)

![리뷰 작성하는 쿼리,
* 사진의 경우는 일단 배제](attachment:ff64bece-430c-45e4-96fa-de1e40b984db:Untitled.png)

리뷰 작성하는 쿼리,
* 사진의 경우는 일단 배제

![홈 화면 쿼리
(현재 선택 된 지역에서 도전이 가능한 미션 목록, 페이징 포함)](attachment:6ad90e83-6e2e-4f56-8779-82d16c7f7303:Untitled.png)

홈 화면 쿼리
(현재 선택 된 지역에서 도전이 가능한 미션 목록, 페이징 포함)

![마이 페이지 화면 쿼리](attachment:2d736cd4-e3ab-4c55-9b1d-ad818cc22dbf:Untitled.png)

마이 페이지 화면 쿼리

```sql
-- 1 페이지 당 10개
SELECT 
    m.id AS mission_id,
    s.id AS store_id,
    s.name AS store_name, 
    m.point AS mission_point,
    m.price AS mission_condition_price,
    um.status AS mission_status,
    um.created_at AS completed_at
FROM user_mission um
JOIN mission m ON um.id = m.id AND um.store_id = m.store_id
JOIN store s ON m.store_id = s.id
WHERE um.user_id = 1              -- 특정 유저의 데이터
  AND um.status = 'SUCCESS'       -- '성공'한 미션만
  AND um.deleted_at IS NULL       -- 삭제되지 않은 미션
ORDER BY um.created_at DESC       -- 최신순 정렬
LIMIT 10 OFFSET 0;                -- 10개씩 가져오기 (0부터 시작)
 
```

- 개인의 것만 조회하기 때문에 created_at 칼럼이 중복되지 않아 동시성 문제가 발생하지 않는다고 가정했습니다.

```sql
UPDATE review
SET 
    answer = '리뷰 감사합니다.',
    answered_at = NOW() -- 답글 작성 시간
WHERE 
    mission_id = 101  -- 해당 미션 ID
    AND store_id = 5  -- 해당 상점 ID
    AND user_id = 12  -- 리뷰를 쓴 유저 ID
    AND deleted_at IS NULL; -- 삭제되지 않은 리뷰인지 확인

```

```sql
 SELECT 
    COUNT(*) AS completed_count, -- 현재 달성 개수 (예: 7)
    10 AS goal_count,            -- 목표 개수 (고정값)
    1000 AS reward_point         -- 보상 포인트
FROM user_mission
WHERE user_id = 1 
  AND status = 'SUCCESS'         -- '성공' 상태인 미션만 카운트
  AND deleted_at IS NULL;

```

```sql
SELECT 
    m.id AS mission_id,
    s.id AS store_id,
    m.price AS condition_price,
    m.point AS reward_point,
    DATEDIFF(m.end_at, NOW()) AS d_day,
    -- 날짜를 YYYYMMDD + ID 앞에 0 패딩
    CONCAT(DATE_FORMAT(m.end_at, '%Y%m%d'), LPAD(m.id, 10, '0')) AS cursor_str
FROM mission m
JOIN store s ON m.store_id = s.id
LEFT JOIN user_mission um ON m.id = um.id AND um.user_id = 1
WHERE um.id IS NULL 
  AND m.end_at > NOW()
  AND s.address_doro LIKE '%안암%'
  -- 마지막 커서보다 큰 데이터만 조회
  AND CONCAT(DATE_FORMAT(m.end_at, '%Y%m%d'), LPAD(m.id, 10, '0')) > '202406010000000105'
ORDER BY m.end_at ASC, m.id ASC
LIMIT 10;

```

```sql
SELECT 
    u.nickname,
    u.email,
    CASE 
        WHEN u.Is_phone_verified = 1 THEN '인증완료'
        ELSE '미인증'
    END AS phone_status,
    u.profile_url,
    IFNULL(SUM(p.amount), 0) AS total_point
FROM user u
LEFT JOIN point p ON u.id = p.user_id
WHERE u.id = 1 -- 현재 로그인한 유저 ID
GROUP BY u.id;

```

---
# 시니어 미션
![내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)](attachment:fb2c37cf-9eb2-4e18-b21c-af42c1dd2cc0:Untitled.png)

내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)

- [x]  미션 1(내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함))에서 정렬 기준을 1순위는 포인트로 2순위는 최신순으로 하여 Cursor기반 페이지네이션을 구현해보세요
    - 설명
        
        ```sql
        SELECT 
            m.id AS mission_id,
            m.point AS mission_point,
            m.price AS condition_price,
            um.status AS mission_status,
            um.created_at AS mission_created_at,
            CONCAT(
                LPAD(m.point, 10, '0'), 
                DATE_FORMAT(um.created_at, '%Y%m%d%H%i%S'), 
                LPAD(um.id, 10, '0')
            ) AS cursor_str
        FROM user_mission um
        JOIN mission m ON um.id = m.id AND um.store_id = m.store_id
        WHERE um.user_id = 1 
          AND um.status IN ('PROGRESS', 'SUCCESS')
          AND um.deleted_at IS NULL
          AND CONCAT(
                LPAD(m.point, 10, '0'), 
                DATE_FORMAT(um.created_at, '%Y%m%d%H%i%S'), 
                LPAD(um.id, 10, '0')
              ) < '0000000500202603251000000000000105'
        ORDER BY m.point DESC, um.created_at DESC, um.id DESC
        LIMIT 10;
        
        ```
        

- [x]  SQL Injection에 대해 조사하고 어떠할 때 일어나고 어떻게 막을 수 있는 지를 적어주세요
    - 설명
        
        # SQL Injection 공격
        
        해커가 웹사이트 입력창에 악의적인 SQL 구문 삽입해 DB를 마음대로 조작하는 공격 기법
        
        ```sql
        SELECT * FROM user WHERE username = '사용자ID' AND password = '비밀번호';
        ```
        
        원래 로그인 시에 위와 같은 로직을 사용했는데
        
        ```sql
        SELECT * FROM user WHERE username = '' OR '1'='1' -- ' AND password = '...';
        ```
        
        해커가 Id에 ‘ or ‘1’=’1 을 넣는다면 뒤 결과는 항상 참이 되어 Password가 맞지 않아도
        
        로그인되어버림
        
        # 해결 방법
        
        ### (1) prepared statement
        
        사용자 입력을 명령어가 아닌 단순 텍스트로 취급하게 만들기!
        
        ```sql
        where id = ?
        ```
        
        물음표 자리에 데이터 안전하게 채움
        
        ### (2) orm 사용
        
        orm은 내부적으로 인젝션 방어 처리가 되어있음
        
        ### (3) 입력값 검증
        
        사용자 입력이 DB 로직으로 들어가기 이전에 특수문자가 포함된 문자열은 검증 단계에서 막는 등의 처리가 필요하다.
        
    
- [x]  다양한 JOIN 방법들에 대해 찾아보고, 각 방식에 대해 비교하여 간단히 정리해주세요.
    - 설명
        
        ![image.png](attachment:1a672f3a-b254-48b9-934d-070de9d7c54f:image.png)
        
        ### inner join
        
        두 개의 테이블을 공통된 칼럼으로 결합하는데, 두 테이블에 모두 존재하는 데이터만 선택하는 조인
        
        카테시안 곱을 한 결과, n * m 개의 경우에서 [n.id](http://n.id) = [m.id](http://m.id) 인 것만 고르는 것과 같음
        
        ### left outer join
        
        두 개의 테이블을 결합할 때 왼쪽 테이블의 모든 행과 오른쪽 테이블의 일치하는 행을 선택하는 조인
        
        오른쪽 테이블에 일치하는 값이 없으면 null 반환
        
        ### cross join
        
        두 테이블의 카테시안 프로덕트이다. 모든 행의 조합을 반환하기 때문에 두 테이블 행의 수를 곱한 만큼의 행을 가진 결과 테이블이 나온다.
        
        ### full outer join
        
        두 테이블 결합 시 두 테이블 모든 행을 반환하는 조인. 한쪽 테이블에만 존재하는 행 경우 다른 테이블은 Null로 둔다.
        
        | **학생 ID** | **이름** |
        | --- | --- |
        | 1 | 김철수 |
        | 2 | 이영희 |
        | 3 | 박민수 |
        
        | **학생 ID** | **동아리명** |
        | --- | --- |
        | 1 | 축구부 |
        | 2 | 농구부 |
        | 4 | 밴드부 |
        
        id 기준으로 full outer join 한 결과는
        
        | **학생 ID** | **이름** | **동아리명** |
        | --- | --- | --- |
        | 1 | 김철수 | 축구부 |
        | 2 | 이영희 | 농구부 |
        | 3 | 박민수 | `NULL` |
        | 4 | `NULL` | 밴드부 |
        
        mysql에서는 full outer join을 제공하지 않아 두 번의 Left 조인을 하고 Union 을 통해 합쳐서 결과를 만들 수 있다.
        
    
    정리된 글을 바탕으로 블로그를 작성하여 주세요