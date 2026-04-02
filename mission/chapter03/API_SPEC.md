# GET /auth/login/{social}

- social 경로를 통해 플랫폼 구분
- 회원가입을 하지 않은 상태라면 유저를 생성하고, 추가 정보 입력 페이지로 리다이렉트

작동 원리

1. 프론트엔드에서 구글로 로그인하기 버튼 클릭 → 구글 로그인 화면

![image.png](attachment:a98a50c1-5739-4a4b-bb08-b523c6631fd8:image.png)

1. 구글 로그인 화면에서 백엔드 주소로 리다이렉트 → 로그인 및 회원가입 로직 수행
2. 자체의 refreshToken을 만들어서 헤더에 담은 후 프론트로 리다이렉트 요청
3. 프론트에서 리프레시토큰을 통해 accessToken을 발급받고 서비스 이용 시작

# Request

request header

- content-type : application/json
- Authorization: Bearer ejalskdjflkj…

```json
{
  "fcm_token": "abcd..."
}
```

# Response

<aside>

성공

```json
http/1.1 302 found
location: https://프론트주소.com/google-redirect
set-cookie: refreshToken=abc...; httpOnly
```

</aside>

<aside>

실패

```json
http/1.1 302 found
location: https://프론트-로그인-실패-주소.com
```

</aside>

# POST /auth/restore-access

- 구글/소셜 로그인 이후, 프론트엔드에서 전달받은 refreshToken을 사용해 실제 서비스 이용에 필요한 accessToken을 발급받기 위한 api

작동 원리

1. 사용자 로그인 완료 후 백엔드로 리다이렉트
2. 백엔드에서 refreshToken을 생성해 httpOnly 쿠키에 저장 후 프론트로 리다이렉트
3. 프론트엔드 진입 시 인증이 필요한 api 호출이 필요하다면 유효한 accessToken을 요청하여 응답받음

# Request

request header

- content-type : application/json
- Authorization: Bearer ejalskdjflkj…

```json
{
  "fcm_token": "abcd..."
}
```

# Response

<aside>

성공

- isNew 파라미터를 통해 프론트 측에서 추가 정보 입력 페이지 혹은 메인화면으로 이동하도록 함

```json
http/1.1 302 found
location: https://프론트주소.com/google-redirect?isNew=true
set-cookie: refreshToken=abc...; httpOnly
```

</aside>

<aside>

실패

```json
http/1.1 302 found
location: https://프론트-로그인-실패-주소.com
```

</aside>

# GET /users/my

![image.png](attachment:2a56834c-d0fc-41cb-add0-8d5100c90d3c:image.png)

- 로그인한 사용자 현재 정보를 불러와 화면에 표시

# Request

request header

- content-type : application/json
- Authorization: Bearer ejalskdjflkj…

# Response

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "성공입니다.",
  "result": {
    "id": 1,
    "nickname": "코딩고양이",
    "email": "user@example.com",
    "is_phone_verified": true,
    "profile_url": "https://image.com",
    "role": "USER",
    "point": 2500
  }
}
```

</aside>

<aside>

실패

```json
{
  "isSuccess": false,
  "code": "NOT_FOUND_USER",
  "message": "해당 아이디를 가진 유저가 없습니다.",
  "result": null
}
```

</aside>

# PATCH /users/my

- 로그인한 사용자의 정보를 수정하는 api

# Request

request header

- content-type : application/json
- Authorization: Bearer ejalskdjflkj…

request body

| **필드명** | **타입** | **필수여부** | **Nullable** | **설명** |
| --- | --- | --- | --- | --- |
| `nickname` | String | 선택 | No | 변경할 닉네임 (최대 10자) |
| `gender` | Enum | 선택 | No | 성별 (`MALE`, `FEMALE`, `NONE`) |
| `birth` | String | 선택 | Yes | 생년월일 (형식: `YYYY-MM-DD`) |
| `phone_number` | String | 선택 | Yes | 전화번호 (하이픈 제외 11자 숫자) |
| `address_doro` | String | 선택 | Yes | 도로명 주소 (최대 20자) |
| `address_detail` | String | 선택 | Yes | 상세 주소 (최대 20자) |
| `profile_url` | String | 선택 | Yes | 프로필 이미지 URL (S3 등 업로드 후 경로) |
| `inquiry_answer_on` | Integer | 선택 | No | 1: 알림 켬, 0: 알림 끔 (1:1문의 답변) |
| `review_on` | Integer | 선택 | No | 1: 알림 켬, 0: 알림 끔 (리뷰 관련) |
| `event_on` | Integer | 선택 | No | 1: 알림 켬, 0: 알림 끔 (이벤트/마케팅) |
| `categoryIds` | Array | 선택 | No | 선호 음식 카테고리 ID 리스트 (예: `[1, 5]`) |
|  |  |  |  |  |

<aside>

요청 바디 예시

```json
{
  "nickname": "맛있으면짖는개",
  "gender": "MALE",
  "birth": "1995-05-20",
  "phone_number": "01012345678",
  "address_doro": "서울시 강남구 테헤란로 1",
  "address_detail": "본관 302호",
  "profile_url": "https://bucket.com",
  "inquiry_answer_on": 1,
  "review_on": 1,
  "event_on": 0,
  "categoryIds": [1, 5, 10]
}
```

</aside>

# Response

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "프로필이 성공적으로 수정되었습니다.",
  "result": null
}
```

</aside>

<aside>

실패

```json
{
  "isSuccess": false,
  "code": "NOT_FOUND_USER",
  "message": "해당 아이디를 가진 유저가 없습니다.",
  "result": null
}
```

400 Bad Request

```json
{
  "isSuccess": false,
  "code": "USER_BAD_REQUEST",
  "message": "닉네임은 10자를 초과할 수 없습니다.",
  "result": null
}
```

</aside>

# PATCH /users/my

- 서비스 탈퇴를 위한 API
- soft delete 삭제

# Request

request header

- content-type : application/json
- Authorization: Bearer ejalskdjflkj…

# Response

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "탈퇴가 정상적으로 수행되었습니다.",
  "result": {
		 "userId": 1,
     "deletedAt": "2026-04-02T17:05:00"
  }
}
```

</aside>

<aside>

실패 - 이미 탈퇴한 유저

```json
{
  "isSuccess": false,
  "code": "USER_NOT_FOUND",
  "message": "이미 탈퇴했거나 존재하지 않는 사용자입니다.",
  "result": null
}
```

401 Unauthorized

```json
{
  "isSuccess": false,
  "code": "UNAUTHORIZED",
  "message": "인증 정보가 유효하지 않습니다.",
  "result": null
}

```

</aside>

# GET /missions/my?status=progress

![image.png](attachment:a815a16d-9c11-42de-b198-1d8ccf17704f:image.png)

- 사용자가 도전중인 미션 목록을 최신순으로 조회합니다.
- 페이지네이션 방식, 커서 기반으로 구현되어습니다.
- bigint id 기반으로 다음 페이지를 호출합니다.

# Request

request header

- content-type : application/json
- Authorization: Bearer ejalskdjflkj…

request parameter

| **필드명** | **타입** | **필수여부** | **설명** |
| --- | --- | --- | --- |
| **`lastId`** | Long (bigint) | 선택 | 이전에 받았던 마지막 미션의 `id` (첫 요청 시에는 사용하지 않음) |
| **`size`** | Integer | 선택 | 한 페이지당 가져올 개수 (기본값: 10) |

# Response

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "미션 목록 조회에 성공했습니다.",
  "result": {
    "missionList": [
      {
        "memberMissionId": 150,
        "missionId": 22,
        "storeName": "치킨나라",
        "price": 15000,
        "point": 1000,
        "createdAt": "2026-04-02T10:00:00"
      },
      {
        "memberMissionId": 145,
        "missionId": 18,
        "storeName": "피자마을",
        "price": 25000,
        "point": 2000,
        "createdAt": "2026-04-01T15:30:00"
      }
    ],
    "lastId": 145,
    "hasNext": true
  }
}

```

</aside>

<aside>

실패

```json
{
  "isSuccess": false,
  "code": "NOT_FOUND_USER",
  "message": "해당 아이디를 가진 유저가 없습니다.",
  "result": null
}
```

요청 파라미터 타입 불일치

```json
{
  "isSuccess": false,
  "code": "PARAMETER_BAD_REQUEST",
  "message": "lastId 파라미터 형식이 올바르지 않습니다.",
  "result": null
}
```

</aside>

# GET /missions/my?status=success

![image.png](attachment:a815a16d-9c11-42de-b198-1d8ccf17704f:image.png)

- 사용자가 도전중인 미션 목록을 최신순으로 조회합니다.
- 페이지네이션 방식, 커서 기반으로 구현되어습니다.
- bigint id 기반으로 다음 페이지를 호출합니다.

# Request

request header

- content-type : application/json
- Authorization: Bearer ejalskdjflkj…

request parameter

| **필드명** | **타입** | **필수여부** | **설명** |
| --- | --- | --- | --- |
| **`lastId`** | Long (bigint) | 선택 | 이전에 받았던 마지막 미션의 `id` (첫 요청 시에는 사용하지 않음) |
| **`size`** | Integer | 선택 | 한 페이지당 가져올 개수 (기본값: 10) |

# Response

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "미션 목록 조회에 성공했습니다.",
  "result": {
    "missionList": [
      {
        "memberMissionId": 150,
        "missionId": 22,
        "storeName": "치킨나라",
        "price": 15000,
        "point": 1000,
        "createdAt": "2026-04-02T10:00:00"
      },
      {
        "memberMissionId": 145,
        "missionId": 18,
        "storeName": "피자마을",
        "price": 25000,
        "point": 2000,
        "createdAt": "2026-04-01T15:30:00"
      }
    ],
    "lastId": 145,
    "hasNext": true
  }
}

```

</aside>

<aside>

실패

```json
{
  "isSuccess": false,
  "code": "NOT_FOUND_USER",
  "message": "해당 아이디를 가진 유저가 없습니다.",
  "result": null
}
```

요청 파라미터 타입 불일치

```json
{
  "isSuccess": false,
  "code": "PARAMETER_BAD_REQUEST",
  "message": "lastId 파라미터 형식이 올바르지 않습니다.",
  "result": null
}
```

</aside>

# GET /missions/new

![image.png](attachment:8407f7bd-edb1-4f7e-a155-47f4001806ad:image.png)

- 현재 사용자가 도전 혹은 성공한 미션이 아닌, 새로운 미션을 조회합니다.

# Request

request header

- content-type : application/json
- Authorization: Bearer ejalskdjflkj…

request parameter

| **필드명** | **타입** | **필수여부** | **설명** |
| --- | --- | --- | --- |
| **`lastId`** | Long (bigint) | 선택 | 이전에 받았던 마지막 미션의 `id` (첫 요청 시에는 사용하지 않음) |
| **`size`** | Integer | 선택 | 한 페이지당 가져올 개수 (기본값: 10) |

# Response

| **필드명** | **타입** | **필수여부** | **Nullable** | **설명** |
| --- | --- | --- | --- | --- |
| `missionId` | Long | Yes | No | 미션 고유 ID (커서 기준값) |
| `storeName` | String | Yes | No | 가게 이름 (예: 반이학생마라탕) |
| `categoryName` | String | Yes | No | 음식 카테고리 (예: 중식당) |
| `priceCondition` | Integer | Yes | No | 달성 조건 금액 (예: 10,000) |
| `rewardPoint` | Integer | Yes | No | 적립 포인트 (예: 500) |
| `dDay` | String | Yes | No | 남은 기간 (예: "D-7") |
| **`lastId`** | Long | Yes | Yes | 현재 페이지의 마지막 ID (다음 요청의 커서) |
| **`hasNext`** | Boolean | Yes | No | 다음 페이지 존재 여부 |

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "COMMON200",
  "message": "성공입니다.",
  "result": {
    "missionList": [
      {
        "missionId": 500,
        "storeName": "반이학생마라탕",
        "categoryName": "중식당",
        "priceCondition": 10000,
        "rewardPoint": 500,
        "dDay": "D-7"
      },
      {
        "missionId": 495,
        "storeName": "김밥천국",
        "categoryName": "분식",
        "priceCondition": 8500,
        "rewardPoint": 300,
        "dDay": "D-3"
      }
    ],
    "lastId": 495,
    "hasNext": true
  }
}

```

</aside>

<aside>

실패

```json
{
  "isSuccess": false,
  "code": "NOT_FOUND_USER",
  "message": "해당 아이디를 가진 유저가 없습니다.",
  "result": null
}
```

</aside>

# PATCH /missions/{mission_id}/success

![image.png](attachment:f6a6b0de-eacb-402b-b22b-232154372db6:image.png)

- 특정 미션에 대해 도전을 시작합니다.
- user_mission 테이블의 status 필드를 PROGRESS로 변경

# Request

request header

- content-type : application/json
- Authorization: Bearer ejalskdjflkj…

# Response

| **필드명** | **타입** | **필수여부** | **Nullable** | **설명** |
| --- | --- | --- | --- | --- |
| `memberMissionId` | Long | Yes | No | 생성된 도전 내역의 고유 ID |
| `missionId` | Long | Yes | No | 도전 시작한 미션 ID |
| `status` | String | Yes | No | 현재 상태 (예: `CHALLENGING`) |
| `startedAt` | String | Yes | No | 도전 시작 일시 |

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "미션 도전을 시작했습니다!",
  "result": {
    "memberMissionId": 120,
    "missionId": 5,
    "status": "CHALLENGING",
    "startedAt": "2026-04-02T17:30:00"
  }
}

```

</aside>

<aside>

이미 도전중인 미션

```json
{
  "isSuccess": false,
  "code": "MISSION_ALREADY_DOING",
  "message": "이미 도전 중인 미션입니다.",
  "result": null
}
```

</aside>

# GET /missions/{mission_id}/request-complete

![image.png](attachment:c8337c0c-2dbb-4ed9-90e9-63ea7a906d41:image.png)

- 특정 미션을 완료하기 위해 사장님에게 보여드릴 인증번호를 발급받는 api입니다.

# Request

request header

- content-type : application/json
- Authorization: Bearer ejalskdjflkj…

# Response

| **필드명** | **타입** | **필수여부** | **Nullable** | **설명** |
| --- | --- | --- | --- | --- |
| **`verificationNumber`** | String | Yes | No | **사장님 구분 번호 (예: "920394810")** |
| `missionId` | Long | Yes | No | 대상 미션 ID |
| `expiresAt` | String | No | Yes | 번호 유효 시간 (예: "2026-04-02T17:30:00") |

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "인증번호가 발급되었습니다.",
  "result": {
    "missionId": 12,
    "verificationNumber": "920394810",
    "expiresAt": "2026-04-02T17:25:00"
  }
}
```

</aside>

<aside>

도전중인 미션 아님 400

```json
{
  "isSuccess": false,
  "code": "NOT_MY_MISSION",
  "message": "도전 중인 미션 리스트에 존재하지 않습니다. 먼저 미션에 도전하세요.",
  "result": null
}

```

이미 보상이 완료된 미션

```json
{
  "isSuccess": false,
  "code": "MISSION4004",
  "message": "이미 보상 지급이 완료된 미션입니다.",
  "result": null
}
```

미션 기간 만료

```json
{
  "isSuccess": false,
  "code": "MISSION4002",
  "message": "해당 미션은 종료 기간이 지나 인증번호를 발급할 수 없습니다.",
  "result": null
}
```

</aside>

# PATCH /missions/{mission_id}/success

![image.png](attachment:c8337c0c-2dbb-4ed9-90e9-63ea7a906d41:image.png)

- 사용자가 화면에 표시된 인증번호를 확인하고 미션 미션 완료 버튼을 눌러 미션을 최종적으로 성공하도록 처리합니다.

# Request

request header

- content-type : application/json
- Authorization: Bearer ejalskdjflkj…

request body

```json
{
  "verificationNumber": "920394810"
}
```

# Response

| **필드명** | **타입** | **필수여부** | **Nullable** | **설명** |
| --- | --- | --- | --- | --- |
| `missionId` | Long | Yes | No | 완료된 미션 ID |
| `rewardPoint` | Integer | Yes | No | 획득한 보상 포인트 |
| `currentTotalPoint` | Integer | Yes | No | 포인트 지급 후 유저의 총 포인트 |
| `completedAt` | String | Yes | No | 완료 처리 일시 (ISO 8601) |

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "미션 성공! 포인트 적립이 완료되었습니다.",
  "result": {
    "missionId": 12,
    "rewardPoint": 500,
    "currentTotalPoint": 1500,
    "completedAt": "2026-04-02T17:15:30"
  }
}
```

</aside>

<aside>

인증번호 일치하지 않음 실패

```json
{
  "isSuccess": false,
  "code": "MISSION4005",
  "message": "인증번호가 일치하지 않습니다. 다시 확인해주세요.",
  "result": null
}
```

</aside>

# GET /stores/{store_id}

![image.png](attachment:b8edc1f9-1434-4801-89a0-66884be35afc:image.png)

- 사용자가 화면에 표시된 인증번호를 확인하고 미션 미션 완료 버튼을 눌러 미션을 최종적으로 성공하도록 처리합니다.

# Response

| **필드명** | **타입** | **필수여부** | **Nullable** | **설명** |
| --- | --- | --- | --- | --- |
| `storeId` | Long | Yes | No | 가게 고유 ID |
| `name` | String | Yes | No | 가게 이름 (예: "반이학생마라탕마라반") |
| `categoryName` | String | Yes | No | 음식 카테고리 (예: "중식당") |
| `isOpened` | Boolean | Yes | No | 영업 상태 (true: 영업 중, false: 영업 종료) |
| `score` | Double | Yes | No | 평균 별점 (예: 4.4) |
| `address` | String | Yes | No | 가게 지번/도로명 주소 (이미지 기준) |
| `storeImageUrl` | String | No | Yes | 가게 대표 이미지 URL 리스트 |

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "가게 정보 조회 성공",
  "result": {
    "storeId": 1,
    "name": "반이학생마라탕마라반",
    "categoryName": "중식당",
    "isOpened": false,
    "score": 4.4,
    "address": "서울시 성북구 안암동5가 102-60",
    "storeImages": [
      "https://bucket.com",
      "https://bucket.com"
    ]
  }
}

```

</aside>

<aside>

존재하지 않는 가게 404

```json
{
  "isSuccess": false,
  "code": "STORE4001",
  "message": "해당 가게 정보를 찾을 수 없습니다.",
  "result": null
}
```

</aside>

# GET /stores/{store_id}/reviews

![image.png](attachment:0256841b-4ce9-4a3d-a86e-c6f50ce3dac8:image.png)

- 특정 가게에 작성된 리뷰들을 최신순으로 조회합니다.
- 커서기반 페이지네이션이 사용됩니다.

# Request

request header

- content-type: application/json

request query param

| **필드명** | **타입** | **필수여부** | **설명** |
| --- | --- | --- | --- |
| **`lastId`** | Long | 선택 | 직전 페이지의 마지막 `reviewId` (첫 요청 시 비움) |
| **`size`** | Integer | 선택 | 한 번에 가져올 리뷰 개수 (기본값: 10) |

# Response

| **필드명** | **타입** | **필수여부** | **Nullable** | **설명** |
| --- | --- | --- | --- | --- |
| `reviewId` | Long | Yes | No | 리뷰 고유 ID (커서 기준값) |
| `nickname` | String | Yes | No | 작성자 닉네임 |
| `score` | Double | Yes | No | 부여한 별점 (예: 4.5) |
| `content` | String | Yes | No | 리뷰 본문 내용 |
| `reviewImages` | Array | No | Yes | 리뷰에 첨부된 이미지 URL 리스트 |
| `createdAt` | String | Yes | No | 리뷰 작성 일시 |
| **`lastId`** | Long | Yes | Yes | 현재 페이지의 마지막 ID (다음 요청의 커서) |
| **`hasNext`** | Boolean | Yes | No | 다음 페이지 존재 여부 |

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "리뷰 목록 조회 성공",
  "result": {
    "reviewList": [
      {
        "reviewId": 120,
        "nickname": "맛있으면짖는개",
        "score": 5.0,
        "content": "마라탕 국물이 끝내줘요! 재료도 신선합니다.",
        "reviewImages": [
          "https://bucket.com",
          "https://bucket.com"
        ],
        "createdAt": "2026-04-02T12:00:00"
      },
      {
        "reviewId": 115,
        "nickname": "안암동미식가",
        "score": 4.0,
        "content": "조금 맵긴 한데 맛있게 먹었습니다.",
        "reviewImages": [],
        "createdAt": "2026-04-01T18:30:00"
      }
    ],
    "lastId": 115,
    "hasNext": true
  }
}

```

</aside>

<aside>

존재하지 않는 가게 404

```json
{
  "isSuccess": false,
  "code": "STORE4001",
  "message": "해당 가게 정보를 찾을 수 없습니다.",
  "result": null
}
```

</aside>

# GET /home

![image.png](attachment:2f173d38-b4ad-4fba-8b25-6636fe9e6fb2:image.png)

- 현재 달성된 미션 개수와 지역, 포인트 3가지 정보를 불러옵니다.

# Request

request header

- content-type: application/json
- authorization: Bearer {token}

# Response

| **필드명** | **타입** | **필수여부** | **Nullable** | **설명** |
| --- | --- | --- | --- | --- |
| `regionName` | String | Yes | No | 현재 설정된 지역명 (예: "안암동") |
| `currentPoint` | Long | Yes | No | 유저가 보유한 총 포인트 (예: 999999) |
| `completedCount` | Integer | Yes | No | 현재까지 달성한 미션 개수 (예: 7) |
| `totalGoalCount` | Integer | Yes | No | 미션 달성 목표 개수 (예: 10) |

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "COMMON200",
  "message": "홈 정보 조회 성공",
  "result": {
    "regionName": "안암동",
    "currentPoint": 999999,
    "completedCount": 7,
    "totalGoalCount": 10,
    "rewardPoint": 1000
  }
}
```

</aside>

<aside>

토큰 만료

```json
{
  "isSuccess": false,
  "code": "AUTH_EXPIRED",
  "message": "로그인 세션이 만료되었습니다. 다시 로그인해주세요.",
  "result": null
}
```

</aside>

# POST /missions/{mission_id}/reviews

![image.png](attachment:62025d73-a58a-46fb-bfca-80ec08c75736:image.png)

- 특정 미션에 대한 리뷰를 작성합니다. 텍스트, 별점, 사진(최대 3장)을 함께 업로드합니다.

# Request

request header

- content-type: application/json
- authorization: Bearer {token}

request form

| **필드명** | **타입** | **필수여부** | **설명** |
| --- | --- | --- | --- |
| `score` | Double | Yes | 별점 (예: 5.0) |
| `content` | String | Yes | 리뷰 내용 (최대 255자) |
| `reviewImages` | File (List) | No | 첨부 사진 파일 (최대 3장) |

# Response

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "201",
  "message": "리뷰 등록에 성공하였습니다.",
  "result": {
    "reviewId": 501,
    "nickname": "맛있으면짖는개",
    "score": 5.0,
    "createdAt": "2026-04-02T17:35:00"
  }
}

```

</aside>

<aside>

400 Bad request 사진 개수 초과

```json
{
  "isSuccess": false,
  "code": "REVIEW_IMAGE_FULL",
  "message": "리뷰 사진은 최대 3장까지만 첨부 가능합니다.",
  "result": null
}
```

400 필수 입력 필드 누락

```json
{
  "isSuccess": false,
  "code": "REVIEW_REQURED_FIELD",
  "message": "별점과 리뷰 내용은 필수 입력 사항입니다.",
  "result": null
}
```

</aside>

# PATCH /reviews/{review_id}

![image.png](attachment:62025d73-a58a-46fb-bfca-80ec08c75736:image.png)

- 작성한  리뷰를 수정합니다.
- 본인이 작성한 리뷰만 수정 가능합니다.

# Request

request header

- content-type: application/json
- authorization: Bearer {token}

request form

| **필드명** | **타입** | **필수여부** | **설명** |
| --- | --- | --- | --- |
| `score` | Double | Yes | 별점 (예: 5.0) |
| `content` | String | Yes | 리뷰 내용 (최대 255자) |
| `reviewImages` | File (List) | No | 첨부 사진 파일 (최대 3장) |

# Response

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "리뷰가 수정되었습니다.",
  "result": {
    "reviewId": 501,
    "score": 4.0,
    "updatedAt": "2026-04-02T17:45:00"
  }
}
```

</aside>

<aside>

403 Forbidden 권한 없음

```json
{
  "isSuccess": false,
  "code": "REVIEW4004",
  "message": "본인이 작성한 리뷰만 수정할 수 있습니다.",
  "result": null
}
```

400 Bad request 사진 개수 초과

```json
{
  "isSuccess": false,
  "code": "REVIEW_IMAGE_FULL",
  "message": "리뷰 사진은 최대 3장까지만 첨부 가능합니다.",
  "result": null
}
```

400 필수 입력 필드 누락

```json
{
  "isSuccess": false,
  "code": "REVIEW_REQURED_FIELD",
  "message": "별점과 리뷰 내용은 필수 입력 사항입니다.",
  "result": null
}
```

</aside>

# PATCH /reviews/{review_id}

- 작성한 리뷰를 삭제처리합니다.
- 이때 소프트삭제합니다.

# Request

request header

- content-type : application/json
- Authorization: Bearer ejalskdjflkj…

# Response

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "리뷰가 삭제되었습니다.",
  "result": {
    "reviewId": 501,
    "deletedAt": "2026-04-02T17:40:00"
  }
}
```

</aside>

<aside>

403 권한없음

```json
{
  "isSuccess": false,
  "code": "FORBIDDEN",
  "message": "본인이 작성한 리뷰만 삭제할 수 있습니다.",
  "result": null
}
```

401 Unauthorized

```json
{
  "isSuccess": false,
  "code": "UNAUTHORIZED",
  "message": "인증 정보가 유효하지 않습니다.",
  "result": null
}

```

</aside>

# GET /reviews/my

- 사용자가 작성한 리뷰 목록을 최신순으로 조회합니다. (삭제되지 않은 리뷰만 조회)
- 페이지네이션 방식, 커서 기반으로 구현되어습니다.
- bigint id 기반으로 다음 페이지를 호출합니다.

# Request

request header

- content-type : application/json
- Authorization: Bearer ejalskdjflkj…

request parameter

| **필드명** | **타입** | **필수여부** | **설명** |
| --- | --- | --- | --- |
| **`lastId`** | Long (bigint) | 선택 | 이전에 받았던 마지막 미션의 `id` (첫 요청 시에는 사용하지 않음) |
| **`size`** | Integer | 선택 | 한 페이지당 가져올 개수 (기본값: 10) |

# Response

| **필드명** | **타입** | **필수여부** | **Nullable** | **설명** |
| --- | --- | --- | --- | --- |
| `reviewId` | Long | Yes | No | 리뷰 고유 ID (커서 기준값) |
| `storeName` | String | Yes | No | 리뷰를 작성한 가게 이름 |
| `score` | Double | Yes | No | 부여한 별점 (0.5 ~ 5.0) |
| `content` | String | Yes | No | 리뷰 본문 내용 |
| `reviewImages` | Array | No | Yes | 리뷰에 첨부된 이미지 URL 리스트 |
| `createdAt` | String | Yes | No | 리뷰 작성 일시 |
| **`lastId`** | Long | Yes | Yes | 현재 페이지의 마지막 ID (다음 요청의 커서) |
| **`hasNext`** | Boolean | Yes | No | 다음 페이지 존재 여부 |

<aside>

성공

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "내 리뷰 목록 조회 성공",
  "result": {
    "reviewList": [
      {
        "reviewId": 501,
        "storeName": "반이학생마라탕",
        "score": 5.0,
        "content": "사장님이 친절하시고 마라탕도 아주 맛있어요!",
        "reviewImages": [
          "https://bucket.com"
        ],
        "createdAt": "2026-04-02T12:00:00"
      },
      {
        "reviewId": 480,
        "storeName": "김밥천국",
        "score": 4.0,
        "content": "간단하게 한 끼 먹기 좋네요.",
        "reviewImages": [],
        "createdAt": "2026-03-25T18:30:00"
      }
    ],
    "lastId": 480,
    "hasNext": true
  }
}

```

작성한 에러가 없는 경우

```json
{
  "isSuccess": true,
  "code": "200",
  "message": "성공입니다.",
  "result": {
    "reviewList": [],
    "lastId": null,
    "hasNext": false
  }
}
```

</aside>

<aside>

400 파라미터 형식 오류

```json
{
  "isSuccess": false,
  "code": "BAD_PARAM",
  "message": "lastId 또는 size 형식이 올바르지 않습니다.",
  "result": null
}
```

</aside>