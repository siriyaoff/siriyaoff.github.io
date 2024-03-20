---
layout: single
title: "HTMLnote3: Multimedia and embedding"
toc: true
toc_label: "Index"
toc_icon: "columns"
classes: wide
categories:
  - Dev
tags:
  - HTML
  - MDN
  - HTML tags
  - Accessibility
  - Embedding
  - WebVTT
  - iframe
  - SVG
  - Responsive image
---
# Multimedia and embedding
## Images in HTML
### `<img>`
- att : `src`, `alt`, `width`, `height`
- hotlink to images on other servers는 하지 않는게 좋음
- `alt`가 필요한 이유
	- Accessibility
	- misspelling of `src`
	- The browser doesn't support the image type
	- SEO
	- Low data mode
- image에 링크를 걸 경우 로딩이 안될 때를 대비해 `alt`에 링크를 써놓는게 좋음
- `width`, `height`
	- `width`, `height`를 써놓았을 때 image 로딩이 느릴 경우 이미지 공간을 미리 확보해놓아 더 빠르게 로드할 수 있음
	- 두 속성으로 이미지의 aspect ratio를 바꿀 수 있지만, 이미지가 왜곡됨  
	-> 이미지 비율을 바꿔야할 경우 CSS를 사용하는게 나음
- `title`로 tooltip을 추가할 수 있지만, hovering with a mouse가 제한된 유저들(터치스크린 등)은 접근성이 떨어짐  
-> 차라리 main article text에 포함시키는게 나음

#### image map
```html
<img src="/examples/images/img_imagemap.jpg" alt="진실혹은거짓" usemap="#vending" style="width:320px; height:240px" />
<map name="vending">
	<area shape="rect" coords="90,60,180,130" alt="거짓"
		href="https://ko.wikipedia.org/wiki/%EA%B1%B0%EC%A7%93%EB%A7%90">
	<area shape="rect" coords="210,200,70,130" alt="진실"
		href="https://ko.wikipedia.org/wiki/%EC%A7%84%EC%8B%A4">
</map>
```
- `<map>`은 하나 이상의 `<area>`를 가짐, `<area>`가 버튼과 같은 역할

### `<figure>`, `<figcaption>` : Annotating images
- provide a semantic container for figures  
	```html
	<figure>
		[FIGURE]
		<figcaption>caption</figcaption>
	</figure>
	```
- [FIGURE]에는 image, a code snippet, video, equations, table 등이 들어갈 수 있음
- `alt`와 `figcaption`는 semantic role이 다름

### CSS background images
- CSS의 `background-image`를 이용  
	```html
	p{
		background-image: url("images/dinosaur.jpg");
	}
	```
- No semantic meaning at all(It can't have any text equivalents)  
※ If an image has meaning, in terms of your content, you should use an HTML image. If an image is purely decoration, you should use CSS background images.

## Video and audio content
### `<video>`
- att : `src`, `controls`, `width`, `height`, `autoplay`, `loop`, `muted`, `preload`, `poster`  
	```html
	<video src="rabbit320.webm" controls>
	  <p>Your browser doesn't support HTML5 video. Here is a <a href="rabbit320.webm">link to the video</a> instead.</p>
	</video>
	```
- fallback content : A paragraph inside the `<video>` tags(=alternative text)
- `controls` : boolean attribute, allow users to control video/audio playback
- Using multiple source formats to improve compatibility
	- container formats : like MP3, MP4, WebM, ...
	- codecs ∈ containers(e.g., WebM : Vorbis/Opus audio codec, VP8/VP9 video codec)  
	※ FLAC과 같은 특이한 경우도 있음(FLAC codec이 FLAC file안에 저장됨)
	- Different browsers support different video and audio formats, different container formats(MP3, MP4, WebM)
	- audio track don't need containers(audio player will play it directly)
	- `<source>`를 이용해서 video URL을 `<video>`안에 element로 넣을 수도 있음  
		```html
		<video controls>
			<source src="rabbit320.mp4" type="video/mp4">
			<source src="rabbit320.webm" type="video/webm">
			<p>fallback content <a href="rabbit320.mp4">here</a></p>
		</video>
		```
	- `<source>`로 URL을 넣으면 browser가 재생가능한 codec을 재생해줌
	- `type`을 추가하지 않으면 브라우저가 하나씩 로딩해서 체크하기 때문에 시간이 오래걸림
- 나머지 att들은 `<video controls width="400" height="400"	autoplay loop muted preload="auto" poster="poster.png"> ...`와 같이 사용
- `width`, `height` : aspect ratio와 맞지 않을 경우 Image와 같이 강제로 바꾸진 않고 가로를 채움
- `autoplay` : makes the audio/video playing right away
- `loop` : makes the audio/video start playing again whenever it finishes
- `muted` : set muted as default
- `poster` : the URL of an image displayed before the video is played
- `preload` : used for buffering large files
	- "none" : does not buffer the file
	- "auto" : buffers the media file
	- "metadata" : buffers only the metadata for the file

### `<audio>`
- works just like the `<video>` element
- differences with `<video>`
	- no `width`, `height`, `poster` attributes(no visual componenet)

### Displaying video text tracks
- reasons why it is needed
	- for auditory impairments
	- users in noisy environments
	- users in environments where having the audio playing would be a distraction
	- users who don't speak the language of the video
- `<track>`, WebVTT file format 이용
	- WebVTT(Web Video Text Tracks) : a format for writing text files containing multiple strings of text along with metadata(e.g. subtitles, captions, timed descriptions)  
		```html
		<video controls>
			<source src="example.mp4" type="video/mp4">
			<source src="example.webm" type="video/webm">
			<track kind="subtitles" src="subtitles_es.vtt" srclang="es" label="Spanish">
		</video>
		```
	- `<track>` should be placed within `<audio>` or `<video>` but after all `<source>` elements
	- `kind` : specifies cues' purpose("subtitles", "captions", "descriptions")
	- `srclang` : language subtitles are written in
	- `label` : name of the track, set it to language to help readers

## From object to iframe
A short history of embedding : `<object>`, `<embed>`(Java Applets, Flash) -> `<iframe>`, `<canvas>`, `<video>`

### `<iframe>`
```html
<iframe src="https://developer.mozilla.org/en-US/docs/Glossary"
		width="100%" height="500" frameborder="0"
		allowfullscreen sandbox>
  <p>
	<a href="/en-US/docs/Glossary">
	   Fallback link for browsers that don't support iframes
	</a>
  </p>
</iframe>
```
- att : `src`, `width`, `height`, `frameborder`, `allowfullscreen`, `sandbox`
- `frameborder` : set a border between this frame and other frames, default : 1
	- can be better achieved using `border : none;` in CSS
- `sandbox` : requests heightened security settings(need browsers > IE 10)  
※ It's a good idea to set the iframe's `src` attribute with JavaScript after the main content is doen with loading. This makes your page usable sooner and decreases your official page load time(an important SEO metric)
- iframe의 id를 선언하고 링크에서 target="ID"로 사용 가능(링크된 문서가 그 frame에서 열림)
	- HTML5 이전에는 `frameset`이 있었지만 이젠 권고하지 않음

### Security concerns
- clickjacking, Intellectual Property issues, ...  
-> 'you can never be too cautious'
- Embed content starts with HTTP for safety
- Always use the `sandbox` attribute
	- sandbox 밖의 나머지 부분을 조작할 수 없는 컨테이너
	- `sandbox=""`으로 permissions 추가 가능하지만 `allow-scripts`, `allow-same-origin`을 동시에 추가하면 sandbox를 사용하지 않는 것과 같음
- CSP(content security policy)를 사용할 수도 있다고 함(HTTP headers라는데 사용법은 모르겠음)

### `<embed>` and `<object>`
- general purpose embedding tools for embedding multiple types of external content, which include plugin technologies(e.g., JavaApplets, Flash, PDF, ...)
- 이제는 잘 쓰지 않는 content이거나 더 좋음 embedding methods가 생겨서 잘 쓰지 않음
- attributes는 [MDN 항목 참고](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Other_embedding_technologies#the_%3Cembed%3E_and_%3Cobject%3E_elements){:target="_blank"}

## Adding vector graphics to the Web
### Raster images
- defined using a grid of pixels(e.g., `.bmp`, `.png`, `.jpg`, `.gif`)

### Vector images
- defined using algorithms work out the shapes
- don't pixelate when zoomed in
- lighter than raster equivalents

### SVG
- XML-based language for describing vector images  
	```html
	<svg version="1.1"
		baseProfile="full"
		width="300" height="200"
		xmlns="http://www.w3.org/2000/svg">
		<rect width="100%" height="100%" fill="black" />
		<circle cx="150" cy="100" r="90" fill="blue" />
	</svg>
	```
- `<rect>`, `<circle>`, `<feColorMatrix>`, `<animate>`, `<mask>`와 같은 features가 있음
- Inkscape, Illustrator와 같은 vector graphics editor 사용
- additional advantages
	- text in vector images는 accessible
	- SVG의 each component는 CSS나 JavaScript로 styling 가능
- disadvantages
	- can getn complicated very quickly, complex SVGs take time to be displayed
	- can be harder to create than raster images
	- not supported in older browsers(supported since IE9)

### Adding SVG to your pages
Adding SVG can be done in 3 ways.

#### 1. The quick way: `<img>`
- 다른 format들 넣는 것처럼 넣으면 됨
- Pros
	- quick, familiar syntax
	- easy to make hyperlink by nesting `<img>`
	- SVG file can ben cached by the browser
- Cons
	- cannot manipulate the image with JavaScript
	- to control the SVG with CSS, you must include inline CSS
	- cannot restyle the image with CSS pseudoclasses(like `:focus`)
- Troubleshooting and cross-browser support
	- `<img src="shape.png" alt="alttext" srcset="shape.svg">`
		- only supporting browsers will load the SVG, olders will load the PNG instead
	- use CSS background  
	```html
	background: url("fallback.png") no-repeat center;
	background-image: url("image.svg");
	background-size: contain;
	```  
	=> older browsers will stick with the PNG, while newer will load the SVG

#### 2. Inlining SVG with `<svg>`
- Pros
	- possible to assign classes, ids to SVG elements and style them with CSS
	- can wrap it in an `<a>` element
	- saves an HTTP request -> reduce loading time a bit
	- the only approach lets you use CSS interactions(`:focus`) and CSS animations on your SVG image
- Cons
	- can causes a lot of duplication
	- extra SVG code increases the size of HTML file
	- cannot cache inline SVG
	- include fallback in a `<foreignObject>` element, but browsers that support SVG still download any fallback images

#### 3. embed an SVG with an `<iframe>`
- definitely not the best method to choose
- Cons
	- `iframe`s have a fallback mechanism, but browsers only displaly the fallback if they lack support fo `iframe`s altogether
	- unless the SVG and webpage have the same origin, you cannot use JavaScript on your main webpage to manipulate the SVG

## Responsive images
### Why responsive images?
- Issues arise when you start to view your site on a narrow screen device
	- 사진이 잘리게 되는데 이미지의 중요한 부분을 보여주는 자른 이미지를 보여줘야함(=art direction problem)
	- 작은 모바일 화면에는 큰 해상도의 이미지가 필요없음(=resolution switching problem)

### How to create responsive images
- CSS arguably has better tools for responsive design than HTML, but we'll be focusing on the HTML `<img>`s

#### 1. Resolution switching: Different sizes
- `<img>`의 `srcset`, `sizes` attribute 사용
- each value : comma-separated list  
	```html
	<img srcset="elva-fairy-480w.jpg 480w,
			 elva-fairy-800w.jpg 800w"
		sizes="(max-width: 600px) 480px,
			800px"
		src="elva-fairy-800w.jpg"
		alt="Elva dressed as a fairy">
	```
- `srcset`
	- `srcset="URL w-descriptor"`
	- w descriptor : the image's intrinsic width in pixels
- `size`
	- `size="Media-condition width-of-the-slot"`
	- media condition : possible state that the screen can be in  
	(e.g., (max-width:600px) means when the viewport width is 600 pixels or less)  
	※ viewport : `<head>`에 정의(`<meta name="viewport" content=width=device-width">`)
	- the width of the slot : 이미지가 채워질 칸의 width
	- the last slot width : default, no media condition
	- similar to switch statement, be careful for the order of the conditions
- browser's process to display : find the width of the slot by compare the viewport with the conditions in the `sizes`, load the image referenced in the `srcset` that size >= the slot size

#### 2. Resolution switching: Same size, different resolutions
- 위 방법에서 w descriptor를 x descriptor로 바꾸고 `sizes`를 없앤 것
- 대신에 img에 `width: 320px;` 내용의 CSS가 필요함
- CSS pixel과 페이지에 접근하는 device의 pixel을 비교해서 적절한 이미지 로드함(if the device has a high resolution of two device pixels per CSS pixel or more, 고화질(2x) 로드
- CSS pixel의 개념을 찾아보고 다시 봐야할듯(CSS pixel은 디바이스에 상관없이 최대 픽셀이 고정인지 잘 모르겠음)

#### 3. Art direction
- resolution switching을 사용할 경우 화면이 너무 작으면 사진을 제대로 알아보기가 어렵다는 문제점이 있음 -> 사진을 브라우저의 해상도별로 다르게 설정
- `<picture>`
	- `<img>`와 비슷하게 사용  
		```html
		<picture>
			<source media="(max-width: 799px)" srcset="elva-480w-close-portrait.jpg">
			<source media="(min-width: 800px)" srcset="elva-800w.jpg">
			<img src="elva-800w.jpg" alt="Chris standing up holding his daughter Elva">
		</picture>
		```
	- `media` : attribute value=media-condition
	- `srcset` : `<img>`처럼 여러 img를 넣을 수 있지만 필요없음
	- fallback contet를 위해 마지막에 `<img>`를 넣어줘야함
	- `<source>`안에 `media`를 넣었을 경우 `sizes`는 넣으면 안됨(`media` : only in art direction scenarios)

#### The reason not to do this with CSS or JavaScript
- browser starts to download any images before the main parser has started to load and interpret the page's CSS and JavaScript -> out of order

#### Modern image formats
```html
<picture>
	<source type="image/svg+xml" srcset="pyramid.svg">
	<source type="image/webp" srcset="pyramid.webp">
	<img src="pyramid.png" alt="regular pyramid built from four equilateral triangles">
</picture>
```