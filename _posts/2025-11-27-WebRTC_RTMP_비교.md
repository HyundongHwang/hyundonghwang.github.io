---
layout: post
title: "251127 WebRTC RTMP 비교"
comments: true
tags: []
---

# WebRTC RTMP 비교

## 개요

실시간 통신과 스트리밍 분야에서 WebRTC와 RTMP는 각각 다른 목적과 특성을 가진 핵심 기술입니다. 이 글에서는 두 기술의 원리, 프로세스, 장단점, 그리고 실제 사용사례를 상세히 비교 분석하여 프로젝트에 적합한 기술 선택을 도와드리겠습니다.

## WebRTC (Web Real-Time Communication)

### 소개
WebRTC는 웹 브라우저와 모바일 애플리케이션에서 실시간 오디오, 비디오, 데이터 통신을 가능하게 하는 오픈소스 표준입니다. Google, Mozilla, Opera 등이 공동으로 개발했습니다.

### 원리
- **P2P 통신**: 클라이언트 간 직접 연결
- **브라우저 네이티브**: 플러그인 없이 브라우저에서 직접 동작
- **실시간 전송**: 초저지연 (20-150ms)
- **미디어 스택**: 오디오/비디오 코덱 내장

### WebRTC 프로세스

#### 1. 연결 설정 과정 (Signaling)
```
Peer A                    Signaling Server                    Peer B
  |                             |                             |
  |--- Offer (SDP) ------------->|                             |
  |                             |-------- Offer ------------->|
  |                             |<------- Answer -------------|
  |<-- Answer (SDP) -------------|                             |
  |                             |                             |
  |--- ICE Candidates --------->|                             |
  |                             |---- ICE Candidates ------->|
  |<-- ICE Candidates -----------|                             |
  |                             |<--- ICE Candidates ---------|
  |                             |                             |
  |============= P2P Connection Established =================|
```

#### 2. 미디어 처리 파이프라인
```
MediaDevices.getUserMedia()
    ↓
MediaStream
    ↓
RTCPeerConnection.addStream()
    ↓
RTP Packets
    ↓
Network (UDP/SRTP)
    ↓
Remote RTCPeerConnection
    ↓
Remote MediaStream
    ↓
<video>/<audio> element
```

#### 3. NAT 트래버설 (STUN/TURN)
```
Client A ←→ NAT A ←→ Internet ←→ STUN Server
                          ↓
                      Public IP 확인
                          ↓
Client A ←――――――――――――――――――――→ Client B (Direct P2P)

NAT 통과 실패 시:
Client A ←→ TURN Server ←→ Client B (Relay)
```

### WebRTC 장점
```
✓ 초저지연 (20-150ms)
✓ P2P 직접 연결로 서버 부하 감소
✓ 브라우저 네이티브 지원 (플러그인 불필요)
✓ 보안성 우수 (DTLS/SRTP 암호화)
✓ 양방향 실시간 통신
✓ 적응적 비트레이트 (자동 품질 조정)
✓ 무료 오픈소스
```

### WebRTC 단점
```
✗ NAT/방화벽 문제 (STUN/TURN 서버 필요)
✗ 대규모 브로드캐스팅 어려움
✗ 브라우저 호환성 이슈 (Safari 제한)
✗ 복잡한 시그널링 구현 필요
✗ 모바일 배터리 소모량 높음
✗ 네트워크 불안정 시 품질 저하
```

## RTMP (Real-Time Messaging Protocol)

### 소개
RTMP는 Adobe Flash Player를 위해 개발된 실시간 스트리밍 프로토콜입니다. 현재는 라이브 스트리밍과 비디오 온디맨드 서비스에 널리 사용됩니다.

### 원리
- **TCP 기반**: 안정적인 스트림 전송
- **서버-클라이언트 모델**: 중앙 서버를 통한 스트리밍
- **청크 기반 전송**: 데이터를 작은 청크로 나누어 전송
- **Flash 기반**: 원래 Flash Player를 위해 설계

### RTMP 프로세스

#### 1. RTMP 연결 및 스트리밍 과정
```
Publisher (OBS/FFmpeg)    RTMP Server          Viewer (Browser/App)
        |                      |                       |
        |--- TCP Handshake --->|                       |
        |--- RTMP Connect ----->|                       |
        |<-- Connection ACK ----|                       |
        |--- Create Stream ---->|                       |
        |--- Publish Stream --->|                       |
        |--- Video/Audio ------>|                       |
        |       Data            |                       |
        |                       |<-- Request Stream ----|
        |                       |---- Video/Audio ----->|
        |                       |        Data           |
```

#### 2. RTMP 청크 구조
```
RTMP Message
    ↓
Chunk Header (1-18 bytes)
    ├── Basic Header (1-3 bytes)
    ├── Message Header (0-11 bytes) 
    └── Extended Timestamp (0-4 bytes)
    ↓
Chunk Data (≤ Chunk Size)
```

#### 3. 스트리밍 파이프라인
```
Video Source (Camera)
    ↓
Video Encoder (H.264)
    ↓
Audio Source (Microphone)
    ↓
Audio Encoder (AAC)
    ↓
RTMP Packager
    ↓
TCP Socket
    ↓
RTMP Server (Nginx-RTMP, SRS, Wowza)
    ↓
HTTP-FLV/HLS/DASH 변환
    ↓
CDN Distribution
    ↓
Player (VLC, JW Player, Video.js)
```

### RTMP 장점
```
✓ 안정적인 스트리밍 (TCP 기반)
✓ 대규모 시청자 지원 (1:N 브로드캐스팅)
✓ 성숙한 생태계 (다양한 서버/도구)
✓ CDN 연동 용이
✓ 녹화 및 저장 기능
✓ 안정적인 품질 유지
✓ 네트워크 적응성 우수
```

### RTMP 단점
```
✗ 높은 지연시간 (2-10초)
✗ Flash Player 의존성 (브라우저 지원 종료)
✗ 서버 인프라 비용
✗ 단방향 통신 (Publisher → Viewer)
✗ 실시간 상호작용 제한
✗ 모바일 브라우저 지원 부족
```

## 성능 비교표

| 항목 | WebRTC | RTMP |
|------|---------|-------|
| **지연시간** | 20-150ms | 2-10초 |
| **확장성** | 제한적 (P2P) | 우수 (서버 기반) |
| **브라우저 지원** | 네이티브 | Flash 필요 |
| **서버 부하** | 낮음 | 높음 |
| **구현 복잡도** | 높음 | 중간 |
| **비용** | TURN 서버 | 스트리밍 서버 |
| **보안** | 내장 암호화 | 추가 구현 필요 |

## 실제 사용사례

### WebRTC 최적 사용사례

#### 1. 화상회의 시스템
```
예시: Zoom, Google Meet, Discord
- 1:1 또는 소규모 그룹 통화
- 실시간 상호작용 필수
- 초저지연 요구사항
- 양방향 오디오/비디오
```

#### 2. 실시간 게임
```
예시: 브라우저 기반 멀티플레이어 게임
- P2P 데이터 채널 활용
- 실시간 동기화
- 낮은 네트워크 지연 중요
```

#### 3. 원격 데스크톱/제어
```
예시: TeamViewer, Chrome Remote Desktop
- 실시간 화면 공유
- 마우스/키보드 제어
- 즉시 반응성 필요
```

#### 4. 라이브 헬프데스크
```
예시: 고객 지원 화상 상담
- 1:1 실시간 상담
- 화면 공유 기능
- 브라우저 접근성
```

### RTMP 최적 사용사례

#### 1. 라이브 스트리밍 플랫폼
```
예시: Twitch, YouTube Live, 아프리카TV
- 1:多 브로드캐스팅
- 수천 명의 동시 시청자
- 안정적인 스트리밍 품질
- 녹화 및 다시보기 기능
```

#### 2. 교육 플랫폼
```
예시: 온라인 강의, 웨비나
- 강사 1명 → 수백명 학생
- 장시간 안정적 스트리밍
- 녹화된 강의 제공
- 채팅 상호작용
```

#### 3. 기업 라이브 이벤트
```
예시: 제품 발표회, 컨퍼런스
- 대규모 시청자
- 전문적인 방송 품질
- 다중 카메라 스위칭
- CDN을 통한 글로벌 배포
```

#### 4. 감시/보안 시스템
```
예시: CCTV 원격 모니터링
- 24/7 연속 스트리밍
- 안정성 최우선
- 녹화 및 저장
- 다중 채널 관리
```

## 하이브리드 아키텍처 사례

### YouTube Live의 경우
```
OBS/스트리머 → RTMP → YouTube 서버 → 
    ↓
HLS/DASH → CDN → 시청자 (지연: 10-30초)
    ↓
WebRTC → 실시간 채팅/슈퍼챗 (지연: <1초)
```

### Facebook Live의 경우
```
모바일 앱 → RTMP → Facebook 서버 → 
    ↓
HTTP Progressive → 시청자 (지연: 3-8초)
    ↓
WebRTC → 실시간 반응/댓글 (지연: <1초)
```

## 선택 가이드

### WebRTC를 선택해야 하는 경우
- 실시간 상호작용이 핵심 (화상회의, 게임)
- 지연시간이 1초 미만이어야 함
- P2P 통신으로 서버 비용 절약
- 소규모 참여자 (2-50명)
- 브라우저 기반 서비스

### RTMP를 선택해야 하는 경우  
- 대규모 시청자 대상 (100명+)
- 방송/스트리밍이 주 목적
- 안정적인 품질이 중요
- 녹화/저장 기능 필요
- 기존 스트리밍 인프라 활용

### 최신 트렌드: WebRTC + RTMP 조합
```
실시간 상호작용: WebRTC (채팅, 반응)
    +
메인 스트림: RTMP → HLS/DASH (영상)
```

## 기술적 구현 고려사항

### WebRTC 구현 시 주의점
- STUN/TURN 서버 설정 필요
- 시그널링 서버 구현 복잡
- 브라우저별 호환성 테스트 중요
- 네트워크 품질에 따른 적응형 비트레이트 구현

### RTMP 구현 시 주의점
- Flash 대체 기술 (HLS/DASH) 병행 필요
- CDN 연동 및 부하 분산 고려
- 다양한 해상도/비트레이트 제공
- 모바일 브라우저 지원을 위한 추가 프로토콜 필요

## 결론 및 권장사항

**WebRTC vs RTMP 핵심 차이점:**

- **목적**: WebRTC (실시간 통신) vs RTMP (스트리밍 방송)
- **지연시간**: WebRTC (초저지연) vs RTMP (버퍼링된 안정성)
- **확장성**: WebRTC (소규모 P2P) vs RTMP (대규모 1:N)
- **상호작용**: WebRTC (양방향) vs RTMP (단방향)

**미래 전망:**
- WebRTC: 실시간 협업, 메타버스, IoT 분야로 확장
- RTMP: HLS/DASH로 대체되는 추세, 하지만 스트리밍 인제스트에서는 여전히 표준
- 하이브리드 접근법이 주류로 자리잡을 전망

**최종 권장사항:**
프로젝트의 핵심 요구사항(지연시간 vs 확장성)을 명확히 하여 적절한 기술을 선택하는 것이 중요합니다. 많은 현대적인 플랫폼들이 두 기술의 장점을 결합한 하이브리드 접근법을 채택하고 있다는 점도 고려해보시기 바랍니다.