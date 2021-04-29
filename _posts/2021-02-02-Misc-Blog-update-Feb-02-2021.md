---
layout: single
title: "블로그 업데이트하기"
categories:
  - Miscellaneous
tags:
  - Blogging
---

포스팅 해야하는 것(20.11.25)
---
- ~~2261 질문 올려놓은거 답변 오면(탐색 이중으로 하는건 왜 시간초과 나는지, 중복된점 처리 관련)~~  
- parametric search 하나 풀어보고 2110에 쓴게 param search 맞는지  
- ~~1697(+-1, *2 문제) 질문 올려놓은거 답변 오면(분할정복 3에 대해서 적용되는거 확인, 증명)~~  

- parametric search는 문제 몇개 풀어보고 포스팅 해야할듯


남은 포스팅
---
- parametric search 하나 풀어보고 2110에 쓴게 param search 맞는지  
- vi v(10)으로 선언하고 v[10]이 원래 호출이 되나??(세그폴트 뜨지않나?)//boj 1806
- boj 1324 풀고 정리
- HTML 노트 정리한거 보완한다음에 포스팅


블로그 바꿔야하는 것
---
- 코드블럭 글자 크기 키우기
- 글 footer부분에 등록한 날짜/수정한 날짜 띄우기
- top버튼 만들기
- 블로그 본문 블럭 너비 수정(모바일도 고려해야할듯, disqus 댓글 참고)


블로그 바꾼 내용
---
- 이제 포스트마다 `<div markdown="1" style="font-size: 16px;font-family: ...">`안에 쓰지 않아도됨
	- [https://evenharder.github.io/blog/jekyll-change-fonts/](https://evenharder.github.io/blog/jekyll-change-fonts/) 참고해서 `assets/css/main.scss` 바꿈
	- 그전에 시도했던 것들
		- `_sass/minimal-mistakes/_variables.scss`에서 `doc-font-size` 바꾸려 했는데 스택오버플로우 찾아본 결과 빌드 시간 아까워서 하지않음(travis.ci 등록해놔서 빌드가 계속 오래걸리는데 travis도 등록 해제 해야 할 듯, 이거도 주요 기능이 뭔지 알아봐야함)
		- `_layouts/single.html`에서 id가 `page--content`인 `<div>`가 있길래 거기 스타일 지정해봄
- BOJ 문제 풀이 포스팅들 목차 수정(문제풀이를 h2로 해놔서 그런지 블로그에서 포스트 미리보기에 문제풀이만 나옴 -> 문제 설명을 바로 적고 밑에 문제풀이 적는 구조로)