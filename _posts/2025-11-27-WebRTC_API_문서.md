---
layout: post
title: "251127 WebRTC API 문서"
comments: true
tags: [WebRTC, JavaScript, API, RealTime, Communication, P2P, Browser, WebDevelopment, Streaming, Video, Audio, DataChannel, ICE, STUN, TURN, MediaStream, RTCPeerConnection, Signaling, DTLS, SRTP, Network]
---

# WebRTC API 문서

## 📖 개요

**WebRTC (Web Real-Time Communication)**는 웹 애플리케이션이 중간 서버 없이 브라우저 간 직접적으로 오디오, 비디오, 데이터를 실시간 교환할 수 있게 하는 기술입니다.

### 핵심 특징
- 플러그인 없이 웹에서 실시간 통신 구현
- P2P 직접 연결로 낮은 지연시간
- 암호화된 보안 통신
- 크로스 브라우저 호환성

## 🏗️ 핵심 구성 요소

### 1. 주요 API 인터페이스

#### `RTCPeerConnection`
```javascript
// 피어 간 WebRTC 연결의 핵심 인터페이스
const peerConnection = new RTCPeerConnection({
  iceServers: [
    { urls: "stun:stun.l.google.com:19302" }
  ]
});
```

**역할:**
- 피어 간 연결 설정 및 관리
- ICE 후보 수집 및 교환
- 미디어 스트림 추가/제거
- 데이터 채널 생성

#### `MediaStream`
```javascript
// 오디오/비디오 트랙 처리
navigator.mediaDevices.getUserMedia({
  video: true,
  audio: true
}).then(stream => {
  // 로컬 스트림 처리
  localVideo.srcObject = stream;
  peerConnection.addStream(stream);
});
```

**역할:**
- 카메라/마이크 접근
- 스트림 관리
- 트랙 조작

#### `RTCDataChannel`
```javascript
// 임의의 데이터 양방향 통신
const dataChannel = peerConnection.createDataChannel("messages");
dataChannel.onopen = () => {
  dataChannel.send("Hello WebRTC!");
};
```

**역할:**
- 텍스트/바이너리 데이터 전송
- 파일 공유
- 게임 상태 동기화

### 2. 연결 설정 프로세스

#### Signaling 과정
```javascript
// 1. Offer 생성 (발신자)
const offer = await peerConnection.createOffer();
await peerConnection.setLocalDescription(offer);
// 시그널링 서버를 통해 상대방에게 전송

// 2. Answer 생성 (수신자)
await peerConnection.setRemoteDescription(offer);
const answer = await peerConnection.createAnswer();
await peerConnection.setLocalDescription(answer);
// 시그널링 서버를 통해 발신자에게 전송

// 3. ICE 후보 교환
peerConnection.onicecandidate = (event) => {
  if (event.candidate) {
    // 시그널링 서버를 통해 상대방에게 전송
    sendIceCandidate(event.candidate);
  }
};
```

#### ICE (Interactive Connectivity Establishment)
- **STUN 서버**: NAT 뒤의 공인 IP 발견
- **TURN 서버**: 직접 연결 불가능 시 릴레이
- **ICE 후보**: 가능한 네트워크 경로들

## 🔒 보안 및 프로토콜

### 암호화 기술
- **DTLS (Datagram Transport Layer Security)**: 데이터 암호화
- **SRTP (Secure RTP)**: 미디어 스트림 암호화
- **강제 암호화**: 모든 WebRTC 통신은 암호화 필수

### 사용 프로토콜
- **RTP (Real-time Transport Protocol)**: 실시간 미디어 전송
- **RTCP (RTP Control Protocol)**: 품질 제어
- **SCTP (Stream Control Transmission Protocol)**: 데이터 채널

## 🎯 실제 구현 예시

### 기본 비디오 채팅 구현
```javascript
class WebRTCVideoChat {
  constructor() {
    this.localConnection = new RTCPeerConnection({
      iceServers: [{ urls: "stun:stun.l.google.com:19302" }]
    });
    this.remoteConnection = new RTCPeerConnection({
      iceServers: [{ urls: "stun:stun.l.google.com:19302" }]
    });
    
    this.setupEventListeners();
  }
  
  async startCall() {
    // 로컬 미디어 스트림 획득
    const localStream = await navigator.mediaDevices.getUserMedia({
      video: true,
      audio: true
    });
    
    // 로컬 비디오에 표시
    document.getElementById("localVideo").srcObject = localStream;
    
    // 피어 연결에 스트림 추가
    localStream.getTracks().forEach(track => {
      this.localConnection.addTrack(track, localStream);
    });
    
    // Offer 생성 및 전송
    const offer = await this.localConnection.createOffer();
    await this.localConnection.setLocalDescription(offer);
    
    // 실제로는 시그널링 서버를 통해 전송
    await this.remoteConnection.setRemoteDescription(offer);
    
    // Answer 생성 및 전송
    const answer = await this.remoteConnection.createAnswer();
    await this.remoteConnection.setLocalDescription(answer);
    await this.localConnection.setRemoteDescription(answer);
  }
  
  setupEventListeners() {
    // 원격 스트림 수신 시
    this.remoteConnection.ontrack = (event) => {
      document.getElementById("remoteVideo").srcObject = event.streams[0];
    };
    
    // ICE 후보 교환
    this.localConnection.onicecandidate = (event) => {
      if (event.candidate) {
        this.remoteConnection.addIceCandidate(event.candidate);
      }
    };
    
    this.remoteConnection.onicecandidate = (event) => {
      if (event.candidate) {
        this.localConnection.addIceCandidate(event.candidate);
      }
    };
  }
}
```

## 🌐 브라우저 호환성

### 지원 현황 (2025년 기준)
- ✅ **Chrome**: 완전 지원
- ✅ **Firefox**: 완전 지원  
- ✅ **Safari**: iOS 11+ 지원
- ✅ **Edge**: Chromium 기반으로 완전 지원
- ⚠️ **Internet Explorer**: 지원 안함

### 호환성 라이브러리
```javascript
// adapter.js 사용 권장
import adapter from 'webrtc-adapter';

// 크로스 브라우저 호환성 자동 처리
const peerConnection = new RTCPeerConnection();
```

## 🎮 주요 사용 사례

### 1. 화상 회의 솔루션
- **Zoom, Google Meet, Microsoft Teams**
- 다중 참가자 지원
- 화면 공유 기능

### 2. 실시간 스트리밍
- **YouTube Live, Twitch**
- 낮은 지연시간
- 상호작용 기능

### 3. 게임 및 협업 도구
- **실시간 멀티플레이어 게임**
- **온라인 화이트보드**
- **공동 문서 편집**

### 4. IoT 및 원격 제어
- **IP 카메라 스트리밍**
- **원격 데스크톱**
- **스마트홈 제어**

## ⚡ 성능 최적화 팁

### 1. 코덱 선택
```javascript
// 비디오 코덱 우선순위 설정
const transceiver = peerConnection.addTransceiver("video", {
  direction: "sendrecv"
});

const capabilities = RTCRtpSender.getCapabilities("video");
// VP9, H.264 등 적절한 코덱 선택
```

### 2. 대역폭 제어
```javascript
// 비트레이트 제한
const sender = peerConnection.getSenders()[0];
const params = sender.getParameters();
params.encodings[0].maxBitrate = 1000000; // 1Mbps
sender.setParameters(params);
```

### 3. 네트워크 적응
```javascript
// 네트워크 상태 모니터링
peerConnection.getStats().then(stats => {
  stats.forEach(report => {
    if (report.type === "inbound-rtp" && report.kind === "video") {
      console.log("패킷 손실률:", report.packetsLost);
      console.log("지터:", report.jitter);
    }
  });
});
```

## 🚨 개발 시 주의사항

### 1. HTTPS 필수
```javascript
// getUserMedia는 HTTPS에서만 동작
if (location.protocol !== "https:") {
  console.error("WebRTC requires HTTPS");
}
```

### 2. 리소스 정리
```javascript
// 연결 종료 시 리소스 정리
function closeConnection() {
  localStream.getTracks().forEach(track => track.stop());
  peerConnection.close();
  peerConnection = null;
}
```

### 3. 에러 처리
```javascript
peerConnection.onconnectionstatechange = () => {
  if (peerConnection.connectionState === "failed") {
    console.error("Connection failed, attempting restart");
    // 재연결 로직
  }
};
```

## 📚 추가 학습 자료

### 공식 문서
- [MDN WebRTC API](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API)
- [W3C WebRTC 명세서](https://www.w3.org/TR/webrtc/)

### 실습 도구
- [webrtc.org](https://webrtc.org) - 공식 사이트
- [adapter.js](https://github.com/webrtcHacks/adapter) - 호환성 라이브러리
- [SimpleWebRTC](https://simplewebrtc.com) - 간단한 구현체

WebRTC는 현대 웹에서 실시간 통신의 표준이 되었으며, 올바른 이해와 구현으로 강력한 실시간 애플리케이션을 만들 수 있습니다! 🚀