---
layout: single
title: "Algolia 레코드 생성, 블로그 업데이트"
categories:
  - Miscellaneous
tags:
  - Algolia
  - bundle
---

# Algolia

- repo 페이지 Settings - Webhooks에서 Travis CI 추가 가능
  - repo에 `.travis.yml` 파일이 있어야 함
  - Travis CI 홈페이지에서 repo 추가
  - 이전에는 travis의 config 파일에서 매번 알고리아 레코드를 생성했었기 때문에 travis를 돌리는데 시간이 오래 걸렸음
- algolia record 추가 방법
  ```bash
  set ALGOLIA_API_KEY={admin api key}
  bundle exec jekyll algolia --config _config.yml
  ```

---

# 블로그 업데이트

- github pages 관련 설정은 repo - Settings - Pages로 바뀜

## `*.github.io`의 fork relationship을 끊고 싶을 때

1. `*.github.io`의 이름을 `*.github.i`로 바꿈
2. 깃헙 프로필 페이지에서 오른쪽 위 + 버튼 - Import repository로 `*.github.i` import하고 이름을 `*.github.io`로 새로 생성
3. `https://*.github.io/` 주소가 정상 작동하는지 확인

- 이때 `github.i` repo의 웹페이지도 `*.github.io/*.github.i` 경로에 남아있기 때문에 3번의 주소가 작동하는 것을 확인하면 `github.i` 레포 지우는게 좋음
