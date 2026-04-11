---
layout: post
title: "260312 Windows VS Code 사용법 리서치"
comments: true
tags:
- windows
- vscode
- 사용법
---

# 260312 Windows VS Code 사용법 리서치

검증 기준일: 2026-03-12 (Asia/Seoul)

## 한눈에 보는 결론

Windows에서 VS Code를 편하게 쓰려면, 초반에 `단축키`, `탭 동작`, `자동 저장`, `시작 화면`, `Git 대안`만 먼저 정리해도 체감이 크게 좋아진다.

- 편의성 체감이 가장 큰 설정
  - `editor.fontSize: 20`
  - `workbench.colorTheme: "Default High Contrast"`
  - `workbench.startupEditor: "none"`
  - `window.restoreWindows: "none"`
  - `files.autoSave: "afterDelay"`
  - `files.autoSaveDelay: 300`
  - `workbench.editor.enablePreview: false`
  - `workbench.editor.enablePreviewFromQuickOpen: false`
- Git는 VS Code 기본 Source Control만으로는 브랜치별 커밋 히스토리를 빠르게 훑기 불편하다.
  - VS Code 안에서 해결: `Git Graph` + `GitLens`
  - JetBrains 느낌에 가장 가까운 외부 대안: `Fork`
  - 키보드 중심/터미널 친화적 대안: `lazygit`

---

## 1. 자주 사용하는 단축키

아래는 Windows 기준으로 가장 자주 쓰는 조합만 추렸다.

| 목적 | 단축키 | 메모 |
|---|---|---|
| 파일 빠르게 찾기 | `Ctrl+P` | 파일명 일부만 입력해 이동 |
| 전체 명령 팔레트 | `Ctrl+Shift+P` | 설정, Git, 테마 변경까지 대부분 여기서 시작 |
| 프로젝트 전체 검색 | `Ctrl+Shift+F` | 문자열 검색/치환 진입점 |
| 현재 파일 내 찾기 | `Ctrl+F` | 기본 검색 |
| 현재 파일 내 치환 | `Ctrl+H` | 간단 치환 |
| 사이드바 토글 | `Ctrl+B` | Explorer 숨김/표시 |
| 터미널 열기/닫기 | `` Ctrl+` `` | 내장 터미널 토글 |
| 새 터미널 | `Ctrl+Shift+`` | PowerShell 세션 추가 |
| 탭 닫기 | `Ctrl+W` | 현재 에디터 닫기 |
| 닫은 탭 다시 열기 | `Ctrl+Shift+T` | 실수 복구에 자주 씀 |
| 저장 | `Ctrl+S` | 자동 저장을 써도 가끔 직접 저장 |
| 모두 저장 | `Ctrl+K`, `S` | 멀티 파일 작업 시 유용 |
| 줄 복사 | `Shift+Alt+Down/Up` | 코드 반복 작성에 빠름 |
| 줄 이동 | `Alt+Down/Up` | 블록 순서 변경 |
| 줄 삭제 | `Ctrl+Shift+K` | 한 줄 즉시 삭제 |
| 주석 토글 | `Ctrl+/` | 가장 많이 쓰는 편집 단축키 |
| 포맷 | `Shift+Alt+F` | Formatter 설치 시 매우 중요 |
| 정의로 이동 | `F12` | 코드 추적 기본 |
| 뒤로 이동 | `Alt+Left` | 정의 탐색 후 복귀 |
| 이름 변경 | `F2` | 심볼 리네임 |
| 여러 커서 | `Alt+Click` | 컬럼 편집/다중 편집 |
| 행 선택 확장 | `Shift+Alt+Right/Left` | 단어/표현식 선택 확장 |

실전에서는 아래 6개만 먼저 손에 익히면 충분하다.

- `Ctrl+P`
- `Ctrl+Shift+P`
- `Ctrl+Shift+F`
- `` Ctrl+` ``
- `Ctrl+/`
- `F12`

참고:
- VS Code 기본 키 바인딩은 Keyboard Shortcuts Reference에서 확인 가능
- 공식 문서: https://code.visualstudio.com/docs/reference/default-keybindings

---

## 2. 많이 설치하는 익스텐션

언어별 확장은 프로젝트에 따라 달라지므로, 여기서는 Windows 개발환경에서 범용적으로 많이 쓰는 축만 정리한다.

### 2-1. 거의 기본으로 깔리는 편

| 익스텐션 | 추천 이유 | 비고 |
|---|---|---|
| GitLens | blame, commit, file history, branch/author 추적이 강력 | 기본 Git UI의 부족함을 많이 메워줌 |
| Git Graph | 브랜치와 커밋 흐름을 그래프로 보기 좋음 | 브랜치 순회/히스토리 확인에 적합 |
| Error Lens | 에러/경고를 에디터 줄 옆에 바로 표시 | 문제를 Problems 패널 열지 않고 봄 |
| Prettier | 포맷 자동화 | JS/TS/JSON/MD 계열에서 매우 흔함 |
| ESLint | JS/TS lint + fix | 프론트엔드/Node 개발이면 사실상 기본 |
| EditorConfig | 팀별 들여쓰기/개행 정책 통일 | 저장소 협업 안정성 높음 |
| Material Icon Theme | 파일/폴더 아이콘 가독성 향상 | Explorer 인지성이 좋아짐 |

### 2-2. 상황 따라 추가

| 익스텐션 | 언제 추천하는가 |
|---|---|
| GitHub Pull Requests and Issues | GitHub PR 리뷰를 VS Code 안에서 보고 싶을 때 |
| Docker | 컨테이너 로그/이미지/실행 상태를 IDE 안에서 볼 때 |
| Remote Development | WSL/SSH/컨테이너 원격 개발 시 |
| Markdown All in One | 문서 작성 비중이 높을 때 |

### 2-3. 개인 추천 우선순위

설치 우선순위를 5개만 꼽으면 아래 순서가 무난하다.

1. GitLens
2. Git Graph
3. Error Lens
4. Prettier
5. EditorConfig

참고 URL:
- GitLens: https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens
- Git Graph: https://marketplace.visualstudio.com/items?itemName=mhutchie.git-graph
- Error Lens: https://marketplace.visualstudio.com/items?itemName=usernamehw.errorlens
- Prettier: https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode
- ESLint: https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint
- EditorConfig: https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig
- Material Icon Theme: https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme

---

## 3. 추천 설정

아래 설정은 질문 조건을 반영해 바로 쓸 수 있는 형태로 정리했다.

### 3-1. 바로 붙여넣어도 되는 `settings.json`

```json
{
  "editor.fontSize": 20,
  "terminal.integrated.fontSize": 18,
  "workbench.colorTheme": "Default High Contrast",

  "workbench.startupEditor": "none",
  "window.restoreWindows": "none",

  "files.autoSave": "afterDelay",
  "files.autoSaveDelay": 300,

  "workbench.editor.enablePreview": false,
  "workbench.editor.enablePreviewFromQuickOpen": false,

  "editor.minimap.enabled": false,
  "editor.renderWhitespace": "selection",
  "editor.guides.indentation": true,
  "editor.stickyScroll.enabled": true,
  "editor.tabSize": 4,
  "editor.wordWrap": "off",

  "editor.formatOnSave": true,
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,

  "explorer.confirmDelete": false,
  "explorer.confirmDragAndDrop": false,

  "git.openRepositoryInParentFolders": "always",
  "diffEditor.ignoreTrimWhitespace": false,

  "terminal.integrated.defaultProfile.windows": "PowerShell"
}
```

### 3-2. 요청하신 항목별 해설

#### 폰트 크기 20pt 정도

- VS Code는 보통 `pt` 대신 숫자 기반 픽셀 값으로 다룬다.
- 체감상 `editor.fontSize: 20`이면 충분히 큰 편이다.
- 터미널도 함께 키우려면 `terminal.integrated.fontSize: 18` 또는 `20`을 같이 쓰는 편이 좋다.

#### 테마: dark high contrast

- 추천 값: `"workbench.colorTheme": "Default High Contrast"`
- 접근성/가독성 관점에서 가장 명확하다.

#### 시작할 때 완전 빈 윈도우

둘 다 같이 넣는 편이 안전하다.

- `"workbench.startupEditor": "none"`
  - 시작 시 Welcome/Getting Started 같은 시작 에디터를 띄우지 않음
- `"window.restoreWindows": "none"`
  - 이전 세션의 폴더/창을 복원하지 않음

즉, VS Code를 켜면 거의 빈 상태에서 시작하도록 만드는 조합이다.

#### 자동 저장

- `"files.autoSave": "afterDelay"`
- `"files.autoSaveDelay": 300`

300ms는 매우 빠른 편이다. 저장 이벤트가 잦아서 불편하면 `500` 또는 `1000`으로 올리면 된다.

#### Explorer에서 파일 클릭 시 현재 탭이 바뀌지 않고 새 탭으로 열리기

핵심은 Preview 탭을 끄는 것이다.

- `"workbench.editor.enablePreview": false`
- `"workbench.editor.enablePreviewFromQuickOpen": false`

VS Code는 기본적으로 Explorer에서 단일 클릭 시 Preview 탭을 재사용하는 경향이 있다. 이 옵션을 끄면 파일을 눌렀을 때 기존 탭 내용을 덮어쓰지 않고 새 에디터 탭으로 유지하기 쉬워진다.

### 3-3. 추가 추천 설정

질문 범위를 넘지 않는 선에서, 실제 체감이 좋은 것만 추렸다.

- `"editor.formatOnSave": true`
  - 저장 시 코드 정리
- `"files.trimTrailingWhitespace": true`
  - 줄 끝 공백 제거
- `"files.insertFinalNewline": true`
  - 파일 끝 개행 자동 유지
- `"editor.minimap.enabled": false`
  - 작은 모니터에서 집중도 개선
- `"editor.stickyScroll.enabled": true`
  - 긴 파일에서 현재 함수/클래스 문맥 유지
- `"git.openRepositoryInParentFolders": "always"`
  - 하위 폴더에서 VS Code를 열어도 저장소 인식이 덜 꼬임
- `"terminal.integrated.defaultProfile.windows": "PowerShell"`
  - Windows에서는 일관성이 좋음

공식 문서 참고:
- Settings: https://code.visualstudio.com/docs/getstarted/settings
- User and Workspace Settings: https://code.visualstudio.com/docs/configure/settings
- Themes: https://code.visualstudio.com/docs/configure/themes

---

## 4. 내장 Git가 불편할 때의 대안

질문하신 포인트는 매우 구체적이다.

- 브랜치를 바꿔 가며 commit list 를 확인하기 불편함
- JetBrains 계열의 Git 도구처럼 한눈에 보기 어렵고 흐름 추적이 답답함

이 문제는 꽤 일반적이다. VS Code 기본 Source Control은 commit, stage, diff, sync 같은 일상 작업에는 충분하지만, `브랜치 히스토리 탐색`과 `시각적 commit graph 탐색`은 전문 Git UI보다 약하다.

### 4-1. 가장 현실적인 선택지

| 대안 | 추천도 | 이유 |
|---|---|---|
| Git Graph | 높음 | VS Code 안에서 브랜치/커밋 그래프 확인에 가장 직접적 |
| GitLens | 높음 | 파일/라인/브랜치 히스토리 추적이 강력 |
| Fork | 매우 높음 | JetBrains류의 GUI Git 경험에 가장 가깝고 Windows에서 편함 |
| lazygit | 중상 | 키보드 중심 사용자에게 빠르고 강력 |
| Git CLI | 중상 | 가장 정확하고 강력하지만 시각성은 약함 |

### 4-2. 추천 조합

#### 조합 A: VS Code 안에서 끝내기

- `Git Graph`
- `GitLens`

이 조합이면 아래 요구를 상당 부분 해결한다.

- 브랜치 그래프 보기
- 특정 브랜치의 커밋 흐름 확인
- 특정 파일의 commit history 확인
- commit 간 diff 추적

다만 JetBrains 수준의 일체형 Git 도구처럼 느껴지지는 않는다. VS Code 확장은 여전히 "편집기 + 확장" 조합이라 UI 일관성은 조금 떨어질 수 있다.

#### 조합 B: 가장 편한 현실적 대안

- 코드 편집: VS Code
- Git 히스토리/브랜치 탐색: `Fork`

이 방식이 실제로 만족도가 높다. 이유는 명확하다.

- 브랜치 목록, commit graph, diff, merge, rebase가 한 화면에서 직관적
- Windows에서 UI 반응성이 좋고 설정이 단순
- JetBrains 내장 Git 도구와 비슷한 "히스토리 탐색 편의성"을 기대하기 좋음

즉, 편집기는 VS Code가 맡고 Git 탐색은 외부 전용 클라이언트가 맡는 식이다.

#### 조합 C: 키보드 중심 파워유저

- VS Code 내장 터미널
- `lazygit`

장점:
- 매우 빠름
- 브랜치/커밋/스테이징 작업 흐름이 일관됨
- 마우스보다 키보드 중심 작업에 강함

단점:
- 초반 진입 장벽이 있음
- JetBrains식 GUI 탐색과는 감성이 다름

### 4-3. 질문에 대한 직접 답

`branch 를 돌면서 commit list 를 확인`하는 작업이 핵심이라면 우선순위는 아래처럼 추천한다.

1. VS Code를 계속 메인 에디터로 쓸 생각이면 `Git Graph`를 먼저 설치
2. 파일/라인 단위 추적이 많으면 `GitLens` 추가
3. 그래도 답답하면 외부 Git 클라이언트를 `Fork`로 분리

즉, "내장 Git 사용성이 나쁜데 대안이 있나?"에 대한 가장 실용적인 답은 아래다.

- VS Code 내부 대안: `Git Graph + GitLens`
- 가장 편한 외부 대안: `Fork`

### 4-4. 참고 URL

- VS Code Source Control: https://code.visualstudio.com/docs/sourcecontrol/overview
- Git Graph: https://marketplace.visualstudio.com/items?itemName=mhutchie.git-graph
- GitLens: https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens
- Fork: https://git-fork.com/
- lazygit: https://github.com/jesseduffield/lazygit

---

## 5. 개인 추천 세팅안

질문 내용을 기준으로 하면 아래 조합이 가장 무난하다.

### 5-1. 라이트한 추천안

- 단축키는 `Ctrl+P`, `Ctrl+Shift+P`, `Ctrl+Shift+F`, `F12`, `Ctrl+/`부터 익히기
- 익스텐션은 `GitLens`, `Git Graph`, `Error Lens`, `Prettier`, `EditorConfig`
- 설정은 `fontSize 20`, `High Contrast`, `startup none`, `restore none`, `autoSave 300ms`, `preview off`

### 5-2. Git까지 포함한 강추천안

- VS Code는 편집용으로 최적화
- Git은 `Fork` 또는 `Git Graph + GitLens`로 보강

특히 JetBrains Git UX가 그리운 경우에는 `VS Code 내장 Git만으로 버티기`보다 `Fork 병행`이 더 만족스럽다.

---

## 6. 사실검증 메모

이번 정리에서 핵심 설정값과 도구 분류는 아래 공식/공식에 준하는 출처를 우선으로 대조했다.

- VS Code 공식 문서
  - settings, themes, source control, default keybindings
- Visual Studio Marketplace
  - GitLens, Git Graph, Prettier, ESLint, Error Lens, EditorConfig, Material Icon Theme
- 각 도구 공식 사이트
  - Fork
  - lazygit GitHub 저장소

따라서 설정 키 이름과 추천 도구 링크는 2026-03-12 기준으로 신뢰 가능하다고 봐도 된다.

---

## 참고 URL 모음

- https://code.visualstudio.com/docs/reference/default-keybindings
- https://code.visualstudio.com/docs/getstarted/settings
- https://code.visualstudio.com/docs/configure/settings
- https://code.visualstudio.com/docs/configure/themes
- https://code.visualstudio.com/docs/sourcecontrol/overview
- https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens
- https://marketplace.visualstudio.com/items?itemName=mhutchie.git-graph
- https://marketplace.visualstudio.com/items?itemName=usernamehw.errorlens
- https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode
- https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint
- https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig
- https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme
- https://git-fork.com/
- https://github.com/jesseduffield/lazygit

---

## 사용자 질문 프롬프트

```text
$hhd-research 

think hard

주제 : windows 에서 vscode 사용법
- 자주 사용하는 단축키
- 많이 설치하는 익스텐션
- 추천 설정
  - 폰트크기를 20pt 정도 설정
  - 테마 : dark high contrast
  - 시작할때 빈 프로젝트, 완전 빈 윈도우로 시작
  - 자동으로 혹은 아주 짧은 시간만에 자동 저장
  - explorer 에서 파일을 클릭하면 현재탭의 내용이 변경되지 않고, 새로운 탭에 파일이 보이면 좋겠음.
  - 그외 추천 설정
- 내장 git 기능을 사용할때 사용성이 나쁜데 이를 위한 대안은?
  - branch 를 돌면서 commit list 를 확인할때 사용성이 나쁨.
  - jetbrain 내장 git 도구처럼 편하지 않음.
```
