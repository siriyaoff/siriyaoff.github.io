---
layout: single
title: "Modify excerpt in Jekyll"
categories:
  - Dev
tags:
  - Blogging
  - Jekyll
  - Page variable
  - Excerpt
---

## Excerpt

![excerpt explanation 1](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/jekyll-excerpt-1.png)

- Jekyll에서 사용되는 블로그의 내용 미리보기이다. 기존에는 첫 h2만 보여서 수정을 하게 되었고, 위 사진은 이후의 내용들도 나오게 한 상태이다.

## In Jekyll

- 위 사진에서 볼 수 있듯이, `archive__item-exceprt` 클래스인 `<p>` element에 담긴다.
- 아래 코드는 `_includes/archive-single.html` 의 코드의 일부분이다.

  ```html
  {% raw %}
  <div class="{{ include.type | default: 'list' }}__item">
    <article class="archive__item" itemscope itemtype="https://schema.org/CreativeWork">
      {% if include.type == "grid" and teaser %}
      <div class="archive__item-teaser">
        <img src="{{ teaser | relative_url }}" alt="" />
      </div>
      {% endif %}
      <h2 class="archive__item-title no_toc" itemprop="headline">
        {% if post.link %}
        <a href="{{ post.link }}">{{ title }}</a> <a href="{{ post.url | relative_url }}" rel="permalink"><i class="fas fa-link" aria-hidden="true" title="permalink"></i><span class="sr-only">Permalink</span></a>
        {% else %}
        <a href="{{ post.url | relative_url }}" rel="permalink">{{ title }}</a>
        {% endif %}
      </h2>
      {% include page__meta.html type=include.type %} {% if post.excerpt %}
      <p class="archive__item-excerpt" itemprop="description">{{ post.excerpt | markdownify | strip_html | truncate: 160 }}</p>
      {% endif %}
    </article>
  </div>
  {% endraw %}
  ```

  - archive-single이 나열되는 포스트들의 컴포넌트 하나하나를 지칭하는 단위인 것을 알 수 있다.
  - `{% raw %}{{ post.excerpt | markdownify | strip_html | truncate: 160 }}{% endraw %}`에서 `markdownify`, `strip_html`, `truncate`는 `post.excerpt`에 적용하는 필터들이다. 각각 이름에서 기능을 유추할 수 있다.
    - `excerpt`라는 필드가 존재하는 것을 알 수 있다.

- 아래 코드는 `docs/_docs/01-quick-start-guide.md`의 front matter 부분이다.

  ```markdown
  ---
  title: "Quick-Start Guide"
  permalink: /docs/quick-start-guide/
  excerpt: "How to quickly install and setup Minimal Mistakes for use with GitHub Pages."
  last_modified_at: 2021-06-07T08:48:05-04:00
  redirect_from:
    - /theme-setup/
  toc: true
  ---
  ```

  - page variable인 `excerpt`를 선언해서 직접 설정도 가능하다.
  - 선언되지 않을 경우 `_config.yml`의 `excerpt_separator`에 의해 구분되는 첫 번째 블럭을 가져온다.
    - 기본으로는 `\n\n`이 설정되어 있어, prettier를 적용해놓았을 경우 자동으로 헤더 다음에 `\n\n`이 삽입되기 때문에 헤더만 excerpt에 들어간다.

## What changed

- seperator를 `\n\n\n`으로 바꾸어 헤더 이후의 내용도 모두 들어가도록 설정해놓았다.
  - `archive-single.html`에서 볼 수 있듯이 `truncate` 필터가 있기 때문에 본문이 모두 exceprt에 들어갔다가 truncate 되어서 160자만 추출된다.
  - 덕분에 빌드 속도가 10초에서 30초 정도로 느려졌다. 포스트가 쌓일수록 더 늦어질 것인데, 효율적인 seperator가 생각나면 다시 바꿔야 할 것 같다.
