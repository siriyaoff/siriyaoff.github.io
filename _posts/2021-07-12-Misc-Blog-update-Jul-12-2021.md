---
layout: single
title: "블로그 업데이트(21.07.12)"
categories:
  - Miscellaneous
tags:
  - Blogging
  - Mathjax
  - Jekyll
---

## 추가해야 하는 포스팅
- parametric search 하나 풀어보고 2110에 쓴게 param search 맞는지  
- vi v(10)으로 선언하고 v[10]이 원래 호출이 되나??(세그폴트 뜨지않나?)//boj 1806
- boj 1324 풀고 정리
- 하루3분 네트워크 pdf 올린거 블로그에 iframe으로 올리기
- boj 15902 푼거 포스팅

## 블로그 바꾼 내용
- mathjax 추가  
	<a href="https://torbjorn.tistory.com/192" target="_blank">https://torbjorn.tistory.com/192</a> 참고함
	
	```html
	<!DOCTYPE html>
	<html>
	<head>
	  <meta charset="utf-8">
	  <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
	  <script id="MathJax-script" async
			  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
	  </script>
	</head>
	<body>
	<p>
	  When \(a \ne 0\), there are two solutions to \(ax^2 +bx + c = 0\) and they are
	  \[x = {-b \pm \sqrt{b^2-4ac} \over 2a}.\]
	</p>
	</body>
	</html>
	```
	위 페이지로 수식 미리보기 가능

	<a href="https://en.wikibooks.org/wiki/LaTeX/Mathematics" target="_blank">문법</a>은 차차 공부해야 함
- 포스트 밑에 읽는데 필요한 시간을 게시 날짜로 바꿈  
	<a href="https://github.com/devinlife/devinlife.github.io/commit/c6a8fe5a2f2a6f208b4ad90528074842e5c3ee66#commitcomment-50500658" target="_blank">helloHaneul님 comment</a> 참고함
	- default에 `read_time: true` 대신 `show_date: true`로 바꾸면 됨
	- `_config.yml` 건드릴 때 tab으로 인덴팅하면 에러남!!
- author, site name 등등 수정  
	<a href="https://mmistakes.github.io/minimal-mistakes/docs/configuration/" target="_blank">mmistakes docs</a> 참고함
- back to top 버튼 만듦
	- <a href="https://github.com/mmistakes/minimal-mistakes/issues/1731" target="_blank">깃헙 issue</a> 참고함  
		핵심은 아래 코드를 `<body>` 안에 추가하는 거였는데, 간단하게 `#site-nav`로 링크만 걸어서 만들 수 있다는게 신기했음  
		```html
		<aside class="sidebar__top">
		<a href="#site-nav"> <i class="fas fa-angle-double-up fa-2x"></i></a>
		</aside>
		```
		`sidebar__top` 클래스에 대해 아래 CSS를 적용하면 끝남:  
		```css
		.sidebar__top {
		  position: fixed;
		  bottom: 1.5em;
		  right: 2em;
		  z-index: 10;
		}
		```
		
		<a href="https://github.com/vfeskov/vanilla-back-to-top" target="_blank">JS로 만드는 back-to-top</a>처럼 다양한 효과(스크롤을 내릴 때 나타남 등등)를 주지는 못함
	
## 남은 이슈들(블로그 관련)
- author, nav 글자 크기
- 블로그 로고, 프로필 이미지 추가