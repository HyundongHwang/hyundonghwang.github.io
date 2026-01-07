---
layout: post
title: "WebCodec API와 MP4Box.js 기술분석"
date: 2024-12-21
categories: [web-development, media-processing]
tags: [webcodec, mp4box, video-processing, browser-api, media-streaming]
---

# WebCodec API와 MP4Box.js 기술분석

웹 브라우저에서 네이티브 수준의 미디어 처리가 가능해진 현재, WebCodec API와 MP4Box.js의 조합은 웹 기반 미디어 애플리케이션의 새로운 가능성을 열어주고 있습니다. 본 분석에서는 두 기술의 심층적인 특징과 실제 활용 방안을 살펴보겠습니다.

## 기술 개요와 배경

### WebCodec API의 등장 배경

```
┌─────────── 웹 미디어 처리 기술의 진화 ───────────┐
│                                               │
│  2010년대 초반:                                │
│  ┌─────────────────────────────────────────┐   │
│  │ <video> 태그 + Media Source Extensions │   │
│  │ • 제한적인 컨테이너 지원                │   │
│  │ • 브라우저 의존적 코덱 지원              │   │
│  │ • 고수준 API만 제공                    │   │
│  └─────────────────────────────────────────┘   │
│                     ↓                         │
│  2010년대 후반:                                │
│  ┌─────────────────────────────────────────┐   │
│  │ WebAssembly + FFmpeg                   │   │
│  │ • 성능 향상하지만 여전히 제약            │   │
│  │ • 메모리 사용량 증가                    │   │
│  │ • 하드웨어 가속 불가                    │   │
│  └─────────────────────────────────────────┘   │
│                     ↓                         │
│  2020년대:                                     │
│  ┌─────────────────────────────────────────┐   │
│  │ WebCodec API                           │   │
│  │ • 네이티브 하드웨어 가속 지원            │   │
│  │ • 저수준 미디어 처리 API                │   │
│  │ • 직접적인 인코더/디코더 제어            │   │
│  └─────────────────────────────────────────┘   │
└───────────────────────────────────────────────┘
```

### MP4Box.js의 역할

MP4Box.js는 GPAC 프로젝트의 JavaScript 포팅으로, MP4 컨테이너 형식의 완전한 파싱과 조작을 웹 환경에서 가능하게 합니다.

```
┌──────── MP4Box.js 핵심 기능 ────────┐
│                                    │
│  MP4 파일 구조 분석:                │
│  ┌────────────────────────────┐     │
│  │ • ftyp (File Type)         │     │
│  │ • moov (Movie Header)      │     │
│  │ • mdat (Media Data)        │     │
│  │ • 트랙별 메타데이터         │     │
│  └────────────────────────────┘     │
│                │                   │
│                ▼                   │
│  실시간 스트림 처리:                 │
│  ┌────────────────────────────┐     │
│  │ • Fragmented MP4 지원      │     │
│  │ • DASH/HLS 세그먼테이션    │     │
│  │ • 점진적 다운로드 지원      │     │
│  └────────────────────────────┘     │
│                │                   │
│                ▼                   │
│  WebCodec 연동:                    │
│  ┌────────────────────────────┐     │
│  │ • EncodedVideoChunk 생성   │     │
│  │ • 코덱 설정 정보 추출       │     │
│  │ • 타이밍 동기화            │     │
│  └────────────────────────────┘     │
└────────────────────────────────────┘
```

## WebCodec API 심층 분석

### VideoDecoder 상세 구현

```typescript
interface VideoDecoderInit {
  output: VideoFrameOutputCallback;
  error: WebCodecsErrorCallback;
}

interface VideoDecoderConfig {
  codec: string;
  description?: AllowSharedBufferSource;
  codedWidth?: number;
  codedHeight?: number;
  displayAspectWidth?: number;
  displayAspectHeight?: number;
  colorSpace?: VideoColorSpaceInit;
  hardwareAcceleration?: HardwareAcceleration;
  optimizeForLatency?: boolean;
}

class AdvancedVideoDecoder {
  private decoder: VideoDecoder;
  private frameBuffer: VideoFrame[] = [];
  private frameCounter: number = 0;
  private decodingStats: DecodingStats;
  
  constructor(config: VideoDecoderConfig) {
    this.decodingStats = new DecodingStats();
    
    this.decoder = new VideoDecoder({
      output: (frame: VideoFrame) => this.handleDecodedFrame(frame),
      error: (error: Error) => this.handleDecodingError(error)
    });
    
    this.configureDecoder(config);
  }
  
  private async configureDecoder(config: VideoDecoderConfig): Promise<void> {
    // 브라우저 지원 확인
    const support = await VideoDecoder.isConfigSupported(config);
    
    if (!support.supported) {
      throw new Error(`Decoder config not supported: ${support.config}`);
    }
    
    // 하드웨어 가속 최적화
    const optimizedConfig = {
      ...config,
      hardwareAcceleration: 'prefer-hardware' as HardwareAcceleration,
      optimizeForLatency: true
    };
    
    this.decoder.configure(optimizedConfig);
    console.log('Decoder configured with hardware acceleration');
  }
  
  private handleDecodedFrame(frame: VideoFrame): void {
    this.frameCounter++;
    this.decodingStats.recordFrame(frame);
    
    // 프레임 버퍼 관리
    this.manageFrameBuffer(frame);
    
    // 렌더링 또는 후처리
    this.processFrame(frame);
  }
  
  private manageFrameBuffer(frame: VideoFrame): void {
    // 최대 버퍼 크기 제한
    const MAX_BUFFER_SIZE = 10;
    
    if (this.frameBuffer.length >= MAX_BUFFER_SIZE) {
      const oldFrame = this.frameBuffer.shift();
      if (oldFrame) {
        oldFrame.close(); // 메모리 해제
      }
    }
    
    this.frameBuffer.push(frame);
  }
  
  private processFrame(frame: VideoFrame): void {
    // Canvas에 프레임 렌더링
    const canvas = document.getElementById('videoCanvas') as HTMLCanvasElement;
    const ctx = canvas.getContext('2d');
    
    if (ctx && canvas) {
      canvas.width = frame.codedWidth;
      canvas.height = frame.codedHeight;
      
      // VideoFrame을 ImageBitmap으로 변환하여 그리기
      createImageBitmap(frame).then(bitmap => {
        ctx.drawImage(bitmap, 0, 0);
        bitmap.close();
        
        // 프레임 닫기 (메모리 관리)
        frame.close();
      });
    }
  }
  
  public decode(chunk: EncodedVideoChunk): void {
    if (this.decoder.state === 'configured') {
      this.decodingStats.recordChunk(chunk);
      this.decoder.decode(chunk);
    }
  }
  
  private handleDecodingError(error: Error): void {
    console.error('Decoding error:', error);
    this.decodingStats.recordError(error);
    
    // 에러 복구 시도
    this.attemptErrorRecovery();
  }
  
  private attemptErrorRecovery(): void {
    // 디코더 재설정 시도
    try {
      this.decoder.reset();
      console.log('Decoder reset successful');
    } catch (error) {
      console.error('Decoder recovery failed:', error);
    }
  }
  
  public getDecodingStats(): DecodingStats {
    return this.decodingStats;
  }
  
  public close(): void {
    // 모든 프레임 해제
    this.frameBuffer.forEach(frame => frame.close());
    this.frameBuffer = [];
    
    // 디코더 닫기
    this.decoder.close();
  }
}

// 디코딩 통계 클래스
class DecodingStats {
  private framesDecoded: number = 0;
  private chunksReceived: number = 0;
  private errors: Error[] = [];
  private startTime: number = Date.now();
  private frameTimestamps: number[] = [];
  
  recordFrame(frame: VideoFrame): void {
    this.framesDecoded++;
    this.frameTimestamps.push(frame.timestamp);
  }
  
  recordChunk(chunk: EncodedVideoChunk): void {
    this.chunksReceived++;
  }
  
  recordError(error: Error): void {
    this.errors.push(error);
  }
  
  getFrameRate(): number {
    if (this.frameTimestamps.length < 2) return 0;
    
    const duration = (Date.now() - this.startTime) / 1000;
    return this.framesDecoded / duration;
  }
  
  getDecodeLatency(): number {
    // 평균 디코딩 지연시간 계산
    if (this.frameTimestamps.length < 2) return 0;
    
    const intervals = [];
    for (let i = 1; i < this.frameTimestamps.length; i++) {
      intervals.push(this.frameTimestamps[i] - this.frameTimestamps[i-1]);
    }
    
    return intervals.reduce((a, b) => a + b, 0) / intervals.length;
  }
  
  getSummary(): object {
    return {
      framesDecoded: this.framesDecoded,
      chunksReceived: this.chunksReceived,
      errorCount: this.errors.length,
      frameRate: this.getFrameRate(),
      decodeLatency: this.getDecodeLatency()
    };
  }
}
```

### VideoEncoder 고급 기능

```typescript
class AdvancedVideoEncoder {
  private encoder: VideoEncoder;
  private encodingQueue: VideoFrame[] = [];
  private isProcessing: boolean = false;
  private encodingStats: EncodingStats;
  
  constructor(config: VideoEncoderConfig) {
    this.encodingStats = new EncodingStats();
    
    this.encoder = new VideoEncoder({
      output: (chunk, metadata) => this.handleEncodedChunk(chunk, metadata),
      error: (error) => this.handleEncodingError(error)
    });
    
    this.configureEncoder(config);
  }
  
  private async configureEncoder(config: VideoEncoderConfig): Promise<void> {
    // 동적 비트레이트 조정을 위한 확장 설정
    const adaptiveConfig = {
      ...config,
      bitrateMode: 'variable' as BitrateMode,
      latencyMode: 'realtime' as LatencyMode,
      
      // 프레임 품질 설정
      alpha: 'discard' as AlphaOption,
      scalabilityMode: 'L1T1' // 단일 레이어 인코딩
    };
    
    const support = await VideoEncoder.isConfigSupported(adaptiveConfig);
    if (!support.supported) {
      throw new Error('Encoder configuration not supported');
    }
    
    this.encoder.configure(adaptiveConfig);
  }
  
  public async encodeFrame(frame: VideoFrame, options?: VideoEncoderEncodeOptions): Promise<void> {
    // 인코딩 큐에 프레임 추가
    this.encodingQueue.push(frame);
    
    if (!this.isProcessing) {
      this.processEncodingQueue(options);
    }
  }
  
  private async processEncodingQueue(options?: VideoEncoderEncodeOptions): Promise<void> {
    this.isProcessing = true;
    
    while (this.encodingQueue.length > 0) {
      const frame = this.encodingQueue.shift();
      if (!frame) continue;
      
      try {
        // 적응적 키프레임 삽입
        const needsKeyframe = this.shouldInsertKeyframe(frame);
        const encodeOptions = {
          ...options,
          keyFrame: needsKeyframe
        };
        
        // 인코딩 시작 시간 기록
        const encodeStartTime = performance.now();
        
        this.encoder.encode(frame, encodeOptions);
        this.encodingStats.recordFrameStart(frame, encodeStartTime);
        
        // 프레임 해제
        frame.close();
        
      } catch (error) {
        console.error('Frame encoding failed:', error);
        frame.close();
      }
    }
    
    this.isProcessing = false;
  }
  
  private shouldInsertKeyframe(frame: VideoFrame): boolean {
    const stats = this.encodingStats.getSummary();
    
    // 조건부 키프레임 삽입
    const timeSinceLastKeyframe = frame.timestamp - stats.lastKeyframeTimestamp;
    const framesSinceLastKeyframe = stats.framesSinceLastKeyframe;
    
    // 2초마다 또는 30프레임마다 키프레임 삽입
    return timeSinceLastKeyframe > 2000000 || framesSinceLastKeyframe > 30;
  }
  
  private handleEncodedChunk(chunk: EncodedVideoChunk, metadata?: EncodedVideoChunkMetadata): void {
    this.encodingStats.recordChunk(chunk, metadata);
    
    // 청크를 스트리밍 또는 저장
    this.processEncodedChunk(chunk, metadata);
  }
  
  private processEncodedChunk(chunk: EncodedVideoChunk, metadata?: EncodedVideoChunkMetadata): void {
    // MP4Box를 사용하여 컨테이너에 패키징
    this.packageToMP4(chunk, metadata);
    
    // 또는 WebRTC를 통한 실시간 전송
    this.streamViaWebRTC(chunk);
  }
  
  private packageToMP4(chunk: EncodedVideoChunk, metadata?: EncodedVideoChunkMetadata): void {
    // MP4Box.js 통합 (다음 섹션에서 상세 설명)
    // 청크를 MP4 컨테이너에 추가하는 로직
  }
  
  private streamViaWebRTC(chunk: EncodedVideoChunk): void {
    // WebRTC를 통한 실시간 스트리밍
    // RTCPeerConnection을 통한 전송 로직
  }
  
  private handleEncodingError(error: Error): void {
    console.error('Encoding error:', error);
    this.encodingStats.recordError(error);
  }
  
  public async flush(): Promise<void> {
    await this.encoder.flush();
    this.encodingStats.recordFlush();
  }
  
  public close(): void {
    // 대기 중인 프레임들 정리
    this.encodingQueue.forEach(frame => frame.close());
    this.encodingQueue = [];
    
    this.encoder.close();
  }
  
  public getEncodingStats(): EncodingStats {
    return this.encodingStats;
  }
}

class EncodingStats {
  private framesEncoded: number = 0;
  private chunksGenerated: number = 0;
  private lastKeyframeTimestamp: number = 0;
  private framesSinceLastKeyframe: number = 0;
  private encodeTimes: number[] = [];
  private errors: Error[] = [];
  
  recordFrameStart(frame: VideoFrame, startTime: number): void {
    this.framesEncoded++;
    // 인코딩 시작 시간 저장 (완료 시 계산용)
  }
  
  recordChunk(chunk: EncodedVideoChunk, metadata?: EncodedVideoChunkMetadata): void {
    this.chunksGenerated++;
    
    if (chunk.type === 'key') {
      this.lastKeyframeTimestamp = chunk.timestamp;
      this.framesSinceLastKeyframe = 0;
    } else {
      this.framesSinceLastKeyframe++;
    }
  }
  
  recordError(error: Error): void {
    this.errors.push(error);
  }
  
  recordFlush(): void {
    console.log('Encoder flush completed');
  }
  
  getSummary(): object {
    return {
      framesEncoded: this.framesEncoded,
      chunksGenerated: this.chunksGenerated,
      lastKeyframeTimestamp: this.lastKeyframeTimestamp,
      framesSinceLastKeyframe: this.framesSinceLastKeyframe,
      errorCount: this.errors.length,
      averageEncodeTime: this.encodeTimes.reduce((a, b) => a + b, 0) / this.encodeTimes.length
    };
  }
}
```

## MP4Box.js 고급 활용

### 실시간 스트림 파싱

```typescript
class RealTimeMP4Parser {
  private mp4boxFile: any;
  private chunkBuffer: ArrayBuffer[] = [];
  private totalBytesReceived: number = 0;
  private onTrackReady?: (trackInfo: any) => void;
  private onSampleReady?: (sample: any) => void;
  
  constructor() {
    this.mp4boxFile = MP4Box.createFile();
    this.setupEventHandlers();
  }
  
  private setupEventHandlers(): void {
    // 파일 준비 완료 이벤트
    this.mp4boxFile.onReady = (info: any) => {
      console.log('MP4 파일 준비 완료:', info);
      this.handleFileReady(info);
    };
    
    // 샘플 데이터 추출 이벤트
    this.mp4boxFile.onSamples = (trackId: number, user: any, samples: any[]) => {
      console.log(`트랙 ${trackId}에서 ${samples.length}개 샘플 추출`);
      this.handleSamples(trackId, samples);
    };
    
    // 세그먼트 생성 이벤트 (DASH/HLS)
    this.mp4boxFile.onSegment = (id: number, user: any, buffer: ArrayBuffer) => {
      console.log(`세그먼트 ${id} 생성: ${buffer.byteLength} bytes`);
      this.handleSegment(id, buffer);
    };
    
    // 에러 처리
    this.mp4boxFile.onError = (error: any) => {
      console.error('MP4Box 에러:', error);
    };
  }
  
  public appendData(data: ArrayBuffer): void {
    // 오프셋 정보 추가 (MP4Box 요구사항)
    data.fileStart = this.totalBytesReceived;
    this.totalBytesReceived += data.byteLength;
    
    // MP4Box에 데이터 공급
    this.mp4boxFile.appendBuffer(data);
  }
  
  private handleFileReady(info: any): void {
    console.log('비디오 정보:', {
      duration: info.duration / info.timescale,
      tracks: info.tracks.length,
      fragmentedMp4: info.isFragmented
    });
    
    // 각 트랙 분석
    info.tracks.forEach((track: any, index: number) => {
      console.log(`트랙 ${index}:`, {
        id: track.id,
        type: track.type,
        codec: track.codec,
        language: track.language
      });
      
      if (track.type === 'video') {
        this.setupVideoTrack(track);
      } else if (track.type === 'audio') {
        this.setupAudioTrack(track);
      }
      
      // 트랙 준비 완료 콜백
      if (this.onTrackReady) {
        this.onTrackReady(track);
      }
    });
    
    // 샘플 추출 시작
    this.startExtraction(info.tracks);
  }
  
  private setupVideoTrack(track: any): VideoDecoderConfig {
    // WebCodec VideoDecoder 설정 생성
    const config: VideoDecoderConfig = {
      codec: track.codec,
      codedWidth: track.video.width,
      codedHeight: track.video.height
    };
    
    // AVCC (AVC Configuration) 데이터 처리
    if (track.codec.startsWith('avc1')) {
      config.description = this.extractAVCC(track);
    }
    
    return config;
  }
  
  private setupAudioTrack(track: any): AudioDecoderConfig {
    // WebCodec AudioDecoder 설정 생성
    const config: AudioDecoderConfig = {
      codec: track.codec,
      sampleRate: track.audio.sample_rate,
      numberOfChannels: track.audio.channel_count
    };
    
    // AAC AudioSpecificConfig 처리
    if (track.codec.startsWith('mp4a')) {
      config.description = this.extractAudioSpecificConfig(track);
    }
    
    return config;
  }
  
  private extractAVCC(track: any): Uint8Array {
    // AVC Configuration Box에서 SPS/PPS 추출
    const avcC = track.avcC;
    if (!avcC) return new Uint8Array();
    
    // H.264 초기화 데이터 구성
    const spsArray = avcC.SPS;
    const ppsArray = avcC.PPS;
    
    // AVCC 형태로 패키징
    let totalLength = 7; // 기본 헤더 크기
    spsArray.forEach((sps: Uint8Array) => totalLength += 2 + sps.length);
    ppsArray.forEach((pps: Uint8Array) => totalLength += 2 + pps.length);
    
    const avccData = new Uint8Array(totalLength);
    let offset = 0;
    
    // AVCC 헤더 작성
    avccData[offset++] = 1; // configurationVersion
    avccData[offset++] = avcC.AVCProfileIndication;
    avccData[offset++] = avcC.profile_compatibility;
    avccData[offset++] = avcC.AVCLevelIndication;
    avccData[offset++] = 0xFF; // lengthSizeMinusOne
    avccData[offset++] = 0xE0 | spsArray.length; // SPS 개수
    
    // SPS 데이터 작성
    spsArray.forEach((sps: Uint8Array) => {
      avccData[offset++] = (sps.length >> 8) & 0xFF;
      avccData[offset++] = sps.length & 0xFF;
      avccData.set(sps, offset);
      offset += sps.length;
    });
    
    // PPS 개수
    avccData[offset++] = ppsArray.length;
    
    // PPS 데이터 작성
    ppsArray.forEach((pps: Uint8Array) => {
      avccData[offset++] = (pps.length >> 8) & 0xFF;
      avccData[offset++] = pps.length & 0xFF;
      avccData.set(pps, offset);
      offset += pps.length;
    });
    
    return avccData;
  }
  
  private extractAudioSpecificConfig(track: any): Uint8Array {
    // AAC AudioSpecificConfig 추출
    const audioConfig = track.audioConfig;
    if (!audioConfig) return new Uint8Array();
    
    // AudioSpecificConfig 구성
    const config = new Uint8Array(2);
    config[0] = (audioConfig.audioObjectType << 3) | (audioConfig.samplingFrequencyIndex >> 1);
    config[1] = ((audioConfig.samplingFrequencyIndex & 1) << 7) | (audioConfig.channelConfiguration << 3);
    
    return config;
  }
  
  private startExtraction(tracks: any[]): void {
    // 각 트랙에 대해 샘플 추출 설정
    tracks.forEach(track => {
      this.mp4boxFile.setExtractionOptions(track.id, this, {
        nbSamples: 100 // 100개 샘플씩 배치 처리
      });
    });
    
    // 추출 시작
    this.mp4boxFile.start();
  }
  
  private handleSamples(trackId: number, samples: any[]): void {
    samples.forEach(sample => {
      // EncodedVideoChunk 또는 EncodedAudioChunk 생성
      const chunk = this.createWebCodecChunk(sample);
      
      // 샘플 준비 완료 콜백
      if (this.onSampleReady) {
        this.onSampleReady({
          trackId,
          chunk,
          sample
        });
      }
    });
  }
  
  private createWebCodecChunk(sample: any): EncodedVideoChunk | EncodedAudioChunk {
    // 샘플 데이터를 WebCodec 청크로 변환
    const chunkData = {
      type: sample.is_sync ? 'key' as const : 'delta' as const,
      timestamp: sample.cts, // Composition Time Stamp
      duration: sample.duration,
      data: sample.data
    };
    
    // 비디오 또는 오디오 청크 생성
    if (sample.description && sample.description.type === 'video') {
      return new EncodedVideoChunk(chunkData);
    } else {
      return new EncodedAudioChunk(chunkData);
    }
  }
  
  private handleSegment(id: number, buffer: ArrayBuffer): void {
    // DASH/HLS 세그먼트 처리
    console.log(`세그먼트 ${id} 처리: ${buffer.byteLength} bytes`);
    
    // 세그먼트를 서버에 업로드하거나 CDN에 저장
    this.uploadSegment(id, buffer);
  }
  
  private async uploadSegment(id: number, buffer: ArrayBuffer): Promise<void> {
    try {
      const response = await fetch(`/api/segments/${id}`, {
        method: 'POST',
        headers: {
          'Content-Type': 'video/mp4'
        },
        body: buffer
      });
      
      if (response.ok) {
        console.log(`세그먼트 ${id} 업로드 완료`);
      }
    } catch (error) {
      console.error(`세그먼트 ${id} 업로드 실패:`, error);
    }
  }
  
  public setTrackReadyCallback(callback: (trackInfo: any) => void): void {
    this.onTrackReady = callback;
  }
  
  public setSampleReadyCallback(callback: (sample: any) => void): void {
    this.onSampleReady = callback;
  }
  
  public createDASHSegments(trackId: number, segmentDuration: number = 2000): void {
    // DASH 세그멘테이션 설정
    this.mp4boxFile.setSegmentOptions(trackId, this, {
      duration: segmentDuration, // 밀리초 단위
      rapAlignement: true // Random Access Point 정렬
    });
  }
  
  public flush(): void {
    // 버퍼링된 데이터 모두 처리
    this.mp4boxFile.flush();
  }
}
```

### MP4 파일 생성 및 수정

```typescript
class MP4FileGenerator {
  private mp4boxFile: any;
  private tracks: Map<number, any> = new Map();
  private nextTrackId: number = 1;
  
  constructor() {
    this.mp4boxFile = MP4Box.createFile();
  }
  
  public addVideoTrack(config: {
    width: number;
    height: number;
    codec: string;
    timescale?: number;
    framerate?: number;
  }): number {
    const trackId = this.nextTrackId++;
    
    // 비디오 트랙 추가
    const trackInfo = this.mp4boxFile.addTrack({
      id: trackId,
      type: 'video',
      width: config.width,
      height: config.height,
      timescale: config.timescale || 90000,
      duration: 0,
      codec: config.codec,
      avcDecoderConfigRecord: this.createAVCConfig(config)
    });
    
    this.tracks.set(trackId, {
      type: 'video',
      info: trackInfo,
      sampleCount: 0
    });
    
    return trackId;
  }
  
  public addAudioTrack(config: {
    sampleRate: number;
    channels: number;
    codec: string;
    timescale?: number;
  }): number {
    const trackId = this.nextTrackId++;
    
    // 오디오 트랙 추가
    const trackInfo = this.mp4boxFile.addTrack({
      id: trackId,
      type: 'audio',
      timescale: config.timescale || config.sampleRate,
      samplerate: config.sampleRate,
      channel_count: config.channels,
      hdlr: 'soun',
      name: 'audio track',
      codec: config.codec
    });
    
    this.tracks.set(trackId, {
      type: 'audio',
      info: trackInfo,
      sampleCount: 0
    });
    
    return trackId;
  }
  
  private createAVCConfig(config: any): Uint8Array {
    // 간단한 AVC 설정 레코드 생성
    // 실제 구현에서는 SPS/PPS 파라미터가 필요
    const avcConfig = new Uint8Array(7);
    avcConfig[0] = 1; // configurationVersion
    avcConfig[1] = 0x64; // AVCProfileIndication (High Profile)
    avcConfig[2] = 0x00; // profile_compatibility
    avcConfig[3] = 0x1f; // AVCLevelIndication (Level 3.1)
    avcConfig[4] = 0xff; // lengthSizeMinusOne
    avcConfig[5] = 0xe0; // SPS 개수 (0)
    avcConfig[6] = 0x00; // PPS 개수 (0)
    
    return avcConfig;
  }
  
  public addSample(trackId: number, chunk: EncodedVideoChunk | EncodedAudioChunk): void {
    const track = this.tracks.get(trackId);
    if (!track) {
      throw new Error(`트랙 ${trackId}를 찾을 수 없습니다`);
    }
    
    // 청크 데이터를 ArrayBuffer로 복사
    const sampleData = new ArrayBuffer(chunk.byteLength);
    chunk.copyTo(sampleData);
    
    // MP4Box 샘플 형태로 변환
    const sample = {
      data: sampleData,
      size: chunk.byteLength,
      duration: chunk.duration || 3000, // 기본값
      dts: chunk.timestamp,
      cts: chunk.timestamp,
      is_sync: chunk.type === 'key'
    };
    
    // 샘플 추가
    this.mp4boxFile.addSample(trackId, sample.data, sample);
    
    track.sampleCount++;
  }
  
  public finalize(): ArrayBuffer {
    // MP4 파일 완성
    const mp4ArrayBuffer = this.mp4boxFile.getBuffer();
    
    console.log('MP4 파일 생성 완료:', {
      size: mp4ArrayBuffer.byteLength,
      tracks: this.tracks.size,
      samples: Array.from(this.tracks.values()).reduce((sum, track) => sum + track.sampleCount, 0)
    });
    
    return mp4ArrayBuffer;
  }
  
  public save(filename: string = 'generated-video.mp4'): void {
    const mp4Buffer = this.finalize();
    
    // Blob 생성 후 다운로드
    const blob = new Blob([mp4Buffer], { type: 'video/mp4' });
    const url = URL.createObjectURL(blob);
    
    const link = document.createElement('a');
    link.href = url;
    link.download = filename;
    link.click();
    
    // 메모리 정리
    URL.revokeObjectURL(url);
  }
}
```

## 통합 활용 사례

### 실시간 비디오 편집기

```typescript
class WebVideoEditor {
  private decoder: AdvancedVideoDecoder;
  private encoder: AdvancedVideoEncoder;
  private mp4Parser: RealTimeMP4Parser;
  private fileGenerator: MP4FileGenerator;
  private canvas: HTMLCanvasElement;
  private ctx: CanvasRenderingContext2D;
  
  constructor(canvasId: string) {
    this.canvas = document.getElementById(canvasId) as HTMLCanvasElement;
    this.ctx = this.canvas.getContext('2d')!;
    
    this.initializeComponents();
  }
  
  private initializeComponents(): void {
    // MP4 파서 설정
    this.mp4Parser = new RealTimeMP4Parser();
    this.mp4Parser.setTrackReadyCallback((track) => this.handleTrackReady(track));
    this.mp4Parser.setSampleReadyCallback((sample) => this.handleSampleReady(sample));
    
    // 파일 생성기 설정
    this.fileGenerator = new MP4FileGenerator();
  }
  
  private handleTrackReady(track: any): void {
    if (track.type === 'video') {
      // 디코더 설정
      const decoderConfig: VideoDecoderConfig = {
        codec: track.codec,
        codedWidth: track.video.width,
        codedHeight: track.video.height
      };
      
      this.decoder = new AdvancedVideoDecoder(decoderConfig);
      
      // 인코더 설정 (편집된 비디오용)
      const encoderConfig: VideoEncoderConfig = {
        codec: 'avc1.42E01E', // H.264 Baseline
        width: track.video.width,
        height: track.video.height,
        bitrate: 2000000, // 2Mbps
        framerate: 30
      };
      
      this.encoder = new AdvancedVideoEncoder(encoderConfig);
      
      // 출력 트랙 생성
      this.fileGenerator.addVideoTrack({
        width: track.video.width,
        height: track.video.height,
        codec: 'avc1.42E01E',
        framerate: 30
      });
    }
  }
  
  private handleSampleReady(sampleData: any): void {
    if (this.decoder) {
      // 디코딩
      this.decoder.decode(sampleData.chunk);
    }
  }
  
  public async loadVideo(file: File): Promise<void> {
    const arrayBuffer = await file.arrayBuffer();
    this.mp4Parser.appendData(arrayBuffer);
  }
  
  public applyFilter(filterType: string): void {
    // 필터 적용 로직
    switch (filterType) {
      case 'sepia':
        this.applySepia();
        break;
      case 'blur':
        this.applyBlur();
        break;
      case 'sharpen':
        this.applySharpen();
        break;
    }
  }
  
  private applySepia(): void {
    const imageData = this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height);
    const data = imageData.data;
    
    for (let i = 0; i < data.length; i += 4) {
      const r = data[i];
      const g = data[i + 1];
      const b = data[i + 2];
      
      data[i] = Math.min(255, (r * 0.393) + (g * 0.769) + (b * 0.189));
      data[i + 1] = Math.min(255, (r * 0.349) + (g * 0.686) + (b * 0.168));
      data[i + 2] = Math.min(255, (r * 0.272) + (g * 0.534) + (b * 0.131));
    }
    
    this.ctx.putImageData(imageData, 0, 0);
  }
  
  private applyBlur(): void {
    // Canvas blur 필터 적용
    this.ctx.filter = 'blur(2px)';
    
    // 현재 캔버스 내용을 복사하여 블러 적용
    const imageData = this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height);
    this.ctx.putImageData(imageData, 0, 0);
    
    // 필터 리셋
    this.ctx.filter = 'none';
  }
  
  private applySharpen(): void {
    // 샤프닝 컨볼루션 필터 구현
    const imageData = this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height);
    const data = imageData.data;
    const width = this.canvas.width;
    const height = this.canvas.height;
    
    // 샤프닝 커널
    const kernel = [
      0, -1, 0,
      -1, 5, -1,
      0, -1, 0
    ];
    
    const output = new Uint8ClampedArray(data);
    
    for (let y = 1; y < height - 1; y++) {
      for (let x = 1; x < width - 1; x++) {
        for (let c = 0; c < 3; c++) { // RGB 채널만
          let sum = 0;
          
          for (let ky = -1; ky <= 1; ky++) {
            for (let kx = -1; kx <= 1; kx++) {
              const pixel = ((y + ky) * width + (x + kx)) * 4 + c;
              const kernelValue = kernel[(ky + 1) * 3 + (kx + 1)];
              sum += data[pixel] * kernelValue;
            }
          }
          
          output[(y * width + x) * 4 + c] = Math.max(0, Math.min(255, sum));
        }
      }
    }
    
    const outputImageData = new ImageData(output, width, height);
    this.ctx.putImageData(outputImageData, 0, 0);
  }
  
  public async exportVideo(): Promise<void> {
    // 편집된 프레임들을 인코딩하여 새 비디오 생성
    
    // Canvas에서 VideoFrame 생성
    const frame = new VideoFrame(this.canvas, {
      timestamp: performance.now() * 1000
    });
    
    // 인코딩
    await this.encoder.encodeFrame(frame);
    
    // MP4 파일로 저장
    this.fileGenerator.save('edited-video.mp4');
  }
  
  public getProcessingStats(): object {
    return {
      decoder: this.decoder?.getDecodingStats().getSummary(),
      encoder: this.encoder?.getEncodingStats().getSummary()
    };
  }
}

// 사용 예시
const videoEditor = new WebVideoEditor('editorCanvas');

// 비디오 파일 로드
document.getElementById('fileInput')?.addEventListener('change', async (event) => {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (file) {
    await videoEditor.loadVideo(file);
  }
});

// 필터 적용
document.getElementById('sepiaBtn')?.addEventListener('click', () => {
  videoEditor.applyFilter('sepia');
});

// 비디오 내보내기
document.getElementById('exportBtn')?.addEventListener('click', () => {
  videoEditor.exportVideo();
});
```

### 라이브 스트리밍 시스템

```typescript
class LiveStreamingSystem {
  private mediaStream: MediaStream | null = null;
  private encoder: AdvancedVideoEncoder;
  private websocket: WebSocket;
  private isStreaming: boolean = false;
  private frameCount: number = 0;
  
  constructor(websocketUrl: string) {
    this.websocket = new WebSocket(websocketUrl);
    this.setupWebSocket();
  }
  
  private setupWebSocket(): void {
    this.websocket.onopen = () => {
      console.log('WebSocket 연결됨');
    };
    
    this.websocket.onmessage = (event) => {
      const message = JSON.parse(event.data);
      this.handleServerMessage(message);
    };
    
    this.websocket.onclose = () => {
      console.log('WebSocket 연결 종료');
      this.stopStreaming();
    };
  }
  
  private handleServerMessage(message: any): void {
    switch (message.type) {
      case 'bitrate_change':
        this.adjustBitrate(message.bitrate);
        break;
      case 'keyframe_request':
        this.requestKeyframe();
        break;
    }
  }
  
  public async startStreaming(constraints: MediaStreamConstraints): Promise<void> {
    try {
      // 미디어 스트림 획득
      this.mediaStream = await navigator.mediaDevices.getUserMedia(constraints);
      
      // 인코더 설정
      const video = constraints.video as MediaTrackConstraints;
      const encoderConfig: VideoEncoderConfig = {
        codec: 'avc1.42E01E',
        width: video.width as number || 1280,
        height: video.height as number || 720,
        bitrate: 2000000,
        framerate: 30,
        latencyMode: 'realtime',
        bitrateMode: 'variable'
      };
      
      this.encoder = new AdvancedVideoEncoder(encoderConfig);
      
      // 비디오 캡처 시작
      this.startVideoCapture();
      
      this.isStreaming = true;
      console.log('라이브 스트리밍 시작');
      
    } catch (error) {
      console.error('스트리밍 시작 실패:', error);
    }
  }
  
  private startVideoCapture(): void {
    if (!this.mediaStream) return;
    
    const video = document.createElement('video');
    video.srcObject = this.mediaStream;
    video.play();
    
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d')!;
    
    video.addEventListener('loadedmetadata', () => {
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;
      
      this.captureFrames(video, canvas, ctx);
    });
  }
  
  private captureFrames(video: HTMLVideoElement, canvas: HTMLCanvasElement, ctx: CanvasRenderingContext2D): void {
    if (!this.isStreaming) return;
    
    // 비디오 프레임을 캔버스에 그리기
    ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
    
    // VideoFrame 생성
    const timestamp = performance.now() * 1000;
    const frame = new VideoFrame(canvas, { timestamp });
    
    // 키프레임 주기 결정 (매 30프레임마다)
    const isKeyframe = this.frameCount % 30 === 0;
    
    // 인코딩
    this.encoder.encodeFrame(frame, { keyFrame: isKeyframe })
      .then(() => {
        frame.close();
        this.frameCount++;
      })
      .catch(error => {
        console.error('프레임 인코딩 실패:', error);
        frame.close();
      });
    
    // 다음 프레임 스케줄링 (30fps)
    setTimeout(() => this.captureFrames(video, canvas, ctx), 1000 / 30);
  }
  
  private streamEncodedChunk(chunk: EncodedVideoChunk, metadata?: EncodedVideoChunkMetadata): void {
    if (!this.websocket || this.websocket.readyState !== WebSocket.OPEN) {
      return;
    }
    
    // 청크 데이터를 ArrayBuffer로 복사
    const chunkData = new ArrayBuffer(chunk.byteLength);
    chunk.copyTo(chunkData);
    
    // 메타데이터와 함께 전송
    const message = {
      type: 'video_chunk',
      timestamp: chunk.timestamp,
      duration: chunk.duration,
      keyframe: chunk.type === 'key',
      data: Array.from(new Uint8Array(chunkData)) // JSON 직렬화를 위해 배열로 변환
    };
    
    this.websocket.send(JSON.stringify(message));
  }
  
  private adjustBitrate(newBitrate: number): void {
    // 동적 비트레이트 조정
    console.log(`비트레이트 조정: ${newBitrate}bps`);
    
    // 새 인코더 설정으로 재구성 (실제로는 더 효율적인 방법 필요)
    // 현재 WebCodec API는 실시간 비트레이트 조정이 제한적
  }
  
  private requestKeyframe(): void {
    console.log('키프레임 요청 받음');
    // 다음 프레임을 키프레임으로 강제 설정
    this.frameCount = 0;
  }
  
  public stopStreaming(): void {
    this.isStreaming = false;
    
    if (this.mediaStream) {
      this.mediaStream.getTracks().forEach(track => track.stop());
      this.mediaStream = null;
    }
    
    if (this.encoder) {
      this.encoder.close();
    }
    
    console.log('라이브 스트리밍 중지');
  }
  
  public getStreamingStats(): object {
    return {
      frameCount: this.frameCount,
      isStreaming: this.isStreaming,
      encoderStats: this.encoder?.getEncodingStats().getSummary(),
      websocketState: this.websocket?.readyState
    };
  }
}

// 사용 예시
const liveStreaming = new LiveStreamingSystem('ws://localhost:8080/stream');

// 스트리밍 시작
document.getElementById('startStream')?.addEventListener('click', () => {
  liveStreaming.startStreaming({
    video: {
      width: 1280,
      height: 720,
      frameRate: 30
    },
    audio: false // 비디오만 스트리밍
  });
});

// 스트리밍 중지
document.getElementById('stopStream')?.addEventListener('click', () => {
  liveStreaming.stopStreaming();
});
```

## 성능 최적화 및 모니터링

### 메모리 관리 최적화

```typescript
class MemoryOptimizer {
  private framePool: VideoFrame[] = [];
  private chunkPool: (EncodedVideoChunk | EncodedAudioChunk)[] = [];
  private maxPoolSize: number = 20;
  private memoryStats: MemoryStats;
  
  constructor() {
    this.memoryStats = new MemoryStats();
    this.startMemoryMonitoring();
  }
  
  public getFrame(): VideoFrame | null {
    if (this.framePool.length > 0) {
      const frame = this.framePool.pop()!;
      this.memoryStats.recordFrameReuse();
      return frame;
    }
    return null;
  }
  
  public releaseFrame(frame: VideoFrame): void {
    if (this.framePool.length < this.maxPoolSize) {
      this.framePool.push(frame);
      this.memoryStats.recordFramePooled();
    } else {
      frame.close();
      this.memoryStats.recordFrameDiscarded();
    }
  }
  
  public optimizeChunkUsage(chunk: EncodedVideoChunk | EncodedAudioChunk): void {
    // 청크 데이터 즉시 처리 후 해제
    this.processChunkImmediately(chunk);
  }
  
  private processChunkImmediately(chunk: EncodedVideoChunk | EncodedAudioChunk): void {
    // 청크를 즉시 처리하여 메모리 점유 시간 최소화
    const data = new ArrayBuffer(chunk.byteLength);
    chunk.copyTo(data);
    
    // 처리 로직
    this.handleChunkData(data, {
      type: chunk.type,
      timestamp: chunk.timestamp,
      duration: chunk.duration
    });
  }
  
  private handleChunkData(data: ArrayBuffer, metadata: any): void {
    // 실제 청크 데이터 처리 로직
    console.log('청크 처리:', metadata);
  }
  
  private startMemoryMonitoring(): void {
    setInterval(() => {
      this.memoryStats.updateStats();
      this.checkMemoryPressure();
    }, 5000); // 5초마다 모니터링
  }
  
  private checkMemoryPressure(): void {
    const stats = this.memoryStats.getCurrentStats();
    
    if (stats.usedJSHeapSize > 100 * 1024 * 1024) { // 100MB 초과
      console.warn('메모리 사용량 높음, 정리 수행');
      this.forceCleanup();
    }
  }
  
  private forceCleanup(): void {
    // 강제 정리
    this.framePool.forEach(frame => frame.close());
    this.framePool = [];
    
    this.chunkPool = [];
    
    // 가비지 컬렉션 유도
    if (window.gc) {
      window.gc();
    }
    
    this.memoryStats.recordCleanup();
  }
  
  public getMemoryReport(): object {
    return this.memoryStats.getReport();
  }
}

class MemoryStats {
  private frameReused: number = 0;
  private framePooled: number = 0;
  private frameDiscarded: number = 0;
  private cleanupCount: number = 0;
  private heapStats: any = {};
  
  recordFrameReuse(): void {
    this.frameReused++;
  }
  
  recordFramePooled(): void {
    this.framePooled++;
  }
  
  recordFrameDiscarded(): void {
    this.frameDiscarded++;
  }
  
  recordCleanup(): void {
    this.cleanupCount++;
  }
  
  updateStats(): void {
    if ('memory' in performance) {
      this.heapStats = {
        usedJSHeapSize: (performance as any).memory.usedJSHeapSize,
        totalJSHeapSize: (performance as any).memory.totalJSHeapSize,
        jsHeapSizeLimit: (performance as any).memory.jsHeapSizeLimit
      };
    }
  }
  
  getCurrentStats(): any {
    return this.heapStats;
  }
  
  getReport(): object {
    return {
      frames: {
        reused: this.frameReused,
        pooled: this.framePooled,
        discarded: this.frameDiscarded,
        reuseRate: this.frameReused / (this.frameReused + this.frameDiscarded)
      },
      memory: this.heapStats,
      cleanups: this.cleanupCount
    };
  }
}
```

### 성능 벤치마킹

```typescript
class PerformanceBenchmark {
  private metrics: Map<string, number[]> = new Map();
  private startTimes: Map<string, number> = new Map();
  
  startMeasure(operationName: string): void {
    this.startTimes.set(operationName, performance.now());
  }
  
  endMeasure(operationName: string): number {
    const startTime = this.startTimes.get(operationName);
    if (!startTime) {
      throw new Error(`측정이 시작되지 않음: ${operationName}`);
    }
    
    const duration = performance.now() - startTime;
    
    if (!this.metrics.has(operationName)) {
      this.metrics.set(operationName, []);
    }
    this.metrics.get(operationName)!.push(duration);
    
    this.startTimes.delete(operationName);
    return duration;
  }
  
  async benchmarkDecoding(): Promise<BenchmarkResult> {
    const testConfig: VideoDecoderConfig = {
      codec: 'avc1.42E01E',
      codedWidth: 1920,
      codedHeight: 1080
    };
    
    this.startMeasure('decoder_init');
    const decoder = new AdvancedVideoDecoder(testConfig);
    this.endMeasure('decoder_init');
    
    // 테스트 청크 생성 (실제로는 실제 비디오 데이터 필요)
    const testChunks = this.generateTestChunks();
    
    this.startMeasure('decoding_batch');
    
    for (const chunk of testChunks) {
      this.startMeasure('single_decode');
      decoder.decode(chunk);
      this.endMeasure('single_decode');
    }
    
    await decoder.flush();
    this.endMeasure('decoding_batch');
    
    decoder.close();
    
    return this.calculateResults('decoding');
  }
  
  async benchmarkEncoding(): Promise<BenchmarkResult> {
    const testConfig: VideoEncoderConfig = {
      codec: 'avc1.42E01E',
      width: 1920,
      height: 1080,
      bitrate: 5000000,
      framerate: 30
    };
    
    this.startMeasure('encoder_init');
    const encoder = new AdvancedVideoEncoder(testConfig);
    this.endMeasure('encoder_init');
    
    // 테스트 프레임 생성
    const testFrames = this.generateTestFrames();
    
    this.startMeasure('encoding_batch');
    
    for (const frame of testFrames) {
      this.startMeasure('single_encode');
      await encoder.encodeFrame(frame);
      this.endMeasure('single_encode');
    }
    
    await encoder.flush();
    this.endMeasure('encoding_batch');
    
    encoder.close();
    
    return this.calculateResults('encoding');
  }
  
  private generateTestChunks(): EncodedVideoChunk[] {
    // 테스트용 청크 생성
    const chunks: EncodedVideoChunk[] = [];
    
    for (let i = 0; i < 100; i++) {
      // 가짜 H.264 데이터 (실제 테스트에는 유효한 데이터 필요)
      const fakeData = new Uint8Array(1000);
      fakeData.fill(i % 256);
      
      chunks.push(new EncodedVideoChunk({
        type: i === 0 ? 'key' : 'delta',
        timestamp: i * 33333, // 30fps
        duration: 33333,
        data: fakeData
      }));
    }
    
    return chunks;
  }
  
  private generateTestFrames(): VideoFrame[] {
    const frames: VideoFrame[] = [];
    
    // 테스트용 캔버스 생성
    const canvas = document.createElement('canvas');
    canvas.width = 1920;
    canvas.height = 1080;
    const ctx = canvas.getContext('2d')!;
    
    for (let i = 0; i < 100; i++) {
      // 간단한 테스트 패턴 그리기
      ctx.fillStyle = `hsl(${i * 3.6}, 50%, 50%)`;
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      
      frames.push(new VideoFrame(canvas, {
        timestamp: i * 33333,
        duration: 33333
      }));
    }
    
    return frames;
  }
  
  private calculateResults(operation: string): BenchmarkResult {
    const initTimes = this.metrics.get(`${operation}_init`) || [];
    const singleTimes = this.metrics.get(`single_${operation.slice(0, -3)}`) || [];
    const batchTimes = this.metrics.get(`${operation}_batch`) || [];
    
    return {
      operation,
      initTime: {
        average: this.calculateAverage(initTimes),
        min: Math.min(...initTimes),
        max: Math.max(...initTimes)
      },
      singleOperationTime: {
        average: this.calculateAverage(singleTimes),
        min: Math.min(...singleTimes),
        max: Math.max(...singleTimes),
        p95: this.calculatePercentile(singleTimes, 0.95)
      },
      batchTime: {
        total: batchTimes[0] || 0,
        throughput: singleTimes.length / ((batchTimes[0] || 1) / 1000) // ops/sec
      }
    };
  }
  
  private calculateAverage(values: number[]): number {
    return values.length > 0 ? values.reduce((a, b) => a + b, 0) / values.length : 0;
  }
  
  private calculatePercentile(values: number[], percentile: number): number {
    const sorted = values.slice().sort((a, b) => a - b);
    const index = Math.ceil(sorted.length * percentile) - 1;
    return sorted[index] || 0;
  }
  
  public async runFullBenchmark(): Promise<{decoding: BenchmarkResult, encoding: BenchmarkResult}> {
    console.log('벤치마크 시작...');
    
    const decodingResults = await this.benchmarkDecoding();
    const encodingResults = await this.benchmarkEncoding();
    
    console.log('벤치마크 완료');
    
    return {
      decoding: decodingResults,
      encoding: encodingResults
    };
  }
}

interface BenchmarkResult {
  operation: string;
  initTime: {
    average: number;
    min: number;
    max: number;
  };
  singleOperationTime: {
    average: number;
    min: number;
    max: number;
    p95: number;
  };
  batchTime: {
    total: number;
    throughput: number;
  };
}

// 벤치마크 실행
const benchmark = new PerformanceBenchmark();
benchmark.runFullBenchmark().then(results => {
  console.log('벤치마크 결과:', results);
});
```

## 결론

WebCodec API와 MP4Box.js의 결합은 웹 플랫폼에서 **전문가급 미디어 처리**를 가능하게 만들었습니다:

### 핵심 장점

1. **하드웨어 가속**: 네이티브 수준의 인코딩/디코딩 성능
2. **저수준 제어**: 정밀한 미디어 처리 및 최적화 가능
3. **실시간 처리**: 라이브 스트리밍과 실시간 편집 지원
4. **표준 호환성**: MP4, WebM 등 표준 컨테이너 형식 완벽 지원

### 실무 적용 분야

- **비디오 편집 도구**: 브라우저 기반 전문 편집 소프트웨어
- **라이브 스트리밍**: 실시간 방송 및 화상회의 시스템  
- **미디어 변환**: 클라우드 기반 트랜스코딩 서비스
- **게임 스트리밍**: 고품질 게임 화면 전송

**웹 기술의 지속적인 발전과 함께, 이러한 API들은 미디어 처리의 새로운 표준으로 자리잡을 것입니다.** 하드웨어 가속과 표준화된 인터페이스를 통해 웹이 미디어 생산과 소비의 핵심 플랫폼으로 성장하고 있습니다.