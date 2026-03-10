---
layout: post
title: 260204 claude json 설정파일 가이드
comments: true
tags:
- performance
- optimization
- react
- claude-code
- api
- tools
- cryptography
- ai
- research
- guide
---
# 🗂️ 260204 Claude.json 설정파일 가이드

## 🧭 소개

`.claude.json` 파일은 Claude Code CLI의 전역 설정 파일로, 사용자의 홈 디렉토리(`C:\Users\hhd20\.claude.json`)에 위치합니다. 이 파일은 Claude Code의 동작 방식, UI 설정, 프로젝트별 권한, 사용 통계 등을 관리하며, Claude Code 실행 시 자동으로 생성 및 업데이트됩니다.

### 🔹 주요 역할
- **사용자 환경 설정**: 자동 업데이트, IDE 연동, 팁 표시 여부 등
- **기능 플래그 관리**: 실험적 기능 활성화/비활성화 (Feature Flags)
- **프로젝트별 설정**: 각 프로젝트의 권한, MCP 서버, 사용 통계 추적
- **인증 정보**: OAuth 계정, 조직 정보
- **사용 이력**: 각 프로젝트별 API 사용량, 비용, 토큰 소비량

### 🗂️ 파일 특성
- JSON 형식의 텍스트 파일
- Claude Code가 자동으로 읽고 쓰는 파일
- 수동 편집 가능하지만 잘못된 형식은 Claude Code 오류를 발생시킬 수 있음
- 백업 권장 (중요한 설정 변경 전)

---

## 🛠️ 중요 설정 항목

### 🛠️ 1. 기본 설정
- **`autoUpdates`**: 자동 업데이트 활성화 여부
- **`autoConnectIde`**: IDE 자동 연결 활성화
- **`installMethod`**: 설치 방법 (native/electron)

### 🛠️ 2. 프로젝트별 설정 (`projects`)
각 프로젝트 경로마다 다음 정보를 관리:
- **`hasTrustDialogAccepted`**: 프로젝트 신뢰 여부 (중요 보안 설정)
- **`allowedTools`**: 허용된 도구 목록
- **`mcpServers`**: MCP(Model Context Protocol) 서버 설정
- **사용 통계**: 비용, API 호출 시간, 토큰 사용량

### ✨ 3. 기능 플래그 (`cachedGrowthBookFeatures`)
Claude Code의 실험적 기능들을 제어:
- **`tengu_prompt_suggestion`**: 프롬프트 제안 기능
- **`tengu_code_diff_cli`**: 코드 diff CLI 기능
- **`tengu_pr_status_cli`**: PR 상태 표시
- **`tengu_system_prompt_global_cache`**: 시스템 프롬프트 전역 캐시

### 🔹 4. 계정 정보 (`oauthAccount`)
- 조직 UUID, 이메일, 역할 정보
- 조직 권한 및 워크스페이스 역할

---

## 🛠️ 추천하는 설정

### 🔹 개발 생산성 향상

1. **IDE 자동 연결 활성화**
   ```json
   "autoConnectIde": true
   ```
   - IDE와 자동으로 연결하여 원활한 작업 환경 구성

2. **프롬프트 제안 기능 활성화**
   ```json
   "tengu_prompt_suggestion": true
   ```
   - AI가 상황에 맞는 프롬프트를 제안하여 작업 효율 향상

3. **코드 diff CLI 활성화**
   ```json
   "tengu_code_diff_cli": true
   ```
   - CLI에서 코드 변경사항을 시각적으로 확인 가능

### ⚡ 성능 최적화

4. **캐시 기능 활성화**
   ```json
   "tengu_cache_plum_violet": true,
   "tengu_compact_cache_prefix": true,
   "tengu_system_prompt_global_cache": true
   ```
   - 시스템 프롬프트 캐싱으로 응답 속도 향상 및 비용 절감

5. **Bash Haiku 프리페치 활성화**
   ```json
   "tengu_bash_haiku_prefetch": true
   ```
   - Bash 명령어 실행 시 빠른 Haiku 모델 사용으로 응답 속도 개선

### 🔐 보안 및 관리

6. **프로젝트별 신뢰 설정 관리**
   - 각 프로젝트마다 `hasTrustDialogAccepted`를 신중히 설정
   - 신뢰하지 않는 프로젝트는 `false`로 유지

7. **자동 업데이트 설정**
   ```json
   "autoUpdates": false,
   "autoUpdatesProtectedForNative": true
   ```
   - 안정성이 중요한 환경에서는 수동 업데이트 권장

---

## 📌 라인별 상세 설명

```json
{
  // === 기본 설정 ===
  "numStartups": 100,
  // Claude Code 실행 횟수 카운터

  "installMethod": "native",
  // 설치 방법: "native" (네이티브 앱) 또는 "electron" (일렉트론 앱)

  "autoUpdates": false,
  // 자동 업데이트 활성화 여부 (false = 수동 업데이트만)

  // === 팁 표시 이력 ===
  "tipsHistory": {
    // 각 팁이 표시된 횟수를 추적 (100 = 더 이상 표시 안 함)
    "new-user-warmup": 7,                    // 신규 사용자 환영 팁
    "plan-mode-for-complex-tasks": 100,      // 복잡한 작업 시 플랜 모드 사용 팁
    "terminal-setup": 94,                    // 터미널 설정 팁
    "memory-command": 87,                    // 메모리 명령어 사용 팁
    "theme-command": 99,                     // 테마 변경 명령어 팁
    "status-line": 80,                       // 상태 표시줄 사용 팁
    "stickers-command": 2,                   // 스티커 명령어 팁
    "prompt-queue": 94,                      // 프롬프트 큐 기능 팁
    "enter-to-steer-in-relatime": 100,       // 실시간 제어를 위한 Enter 키 팁
    "todo-list": 100,                        // 할 일 목록 기능 팁
    "ide-upsell-external-terminal": 2,       // 외부 터미널에서 IDE 사용 권장
    "install-github-app": 94,                // GitHub 앱 설치 안내
    "install-slack-app": 94,                 // Slack 앱 설치 안내
    "drag-and-drop-images": 94,              // 이미지 드래그 앤 드롭 기능
    "double-esc-code-restore": 94,           // ESC 두 번으로 코드 복원
    "continue": 94,                          // Continue 기능 사용 팁
    "default-permission-mode-config": 2,     // 기본 권한 모드 설정
    "shift-tab": 94,                         // Shift+Tab 단축키 팁
    "image-paste": 100,                      // 이미지 붙여넣기 기능
    "desktop-app": 87,                       // 데스크톱 앱 사용 권장
    "web-app": 87,                           // 웹 앱 사용 안내
    "mobile-app": 87,                        // 모바일 앱 사용 안내
    "guest-passes": 98,                      // 게스트 패스 기능
    "custom-agents": 87,                     // 커스텀 에이전트 사용법
    "permissions": 94,                       // 권한 관리 팁
    "rename-conversation": 94,               // 대화 이름 변경 기능
    "custom-commands": 94,                   // 커스텀 명령어 설정
    "frontend-design-plugin": 11,            // 프론트엔드 디자인 플러그인
    "agent-flag": 87,                        // 에이전트 플래그 사용
    "git-worktrees": 94                      // Git Worktree 기능
  },

  "promptQueueUseCount": 2,
  // 프롬프트 큐 기능 사용 횟수

  "autoConnectIde": true,
  // IDE 자동 연결 활성화 (VS Code, IntelliJ 등과 자동 연동)

  // === Statsig 게이트 (A/B 테스트 플래그) ===
  "cachedStatsigGates": {
    "tengu_prompt_suggestion": true
    // 프롬프트 제안 기능 활성화
  },

  // === GrowthBook 기능 플래그 (실험적 기능 제어) ===
  "cachedGrowthBookFeatures": {
    "tengu_1p_event_batch_config": {
      // 이벤트 배치 처리 설정
      "scheduledDelayMillis": 5000,          // 배치 처리 지연 시간 (5초)
      "maxExportBatchSize": 200,             // 최대 배치 크기
      "maxQueueSize": 8192                   // 최대 큐 크기
    },
    "tengu_mcp_tool_search": true,           // MCP 도구 검색 기능
    "tengu_scratch": false,                  // 스크래치 패드 기능 (비활성화)
    "tengu_log_segment_events": false,       // Segment 이벤트 로깅
    "tengu_log_datadog_events": true,        // Datadog 이벤트 로깅
    "tengu_pid_based_version_locking": true, // PID 기반 버전 잠금
    "tengu_event_sampling_config": {},       // 이벤트 샘플링 설정 (비어있음)
    "tengu_tool_pear": false,                // Pear 도구 (비활성화)
    "tengu_thinkback": false,                // Thinkback 기능 (비활성화)
    "tengu_sumi": true,                      // Sumi 기능
    "tengu_disable_bypass_permissions_mode": false, // 권한 우회 모드 비활성화 옵션
    "tengu_c4w_usage_limit_notifications_enabled": true, // 사용량 제한 알림
    "tengu-top-of-feed-tip": {
      // 피드 상단 팁 설정
      "tip": "",                             // 팁 내용 (비어있음)
      "color": ""                            // 팁 색상 (비어있음)
    },
    "tengu_react_vulnerability_warning": false, // React 취약점 경고
    "tengu_code_diff_cli": true,             // CLI 코드 diff 기능
    "tengu_post_compact_survey": false,      // 압축 후 설문조사
    "enhanced_telemetry_beta": false,        // 향상된 원격 측정 베타
    "tengu_tool_search_unsupported_models": [
      "haiku"                                // 도구 검색 미지원 모델 목록
    ],
    "tengu_ant_attribution_header_new": true, // 새 ANT 속성 헤더
    "tengu_streaming_tool_execution2": true,  // 스트리밍 도구 실행 v2
    "tengu_session_memory": false,           // 세션 메모리 기능 (비활성화)
    "tengu_prompt_suggestion": true,         // 프롬프트 제안 기능
    "tengu_keybinding_customization": false, // 키바인딩 커스터마이징 (비활성화)
    "tengu_claudeai_mcp_connectors": false,  // Claude.ai MCP 커넥터
    "tengu_plank_river_frost": "user_intent", // Plank River Frost 모드
    "tengu_bash_haiku_prefetch": true,       // Bash 명령 시 Haiku 모델 프리페치
    "tengu_permission_explainer": true,      // 권한 설명 기능
    "tengu_sm_compact": false,               // SM 압축 모드
    "tengu_compact_streaming_retry": false,  // 압축 스트리밍 재시도
    "tengu_attribution_header": true,        // 속성 헤더
    "tengu_brass_pebble": false,             // Brass Pebble 기능
    "tengu_cache_plum_violet": true,         // 캐시 Plum Violet 기능
    "tengu_scarf_coffee": false,             // Scarf Coffee 기능
    "tengu_keybinding_customization_release": true, // 키바인딩 커스터마이징 릴리스
    "tengu_pr_status_cli": true,             // CLI PR 상태 표시
    "tengu_marble_kite": false,              // Marble Kite 기능
    "tengu_kv7_prompt_sort": false,          // KV7 프롬프트 정렬
    "tengu_plum_vx3": false,                 // Plum VX3 기능
    "tengu_marble_anvil": true,              // Marble Anvil 기능
    "tengu_compact_cache_prefix": true,      // 압축 캐시 prefix
    "tengu_coral_fern": false,               // Coral Fern 기능
    "tengu_cork_m4q": false,                 // Cork M4Q 기능
    "tengu_system_prompt_global_cache": true, // 시스템 프롬프트 전역 캐시
    "tengu_workout": false,                  // Workout 기능
    "tengu_tst_kx7": false,                  // TST KX7 테스트
    "tengu_oboe": true,                      // Oboe 기능
    "tengu_tst_names_in_messages": false,    // 메시지 내 이름 테스트
    "tengu_vinteuil_phrase": false           // Vinteuil Phrase 기능
  },

  // === 사용자 식별 정보 ===
  "userID": "15a9fc336b1e7661cc4a1c52708382a72d82ac6308aff41c8d8935d198c8a26c",
  // 익명화된 사용자 고유 ID (해시값)

  "firstStartTime": "2026-01-19T09:21:44.387Z",
  // Claude Code 최초 실행 시간 (ISO 8601 형식)

  // === 마이그레이션 상태 ===
  "sonnet45MigrationComplete": true,
  // Sonnet 4.5 모델 마이그레이션 완료 여부

  "opus45MigrationComplete": true,
  // Opus 4.5 모델 마이그레이션 완료 여부

  "opusProMigrationComplete": true,
  // Opus Pro 마이그레이션 완료 여부

  "thinkingMigrationComplete": true,
  // Thinking 모드 마이그레이션 완료 여부

  // === 외부 통합 ===
  "cachedChromeExtensionInstalled": false,
  // Chrome 확장 프로그램 설치 여부 (캐시된 값)

  "changelogLastFetched": 1770183411927,
  // 변경 로그 마지막 조회 시간 (Unix timestamp, 밀리초)

  "autoUpdatesProtectedForNative": true,
  // 네이티브 앱에서 자동 업데이트 보호 활성화

  "claudeCodeFirstTokenDate": "2025-08-06T05:37:42.611986Z",
  // Claude Code에서 첫 토큰 생성 날짜

  // === 온보딩 상태 ===
  "hasCompletedOnboarding": true,
  // 온보딩 완료 여부

  "lastOnboardingVersion": "2.1.12",
  // 마지막으로 본 온보딩 버전

  // === 프로젝트별 설정 ===
  "projects": {
    // 프로젝트 경로를 키로 사용한 설정 객체

    "C:/Users/hhd20/project/hhddev": {
      // hhddev 프로젝트 설정
      "allowedTools": [],
      // 허용된 도구 목록 (비어있음 = 모든 도구 허용 또는 기본값)

      "mcpContextUris": [],
      // MCP 컨텍스트 URI 목록

      "mcpServers": {},
      // MCP 서버 설정 (비어있음)

      "enabledMcpjsonServers": [],
      // 활성화된 MCP JSON 서버 목록

      "disabledMcpjsonServers": [],
      // 비활성화된 MCP JSON 서버 목록

      "hasTrustDialogAccepted": true,
      // 프로젝트 신뢰 대화상자 수락 여부 (중요: 보안 설정)

      "projectOnboardingSeenCount": 52,
      // 프로젝트 온보딩 표시 횟수

      "hasClaudeMdExternalIncludesApproved": false,
      // CLAUDE.md 외부 포함 승인 여부

      "hasClaudeMdExternalIncludesWarningShown": false,
      // CLAUDE.md 외부 포함 경고 표시 여부

      "reactVulnerabilityCache": {
        // React 취약점 캐시 정보
        "detected": false,                   // 취약점 감지 여부
        "package": null,                     // 취약한 패키지 (없음)
        "packageName": null,                 // 패키지 이름 (없음)
        "version": null,                     // 패키지 버전 (없음)
        "packageManager": null               // 패키지 매니저 (없음)
      },

      // === 마지막 세션 사용 통계 ===
      "lastCost": 0.26555609999999996,
      // 마지막 세션 총 비용 (USD)

      "lastAPIDuration": 139507,
      // 마지막 API 호출 총 시간 (밀리초)

      "lastAPIDurationWithoutRetries": 139455,
      // 재시도 제외 API 호출 시간 (밀리초)

      "lastToolDuration": 10,
      // 마지막 도구 실행 시간 (밀리초)

      "lastDuration": 10277120,
      // 마지막 전체 작업 시간 (밀리초, 약 2.8시간)

      "lastLinesAdded": 19,
      // 마지막 세션에서 추가된 코드 라인 수

      "lastLinesRemoved": 12,
      // 마지막 세션에서 삭제된 코드 라인 수

      "lastTotalInputTokens": 9672,
      // 마지막 세션 총 입력 토큰 수

      "lastTotalOutputTokens": 7229,
      // 마지막 세션 총 출력 토큰 수

      "lastTotalCacheCreationInputTokens": 25012,
      // 마지막 세션 캐시 생성 입력 토큰 수

      "lastTotalCacheReadInputTokens": 222832,
      // 마지막 세션 캐시 읽기 입력 토큰 수 (캐시 히트)

      "lastTotalWebSearchRequests": 0,
      // 마지막 세션 웹 검색 요청 수

      "lastModelUsage": {
        // 모델별 사용 통계
        "claude-haiku-4-5-20251001": {
          // Haiku 4.5 모델 사용 통계
          "inputTokens": 9586,               // 입력 토큰
          "outputTokens": 183,               // 출력 토큰
          "cacheReadInputTokens": 0,         // 캐시 읽기 입력 토큰
          "cacheCreationInputTokens": 4615,  // 캐시 생성 입력 토큰
          "webSearchRequests": 0,            // 웹 검색 요청
          "costUSD": 0.01626975              // USD 비용
        },
        "claude-sonnet-4-5-20250929": {
          // Sonnet 4.5 모델 사용 통계
          "inputTokens": 86,
          "outputTokens": 7046,
          "cacheReadInputTokens": 222832,
          "cacheCreationInputTokens": 20397,
          "webSearchRequests": 0,
          "costUSD": 0.24928635000000002
        }
      },

      "lastSessionId": "75bbbe26-e4e5-40e6-9103-f3ad58ae9e3d",
      // 마지막 세션 고유 ID

      "lastFpsAverage": 0.38,
      // 마지막 평균 FPS (UI 렌더링 성능)

      "lastFpsLow1Pct": 398.26
      // 최하위 1% FPS (성능 병목 지표)
    },

    "C:/Users/hhd20": {
      // 홈 디렉토리 프로젝트 설정 (최소 설정)
      "allowedTools": [],
      "mcpContextUris": [],
      "mcpServers": {},
      "enabledMcpjsonServers": [],
      "disabledMcpjsonServers": [],
      "hasTrustDialogAccepted": false,       // 신뢰하지 않음 (보안상 권장)
      "projectOnboardingSeenCount": 0,
      "hasClaudeMdExternalIncludesApproved": false,
      "hasClaudeMdExternalIncludesWarningShown": false
    },

    "C:/Users/hhd20/project/mepia": {
      // mepia 프로젝트 설정
      "allowedTools": [],
      "mcpContextUris": [],
      "mcpServers": {},
      "enabledMcpjsonServers": [],
      "disabledMcpjsonServers": [],
      "hasTrustDialogAccepted": true,        // 신뢰된 프로젝트
      "projectOnboardingSeenCount": 10,
      "hasClaudeMdExternalIncludesApproved": false,
      "hasClaudeMdExternalIncludesWarningShown": false,
      "reactVulnerabilityCache": {
        "detected": false,
        "package": null,
        "packageName": null,
        "version": null,
        "packageManager": null
      },

      // === mepia 프로젝트 사용 통계 ===
      "lastCost": 0.9671396999999999,
      // 상당히 높은 비용 (약 $0.97)

      "lastAPIDuration": 331004,
      // API 호출 시간 (약 5.5분)

      "lastAPIDurationWithoutRetries": 330987,
      "lastToolDuration": 23682,
      // 도구 실행 시간 (약 23초)

      "lastDuration": 16870236,
      // 전체 작업 시간 (약 4.7시간)

      "lastLinesAdded": 1023,
      // 많은 코드 추가 (1000+ 라인)

      "lastLinesRemoved": 0,
      // 코드 삭제 없음 (순수 추가 작업)

      "lastTotalInputTokens": 3213,
      "lastTotalOutputTokens": 18703,
      // 출력 토큰이 매우 많음 (대량 코드 생성)

      "lastTotalCacheCreationInputTokens": 153654,
      "lastTotalCacheReadInputTokens": 327564,
      "lastTotalWebSearchRequests": 1,
      // 웹 검색 1회 사용

      "lastModelUsage": {
        "claude-haiku-4-5-20251001": {
          "inputTokens": 3113,
          "outputTokens": 129,
          "cacheReadInputTokens": 0,
          "cacheCreationInputTokens": 0,
          "webSearchRequests": 0,
          "costUSD": 0.0037579999999999996    // Haiku 비용 (저렴)
        },
        "claude-sonnet-4-5-20250929": {
          "inputTokens": 100,
          "outputTokens": 18574,               // 대부분의 출력이 Sonnet
          "cacheReadInputTokens": 327564,      // 대량 캐시 사용 (비용 절감)
          "cacheCreationInputTokens": 153654,
          "webSearchRequests": 1,
          "costUSD": 0.9633817                 // Sonnet 비용 (대부분)
        }
      },

      "lastSessionId": "4c40f0c4-9936-4775-9c05-81384835f186",
      "lastFpsAverage": 2.27,
      "lastFpsLow1Pct": 173.87
    }
  },

  // === 릴리스 노트 및 공지사항 ===
  "lastReleaseNotesSeen": "2.1.31",
  // 마지막으로 본 릴리스 노트 버전

  "hasShownOpus45Notice": {
    // Opus 4.5 공지 표시 여부 (계정별)
    "b9c5c68f-5a47-4ecc-9fe1-6b37ab0cd87a": true,
    "879b4371-b635-47d3-a9d3-53c7057e9ad5": true
  },

  // === 접근 권한 캐시 ===
  "s1mAccessCache": {
    // S1M (Sonnet 1 Million?) 접근 캐시
    "b9c5c68f-5a47-4ecc-9fe1-6b37ab0cd87a": {
      "hasAccess": false,                    // 접근 권한 없음
      "hasAccessNotAsDefault": false,        // 기본이 아닌 접근 권한 없음
      "timestamp": 1768868876436             // 캐시 시간
    },
    "879b4371-b635-47d3-a9d3-53c7057e9ad5": {
      "hasAccess": false,
      "hasAccessNotAsDefault": false,
      "timestamp": 1770183254372
    }
  },

  "groveConfigCache": {
    // Grove 설정 캐시 (내부 기능)
    "0b7a20a1-1c03-481c-a02d-2fde42c984cc": {
      "grove_enabled": true,
      "timestamp": 1768817516047
    },
    "8f47a74a-c435-4682-b351-abb6d3280c87": {
      "grove_enabled": true,
      "timestamp": 1770115780450
    }
  },

  // === 마켓플레이스 ===
  "officialMarketplaceAutoInstallAttempted": true,
  // 공식 마켓플레이스 자동 설치 시도 여부

  "officialMarketplaceAutoInstalled": true,
  // 공식 마켓플레이스 자동 설치 완료 여부

  // === 분석 및 추적 ===
  "anonymousId": "claudecode.v1.3f0b124a-5360-4f0e-89a5-f665b0d9b177",
  // 익명 분석 ID (원격 측정용)

  "lastPlanModeUse": 1769045765469,
  // 플랜 모드 마지막 사용 시간 (Unix timestamp, 밀리초)

  // === 게스트 패스 (초대 기능) ===
  "passesEligibilityCache": {
    // 게스트 패스 자격 캐시
    "879b4371-b635-47d3-a9d3-53c7057e9ad5": {
      "eligible": true,                      // 자격 있음
      "referral_code_details": {
        "code": "1cGkYTLewg",                // 추천 코드
        "campaign": "claude_code_guest_pass", // 캠페인 이름
        "referral_link": "https://claude.ai/referral/1cGkYTLewg" // 추천 링크
      },
      "referrer_reward": null,               // 추천인 보상 (없음)
      "remaining_passes": 3,                 // 남은 패스 수
      "timestamp": 1770182846035
    }
  },

  // === GitHub 연동 ===
  "githubRepoPaths": {
    // GitHub 저장소 경로 매핑
    "bimsontop/mepia": [
      "C:\\Users\\hhd20\\project\\mepia"     // 로컬 경로
    ]
  },

  "passesUpsellSeenCount": 4,
  // 게스트 패스 업셀 표시 횟수

  // === 피드백 ===
  "feedbackSurveyState": {
    "lastShownTime": 1769704340569           // 마지막 피드백 설문 표시 시간
  },

  // === OAuth 계정 정보 ===
  "oauthAccount": {
    "accountUuid": "8f47a74a-c435-4682-b351-abb6d3280c87",
    // 계정 고유 ID

    "emailAddress": "dsone5@bimsontop.com",
    // 계정 이메일

    "organizationUuid": "879b4371-b635-47d3-a9d3-53c7057e9ad5",
    // 조직 고유 ID

    "hasExtraUsageEnabled": false,
    // 추가 사용량 활성화 여부

    "displayName": "Aiden",
    // 표시 이름

    "organizationRole": "admin",
    // 조직 내 역할 (관리자)

    "workspaceRole": null,
    // 워크스페이스 역할 (설정 안 됨)

    "organizationName": "dsone5@bimsontop.com's Organization"
    // 조직 이름
  },

  // === 게스트 패스 UI 상태 ===
  "hasVisitedPasses": false,
  // 게스트 패스 페이지 방문 여부

  "passesLastSeenRemaining": 3,
  // 마지막으로 본 남은 패스 수

  // === 클라이언트 데이터 캐시 ===
  "clientDataCache": {
    "data": {},                              // 클라이언트 데이터 (비어있음)
    "timestamp": 1770183464030               // 캐시 시간
  }
}
```

---

## 🔗 추가 참고사항

### 🗂️ 파일 백업
`.claude.json` 파일을 수정하기 전에는 반드시 백업을 만들어두세요.
```bash
cp ~/.claude.json ~/.claude.json.backup
```

### 🔹 문제 해결
파일이 손상되었거나 Claude Code가 제대로 작동하지 않을 경우:
1. Claude Code 종료
2. `.claude.json` 파일 삭제 또는 백업으로 복원
3. Claude Code 재시작 (새 설정 파일 자동 생성)

### ⚠️ 보안 주의사항
- `oauthAccount` 섹션에는 민감한 정보가 포함되어 있으므로 공개하지 마세요
- `userID`, `anonymousId` 등은 개인 식별이 가능한 정보입니다
- Git 저장소에 이 파일을 커밋하지 마세요

### ⚡ 성능 모니터링
프로젝트별 `lastModelUsage` 섹션을 확인하여:
- 비용 추적: `costUSD` 값 확인
- 캐시 효율성: `cacheReadInputTokens` vs `inputTokens` 비율 확인
- 모델 사용 패턴: Haiku vs Sonnet 사용 비율 분석

---

## 📌 변경 이력
- 2026-02-04: 초기 문서 작성
