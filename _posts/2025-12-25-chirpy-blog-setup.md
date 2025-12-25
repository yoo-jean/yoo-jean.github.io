---
title: Chirpy 블로그 세팅 기록 (GitHub Pages + Jekyll)
date: 2025-12-24 18:00:00 +0900
categories: [Blog, Jekyll]
tags: [jekyll, chirpy, github-pages, github-actions, blog]
---

## 개요
개발 공부 내용을 정리하고 기록하기 위해  
**GitHub Pages + Jekyll + Chirpy 테마**로 기술 블로그를 구축했다.

이 글은 블로그를 처음 세팅하면서 겪었던 문제들과  
그 해결 과정을 정리한 **첫 번째 기록**이다.

---

## 사용한 기술 스택
- GitHub Pages
- Jekyll
- Chirpy Theme
- GitHub Actions
- macOS (zsh)

---

## 1. Chirpy 테마를 선택한 이유
기술 블로그를 운영하면서 가장 중요하게 본 기준은 다음과 같다.

- 코드 블록 가독성
- 카테고리 / 태그 기반 글 정리
- 검색 기능
- 다크 모드 지원

Chirpy는 위 조건을 모두 만족했고,  
**기술 문서 및 트러블슈팅 기록에 최적화된 테마**라고 판단했다.

---

## 2. 로컬 개발 환경 세팅
macOS 기본 Ruby 대신 `rbenv`를 사용해 Ruby 버전을 관리했다.

### Ruby 설치
```bash
brew install rbenv ruby-build
rbenv install 3.2.2
rbenv global 3.2.2
```

### Jekyll 로컬 실행
```bash
bundle install
bundle exec jekyll serve -H 0.0.0.0
```

---

## 3. GitHub Pages 배포 방식
### User Pages 사용
레포 이름을 `username.github.io` 형식으로 생성하여  
**User Pages 방식**으로 배포했다.

```yml
# _config.yml
url: https://yoo-jean.github.io
baseurl: ""
```

---

## 4. 세팅 중 가장 많이 막혔던 문제들

### 1) Pages 주소에 `/repo-name`이 붙는 문제
- 원인: Project Pages로 인식됨
- 해결: 레포 이름을 `username.github.io`로 변경

### 2) `--- layout: home ---` 문구가 그대로 노출됨
- 원인: Pages Source가 branch 기반 배포
- 해결: Settings → Pages → Source를 GitHub Actions로 변경

### 3) GitHub Actions 빌드 실패
- 원인: Chirpy JS dist 파일 미존재 상태에서 HTML-Proofer 실행
- 해결: pages-deploy.yml에서 HTML-Proofer 단계 비활성화

---

## 5. 정리
이번 블로그 세팅을 통해 느낀 점은 다음과 같다.

- Jekyll + GitHub Pages는 자유도가 높은 대신 초기 세팅 난이도가 있다.
- GitHub Actions와 Pages의 역할을 이해하는 것이 중요하다.
- 한 번 세팅해두면 이후에는 글 작성에만 집중할 수 있다.

---

## 마치며
블로그 세팅 과정에서 겪은 시행착오 자체가  
가장 좋은 첫 글 소재가 되었다.

앞으로 이 블로그에는  
**Java, Spring, CS, 트러블슈팅 기록**을 중심으로 정리해 나갈 예정이다.
