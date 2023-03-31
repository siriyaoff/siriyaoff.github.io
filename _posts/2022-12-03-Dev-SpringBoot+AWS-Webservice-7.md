---
layout: single
title: "Spring Boot와 AWS로 혼자 구현하는 웹 서비스 7 - 1인 개발 시 팁(Ch.11)"
categories:
  - Dev
tags:
  - CDN
---

### 댓글

- Disqus
  - 댓글을 작성한 사람에 대한 기능이 많음
- LiveRe
  - 국내 서비스
  - SNS 계정과 연동 가능
- Utterances
  - 깃헙 댓글
  - 마크다운 사용 가능

### 외부 서비스 연동

- Zapier
  - Trigger로 이벤트 발생 조건 생성 후 Action으로 작업 결정
  - 월 100건 무료 지원
  - SNS와의 연동으로 게시글 알림 등 가능
- IFTTT
  - Zapier와 비슷
  - 블로그에 글 작성하면 트위터에 링크 자동 공유 등

### 방문자 분석

- Google Analytics
  - 구글 광고를 사용하면 연동됨

## CDN

- Content Delivery Network : 정적 콘텐츠를 로드할 때 가장 가까운 서버에서 가져가도록 지원하는 서비스
  - 트래픽 분산 가능
- Cloudflare
  - 정적 파일 캐싱하는 서비스는 무료로 사용 가능

## 이메일 마케팅

- Mailchimp
  - 2000명에게 월 12000개까지 무료
  - 뉴스레터 서비스에 대한 구독자 관리, A/B 테스트까지 지원

## 1인 개발 팁

- MVP를 먼저 만들고 부가적인 기능 구현
- 혼자 개발하는게 개발이 빨리 완료될 때가 많음(의견 충돌 등 때문)
