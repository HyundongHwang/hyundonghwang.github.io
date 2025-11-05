# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

이 프로젝트는 Jekyll 기반의 개인 기술 블로그입니다. GitHub Pages에서 호스팅되며, STREAM 앱 개발 관련 기술 글과 다양한 개발 경험을 공유합니다.

## 빌드 및 로컬 개발

### Jekyll 서버 실행
```bash
# 로컬 개발 서버 시작
bundle exec jekyll serve

# 드래프트 포함하여 실행
bundle exec jekyll serve --drafts

# 라이브 리로드 (변경사항 자동 반영)
bundle exec jekyll serve --livereload
```

### 빌드
```bash
# 사이트 빌드 (결과물은 _site 디렉토리에 생성)
bundle exec jekyll build
```

## 아키텍처 및 구조

### 템플릿 시스템
- **Lanyon 테마** 기반의 Jekyll 사이트
- `_layouts/`: 기본 레이아웃 템플릿
  - `default.html`: 기본 레이아웃 (사이드바, 헤더 포함)
  - `post.html`: 블로그 글 레이아웃
  - `page.html`: 정적 페이지 레이아웃
- `_includes/`: 재사용 가능한 컴포넌트
  - `sidebar.html`: 네비게이션 사이드바
  - `head.html`: HTML 헤드 섹션
  - `google_analytics.html`: 구글 애널리틱스 스크립트
  - `comments.html`: Disqus 댓글 시스템

### 콘텐츠 구조
- `_posts/`: 마크다운 블로그 글 (YYYY-MM-DD-제목.md 형식)
- `_config.yml`: Jekyll 설정 파일 (사이트 메타데이터, 플러그인 설정)
- `index.html`: 메인 페이지 (페이지네이션 포함)
- `archive.md`, `tags.md`: 글 목록 및 태그 페이지

### 블로그 글 작성 규칙
- Front Matter 필수 필드:
  ```yaml
  ---
  layout: post
  title: "글 제목"
  date: YYYY-MM-DD
  categories: [카테고리1, 카테고리2]
  tags: [태그1, 태그2, 태그3]
  ---
  ```
- 파일명: `YYYY-MM-DD-영문제목.md` 형식
- 최근 글들은 STREAM 앱 개발 관련 기술 이슈를 다룸

### 스타일링
- CSS 파일 위치: `public/css/`
  - `lanyon.css`: 메인 테마 스타일
  - `poole.css`: 기본 타이포그래피
  - `syntax.css`: 코드 하이라이팅
- D2Coding 폰트 사용 (`font-d2coding/`)

### 기능
- **페이지네이션**: 메인 페이지에서 20개 글씩 표시
- **사이드바 토글**: 모바일 친화적 네비게이션
- **Disqus 댓글**: 각 글에 댓글 기능
- **구글 애널리틱스**: 방문자 트래킹
- **RSS/Atom 피드**: `feed.xml`, `atom.xml`
- **Sitemap**: SEO를 위한 `sitemap.xml`

## 개발 시 주의사항

### 새 글 작성시
- 파일명과 front matter의 date가 일치해야 함
- categories와 tags는 기존 것과 일관성 유지
- 최근 글들은 주로 Unity, 크로스플랫폼, 멀티미디어 관련 기술 주제

### 디자인 수정시
- Lanyon 테마의 기본 구조 유지
- 모바일 반응형 고려
- D2Coding 폰트 활용 (한글 개발 블로그 특성)

### 배포
- GitHub Pages 자동 빌드 (push시 자동 배포)
- `master` 브랜치가 배포 브랜치