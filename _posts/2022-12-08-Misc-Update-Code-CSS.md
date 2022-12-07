---
layout: single
title: "mmistakes inline, block 코드 style 수정"
categories:
  - Miscellaneous
tags:
  - CSS
  - Blogging
  - scss
  - selector
---

이전에 코드 블럭과 인라인의 글자 크기를 조절한 적이 있었는데 inline code는 제대로 조절이 되지 않았다. 이번에 백준 1513번 문제 풀이를 올리면서 인라인 코드 스타일의 가독성이 너무 떨어져서 바꾸기로 마음먹었다.

## 코드 스타일 관련 파일들

- `assets/css/main.scss` : 이전에 코드 css style을 바꾸기 위해 수정했던 파일  
  아래와 같은 selector를 지정해서 바꿈

  - `.highlight` : code block의 클래스
  - `.language-plaintext` : inline code의 클래스

- `_sass/minimal-mistakes/_page.scss` : 페이지 내부의 element들에 대한 css
- `_sass/minimal-mistakes/_syntax.scss` : 코드 하이라이팅에 관련된 css
- `_sass/minimal-mistakes/_variables.scss` : `/minimal-mistakes` 내부에서 사용되는 scss 변수들 선언

## 문제점

- 로컬에서 `main.scss`의 `.language-plaintext`를 계속 바꿔봐도 inline code의 스타일이 바뀌지 않았음
- 같은 파일의 `.highlight`는 바꾸면 적용되었음

![updatecss1](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/updatecss1.png)

- `.highlight`는 코드 블럭의 조상 클래스 중 하나였고, 이후 자식 클래스들부터 `#include`와 같은 하이라이트 클래스까지 모두 `em`으로 font-size를 조절했기 때문에 `.highlight`의 font-size를 조절해도 적용이 됨

![updatecss2](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/updatecss2.png)

- `.language-plaintext`는 `.page__content :not(pre)>code`보다 specificity가 낮고, 둘 다 같은 대상에게 적용되는 것이기 때문에 `.language-plaintext`는 overwrite됨
  - `.language-plaintext` : 1 class -> 010
  - `.page__content :not(pre)>code` : 1 class + 2 elements -> 012

## 해결 방법

- `.highlight`의 font-size는 따로 선언한 변수인 `$type-size-4-5`로 설정되어 있었음
  - inheritance를 위해서 `$type-size-4-5`의 값을 `14px` -> `1.12em`으로 바꿈
- `.language-plaintext`는 selector의 우선순위를 올려주거나, 우선순위가 더 높은 `_syntax.scss`의 `.page__content :not(pre)>code`를 가리키는 rule를 수정하면 됨
  - 위 사진에서 볼 수 있듯이 inline code는 클래스를 두 개 가지기 때문에 `.language-plaintext.highlighter-rouge`로 우선순위를 020으로 높여줄 수 있음
  - 두 rule이 동일한 대상을 가리키기 때문에 가독성을 위해서 `.language-plaintext`를 지우고 `_syntax.scss`를 수정하는 방법을 선택함
