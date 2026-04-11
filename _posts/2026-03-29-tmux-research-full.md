---
layout: post
title: "260329 tmux 리서치 총정리"
comments: true
tags:
- tmux
---

# 260329 tmux 리서치 총정리

안녕하세요! 👋  
이번 문서는 현재 세션에서 다룬 **tmux 관련 리서치와 실전 Q&A**를 한 번에 볼 수 있도록 정리한 통합 문서입니다.  
핵심은 "실무에서 바로 쓰는 tmux" 입니다. 🚀

---

## 1) tmux 소개

`tmux`는 **터미널 멀티플렉서**입니다.

- 하나의 터미널 화면에서 여러 작업을 동시에 수행할 수 있습니다.
- SSH 연결이 끊겨도 세션이 살아 있어, 나중에 다시 붙어서(`attach`) 이어서 작업할 수 있습니다.
- 구조는 `server -> session -> window -> pane` 계층으로 이해하면 쉽습니다.

### 빠른 개념도 (삽화)

```text
tmux server
  └─ session (예: dev)
      ├─ window 1
      │   ├─ pane 1 (shell)
      │   └─ pane 2 (logs)
      └─ window 2
          ├─ pane 1 (editor)
          └─ pane 2 (test)
```

---

## 2) 역사

- 개발자: **Nicholas Marriott**
- 초기 공개: **2007-11-20**
- 최신 안정 릴리스(리서치 시점): **3.6a (2025-12-05)**
- OpenBSD 기반 생태계에서 시작해 Linux/macOS/BSD로 널리 확장되었습니다.

---

## 3) 특징

- 💾 **세션 지속성**: detach 후 재접속 가능
- 🧩 **분할 작업**: window/pane으로 병렬 작업
- 🧠 **키 바인딩 중심 UX**: 빠른 손동작 작업
- 🛠️ **자동화 친화적**: `tmux` 명령 + `~/.tmux.conf`
- 👥 **다중 클라이언트**: 같은 세션에 여러 클라이언트 접속 가능

---

## 4) 장단점

### 장점 ✅

- SSH 환경에서 매우 안정적
- 멀티태스킹 생산성 높음
- 장시간 작업 복구/유지에 강함
- 팀 공용 운영 세션 운용 가능

### 단점 ⚠️

- 처음엔 키맵/개념 학습 필요
- 설정(`tmux.conf`)이 길어질 수 있음
- GUI 툴보다 초반 진입장벽 존재

---

## 5) 자주 쓰는 명령어

```bash
tmux new -s dev
tmux ls
tmux attach -t dev
tmux kill-session -t dev

tmux split-window -h
tmux split-window -v
tmux select-layout tiled
tmux resize-pane -Z
tmux list-keys
```

---

## 6) 단축키: `Ctrl-b + ?` 의미

`Ctrl-b ?`는 **기본 키 바인딩 목록을 보여주는 도움말**입니다.  
즉, tmux 내부의 단축키 세트를 즉시 확인할 수 있는 entry point입니다. 🔎

---

## 7) 기본 단축키 세트 (핵심)

> 기본 prefix: `Ctrl-b`

### 생성/종료/이동

- `Ctrl-b c`: 새 window 생성
- `Ctrl-b n / p / l`: 다음/이전/직전 window 이동
- `Ctrl-b &`: 현재 window 종료
- `Ctrl-b d`: detach

### pane 관련

- `Ctrl-b %`: 좌우 분할 pane 생성
- `Ctrl-b "`: 상하 분할 pane 생성
- `Ctrl-b x`: 현재 pane 종료
- `Ctrl-b z`: pane zoom/restore 토글
- `Ctrl-b q`: pane 번호 일시 표시
- `Ctrl-b o`: 다음 pane 이동
- `Ctrl-b ;`: 이전 활성 pane으로 이동
- `Ctrl-b { / }`: pane 위치 교환

### 레이아웃/리사이즈

- `Ctrl-b Space`: preset layout 순환
- `Ctrl-b M-1..M-7`: 특정 preset layout 선택
- `Ctrl-b C-방향키`: pane 크기 1칸 조절
- `Ctrl-b M-방향키`: pane 크기 5칸 조절

### 실행/도움말

- `Ctrl-b :`: tmux command prompt
- `Ctrl-b ?`: 키 바인딩 목록 보기

---

## 8) pane를 자세히 쓰는 방법

### 8-1. 기본 흐름

1. pane 생성 (`%`, `"`)  
2. pane 간 이동 (방향키 / `o` / `;`)  
3. 필요시 `z`로 집중 모드  
4. `Space` 또는 `select-layout tiled`로 정렬

### 8-2. pane 이동 방식 (방향키 외)

- `Ctrl-b o`: 순환 이동
- `Ctrl-b ;`: 직전 pane 복귀
- `Ctrl-b q` + 숫자: 번호로 직접 이동
- 명령 방식: `select-pane -t {left-of,right-of,up-of,down-of}`
- mouse mode ON 시 클릭 이동 가능

### 8-3. `Ctrl-b Space` 정렬 기준

`Space`는 **미리 정의된 layout preset을 순환**합니다.

- `even-horizontal`
- `even-vertical`
- `main-horizontal`
- `main-vertical`
- `tiled`
- (버전에 따라 mirrored 계열 포함)

즉, tmux가 현재 pane 개수/창 크기에 맞춰 가능한 레이아웃을 순차 적용합니다.

### 8-4. `Ctrl-b z` zoom / restore

- 현재 pane을 크게 확대(zoom)
- 다시 누르면 기존 분할 상태로 복원
- 작업 컨텍스트를 잃지 않고 특정 pane에 집중할 때 매우 유용

---

## 9) window 여러 개를 pane으로 합치기

tmux 기본에는 "윈도우들을 한 번에 pane으로 병합"하는 단일 기본키는 없습니다.

실전에서는 `join-pane` 반복 + `select-layout tiled` 조합을 씁니다.

```bash
for w in {1..7}; do tmux join-pane -s :$w -t :0; done
tmux select-layout -t :0 tiled
```

> 인덱스 변동이 있을 수 있어 실제 운영에서는 `window_id` 기준 스크립트가 더 안전합니다.

---

## 10) 12개 pane를 빠르게 구성 (3x4 근사)

```bash
for i in $(seq 1 11); do tmux split-window -t :.; done
tmux select-layout tiled
```

- 기존 1 + 추가 11 = 총 12 pane
- 화면 비율에 따라 3x4 또는 유사 격자 형태로 정렬

---

## 11) 추천 `tmux.conf` (요구사항 반영)

요구사항: prefix 유지 + pane 이동 최적화 + 생성 즉시 균등정렬 + 12-pane 단축키

```tmux
set -g prefix C-b
bind C-b send-prefix

bind -r h select-pane -L
bind -r j select-pane -D
bind -r k select-pane -U
bind -r l select-pane -R

bind -r Left  select-pane -L
bind -r Down  select-pane -D
bind -r Up    select-pane -U
bind -r Right select-pane -R

unbind '"'
unbind %
bind '"' split-window -v \; select-layout tiled
bind %   split-window -h \; select-layout tiled

bind = select-layout tiled

bind P run-shell 'target="$(tmux display-message -p "#{session_name}:#{window_index}")"; \
while [ "$(tmux list-panes -t "$target" | wc -l)" -lt 12 ]; do \
  tmux split-window -t "$target"; \
done; \
tmux select-layout -t "$target" tiled'

bind r source-file ~/.tmux.conf \; display-message "tmux.conf reloaded"
```

---

## 12) 사실 / 추정 / 검증필요

### 사실 ✅

- `Ctrl-b z`는 zoom 토글입니다.
- `Ctrl-b Space`는 preset layout 순환입니다.
- `Ctrl-b ?`는 key bindings 조회입니다.
- pane 이동은 방향키 외 `o`, `;`, 번호 선택 등 다양한 방식이 있습니다.

### 추정 🤔

- 사용자의 작업 흐름상 `tiled` + zoom 토글 + last-pane 왕복 조합이 가장 효율적일 가능성이 큽니다.

### 검증필요 🔬

- 사용 환경의 tmux 버전/커스텀 설정에 따라 기본키가 일부 다를 수 있음 (`tmux list-keys`로 최종 확인 권장).

---

## 13) 참고 URL

- tmux 공식 저장소: https://github.com/tmux/tmux
- tmux 매뉴얼(man page): https://man7.org/linux/man-pages/man1/tmux.1.html
- tmux 변경 이력(CHANGES): https://raw.githubusercontent.com/tmux/tmux/master/CHANGES
- Wikipedia 요약: https://en.wikipedia.org/wiki/Tmux
- 공식 문서 홈페이지: https://tmux.github.io/

---

## 14) 마무리

이번 리서치의 결론은 간단합니다. ✨  
tmux는 "키맵을 조금만 익히면" 서버/원격/장시간 작업에서 압도적인 생산성을 주는 도구입니다.  
특히 pane 전략(이동, 레이아웃, zoom)을 익히면 체감이 급격히 좋아집니다.

---

## 작성 시 사용한 사용자 질문 프롬프트

```text
hhd-research 
hhd-md

think ultra hard

주제 : tmux 

- 소개 
- 역사
- 특징
- 장단점
- 명령어
- 단축키

추가 요구 사항
- ctrl b + ?
  - 이러한 단축키 명령어 셋트 모두 알려주세요
- pane 을 사용하는 자세한 방법
- ctrl b 방향키 : pane 간 이동
  - pane 간 이동 방식이 더 있는지?
- ctrl b space 
  - pane 이 보기 좋게 정렬되던데 어떤 기준인지?
- ctrl b z 
  - zoom / restore

hhd-md 

현재세션의 tmux 관련 리서치 내용 전부
```
