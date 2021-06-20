---
layout: single
title: "Markdown 문법 몇 가지"
categories:
  - Misc
tags:
  - github-pages
  - Markdown
---
## 주석에도 list를 사용할 수 있음

---
Markdown  
```
> 주석 안에서
> - list 1
> - list 2
> 	- list 2-1
> 	- list 2-2  
>		content in list 2-2
>
>	content in list 2
> - list 3  
> 	content in list 3
>
> 와 같이 list를 사용 가능
```

---
Markup  
> 주석 안에서
> - list 1
> - list 2
> 	- list 2-1
> 	- list 2-2  
>		content in list 2-2
>
>	content in list 2
> - list 3  
> 	content in list 3
>
> 와 같이 list를 사용 가능

## backtick을 이용한 코드 표현
본문에서는 자유롭게 사용할 수 있고, title에서도 제한없이 사용 가능

하지만 tags에는 사용할 수 없음  
(사용하면 header가 인식되지 않음)

## 표 사용할 때 주의할 점
보통 헤더나 리스트 같은 block element는 본문과 이어서 써도 잘 인식되는데, 표는 이어서 쓰면 가끔 인식되지 않을 때가 있기 때문에 웬만하면 위아래로 한 줄씩 띄우고 쓰는게 좋음

## 새창으로 링크
markdown의 링크(`[content](link)`)는 현재 창에서 링크를 열기 때문에 뒤에 `{:target: "_blank"}`를 사용했었음  
블로그에 올라갔을 때는 정상적으로 작동하지만, `_blank` 때문에 편집할 때는 뒤에 모든 내용들이 italicize되어서 불편했음

`<a href="link" target="_blank">필요하면 더 공부하기</a>`와 같이 전부 `<a>` 태그로 링크하는게 더 편함  
텍스트 스타일은 markdown의 링크와 같게 나옴!

## 구분선
`---`으로 구분선을 나타낼 수 있는데 표와 마찬가지로 이어서 쓰면 인식되지 않을 때가 있음
- n 번째 줄에서 텍스트를 쓰고 공백 두 개+엔터로 끝낸 다음 n+1 번째 줄로 넘어와서 `---`을 쓰면 텍스트로 인식되어서 em dash가 입력됨
- 그 외 텍스트를 입력하다가 공백 두 개 없이 다음 줄에 `---`을 쓰거나,  
	`---`을 쓰고 공백 없이 다음 줄에 텍스트를 입력하면 정상적으로 구분선이 인식됨