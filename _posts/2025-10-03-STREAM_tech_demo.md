---
layout: post
title: "251003 STREAM 기술 데모"
comments: true
tags: [STREAM, AI, 댄스평가, 튜닝도구, 비디오인코딩, 쇼츠플레이어, 데모]
---

STREAM을 개발하면서 가끔 기능이 아닌 내부 기술을 예를 들면 튜닝이나 디버깅 하는 화면을 캡쳐해서 보여줄 일이 종종 있었습니다.
기능이 개발된후 내부 기술의 내용이 문서가 아닌 화면캡쳐 영상으로 남게 되면 더 전달력 있을때가 많았습니다.
STREAM은 멀티미디어가 핵심적인 기술 기능이라 더 그렇기도 했습니다.
광고소재 영상을 만들려고 각잡고 편집을 들어가는 그런 상황이 아니라도, 그냥 개발자들끼리 코드리뷰 하다가, 좀 중요한 얘기를 남겨두고 싶거나, 혹은 기능의 데모가 구현됬는데 향후 커뮤니케이션 할때 쓸려고 남겨두면 아주 요긴할때가 많았습니다.
저는 이럴때마다 주로 [OBS Studio](https://obsproject.com/) 를 실행해서 마이크소스 포함한 화면캡쳐를 하고, 공유할 필요가 생기면 youtube개인계정에 올려서 youtube 링크 공유하면 편했습니다.
그래서 공유가 필요했었던 기술 데모 영상을 3개를 준비했습니다.

<br/>
<br/>
<br/>

# 댄스 점수 튜닝 도구

- `내 댄스영상을 촬영하고 어떻게 틀린점을 찾고, 그 평가를 할 것인가에 대한 솔루션`이며, 이 솔루션을 튜닝하는 도구입니다.
- 배경음악으로 분석된 파트와 스텝을 먼저 나누어 준비하고, AI기술과 휴리스틱 알고리즘을 모두 사용해서 실시간으로 평가하고 그 기록을 남겨서 보관/공유를 할 수 있습니다.
- 이 기능의 동작은 모바일단말에서 수행되지만, 한프레임씩 실제 눈으로 비교해 가며, 알고리즘을 튜닝하는 단계를 거쳐야 했습니다.
- 더 자세한 내용은 나중에 별도 포스트로 또 다루겠습니다.

<div class="video-container">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/KbUeO6bEQhA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
</div>

<br/>
<br/>
<br/>

# 결과 영상 편집후 출력 인코딩 기능

- 내 댄스영상이 촬영되고 평가가 완료되어 에셋들이 준비되었다면,
- 이를 이용해서 간단한 설정만으로 유투브, 인스타그램, 틱톡에 공유할 만한 레이아웃배치, 효과등이 자동으로 포함된 영상을 제작할 수 있습니다.
- 영상제작은 유니티앱 내부에 오브젝트, 배경, 카메라, 오디오소스를 마치 드라마 세트장처럼 배치하고 로지컬 카메라로 촬영 하는 방식으로 구현되었습니다.
- 더 자세한 내용은 나중에 별도 포스트로 또 다루겠습니다.

<div class="video-container">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/9dTAT3rbBBg" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
</div>

<br/>
<br/>
<br/>

# 쇼츠 플레이어 웹버전

- 앞서 유니티로 앱을 만드는것이 궁극의 기술스텍이라고도 했지만,,, 막힐때도 있었습니다.
- 대표적으로 HLS 스트리밍 플레이어의 느린 시작시간 문제가 있었습니다.
- 이것은 숏폼영상 UX에서 치명적인 단점이었고, 이를 개선하기 위해 이 기능을 아예 웹앱으로 만들어서 유니티앱에서 웹뷰 임베딩 렌더링 하는 PoC을 했습니다.
- 결과는 매우 성공적이었고, UX나 기능적인 측면에서 유니티가 부족한 경우가 생기면 웹뷰 임베딩으로 부분적으로 선회하는 전략을 사용할 수 있다고 생각합니다.
- 현재는 유니티만 사용해서도 숏폼영상 플레이를 빠르게 할 수 있는 또다른 대안을 찾아서, 이제는 사용하지 않지만, 이 작업을 하면서 유니티앱에서 웹뷰 임베딩을 잘 하는 노하우를 축적했습니다.
- 아직까지는 유니티앱에서 이 기능 이외에 특별한 기능 한계를 더 발견한것은 없지만, 혹시 향후 리거시 기능이 웹앱으로 되어 있는 경우 개발시간 단축을 위해 임베딩 웹뷰를 사용할때가 생긴다면 잘 사용할 수 있을겁니다.

<div class="video-container">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/0r3mlmSTTbs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
</div>