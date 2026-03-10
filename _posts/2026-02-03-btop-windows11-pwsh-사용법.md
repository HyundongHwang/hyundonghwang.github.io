---
layout: post
title: 260203 btop windows11 pwsh 사용법
comments: true
tags:
- performance
- windows
- powershell
- gpu
- tools
- research
- guide
- analysis
---
# 🛠️ btop: Windows 11 PowerShell 환경에서의 시스템 모니터링 완벽 가이드

## 📚 목차
1. [btop 소개](#btop-소개)
2. [왜 btop을 사용해야 하는가](#왜-btop을-사용해야-하는가)
3. [장단점 분석](#장단점-분석)
4. [주요 기능](#주요-기능)
5. [Windows 11 설치 방법](#windows-11-설치-방법)
6. [사용법 및 단축키](#사용법-및-단축키)
7. [커스터마이징](#커스터마이징)
8. [트러블슈팅](#트러블슈팅)

---

## 🧭 btop 소개

**btop**은 리소스 모니터링 도구로, CPU, 메모리, 디스크, 네트워크, 프로세스의 사용량과 통계를 실시간으로 보여주는 터미널 기반 시스템 모니터입니다. btop은 bashtop과 bpytop의 후속작으로, C++로 완전히 재작성되어 성능과 안정성이 크게 향상되었습니다.

### 🔹 btop의 진화 과정

```
bashtop (Bash) → bpytop (Python) → btop++ (C++)
  2019            2020              2021~현재
```

btop은 2026년 현재까지도 활발하게 개발 및 유지보수되고 있으며, Linux, macOS는 물론 Windows 환경에서도 사용할 수 있도록 **btop4win** 프로젝트가 진행되고 있습니다.

---

## 🎯 왜 btop을 사용해야 하는가

### 🔹 1. 직관적이고 아름다운 인터페이스

전통적인 `top`이나 `htop`과 달리 btop은 다음과 같은 특징을 제공합니다:

- **실시간 그래프 시각화**: CPU, 메모리, 네트워크를 시간 흐름에 따라 그래프로 표시
- **히트맵 형식**: 각 CPU 코어별 사용률을 색상으로 구분하여 한눈에 파악 가능
- **풀 마우스 지원**: 클릭만으로 프로세스 선택, 메뉴 접근 가능

### 🔹 2. 종합적인 시스템 모니터링

하나의 화면에서 모든 시스템 리소스를 통합 관리할 수 있습니다:

```
┌─────────────────────────────────────────────────────┐
│ CPU [████████████████░░░░░░░░░░] 65%                │
│ Core 1 [██████████░░░░░░░░░░] Core 2 [████░░░░░░░░]│
├─────────────────────────────────────────────────────┤
│ Mem [████████████░░░░░░░░] 8.2GB / 16GB             │
│ Swap [░░░░░░░░░░░░░░░░░░░░] 0 / 4GB                 │
├─────────────────────────────────────────────────────┤
│ Network ↓ 1.2MB/s ↑ 0.3MB/s                         │
├─────────────────────────────────────────────────────┤
│ Processes (sorted by CPU)                            │
│ PID   USER    CPU%  MEM%  COMMAND                    │
│ 1234  user    45.2  12.1  chrome.exe                 │
│ 5678  user    12.5   3.4  code.exe                   │
└─────────────────────────────────────────────────────┘
```

### ⚡ 3. 성능 효율성

C++로 작성되어 Python 기반인 bpytop 대비 리소스 사용량이 현저히 낮으며, 모니터링 자체가 시스템에 미치는 영향이 최소화되어 있습니다.

### 🔹 4. 2026년 "Linux의 해"에 걸맞은 도구

2026년은 "Linux의 해"로 불리며, btop은 Linux 커뮤니티에서 가장 활발하게 논의되고 권장되는 모니터링 도구 중 하나입니다. Windows 사용자도 WSL이나 btop4win을 통해 동일한 경험을 누릴 수 있습니다.

---

## ⚠️ 장단점 분석

### ✅ 장점

#### ✅ 뛰어난 사용자 경험

- **초보자 친화적**: 직관적인 인터페이스로 시스템 모니터링 경험이 없어도 쉽게 사용 가능
- **게임 스타일 메뉴**: 메뉴 시스템이 게임 UI처럼 직관적으로 설계됨
- **Vim 스타일 키바인딩**: `j`, `k` 등 Vim 사용자에게 익숙한 단축키 지원

#### ✅ 포괄적인 모니터링

- CPU, 메모리, 디스크, 네트워크, 프로세스를 하나의 화면에서 통합 관리
- GPU 모니터링 지원 (NVIDIA, AMD, Intel iGPU - Linux 전용, Windows는 OHMR 버전 필요)
- 배터리 인디케이터 (노트북 환경)
- 실시간 그래프 및 히트맵

#### ✅ 강력한 프로세스 관리

- 프로세스 정렬 및 필터링
- 트리 뷰 레이아웃으로 부모-자식 프로세스 관계 파악
- 프로세스 종료 및 시그널 전송 기능

#### ✅ 높은 커스터마이징 가능성

- 다양한 내장 테마
- 사용자 정의 테마 생성 가능
- bpytop, bashtop 테마 파일과 호환

#### ✅ 경량 및 효율성

- C++로 작성되어 최소한의 리소스 사용
- 모니터링 도구 자체가 시스템에 미치는 영향 최소화

### ⚠️ 단점

#### ❌ Windows 지원 제한

- **공식 Windows 지원 없음**: btop4win은 커뮤니티 프로젝트로, 공식적으로는 Linux/macOS만 지원
- **패키지 매니저 미지원**: Scoop, Winget, Chocolatey 등에 공식 패키지가 없어 수동 설치 필요 (2026년 2월 기준)
- **기능 제한**: Windows 버전은 표준 btop4win과 OHMR 버전으로 나뉘며, 표준 버전은 GPU/온도 모니터링 불가

#### ❌ 관리자 권한 요구 (Windows OHMR 버전)

- GPU 모니터링, CPU 온도 측정을 위해서는 관리자 권한으로 실행해야 함
- 표준 버전도 일부 프로세스 정보를 완전히 보려면 관리자 권한 권장

#### ❌ 러닝 커브 (상대적)

- htop보다는 기능이 많아 처음 접하는 사용자에게는 다소 복잡할 수 있음
- 하지만 메뉴 시스템이 잘 되어있어 금방 적응 가능

### ⚖️ btop vs htop vs top 비교표

| 특징 | top | htop | btop |
|------|-----|------|------|
| 인터페이스 | 텍스트 기반 | 컬러 + 인터랙티브 | 그래프 + 히트맵 + 풀 마우스 |
| 리소스 사용량 | 최소 | 낮음 | 낮음 (C++) |
| GPU 모니터링 | ❌ | ❌ | ✅ (Linux) |
| 네트워크 모니터링 | ❌ | 제한적 | ✅ 실시간 그래프 |
| 테마 지원 | ❌ | 제한적 | ✅ 다양한 테마 |
| 배우기 쉬움 | 보통 | 쉬움 | 매우 쉬움 |
| Windows 지원 | ✅ (WSL) | ✅ (WSL) | ⚠️ (btop4win) |

---

## 📌 주요 기능

### 🔹 1. 실시간 시각화

#### ▫️ CPU 모니터링
- 전체 CPU 사용률 및 각 코어별 사용률을 히트맵으로 표시
- 시간 흐름에 따른 CPU 사용률 그래프
- CPU 클럭 속도 표시 (OHMR 버전에서 정확도 향상)

#### ▫️ 메모리 모니터링
- RAM 및 Swap 사용량을 그래프와 퍼센트로 표시
- 캐시, 버퍼 등 상세 메모리 정보
- 시간별 메모리 사용 추세

#### ▫️ 디스크 I/O
- 읽기/쓰기 속도 실시간 표시
- 각 디스크별 사용률 및 여유 공간

#### ▫️ 네트워크
- 업로드/다운로드 속도 실시간 그래프
- 총 전송량 표시

### 🔹 2. 프로세스 관리

#### ▫️ 정렬 및 필터링
- CPU, 메모리, PID, 이름 등 다양한 기준으로 정렬
- 사용자, 프로세스 이름, 명령어 기반 필터링

#### ▫️ 트리 뷰
- 부모-자식 프로세스 관계를 트리 구조로 표시
- 프로세스 그룹 관리 용이

#### ▫️ 프로세스 제어
- 프로세스 종료 (Kill)
- 다양한 시그널 전송 (SIGTERM, SIGKILL 등)
- 우선순위 변경

### 🎮 3. GPU 모니터링 (제한적)

#### ▫️ Linux 환경
- NVIDIA GPU: 공식 드라이버 설치 시 자동 인식
- AMD GPU: ROCm SMI 설치 필요
- Intel iGPU: 기본 지원

#### ▫️ Windows 환경 (btop4win-OHMR)
- Libre Hardware Monitor 기반
- GPU 사용률, VRAM, 클럭, 온도 모니터링
- 관리자 권한 필수

### 🔹 4. 커스터마이징

#### ▫️ 테마
- 다양한 내장 테마 (Default, TTY, Catppuccin 등)
- 사용자 정의 테마 생성 및 공유
- 24비트 트루컬러 지원

#### ▫️ 레이아웃
- 각 모니터링 섹션 크기 조절
- 섹션 숨김/표시 토글

#### ▫️ 업데이트 주기
- `+`, `-` 키로 업데이트 간격 조정 (100ms 단위)

---

## 🛠️ Windows 11 설치 방법

### 🔹 PowerShell 7 준비 (권장)

btop4win은 PowerShell 5.1에서도 작동하지만, PowerShell 7.5.4 이상 사용을 권장합니다.

#### 🛠️ PowerShell 7 설치 (Winget 사용)

```powershell
# PowerShell 7 설치
winget install --id Microsoft.PowerShell --source winget

# 설치 확인
pwsh --version
```

#### 🛠️ PowerShell 7 설치 (MSI 인스토러)

1. [PowerShell GitHub Releases](https://github.com/PowerShell/PowerShell/releases) 페이지 방문
2. 최신 `.msi` 파일 다운로드
3. 설치 프로그램 실행

### 🛠️ btop4win 설치

#### ▫️ 방법 1: GitHub Releases에서 직접 다운로드 (권장)

1. **다운로드**
   - [btop4win Releases](https://github.com/aristocratos/btop4win/releases) 페이지 방문
   - 최신 릴리스 선택 (2026년 2월 기준: v1.0.5+)

2. **버전 선택**

   **btop4win (표준 버전)**
   - GPU 모니터링 미지원
   - CPU 온도 모니터링 미지원
   - CPU 클럭 정확도 낮음
   - ✅ 관리자 권한 불필요

   **btop4win-OHMR (향상된 버전)**
   - ✅ GPU 모니터링 지원 (Libre Hardware Monitor 사용)
   - ✅ CPU 온도 모니터링 지원
   - ✅ 정확한 CPU 클럭 표시
   - ❌ 관리자 권한 필수

3. **압축 해제 및 실행**

   ```powershell
   # 다운로드한 zip 파일 압축 해제
   Expand-Archive -Path "btop4win-v1.0.5.zip" -DestinationPath "C:\Tools\btop4win"

   # btop4win 실행
   cd C:\Tools\btop4win
   .\btop4win.exe
   ```

4. **환경 변수 설정 (선택사항)**

   PowerShell에서 어디서든 `btop`을 실행할 수 있도록 PATH 추가:

   ```powershell
   # 시스템 환경 변수에 추가 (관리자 권한 필요)
   [Environment]::SetEnvironmentVariable(
       "Path",
       "$env:Path;C:\Tools\btop4win",
       [EnvironmentVariableTarget]::Machine
   )

   # 또는 사용자 환경 변수에만 추가
   [Environment]::SetEnvironmentVariable(
       "Path",
       "$env:Path;C:\Tools\btop4win",
       [EnvironmentVariableTarget]::User
   )
   ```

#### 🛠️ 방법 2: Winget 설치 (비공식 - 2026년 2월 기준 실험적)

일부 서드파티 리포지토리에서 Winget 패키지를 제공할 수 있으나, 공식적으로는 지원되지 않습니다.

```powershell
# 패키지 검색
winget search btop

# 발견되면 설치
# winget install aristocratos.btop4win
```

**주의**: 공식 Winget 리포지토리에는 아직 등록되지 않았으므로, GitHub Releases 직접 다운로드 방법을 권장합니다.

### 🔹 WSL (Windows Subsystem for Linux) 사용

Windows에서 완전한 btop 기능을 사용하려면 WSL2를 통한 Linux 환경 사용을 고려할 수 있습니다.

```powershell
# WSL2 설치
wsl --install

# Ubuntu 등 배포판 설치 후
wsl

# Linux 환경에서 btop 설치
sudo apt update
sudo apt install btop
```

---

## 🛠️ 사용법 및 단축키

### 🔹 기본 실행

```powershell
# btop 실행
.\btop4win.exe

# 또는 PATH 설정 후
btop
```

### 🔹 주요 단축키 정리

#### 📌 메뉴 및 도움말

| 단축키 | 기능 |
|--------|------|
| `Esc`, `m` | 메인 메뉴 표시 |
| `F1`, `h` | 도움말 화면 표시 |
| `F2`, `o` | 옵션 메뉴 표시 |
| `Ctrl+C`, `q` | btop 종료 |

#### 📌 프로세스 네비게이션

| 단축키 | 기능 |
|--------|------|
| `↑`, `↓` | 프로세스 목록에서 위/아래 이동 |
| `j`, `k` | 프로세스 목록에서 위/아래 이동 (Vim 스타일) |
| `PgUp`, `PgDn` | 한 페이지씩 이동 |
| `Home`, `End` | 첫 페이지 / 마지막 페이지로 이동 |
| `←`, `→` | 정렬 기준 열 변경 |
| `Enter` | 선택한 프로세스의 상세 정보 표시 |

#### 📌 프로세스 관리

| 단축키 | 기능 |
|--------|------|
| `t` | 프로세스 종료 (SIGTERM) |
| `k` | 프로세스 강제 종료 (SIGKILL) |
| `s` | 프로세스에 시그널 전송 (목록에서 선택) |

#### 📌 필터링 및 검색

| 단축키 | 기능 |
|--------|------|
| `f` | 프로세스 필터 입력 |
| `F9` | 사용자, 이름, 명령어 기준으로 필터링 |
| `F10` | 특정 프로세스 검색 |

#### 📌 뷰 옵션

| 단축키 | 기능 |
|--------|------|
| `t` | 트리 뷰 토글 (부모-자식 프로세스 관계 표시) |
| `+` | 업데이트 주기에 100ms 추가 |
| `-` | 업데이트 주기에서 100ms 감소 |

#### 📌 마우스 사용

- **클릭**: 프로세스 선택, 메뉴 버튼 클릭
- **스크롤**: 프로세스 목록 스크롤
- **모든 하이라이트된 키 버튼**: 클릭 가능

### 🔹 사용 시나리오 예시

#### ▫️ 시나리오 1: CPU 점유율 높은 프로세스 찾기

```
1. btop 실행
2. 프로세스 섹션에서 기본적으로 CPU 사용률 순으로 정렬됨
3. 가장 위에 있는 프로세스가 CPU를 가장 많이 사용하는 프로세스
4. Enter 키로 상세 정보 확인
5. 필요 시 t 또는 k 키로 종료
```

#### ▫️ 시나리오 2: 특정 프로세스 검색 및 종료

```
1. F10 키를 눌러 검색 모드 진입
2. 프로세스 이름 입력 (예: "chrome")
3. Enter로 검색
4. ↑, ↓ 키로 원하는 프로세스 선택
5. t 키로 프로세스 종료 (응답 없으면 k 키로 강제 종료)
```

#### ▫️ 시나리오 3: 네트워크 사용량 모니터링

```
1. btop 실행
2. 상단 네트워크 섹션에서 실시간 업로드/다운로드 속도 확인
3. 그래프로 시간별 트렌드 파악
4. 프로세스 목록을 네트워크 사용량 기준으로 정렬하려면
   ← 또는 → 키로 정렬 열 변경
```

---

## 📌 커스터마이징

### 🗂️ 설정 파일 위치

Windows 환경에서 btop4win의 설정 파일 위치:

```
%APPDATA%\btop\btop.conf
```

또는

```
C:\Users\{사용자이름}\AppData\Roaming\btop\btop.conf
```

### 🔹 테마 변경

#### 🗂️ 1. 설정 파일 편집

```powershell
# 설정 파일 열기
notepad "$env:APPDATA\btop\btop.conf"
```

#### ▫️ 2. `color_theme` 항목 수정

```ini
# 기본 테마
color_theme = "Default"

# TTY 모드 (16색상)
color_theme = "TTY"

# 사용자 정의 테마
color_theme = "catppuccin_mocha"
```

### 🔹 사용자 정의 테마 생성

#### ▫️ 1. 테마 디렉토리 생성

```powershell
New-Item -Path "$env:APPDATA\btop\themes" -ItemType Directory -Force
```

#### 🗂️ 2. 테마 파일 작성

테마 파일은 `.theme` 확장자를 가지며, 다음과 같은 형식입니다:

```ini
# 예시: mytheme.theme
theme[main_bg]="#1e1e2e"
theme[main_fg]="#cdd6f4"
theme[title]="#f38ba8"
theme[hi_fg]="#89b4fa"
theme[selected_bg]="#45475a"
theme[selected_fg]="#cdd6f4"
# ... 기타 색상 설정
```

#### 🏢 3. 테마 적용

```ini
# btop.conf 파일에서
color_theme = "mytheme"
```

### 🔹 인기 테마

- **Default**: 기본 btop 테마
- **TTY**: 16색상 터미널용
- **Catppuccin**: 현대적이고 부드러운 파스텔 톤 ([Catppuccin btop](https://github.com/catppuccin/btop))
- **Nord**: 북유럽 스타일의 차가운 색상 테마

### 🛠️ 기타 설정 옵션

```ini
# 업데이트 주기 (밀리초)
update_ms = 2000

# 트루컬러 지원 활성화
truecolor = True

# 테마 배경 투명도
theme_background = True

# CPU 그래프 모드 (선 그래프 또는 막대 그래프)
cpu_graph_upper = "total"
cpu_graph_lower = "total"

# 프로세스 트리 뷰 기본 활성화
proc_tree = False

# 시스템 온도 표시 (OHMR 버전 전용)
show_cpu_freq = True
```

---

## 📌 트러블슈팅

### 🔹 1. btop4win이 실행되지 않음

#### ▫️ 문제: "ANSI escape sequences not supported" 오류

**원인**: Windows 10 1607 이하 버전에서는 표준 터미널이 ANSI 이스케이프 시퀀스를 지원하지 않습니다.

**해결 방법**:
- Windows 10을 1607 이상으로 업데이트
- 또는 ANSI를 지원하는 터미널 사용 (Windows Terminal, ConEmu 등)

```powershell
# Windows Terminal 설치
winget install --id Microsoft.WindowsTerminal --source winget
```

### 🔹 2. 프로세스 정보가 일부 누락됨

#### ▫️ 문제: 일부 프로세스 정보가 표시되지 않음

**원인**: 관리자 권한이 없으면 시스템 프로세스나 다른 사용자의 프로세스 정보를 읽을 수 없습니다.

**해결 방법**:
- 관리자 권한으로 PowerShell 실행 후 btop 실행

```powershell
# PowerShell을 관리자 권한으로 실행
Start-Process pwsh -Verb RunAs

# btop 실행
.\btop4win.exe
```

### 🎮 3. GPU 모니터링이 작동하지 않음

#### 🎮 문제: GPU 정보가 표시되지 않음

**원인**: 표준 btop4win 버전은 GPU 모니터링을 지원하지 않습니다.

**해결 방법**:
- btop4win-OHMR 버전으로 교체
- 관리자 권한으로 실행

```powershell
# OHMR 버전 다운로드 및 압축 해제 후
# 관리자 권한으로 실행
Start-Process -FilePath "C:\Tools\btop4win-OHMR\btop4win.exe" -Verb RunAs
```

### 🔹 4. CPU 온도가 표시되지 않음

#### ▫️ 문제: CPU 온도 정보가 나타나지 않음

**원인**: 표준 btop4win은 온도 모니터링을 지원하지 않습니다.

**해결 방법**:
- btop4win-OHMR 버전 사용
- `btop.conf`에서 `enable_ohmr = True` 설정 확인

```ini
# btop.conf
enable_ohmr = True
show_cpu_freq = True
```

### 🔹 5. 한글 프로세스 이름이 깨짐

#### ▫️ 문제: 한글로 된 프로세스 이름이나 경로가 깨져서 표시됨

**원인**: 터미널의 인코딩 설정 문제

**해결 방법**:
- PowerShell의 인코딩을 UTF-8로 설정

```powershell
# PowerShell 프로필에 추가 ($PROFILE)
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$OutputEncoding = [System.Text.Encoding]::UTF8
```

### 🔹 6. 업데이트 주기가 너무 빠르거나 느림

#### ▫️ 문제: 화면 깜빡임이 심하거나 반응이 느림

**해결 방법**:
- `+`, `-` 키로 실시간 조정
- 또는 `btop.conf` 파일에서 `update_ms` 값 변경

```ini
# 2초마다 업데이트 (기본값)
update_ms = 2000

# 1초마다 업데이트 (더 빠름)
update_ms = 1000

# 5초마다 업데이트 (더 느림, 시스템 부하 감소)
update_ms = 5000
```

### 🔹 7. Windows Terminal에서 색상이 이상함

#### ▫️ 문제: 색상이 제대로 표시되지 않거나 일부 색상이 흑백으로 나옴

**해결 방법**:
- `btop.conf`에서 트루컬러 설정 확인

```ini
truecolor = True
```

- Windows Terminal 설정에서 "고급" 탭 > "색 구성표" 확인

---

## 📌 마치며

btop은 2026년 현재 가장 현대적이고 강력한 터미널 기반 시스템 모니터링 도구입니다. Windows 11 환경에서는 btop4win을 통해 일부 제한적이지만 충분히 유용한 모니터링 경험을 제공합니다.

### 🔹 주요 요약

- **btop**은 CPU, 메모리, 디스크, 네트워크, 프로세스를 실시간 그래프로 모니터링하는 C++ 기반 도구
- **직관적인 UI**와 **풀 마우스 지원**으로 초보자도 쉽게 사용 가능
- **Windows 11 PowerShell 환경**에서는 btop4win (표준/OHMR)으로 사용
- **표준 버전**: 관리자 권한 불필요, GPU/온도 미지원
- **OHMR 버전**: 관리자 권한 필수, GPU/온도 모니터링 지원
- **Vim 스타일 단축키**, **프로세스 필터링**, **테마 커스터마이징** 등 고급 기능 제공

### 🔹 언제 사용하면 좋을까?

- 시스템 리소스 사용량을 **실시간으로 시각적**으로 파악하고 싶을 때
- CPU를 많이 사용하는 프로세스를 **빠르게 찾아 종료**해야 할 때
- 네트워크 트래픽을 **모니터링**하고 싶을 때
- **터미널 환경**을 선호하며 GUI 작업 관리자 대신 사용하고 싶을 때
- **개발자, 시스템 관리자**로서 시스템 상태를 효율적으로 관리하고 싶을 때

### 🔹 다음 단계

1. **btop4win 설치**: [GitHub Releases](https://github.com/aristocratos/btop4win/releases)에서 최신 버전 다운로드
2. **단축키 숙지**: `F1` 키로 도움말 확인하며 단축키 익히기
3. **테마 커스터마이징**: 자신만의 색상 테마로 btop 꾸미기
4. **WSL 환경 고려**: 완전한 btop 기능이 필요하다면 WSL2 + Linux btop 사용

btop과 함께 Windows 11 PowerShell 환경에서도 효율적인 시스템 모니터링을 경험해보세요!

---

## 🔗 참고 자료

### 🔹 공식 문서 및 리포지토리
- [btop GitHub 공식 리포지토리](https://github.com/aristocratos/btop)
- [btop4win GitHub 리포지토리](https://github.com/aristocratos/btop4win)
- [btop4win Releases](https://github.com/aristocratos/btop4win/releases)

### 🛠️ 설치 및 가이드
- [PowerShell 7 설치 가이드](https://pureinfotech.com/install-powershell-7-windows-11/)
- [Microsoft Learn - Install PowerShell on Windows](https://learn.microsoft.com/en-us/powershell/scripting/install/install-powershell-on-windows?view=powershell-7.5)
- [btop - Terminal Trove](https://terminaltrove.com/btop/)

### ⚖️ 리뷰 및 비교
- [OSXDaily - btop for MacOS (2026년 1월)](https://osxdaily.com/2026/01/22/btop-for-macos-is-an-excellent-terminal-system-resource-monitor/)
- [It's FOSS - Btop++: Linux System Monitoring Tool](https://itsfoss.com/btop-plus-plus/)
- [blackMORE Ops - Atop vs Btop vs Htop vs Top Comparison](https://www.blackmoreops.com/top-atop-btop-htop-linux-monitoring-tools-comparison/)
- [SecOps Solution - atop vs btop vs htop](https://www.secopsolution.com/blog/atop-vs-btop-vs-htop)
- [Linux Blog - btop - the htop alternative](https://linuxblog.io/btop-the-htop-alternative/)

### 🛠️ 기능 및 사용법
- [TecMint - btop: The Ultimate Real-Time System Monitoring Tool](https://www.tecmint.com/btop-system-monitoring-tool-for-linux/)
- [Hari Venu - Meet btop: The Ultimate Resource Monitor](https://harivenu.com/article/meet-btop-ultimate-resource-monitor-linux-enthusiasts)
- [LinuxMan - How To Monitor Linux Resources Using The BTOP Command](https://linuxman.co/linux-desktop/introduction-to-btop/)
- [Linux Hint - Btop++ System Monitoring Tool for Linux](https://linuxhint.com/btop-system-monitoring-tool/)

### 🔹 커스터마이징 및 테마
- [Catppuccin btop Theme](https://github.com/catppuccin/btop)
- [DeepWiki - Themes and Customization](https://deepwiki.com/aristocratos/btop4win/3.4-themes-and-customization)
- [GitHub - btop-theme Topics](https://github.com/topics/btop-theme)

### 🔹 커뮤니티 및 토론
- [DEV Community - BTOP++: The Resource Monitor I Didn't Know I Needed](https://dev.to/igorgbr/btop-the-resource-monitor-i-didnt-know-i-needed-389d)
- [programming.dev - Which one do you prefer? htop, btop or top?](https://programming.dev/post/11085929)
- [TrueNAS Community - Use btop instead of htop?](https://forums.truenas.com/t/use-btop-instead-of-htop/4491)

### 🔹 Windows 관련
- [btop4win GitHub Issues - GPU Monitoring not showing](https://github.com/aristocratos/btop4win/issues/10)
- [btop4win GitHub Issues - Chocolatey Package Request](https://github.com/aristocratos/btop4win/issues/14)

### 🔹 기타 유용한 링크
- [Debian Screenshots - btop](https://screenshots.debian.net/package/btop)
- [Snapcraft - Install btop on Linux](https://snapcraft.io/btop)
- [machaddr.substack - A Guide to Linux System Monitoring](https://machaddr.substack.com/p/a-guide-to-linux-system-monitoring)

---

**작성일**: 2026년 2월 3일
**대상 환경**: Windows 11, PowerShell 7.5+
**btop4win 버전**: v1.0.5 기준

