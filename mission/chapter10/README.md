# 10주차 미션: CI/CD 배포 자동화

## 1. 미션 목표

- GitHub Actions와 AWS EC2를 이용해 Node.js API를 자동 배포한다.
- 배포된 EC2 환경에서 Google OAuth 로그인이 동작하도록 수정한다.
- 결과 화면뿐 아니라 설정과 실패·수정 과정을 함께 기록한다.

## 2. 공통 미션

### 2-1. GitHub Actions와 EC2로 CI/CD 구축

구성한 파이프라인은 다음과 같다.

```text
main push
  → npm ci
  → Prisma·TSOA 코드 생성
  → TypeScript build
  → artifact 생성
  → SSH/rsync로 EC2 전송
  → Prisma migration 적용
  → systemd restart
  → 서비스 상태와 HTTP 응답 확인
```

구현 파일:

- `.github/workflows/deploy-main.yml`
- `package.json`의 `build`, `start`, `dev` scripts
- `.env.example`

진행 상태:

- [x] 배포용 build/start 스크립트 구성
- [x] GitHub Actions build/deploy job 작성
- [x] 로컬 `npm run build` 성공
- [ ] EC2 인스턴스와 탄력적 IP 준비
- [ ] GitHub Actions Secrets 등록
- [ ] `main` 병합 후 Actions 성공
- [ ] EC2 systemd 서비스 `active` 확인
- [ ] 외부에서 API 응답 확인

### 2-2. 배포 환경에서 Google OAuth 수정

기존 코드의 콜백 주소는 localhost로 고정되어 있어 배포 후 사용할 수 없었다.

```ts
callbackURL:
  process.env.PASSPORT_GOOGLE_CALLBACK_URL ??
  "http://localhost:3000/oauth2/callback/google";
```

수정 내용:

1. 콜백 주소를 `PASSPORT_GOOGLE_CALLBACK_URL` 환경변수로 분리했다.
2. GitHub Secret의 배포용 `.env`에는 EC2 콜백 주소를 넣는다.
3. Google Cloud Console의 승인된 리디렉션 URI에도 같은 주소를 등록한다.

진행 상태:

- [x] OAuth 콜백 URL 환경변수화
- [ ] Google Cloud Console에 EC2 콜백 URI 등록
- [ ] EC2 주소에서 Google 로그인 성공 확인
- [ ] 발급된 Access Token으로 `/mypage` 호출 확인

## 3. 파이프라인 분석

build job과 deploy job을 분리했다. build job이 실패하면 deploy job이 실행되지 않기 때문에 컴파일되지 않는 코드가 서버로 전달되는 것을 막을 수 있다.

GitHub-hosted runner에서 의존성 설치와 TypeScript 컴파일을 수행하고, EC2에는 실행에 필요한 `dist`와 운영 의존성만 전송한다. 이 방식은 작은 EC2 인스턴스가 직접 빌드할 때 생기는 CPU와 메모리 부담을 줄인다.

배포 서버에서는 새 파일을 `incoming`에 먼저 받은 뒤 기존 `current`와 교체한다. 이전 배포는 `previous`에 남겨 두어 문제가 발생했을 때 확인하거나 롤백할 수 있도록 했다.

## 4. 실습 증빙

아래 항목은 실제 AWS와 GitHub 작업을 진행하며 추가한다.

1. EC2 인스턴스와 탄력적 IP
2. Node.js와 MySQL 설치 결과
3. Secret 값이 가려진 GitHub Actions 설정
4. build job 성공 화면
5. deploy job 성공 화면
6. systemd 서비스 상태
7. 외부 API 호출 결과
8. Google Cloud Console 콜백 설정
9. 배포 주소에서 Google 로그인 성공 결과

## 5. 트러블슈팅

### 워크북의 workflow를 그대로 사용하면 build가 실패함

- **이슈:** `npm run build`와 `dist`를 사용하지만 기존 프로젝트에는 build script가 없었다.
- **문제:** 개발 환경은 `tsx`로 TypeScript를 직접 실행하고 있었다.
- **해결:** Prisma와 TSOA 생성 후 `tsc`로 컴파일하는 build script를 만들고, 운영 start 명령을 `node dist/index.js`로 분리했다.

### TSOA 생성 파일이 ESM 규칙과 충돌함

- **이슈:** 생성된 controller import에 `.js` 확장자가 없어 NodeNext 컴파일이 실패했다.
- **문제:** 프로젝트의 생성 코드 방식과 ESM 모듈 해석 규칙이 일치하지 않았다.
- **해결:** 서버 출력 형식을 CommonJS로 통일하고 빌드를 다시 검증했다.

### 배포 후 OAuth가 localhost로 돌아감

- **이슈:** 로그인 성공 후 EC2가 아닌 localhost 콜백으로 이동한다.
- **문제:** 콜백 URL이 코드에 하드코딩되어 있고 Google Console에도 로컬 URI만 등록되어 있다.
- **해결:** 콜백을 환경변수로 분리하고 코드와 Google Console에 동일한 배포 URI를 설정한다.

## 6. 시니어 미션 계획

현재 systemd `restart` 방식에는 짧은 다운타임이 존재한다. 기본 미션을 완료한 후 다음 순서로 PM2 무중단 배포를 적용할 예정이다.

1. PM2 cluster mode 설정
2. `ecosystem.config.cjs` 작성
3. `pm2 startup`, `pm2 save`로 부팅 자동 실행
4. 배포 단계에서 `pm2 reload` 사용
5. 연속 요청을 보내며 실패 응답이 발생하지 않는지 확인

## 7. 회고

로컬에서 실행되는 코드를 그대로 서버에 복사하는 것만으로는 배포가 완성되지 않는다. CI 환경에서 재현 가능한 빌드를 만들고, Secret을 안전하게 전달하며, 서버 프로세스 상태와 실제 HTTP 응답까지 확인해야 하나의 배포라고 할 수 있다.

또한 OAuth처럼 외부 서비스가 포함된 기능은 애플리케이션 코드뿐 아니라 배포 주소와 외부 콘솔 설정이 함께 일치해야 한다는 점을 확인했다.
