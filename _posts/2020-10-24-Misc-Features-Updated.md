---
layout: single
title: "풍성해진 블로그"
categories:
  - Miscellaneous
tags:
  - Blogging
---
<div markdown="1" style="font-size:18px;font-family:'Consolas', 맑은 고딕;">

[추가하고 싶었던 것들](https://siriyaoff.github.io/miscellaneous/Misc-Features-To-Add/){: target="_blank"}
이 있었는데 어쩌다보니 다 추가했다!!!!

## 검색 기능
상단 카테고리 옆에 검색 표시가 생겼다. 조금 있으면 없는게 어색해지겠지만 아직 있는게 신기하다. 클릭하면 내 포스트들의 제목, 본문을 검색할 수 있다. 검색 엔진은 algolia를 이용했고,
<https://xinfolab.github.io/blog/blog-maker-4/>{:target="_blank"}
에서 도움을 많이 받았다. 도중에 travis ci를 이용해봤는데 이미지가 되게 친근해서 첫인상이 좋았다. github repo에서 빌드하는 것을 실시간으로 연동해서 보여주는 사이트인 것같다.
algolia에 가입하고 발급받은 내 key를 이용해서 `_config.yml`를 수정하고 `Gemfile`, `Rakefile`, `.travis.yml`을 고치면 된다. `.travis.yml`은 `Gemfile` 경로를 `docs/Gemfile`에서 `Gemfile`로 고쳐야 하는데 업데이트하면서 mmistakes에서 Gemfile의 경로가 바뀐 것 같다. mmistakes 홈페이지에서 초기 버전을 받았을 때 `docs`안에서 `Gemfile`을 본 것 같다.(내 블로그에는 docs 경로 자체가 없다)

## 댓글 기능
disqus가입하고 short name만 `_config.yml`에 적어주면 된다. 검색기능보다 훨씬 간단하게 추가했다.
<https://devinlife.com/howto%20github%20pages/blog-disqus/>{:target="_blank"}
에서 도움을 많이 받았다.

둘다 ID : hwy16016@gmail.com

## 추가하고 싶은 기능
1. 구글에 노출
- Google Search Console 등록하고 인증은 했다. 근데 sitemap.xml이 제대로 인식되지 않는다.

</div>