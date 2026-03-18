- [x]  일반(주니어) 미션에서 제외되었던 부분까지 ERD를 설계해 주세요. 지도 검색, 내 포인트 관리, 알림, 사장님의 자신의 점포 관리하는 부분이 포함됩니다.
    - 어떻게 설계하였는지 간단하게 정리하여 주세요. ERD 자체를 첨부해 주셔도 좋습니다.
        
        ![UMC_10th.png](attachment:56dd1763-ebae-4cda-bf48-d6ef5b3eaa15:UMC_10th.png)
        
- [x]  미션 자료로 제공된 피그마를 보고 ERD를 설계한 후 제 1,2,3 정규화를 통해 제 1,2,3 정규형을 만들고 각각 중복된 데이터가 어떻게 변화하였고 어떠한 이점이 있었는 지 작성하여 주세요
    - 내용
        
        정규화 이전
        
        | **mission_id** | **user_id** | **store_id** | **user_nickname** | **store_name** | **content** | **score** | url |
        | --- | --- | --- | --- | --- | --- | --- | --- |
        | 1 | 10 | 500 | 라이언 | 맥도날드 | 맛있어요 | 5 | 주소1 |
        | 2 | 10 | 501 | 라이언 | 스타벅스 | 친절해요 | 4 | 주소2, 주소3 |
        
        ### 제1정규형
        
        모든 칼럼은 원자값을 가져야한다는 원칙을 지키기 위해 중복 그룹 제거
        
        한 리뷰에 사진 url 여러개가 url1, url2 컴마로 한번에 들어있음
        
        2개의 row로 분리하기
        
        | **mission_id** | **user_id** | **store_id** | **user_nickname** | **store_name** | **content** | **score** | url |
        | --- | --- | --- | --- | --- | --- | --- | --- |
        | 1 | 10 | 500 | 라이언 | 맥도날드 | 맛있어요 | 5 | 주소1 |
        | 2 | 10 | 501 | 라이언 | 스타벅스 | 친절해요 | 4 | 주소2 |
        | 2 | 10 | 501 | 라이언 | 스타벅스 | 친절해요 | 4 | 주소3 |
        
        ### 제2정규형
        
        기본키가 복합키일때 부분함수적 종속성을 제거해야한다.
        
        예를 들어 store_name은 store_id에 종속된다.
        
        따라서 store 테이블과 user 테이블로 분리한다.
        
        ### 제3정규형
        
        이행적 함수적 종속성을 제거한다.
        
        예시에서는 없지만 store_category_name 또한 위 테이블에 있었다면 store_category 테이블을 별도로 분리해야한다. 
        

- [x]  피그마의 홈 부분에서 한 사람이 “미션 도전!” 버튼을 빠르게 여러 번 눌렀을 때 여러 가지 이유(비동기 로직 등)로 요청이 지연되어 완전히 처리하기 전 두 번 요청이 들어갈 수 있습니다. 이를 해결할 수 있는 방법에 대해 작성하여 주세요 (ERD 직접적으로 관련이 있기보다는 설계할 때 한번쯤 고민해보면 좋을 것 추가시켜 놓았습니다) (다양한 방법이 있으니 찾아봐 주세요)
    - 내용들을 간단하게 정리하여 주세요
        
        ### 프론트엔드에서 해결할 부분
        
        1. 미션도전 클릭시 미션도전 버튼을 비활성화하기
        2. 짧은 시간 이내에 발생하는 이벤트 하나로 묶어 마지막 요청만 보내기
        
        ### 백엔드에서 해결할 부분
        
        1. 데이터베이스 상에서 사용자 id, 미션 Id, 날짜 등을 조합하여 복합 유니크 키를 설정함으로써 DB 수준에서 에러를 뱉기
        2. Redis 분산 락 : 동시에 두 명이 들어오지마
            - 위의 문제와 같이 짧은 시간(2~3초) 동안 락을 걸고, 작업이 끝나면 바로 품
            - 만약 락이 아직 만료되지 않아 획득하지 못했다면 redis client에서 아무것도 반환하지 않으므로 아래와 같이 코드 수준에서 에러 발생시킴으로써 문제 해결
            - 해당 redis 클라이언트는 확장된 N개의 모든 서버에 대해 하나의 좌물쇠 역할을 함.
        
        ```tsx
        const redis = require('redis');
        const client = redis.createClient();
        
        async function challengeMission(userId, missionId) {
            const lockKey = `lock:mission:${userId}:${missionId}`;
            
            // 1. 락 획득 시도 (PX: 3000ms 동안만 유효, 만료 시간 설정은 필수!)
            // 성공 시 'OK' 반환, 이미 있으면 null 반환
            const acquired = await client.set(lockKey, 'locked', {
                NX: true,
                PX: 3000 
            });
        
            if (!acquired) {
                throw new Error("이미 처리 중인 요청입니다. 잠시 후 다시 시도해주세요.");
            }
        
            try {
                // 2. 비즈니스 로직 수행 (DB Insert 등)
                // const result = await db.query('INSERT INTO user_missions ...');
                return { success: true };
            } finally {
                // 3. 작업 완료 후 반드시 락 해제
                await client.del(lockKey);
            }
        }
        
        ```
        
        1. 멱등성 키 : 클라이언트 요청마다 고유한 uuid를 헤더에 담아 보내, 이 키 를 서버에서 일정 시간 저장해두고, 동일한 키 요청이 다시 들어올 경우 기존 저장된 응답 반환
            - Redis 캐시 사용
            - 멱등성이란 연산을 여러 번 수행해도 결과가 달라지지 않는 성질을 말함
        
        ```tsx
        // 클라이언트 코드
        // 1. 유일 키
        const idempotencyKey = uuidv4(); 
        
        async function challengeMission() {
          try {
            const response = await axios.post('/mission/challenge', 
              { missionId: 101 }, 
              { 
                headers: { 
                  // 2. 헤더에 담아서 전송
                  'Idempotency-Key': idempotencyKey 
                } 
              }
            );
            console.log("성공:", response.data);
          } catch (error) {
            // 네트워크 에러 등으로 실패해서 다시 호출해도 
            // 동일한 idempotencyKey를 쓰면 서버에서 중복 방지가 됨
            console.error("실패:", error);
          }
        }
        
        ```
        
        ```tsx
        async function handleMissionRequest(req, res) {
            const idempotencyKey = req.headers['idempotency-key'];
            const { userId, missionId } = req.body;
        
            if (!idempotencyKey) {
                return res.status(400).send("Idempotency-Key가 필요합니다.");
            }
        
            // 1. 기존 요청 확인
            const cachedResponse = await client.get(`idempotency:${idempotencyKey}`);
            if (cachedResponse) {
                return res.json(JSON.parse(cachedResponse)); // 동일 결과 즉시 반환
            }
        
            // 2. 로직 수행
            const result = { status: 'success', message: '미션 도전 완료!' };
            
            // 3. 결과 저장 (예: 24시간 동안 유효)
            await client.set(`idempotency:${idempotencyKey}`, JSON.stringify(result), {
                EX: 86400
            });
        
            return res.json(result);
        }
        
        ```
        
    


## 미션기록
