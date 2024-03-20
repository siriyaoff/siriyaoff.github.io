---
layout: single
title: "HTMLnote1: HTML Basics"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - HTML
  - MDN
---
# HTML Basics
HTML : HyperText Markup Language  
UX : User eXperience  
Google search engine treats a hyphen as a word-separator but does not for an underscore.  
-> 파일 이름 구분은 -으로

## Basic structure of an website  
web-project -> test-site -> (`Index.html`, `/images`, `/styles`, `/scripts`)
- `/styles` : CSS codes about text, bgcolor ...
- `/scripts` : JavaScripts for interactive functions  
※ Windows에서는 path에 `\`를 사용하지만 html에서는 `/` 사용

tags ∈ elements ∈ DOM(Document Object Model)

## Anatomy of an HTML element
![Anatomy of an HTML element](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/HTML_basics/grumpy-cat-small.png)
- opening tag는 attribute를 포함할 수도 있음
- Attribute rules
	- separator : `' '`
	- assignment : `=`
	- value format : `"VALUE"` or `'VALUE'`  
	->`att1="val1" att2="val2"`
- Nesting : to put elements inside other elements
	- `<p> My cat is <strong>very</strong> grumpy.</p>`
- Empty elements(=Void elements)
	- no closing tag, no inner content(does not wrap content to affect it)
	- `<img src="images/firefox-icon.png" alt="My test image">`
	- 정확하게는 child nodes(nested elements, text nodes...)를 가질 수 없는 elements들(closing tag가 없는 element임)

>  but they aren't handy on their own.

## Anatomy of an HTML document  
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title>My test page</title>
	</head>
	<body>
		<img src="images/firefox-icon.png" alt="My test image">
	</body>
</html>
```
- `<!DOCTYPE html>` : 파일의 HTML버전을 명시(HTML은 버전별로 지원하는 태그가 다르기 때문에 미리 HTML버전을 명시해서 브라우저가 올바르게 표시하게 만듦)
- `<html></html>` : wraps all the content on the entire page
	- root element라고도 함
- `<head></head>` : 브라우저가 표시하는 페이지에는 나타나지 않지만 넣어야 하는 것들(페이지 설명, CSS, keyword ...)을 보관하는 컨테이너
	- `<meta charset="utf-8">` : sets the character set(utf-8을 많이 사용)
	- `<title></title>` : sets the title of page
- `<body></body>` : includes content shown to visitors
	- `<img src="images/firefox-icon.png" alt="My test image">` : src, alt attributes  
	※ "alt text = descriptive text"  
	-> The Firefox logo: a flaming fox surrounding the Earth.
	
## Essential elements
### Headings
- 6 heading levels, <h1>-<h6>
- `<h1>Main title</h1>` : generally used once per page  
※ SEO(search Engine Optimization) considers headings as keywords  
-> Don't skip levels to expose my page

### Paragraphs
- `<p>Single paragraph</p>`

### Lists
- ordered/unordered lists(`<ol>` / `<ul>`) wrap list items
- list items are put inside an `<li>`(list item) element

### Links
- attribute `href` : put address to link(**h**ypertext **ref**erence)  
-> Don't omit protocol at the beginning
- `<a href="https://www.mozilla.org/en-US/about/manifesto/">Mozilla Manifesto</a>`