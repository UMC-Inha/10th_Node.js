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