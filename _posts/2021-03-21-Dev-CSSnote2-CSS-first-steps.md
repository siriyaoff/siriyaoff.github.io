---
layout: single
title: "CSSnote2: CSS first steps"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - CSS
  - MDN
  - DOM
  - Specificity
  - Syntax
---
CSS first steps
# What is CSS?
Document : a text file structured using a markup language(html, xml, ...)  
Presenting a document = converting a document  
Browsers(kinds of user agent) are designed to present documents visually

## CSS syntax
CSS is a rule-based language.

An example of the rule:
```css
h1 {
	color: red;
	font-size: 5em;
}
```
- The rule opens with a **selector**. Then we have a set of curly braces `{ }`. Inside, there will be one or more **declarations**(=`[PROPERTY]: [VALUE];`)

## CSS Modules
Property를 잘 찾을 수 있게 Module로 분류해놓음(MDN reference에서 찾을 수 있음, e.g. [Backgrounds and Borders](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Backgrounds_and_Borders){:target="_blank"})

### CSS specifications
모든 web standards technologies는 standards organizations가 만드는 specifications(or "specs")라는 문서에 의해 정의된다. CSS의 경우 W3C의 CSS Working Group에 의해서 specification이 만들어진다. CSS specs는 user agents에서 기능들이 작동하도록 구현하는데 도움을 주는 목적으로 작성된 것이다. 즉CSS에 대한 이해를 위해 읽는게 아니고, CSS, browser support, specs의 관계에 대해서만 이해하고 있으면 됨
- Once CSS has been specified then it is only useful for us in developing web pages if one or more browsers have implemeneted it.

=> spec에 정의되어있으면 css code에 사용할 수 있지만 browser에서 구현해놓아야 적용됨

# Getting started with CSS
- Add CSS to an HTML document by adding `<link>` in the head of the document.  
`<link rel="stylesheet" href="styles.css">`
- `selector` matches an HTML element name directly  
Choose multipe elements with `,`

## Adding a class
In HTML, we can add a class to elements:  
`<li class="special">Item two</li>`

In CSS, we can target the class by creating a selector starts with a full stop(`.`)
- `.special {` : any element has a class of `special`
- `li.special {` : any `<li>` has a class of `special`
- `li.special, span.special {` : any `<li>` or `<span>` have a class of `special`  
※ `.special, h1`과 같이 생성자를 섞어서 사용할 수도 있음

※ 한 element에 대해 모두 같은 스타일을 적용할 일은 잘 없기 때문에 항상 선언할 때 `class`를 추가하는 것이 좋음!

## Styling things based on their location in a document
- There are a number of combinators for kinds of this purpose, such as `descendant combinator`, `adjacent sibling combinator`, ...

### Descendant combinator `' '`
`A B {` : Target `B` element nested inside `A`(B is a descendant of A)  
e.g. `li em { ...` selects any `<em>` nested inside `<li>`

### Adjacent sibling combinator `+`
`A + B {` : Target `B` right after `A`  
`A`, `B` is at the same hierarchy level in the HTML  
e.g. `h1 + p { ...` selects any `<p>` directly after `<h1>`

## Styling things based on state
Some elements can have several states like visited, unvisited, hover, clicked(activated), ...  
We can style elements in specific states by using `:`  
`a:visited { ...` selects any `<a>` visited

**Combinations possible!**  
- `body h1 + p .special { ...` : any element with a class of `special` inside a `<p>` which comes just after an `<h1>` which is inside a `<body>`  
- `p a.ac:hover { ...` : `<a>`  with class `ac`, state `hover`, inside a `<p>`  
※ `ul+p` : `<ul>`이 끝난 직후의 `<p>`(`</ul>` 다음의 `<p>`)

***************************
궁금한거
`a`에서 `font-weight: bold;`, `a:visited`에서 `font-weight: normal;`이라 선언하면 visited상태인 링크는 bold가 그대로 적용되어있음. visited가 우선일 것 같은데 왜이런지 모르겠음(color는 둘다 설정되어있으면 visited걸로 바뀌는데 font-weight, font-style은 a에서 900, 이탈릭이고 visited에 normal, normal일때 a꺼가 그대로 유지됨) : pseudo-class 우선순위 찾아봐야할듯!
```css
a {
	color: red;
	font-weight: 900;
	font-style: italic;
}

a:visited {
	color: green;
	font-weight: normal;
	font-style: normal;
}
```

- 2010.03부터 `:visited`에서 사용 가능한 property가 제한됨(`color`, `background-color`, `border-*-color`, `outline-color` 등만 가능)<br>추가로 `:visited + span`과 같이 selector를 정의하면 link가 unvisited일 때도 적용됨<br>개인정보 관련된 이유임([여기](https://hacks.mozilla.org/2010/03/privacy-related-changes-coming-to-css-vistited/) 참고)<br>- 2021.05.22

`:`는 combinator가 아닌가?(정의?) : pseudo-class임!

***************************

# How CSS is structured
## Applying CSS to HTML
Three methods available to apply CSS to a document: with an external stylesheet, with an internal stylesheet, and with inline styles.
### 1. External stylesheet
`<link rel="stylesheet" href="styles.css">` in `<head>` of an HTML document.
- most common and useful method
- possible to link a single CSS file to multiple web pages

### 2. Internal stylesheet
`<style> CSS CODE </style>` in `<head>` of an HTML document
- useful in some circumstances such as blocked from modify external CSS files.  
but for sites with many pages, internal stylesheet is less efficient.(one simple styling change requires editing multiple pages.)

### 3. Inline styles
`<p style="color:red;bacground-color: yellow;"> ... </p>` for each element needs styling
- **Avoid it when possible!**
	- the least efficient for maintenance
	- mixes presentational code(CSS) with HTML
- few circumstances such as email(to achieve compatibility with email clients), only allowed to edit the HTML body.

## Selectors
valid selectors:  
```css
h1
a:link
.classes
#ids /*id should be unique!*/
*
.box p
.box p:first-child
h1, h2, .intro
```
- `A:first-child` : 같은 부모를 가진 A elements 중 첫 번째 자식 선택(div안에 첫 번째 p, body안에 첫 번째 p 등으로 여러 개가 선택될 수 있음)  
※ Learn more about selectors in the next module: CSS selectors

### Specificity
If two selectors select the same element, CSS apply the rules by **cascade** and **specificity**.

**cascade** rule  
Later styles replace conflicting styles that appear earlier in the stylesheet.  
e.g.  
```css
p {
	color: red;
}

p {
	color: blue;
}
```
- the paragraph text will be blue by the cascade rule.

**specificity** rule  
If two selectors select the same element, CSS apply the rule that has more specificity. It is stronger than the cascade rule.

## Properties and values
- **case-sensitive**!!
- based on US spelling(no `colour`)
- If a property or a value is not valid, it is completely ignored by the browser's CSS engine.

### Functions
There are some functions we can use within CSS, such as `calc()`, `rotate()`  
e.g. `width: calc(90%-30px);`, `transform: rotate(0.8turn);`

## @rules
@rules("at-rules") provide instruction for what CSS should perform or how it should behave.  
**Example**  
`@import`
- to import a stylesheet into another CSS stylesheet
- `@import 'styles2.css';`

`@media`
- to create media queries(conditional logic for applying CSS styling(responsive image에서 사용하는 것)
```css
@media (min-width: 30em) {
  body {
    background-color: blue;
  }
}
```

## Shorthands
Properties set several values in a single line, such as `font`, `background`, `padding`, `border`, `margin`
e.g.  
`background: red url(bg-graphic.png) 10px 10px repeat-x fixed;`  
is equivalent to  
```
background-color: red;
background-image: url(bg-graphic.png);
background-position: 10px 10px;
background-repeat: repeat-x;
background-attachment: fixed;
```

> **Warning**: An omission in CSS shorthand can **override previously set values**.
> A value not specified in CSS shorthand reverts to its initial value.

## Comments
`/* Handle basic element styling */`  
`/* Handle specific elements nested in the DOM  */`  
와 같이 주석 달아놓는게 좋음.

**comment out the code**  
```css
/*.special {
  color: red;
}*/
```

## White space
There is no semantic role for white spaces, but it improves readability.

Property names and property values never have white space!

# How CSS works
## How does CSS actually work?
Simplified steps when a browser loads a webpage: 
1. The browser loads the HTML.
2. It converts the HTML into a DOM(Document Object Model). The DOM represents the document in the computer's memory.
3. The browser fetches most of the resources(images, CSS, ...) linked to by the HTML document. + handling JavaScript
4. The browser parses the fetched CSS, and sorts the rules by selector types into different "buckets"(element, class, ID, ...). Based on the selectors, it works out which rules should be applied to which nodes in the DOM, and attaches style to them as required (this intermediate step is called a render tree).
5. The render tree is laid out with the rules applied appropriately.
6. The visual display of the page is shown on the screen (this stage is called painting).

A Diagram of the process:  
![simple view of the process](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps/How_CSS_works/rendering.svg)

## About the DOM
A DOM is a **tree-like structure**. Each element, attribute, and piece of text in the markup language becomes a DOM node in the tree structure. The nodes are defined by their relationship to other DOM nodes. It is **where CSS and the document's content meet up**.

## A real DOM representation
**Example**:  
HTML code:  
```html
<p>
  Let's use:
  <span>Cascading</span>
  <span>Style</span>
  <span>Sheets</span>
</p>
```

converted DOM:  
```
P
├─ "Let's use:"
├─ SPAN
|  └─ "Cascading"
├─ SPAN
|  └─ "Style"
└─ SPAN
   └─ "Sheets"
```

rendered outputs:  
```
Let's use: Cascading Style Sheets
```

## Applying CSS to the DOM
- After a DOM created, the browser will parse the CSS, sort it, apply the rules, and paint the final visual representation to screen.

## What happens if a browser encounters CSS it doesn't understand?
- Just moves on to the next bit of CSS
- cascade rule을 이용해서 새로운 기능을 지원하지 않는 browser에 대해 alternatives를 제공할 수 있음

e.g.  
```css
.box {
  width: 500px;
  width: calc(100%-50px);
}
```