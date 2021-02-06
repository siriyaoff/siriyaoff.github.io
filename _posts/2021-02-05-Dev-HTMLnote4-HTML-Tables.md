---
layout: single
title: "HTMLnote4: HTML Tables"
categories:
  - Dev
tags:
  - HTML
  - MDN
  - HTML tags
  - Accessibility
---
# HTML Tables
## HTML table basics
- When should you NOT use HTML tables?
	- a lot of people use HTML tables to lay out web pages(one row for header, ...)
	- it was because CSS support across browsers used to be terrible(they are much less nowadays)
	- Using tables for layout rather than CSS layout techniques is a bad idea
		- layout table reduce accessibility for visually impaired users
		- tables produce tag soup
		- tables are not automatically responsive : `<header>`, `<section>`, `<div>`같은 것들은 부모 element의 100%로 너비가 설정되지만 table은 담기는 content에 따라서 너비가 설정되기 때문에 layout으로 쓸 경우 기기별로 너비를 설정해야해서 번거로움

- elements of `<table>`
	- `<tr>` : table row
	- `<td>` : table data(empty cell : `<td>&nbsp;</td>`)
	- `<th>` : table header(works in exactly the same way as a `<td>`)
		- style : bold, centered
		- `scope` attribute로 한번에 묶을 수 있음

- Allowing cells to span multiple rows and columns
	- `<td>`, `<th>`의 `colspan`, `rowspan` attributes 이용  
	e.g. `<th colspan="2">Animals</th>` -> spanning two columns

- Providing common styling to columns
	- an entire column에 대해 스타일을 지정하려면 CSS의 `:nth-child`를 사용하거나 `<td>`, `<th>`에서 일일히 지정해줘야했음 -> `<col>`, `<colgroup>` to define styling information for a column
	- limited to a few properties: `border`, `background`, `width`, `visibility` for other properties, style every `<td>` or `<th>`, or use complex selector  
	- 모든 column이 있어야함, 스타일링이 필요하지 않은 col의 개수가 많을 경우 span을 이용해 생략 가능(`<col span="2">` 이런 식으로)  
		```html
		<table>
			<colgroup>
				<col>
				<col style="background-color: yellow">
			</colgroup>
			<tr>
				<th>Data 1</th>
				<th>Data 2</th>
			</tr>
			<tr>
				<td>Calcutta</td>
				<td>Orange</td>
			</tr>
			<tr>
				<td>Robots</td>
				<td>Jazz</td>
			</tr>
		</table>
	```
	- If we wanted to apply the styling information to both columns, we could just include one `<col>` element with a span attribute on it like `<col style="background-color: yellow" span="2">`
	- `style`에서 `border`의 굵기를 1px로 설정하면 안보임, 최소 2px, solid : 실선  
	e.g. `style="border: 2px solid #000000;"`
	- `table`, `td` 모두 border설정해서 테두리가 겹칠 경우 `border-collapse="collapse"` 넣어서 하나 없앨 수 있음

## HTML table advanced features and accessibility
- Adding a caption to table with `<caption>`
	- nest `<caption>` directly beneath the `<table>` tag  
		```html
		<table>
			<caption>Dinosaurs in the Jurassic period</caption>
			...
		</table>
	```
	- `<table>`의 `summary` 속성도 같은 역할이지만 HTML5에서 잘 쓰이지 않고 페이지에 나타나지 않음

- Adding structure with `<thead>`, `<tfoot>`, and `<tbody>`
	- these don't make the table any more accessible and don't result in any visual enhancement on their own but userful for styling and layout-acting as useful hooks for adding CSS
	- `<thead>`
		- wrap the header part of the table
		- if you're using `<col>`/`<colgroup>`, `<thead>` should come just below those
	- `<tfoot>`
		- wrap the footer part of the table
	- `<tbody>`
		- wrap the other parts of the table
		- always included in every table(if you don't specify it, browser would add it for you)
	- `<thead>`, `<tfoot>`은 자동으로 안의 내용이 위/밑에 위치하게됨
	- header가 하나의 열일 경우 `<col>`을 이용해 스타일링

- Nesting tables
	- `<td>`안에 그냥 집어넣으면 됨, 근데 nesting하면 table이 여러 개가 되니까 id를 사용하는게 나음(`<td id="nested">`이런 식으로))
	- 최선은 nesting table을 사용하지 않는 것

- Tables for visually impaired users
	- Using column and row headers(`<th>`)
	- The `scope` attribute
		- `<th>`에 대해 header for row인지 column인지 정의함  
		e.g. `<th scope="row">Haircut</th>`
		- `row`, `col` 이외에도 `colgroup`, `rowgroup`도 value에 넣을 수 있음(col/row가 span되어있을 경우 사용)
	- The id and headers attributes
		- 헤더인 `<th>`에 `id="asdf"` attribute 추가, 그 헤더에 속하는 `<td>`에 `headers="asdf"` 추가
		- 여러 header에 속할 수도 있는데 이때 id는 space로 구분
	- usually `scope` approach is enough for most tables