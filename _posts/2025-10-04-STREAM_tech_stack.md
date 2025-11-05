---
layout: post
title: "251004 STREAM 기술 스텍"
comments: true
tags: [STREAM, 기술스텍, Unity, AWS, 아키텍처, 인프라, 시스템다이어그램]
---

STREAM을 2년간 개발하면서 클라/서버 뿐 아니라, 운영툴, 통계툴 등 개발이 필요한 모든 분야를 바닥부터 만들었습니다.
1년동안 첫출시와 초기기능을 혼자서 개발하다가, 2024년 가을에 개발자J가 입사했고, 기획자Z(대표겸 공동창업자)와 3인이 만들어 나갔습니다.
클라이언트쪽 코드와 구현이 70%정도로 많긴하고, 서버, 운영툴, 통계툴은 코드의 절대량은 적지만, 종류와 구성이 다양합니다.
유니티가 중심이 되는 클라이언트쪽 기술은 사업시작전 준비된것도 좀 있었고, 고도화?된 부분도 있었는데, 그 이외의 부분은 처음엔 작고 쉽게 진행하다가 점차 솔루션이 자리를 잡아가게 되었습니다.
개발로 기능을 만들어가는 길이 원래 다양하지만, 아래 기술스텍들은 이렇게 극소규모로 개발할때, 클라이언트앱에 포커스를 하면서도 그외부분도 적당한 성능과 확장성까지 챙기는 솔루션이라 생각합니다. 

# 시스템 다이어그램 
- ![](https://i.imgur.com/8jOCaXI.png)
    
# 사용언어, 프레임워크
- 클라이언트
    - Unity C# : 82%
    - Android kotlin : 5%
    - iOS objectiveC : 5%
    - react+TypeScript+TailWindCSS : 5%
    - shell script : 3%
- 서버
    - python : 90%
    - shell script : 10%
    - ffmpeg
    - openCV
    - tflite
    - FastAPI
    - gradio
    - aws-cli
    - rclone
    - firebase

# 클라이언트
- Unity
    - 90% 이상 대부분의 기능 구현
    - 그래픽스, 멀티미디어, AI, GUI
- 외부 Unity 라이브러리
    - [AvProVideo](https://github.com/RenderHeads/UnityPlugin-AVProVideo)
        - 4OS(=iOS, Android, Windows, MacOS) 지원
        - 기본 mp4, hls 동영상 플레이어
        - 스크러빙 seek 지원
        - RenderTexture 출력 지원
        - AudioOutput 출력 지원
    - [AvProCapture](https://github.com/RenderHeads/UnityPlugin-AVProMovieCapture)
        - 4OS 지원
        - mp4(h264+aac) 인코딩
        - RenderTexture 입력 지원
        - AudioListener 입력 지원
    - [VideoKit](https://github.com/videokit-ai/videokit)
        - 4OS 지원
        - 카메라 프리뷰 렌더링
    - [TfliteUnity](https://github.com/asus4/tf-lite-unity-sample)
        - 4OS 지원
        - 다수의 tflite 모델 호스팅
        - OpenPose 추론 사용
    - [UniWebView](https://docs.uniwebview.com/)
        - 4OS 지원
        - 유니티앱 내부에서 WebView 렌더링
        - 렌더링 측면에서 많은 제약사항 있음.
        - 유니티와 Web간의 텍스트기반 동기/비동기 통신 지원.
- 자체제작 Unity 라이브러리
    - ZAdvCam
        - iOS, Android 의 카메라 API의 고급기능을 유니티앱으로 연동.
        - 하지만 현재 기술한계로 iOS만 개발됨.
        - 프로세스
            - Unity 에서 RenderTexture를 iOS로 전달
            - iOS에서 MetalTexture로 변환
            - AvFoundation Camera API 로 카메라를 탐색하고, 초기화, 비디오 샘플 수집 시작
            - 유니티앱에서 iOS로 통신하여 AvFoundataion Camera API를 상세하게 제어.
            - Metal API 로 비디오 샘플을 MetalTexture에 렌더링
            - 연결된 Unity RenderTexture에 렌더링되고 유니티앱에서 활용가능.
    - [ZCodeBehind](https://github.com/StreamStudioZebra/ZCodeBehind)
        - WPF, ASP.NET의 CodeBehind와 동일한 기능을 Unity의 UGUI에 대해 구현
        - Unity씬에서 이름을 붙이면 자동으로 C#코드에서 동일한 이름의 바인딩 코드들이 생성됨
        - 유니티씬과 C# 코드에서 작성된 바인딩 코드가 없어져서, 리펙토링등 구조변경이 쉬워지고, 바인딩 관련 버그도 원천봉쇄됨.
    - ZAos
        - Android 용 범용 플러그인
        - 카카오톡 네이티브 로그인
        - 시스템 상태바, 앱바제어
        - mp4 동영상 파일의 메타정보 조회
    - ZIosPlugin
        - iOS용 범용 플러그인
        - mp4 동영상 파일의 메타정보 조회
        - 타겟 광고를 위한 사용자 행동 추적동의 권한 제어
- React 기반 웹앱
    - 유니티에서 웹뷰를 통한 확장 혹은 코드푸시를 이용하기 위한 기능.
    - 사용기술 : React, TypeScript, TailWindCss
    - TikTok뷰를 대체하기 위한 프로토타입 제작

# 서버
- FireBase
    - FireStore
        - Unity 앱, 각종 서버, 운영툴 에서 사용하는 DB.
        - NOSQL이지만 RDB처럼 컬럼을 정규화 해서 사용함.
        - 컬럼명 실수를 줄이기 위해서 ORM 유틸리티 제작하여 사용.
        - 속도를 높이기 위해 각 제품들 내부에서 캐시 사용.
    - GA
        - 클라이언트 내부의 화면이동, 유저의 대부분의 액션 기록
    - Messaging
        - 푸시메시지 발송
            - 유투브 영상 다운로드 완료 알림
            - Danmix 배경제거 완료 알림
            - 좋아요, 댓글, 등 소셜 알림
    - CrashReport
        - iOS, Android 크래시 로그 수집
        - 크래시 콜스텍, 일반로그 모두 수집
- AWS
    - S3
        - 동영상, 이미지
        - 인앱 웹뷰용 static react CSR website
    - CloudFront
        - 동영상, 이미지 CDN
    - EC2
        - 미디어 서버
            - python
            - ffmpeg
            - openCV
        - CMS
            - python
            - gradio
            - ffmpeg
            - openCV
        - 통계 서버
            - python
        - gitea
    - Lambda
        - 웹 API 서버
        - python
        - FastAPI
        - docker
    - Batch
        - Danmix 배경지우기 분산처리
        - rembg
        - ffmpeg
        - openCV
        - docker
- 가정용 서버
    - 유투브 영상 다운로드 서버
    - python
    - ffmpeg
- retool
    - 각종 지표, 통계
    - 사용자 활동 추적, 관리
    - javascript plugin
- tinyurl
    - 랜플 단축 url
    - 유저 영상 단축 url