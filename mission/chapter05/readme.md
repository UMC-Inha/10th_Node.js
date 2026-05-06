1-1. 특정 지역에 가게 추가하기 API

```tsx
app.post("/api/v1/stores", async (req: Request, res: Response) => {
  try {
    const { name, city, district, neighborhood } = req.body;

    // 필수값 검증
    if (!name || !city || !district || !neighborhood || !neighborhood) {
      return res.status(400).json({
        success: false,
        code: 400,
        message: "잘못된 요청입니다.",
        error: { detail: "필수 값 누락" }
      });
    }

    // 가게 생성
    const store = await db.store.create({
      data: {
       	name,
        	city,
       	district,
       	neighborhood,
	detail
      }
    });

    // 성공 응답
    return res.status(201).json({
      success: true,
      code: 201,
      message: "가게 생성 성공",
      data: {
        storeId: store.id
      }
    });

		// 실패 응답
  } catch (err) {
    console.error(err);
    return res.status(500).json({
      success: false,
      code: 500,
      message: "서버 에러",
      error: { detail: "unexpected error" }
    });
  }
});
```

**1-2. 가게에 리뷰 추가하기 API**

```tsx
app.post("/api/v1/reviews", async (req: Request, res: Response) => {
  try {
    const { userId, storeId, rating, content, userMissionId, images } = req.body;

    // 가게 존재 확인
    const store = await db.store.findUnique({
      where: { id: storeId }
    });

    if (!store) {
      return res.status(404).json({
        success: false,
        code: 404,
        message: "가게를 찾을 수 없습니다.",
        error: { detail: "storeId 없음" }
      });
    }

    // 미션 존재 확인
    const mission = await db.userMission.findUnique({
      where: { id: userMissionId }
    });

    if (!mission) {
      return res.status(404).json({
        success: false,
        code: 404,
        message: "미션을 찾을 수 없습니다.",
        error: { detail: "userMissionId 없음" }
      });
    }

    // 중복 리뷰 확인 (같은 user + 같은 mission 기준)
    const existingReview = await db.review.findFirst({
      where: {
        userId,
        userMissionId
      }
    });

    if (existingReview) {
      return res.status(409).json({
        success: false,
        code: 409,
        message: "이미 리뷰가 작성되었습니다.",
        error: { detail: "중복 리뷰" }
      });
    }

    // 리뷰 생성
    const review = await db.review.create({
      data: {
        userId,
        storeId,
        rating,
        content,
        userMissionId,
        images: {
          create: images.map((url: string) => ({
            url
          }))
        }
      }
    });

    // 성공 응답
    return res.status(201).json({
      success: true,
      code: 201,
      message: "리뷰 작성 성공",
      data: {
        reviewId: review.id
      }
    });
    
		// 실패 응답
  } catch (err) {
    console.error(err);
    return res.status(500).json({
      success: false,
      code: 500,
      message: "서버 에러",
      error: { detail: "unexpected error" }
    });
  }
});
```

1-3. 가게에 미션 추가하기 API

```tsx
app.post("/api/v1/stores/:storeId/missions", async (req: Request, res: Response) => {
  try {
    const storeId = Number(req.params.storeId);
    const { title, reward } = req.body;

    // 가게 존재 확인
    const store = await db.store.findUnique({
      where: { id: storeId }
    });

    if (!store) {
      return res.status(404).json({
        success: false,
        code: 404,
        message: "가게를 찾을 수 없습니다.",
        error: { detail: "storeId 없음" }
      });
    }

    // 미션 생성
    const mission = await db.mission.create({
      data: {
        storeId,
        title,
        reward
      }
    });

    // 성공 응답
    return res.status(201).json({
      success: true,
      code: 201,
      message: "미션 생성 성공",
      data: {
        missionId: mission.id
      }
    });
    
		// 실패 응답
  } catch (err) {
    console.error(err);
    return res.status(500).json({
      success: false,
      code: 500,
      message: "서버 에러",
      error: { detail: "unexpected error" }
    });
  }
});
```

**1-4. 가게의 미션을 도전 중인 미션에 추가(미션 도전하기) API**

- 도전하려는 미션이 이미 도전 중이지는 않은지 검증이 필요합니다.
- 1-3번 API를 구현하지 않은 경우, 1-4번에서는 DB에 미션 정보를 수동으로 기입한 후 진행해야 합니다.

```tsx
app.post("/api/v1/missions/:missionId/challenge", async (req: Request, res: Response) => {
  try {
    const missionId = Number(req.params.missionId);
    const { userId } = req.body;

    // 미션 존재 확인
    const mission = await db.mission.findUnique({
      where: { id: missionId }
    });

    if (!mission) {
      return res.status(404).json({
        success: false,
        code: 404,
        message: "미션을 찾을 수 없습니다.",
        error: { detail: "missionId 없음" }
      });
    }

    // 이미 도전 중인지 확인
    const existing = await db.userMission.findFirst({
      where: {
        userId,
        missionId
      }
    });

    if (existing) {
      return res.status(409).json({
        success: false,
        code: 409,
        message: "이미 도전 중인 미션입니다.",
        error: { detail: "중복 미션 도전" }
      });
    }

    // 도전 생성
    const userMission = await db.userMission.create({
      data: {
        userId,
        missionId,
        status: "IN_PROGRESS"
      }
    });

    // 성공 응답
    return res.status(201).json({
      success: true,
      code: 201,
      message: "미션 도전 성공",
      data: {
        userMissionId: userMission.id
      }
    });

		// 실패 응답
  } catch (err) {
    console.error(err);
    return res.status(500).json({
      success: false,
      code: 500,
      message: "서버 에러",
      error: { detail: "unexpected error" }
    });
  }
});
```

1. **Controller → Service → Repository → DB로 이어지는 요청 흐름을 정리해 주세요.**
    1. 사용자가  **`POST /api/v1/reviews`** 요청을 보냄
        
        
    2. **Controller**
        1. 요청(Request Body)을 받음
        2. 필요한 값 추출: userId, storeId, reting, content등
        3. Service 계층으로 전달
            
            
    3. **Service**
        1. 비지니스 로직 수행
        2. 가게 존재 여부 확인
        3. 미션 존재 여부 확인
        4. 이미 리뷰가 존재하는지 검증
        5. 문제 없을 시 Repository 호출
            
            
    4. **Repository**
        1. DB 접근
        2. 리뷰 데이터를 DB에 저장
        3. 저장된 결과 반환
            
            
    5. Repository → Service → Controller 순으로 저장 결과를 위로 반환
        
        
    6. **Controller**
        1. 최종 응답 생성
        2. 클라이언트에게 반환
    
2. 회원가입 API에 비밀번호 해싱 과정을 추가해 주세요.
    
    ```tsx
    import bcrypt from "bcrypt";
    
    const SALT_ROUNDS = 10;
    
    async function signup(userData: any) {
      const { password, ...rest } = userData;
    
      // 비밀번호 해싱
      const hashedPassword = await bcrypt.hash(password, SALT_ROUNDS);
    
      // DB 저장
      const user = await db.user.create({
        data: {
          ...rest,
          password: hashedPassword
        }
      });
    
      return user;
    }
    
    	// 로그인 시 비교
    const isMatch = await bcrypt.compare(password, user.password);
    
    if (!isMatch) {
      throw new Error("비밀번호 불일치");
    }
    ```
    

### 시니어 미션
- 현재는 API에서 오류가 발생하면 HTML로 된 콘텐츠가 내려오는데, 이 부분을 JSON 형태의 콘텐츠가 응답으로 내려가도록 개선해주세요.
    1. 전역 에러 핸들러
        
        index.ts
        
        ```tsx
        import { Request, Response, NextFunction } from "express";
        // 전역 에러 핸들러
        app.use((err: any, req: Request, res: Response, next: NextFunction) => {
          console.error(err);
          const status = err.status || 500;
          res.status(status).json({
            success: false,
            code: status,
            message: err.message || "서버 에러",
            error: {
              detail: err.detail || "unexpected error"
            }
          });
        });
        ```
        
    2. 404도 JSON으로 변경
        
        라우터
        
        ```tsx
        app.use((req: Request, res: Response) => {
          res.status(404).json({
            success: false,
            code: 404,
            message: "Not Found",
            error: {
              detail: "존재하지 않는 API입니다."
            }
          });
        });
        ```
    