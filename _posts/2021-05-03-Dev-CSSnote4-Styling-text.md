---
layout: single
title: "CSSnote4: Styling text"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - CSS
  - MDN
  - link
  - list
  - Web fonts
---
Styling text
# Fundamental text and font styling
## What is involved in styling text in CSS?
The CSS properties used to style text generally fall into two categories
- Font styles : properties that affect the font(font 종류, 크기, bold, italic 등)
- Text layout styles : properties that affect layout features(spacing, text alignment 등)

**Note**: 하나의 element 안에 있는 text들은 one single entity로 CSS의 영향을 받음  subsection을 선택하기 위해선 pseudo-element(`::first-letter`, `::selection` 등)을 사용해야 함

## Fonts
### Color
Use `color` property

### Font families
Use `font-family`  
글꼴이 지원되지 않으면 default font를 사용함

#### Web safe fonts
generally available한 font들

| Name            | Generic type | Notes                                                                                  |
| :-------------- | :----------- | :------------------------------------------------------------------------------------- |
| Arial           | sans-serif   | *Helvetica*이 alternative로 사용하기 좋음(둘의 모양이 비슷함, *Arial*이 더 범용성있음) |
| Courier New     | monospace    | preferred alternative : *Courier*(더 오래된 버전)                                      |
| Georgia         | serif        |                                                                                        |
| Times New Roman | serif        | preferred alternative : *Times*(더 오래된 버전)                                        |
| Trebuchet MS    | sans-serif   | mobile OSes에서는 widely available하지 않음                                            |
| Verdana         | sans-serif   |                                                                                        |

#### Default fonts
CSS defines five generic names for fonts : `serif`, `sans-serif`, `monospace`, `cursive`, `fantasy`  
이 글꼴들은 브라우저, OS에 따라서 실제 사용되는 font face가 다를 수 있음!  
특히 `cursive`와 `fantasy`는 사용할 때 주의해야함(특이한 글꼴이라서 많이 다를 수 있음)

| Term         | Definition                                                                                                          | Example                                                          |
| :----------- | :------------------------------------------------------------------------------------------------------------------ | :--------------------------------------------------------------- |
| `serif`      | Fonts that have serifs(the flourishes and other small details you see at the ends of the strokes in some typefaces) | <span style="font-family:serif;">My big red elephant</span>      |
| `sans-serif` | Fonts that don't have serifs                                                                                        | <span style="font-family:sans-serif;">My big red elephant</span> |
| `monospace`  | Fonts where every character has the same width, typically used in code listings                                     | <span style="font-family:monospace;">My big red elephant</span>  |
| `cursive`    | Fonts that are intended to emulate handwriting, with flowing, connected strokes                                     | <span style="font-family:cursive;">My big red elephant</span>    |
| `fantasy`    | Fonts that are intended to be decorative                                                                            | <span style="font-family:fantasy;">My big red elephant</span>    |

#### Font stacks
Supply a font stack so that the browser has multiple fonts it can choose from.  
e.g. `font-family: "Trebuchet MS", Verdana, sans-serif;`  
브라우저가 font를 지원하지 않는 것을 대비해서 마지막에 suitable generic font를 추가해놓는게 좋음  

**Note**: font name이 한 단어보다 많을 경우 double quotes로 감싸야 함

### Font size
Use `font-size`  
`font-size`는 `px`, `em`, `rem` 등 많은 unit들을 사용 가능  
기본적으로 parent element로부터 inherit됨(16px가 root element의 default size임)  
`em`은 parent element의 크기가 기준이기 때문에 container elements에 대해서는 `font-size` 설정을 피하는게 나음

### Font style, font weight, text transform, and text decoration
Four common properties to alter the visual weight/emphasis of text:
- `font-style` : Use to turn *italic* text on and off<br>possible values:
	- `normal`
	- `italic` : set the text to use the *italic version* of the font<br>(if not available, it will simulate italics with oblique instead)
	- `oblique` : set the text to use a simulated version of an italic font(*slanting* the normal version)
- `font-weight` : Set boldness of the text<br>values available:
	- `normal`, `bold`
	- `lighter`, `bolder` : calculate with parent element's boldness
	- `100` - `900` : numeric boldness
- `text-transform` : Set font to be transformed<br>values include:
	- `none`
	- `uppercase`, `lowercase`
	- `capitalize`
	- `full-width` : Transform all glyphs to be written inside a fixed-width square(similar to monospace font)
- `text-decoration` : Set text decorations(underline ...)<br>available values:
	- `none`
	- `underline`
	- `overline` : <span style="text-decoration:overline;">Gives the text an overline</span>
	- `line-through` : ~~취소선~~
	
	multiple value 입력 가능  
	`text-decoration`은 `text-decoration-line`, `-style`, `-color`의 shorthand임

### Text drop shadows
Use `text-shadow`  
*CSS building blocks*에서 설명한대로 `text-shadow: [x-offset] [y-offset] [blur radius] [color];`의 포맷을 가짐(`inset`, `spread radius`는 없음)  

#### Multiple shadows
shadow 여러 개 가능

## Text layout
### Text alignment
Use `text-align`
- `left`
- `right`
- `center`
- `justify` : Make the text spread out<br>이걸 사용할 때는 긴 단어는 `-`을 이용해서 나눠야 하는 것을 고려해야함

### Line height
Use `line-height` to set the height of each line of text  
숫자만 넣으면 font size에 곱해져서 계산됨  
`1.5`-`2`가 보통

### Letter and word spacing
Use `letter-spacing` and `word-spacing` to set the spacing between letters and words  
e.g. `letter-spacing: 4px;`

### Other properties worth looking at
<a href="https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Fundamentals#other_properties_worth_looking_at" target="_blank">다른 properties로 많음</a>

## Font shorthand
`font`를 shorthand로 사용할 때 `font-style`, `font-variant`, `font-weight`, `font-stretch`, `font-size`, `line-height`, `font-family` 순서로 작성되어야 함  
`font-size`, `font-family` 두 개만 required임  
`font-size`, `line-height` 사이에는 `/`를 사용해야함(둘 다 length and size units가 들어가기 때문)  
e.g. `font: italic normal bold normal 3em/1.5 Helvetica, Arial, sans-serif;`

# Styling lists
## A simple list example
Styling defaults:
- `<ul>`, `<ol>` elements의 top, bottom `margin` : `1em(16px)`, `padding-left` : `40px`
- `<li>`의 spacing은 default가 없음
- `<dl>`의 top, bottom `margin` : `1em(16px)`
- `<dd>`의 `margin-left` : `40px`
- `<p>`의 top, bottom `margin` : `1em(16px)`

## Handling list spacing
list가 surrounding elements와 같은 horizontal, vertical spacing을 가지도록 여백을 조절해야함  
`line-height`, `font-size`를 조절해서 가능함

## List-specific styles
general spacing techuiques for lists(`<ul>`, `<ol>`)
- `list-style-type` : Set the type of bullets
- `list-style-position` : Set whether the bullets appear inside or outside the list items
- `list-style-image` : Allow you to use a custom image for the bullet

### Bullet styles
Use `list-style-type`  
values available:
- for `<ol>` : `upper-roman`, `lower-roman`, `decimal`, `upper-alpha`, `lower-alpha`, `none`
- for `<ul>` : `<string>`(e.g. `'-'`), `disc`, `circle`, `square`, `none`

### Bullet position
Use `list-style-position`  
values available:
- `outside` : default, bullets가 list item의 바깥에 배치됨
- `inside` : bullets가 list item의 안에 배치됨

### Using a custom bullet image
Use `list-style-image`  
Use `url()` to link the image  
근데 이 property는 bullet의 position, size 등을 조절하기 어려움  
오히려 `<li>`에 `background`를 이용하는게 더 편함

#### Example
CSS:  
```css
ul {
  padding-left: 2rem;
  list-style-type: none;
}

ul li {
  padding-left: 2rem;
  background-image: url(star.svg);
  background-position: 0 0;
  background-size: 1.6rem 1.6rem;
  background-repeat: no-repeat;
}
```
- `<ul>`의 `padding-left` `40px`를 `ul`, `ul li`에 `20px`씩 나눔<br>∵ `<ul>`의 list item이 `<ol>`, `<dl>`과 같은 여백을 가지면서 background로 listmarker를 넣기 위함(`ul li`에 `padding-left`가 없으면 background가 `<li>`의 본문과 겹쳐버림
- `list-style-type`을 `none`으로 설정<br>∵ background로 넣기 위함

### list-style shorthand
`list-style`에 `list-style-type`, `list-style-image`, `list-style-position`을 한꺼번에 쓸 수 있음  
default : `disc`, `none`, `outside`  
순서는 상관없음  
만약 `type`, `image`가 둘 다 명시된 경우 type이 fallback으로 사용됨

#### Example
```css
ul {
  list-style-type: square;
  list-style-image: url(example.png);
  list-style-position: inside;
}
```

is equal to 

```css
ul {
  list-style: square url(example.png) inside;
}
```

## Conrolling list counting
ordered list의 list marker index를 바꾸는 방법

### start
`start` attribute로 시작지점 설정  
e.g. `<ol start="4">`

### reversed
`reversed` attribute로 증가할지 감소할지 설정  
e.g. `<ol start="4" reversed>` => 4, 3, 2, 1, 0, -1, ... 순으로 index

### value
`value` attribute로 list marker를 직접 지정  
다른 attributes와 다르게 `<li>`에 넣어야 함  
non-number `list-style-type`을 사용하고 있으면 적용되지 않음  
value를 지정하면 다음 `<li>`도 영향을 받음  
e.g. `<li value="2">`로 설정해놓으면 다음 li는 marker가 3이 됨

# Styling links
## Let's look at some link
### Link states
different pseudo-classes:
- `:link` : a link which has a destination
- `:visited`
- `:hover`
- `:focus` : a link when it has been focused(e.g. tab같은 걸로 focus받았을 때)
- `:active` : a link when it is being activated(e.g. clicked on)

### Default styles
- links are underlined
- Unvisited links are blue
- Visited links are purple
- Hovering a link makes the mouse pointer change to a little hand icon
- Focused links have an outline around them(tab키로 페이지의 링크들을 focus할 수 있음)
- Active links are red

※ `#`을 dummy link로 사용 가능  
e.g. `<a href="#">links</a>`

이 default styles는 1990년대 초기 브라우저들과 거의 동일함(혼동을 유발하지 않기 위해)  
최소한 다음과 같은 양식은 있어야 함
- Use underlining or highlight for links
- Make them react in some way when hovered/focused, and in a slightly different way when activated

`cursor` property를 사용해서 마우스 커서 조절 가능

### Styling some links
```css
a {
}

a:link {
}

a:visited {
}

a:focus {
}

a:hover {
}

a:active {
}
```
위의 순서(LVFHA)대로 ruleset을 정의하는게 좋음(activated면 hover이기도 하기 때문)  

※ underlining은 `border-bottom`, `text-decoration`을 이용해서 넣을 수 있는데,<br>`border-bottom`이 더 밑쪽에 줄이 그어짐, y, g같은 글자의 꼬리가 줄에 안닿여서 더 좋음  
`border-bottom: 1px solid;`같이 색을 지정하지 않으면 `color`과 같은 색으로 설정됨

## Including icons on links
```css
a[href*="http"] {
  background: url('https://mdn.mozillademos.org/files/12982/external-link-52.png') no-repeat 100% 0;
  background-size: 16px 16px;
  padding-right: 19px;
}
```
- `background` shorthand로 img 집어넣음
- icon이 글자보다 더 커서 잘릴 경우를 대비해서 `background-size`를 이용해서 이미지를 resize해줌<br>(IE 9 이후의 버전에서만 지원됨, 이전의 브라우저들에 대해서는 resizing한 이미지를 참조해줘야함)
- `padding-right`를 넣어서 icon이 들어갈 공간 확보
- external link는 `http`가 링크에 포함되어있음 => `a[href*="http"]` selector 이용<br>내부 링크는 relative link를 이용하면 됨

## Styling links as buttons
`<li>`에 `display: inline;`을 적용하여 navigation bar를 구현할 수 있음

#### Example
CSS:  
```css
body,html {
  margin: 0;
  font-family: sans-serif;
}

ul {
  padding: 0;
  width: 100%;
}

li {
  display: inline;
}

a {
  outline: none;
  text-decoration: none;
  display: inline-block;
  width: 19.5%;
  margin-right: 0.625%;
  text-align: center;
  line-height: 3;
  color: black;
}

li:last-child a {
  margin-right: 0;
}

a:link, a:visited, a:focus {
  background: yellow;
}

a:hover {
  background: orange;
}

a:active {
  background: red;
  color: white;
}
```

HTML:  
```html
<ul>
  <li><a href="#">Home</a></li><li><a href="#">Pizza</a></li><li><a href="#">Music</a></li><li><a href="#">Wombats</a></li><li><a href="#">Finland</a></li>
</ul>
```
- navigation bar 이외에 `<ul>` 등의 다른 element에 의한 padding, margin을 다 없앰
- HTML에서 모든 `<li>`를 같은 줄에 적음<br>∵ inline elements가 다른 줄에 적히면 사이에 space가 생길 수 있음
- `<a>`에 `display: inline-block;`을 적용해서 inline element지만 sizing이 가능하도록 만듦
- 마지막 `<a>`의 margin-right는 여백이 되기 때문에 `li:last-child a`를 이용해서 없애줌
- `<a>` 대신 `<li>`를 수정해서 nav bar를 만들 수도 있음
- `outline: none;`은 링크가 탭으로 active될 경우 생기는 테두리를 없애줌

# Web fonts
## Font families recap
web safe font를 가지는 font stack을 사용하면 각 font마다 test해야하기 때문에 overhead 발생

## Web fonts
Web fonts를 사용하면 download해야 하는 font files를 지정할 수 있음  
CSS의 가장 윗부분에 `@font-face` block을 정의해야 `@font-face`의 font family를 사용 가능  


#### Example
CSS:  
```css
@font-face {
  font-family: "myFont";
  src: url("myFont.woff2");
}
```

HTML:  
```html
html {
  font-family: "myFont", "Bitstream Vera Serif", serif;
}
```

- WOFF/WOFF2(Web Open Font Format 1/2)은 웬만한 브라우저들이 다 지원함
- WOFF2는 TTF, OTF를 둘 다 지원함
	- TTF(True Type Font) : 일반적인 형식
	- OTF(Open Type Font) : 비교적 최근에 나옴
- font files를 나열한 순서도 중요함(위에서부터 확인함 => 첫 번째가 가장 범용성이 높은 format이어야 함)
- legacy browser를 사용하는 경우 EOT, TTF, SVG webfonts를 준비해야 함

## Active learning: A web font example
Three types of sites to obtain fonts:
- Free font distributor : e.g. Font Squirrel
- Paid font distributor : e.g. fonts.com
- Online font service : e.g. google fonts

### Generating the required code
Font squirrel의 webfont generator에 font들을 올리고 legacy browser를 지원해야 하는 경우 SVG, EOT, TTF도 선택하고 download kit  
font file의 크기가 큰 경우 ttf를 woff2, svg 등으로 바꿔야 함

### Implementing the code in your demo
1. 다운받은 webfonts kit을 웹페이지와 같은 경로에 저장(압축 풀고 `fonts`와 같이 이름 바꿈)
2. `stylesheet.css`의 내용을 내 css 파일의 가장 위에 넣음
3. `@font-face`에서 url 수정
4. `@font-face`에 정의된 `font-family`를 사용 가능

## Using an online font service
How to use google fonts:
1. font들 찾아서 맨 밑에 XOR버튼 누름
2. 다 선택한 다음 HTML code를 내 page header에 붙여넣기
3. 밑에 CSS 코드처럼 font 사용 가능

## @font-face in more detail
```css
@font-face {
  font-family: 'zantrokeregular';
  src: url('zantroke-webfont.woff2') format('woff2'),
       url('zantroke-webfont.woff') format('woff');
  font-weight: normal;
  font-style: normal;
}
```
- `font-family` : specifies font name
- `src` : specifies paths to the font files to be imported, `format('woff2')`는 optional
- `font-weight`/`font-style` : specifies boldness, style(italic ...)<br>같은 font를 boldness 별로 여러 개 import하고 있다면 `font-weight`를 적어야 boldness별로 적용 가능

## Variable fonts
Variable fonts allow many different variations of a typeface(weight, style ...) into a single file(OTF와 비슷)

# Assessment09-Typesetting-a-community-school-homepage
CSS:  
```css
/* General setup */
@import url('https://fonts.googleapis.com/css2?family=Krona+One&family=Open+Sans:ital,wght@0,300;0,600;1,300&display=swap');

* {
  box-sizing: border-box;
}

body {
  margin: 0 auto;
  min-width: 1000px;
  max-width: 1400px;
}

/* Layout */

section {
  float: left;
  width: 50%;
}

aside {
  float: left;
  width: 30%;
}

nav {
  float: left;
  width: 20%;
}

footer {
  clear: both;
}

header, section, aside, nav, footer {
  padding: 20px;
}

/* header and footer */

header, footer {
  border-top: 5px solid #a66;
  border-bottom: 5px solid #a66;
}

/* WRITE YOUR CODE BELOW HERE */

html {
  font-size: 10px;
}

body {
  font-family: 'Open Sans', sans-serif;
  line-height: 1.6em;
  letter-spacing: 1px;
  word-spacing: 2px;
}

h1, h2 {
  font-family: 'Krona One', sans-serif;
  letter-spacing: 3px;
}

h1 {
  font-size: 2em;
  text-align: center;
}

h2 {
  font-size: 1.5em;
}

section h2+p {
  text-indent: 20px;
}

a:link, a:visited, a:focus, a:hover {
  color: #a66;
}

a:hover, a:focus {
  text-decoration: none;
}

a {
  outline: none;
}

a:active {
  outline: 1px solid #a66;
}

a[href*='http'] {
  padding-right: 15px;
  background: url("https://mdn.mozillademos.org/files/12982/external-link-52.png") no-repeat 100%;
  background-size: 10px 10px;
}

li {
  line-height: 1.6em;
}

ul, ol {
  margin: 16px 0px;
}

ul {
  list-style-type: circle;
}

ol {
  list-style-type: lower-alpha;
}

nav ul {
  list-style-type: none;
  margin: 0;
  padding: 0;
}

nav ul li {
  padding: 10px 0;
}

nav li a {
  display: inline-block;
  border: 1px solid #b77;
  width: 100%;
  height: 5rem;
  line-height: 5rem;
  font-size: 1.2rem;
  text-align: center;
  text-decoration: none;
}

nav li a:hover {
  background-color: #fba;
}
```

HTML:  
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>St Huxley's Community College</title>
    <link href="style.css" type="text/css" rel="stylesheet">
  </head>
  <body>

  <header>
    <h1>St Huxley's Community College</h1>
  </header>

  <section>
      <h2>Brave new world</h2>

      <p>It's a brave new world out there. Our children are being put in increasing more competitive situations, both during recreation, and as they start to move into the adult world of <a href="https://en.wikipedia.org/wiki/Examination">examinations</a>, <a href="https://en.wikipedia.org/wiki/Jobs">jobs</a>, <a href="https://en.wikipedia.org/wiki/Career">careers</a>, and other life choices. Having the wrong mindset, becoming <a href="https://en.wikipedia.org/wiki/Emotion">too emotional</a>, or making the wrong choices can contribute to them experiencing difficulty in taking their rightful place in today's ideal society.</p>

      <p>As concerned parents, guardians or carers, you will no doubt want to give your children the best possible start in life — and you've come to the right place.</p>

      <h2>The best start in life</h2>

      <p>At St. Huxley's, we pride ourselves in not only giving our students a top quality education, but also giving them the societal and emotional intelligence they need to win big in the coming utopia. We not only excel at subjects such as genetics, data mining, and chemistry, but we also include compulsory lessons in:</p>

      <ul>
        <li>Emotional control</li>
        <li>Judgement</li>
        <li>Assertion</li>
        <li>Focus and resolve</li>
      </ul>

      <p>If you are interested, then you next steps will likely be to:</p>
      
      <ol>
        <li><a href="#">Call us</a> for more information</li>
        <li><a href="#">Ask for a brochure</a>, which includes signup form</li>
        <li><a href="#">Book a visit</a>!</li>
      </ol>
  </section>

  <aside>

    <h2>Top course choices</h2>

    <ul>
      <li><a href="#">Genetic engineering</a></li>
      <li><a href="#">Genetic mutation</a></li>
      <li><a href="#">Organic Chemistry</a></li>
      <li><a href="#">Pharmaceuticals</a></li>
      <li><a href="#">Biochemistry with behaviour</a></li>
      <li><a href="#">Pure biochemistry</a></li>
      <li><a href="#">Data mining</a></li>
      <li><a href="#">Computer security</a></li>
      <li><a href="#">Bioinformatics</a></li>
      <li><a href="#">Cybernetics</a></li>
    </ul>

    <p><a href="#">See more</a></p>

  </aside>

  <nav>

    <ul>
      <li><a href="#">Home</a></li>
      <li><a href="#">Finding us</a></li>
      <li><a href="#">Courses</a></li>
      <li><a href="#">Staff</a></li>
      <li><a href="#">Media</a></li>
      <li><a href="#">Prospectus</a></li>
    </ul>

  </nav>

  <footer>

    <p>&copy; 2016 St Huxley's Community College</p>

  </footer>

  </body>
</html>
```

- url의 parameter에 quotes를 넣는건 optional(안쓰든, single, double을 쓰든)
- background-position은 horizontal, vertical 순임<br>`padding`, `margin`은 vertical, horizontal 순!
- `box-sizing` : `content-box`(standard), `border-box`(alternative)<br>`display` : `inline-block`, ...
- `display: inline-block;`을 적용하고 width, height를 선언할 경우, 모두 border까지 포함한 크기로 계산됨(`box-sizing: border-box;`와 동일)
	- 이 과제에서는 inline-block 안에 `height`, `line-height`를 같이 썼는데, 사실 `height`는 높이를 꼭 `5rem`으로 맞춰야 할 필요가 없으면 사용하지 않아도 됨
