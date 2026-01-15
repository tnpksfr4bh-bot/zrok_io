# Zrok 셀프 호스트 - Docker 빠른 시작 가이드

Docker를 사용한 Zrok 셀프 호스트 설정을 **순차적으로** 실행할 수 있는 명령어 가이드입니다.

---

## 📋 사전 요구사항

- Windows 10/11 Pro (Hyper-V 지원)
- Docker Desktop 설치됨
- 도메인 (와일드카드 DNS 설정 필요)

---

## 🚀 Step 1: Docker Desktop 설치 확인

```powershell
# Docker 버전 확인
docker --version

# Docker Compose 버전 확인
docker compose version
```

> Docker Desktop이 없다면: https://www.docker.com/products/docker-desktop/ 에서 설치

---

## 🚀 Step 2: 작업 디렉토리 생성

```powershell
# 디렉토리 생성 및 이동
mkdir C:\zrok-instance
cd C:\zrok-instance
```

---

## 🚀 Step 3: Zrok Docker 파일 다운로드

```powershell
# GitHub에서 zrok 저장소 다운로드
Invoke-WebRequest -Uri "https://github.com/openziti/zrok/archive/refs/heads/main.zip" -OutFile "zrok-main.zip"

# 압축 해제
Expand-Archive -Path "zrok-main.zip" -DestinationPath "."

# Docker Compose 파일만 복사
Copy-Item -Path "zrok-main\docker\compose\zrok-instance\*" -Destination "." -Recurse

# 정리
Remove-Item -Path "zrok-main.zip", "zrok-main" -Recurse -Force

# 파일 확인
Get-ChildItem
```

---

## 🚀 Step 4: 환경 설정 파일 생성

```powershell
# .env 파일 생성
@"
# ========================================
# Zrok Self-Host 설정
# ========================================

# 🔴 필수: 아래 값들을 변경하세요!
ZROK_DNS_ZONE=zrok.yourdomain.com
ZROK_USER_EMAIL=admin@yourdomain.com
ZROK_USER_PWD=YourSecurePassword123!

# OpenZiti 관리자 비밀번호
ZITI_PWD=ZitiAdminPass456!

# Zrok 관리자 토큰 (32자 이상 랜덤 문자열)
ZROK_ADMIN_TOKEN=ChangeThisToRandomString789ABC

# ========================================
# 네트워크 설정
# ========================================

# 로컬 테스트용 (외부 접근 불가)
ZROK_INSECURE_INTERFACE=127.0.0.1

# 외부 접근 허용 시 아래 주석 해제
# ZROK_INSECURE_INTERFACE=0.0.0.0
"@ | Out-File -FilePath ".env" -Encoding UTF8

# 생성 확인
Get-Content .env
```

> ⚠️ **중요**: `.env` 파일을 열어서 `ZROK_DNS_ZONE`, 비밀번호 등을 실제 값으로 변경하세요!

---

## 🚀 Step 5: DNS 설정 (도메인 관리 패널에서)

도메인 DNS 관리 페이지에서 다음 레코드를 추가합니다:

| 유형 | 이름 | 값 |
|------|------|-----|
| A | `zrok` | `서버_공인_IP` |
| A | `*.zrok` | `서버_공인_IP` |

예시 (yourdomain.com 기준):
- `zrok.yourdomain.com` → 서버 IP
- `*.zrok.yourdomain.com` → 서버 IP

---

## 🚀 Step 6: Docker Compose 실행

```powershell
# 컨테이너 빌드 및 시작
docker compose up --build --detach
```

---

## 🚀 Step 7: 서비스 상태 확인

```powershell
# 실행 중인 컨테이너 확인
docker compose ps
```

**정상 출력 예시:**
```
NAME                  STATUS
zrok-controller       running
zrok-frontend         running
ziti-quickstart       running
```

---

## 🚀 Step 8: 로그 확인 (초기화 대기)

```powershell
# 실시간 로그 확인 (Ctrl+C로 종료)
docker compose logs -f
```

> 모든 서비스가 `ready` 또는 `started` 메시지를 출력할 때까지 대기 (약 1-2분)

---

## 🚀 Step 9: 사용자 계정 생성

```powershell
# 첫 번째 사용자 생성 (.env에 설정된 이메일/비밀번호 사용)
docker compose exec zrok-controller zrok admin create account $env:ZROK_USER_EMAIL $env:ZROK_USER_PWD

# 또는 직접 지정
docker compose exec zrok-controller zrok admin create account user@example.com MyPassword123!
```

**출력 예시:**
```
heMqncCyxZcx
```

> 📝 이 토큰(`heMqncCyxZcx`)을 기록하세요! 클라이언트 연결에 필요합니다.

---

## 🚀 Step 10: 웹 콘솔 접속 테스트

브라우저에서 접속:
```
http://localhost:18080
```

또는 도메인 설정 완료 시:
```
http://zrok.yourdomain.com
```

---

## 🚀 Step 11: 클라이언트에서 연결 테스트

**다른 PC 또는 새 PowerShell 창에서:**

```powershell
# 1. zrok 설치 (아직 안 했다면)
winget install openziti.zrok

# 2. 셀프 호스트 서버 주소 설정
zrok config set apiEndpoint http://zrok.yourdomain.com:18080

# 3. 토큰으로 환경 활성화
zrok enable heMqncCyxZcx

# 4. 로컬 서버 공유 테스트
zrok share public localhost:8080
```

---

## 📋 명령어 요약 (복사-붙여넣기용)

```powershell
# === 전체 설치 명령어 (순차 실행) ===

# 1. 디렉토리 생성
mkdir C:\zrok-instance
cd C:\zrok-instance

# 2. 파일 다운로드
Invoke-WebRequest -Uri "https://github.com/openziti/zrok/archive/refs/heads/main.zip" -OutFile "zrok-main.zip"
Expand-Archive -Path "zrok-main.zip" -DestinationPath "."
Copy-Item -Path "zrok-main\docker\compose\zrok-instance\*" -Destination "." -Recurse
Remove-Item -Path "zrok-main.zip", "zrok-main" -Recurse -Force

# 3. .env 파일 편집 (메모장으로 열기)
notepad .env

# 4. Docker 시작
docker compose up --build --detach

# 5. 상태 확인
docker compose ps

# 6. 사용자 생성
docker compose exec zrok-controller zrok admin create account admin@example.com MyPassword123!
```

---

## 🔧 유용한 관리 명령어

| 작업 | 명령어 |
|------|--------|
| 서비스 시작 | `docker compose up -d` |
| 서비스 중지 | `docker compose down` |
| 서비스 재시작 | `docker compose restart` |
| 로그 보기 | `docker compose logs -f` |
| 특정 서비스 로그 | `docker compose logs zrok-controller` |
| 컨테이너 쉘 접속 | `docker compose exec zrok-controller bash` |
| 사용자 추가 | `docker compose exec zrok-controller zrok admin create account <email> <password>` |

---

## ❓ 문제 해결

### 컨테이너가 시작되지 않음
```powershell
docker compose logs --tail=50
docker compose down
docker compose up --build --detach
```

### 포트 충돌
```powershell
# 사용 중인 포트 확인
netstat -an | findstr "18080"
netstat -an | findstr "8080"
```

### 완전 초기화
```powershell
docker compose down -v
Remove-Item -Path ".\data" -Recurse -Force -ErrorAction SilentlyContinue
docker compose up --build --detach
```

---

## 📊 서비스 포트 정보

| 서비스 | 포트 | 용도 |
|--------|------|------|
| Zrok Controller | 18080 | API + 웹 콘솔 |
| Zrok Frontend | 8080 | 공개 프록시 |
| Ziti Controller | 1280 | OpenZiti 관리 |
| Ziti Router | 3022 | 트래픽 라우팅 |

---

> 📅 **최종 업데이트**: 2026년 1월  
> 🎯 **대상**: Windows + Docker Desktop 사용자
