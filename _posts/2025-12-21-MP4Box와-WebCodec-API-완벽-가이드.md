---
layout: post
title: "251221 MP4Box와 WebCodec API 완벽 가이드"
comments: true
tags:
- mp4box
- webcodec-api
- video-processing
- javascript
- media-api
---

# MP4Box와 WebCodec API 완벽 가이드

웹 브라우저에서의 고성능 비디오 처리가 새로운 전환점을 맞고 있습니다. MP4Box.js와 WebCodec API의 결합은 네이티브 애플리케이션 수준의 비디오 조작 능력을 웹에서 구현할 수 있게 해줍니다.

## 기술 개요

### MP4Box.js란?

MP4Box.js는 GPAC 프로젝트의 JavaScript 포트로, MP4 컨테이너 형식을 완전히 파싱하고 조작할 수 있는 강력한 라이브러리입니다.

```
┌────────────── MP4Box.js 핵심 기능 ──────────────┐
│                                               │
│  ┌─────────────┐    ┌─────────────────────┐   │
│  │   MP4 File  │───►│   MP4Box Parser     │   │
│  │   Stream    │    │                     │   │
│  └─────────────┘    │ • Track Extraction  │   │
│                     │ • Metadata Reading  │   │
│  ┌─────────────┐    │ • Sample Access     │   │
│  │ Segmented   │◄───│ • Live Streaming    │   │
│  │   Output    │    │ • Format Conversion │   │
│  └─────────────┘    └─────────────────────┘   │
│                                               │
│  지원 기능:                                    │
│  • DASH/HLS 세그멘테이션                       │
│  • 실시간 스트림 처리                          │
│  • 메타데이터 편집                             │
│  • 다중 트랙 관리                              │
└───────────────────────────────────────────────┘
```

### WebCodec API 소개

WebCodec API는 브라우저에서 네이티브 수준의 비디오/오디오 인코딩과 디코딩을 가능하게 하는 새로운 웹 표준입니다.

```javascript
// WebCodec API 기본 구조
const decoder = new VideoDecoder({
  output: (frame) => {
    // 디코딩된 프레임 처리
    console.log('Frame decoded:', frame);
  },
  error: (error) => {
    console.error('Decode error:', error);
  }
});

// 인코더 설정
const encoder = new VideoEncoder({
  output: (chunk, metadata) => {
    // 인코딩된 청크 처리
    console.log('Chunk encoded:', chunk);
  },
  error: (error) => {
    console.error('Encode error:', error);
  }
});
```

## MP4Box.js 심화 활용

### 1. MP4 파일 파싱과 메타데이터 추출

```javascript
import MP4Box from 'mp4box';

class MP4Analyzer {
  constructor() {
    this.mp4boxfile = MP4Box.createFile();
    this.setupEventHandlers();
  }
  
  setupEventHandlers() {
    // 파일이 준비되면 호출
    this.mp4boxfile.onReady = (info) => {
      console.log('MP4 정보:', info);
      console.log('총 재생 시간:', info.duration / info.timescale, '초');
      console.log('트랙 수:', info.tracks.length);
      
      // 각 트랙 정보 출력
      info.tracks.forEach((track, index) => {
        console.log(`트랙 ${index + 1}:`, {
          id: track.id,
          type: track.type,
          codec: track.codec,
          width: track.video?.width,
          height: track.video?.height,
          sampleRate: track.audio?.sample_rate,
          channels: track.audio?.channel_count
        });
      });
    };
    
    // 에러 처리
    this.mp4boxfile.onError = (error) => {
      console.error('MP4 파싱 에러:', error);
    };
    
    // 세그먼트 추출 완료
    this.mp4boxfile.onSegment = (id, user, buffer) => {
      console.log('세그먼트 추출됨:', { id, buffer: buffer.byteLength });
    };
  }
  
  async loadFile(file) {
    const arrayBuffer = await file.arrayBuffer();
    arrayBuffer.fileStart = 0;
    
    // MP4Box에 데이터 공급
    this.mp4boxfile.appendBuffer(arrayBuffer);
  }
  
  extractVideoTrack() {
    const info = this.mp4boxfile.getInfo();
    const videoTrack = info.tracks.find(track => track.type === 'video');
    
    if (videoTrack) {
      // 비디오 트랙의 모든 샘플 추출
      this.mp4boxfile.setExtractionOptions(videoTrack.id, null, {
        nbSamples: 100 // 처음 100개 샘플만
      });
      
      this.mp4boxfile.start();
    }
  }
  
  createDASHSegments() {
    const info = this.mp4boxfile.getInfo();
    
    info.tracks.forEach(track => {
      // DASH 세그멘테이션 설정
      this.mp4boxfile.setSegmentOptions(track.id, null, {
        duration: 2000, // 2초 세그먼트
        rapAlignement: true
      });
    });
    
    this.mp4boxfile.start();
  }
}

// 사용 예시
const analyzer = new MP4Analyzer();

// 파일 선택 이벤트
document.getElementById('fileInput').addEventListener('change', async (event) => {
  const file = event.target.files[0];
  if (file) {
    await analyzer.loadFile(file);
    analyzer.extractVideoTrack();
  }
});
```

### 2. 실시간 스트림 처리

```javascript
class MP4StreamProcessor {
  constructor() {
    this.mp4boxfile = MP4Box.createFile();
    this.setupStreamHandling();
  }
  
  setupStreamHandling() {
    this.mp4boxfile.onReady = (info) => {
      console.log('스트림 준비 완료');
      
      // 각 트랙에 대해 실시간 추출 설정
      info.tracks.forEach(track => {
        this.mp4boxfile.setExtractionOptions(track.id, this, {
          nbSamples: 1000
        });
      });
      
      this.mp4boxfile.start();
    };
    
    // 샘플이 추출될 때마다 호출
    this.mp4boxfile.onSamples = (id, user, samples) => {
      console.log(`트랙 ${id}에서 ${samples.length}개 샘플 추출`);
      
      samples.forEach(sample => {
        this.processSample(sample);
      });
    };
  }
  
  processSample(sample) {
    // 샘플 데이터를 WebCodec으로 전달
    const chunk = new EncodedVideoChunk({
      type: sample.is_sync ? 'key' : 'delta',
      timestamp: sample.cts,
      duration: sample.duration,
      data: sample.data
    });
    
    // 디코더로 전달
    this.decoder?.decode(chunk);
  }
  
  async processStream(stream) {
    const reader = stream.getReader();
    let offset = 0;
    
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      
      // ArrayBuffer에 오프셋 정보 추가
      value.fileStart = offset;
      offset += value.byteLength;
      
      // MP4Box에 청크 공급
      this.mp4boxfile.appendBuffer(value);
    }
  }
}

// 네트워크 스트림 처리 예시
const processor = new MP4StreamProcessor();

fetch('https://example.com/video.mp4')
  .then(response => processor.processStream(response.body))
  .catch(console.error);
```

## WebCodec API 고급 활용

### 1. 하드웨어 가속 비디오 디코딩

```javascript
class HardwareAcceleratedDecoder {
  constructor(config) {
    this.frames = [];
    this.decoder = null;
    this.init(config);
  }
  
  async init(config) {
    // 하드웨어 지원 확인
    const support = await VideoDecoder.isConfigSupported(config);
    console.log('디코더 지원:', support);
    
    if (support.supported) {
      this.decoder = new VideoDecoder({
        output: (frame) => this.handleFrame(frame),
        error: (error) => this.handleError(error)
      });
      
      this.decoder.configure(config);
    }
  }
  
  handleFrame(frame) {
    console.log('프레임 디코딩 완료:', {
      timestamp: frame.timestamp,
      duration: frame.duration,
      codedWidth: frame.codedWidth,
      codedHeight: frame.codedHeight
    });
    
    // Canvas에 프레임 그리기
    this.drawFrameToCanvas(frame);
    
    // 메모리 해제
    frame.close();
  }
  
  drawFrameToCanvas(frame) {
    const canvas = document.getElementById('videoCanvas');
    const ctx = canvas.getContext('2d');
    
    // 캔버스 크기 조정
    canvas.width = frame.codedWidth;
    canvas.height = frame.codedHeight;
    
    // VideoFrame을 ImageBitmap으로 변환하여 그리기
    createImageBitmap(frame).then(bitmap => {
      ctx.drawImage(bitmap, 0, 0);
      bitmap.close();
    });
  }
  
  handleError(error) {
    console.error('디코딩 에러:', error);
  }
  
  decode(chunk) {
    if (this.decoder?.state === 'configured') {
      this.decoder.decode(chunk);
    }
  }
  
  close() {
    this.decoder?.close();
  }
}

// 사용 예시
const decoderConfig = {
  codec: 'avc1.42E01E', // H.264 Baseline Profile
  codedWidth: 1920,
  codedHeight: 1080,
  hardwareAcceleration: 'prefer-hardware'
};

const decoder = new HardwareAcceleratedDecoder(decoderConfig);
```

### 2. 실시간 비디오 인코딩

```javascript
class RealTimeVideoEncoder {
  constructor(config) {
    this.encoder = null;
    this.chunks = [];
    this.init(config);
  }
  
  async init(config) {
    const support = await VideoEncoder.isConfigSupported(config);
    
    if (support.supported) {
      this.encoder = new VideoEncoder({
        output: (chunk, metadata) => this.handleEncodedChunk(chunk, metadata),
        error: (error) => this.handleError(error)
      });
      
      this.encoder.configure(config);
    }
  }
  
  handleEncodedChunk(chunk, metadata) {
    console.log('청크 인코딩 완료:', {
      type: chunk.type,
      timestamp: chunk.timestamp,
      byteLength: chunk.byteLength
    });
    
    // 청크를 배열에 저장
    const chunkData = new Uint8Array(chunk.byteLength);
    chunk.copyTo(chunkData);
    
    this.chunks.push({
      data: chunkData,
      type: chunk.type,
      timestamp: chunk.timestamp,
      duration: chunk.duration,
      metadata: metadata
    });
    
    // 실시간 스트리밍이나 저장 처리
    this.processEncodedChunk(chunk, metadata);
  }
  
  processEncodedChunk(chunk, metadata) {
    // WebRTC를 통한 실시간 전송이나
    // 파일 저장 등의 처리를 여기서 수행
  }
  
  handleError(error) {
    console.error('인코딩 에러:', error);
  }
  
  async encodeFrame(videoFrame) {
    if (this.encoder?.state === 'configured') {
      // 키프레임 요청 여부 결정
      const keyFrame = this.chunks.length % 30 === 0; // 30프레임마다 키프레임
      
      this.encoder.encode(videoFrame, { keyFrame });
    }
  }
  
  async flush() {
    await this.encoder?.flush();
  }
  
  close() {
    this.encoder?.close();
  }
  
  exportVideo() {
    // 인코딩된 청크들을 MP4 파일로 결합
    return this.createMP4FromChunks();
  }
  
  createMP4FromChunks() {
    const mp4boxfile = MP4Box.createFile();
    
    // MP4Box를 사용해 청크들을 MP4 컨테이너에 패키징
    // 구현 세부사항은 용도에 따라 달라짐
  }
}

// 웹캠에서 실시간 인코딩 예시
const encoderConfig = {
  codec: 'avc1.42E01E',
  width: 1280,
  height: 720,
  bitrate: 2000000, // 2Mbps
  framerate: 30,
  hardwareAcceleration: 'prefer-hardware'
};

const encoder = new RealTimeVideoEncoder(encoderConfig);

// 웹캠 스트림 캡처
navigator.mediaDevices.getUserMedia({ video: true })
  .then(stream => {
    const video = document.createElement('video');
    video.srcObject = stream;
    video.play();
    
    video.addEventListener('loadedmetadata', () => {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      
      function captureFrame() {
        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;
        ctx.drawImage(video, 0, 0);
        
        // Canvas에서 VideoFrame 생성
        const frame = new VideoFrame(canvas, {
          timestamp: performance.now() * 1000
        });
        
        encoder.encodeFrame(frame);
        frame.close();
        
        requestAnimationFrame(captureFrame);
      }
      
      captureFrame();
    });
  });
```

## MP4Box와 WebCodec 통합 활용

### 1. 고성능 비디오 편집기

```javascript
class WebVideoEditor {
  constructor() {
    this.mp4box = MP4Box.createFile();
    this.decoder = null;
    this.encoder = null;
    this.timeline = [];
    this.setupMP4Box();
  }
  
  setupMP4Box() {
    this.mp4box.onReady = (info) => {
      console.log('비디오 로드 완료:', info);
      this.setupCodecs(info);
    };
    
    this.mp4box.onSamples = (id, user, samples) => {
      samples.forEach(sample => this.processSample(sample));
    };
  }
  
  async setupCodecs(info) {
    const videoTrack = info.tracks.find(track => track.type === 'video');
    
    if (videoTrack) {
      // 디코더 설정
      this.decoder = new VideoDecoder({
        output: (frame) => this.processFrame(frame),
        error: (error) => console.error('Decode error:', error)
      });
      
      this.decoder.configure({
        codec: videoTrack.codec,
        codedWidth: videoTrack.video.width,
        codedHeight: videoTrack.video.height
      });
      
      // 인코더 설정 (편집된 비디오 출력용)
      this.encoder = new VideoEncoder({
        output: (chunk, metadata) => this.handleEditedChunk(chunk, metadata),
        error: (error) => console.error('Encode error:', error)
      });
      
      this.encoder.configure({
        codec: 'avc1.42E01E',
        width: videoTrack.video.width,
        height: videoTrack.video.height,
        bitrate: 5000000,
        framerate: 30
      });
    }
  }
  
  processSample(sample) {
    const chunk = new EncodedVideoChunk({
      type: sample.is_sync ? 'key' : 'delta',
      timestamp: sample.cts,
      duration: sample.duration,
      data: sample.data
    });
    
    this.decoder.decode(chunk);
  }
  
  processFrame(frame) {
    // 프레임에 효과 적용
    const editedFrame = this.applyEffects(frame);
    
    // 편집된 프레임 인코딩
    this.encoder.encode(editedFrame);
    
    // 메모리 정리
    frame.close();
    if (editedFrame !== frame) {
      editedFrame.close();
    }
  }
  
  applyEffects(frame) {
    // Canvas를 사용한 실시간 효과 적용
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    
    canvas.width = frame.codedWidth;
    canvas.height = frame.codedHeight;
    
    // 프레임을 캔버스에 그리기
    createImageBitmap(frame).then(bitmap => {
      ctx.drawImage(bitmap, 0, 0);
      
      // 효과 적용 (예: 필터, 색상 보정 등)
      this.applyFilter(ctx, canvas.width, canvas.height);
      
      bitmap.close();
    });
    
    // 편집된 캔버스에서 새 VideoFrame 생성
    return new VideoFrame(canvas, {
      timestamp: frame.timestamp
    });
  }
  
  applyFilter(ctx, width, height) {
    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    
    // 간단한 세피아 필터
    for (let i = 0; i < data.length; i += 4) {
      const r = data[i];
      const g = data[i + 1];
      const b = data[i + 2];
      
      data[i] = Math.min(255, (r * 0.393) + (g * 0.769) + (b * 0.189));
      data[i + 1] = Math.min(255, (r * 0.349) + (g * 0.686) + (b * 0.168));
      data[i + 2] = Math.min(255, (r * 0.272) + (g * 0.534) + (b * 0.131));
    }
    
    ctx.putImageData(imageData, 0, 0);
  }
  
  handleEditedChunk(chunk, metadata) {
    console.log('편집된 청크:', chunk.byteLength, 'bytes');
    // 편집된 비디오 청크를 저장하거나 스트리밍
  }
  
  async loadVideo(file) {
    const arrayBuffer = await file.arrayBuffer();
    arrayBuffer.fileStart = 0;
    this.mp4box.appendBuffer(arrayBuffer);
  }
  
  startEditing() {
    const info = this.mp4box.getInfo();
    const videoTrack = info.tracks.find(track => track.type === 'video');
    
    if (videoTrack) {
      this.mp4box.setExtractionOptions(videoTrack.id, this, {
        nbSamples: 1000
      });
      this.mp4box.start();
    }
  }
}

// 사용 예시
const editor = new WebVideoEditor();

document.getElementById('videoInput').addEventListener('change', async (event) => {
  const file = event.target.files[0];
  if (file) {
    await editor.loadVideo(file);
    editor.startEditing();
  }
});
```

### 2. 적응형 스트리밍 플레이어

```javascript
class AdaptiveStreamPlayer {
  constructor(videoElement) {
    this.videoElement = videoElement;
    this.mp4box = MP4Box.createFile();
    this.decoder = null;
    this.qualities = [];
    this.currentQuality = 0;
    this.setupPlayer();
  }
  
  setupPlayer() {
    this.mp4box.onReady = (info) => {
      this.initializeDecoder(info);
    };
    
    this.mp4box.onSamples = (id, user, samples) => {
      this.decodeSamples(samples);
    };
  }
  
  async initializeDecoder(info) {
    const videoTrack = info.tracks.find(track => track.type === 'video');
    
    this.decoder = new VideoDecoder({
      output: (frame) => this.displayFrame(frame),
      error: (error) => console.error('Decoder error:', error)
    });
    
    this.decoder.configure({
      codec: videoTrack.codec,
      codedWidth: videoTrack.video.width,
      codedHeight: videoTrack.video.height,
      hardwareAcceleration: 'prefer-hardware'
    });
  }
  
  decodeSamples(samples) {
    samples.forEach(sample => {
      const chunk = new EncodedVideoChunk({
        type: sample.is_sync ? 'key' : 'delta',
        timestamp: sample.cts,
        duration: sample.duration,
        data: sample.data
      });
      
      this.decoder.decode(chunk);
    });
  }
  
  displayFrame(frame) {
    // VideoFrame을 video 엘리먼트에 표시
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    
    canvas.width = frame.codedWidth;
    canvas.height = frame.codedHeight;
    
    createImageBitmap(frame).then(bitmap => {
      ctx.drawImage(bitmap, 0, 0);
      
      // Canvas의 stream을 video 엘리먼트에 연결
      if (!this.videoElement.srcObject) {
        this.videoElement.srcObject = canvas.captureStream();
      }
      
      bitmap.close();
    });
    
    frame.close();
  }
  
  async loadSegment(segmentUrl) {
    try {
      const response = await fetch(segmentUrl);
      const arrayBuffer = await response.arrayBuffer();
      arrayBuffer.fileStart = 0;
      
      this.mp4box.appendBuffer(arrayBuffer);
    } catch (error) {
      console.error('세그먼트 로딩 에러:', error);
    }
  }
  
  // 네트워크 상태에 따른 품질 조정
  adjustQuality() {
    const connection = navigator.connection;
    
    if (connection) {
      const effectiveType = connection.effectiveType;
      
      switch (effectiveType) {
        case '4g':
          this.currentQuality = 2; // 고화질
          break;
        case '3g':
          this.currentQuality = 1; // 중화질
          break;
        default:
          this.currentQuality = 0; // 저화질
          break;
      }
      
      console.log('품질 조정:', this.qualities[this.currentQuality]);
    }
  }
  
  startPlayback() {
    const videoTrack = this.mp4box.getInfo().tracks.find(track => track.type === 'video');
    
    if (videoTrack) {
      this.mp4box.setExtractionOptions(videoTrack.id, this, {
        nbSamples: 100
      });
      this.mp4box.start();
    }
  }
}

// 사용 예시
const player = new AdaptiveStreamPlayer(document.getElementById('adaptiveVideo'));

// HLS 매니페스트 파싱 및 세그먼트 로딩 (의사 코드)
async function playHLSStream(manifestUrl) {
  const manifest = await parseHLSManifest(manifestUrl);
  player.qualities = manifest.qualities;
  
  // 첫 세그먼트 로딩
  await player.loadSegment(manifest.segments[0].url);
  player.startPlayback();
  
  // 나머지 세그먼트들을 순차적으로 로딩
  for (let i = 1; i < manifest.segments.length; i++) {
    await player.loadSegment(manifest.segments[i].url);
  }
}
```

## 성능 최적화 및 브라우저 지원

### 브라우저 지원 현황

```javascript
// 기능 지원 감지
class FeatureDetection {
  static checkWebCodecSupport() {
    const results = {
      videoDecoder: typeof VideoDecoder !== 'undefined',
      videoEncoder: typeof VideoEncoder !== 'undefined',
      audioDecoder: typeof AudioDecoder !== 'undefined',
      audioEncoder: typeof AudioEncoder !== 'undefined'
    };
    
    console.log('WebCodec 지원:', results);
    return results;
  }
  
  static async checkCodecSupport() {
    const codecs = [
      'avc1.42E01E', // H.264 Baseline
      'avc1.4D401E', // H.264 Main
      'avc1.64001E', // H.264 High
      'hev1.1.6.L93.B0', // H.265
      'av01.0.04M.08', // AV1
      'vp8', // VP8
      'vp09.00.10.08' // VP9
    ];
    
    for (const codec of codecs) {
      try {
        const decoderSupport = await VideoDecoder.isConfigSupported({
          codec,
          codedWidth: 1920,
          codedHeight: 1080
        });
        
        const encoderSupport = await VideoEncoder.isConfigSupported({
          codec,
          width: 1920,
          height: 1080,
          bitrate: 5000000,
          framerate: 30
        });
        
        console.log(`${codec}:`, {
          decoder: decoderSupport.supported,
          encoder: encoderSupport.supported
        });
        
      } catch (error) {
        console.log(`${codec}: 지원되지 않음`);
      }
    }
  }
  
  static checkMP4BoxSupport() {
    try {
      const mp4box = MP4Box.createFile();
      console.log('MP4Box 지원: 예');
      return true;
    } catch (error) {
      console.log('MP4Box 지원: 아니요');
      return false;
    }
  }
}

// 사용 예시
FeatureDetection.checkWebCodecSupport();
FeatureDetection.checkCodecSupport();
FeatureDetection.checkMP4BoxSupport();
```

### 메모리 관리 최적화

```javascript
class MemoryOptimizedProcessor {
  constructor() {
    this.framePool = [];
    this.maxPoolSize = 10;
    this.activeFrames = new Set();
  }
  
  getFrame() {
    if (this.framePool.length > 0) {
      return this.framePool.pop();
    }
    return null; // 새 프레임 생성 필요
  }
  
  releaseFrame(frame) {
    if (this.framePool.length < this.maxPoolSize) {
      this.framePool.push(frame);
    } else {
      frame.close();
    }
    
    this.activeFrames.delete(frame);
  }
  
  trackFrame(frame) {
    this.activeFrames.add(frame);
    
    // 메모리 사용량 모니터링
    if (this.activeFrames.size > 20) {
      console.warn('너무 많은 프레임이 활성 상태입니다:', this.activeFrames.size);
    }
  }
  
  cleanupAll() {
    this.activeFrames.forEach(frame => frame.close());
    this.activeFrames.clear();
    
    this.framePool.forEach(frame => frame.close());
    this.framePool = [];
  }
  
  // 메모리 사용량 리포팅
  getMemoryStats() {
    return {
      activeFrames: this.activeFrames.size,
      pooledFrames: this.framePool.length,
      estimatedMemoryMB: (this.activeFrames.size + this.framePool.length) * 8 // 대략적인 추정
    };
  }
}

// 가비지 컬렉션 최적화
class GCOptimizedDecoder {
  constructor(config) {
    this.decoder = new VideoDecoder({
      output: (frame) => this.handleFrame(frame),
      error: (error) => console.error(error)
    });
    
    this.decoder.configure(config);
    this.processedFrames = 0;
    this.gcThreshold = 100;
  }
  
  handleFrame(frame) {
    // 프레임 처리
    this.processFrame(frame);
    
    // 프레임 해제
    frame.close();
    
    this.processedFrames++;
    
    // 주기적으로 가비지 컬렉션 유도
    if (this.processedFrames % this.gcThreshold === 0) {
      // 작은 지연을 두고 GC 실행 유도
      setTimeout(() => {
        if (window.gc) {
          window.gc(); // Chrome DevTools에서만 사용 가능
        }
      }, 0);
    }
  }
  
  processFrame(frame) {
    // 실제 프레임 처리 로직
  }
}
```

## 실무 활용 사례

### 1. 브라우저 기반 비디오 편집 도구

```javascript
class BrowserVideoEditor {
  constructor() {
    this.tracks = [];
    this.effects = [];
    this.timeline = new VideoTimeline();
  }
  
  async addVideoTrack(file) {
    const track = new VideoTrack(file);
    await track.initialize();
    this.tracks.push(track);
    return track;
  }
  
  addEffect(trackIndex, effect, startTime, duration) {
    this.effects.push({
      trackIndex,
      effect,
      startTime,
      duration
    });
  }
  
  async renderVideo(outputConfig) {
    const renderer = new VideoRenderer(outputConfig);
    
    for (let time = 0; time < this.timeline.duration; time += 1/30) {
      const frame = await this.renderFrameAtTime(time);
      renderer.addFrame(frame);
    }
    
    return await renderer.finalize();
  }
  
  async renderFrameAtTime(time) {
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    
    // 각 트랙에서 해당 시간의 프레임 가져오기
    for (const track of this.tracks) {
      const frame = await track.getFrameAtTime(time);
      if (frame) {
        ctx.drawImage(frame, 0, 0);
        frame.close();
      }
    }
    
    // 효과 적용
    this.applyEffectsAtTime(ctx, time);
    
    return new VideoFrame(canvas, { timestamp: time * 1000000 });
  }
  
  applyEffectsAtTime(ctx, time) {
    for (const effect of this.effects) {
      if (time >= effect.startTime && time <= effect.startTime + effect.duration) {
        effect.effect.apply(ctx, time - effect.startTime);
      }
    }
  }
}
```

### 2. 실시간 화상회의 시스템

```javascript
class WebRTCVideoProcessor {
  constructor() {
    this.localStream = null;
    this.encoder = null;
    this.decoder = null;
    this.peerConnections = new Map();
  }
  
  async initializeLocalVideo() {
    this.localStream = await navigator.mediaDevices.getUserMedia({
      video: { width: 1280, height: 720 }
    });
    
    // 인코더 설정
    this.encoder = new VideoEncoder({
      output: (chunk) => this.broadcastChunk(chunk),
      error: (error) => console.error('Encode error:', error)
    });
    
    this.encoder.configure({
      codec: 'avc1.42E01E',
      width: 1280,
      height: 720,
      bitrate: 1000000,
      framerate: 30,
      latencyMode: 'realtime'
    });
    
    this.startCapturing();
  }
  
  startCapturing() {
    const video = document.createElement('video');
    video.srcObject = this.localStream;
    video.play();
    
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    
    const captureFrame = () => {
      if (video.readyState === video.HAVE_ENOUGH_DATA) {
        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;
        ctx.drawImage(video, 0, 0);
        
        const frame = new VideoFrame(canvas, {
          timestamp: performance.now() * 1000
        });
        
        this.encoder.encode(frame, { keyFrame: false });
        frame.close();
      }
      
      requestAnimationFrame(captureFrame);
    };
    
    video.addEventListener('loadedmetadata', captureFrame);
  }
  
  broadcastChunk(chunk) {
    // 모든 피어에게 인코딩된 청크 전송
    for (const [peerId, connection] of this.peerConnections) {
      this.sendChunkToPeer(peerId, chunk);
    }
  }
  
  sendChunkToPeer(peerId, chunk) {
    const data = new Uint8Array(chunk.byteLength);
    chunk.copyTo(data);
    
    const message = {
      type: 'video-chunk',
      data: data,
      timestamp: chunk.timestamp,
      chunkType: chunk.type
    };
    
    // WebRTC 데이터 채널을 통해 전송
    const connection = this.peerConnections.get(peerId);
    if (connection && connection.dataChannel) {
      connection.dataChannel.send(JSON.stringify(message));
    }
  }
  
  handleRemoteChunk(chunk) {
    if (!this.decoder) {
      this.initializeDecoder();
    }
    
    const encodedChunk = new EncodedVideoChunk({
      type: chunk.chunkType,
      timestamp: chunk.timestamp,
      data: chunk.data
    });
    
    this.decoder.decode(encodedChunk);
  }
  
  initializeDecoder() {
    this.decoder = new VideoDecoder({
      output: (frame) => this.displayRemoteFrame(frame),
      error: (error) => console.error('Decode error:', error)
    });
    
    this.decoder.configure({
      codec: 'avc1.42E01E',
      codedWidth: 1280,
      codedHeight: 720
    });
  }
  
  displayRemoteFrame(frame) {
    const remoteVideo = document.getElementById('remoteVideo');
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    
    canvas.width = frame.codedWidth;
    canvas.height = frame.codedHeight;
    
    createImageBitmap(frame).then(bitmap => {
      ctx.drawImage(bitmap, 0, 0);
      
      if (!remoteVideo.srcObject) {
        remoteVideo.srcObject = canvas.captureStream();
      }
      
      bitmap.close();
    });
    
    frame.close();
  }
}
```

## 미래 전망과 발전 방향

웹 플랫폼의 미디어 처리 기술은 계속해서 발전하고 있습니다:

### 1. 하드웨어 가속의 확산
- GPU 기반 비디오 처리의 표준화
- 전용 미디어 프로세서 지원 확대
- 모바일 디바이스에서의 성능 최적화

### 2. 새로운 코덱 지원
- AV1 코덱의 광범위한 지원
- VVC(H.266) 표준화 진행
- 실시간 스트리밍 최적화된 코덱들

### 3. WebAssembly 통합
- 고성능 미디어 라이브러리의 웹 포팅
- FFmpeg 웹 버전의 성능 향상
- 커스텀 코덱 구현의 용이성

## 결론

MP4Box.js와 WebCodec API의 조합은 웹에서 네이티브급 비디오 처리를 가능하게 만들었습니다. 이러한 기술들을 통해:

- **전문적인 비디오 편집**: 브라우저에서 완전한 비디오 편집 도구 구현
- **실시간 스트리밍**: 지연 시간 없는 고품질 라이브 스트리밍
- **효율적인 압축**: 하드웨어 가속을 활용한 실시간 인코딩/디코딩
- **적응형 재생**: 네트워크 상황에 맞는 동적 품질 조정

웹 플랫폼이 미디어 처리의 새로운 표준으로 자리잡고 있는 현재, 이러한 기술들을 마스터하는 것은 미래 웹 애플리케이션 개발에 있어 필수적인 역량이 될 것입니다.