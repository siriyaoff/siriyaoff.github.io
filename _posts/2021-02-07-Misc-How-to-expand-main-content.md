---
layout: single
title: "Mmistakes 본문 영역 너비, 글자 크기 수정"
categories:
  - Miscellaneous
tags:
  - Blogging
---

- HTMLnote같은 포스트들은 텍스트 양이 상당히 많은데 본문 영역 너비가 좁아서 가독성이 떨어진다는 느낌을 받았다.
- 글자 크기도 너무 커서 짧은 문장도 줄바꿈이 자주 일어났다.

## 본문 영역 너비

- 페이지의 YAML Front Matter(`---`으로 감싸진 부분)에 `classes: wide`를 추가하면 넓어진다.  
  `docs/_docs/10-layouts.md`에 나와있다.

  >     ### Wide page
  >
  >     To expand the main content to the right, filling the space of what is normally occupied by the table of contents. Add the following to a post or page's YAML Front Matter:
  >
  >     ```yaml
  >     classes: wide
  >     ```
  >
  >     **Note:** If the page contains a table of contents, it will no longer appear to the right. Instead it will be forced into the main content container directly following the page's title.
  >     {: .notice--info}

- `_layouts/single.html`의 YAML Front Matter에 추가해놓으면 포스트를 쓸 때마다 추가하지 않아도 된다.

## 글자 크기 수정

- <https://devinlife.com/howto%20github%20pages/github-pages-settings/>{:target="\_blank"}를 참고했다.

- mmistakes repo의 issue에 [관련된 이슈](https://github.com/mmistakes/minimal-mistakes/discussions/1219){:target="\_blank"}가 있었다.

```css
html {
  font-size: 12px; // originally 16px
  @include breakpoint($medium) {
    font-size: 14px; // originally 18px
  }

  @include breakpoint($large) {
    font-size: 16px; // originally 20px
  }

  @include breakpoint($x-large) {
    font-size: 18px; // originally 22px
  }
}
```

- 위 코드를 `assets/css/main.scss`의 `@import ...` 밑에 추가하면 된다.

- breakpoint는 모바일, 태블릿, 데스크탑 등의 환경에 대응하기 위해 만들어진 것이다. HTML의 responsive image와 비슷한 원리이다. 브라우저의 크기에 따라서 페이지의 구조나 스타일을 바꿔서 보여주는 것이다. `_sass/_variables.scss`에서 기준이 되는 너비를 알 수 있다.

- `classes: wide`를 추가하지 않고 글자 크기도 원래대로 두면, 블로그를 전체화면에서 반쪽으로 바꾸었을 때 페이지가 wide로 바뀌면서 글자 크기가 줄어든다. 하지만 글자크기를 모두 16px로 고정하고 포스트들의 main content 너비도 wide로 해놓으니까 화면을 반쪽으로 전환해도 이펙트가 없다. 이점은 좀 아쉽다.
