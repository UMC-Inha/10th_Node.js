# 10주차 핵심 키워드

## 1. CI/CD

### CI: Continuous Integration

CI는 개발자가 변경한 코드를 자주 통합하고, 통합 과정에서 빌드와 테스트 같은 검증을 자동으로 수행하는 방식이다.

```text
코드 push 또는 Pull Request
        ↓
의존성 설치
        ↓
정적 검사·테스트·빌드
        ↓
통합 가능한 상태인지 확인
```

CI의 장점은 다음과 같다.

- 코드 충돌과 결함을 비교적 이른 시점에 발견한다.
- 사람마다 다른 빌드 과정을 하나의 절차로 통일한다.
- 반복적인 검증을 자동화하여 실수를 줄인다.
- 배포 가능한 결과물을 일관되게 만들 수 있다.

### CD: Continuous Delivery / Continuous Deployment

CD는 CI를 통과한 결과물을 실제 실행 환경까지 전달하는 과정을 자동화한다.

| 구분 | 의미 |
| --- | --- |
| Continuous Delivery | 운영 배포 직전까지 자동화하고 최종 배포는 사람이 승인 |
| Continuous Deployment | 검증을 통과한 변경을 운영 환경까지 자동 배포 |

이번 실습에서는 `main` 브랜치에 push되면 GitHub Actions가 빌드한 뒤 EC2의 systemd 서비스를 재시작하므로 Continuous Deployment에 가까운 흐름을 구성한다.

### CI와 CD를 분리하는 이유

빌드와 배포를 별도 job으로 나누면 실패 위치를 쉽게 구분할 수 있다. 또한 한 번 생성한 아티팩트를 여러 서버에 동일하게 배포할 수 있어 서버마다 다시 빌드하면서 생기는 차이를 줄일 수 있다.

## 2. GitHub Actions

GitHub Actions는 GitHub 저장소에서 발생하는 이벤트를 기준으로 자동화 작업을 실행하는 서비스다. 자동화 정의는 `.github/workflows/*.yml`에 작성한다.

### 주요 구성 요소

| 요소 | 설명 |
| --- | --- |
| Workflow | 하나의 자동화 전체 정의 |
| Event | workflow를 실행하는 조건 |
| Job | 같은 runner에서 수행되는 작업 묶음 |
| Step | job 내부의 개별 명령 또는 Action |
| Action | 재사용 가능한 자동화 단위 |
| Runner | workflow 명령을 실행하는 가상 머신 |
| Artifact | job 사이 또는 실행 이후 전달·보관하는 결과물 |
| Secret | SSH 키와 환경변수처럼 공개하면 안 되는 값 |

### 실행 조건 예시

```yaml
on:
  push:
    branches:
      - main
  workflow_dispatch:
```

- `main` 브랜치에 push가 발생하면 자동 실행된다.
- `workflow_dispatch`가 있으면 Actions 화면에서 수동 실행할 수 있다.

### build와 deploy 의존 관계

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

  deploy:
    runs-on: ubuntu-latest
    needs: build
```

`needs: build`로 인해 build가 성공해야 deploy가 시작된다. build가 실패하면 불완전한 결과물이 서버로 전달되지 않는다.

### `npm ci`를 사용하는 이유

`npm ci`는 `package-lock.json`에 기록된 버전을 그대로 설치한다. lock file을 수정하지 않으므로 CI 환경에서 빠르고 재현 가능한 설치에 적합하다.

### Secrets 사용 시 주의점

- Secret 값을 코드나 README에 기록하지 않는다.
- 로그에 Secret을 출력하는 명령을 작성하지 않는다.
- SSH 키는 배포 전용 키로 분리하고 필요한 권한만 부여한다.
- GitHub Environment와 승인 규칙을 사용하면 운영 배포를 더 엄격하게 관리할 수 있다.

## 3. Reverse Proxy

Reverse Proxy는 클라이언트 요청을 먼저 받은 뒤 내부 애플리케이션 서버로 전달하는 서버다. Node.js 애플리케이션 앞에 Nginx를 두는 구성이 대표적이다.

```text
Client
   ↓ HTTPS :443
Nginx
   ↓ HTTP :3000
Node.js Application
```

### Forward Proxy와의 차이

| 구분 | 대신하는 대상 | 목적 |
| --- | --- | --- |
| Forward Proxy | 클라이언트 | 클라이언트 익명화, 접근 제어, 캐시 |
| Reverse Proxy | 서버 | 서버 보호, TLS 종료, 라우팅, 로드밸런싱 |

### Reverse Proxy의 장점

- 애플리케이션의 내부 포트를 외부에 직접 공개하지 않는다.
- HTTPS 인증서를 Nginx에서 일괄 관리할 수 있다.
- 여러 애플리케이션으로 요청을 분배할 수 있다.
- 정적 파일, 압축, 캐시 등을 효율적으로 처리할 수 있다.
- 요청 크기 제한과 속도 제한 같은 방어 정책을 적용할 수 있다.

단, Reverse Proxy가 단일 장애점이 되지 않도록 상태 확인과 이중화 전략이 필요할 수 있다.

## 4. HTTPS

HTTPS는 HTTP 통신을 TLS로 보호한다. 이를 통해 다음 세 가지 보안 특성을 제공한다.

| 특성 | 의미 |
| --- | --- |
| 기밀성 | 제3자가 통신 내용을 읽기 어렵게 암호화 |
| 무결성 | 전송 중 데이터가 변조되었는지 확인 |
| 인증 | 인증서를 통해 접속한 서버의 신원을 확인 |

### TLS 연결 흐름

1. 클라이언트가 서버에 연결을 요청한다.
2. 서버가 인증서와 공개키 정보를 전달한다.
3. 클라이언트가 인증서의 도메인과 신뢰 체인을 검증한다.
4. 양쪽이 안전하게 세션 키를 합의한다.
5. 이후 HTTP 데이터는 세션 키로 암호화하여 전송한다.

### 배포 환경에서 HTTPS가 필요한 이유

OAuth 로그인에는 인증 코드와 토큰처럼 민감한 데이터가 오간다. HTTP만 사용하면 네트워크 중간에서 정보가 노출되거나 변조될 수 있다. 실제 서비스에서는 도메인, TLS 인증서, Nginx를 구성하고 Google OAuth 콜백도 HTTPS 주소로 등록해야 한다.

## 5. 네 키워드의 관계

```text
GitHub Actions
  ├─ CI: 설치·생성·빌드·검증
  └─ CD: 아티팩트를 EC2로 전달하고 프로세스 재시작
                                      ↓
Client ── HTTPS ──> Reverse Proxy ──> Node.js API
```

CI/CD는 코드를 서버까지 안전하고 반복 가능하게 전달하는 과정이고, Reverse Proxy와 HTTPS는 배포된 서버가 외부 요청을 안전하게 받도록 만드는 운영 구조다.

## 6. 이번 실습에서 느낀 점

CI/CD는 단순히 push 후 자동으로 서버를 재시작하는 기능이 아니다. 같은 의존성, 같은 빌드 명령, 같은 아티팩트를 사용해 배포 결과의 차이를 줄이는 것이 핵심이다. 또한 배포 자동화가 성공했더라도 HTTPS, Secret 관리, 장애 시 롤백과 무중단 배포까지 고려해야 실제 운영에 가까운 파이프라인이 된다.
