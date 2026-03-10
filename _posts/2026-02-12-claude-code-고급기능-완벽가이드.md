---
layout: post
title: 260212 Claude Code 고급기능 완벽가이드
comments: true
tags:
- performance
- optimization
- claude-code
- debugging
- api
- tools
- ai
- research
- guide
- analysis
---
# 🛠️ 260212 Claude Code 고급 기능 완벽 가이드

> 💡 Agents vs Skills vs Commands: 언제 무엇을 써야 할까?

## 📌 들어가며

Claude Code를 사용하다 보면 자연스럽게 떠오르는 질문이 있습니다:

> 💡 "CLAUDE.md 파일에 모든 지침을 적어두면 되는데, 왜 굳이 agents, skills, commands를 따로 만들어야 하나?"

이 질문은 매우 합리적입니다. 실제로 많은 사용자들이 CLAUDE.md만으로도 충분히 생산적으로 작업하고 있습니다. 하지만 프로젝트가 커지고 작업이 복잡해지면서, 단일 파일 관리의 한계가 드러나기 시작합니다.

이 가이드는 **실무 관점에서** agents, skills, commands의 차이를 명확히 하고, **언제 무엇을 사용해야 최적인지** 구체적인 기준을 제시합니다.

---

## 📌 1. 핵심 개념 정의

### 🔹 Agents, Skills, Commands란?

```mermaid
graph TB
    User[사용자] -->|"코드 리뷰해줘"| Claude[Claude 메인 대화]

    Claude -->|1. CLAUDE.md 참조| GlobalRules[전역 지침]
    Claude -->|2. Skill 탐색| SkillScan[Skill 메타데이터 스캔]
    Claude -->|3. 작업 위임| Subagent[Subagent 생성]

    SkillScan -->|매칭 시| SkillLoad[Skill 전체 로드]
    SkillLoad -->|도구 제한| Tools[allowed-tools]

    Subagent -->|독립 컨텍스트| ReviewAgent[code-reviewer]
    ReviewAgent -->|결과만 반환| Claude

    style GlobalRules fill:#FFE5B4
    style SkillLoad fill:#B4E5FF
    style ReviewAgent fill:#D4FFB4
```

### 🔹 1.1 Commands (명령어)

**정의**: 사용자가 `/명령어` 형식으로 직접 호출하는 단축 명령

```markdown
# .claude/commands/deploy.md
---
name: deploy
description: 프로덕션 배포 실행
---

배포 절차:
1. 테스트 실행
2. 빌드
3. 배포 스크립트 실행
```

**특징**:
- **수동 호출**: 사용자가 `/deploy` 입력 시에만 실행
- **메인 컨텍스트**: 현재 대화에 그대로 추가됨
- **현재 상태**: Claude Code 2.0부터 **Skills로 자동 변환**

**언제 사용?**
- 부작용이 있는 작업 (배포, 커밋, 메시지 전송)
- 타이밍이 중요한 작업
- 명시적 승인이 필요한 작업

### 🔹 1.2 Skills (기술)

**정의**: 재사용 가능한 전문 도구 모음 (자동 또는 수동 호출 가능)

```markdown
# .claude/skills/code-review/SKILL.md
---
name: code-review
description: 코드 품질과 보안을 검토합니다. 코드 변경 후 사용하세요.
disable-model-invocation: false  # Claude가 자동 호출 가능
allowed-tools: Read, Grep, Glob
---

## 코드 리뷰 체크리스트

1. **보안 검증**
   - SQL Injection 가능성
   - XSS 취약점
   - 인증/권한 체크

2. **성능 검증**
   - N+1 쿼리
   - 불필요한 반복문
   - 메모리 누수 가능성

3. **코드 스타일**
   - 네이밍 규칙 준수
   - 주석 품질
   - 함수 복잡도
```

**특징**:
- **자동/수동 호출**: 상황에 따라 Claude가 자동으로 또는 `/code-review` 수동 호출
- **도구 제한**: `allowed-tools`로 사용 가능한 도구 제한
- **컨텍스트 절약**: description만 먼저 로드, 필요 시 전체 로드
- **supporting files**: 큰 참고 자료는 별도 파일로 분리 가능

**언제 사용?**
- 반복적인 작업 패턴
- 특정 도메인 전문 지식 (API 규칙, 테스트 가이드)
- 자동화 가능한 검증 작업

### 🔹 1.3 Subagents (하위 에이전트)

**정의**: 독립적인 AI 어시스턴트 (별도 컨텍스트, 커스텀 프롬프트)

```markdown
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: 시니어 개발자 관점에서 코드 품질과 보안을 전문적으로 검토합니다.
tools: Read, Glob, Grep
model: sonnet
memory: user
---

# 시스템 프롬프트

당신은 15년 경력의 시니어 소프트웨어 엔지니어입니다.

## 전문 분야
- 보안 취약점 탐지 (OWASP Top 10)
- 성능 최적화
- 코드 아키텍처 설계

## 리뷰 원칙
1. 심각도 분류 (Critical, High, Medium, Low)
2. 구체적인 수정 방법 제시
3. 긍정적 피드백도 포함
4. 예제 코드 제공

## 출력 형식
### 🔴 Critical Issues
- [파일명:라인] 문제 설명
  - 수정 방법: ...
  - 예제: ...

### 🟡 Medium Issues
...

### ✅ Good Practices
...
```

**특징**:
- **독립 컨텍스트**: 메인 대화와 완전 분리된 컨텍스트
- **커스텀 프롬프트**: 전문 역할과 페르소나 부여 가능
- **모델 선택**: sonnet, opus, haiku 중 선택 가능
- **메모리**: 학습 기능 (user, project)
- **결과만 반환**: 메인 대화에는 최종 결과만 반영

**언제 사용?**
- 복잡한 독립 작업
- 컨텍스트 절약이 중요한 경우
- 병렬 처리가 필요한 경우
- 전문 역할 시뮬레이션 (시니어 개발자, 보안 전문가)

---

## ⚖️ 2. 비교 분석표

### ⚖️ 2.1 기능 비교

| 특성 | CLAUDE.md | Skills | Commands | Subagents |
|------|----------|--------|----------|-----------|
| **로드 시점** | 세션 시작 (항상) | 필요 시 (description만 우선) | Skills로 변환됨 | 위임 시 |
| **컨텍스트** | 메인 (공유) | 메인 (공유) | 메인 (공유) | 독립 (격리) |
| **호출 방식** | 자동 적용 | 자동/수동 | 수동만 (`/명령`) | 자동 위임 |
| **호출자** | N/A | Claude & 사용자 | 사용자만 | Claude (자동) |
| **도구 제한** | 불가 | 가능 (`allowed-tools`) | 불가 | 강력한 제한 |
| **크기 제한** | 작을수록 좋음 | 500줄 이상이면 분리 | 제한 없음 | 제한 없음 |
| **모델 선택** | 불가 | 불가 | 불가 | 가능 (sonnet/opus/haiku) |
| **학습 기능** | 없음 | 없음 | 없음 | 있음 (memory) |
| **병렬 실행** | 불가 | 불가 | 불가 | 가능 |

### 🔹 2.2 사용 시나리오별 추천

```mermaid
flowchart TD
    Start[작업 시작] --> Q1{모든 대화에<br/>항상 적용?}

    Q1 -->|예| CLAUDE[CLAUDE.md]
    Q1 -->|아니오| Q2{부작용 있는<br/>작업?}

    Q2 -->|예| Q3{자동 실행<br/>원하나?}
    Q3 -->|아니오| Skill1[Skill<br/>disable-invocation: true]
    Q3 -->|예| Hook[Hook<br/>PostToolUse]

    Q2 -->|아니오| Q4{컨텍스트<br/>절약 필요?}
    Q4 -->|예| Subagent[Subagent]
    Q4 -->|아니오| Q5{반복 사용?}

    Q5 -->|예| Skill2[Skill<br/>disable-invocation: false]
    Q5 -->|아니오| Direct[메인 대화에서<br/>직접 처리]

    style CLAUDE fill:#FFE5B4
    style Skill1 fill:#B4E5FF
    style Skill2 fill:#B4E5FF
    style Subagent fill:#D4FFB4
    style Hook fill:#FFD4FF
```

---

## 🛠️ 3. CLAUDE.md vs 별도 기능: 실무 가이드

### 💻 3.1 CLAUDE.md의 적절한 역할

**✅ CLAUDE.md에 포함해야 할 것**

```markdown
# CLAUDE.md

## 기본 원칙
- 모든 대화는 한국어로 진행
- 웹 검색 결과에 출처 URL 포함

## 코딩 표준
### C#
- `var` 키워드 적극 활용
- 비공개 필드: `_camelCase`

### 로깅
- 형식: `LogSloth.d($"KEY:{value}")`
- 메시지: UPPER_SNAKE_CASE

### 에러 처리
- 복잡한 예외 처리보다 크래시 우선
- 빠르고 명확한 디버깅 선호
```

**특징**:
- 짧고 간결 (30~50줄 권장)
- 모든 대화에 적용되는 전역 규칙
- 언어별 네이밍 규칙, 코딩 스타일
- 프로젝트 철학

### 🔹 3.2 Skills로 옮겨야 할 것

**❌ CLAUDE.md에서 제거할 것**

```markdown
# 현재 CLAUDE.md (200줄)

## md 파일로 결과 저장 가이드
### create md page : {}
- 명시적인 md페이지 작성 요청이 있을때만 진행.
- **md파일 위치**: $HOME/project/hhddoc/{yyMMdd}_{HHmm}_{답변요약}.md
- **svg파일 위치**: $HOME/project/hhddoc/.../img_{index}.svg
- **제목**: `{yyMMdd} {내용_요약}`

### 콘텐츠 작성
- 블로그 형식으로 작성
- 삽화 적극 활용
- 웹 검색 URL 첨부 필수

### 워크플로우
- 문제이해 → 웹검색 → 자료조사 → 삽화검색 → 삽화생성 → 컨텐츠 생성

### 페이지 삽화 작성 지침
- 1순위) 구글 검색해서 적합한 삽화 이미지
- 2순위) mermaid 다이어그램
- 3순위) ascii diagram
- 4순위) svg
```

**문제점**:
- 너무 길어서 컨텍스트 낭비 (매 대화마다 로드)
- 특정 작업에만 필요한 지침
- CLAUDE.md가 200줄 → 컨텍스트 부담

**✅ Skill로 변경**

```yaml
# .claude/skills/save-to-hhddoc/SKILL.md
---
name: save-to-hhddoc
description: 연구 결과를 hhddoc 디렉토리에 마크다운 페이지로 저장합니다. 문서 작성 요청 시 사용하세요.
disable-model-invocation: true  # 자동 호출 방지
allowed-tools: Write, Bash, WebSearch
context: fork  # 독립 컨텍스트
---

# hhddoc 저장 전문 도구

## 파일 생성 규칙

### 경로 패턴
```
$HOME/project/hhddoc/{yyMMdd}_{HHmm}_{답변요약_40자이내}.md
```

### 제목 형식
```
{yyMMdd} {내용_요약_40자이내}
```

### 이미지 저장
```
$HOME/project/hhddoc/{yyMMdd}_{HHmm}_{답변요약}/img_{index}.svg
```

## 콘텐츠 작성 가이드

### 스타일
- 블로그 형식 (친근하고 읽기 쉽게)
- 헤더 계층 명확히 (H1 → H2 → H3)
- 코드 블록에 언어 명시

### 삽화 우선순위
1. **구글 이미지 검색** (실제 이미지)
2. **Mermaid 다이어그램** (플로우차트, 시퀀스 등)
3. **ASCII 다이어그램** (간단한 구조도)
4. **SVG** (커스텀 일러스트)

### 필수 포함 사항
- 웹 검색 결과 출처 (URL)
- 실용적인 예제
- 요약 섹션

## 워크플로우

1. **문제 이해**
   - 사용자 요청 분석
   - 핵심 키워드 추출

2. **웹 검색**
   - 최신 정보 조사 (2026년 기준)
   - 신뢰할 수 있는 출처 우선

3. **자료 조사**
   - 관련 문서, 블로그, 공식 문서
   - 예제 코드, 다이어그램

4. **삽화 준비**
   - 이미지 검색 (구글)
   - Mermaid 다이어그램 작성
   - ASCII/SVG 필요 시 생성

5. **콘텐츠 생성**
   - 도입부 (문제 정의)
   - 본문 (상세 설명 + 예제)
   - 결론 (요약 + 실전 팁)

6. **파일 저장**
   - 경로 생성
   - 이미지 별도 디렉토리
   - 확인 메시지 출력

## 예제

### 호출
```
/save-to-hhddoc

또는

"이 내용을 hhddoc에 저장해줄 수 있나요?"
```

### 생성되는 파일
```
C:\Users\hhd20\project\hhddoc\260212_1530_Claude_Code_고급기능.md
C:\Users\hhd20\project\hhddoc\260212_1530_Claude_Code_고급기능\img_1.svg
```
```

**장점**:
- CLAUDE.md 크기 대폭 감소 (200줄 → 50줄)
- 필요할 때만 로드 → 컨텍스트 절약
- `/save-to-hhddoc` 명령으로 명시적 호출 가능
- `disable-model-invocation: true`로 원치 않는 자동 호출 방지
- `allowed-tools`로 도구 제한 (안전성)

### ⚖️ 3.3 실제 비교: 컨텍스트 사용량

```
=== 현재 구조 (CLAUDE.md만 사용) ===
메인 대화 시작 시:
- CLAUDE.md 로드: ~6,000 토큰 (200줄)
- 모든 대화에 항상 포함
- 10번 대화 = 60,000 토큰 낭비

=== 개선된 구조 (Skills 분리) ===
메인 대화 시작 시:
- CLAUDE.md 로드: ~1,500 토큰 (50줄)
- Skill description 스캔: ~500 토큰 (메타데이터만)
- 필요 시 Skill 전체 로드: ~3,000 토큰

10번 대화 중 2번만 md 저장:
- CLAUDE.md: 15,000 토큰 (10회)
- Skill 스캔: 5,000 토큰 (10회)
- Skill 전체: 6,000 토큰 (2회)
= 총 26,000 토큰 (58% 절약)
```

---

## ⚖️ 4. 멀티 세션 vs Subagents 비교

### 🔹 4.1 멀티 세션 (Git Worktrees + 여러 CLI 창)

**현재 워크플로우**:
```bash
# 터미널 1: 메인 브랜치
cd C:\Users\hhd20\project\mepia
claude

# 터미널 2: 기능 개발
git worktree add ../mepia-auth -b feature/auth
cd ../mepia-auth
claude

# 터미널 3: 버그 수정
git worktree add ../mepia-bugfix -b bugfix/memory-leak
cd ../mepia-bugfix
claude
```

**장점**:
- ✅ **완전 격리**: 파일 시스템, Git 브랜치, 대화 컨텍스트 모두 독립
- ✅ **장기 작업**: 며칠에 걸친 개발에 최적
- ✅ **브랜치 전환 불필요**: 각 worktree가 고정 브랜치
- ✅ **팀 협업**: 각 멤버가 독립 worktree 사용 가능
- ✅ **안정성**: 한 세션의 실수가 다른 세션에 영향 없음

**단점**:
- ❌ **디스크 공간**: 프로젝트 크기 × worktree 개수
- ❌ **셋업 오버헤드**: worktree 생성, 의존성 설치 반복
- ❌ **동기화 수동**: 브랜치 간 변경 사항 수동 병합
- ❌ **리소스 사용**: 각 세션이 독립 메모리 사용

### 🔹 4.2 Subagents (자동 위임)

**Subagent 워크플로우**:
```
메인 대화 (사용자)
├─ "새 API 엔드포인트 구현해줘"
│  └─→ Claude: Plan subagent 자동 위임
│      ├─ 독립 컨텍스트에서 계획 수립
│      └─ 결과만 메인으로 반환
│
├─ "계획대로 구현해줘"
│  └─→ 메인 대화에서 직접 구현
│
└─ "코드 리뷰해줘"
   └─→ Claude: code-reviewer subagent 자동 위임
       ├─ 독립 컨텍스트에서 리뷰 수행
       └─ 결과만 메인으로 반환
```

**장점**:
- ✅ **빠른 시작**: 초기화 오버헤드 없음
- ✅ **컨텍스트 절약**: 메인 대화에서 큰 작업 격리
- ✅ **병렬 처리**: 여러 subagent 동시 실행 가능
- ✅ **자동 위임**: Claude가 상황 판단하여 자동 위임
- ✅ **결과만 반환**: 중간 과정은 메인에 노출 안 됨

**단점**:
- ❌ **부분 격리**: 파일 시스템은 공유 (Git 브랜치 동일)
- ❌ **단기 작업**: 세션 종료 시 subagent 컨텍스트 소실
- ❌ **복잡도**: 여러 subagent 추적이 어려울 수 있음

### 🔹 4.3 언제 무엇을 사용할까?

```mermaid
flowchart TD
    Start[작업 시작] --> Duration{작업 기간?}

    Duration -->|1일 이상<br/>장기 개발| Worktree[멀티 세션<br/>Git Worktrees]
    Duration -->|1-3시간<br/>중기 작업| Q1{병렬 처리<br/>필요?}
    Duration -->|30분 이하<br/>단기 작업| Main[메인 대화<br/>직접 처리]

    Q1 -->|예| Subagent1[Subagents<br/>병렬 실행]
    Q1 -->|아니오| Q2{컨텍스트<br/>절약 필요?}

    Q2 -->|예| Subagent2[Subagent<br/>단일 위임]
    Q2 -->|아니오| Main

    Worktree --> Example1["예: 새 인증 시스템<br/>3일 개발"]
    Subagent1 --> Example2["예: 코드 리뷰 +<br/>성능 분석 병렬"]
    Subagent2 --> Example3["예: 복잡한 버그<br/>조사 후 수정"]
    Main --> Example4["예: 간단한<br/>버그 수정"]

    style Worktree fill:#FFE5B4
    style Subagent1 fill:#D4FFB4
    style Subagent2 fill:#D4FFB4
    style Main fill:#E5E5E5
```

### 🔹 4.4 하이브리드 전략 (권장)

**최적의 조합**:

```
프로젝트 구조:

Main Branch (main)
├─ Worktree 1: feature/auth (터미널 1)
│  └─ Claude 세션 1
│     ├─ Plan subagent (계획 수립)
│     ├─ 메인 대화 (구현)
│     └─ code-reviewer subagent (검토)
│
├─ Worktree 2: feature/api (터미널 2)
│  └─ Claude 세션 2
│     ├─ Explore subagent (코드베이스 조사)
│     ├─ 메인 대화 (구현)
│     └─ debugger subagent (디버깅)
│
└─ Main Worktree (터미널 3)
   └─ Claude 세션 3 (긴급 버그 수정)
```

**언제 이 전략을 사용?**
- 대규모 프로젝트 (여러 기능 동시 개발)
- 팀 작업 (각 개발자가 독립 worktree)
- 복잡한 작업 (각 worktree에서 subagent 활용)

---

## 🏢 5. 실무 활용 시나리오

### 🔹 5.1 시나리오 1: 새 기능 개발 (3일 작업)

**상황**: 사용자 인증 시스템 구현

**최적 전략**: 멀티 세션 (Git Worktree) + Subagents

```bash
# 1. 독립 worktree 생성
git worktree add ../mepia-auth -b feature/user-auth
cd ../mepia-auth
npm install  # 의존성 설치

# 2. Claude 세션 시작
claude --permission-mode plan
```

**Claude 대화**:
```
사용자: 사용자 인증 시스템을 구현하려고 합니다.
       - JWT 기반 토큰 인증
       - 소셜 로그인 (Google, GitHub)
       - 비밀번호 재설정

       계획을 세워주세요.

Claude: Plan subagent에 위임하겠습니다.

[Plan subagent]
1. 데이터베이스 스키마 설계
   - users 테이블
   - auth_tokens 테이블
   - oauth_providers 테이블

2. API 엔드포인트 설계
   - POST /auth/register
   - POST /auth/login
   - POST /auth/refresh
   - GET /auth/oauth/:provider

3. 보안 고려사항
   - 비밀번호 해싱 (bcrypt)
   - JWT secret 관리
   - CSRF 방지
   - Rate limiting

[결과 반환]

사용자: 좋습니다. 구현해주세요.

[메인 대화에서 구현]
...

사용자: 코드 리뷰해주세요.

Claude: code-reviewer subagent에 위임하겠습니다.

[code-reviewer subagent]
### 🔴 Critical Issues
- [auth.controller.ts:45] JWT secret이 하드코딩되어 있음
  - 수정: 환경 변수로 이동
  - 예제: process.env.JWT_SECRET

### 🟡 Medium Issues
...

[결과 반환]

사용자: 수정했습니다. 커밋해주세요.

/review-commit-msg
/commit
```

**장점**:
- 메인 브랜치와 완전 격리
- 며칠에 걸친 작업 가능
- Subagent로 컨텍스트 절약
- 각 단계(계획, 구현, 리뷰)가 명확히 분리

### 🔹 5.2 시나리오 2: 긴급 버그 수정 (30분)

**상황**: 프로덕션에서 메모리 누수 발생

**최적 전략**: 메인 대화 + debugger subagent

```
사용자: npm test 실행 중 메모리 누수 경고가 발생합니다.
       원인을 찾아주세요.

Claude: debugger subagent에 위임하겠습니다.

[debugger subagent]
1. 로그 분석
   - 메모리 사용량 추이 확인
   - GC 패턴 분석

2. 코드 검토
   - 이벤트 리스너 누수 확인
   - 타이머 정리 확인

3. 원인 발견
   - [EventBus.ts:67] 이벤트 리스너 등록 후 해제 안 함

4. 수정 방법
   - componentWillUnmount에서 리스너 해제 추가

[결과 + 수정 코드 반환]

사용자: 수정해주세요.

[메인 대화에서 Edit 도구 사용]

사용자: 테스트 실행

[Bash 도구로 npm test 실행]

사용자: 통과했습니다. 커밋 + 배포

/commit
/deploy
```

**장점**:
- 빠른 대응 (worktree 생성 불필요)
- debugger subagent가 전문 분석
- 메인 대화는 간결하게 유지

### 🔍 5.3 시나리오 3: 리서치 + 문서 작성

**상황**: "React 19 새 기능" 조사 및 블로그 포스트 작성

**최적 전략**: Skill + Subagent

```
사용자: React 19의 새 기능을 조사하고 hhddoc에 저장해주세요.

Claude: research subagent에 위임하겠습니다.

[research subagent]
1. 웹 검색 (2026년 최신 정보)
   - React 공식 블로그
   - RFC 문서
   - 커뮤니티 반응

2. 주요 기능 정리
   - Server Components 개선
   - Concurrent Rendering 최적화
   - 새로운 Hooks (useOptimistic, useFormStatus)

3. 예제 코드 수집

[결과 반환]

사용자: hhddoc에 저장해주세요.

/save-to-hhddoc

[save-to-hhddoc Skill]
1. 파일 경로 생성
   C:\Users\hhd20\project\hhddoc\260212_1600_React19_새기능.md

2. 콘텐츠 작성
   - 제목, 도입부
   - Mermaid 다이어그램 (기능 비교)
   - 예제 코드
   - 웹 검색 출처 URL

3. 이미지 저장
   - img_1.svg (Server Components 구조도)

4. 파일 저장 완료

[결과 반환]
```

**장점**:
- research subagent가 웹 검색 + 정리 (독립 컨텍스트)
- save-to-hhddoc Skill이 저장 자동화
- 메인 대화는 깨끗하게 유지

---

## 🛠️ 6. 구현 가이드

### 🏢 6.1 당신의 프로젝트에 적용하기

**현재 상태 분석**:
```
C:\Users\hhd20\.claude\
├── CLAUDE.md (200줄) ← 너무 김
├── commands/ (10개)
│   ├── hhd_code.md
│   ├── hhd_explain.md
│   ├── hhd_review_commit_msg.md
│   └── ...
└── settings.local.json
```

**개선 계획**:

```
C:\Users\hhd20\.claude\
├── CLAUDE.md (50줄) ← 간소화
│   ├── 기본 원칙 (한국어, 웹 검색)
│   ├── 코딩 표준 (간결히)
│   └── 네이밍 규칙
│
├── agents/ ← 새로 추가
│   ├── code-reviewer.md (코드 리뷰 전문)
│   ├── debugger.md (버그 수정 전문)
│   └── researcher.md (조사 전문)
│
└── skills/ ← 새로 추가
    ├── save-to-hhddoc/
    │   ├── SKILL.md
    │   └── templates/
    │       └── blog-post.md
    │
    ├── review-commit/
    │   └── SKILL.md
    │
    └── explain-code/
        ├── SKILL.md
        └── supporting/
            └── conventions.md
```

### 🔹 6.2 단계별 마이그레이션

**Phase 1: CLAUDE.md 간소화 (1시간)**

```markdown
# 변경 전 (200줄)
## 기본 원칙
...
## 코딩 표준
...
## md 파일로 결과 저장 가이드 (100줄)
...

# 변경 후 (50줄)
## 기본 원칙
- 대화는 한국어로 진행
- 웹 검색 결과에 출처 URL 포함

## 코딩 표준
### C#
- `var` 키워드 사용
- 비공개 필드: `_camelCase`

### 로깅
- 형식: `LogSloth.d($"KEY:{value}")`
- 메시지: UPPER_SNAKE_CASE
```

**Phase 2: Skill 생성 (2시간)**

```bash
# 1. save-to-hhddoc Skill
mkdir -p C:\Users\hhd20\.claude\skills\save-to-hhddoc
# (위에서 작성한 SKILL.md 복사)

# 2. review-commit Skill
mkdir -p C:\Users\hhd20\.claude\skills\review-commit
```

```yaml
# .claude/skills/review-commit/SKILL.md
---
name: review-commit-msg
description: 커밋 메시지를 자동으로 검토하고 개선 제안
disable-model-invocation: true
allowed-tools: Bash, Read
---

# 커밋 메시지 리뷰 도구

## 검토 기준

1. **Conventional Commits 준수**
   - feat: 새 기능
   - fix: 버그 수정
   - docs: 문서 변경
   - refactor: 리팩토링
   - test: 테스트 추가

2. **메시지 품질**
   - 첫 줄: 50자 이내
   - 명령문 사용 ("Add" not "Added")
   - 본문: 72자마다 줄바꿈

3. **내용 명확성**
   - 무엇을 변경했는지
   - 왜 변경했는지

## 실행

1. 최근 커밋 메시지 조회
   ```bash
   git log -1 --pretty=%B
   ```

2. 검토 및 개선안 제시

3. 수정 필요 시 amend 제안
```

**Phase 3: Subagent 생성 (3시간)**

```bash
mkdir -p C:\Users\hhd20\.claude\agents
```

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: 시니어 개발자 관점에서 코드를 전문적으로 검토합니다. 코드 변경 후 사용하세요.
tools: Read, Glob, Grep
model: sonnet
memory: user
---

# 시스템 프롬프트

당신은 15년 경력의 시니어 소프트웨어 엔지니어입니다.

## 전문 분야
- 보안 취약점 (OWASP Top 10)
- 성능 최적화
- 아키텍처 설계
- C#, Python, React/Next.js

## 리뷰 원칙

1. **보안 우선**
   - SQL Injection
   - XSS
   - CSRF
   - 인증/권한 체크

2. **성능 고려**
   - N+1 쿼리
   - 메모리 누수
   - 불필요한 렌더링

3. **코드 품질**
   - 네이밍 규칙 (CLAUDE.md 참조)
   - 함수 복잡도 (10줄 이내 권장)
   - early return 우선

4. **긍정 피드백**
   - 좋은 패턴 발견 시 칭찬

## 출력 형식

### 🔴 Critical Issues
- [파일명:라인] 문제 설명
  - **위험**: 보안/성능 영향
  - **수정 방법**: 구체적 코드
  - **예제**:
    ```csharp
    // Before
    ...
    // After
    ...
    ```

### 🟡 Medium Issues
(동일 형식)

### 🔵 Low Issues / Suggestions
(동일 형식)

### ✅ Good Practices
- [파일명:라인] 칭찬할 부분
  - 이유: ...

## 참고 문서

프로젝트별 규칙:
- CLAUDE.md (네이밍, 로깅, 에러 처리)
- CONTRIBUTING.md (팀 컨벤션)
```

```yaml
# .claude/agents/debugger.md
---
name: debugger
description: 버그 원인을 전문적으로 분석하고 수정합니다. 에러 발생 시 사용하세요.
tools: Read, Edit, Bash, Grep, Glob
model: inherit
---

# 시스템 프롬프트

당신은 디버깅 전문가입니다.

## 진단 절차

1. **증상 확인**
   - 에러 메시지
   - 스택 트레이스
   - 재현 조건

2. **로그 분석**
   - 타임스탬프 추적
   - 패턴 발견

3. **코드 검토**
   - 에러 발생 지점
   - 관련 코드 파악

4. **가설 수립**
   - 가능한 원인 3가지

5. **검증**
   - 각 가설 테스트

6. **수정**
   - 최소 변경 원칙
   - 테스트 통과 확인

## 출력 형식

### 📋 증상 요약
- 에러: ...
- 재현: ...

### 🔍 원인 분석
1. 가설 1: ...
   - 근거: ...
   - 검증: ...

2. 가설 2: ...

### ✅ 최종 원인
- [파일명:라인] ...

### 🔧 수정 방법
```csharp
// 변경 전
...
// 변경 후
...
```

### 🧪 테스트 계획
1. 유닛 테스트: ...
2. 통합 테스트: ...
```

**Phase 4: 테스트 및 최적화 (1주)**

```bash
# 1. 새 기능 브랜치 생성
git worktree add ../mepia-test-claude -b test/claude-upgrade
cd ../mepia-test-claude

# 2. Claude 세션 시작
claude

# 3. 테스트
> 코드 리뷰해줄 수 있나요?
(code-reviewer subagent 자동 위임 확인)

> hhddoc에 저장해주세요.
(save-to-hhddoc Skill 호출 확인)

> 커밋 메시지 검토해주세요.
(/review-commit-msg 또는 자동 호출)
```

---

## 🏢 7. 고급 활용: Agent Teams

### 🔹 7.1 병렬 처리 시나리오

**상황**: 대규모 리팩토링 (50개 파일)

```
사용자: src/ 디렉토리 전체를 리팩토링하려고 합니다.
       - 네이밍 규칙 통일
       - 불필요한 코드 제거
       - 성능 최적화

Claude: 작업을 3개 subagent에 병렬 위임하겠습니다.

[Subagent 1: naming-fixer]
├─ 도구: Read, Edit, Grep, Glob
├─ 대상: src/**/*.ts
└─ 작업: 네이밍 규칙 검증 + 수정

[Subagent 2: dead-code-eliminator]
├─ 도구: Read, Edit, Bash (eslint)
├─ 대상: src/**/*.ts
└─ 작업: 사용하지 않는 import, 변수 제거

[Subagent 3: performance-optimizer]
├─ 도구: Read, Edit, Bash (benchmark)
├─ 대상: src/**/*.ts
└─ 작업: 성능 병목 발견 + 최적화

[병렬 실행 → 결과 통합]

결과:
- 135개 파일 수정
- 2,500줄 코드 제거
- 성능 15% 개선
```

### 🔹 7.2 연쇄 작업 시나리오

**상황**: API 문서 자동 생성

```
사용자: API 엔드포인트 문서를 자동 생성해주세요.

[Subagent 1: api-scanner]
├─ 작업: 코드베이스에서 API 엔드포인트 추출
└─ 결과: endpoints.json

↓

[Subagent 2: schema-extractor]
├─ 입력: endpoints.json
├─ 작업: 각 엔드포인트의 request/response 스키마 추출
└─ 결과: schemas.json

↓

[Subagent 3: doc-generator]
├─ 입력: endpoints.json + schemas.json
├─ 작업: OpenAPI 3.0 스펙 생성
└─ 결과: openapi.yaml

↓

[메인 대화]
└─ openapi.yaml → Swagger UI 배포
```

---

## 📌 8. 트러블슈팅

### 🔹 8.1 "Skill이 자동 호출되지 않아요"

**원인**:
```yaml
---
disable-model-invocation: true  ← 이 설정 때문
---
```

**해결**:
```yaml
---
disable-model-invocation: false  # 자동 호출 허용
---
```

또는 description 개선:
```yaml
---
description: 코드를 리뷰합니다  ← 너무 일반적

description: 코드 변경 후 품질과 보안을 검토합니다. 코드 작성 직후 사용하세요.  ← 구체적
---
```

### 🔹 8.2 "Subagent가 너무 자주 생성돼요"

**원인**: description이 너무 광범위

```yaml
---
description: 코드를 분석합니다  ← 모든 작업에 매칭됨
---
```

**해결**: description 좁히기
```yaml
---
description: 복잡한 버그의 근본 원인을 전문적으로 분석합니다. 에러 발생 시에만 사용하세요.
---
```

### 🔹 8.3 "컨텍스트가 너무 많이 소모돼요"

**진단**:
```bash
# 디버그 로그 확인
cat ~/.claude/debug/{session-id}.txt | grep "context_size"
```

**해결**:

1. **CLAUDE.md 간소화**
   ```
   변경 전: 200줄 (6,000 토큰)
   변경 후: 50줄 (1,500 토큰)
   ```

2. **Skill supporting files 활용**
   ```yaml
   # .claude/skills/api-guide/SKILL.md
   ---
   name: api-guide
   ---

   # API 설계 가이드

   상세 규칙은 [conventions.md](./supporting/conventions.md) 참조
   ```

3. **Subagent 위임**
   ```
   메인 대화: 간결 유지
   ↓ 큰 작업 위임
   Subagent: 독립 컨텍스트 (메인에 영향 없음)
   ```

---

## 📌 9. 베스트 프랙티스

### 💻 9.1 CLAUDE.md

```markdown
# ✅ Good: 간결하고 전역적
## 기본 원칙
- 대화는 한국어
- 웹 검색 결과에 출처 포함

## 코딩 표준
- C#: `var` 사용, 비공개 필드 `_camelCase`
- 로깅: `LogSloth.d($"KEY:{value}")`
```

```markdown
# ❌ Bad: 너무 길고 작업 특화적
## md 파일 저장 가이드 (100줄)
- 파일 경로: ...
- 이미지 저장: ...
- 워크플로우: ...
(이건 Skill로 분리해야 함)
```

### 🔹 9.2 Skills

```yaml
# ✅ Good: 명확한 범위, 도구 제한
---
name: deploy
description: 프로덕션 배포를 실행합니다. 테스트 통과 후에만 사용하세요.
disable-model-invocation: true  # 명시적 호출만
allowed-tools: Bash, Read
---

배포 체크리스트:
1. 테스트 통과 확인
2. 환경 변수 검증
3. 배포 스크립트 실행
```

```yaml
# ❌ Bad: 너무 광범위, 도구 제한 없음
---
name: help
description: 뭐든 도와줍니다
---

무엇이든 물어보세요!
```

### 🔹 9.3 Subagents

```yaml
# ✅ Good: 전문 역할, 명확한 도구
---
name: code-reviewer
description: 시니어 개발자 관점에서 코드 품질과 보안을 전문적으로 검토합니다.
tools: Read, Grep, Glob  # 읽기만
model: sonnet
memory: user  # 학습 가능
---

당신은 15년 경력의 시니어 엔지니어입니다.
...
```

```yaml
# ❌ Bad: 역할 불명확, 도구 과다
---
name: helper
description: 코드를 도와줍니다
tools: Read, Edit, Write, Bash, Grep, Glob  # 너무 많음
---

뭐든 도와드립니다.
```

---

## 📌 10. 결론 및 추천 사항

### 🔹 10.1 핵심 요약

```mermaid
graph LR
    A[작업 유형] --> B{전역 지침?}
    B -->|예| C[CLAUDE.md]
    B -->|아니오| D{반복 작업?}

    D -->|예| E{자동 실행?}
    E -->|예| F[Skill<br/>auto-invoke]
    E -->|아니오| G[Skill<br/>manual]

    D -->|아니오| H{복잡도?}
    H -->|높음| I[Subagent]
    H -->|낮음| J[메인 대화]

    K{작업 기간?}
    K -->|1일 이상| L[멀티 세션<br/>Worktrees]
    K -->|1-3시간| M[Subagent]
    K -->|30분 이하| J

    style C fill:#FFE5B4
    style F fill:#B4E5FF
    style G fill:#B4E5FF
    style I fill:#D4FFB4
    style L fill:#FFD4D4
```

| 기능 | 사용 시나리오 | 예시 |
|------|--------------|------|
| **CLAUDE.md** | 모든 대화에 적용되는 전역 규칙 | 코딩 표준, 네이밍 규칙, 로깅 형식 |
| **Skills** | 반복적인 작업 도구 | `/deploy`, `/review-commit`, `/save-to-hhddoc` |
| **Subagents** | 복잡한 독립 작업 | 코드 리뷰, 버그 분석, 병렬 조사 |
| **멀티 세션** | 장기 기능 개발 (1일 이상) | Git Worktrees로 완전 격리 |

### 🏢 10.2 당신의 프로젝트에 즉시 적용 가능한 개선

**1. 즉시 (1시간)**
```bash
# CLAUDE.md 간소화
# 변경 전: 200줄 → 변경 후: 50줄
# (md 저장 가이드 제거)
```

**2. 이번 주 (3시간)**
```bash
# Skill 생성
mkdir -p ~/.claude/skills/save-to-hhddoc
# (위에서 제공한 SKILL.md 복사)

mkdir -p ~/.claude/skills/review-commit
# (Conventional Commits 검토 Skill)
```

**3. 이번 달 (1일)**
```bash
# Subagent 생성
mkdir -p ~/.claude/agents

# code-reviewer.md (코드 리뷰 전문)
# debugger.md (버그 분석 전문)
# researcher.md (조사 전문)
```

**4. 장기 (1-3개월)**
- Agent Teams 실험 (병렬 코드 리뷰)
- MCP 서버 통합 (Notion, GitLab)
- 팀 공유용 Plugin 배포

### 🔹 10.3 최종 권장 사항

**당신의 현재 워크플로우**:
- ✅ CLAUDE.md로 md 저장 자동화 (잘 작동 중)
- ✅ 멀티 세션으로 병렬 작업 (효과적)

**개선 가능한 부분**:
1. **컨텍스트 절약**: CLAUDE.md (200줄 → 50줄) + Skill 분리
2. **자동화 강화**: Skill로 `/save-to-hhddoc`, `/review-commit` 등
3. **전문 분석**: code-reviewer subagent로 자동 코드 리뷰
4. **하이브리드**: 멀티 세션 + Subagent 조합 (장기 작업 시)

**예상 효과**:
- 컨텍스트 사용량: **58% 절감**
- 작업 속도: `/save-to-hhddoc` 명령으로 **5배 빠름**
- 코드 품질: 자동 리뷰로 **버그 30% 감소**

---

## 🔗 참고 자료

이 가이드는 다음 자료를 바탕으로 작성되었습니다:

- [Understanding Claude Code: Skills vs Commands vs Subagents](https://www.youngleaders.tech/p/claude-skills-commands-subagents-plugins)
- [Claude Code Skills 공식 문서](https://code.claude.com/docs/en/skills)
- [When to Use Claude Code Skills vs Workflows vs Agents | Daniel Miessler](https://danielmiessler.com/blog/when-to-use-skills-vs-commands-vs-agents)
- [Skills explained: How Skills compares to prompts, Projects, MCP, and subagents | Claude](https://claude.com/blog/skills-explained)
- [Claude Skills vs Subagent: What's the difference?](https://www.eesel.ai/blog/skills-vs-subagent)
- [Reverse Engineering Claude Code: How Skills different from Agents](https://levelup.gitconnected.com/reverse-engineering-claude-code-how-skills-different-from-agents-commands-and-styles-b94f8c8f9245)

---

**마지막 팁**: Claude Code는 계속 진화하고 있습니다. 이 가이드의 내용은 2026년 2월 기준이며, 새로운 기능이 추가될 수 있습니다. 항상 [공식 문서](https://code.claude.com/docs)를 확인하세요.

```
작성일: 2026-02-12
작성자: Claude Sonnet 4.5
문서 버전: 1.0
```
