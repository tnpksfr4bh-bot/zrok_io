# Zrok 셀프 호스트 가이드 (Docker 미사용)

이 가이드는 Docker를 사용하지 않고 **바이너리만으로** Zrok 셀프 호스트 환경을 구축하는 방법을 설명합니다.

> ⚠️ **중요**: Zrok 셀프 호스트 **서버**는 Linux에서 운영하는 것이 공식적으로 권장됩니다.  
> Windows/Mac은 **클라이언트**로 해당 서버에 접속하여 사용합니다.

---

## 📋 목차

1. [아키텍처 개요](#아키텍처-개요)
2. [사전 요구사항](#사전-요구사항)
3. [Linux 서버 설치](#linux-서버-설치)
4. [클라이언트 사용법 (Windows/Mac)](#클라이언트-사용법-windowsmac)
5. [서비스 관리](#서비스-관리)
6. [문제 해결](#문제-해결)

---

## 🏗️ 아키텍처 개요

```
┌─────────────────────────────────────────────────────────────┐
│                    Linux 서버 (Self-Host)                    │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │ Ziti Controller │  │  Ziti Router    │  │ Zrok         │ │
│  │    (포트 1280)   │  │                 │  │ Controller   │ │
│  └─────────────────┘  └─────────────────┘  │ (포트 18080)  │ │
│                                            └──────────────┘ │
│  ┌─────────────────────────────────────────────────────────┐│
│  │              Zrok Public Frontend (포트 8080)            ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │ HTTPS
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  클라이언트 (Windows/Mac/Linux)                              │
│  - zrok enable <token>                                       │
│  - zrok share public localhost:3000                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 📋 사전 요구사항

### 서버 요구사항 (Linux)

| 항목 | 요구사항 |
|------|----------|
| **서버** | 공인 IP를 가진 Linux 서버 (Ubuntu 20.04+, Debian 11+, RHEL 8+) |
| **RAM** | 최소 2GB (권장 4GB) |
| **저장공간** | 최소 10GB |
| **도메인** | 와일드카드 DNS 레코드 (예: `*.zrok.yourdomain.com` → 서버 IP) |

### 클라이언트 요구사항 (Windows/Mac)

| 항목 | 요구사항 |
|------|----------|
| **운영체제** | Windows 10/11, macOS 10.15+ |
| **네트워크** | 인터넷 연결 (서버에 접속 가능해야 함) |
| **소프트웨어** | zrok 클라이언트 설치 |

---

## 🐧 Linux 서버 설치

### 1단계: OpenZiti 설치

Zrok은 OpenZiti 네트워크 위에서 동작합니다.

#### Ubuntu/Debian

```bash
# OpenZiti 저장소 추가
curl -sSLf https://get.openziti.io/tun/package-repos.gpg | sudo gpg --dearmor -o /usr/share/keyrings/openziti.gpg
echo "deb [signed-by=/usr/share/keyrings/openziti.gpg] https://packages.openziti.org/zitipax-openziti-deb-stable debian main" | sudo tee /etc/apt/sources.list.d/openziti.list

# 패키지 설치
sudo apt update
sudo apt install -y openziti-controller openziti-router
```

#### RHEL/CentOS/Fedora

```bash
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://packages.openziti.org/zitipax-openziti-rpm-stable/zitipax-openziti-rpm-stable.repo
sudo dnf install -y openziti-controller openziti-router
```

---

### 2단계: Ziti Controller 설정

#### 설정 파일 수정

`/opt/openziti/etc/controller/bootstrap.env` 파일을 편집합니다:

```bash
sudo nano /opt/openziti/etc/controller/bootstrap.env
```

```bash
# 서버의 FQDN 또는 공인 IP
ZITI_CTRL_ADVERTISED_ADDRESS=zrok.yourdomain.com
ZITI_CTRL_ADVERTISED_PORT=1280

# 강력한 관리자 비밀번호 설정
ZITI_PWD=YourStrongZitiPassword123!
```

#### 서비스 시작

```bash
sudo systemctl enable ziti-controller
sudo systemctl start ziti-controller
sudo systemctl status ziti-controller
```

#### 로그인 테스트

```bash
ziti edge login localhost:1280 -u admin -p YourStrongZitiPassword123!
```

---

### 3단계: Ziti Router 설정

#### 라우터 생성

```bash
# Ziti Controller에 로그인된 상태에서
ziti edge create edge-router "router1" -o /tmp/router1.jwt
```

#### 설정 파일 수정

`/opt/openziti/etc/router/bootstrap.env` 파일을 편집합니다:

```bash
sudo nano /opt/openziti/etc/router/bootstrap.env
```

```bash
ZITI_CTRL_ADVERTISED_ADDRESS=zrok.yourdomain.com
ZITI_CTRL_ADVERTISED_PORT=1280
ZITI_ROUTER_ADVERTISED_ADDRESS=zrok.yourdomain.com

# router1.jwt 파일의 내용을 여기에 입력
ZITI_BOOTSTRAP_ENROLLMENT_TOKEN=$(cat /tmp/router1.jwt)
```

#### 서비스 시작

```bash
sudo systemctl enable ziti-router
sudo systemctl start ziti-router
sudo systemctl status ziti-router
```

#### 라우터 상태 확인

```bash
ziti edge list edge-routers
```

---

### 4단계: Zrok 설치

```bash
# Ubuntu/Debian
sudo apt install -y zrok

# 또는 바이너리 직접 다운로드
curl -sSLf https://github.com/openziti/zrok/releases/latest/download/zrok_linux_amd64.tar.gz -o zrok.tar.gz
tar -xzf zrok.tar.gz
sudo mv zrok /usr/local/bin/
sudo chmod +x /usr/local/bin/zrok

# 확인
zrok version
```

---

### 5단계: Zrok Controller 설정

#### 디렉토리 생성

```bash
sudo mkdir -p /etc/zrok
sudo mkdir -p /var/lib/zrok
```

#### 관리자 토큰 생성

```bash
# 32자리 랜덤 토큰 생성
LC_ALL=C tr -dc _A-Z-a-z-0-9 < /dev/urandom | head -c32
# 예: Q8V0LqnNb5wNX9kE1fgQ0H6VlcvJybB1
```

#### 설정 파일 생성

`/etc/zrok/ctrl.yml` 파일을 생성합니다:

```bash
sudo nano /etc/zrok/ctrl.yml
```

```yaml
#    _____ __ ___
#   |_  / '__/ _ \| |/ /
#    / /| | | (_) |   <
#   /___|_|  \___/|_|\_\

v: 4

admin:
  secrets:
    - Q8V0LqnNb5wNX9kE1fgQ0H6VlcvJybB1   # 위에서 생성한 토큰

endpoint:
  host: 0.0.0.0
  port: 18080

invites:
  invites_open: true

store:
  path: /var/lib/zrok/zrok.db
  type: sqlite3

ziti:
  api_endpoint: "https://127.0.0.1:1280"
  username: admin
  password: "YourStrongZitiPassword123!"   # Ziti Controller 비밀번호
```

---

### 6단계: Zrok Bootstrap

OpenZiti 네트워크를 Zrok용으로 초기화합니다:

```bash
# 환경 변수 설정
export ZROK_ADMIN_TOKEN=Q8V0LqnNb5wNX9kE1fgQ0H6VlcvJybB1
export ZROK_API_ENDPOINT=http://127.0.0.1:18080

# Bootstrap 실행
zrok admin bootstrap /etc/zrok/ctrl.yml
```

**예상 출력:**
```
[   0.002]    INFO main.(*adminBootstrap).run: {...}
[   0.002]    INFO zrok/controller/store.Open: database connected
...
[   0.120] WARNING zrok/controller.Bootstrap: missing public frontend for ziti id 'sqJRAINSiB'; 
           please use 'zrok admin create frontend sqJRAINSiB public https://{token}.zrok.yourdomain.com' 
           to create a frontend instance
[   0.140]    INFO main.(*adminBootstrap).run: bootstrap complete!
```

> 📝 **frontend identity ID** (예: `sqJRAINSiB`)를 기록해두세요!

---

### 7단계: Zrok Controller 서비스 등록

#### Systemd 서비스 파일 생성

```bash
sudo nano /etc/systemd/system/zrok-controller.service
```

```ini
[Unit]
Description=Zrok Controller
After=network.target ziti-controller.service ziti-router.service

[Service]
Type=simple
User=root
Environment="ZROK_ADMIN_TOKEN=Q8V0LqnNb5wNX9kE1fgQ0H6VlcvJybB1"
ExecStart=/usr/local/bin/zrok controller /etc/zrok/ctrl.yml
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

#### 서비스 시작

```bash
sudo systemctl daemon-reload
sudo systemctl enable zrok-controller
sudo systemctl start zrok-controller
sudo systemctl status zrok-controller
```

---

### 8단계: Public Frontend 생성

#### Frontend 레코드 생성

```bash
export ZROK_ADMIN_TOKEN=Q8V0LqnNb5wNX9kE1fgQ0H6VlcvJybB1
export ZROK_API_ENDPOINT=http://127.0.0.1:18080

# sqJRAINSiB를 Bootstrap에서 확인한 ID로 교체
zrok admin create frontend sqJRAINSiB public http://{token}.zrok.yourdomain.com:8080
```

**출력:**
```
[   0.037]    INFO main.(*adminCreateFrontendCommand).run: created global public frontend 'WEirJNHVlcW9'
```

#### Frontend 설정 파일 생성

```bash
sudo nano /etc/zrok/http-frontend.yml
```

```yaml
v: 3
host_match: zrok.yourdomain.com
address: 0.0.0.0:8080
```

---

### 9단계: Frontend 서비스 등록

```bash
sudo nano /etc/systemd/system/zrok-frontend.service
```

```ini
[Unit]
Description=Zrok Public Frontend
After=network.target zrok-controller.service

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/zrok access public /etc/zrok/http-frontend.yml
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable zrok-frontend
sudo systemctl start zrok-frontend
sudo systemctl status zrok-frontend
```

---

### 10단계: 방화벽 설정

```bash
# UFW 사용 시
sudo ufw allow 1280/tcp    # Ziti Controller
sudo ufw allow 18080/tcp   # Zrok Controller
sudo ufw allow 8080/tcp    # Zrok Frontend
sudo ufw allow 3022/tcp    # Ziti Router (설정에 따라 다름)

# 또는 firewalld 사용 시
sudo firewall-cmd --permanent --add-port=1280/tcp
sudo firewall-cmd --permanent --add-port=18080/tcp
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --permanent --add-port=3022/tcp
sudo firewall-cmd --reload
```

---

### 11단계: 사용자 계정 생성

```bash
export ZROK_ADMIN_TOKEN=Q8V0LqnNb5wNX9kE1fgQ0H6VlcvJybB1
export ZROK_API_ENDPOINT=http://127.0.0.1:18080

# 사용자 계정 생성
zrok admin create account user@example.com SecureUserPassword123!
```

**출력:**
```
SuGzRPjVDIcF
```

> 📝 이 토큰(`SuGzRPjVDIcF`)을 클라이언트 사용자에게 전달합니다.

---

### 12단계: 리버스 프록시 설정 (권장)

프로덕션 환경에서는 Caddy 또는 Nginx를 사용하여 TLS를 적용합니다.

#### Caddy 설치 및 설정

```bash
sudo apt install -y caddy
sudo nano /etc/caddy/Caddyfile
```

```caddy
# Zrok Controller API
zrok.yourdomain.com {
    reverse_proxy localhost:18080
}

# Zrok Public Frontend (와일드카드 - DNS 플러그인 필요)
*.zrok.yourdomain.com {
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
    reverse_proxy localhost:8080
}
```

```bash
sudo systemctl restart caddy
```

---

## 🖥️ 클라이언트 사용법 (Windows/Mac)

### 1단계: Zrok 클라이언트 설치

#### Windows (PowerShell)

```powershell
winget install openziti.zrok
```

#### macOS

```bash
brew install openziti/tap/zrok
```

### 2단계: API 엔드포인트 설정

```bash
# 셀프 호스트 서버 주소 설정
zrok config set apiEndpoint https://zrok.yourdomain.com

# 또는 TLS 없이 테스트 시
zrok config set apiEndpoint http://서버IP:18080
```

### 3단계: 환경 활성화

관리자로부터 받은 토큰으로 활성화:

```bash
zrok enable SuGzRPjVDIcF
```

**성공 출력:**
```
zrok environment '2AS1WZ3Sz' enabled for 'SuGzRPjVDIcF'
```

### 4단계: 상태 확인

```bash
zrok status --secrets
```

**출력:**
```
Config:
  CONFIG       VALUE                      SOURCE
  apiEndpoint  https://zrok.yourdomain.com   config

Environment:
  PROPERTY       VALUE
  Secret Token   SuGzRPjVDIcF
  Ziti Identity  2AS1WZ3Sz
```

### 5단계: 로컬 서버 공유

```bash
# 로컬 웹 서버를 공개적으로 공유
zrok share public localhost:3000
```

**출력:**
```
Access your share at: https://abc123.zrok.yourdomain.com
```

---

## 🔧 서비스 관리

### 서비스 상태 확인

```bash
sudo systemctl status ziti-controller
sudo systemctl status ziti-router
sudo systemctl status zrok-controller
sudo systemctl status zrok-frontend
```

### 모든 서비스 재시작

```bash
sudo systemctl restart ziti-controller ziti-router zrok-controller zrok-frontend
```

### 로그 확인

```bash
# Zrok Controller 로그
journalctl -u zrok-controller -f

# Zrok Frontend 로그
journalctl -u zrok-frontend -f
```

---

## ❓ 문제 해결

### Bootstrap 재실행 시 오류

```bash
zrok admin bootstrap --skip-frontend /etc/zrok/ctrl.yml
```

### Frontend Identity ID 찾기

```bash
ziti edge login localhost:1280 -u admin -p <비밀번호>
ziti edge list identities
```

### 연결 테스트

```bash
# Controller API 테스트
curl http://localhost:18080/api/v1/version

# Frontend 테스트
curl -H "Host: test.zrok.yourdomain.com" http://localhost:8080
```

### 클라이언트 연결 문제

```bash
# 클라이언트에서 설정 초기화
zrok disable

# 다시 설정
zrok config set apiEndpoint https://zrok.yourdomain.com
zrok enable <토큰>
```

---

## 📝 요약 체크리스트

### 서버 설정

- [ ] OpenZiti Controller 설치 및 시작
- [ ] OpenZiti Router 설치 및 시작
- [ ] Zrok 바이너리 설치
- [ ] `/etc/zrok/ctrl.yml` 설정
- [ ] `zrok admin bootstrap` 실행
- [ ] Zrok Controller 서비스 등록 및 시작
- [ ] Frontend 생성 (`zrok admin create frontend`)
- [ ] `/etc/zrok/http-frontend.yml` 설정
- [ ] Zrok Frontend 서비스 등록 및 시작
- [ ] 방화벽 포트 열기
- [ ] (선택) 리버스 프록시 설정

### 클라이언트 설정

- [ ] Zrok 클라이언트 설치
- [ ] API 엔드포인트 설정
- [ ] 토큰으로 환경 활성화

---

## 📚 참고 자료

- [Zrok 공식 셀프 호스트 가이드 (Linux)](https://docs.zrok.io/docs/guides/self-hosting/linux/)
- [OpenZiti 설치 문서](https://openziti.io/docs/category/deployments)
- [Zrok GitHub Repository](https://github.com/openziti/zrok)
- [Zrok 설정 파일 예시](https://github.com/openziti/zrok/blob/main/etc/ctrl.yml)

---

> 📅 **최종 업데이트**: 2026년 1월  
> 🎯 **대상**: Linux 서버 관리자, Windows/Mac 클라이언트 사용자
