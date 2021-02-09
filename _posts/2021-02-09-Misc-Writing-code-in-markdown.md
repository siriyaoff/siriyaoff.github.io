---
layout: single
title: "Markdown에서 코드 표시(표 안에)"
categories:
  - Miscellaneous
tags:
  - Markdown
---
# Markdown에서 지원하는 문법
- Inline : backtick(\`) 1개로 감싸서 표시  
e.g. `` `code to show` `` = `code to show`

- Code block : backtick(\`) 3개로 감싸서 표시

e.g.  
\`\`\`  
code  
to  
show  
\`\`\`  
=  
```
code
to
show
```

- Inline은 table에서 사용할 수 있다.

```
| ㅁ | ㅁ |
| :--- | :--- |
| ㅁ | `&lt;` |
```

는

| ㅁ | ㅁ |
| :--- | :--- |
| ㅁ | `&lt;` |

와 같음

- **하지만** markdown은 table row에 inline element만 허용하기 때문에 code block과 같은 block element를 넣기 위해서는 `<table>`, `<pre>`를 이용해야함  

# table에서 코드를 표시하는 방법
- 마크다운에서 지원하는 표 문법과 `<br>`과 `` ` ``를 이용해서 코드를 한 줄로 표시하는 방법(언어 명시를 하지 않기 때문에 언어별로 highlighting은 불가능, text indention도 일일히 해야함)
- `<table>`, `<pre>`, `<code>`을 이용해서 아예 HTML의 표를 사용하고 코드를 그대로 표시하는 방법

이렇게 두 가지의 방법이 있다.

## `<br>`, `` ` `` 이용
```
| ㅁ | ㅁ |
| :--- | :--- |
| ㅁ | `code`<br>`to`<br>`show` |
```
는

| ㅁ | ㅁ |
| :--- | :--- |
| ㅁ | `code`<br>`to`<br>`show` |

와 같음

## `<table>`, `<pre>`, `<code>` 이용
```html
<table>
	<tr>
		<th>ㅁ</th>
		<th>ㅁ</th>
	</tr>
	<tr>
		<td>ㅁ</td>
		<td><pre><code>code
to
show
</code></pre></td>
	</tr>
</table>
```
는

<table>
	<tr>
		<th>ㅁ</th>
		<th>ㅁ</th>
	</tr>
	<tr>
		<td>ㅁ</td>
		<td><pre><code>code
to
show
</code></pre></td>
	</tr>
</table>

와 같음

------

- `<pre>`의 attribute `class="lang"`을 이용하면 언어별로 highlighting 가능하다는데 적용이 안되는듯(아래 코드가 class="cpp" 적용한 상태)

<pre class="cpp">#include<&zwj;iostream>
using namespace std;

int main()
{
	cout<<"Hello world!"<<endl;
	return 0;
}
</pre>

## 정리
표가 아닌 곳에서 코드를 보여줘야할 경우는 그냥 마크다운에서 지원해주는 문법을 사용하는 것이 편리하다.(html tag도 섞이지 않고 언어별로 명시만 해주면 알아서 highlighting도 해줌)  
하지만 표 안에 여러 줄의 코드를 넣어야할 경우는 위에서 말한 방법을 이용해야한다. 코드가 짧을 경우 마크다운으로 표를 만들고 `<br>`을 이용하는 편이 낫다. 하지만 코드가 길 경우 어쩔 수 없이 html tag를 사용해야 한다.

이 포스트를 작성하다가 알게되었는데, <&zwj;div>(코드 : `<&zwj;div>`)와 같이 여는 꺽새 뒤에 `&zwj;`를 붙이면 HTML 태그로 인식하지 않는다.(backtick안에 있으면 그대로 표시됨)