# Zrok 설치 방법

## Zrok이란?

Zrok은 로컬 서비스를 인터넷에 안전하게 노출시키는 오픈소스 터널링 도구입니다. 

### 주요 특징
- 🔒 **보안 터널링**: 로컬 개발 서버를 안전하게 외부에 공개
- 🌐 **즉시 공유**: 별도의 서버 설정 없이 URL 하나로 공유 가능
- 💻 **크로스 플랫폼**: Windows, macOS, Linux 모두 지원
- 🆓 **오픈소스**: 무료로 사용 가능

### 사용 사례
- 로컬 웹 개발 서버를 팀원이나 클라이언트에게 공유
- 웹훅(Webhook) 테스트
- 원격 데모 시연
- IoT 기기 접근

---

## 사전 요구사항

| 항목 | 요구사항 |
|------|----------|
| **운영체제** | Windows 10/11, macOS 10.15+, Linux (Ubuntu 18.04+) |
| **아키텍처** | x86_64 (AMD64) 또는 ARM64 |
| **네트워크** | 인터넷 연결 필요 |
| **권한** | 관리자 권한 (PATH 설정 시 필요) |

---

## Windows

### 방법 1: 바이너리 다운로드 (권장)

1. [Zrok GitHub Releases](https://github.com/openziti/zrok/releases) 페이지로 이동합니다.
2. 최신 릴리즈에서 `zrok_<버전>_windows_amd64.zip` 파일을 다운로드합니다.
3. 다운로드한 ZIP 파일을 압축 해제합니다.
4. `zrok.exe` 파일을 원하는 디렉토리 (예: `C:\Program Files\zrok\`)로 이동합니다.
5. 시스템 환경 변수 PATH에 해당 디렉토리를 추가합니다:

   **PATH 설정 방법 (상세)**:
   1. `Win + R` 키를 눌러 실행 창을 엽니다
   2. `sysdm.cpl`을 입력하고 Enter
   3. **고급** 탭 → **환경 변수** 버튼 클릭
   4. **시스템 변수**에서 `Path`를 선택하고 **편집** 클릭
   5. **새로 만들기** 클릭 후 `C:\Program Files\zrok` 입력
   6. **확인**을 눌러 모든 창 닫기

6. **새 PowerShell 또는 명령 프롬프트 창**을 열고 설치 확인:
   ```powershell
   zrok version
   ```
   > 💡 **팁**: 기존에 열려있던 터미널에서는 PATH가 적용되지 않습니다. 반드시 새 창을 여세요!

### 방법 2: Winget 패키지 매니저 사용

Windows 10/11에서 Winget이 설치되어 있다면 다음 명령을 실행합니다:

```powershell
winget install openziti.zrok
```

> 💡 **Winget이 설치되어 있는지 확인하려면**: `winget --version` 명령 실행

### 방법 3: Scoop 패키지 매니저 사용

Scoop이 설치되어 있다면:

```powershell
scoop bucket add extras
scoop install zrok
```

> 💡 **Scoop 설치 방법** (없는 경우):
> ```powershell
> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
> Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
> ```

---

## macOS

### Homebrew 사용 (권장)

```bash
brew install openziti/tap/zrok
```

> 💡 **Homebrew가 설치되어 있지 않다면**:
> ```bash
> /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
> ```

### 바이너리 다운로드

1. [Zrok GitHub Releases](https://github.com/openziti/zrok/releases)에서 다운로드:
   - **Intel Mac**: `zrok_<버전>_darwin_amd64.tar.gz`
   - **Apple Silicon (M1/M2/M3)**: `zrok_<버전>_darwin_arm64.tar.gz`
2. 압축 해제 및 설치:
   ```bash
   tar -xzf zrok_*_darwin_*.tar.gz
   sudo mv zrok /usr/local/bin/
   chmod +x /usr/local/bin/zrok
   ```
3. 설치 확인:
   ```bash
   zrok version
   ```

---

## Linux

### 원라인 설치 스크립트 (권장)

```bash
curl -sSLf https://get.openziti.io/install.bash | sudo bash -s zrok
```

### 바이너리 다운로드

1. [Zrok GitHub Releases](https://github.com/openziti/zrok/releases)에서 해당 아키텍처의 tar.gz 파일을 다운로드합니다:
   - **x86_64**: `zrok_<버전>_linux_amd64.tar.gz`
   - **ARM64**: `zrok_<버전>_linux_arm64.tar.gz`

2. 압축 해제 및 설치:
   ```bash
   tar -xzf zrok_*_linux_*.tar.gz
   sudo mv zrok /usr/local/bin/
   sudo chmod +x /usr/local/bin/zrok
   ```

3. 설치 확인:
   ```bash
   zrok version
   ```

### APT (Ubuntu/Debian)

```bash
# OpenZiti 저장소 추가
curl -sSLf https://get.openziti.io/tun/package-repos.gpg | sudo gpg --dearmor -o /usr/share/keyrings/openziti.gpg
echo "deb [signed-by=/usr/share/keyrings/openziti.gpg] https://packages.openziti.org/zitipax-openziti-deb-stable debian main" | sudo tee /etc/apt/sources.list.d/openziti.list

# 패키지 설치
sudo apt update
sudo apt install zrok
```

### RPM (RHEL/CentOS/Fedora)

```bash
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://packages.openziti.org/zitipax-openziti-rpm-stable/zitipax-openziti-rpm-stable.repo
sudo dnf install zrok
```

---

## 설치 후 설정

### 1단계: Zrok 계정 생성

Zrok을 사용하려면 먼저 계정을 생성해야 합니다:

1. [https://zrok.io](https://zrok.io) 에 접속합니다.
2. **"Sign Up"** 버튼을 클릭합니다.
3. 이메일 주소를 입력하고 회원가입을 완료합니다.
4. 이메일로 전송된 인증 링크를 클릭하여 계정을 활성화합니다.

### 2단계: 환경 토큰 발급

1. [https://zrok.io](https://zrok.io) 에 로그인합니다.
2. 대시보드에서 **"Enable Your Environment"** 섹션을 찾습니다.
3. 표시된 **환경 토큰(Environment Token)** 을 복사합니다.
   > ⚠️ **주의**: 토큰은 비밀번호처럼 취급하세요. 다른 사람과 공유하지 마세요!

### 3단계: 환경 활성화

터미널(Windows: PowerShell, macOS/Linux: Terminal)을 열고 다음 명령을 실행합니다:

```bash
zrok enable <복사한_토큰>
```

예시:
```bash
zrok enable abc123def456ghi789
```

성공 시 다음과 같은 메시지가 표시됩니다:
```
zrok environment enabled for account: your@email.com
```

---

## 기본 사용법

### 로컬 웹 서버 공유하기

로컬에서 실행 중인 웹 서버(예: localhost:3000)를 공유하려면:

```bash
zrok share public localhost:3000
```

실행 결과:
```
Access your share at: https://abc123.share.zrok.io
```

생성된 URL을 다른 사람과 공유하면 됩니다!

### 비공개 공유 (Private Share)

특정 사용자만 접근할 수 있도록 비공개로 공유하려면:

```bash
zrok share private localhost:3000
```

### 공유 종료

공유를 종료하려면 터미널에서 `Ctrl + C`를 누릅니다.

---

## 설치 확인 및 테스트

설치가 정상적으로 완료되었는지 확인하려면:

```bash
# 버전 확인
zrok version

# 환경 상태 확인
zrok status

# 도움말 보기
zrok help
```

---

## 문제 해결 (FAQ)

### ❓ "zrok 명령을 찾을 수 없습니다" 오류

**원인**: PATH 환경 변수에 zrok이 등록되지 않음

**해결 방법**:
1. zrok.exe 파일 위치 확인
2. 해당 경로를 시스템 PATH에 추가
3. 터미널을 재시작

### ❓ "enable" 후에도 공유가 안 됩니다

**해결 방법**:
1. `zrok status` 명령으로 환경 상태 확인
2. 토큰이 올바른지 확인
3. 네트워크 연결 상태 확인
4. 방화벽에서 zrok 허용 여부 확인

### ❓ 토큰을 분실했습니다

1. [https://zrok.io](https://zrok.io) 에 로그인
2. 대시보드에서 새 토큰 발급 가능

### ❓ 환경을 초기화하고 싶습니다

```bash
zrok disable
```
이후 새 토큰으로 다시 `zrok enable` 실행

---

## 유용한 명령어 모음

| 명령어 | 설명 |
|--------|------|
| `zrok version` | 설치된 버전 확인 |
| `zrok enable <token>` | 환경 활성화 |
| `zrok disable` | 환경 비활성화 |
| `zrok status` | 현재 환경 상태 확인 |
| `zrok share public <target>` | 공개 공유 시작 |
| `zrok share private <target>` | 비공개 공유 시작 |
| `zrok help` | 도움말 표시 |

---

더 자세한 사용법은 [공식 문서](https://docs.zrok.io/)를 참조하세요.