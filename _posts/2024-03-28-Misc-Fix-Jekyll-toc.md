---
layout: single
title: "Jekyll toc 설정 방법 및 목차가 깨질 때 해결 방법"
categories:
  - Misc
tags:
  - Jekyll
  - Markdown
---

## Jekyll 블로그 TOC(Table of Contents) 설정 방법
- Jekyll 블로그 포스트할 때 YAML front formatter에 toc 관련 태그들을 추가해서 toc 설정 가능  
  ```yaml
  toc: true
  toc_label: "Index"
  toc_icon: "columns"
  ```

## 문제점
- 아래 사진과 같이 toc가 깨지는 경우가 발생했음  
  ![Jekyll-toc-broken](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/Jekyll-toc-broken.png)
  - 일부 글에서만 깨짐
- 가이드에서 아래와 같은 설명을 찾을 수 있었음:
  ```markdown
  Good headings:

  # Heading
  ## Heading
  ### Heading
  ### Heading
  # Heading
  ## Heading

  Bad headings:

  # Heading
  ### Heading (skipped H2)
  ##### Heading (skipped H4)
  ```
- 일부 글에서 `#### Example`이라는 헤더를 헤더 순서가 아닌 스타일로 사용하면서 h2 뒤에 h4가 바로 오는 부분이 존재했음
  - semantic tag의 중요성을 다시 깨닫게 됨

## 해결방법
- `#### Example` 헤더를 `**Example**  `으로 변경해서 헤더가 한 번에 2단계 이상 변화하는 부분을 모두 없앰

## 이후
- 가이드를 보면서 `{% raw %}{% capture notice-text %}...{% endcapture %}{% endraw %}`와 같은 스타일도 있다는 것을 알게 됨
- 노션으로 Markdown 코드를 옮길 때 1600-2000줄 정도가 한 번에 옮길 수 있는 최대 용량임