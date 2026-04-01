# 시니어 미션
# API 디자인 모범사례

https://learn.microsoft.com/ko-kr/azure/architecture/best-practices/api-design - 문서를 읽고 주요 내용을 간단히 정리해주세요.

# 1. 나는 이런 타입을 원해 : content-type과 accept

### content-type : 내가 보내는 데이터는 ~ 타입이야.

프론트엔드 서버에게 요청할 때 Content-Type: application/json

- 서버야 내가 보내는거 json이야~

서버에서 프론트로 응답할 때 

- 프론트야. 내가 보낸 데이터는 json 형식이야.

### accept: 나는 이 타입으로 받고싶어.

- 프론트엔드가 서버에게 데이터를 요청할 때 내가 받고 싶은 타입을 미리 지정해주는것

```tsx
fetch('https://example.com', {
  method: 'GET',
  headers: {
    'Accept': 'application/json'
  }
})
```

# 2. 너무 오래 걸리는 작업이라면?? → 비동기 메서드

경우에 따라서 post, put 같은 api는 완료하는데 시간이 걸릴 수 있다.

빠르게 끝나는 작업이면, 1초 이내로 200OK나 201 Created를 프론트에게 주면 되지만,

처리 시간이 길다면 비동기 패턴을 도입해야 할 수 있다.

방식은, 백엔드에서 대기 상태를 확인할 수 있는 주소를 202 상태코드와 함께 Location 헤더에 

담아서 반환한다.

```
HTTP/1.1 202 Accepted
Location: /api/status/12345
```

프론트는 GET /api/status/12345를 통해 현재 상황을 알 수 있다.

# 3. 대용량 전송 한 번에 하기 부담스러워 : 부분 응답 Accept-Ranges 헤더

 대용량 리소스를 전송할 때 불안정하고 일시적인 연결로 인해서 문제가 발생할 수 있다.

이를 해결하기 위해 큰 이진 리소스의 부분 검색을 지원하는 것이 추천된다.

이런 리소스에 대해서는 기존 GET 메소드가 아니라 HEAD 메소드를 사용한다.

1. 프론트는 HEAD 메소드를 통해서 대용량 리소스에 대한 정보를 백엔드에게 묻는다.
2. 서버 : 총 1GB이고, 부분 전송(Accept-Ranges) 가능하다는 답변 반환

```tsx
HTTP/1.1 200 OK

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
```

1. 프론트에서 해당 리소스가 너무 크다는 것을 인지하면, 앞 부분 1MB만 먼저 가져오겠다고 GET + range 헤더를 통해 데이터를 요청한다.

```tsx
GET https://api.contoso.com/products/10?fields=productImage
Range: bytes=0-2499
```

1. 백엔드는 이에 대한 답변으로 상태코드 206(부분 응답임을 나타냄)와 함께 데이터를 반환한다.

```tsx
HTTP/1.1 206 Partial Content

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580

[...]
```

# API 버저닝

실제로 서비스를 배포하다보면 서버를 항상 띄워놓고 새로운 버전의 API를 도입해야하는 경우가 있습니다. 이런 상황에서 어떤 전략을 사용할 수 있는지 고민해서 정리해주세요

# uri, 쿼리 버저닝

- https://example.com/v1
- ?version=1 파라미터 붙이기

주소창에 직접 버전을 명시하는 방식이다.

브라우저에서 바로 확인이 가능하고 캐싱이 쉽다.

다만, REST 원칙(같은 자원은 하나의 같은 주소를 갖는다.)에 벗어나는 모습을 가진다.

uri 방식

```tsx
// v1 라우터
v1Router.get('/user', (req, res) => res.send("v1 유저 정보"));
// v2 라우터
v2Router.get('/user', (req, res) => res.send("v2 유저 정보 (필드 추가됨)"));

app.use('/v1', v1Router);
app.use('/v2', v2Router);
```

```tsx
const apiV1 = axios.create({ baseURL:'https://api.com/v1' });
apiV1.get('/user');// 호출: https://api.com/user
```

쿼리 사용하는 경우

```tsx
app.get('/user', (req, res) => {
  const version = req.query.version;
  if (version ==='2')
    return res.send("v2 데이터");
  res.send("v1 데이터 (기본값)");
});
```

```tsx
axios.get('/user', { params: { version: '2' } });
```

# 헤더 버저닝

주소는 그대로 두고 http 요청 헤더에 버전 정보를 숨겨서 보내는 방식이다.

브라우저 주소창으로 테스트하긴 어렵고, postman이나 curl과 같은 도구를 사용해서 조회해야한다.

```tsx
app.get('/user', (req, res) => {
  const version = req.headers['x-api-version'];
  if (version === 'v1') 
    return res.send("v1");
  res.send("v2");
});
```

```tsx
const api = axios.create({
  headers: { 'x-api-version': 'v1' }
});
api.get('/user');
```

# 미디어 타입 버저닝

- Accept: application/vnd.example.v1+json

데이터 형식을 지정할 때 버전 정보도 같이 넣는 방식이다.

```tsx
app.get('/user', (req, res) => {
  const accept = req.headers['accept'];
  if (accept.includes('v2')) return res.json({ msg: "v2 응답" });
  res.json({ msg: "v1 응답" });
});
```

```tsx
axios.get('/user', {
  headers: { 'Accept': 'application/vnd.myapi.v2+json' }
});
```

# 멱등적 설계

네트워크 오류나 중복 요청에도 데이터가 꼬이지 않도록 어떻게 API를 설계할 수 있을까요?

 https://docs.tosspayments.com/blog/what-is-idempotency → 다음 문서를 보고 정리해주세요 

# 멱등성이란

첫 번째 수행을 한 뒤 여러 차례 적용해도 결과를 변형시키지 않는 작업을 말한다.

즉, 멱등성을 가진 작업의 결과는 몇 번을 수행하든 결과가 같다.

어떤 api가 멱등성이 있어야할까? 예를 들어 GET은 계속 같은 결과를 반환해주는 것이 정상일 것이다.

| **메서드** | **멱등성** |
| --- | --- |
| CONNECT | X |
| DELETE | O |
| GET | O |
| HEAD | O |
| OPTIONS | O |
| POST | X |
| PUT | O |
| PATCH | X |
| TRACE | O |
- 멱등한 delete /users/123
- 멱등하지 않은 delete /user/last

delete는 첫 번째 삭제는 200, 두 번째 삭제는 404아닌가?

http에서의 멱등성은 응답 내용(body)이나 상태코드가 아니라 서버에 남겨진 리소스의 상태를 기준으로 판단한다.

# 멱등한 요청인지 아는 법

저번 워크북에서 중복 요청에 대한 대처법 중에서 멱등성키를 이용하는 방법이 있었다.

그때는 캐시에 1초 저장하는 방법을 사용했는데, 이와 같이 idempotency DB를 따로 마련하여 동일한 요청에 대해서는 유일한 키(멱등키)로 식별하여 동일한 응답을 반환하도록 한다.

```tsx
// 프론트 측에서 멱등키 생성
let idempotentKey = generateUUIDv4()
function async cancelPayment(idempotencyKey: string) {
  try {
    return await axios.post("https://myshop/cancel-payment",
      {
        orderId: UINQUE_ORDER_ID
        amount: 100,
      },
      {
        headers: {
          "Idempotency-Key": idempotentKey // 헤더에 멱등키를 추가합니다.
        }
      }
    )
  } catch(e) {
    if (e.name === "TIMEOUT") { // 타임아웃이 일어났을 때 같은 요청을 보낼 수 있습니다.
        return await cancelPayment(idempotencyKey)
    }
    console.error("ERROR")
  }
}
const response = await cancelPayment(idempotentKey);
```

서버측에서는 멱등성DB에 멱등키와 그에 해당하는 요청기록을 추가하고, 처리에 성공했다면 응답을 돌려준다.

같은 요청이 반복되면 요청에 멱등키가 포함되어있는지, 있다면 이미 저장된 값을 반환한다.

```tsx
const idempotencyResponses = new Map();
let cancelReq = {
  orderId: req.body.orderId
  amount: req.body.amount,
};
let idempotencyKey = req.headers.idempotencyKey || null // 요청 헤더에서 멱등키를 가져옵니다.
// 멱등키가 있고 멱등 응답도 저장되어 있다면 실제 처리하지 않고 저장된 응답을 내보냅니다.
if (idempotencyKey != null && idempotencyResponses.has(idempotencyKey)) {
  const response = idempotencyResponses.get(idempotencyKey);
  return res.status(response.status).json(response);
};
const result = cancelProcessor.cancel(cancelReq); // 실제로 취소를 처리합니다.
// 멱등키가 있으면 멱등응답을 저장합니다.
if (idempotencyKey != null) {
  idempotencyResponses.set(idempotencyKey, result);
}
const responseBody = {
  message: `결제 취소 성공`,
};
return res.status(200).json(responseBody);
```

# 멱등성 미들웨어

모즌 api에 멱등성을 적용하기 위에 위 코드를 복붙하면 코드가 너무 길어지니, 미들웨어나 데코레이터 형태로 공통 컴포넌트로 분리하는 것이 권장된다.

아래 예시 코드에서는 레디스 사용함.

```tsx
const Redis = require('ioredis');
const redis = new Redis(); // Redis 연결 (기본 로컬호스트)

const idempotencyMiddleware = async (req, res, next) => {
  const idempotencyKey = req.headers['idempotency-key'];

  // 1. 멱등키가 없으면 그냥 통과
  if (!idempotencyKey) {
    return next();
  }

  try {
    // 2. Redis에서 해당 키로 저장된 응답이 있는지 확인
    const cachedResponse = await redis.get(`idempotency:${idempotencyKey}`);

    if (cachedResponse) {
      const parsedResponse = JSON.parse(cachedResponse);
      console.log("Redis 캐시 응답 반환:", idempotencyKey);
      return res.status(parsedResponse.status).json(parsedResponse.body);
    }

    // 3. 중복 요청 방지 (Lock 기능: 현재 처리 중임을 표시)
    // 'nx': 키가 없을 때만 설정, 'ex': 10초 후 자동 삭제 (처리 중 서버 다운 대비)
    const lock = await redis.set(`lock:${idempotencyKey}`, 'processing', 'EX', 10, 'NX');
    if (!lock) {
      return res.status(409).json({ message: "이미 처리 중인 요청입니다." });
    }

    // 4. 응답 가로채기 (res.json 오버라이딩)
    const originalJson = res.json;
    res.json = async function (body) {
      // 비즈니스 로직 완료 후 결과를 Redis에 저장 (24시간 동안 보관)
      const responseData = {
        status: res.statusCode,
        body: body
      };
      
      await redis.set(`idempotency:${idempotencyKey}`, JSON.stringify(responseData), 'EX', 86400);
      await redis.del(`lock:${idempotencyKey}`); // 처리가 끝났으므로 락 해제
      
      return originalJson.call(this, body);
    };

    next();
  } catch (error) {
    console.error("멱등성 체크 중 오류:", error);
    next(); // 오류 시 로직은 진행시키되 로그 남김
  }
};

module.exports = idempotencyMiddleware;
```

여러 api에서 사용하기

```tsx
const idempotency = require('./idempotencyMiddleware');

// 결제 취소 API (Postman 등에서 Header에 'Idempotency-Key'를 넣어서 테스트)
router.post('/payments/cancel', idempotency, async (req, res) => {
  // 실제 결제 취소 로직...
  const result = { success: true, transactionId: "TX_12345" }; // 더미 데이터
  res.status(200).json(result);
});
```