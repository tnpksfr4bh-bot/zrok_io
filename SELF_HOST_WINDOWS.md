# Zrok 셀프 호스팅 가이드 (Windows)

이 가이드는 Windows 환경에서 zrok을 셀프 호스팅하는 방법을 단계별로 설명합니다.

---

## 📋 목차

1. [개요](#개요)
2. [사전 요구사항](#사전-요구사항)
3. [아키텍처 이해](#아키텍처-이해)
4. [방법 1: Docker Desktop 사용 (권장)](#방법-1-docker-desktop-사용-권장)
5. [방법 2: 수동 설치](#방법-2-수동-설치)
6. [사용자 계정 생성](#사용자-계정-생성)
7. [클라이언트 환경 연결](#클라이언트-환경-연결)
8. [TLS/HTTPS 설정](#tlshttps-설정)
9. [문제 해결](#문제-해결)
10. [유용한 명령어](#유용한-명령어)

---

## 개요

### 셀프 호스팅이란?

zrok.io 공용 서비스 대신 **자체 zrok 인스턴스**를 운영하는 것입니다.

### 셀프 호스팅의 장점

| 장점 | 설명 |
|------|------|
| 🔒 **완전한 데이터 통제** | 모든 트래픽이 자체 서버를 통과 |
| 🏢 **기업 내부망** | 인터넷 노출 없이 내부 서비스 공유 |
| ⚡ **성능 최적화** | 지리적으로 가까운 서버 사용 가능 |
| 💰 **비용 절감** | 대규모 사용 시 비용 효율적 |
| 🎨 **커스터마이징** | 도메인, 브랜딩 등 자유롭게 설정 |

---

## 사전 요구사항

### 필수 요구사항

| 항목 | 요구사항 |
|------|----------|
| **운영체제** | Windows 10/11 Pro 또는 Enterprise (Hyper-V 지원) |
| **RAM** | 최소 8GB (권장 16GB) |
| **저장공간** | 최소 20GB 여유 공간 |
| **네트워크** | 고정 IP 또는 DDNS (외부 접근 시) |
| **도메인** | 와일드카드 DNS 레코드 지원 도메인 (예: `*.zrok.yourdomain.com`) |

### 필수 소프트웨어

```
✅ Docker Desktop for Windows
✅ Git (선택사항)
✅ 텍스트 에디터 (VS Code 권장)
```

---

## 아키텍처 이해

zrok 셀프 호스팅은 다음 컴포넌트로 구성됩니다:

```
┌─────────────────────────────────────────────────────────────┐
│                    Zrok Self-Hosted Instance                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐ │
│  │   OpenZiti  │    │   OpenZiti  │    │  Zrok Controller│ │
│  │ Controller  │◄──►│   Router    │◄──►│    + Frontend   │ │
│  │  (제어부)   │    │  (라우팅)   │    │   (API + UI)    │ │
│  └─────────────┘    └─────────────┘    └─────────────────┘ │
│         │                  │                   │           │
│         └──────────────────┴───────────────────┘           │
│                            │                               │
│                     Zero-Trust Network                     │
└─────────────────────────────────────────────────────────────┘
                             │
                    ┌────────┴────────┐
                    │   클라이언트들   │
                    │  (zrok enable)  │
                    └─────────────────┘
```

| 컴포넌트 | 역할 |
|----------|------|
| **OpenZiti Controller** | 네트워크 정책 및 인증 관리 |
| **OpenZiti Router** | 암호화된 트래픽 라우팅 |
| **Zrok Controller** | zrok API 서버 + 웹 콘솔 |
| **Zrok Frontend** | 공개 접근 프록시 (HTTP/HTTPS) |

---

## 방법 1: Docker Desktop 사용 (권장)

Docker를 사용하면 가장 쉽게 셀프 호스팅 환경을 구축할 수 있습니다.

### 1단계: Docker Desktop 설치

1. [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/) 다운로드
2. 설치 진행 (WSL 2 백엔드 권장)
3. 설치 완료 후 재부팅
4. Docker Desktop 실행 확인:
   ```powershell
   docker --version
   docker compose version
   ```

### 2단계: 프로젝트 파일 다운로드

PowerShell을 **관리자 권한**으로 실행하고:

```powershell
# 작업 디렉토리 생성
mkdir C:\zrok-instance
cd C:\zrok-instance

# 자동 다운로드 스크립트 실행
Invoke-WebRequest -Uri "https://get.openziti.io/zrok-instance/fetch.bash" -OutFile "fetch.bash"

# 또는 수동으로 다운로드
Invoke-WebRequest -Uri "https://github.com/openziti/zrok/archive/refs/heads/main.zip" -OutFile "zrok-main.zip"
Expand-Archive -Path "zrok-main.zip" -DestinationPath "."
Copy-Item -Path "zrok-main\docker\compose\zrok-instance\*" -Destination "." -Recurse
Remove-Item -Path "zrok-main.zip", "zrok-main" -Recurse -Force
```

### 3단계: 환경 설정 파일 생성

`.env` 파일을 생성합니다:

```powershell
# .env 파일 생성
New-Item -Path ".env" -ItemType File
```

다음 내용을 `.env` 파일에 작성합니다:

```env
# ============================================
# Zrok Self-Host 기본 설정
# ============================================

# 필수 설정 - 반드시 변경하세요!
ZROK_DNS_ZONE=zrok.yourdomain.com
ZROK_USER_EMAIL=admin@yourdomain.com
ZROK_USER_PWD=YourSecurePassword123!

# OpenZiti 관리자 비밀번호
ZITI_PWD=ZitiAdminPassword456!

# Zrok 관리자 토큰 (랜덤하게 생성 권장)
ZROK_ADMIN_TOKEN=YourRandomAdminToken789

# ============================================
# 네트워크 설정
# ============================================

# 로컬 테스트용 (외부 접근 불가)
ZROK_INSECURE_INTERFACE=127.0.0.1

# 외부 접근 허용 시 (주의: 보안 설정 필수)
# ZROK_INSECURE_INTERFACE=0.0.0.0

# 서비스 포트
ZROK_CTRL_PORT=18080
ZROK_FRONTEND_PORT=8080
ZROK_OAUTH_PORT=8081
ZITI_CTRL_ADVERTISED_PORT=80
ZITI_ROUTER_PORT=3022
```

> ⚠️ **중요**: 모든 비밀번호와 토큰은 반드시 **강력하고 고유한 값**으로 변경하세요!

### 4단계: Docker Compose 실행

```powershell
# 컨테이너 빌드 및 시작
docker compose up --build --detach

# 로그 확인
docker compose logs -f
```

### 5단계: 서비스 상태 확인

```powershell
# 실행 중인 컨테이너 확인
docker compose ps

# 개별 서비스 로그 확인
docker compose logs zrok-controller
docker compose logs zrok-frontend
docker compose logs ziti-quickstart
```

정상 실행 시 다음과 같은 컨테이너가 동작합니다:
- `zrok-controller`
- `zrok-frontend`
- `ziti-quickstart`

### 6단계: 웹 콘솔 접속

브라우저에서 다음 주소로 접속:

```
http://localhost:18080
```

---

## 방법 2: 수동 설치

Docker 없이 직접 바이너리를 설치하는 방법입니다. (고급 사용자용)

### 1단계: OpenZiti 설치

Windows에서 OpenZiti를 먼저 설치해야 합니다.

1. [OpenZiti Releases](https://github.com/openziti/ziti/releases)에서 Windows 바이너리 다운로드
2. `ziti.exe`를 `C:\Program Files\OpenZiti\` 에 배치
3. PATH 환경 변수에 추가

### 2단계: OpenZiti Controller 설정

```powershell
# 설정 디렉토리 생성
mkdir C:\zrok\ziti
cd C:\zrok\ziti

# Controller 설정 파일 생성 (ziti-controller.yaml)
# (OpenZiti 공식 문서 참조)
```

### 3단계: OpenZiti Router 설정

```powershell
# Router JWT 생성
ziti edge login localhost:1280 -u admin -p <password>
ziti edge create edge-router "router1" -o router1.jwt

# Router 등록
ziti router enroll router1.jwt
```

### 4단계: Zrok Controller 설정

`C:\zrok\etc\ctrl.yml` 파일 생성:

```yaml
# zrok controller configuration
v: 4

admin:
  # 관리자 토큰 (랜덤하게 생성)
  secrets:
    - YourRandomAdminToken789

endpoint:
  host: 0.0.0.0
  port: 18080

invites:
  invites_open: true

store:
  path: C:\zrok\data\zrok.db
  type: sqlite3

ziti:
  api_endpoint: "https://127.0.0.1:1280"
  username: admin
  password: "YourZitiPassword"

# TLS 설정 (선택사항)
#tls:
#  cert_path: "C:\zrok\certs\zrok.crt"
#  key_path: "C:\zrok\certs\zrok.key"
```

### 5단계: Zrok 부트스트랩

```powershell
# 환경 변수 설정
$env:ZROK_ADMIN_TOKEN = "YourRandomAdminToken789"
$env:ZROK_API_ENDPOINT = "http://127.0.0.1:18080"

# OpenZiti 부트스트랩
zrok admin bootstrap C:\zrok\etc\ctrl.yml
```

### 6단계: Zrok Controller 실행

```powershell
zrok controller C:\zrok\etc\ctrl.yml
```

### 7단계: Zrok Frontend 설정

`C:\zrok\etc\http-frontend.yml` 파일 생성:

```yaml
v: 3
host_match: zrok.yourdomain.com
address: 0.0.0.0:8080
```

### 8단계: Frontend 실행

새 PowerShell 창에서:

```powershell
zrok access public C:\zrok\etc\http-frontend.yml
```

---

## 사용자 계정 생성

### Docker 방식

```powershell
# 첫 번째 사용자 생성 (.env에 설정된 값 사용)
docker compose exec zrok-controller bash -c 'zrok admin create account ${ZROK_USER_EMAIL} ${ZROK_USER_PWD}'

# 추가 사용자 생성
docker compose exec zrok-controller zrok admin create account user@example.com SecurePassword123
```

### 수동 설치 방식

```powershell
# 환경 변수 설정 필요
$env:ZROK_ADMIN_TOKEN = "YourRandomAdminToken789"
$env:ZROK_API_ENDPOINT = "http://127.0.0.1:18080"

# 계정 생성
zrok admin create account admin@yourdomain.com YourSecurePassword
```

**출력 예시:**
```
heMqncCyxZcx
```

> 💡 출력된 토큰(`heMqncCyxZcx`)은 **계정 활성화 토큰**입니다. 이 토큰을 사용하여 클라이언트 환경을 연결합니다.

---

## 클라이언트 환경 연결

셀프 호스팅 서버에 클라이언트를 연결하는 방법입니다.

### 1단계: 클라이언트에 zrok 설치

클라이언트 PC에서 [INSTALL.md](INSTALL.md)를 참조하여 zrok을 설치합니다.

### 2단계: API 엔드포인트 설정

```powershell
# 셀프 호스팅 서버 주소로 설정
zrok config set apiEndpoint http://YOUR_SERVER_IP:18080

# 또는 TLS 사용 시
zrok config set apiEndpoint https://zrok.yourdomain.com
```

### 3단계: 환경 활성화

계정 생성 시 받은 토큰으로 환경을 활성화합니다:

```powershell
zrok enable heMqncCyxZcx
```

**성공 출력:**
```
zrok environment 'abc123' enabled for 'heMqncCyxZcx'
```

### 4단계: 공유 테스트

```powershell
# 로컬 웹 서버 공유 테스트
zrok share public localhost:8080
```

---

## TLS/HTTPS 설정

프로덕션 환경에서는 TLS를 사용하는 것이 **필수**입니다.

### Caddy 사용 (권장)

`.env` 파일에 추가:

```env
# Caddy TLS 설정
COMPOSE_FILE=compose.yml:compose.caddy.yml

# DNS 제공자 설정 (Cloudflare 예시)
CADDY_DNS_PLUGIN=cloudflare
CADDY_DNS_PLUGIN_TOKEN=your-cloudflare-api-token
CADDY_ACME_API=https://acme-v02.api.letsencrypt.org/directory
CADDY_HTTPS_PORT=443
CADDY_INTERFACE=0.0.0.0
```

### Traefik 사용

`.env` 파일에 추가:

```env
# Traefik TLS 설정
COMPOSE_FILE=compose.yml:compose.traefik.yml

# DNS 제공자 설정 (DigitalOcean 예시)
TRAEFIK_DNS_PROVIDER=digitalocean
TRAEFIK_DNS_PROVIDER_TOKEN=your-digitalocean-api-token
TRAEFIK_ACME_API=https://acme-v02.api.letsencrypt.org/directory
TRAEFIK_HTTPS_PORT=443
TRAEFIK_INTERFACE=0.0.0.0
```

### DNS 와일드카드 설정

도메인 DNS에 다음 레코드를 추가합니다:

| 유형 | 호스트 | 값 |
|------|--------|-----|
| A | `*.zrok.yourdomain.com` | `서버_IP_주소` |
| A | `zrok.yourdomain.com` | `서버_IP_주소` |

---

## 방화벽 설정

Windows 방화벽에서 다음 포트를 허용해야 합니다:

### PowerShell로 방화벽 규칙 추가

```powershell
# Zrok Controller (API)
New-NetFirewallRule -DisplayName "Zrok Controller" -Direction Inbound -Protocol TCP -LocalPort 18080 -Action Allow

# Zrok Frontend (HTTP)
New-NetFirewallRule -DisplayName "Zrok Frontend HTTP" -Direction Inbound -Protocol TCP -LocalPort 8080 -Action Allow

# HTTPS (TLS 사용 시)
New-NetFirewallRule -DisplayName "Zrok HTTPS" -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow

# OpenZiti Controller
New-NetFirewallRule -DisplayName "Ziti Controller" -Direction Inbound -Protocol TCP -LocalPort 1280 -Action Allow

# OpenZiti Router
New-NetFirewallRule -DisplayName "Ziti Router" -Direction Inbound -Protocol TCP -LocalPort 3022 -Action Allow
```

### 포트 요약

| 포트 | 프로토콜 | 용도 |
|------|----------|------|
| 18080 | TCP | Zrok Controller API |
| 8080 | TCP | Zrok Frontend (HTTP) |
| 443 | TCP | HTTPS (TLS 사용 시) |
| 1280 | TCP | OpenZiti Controller |
| 3022 | TCP | OpenZiti Router |

---

## 문제 해결

### ❓ Docker 컨테이너가 시작되지 않음

```powershell
# 상세 로그 확인
docker compose logs --tail=100

# 컨테이너 재시작
docker compose down
docker compose up --build --detach
```

### ❓ "connection refused" 오류

**원인**: 서비스가 아직 시작되지 않았거나 포트가 잘못됨

**해결**:
1. 서비스 상태 확인: `docker compose ps`
2. 포트 확인: `netstat -an | findstr "18080"`
3. 방화벽 규칙 확인

### ❓ 클라이언트가 연결되지 않음

**원인**: API 엔드포인트 설정 오류

**해결**:
```powershell
# 현재 설정 확인
zrok status

# 엔드포인트 재설정
zrok config set apiEndpoint http://YOUR_SERVER_IP:18080
```

### ❓ TLS 인증서 오류

**원인**: 와일드카드 DNS 레코드 미설정 또는 인증서 발급 실패

**해결**:
1. DNS 레코드 확인: `nslookup test.zrok.yourdomain.com`
2. Caddy 로그 확인: `docker compose logs caddy`
3. Let's Encrypt rate limit 확인

### ❓ 웹 콘솔 접속 불가

1. 서버에서 직접 테스트:
   ```powershell
   curl http://localhost:18080
   ```
2. 외부에서 접속 시 방화벽 확인
3. ZROK_INSECURE_INTERFACE 설정 확인 (127.0.0.1 → 0.0.0.0)

---

## 유용한 명령어

### Docker 관련

| 명령어 | 설명 |
|--------|------|
| `docker compose up -d` | 백그라운드에서 시작 |
| `docker compose down` | 모든 컨테이너 중지 |
| `docker compose logs -f` | 실시간 로그 보기 |
| `docker compose restart` | 모든 서비스 재시작 |
| `docker compose exec zrok-controller bash` | 컨테이너 쉘 접속 |

### Zrok Admin 관련

| 명령어 | 설명 |
|--------|------|
| `zrok admin create account <email> <password>` | 사용자 생성 |
| `zrok admin bootstrap <config>` | OpenZiti 초기화 |
| `zrok admin create frontend <id> public <url>` | 프론트엔드 생성 |

### 상태 확인

| 명령어 | 설명 |
|--------|------|
| `zrok status` | 현재 환경 상태 |
| `zrok version` | 버전 확인 |
| `ziti edge list edge-routers` | Ziti 라우터 목록 |
| `ziti edge list identities` | Ziti ID 목록 |

---

## Windows 서비스로 등록 (선택사항)

Docker Desktop 대신 영구적으로 실행하려면 Windows 서비스로 등록할 수 있습니다.

### NSSM 사용

1. [NSSM](https://nssm.cc/download) 다운로드
2. 서비스 등록:
   ```powershell
   nssm install ZrokController "C:\zrok\zrok.exe" "controller C:\zrok\etc\ctrl.yml"
   nssm install ZrokFrontend "C:\zrok\zrok.exe" "access public C:\zrok\etc\http-frontend.yml"
   ```

---

## 참고 자료

- [Zrok 공식 문서](https://docs.zrok.io/)
- [Zrok GitHub](https://github.com/openziti/zrok)
- [OpenZiti 문서](https://openziti.io/docs/)
- [Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/)

---

> 📝 **문서 버전**: 1.0  
> 📅 **최종 업데이트**: 2026년 1월  
> 🎯 **대상**: Windows 10/11 사용자
