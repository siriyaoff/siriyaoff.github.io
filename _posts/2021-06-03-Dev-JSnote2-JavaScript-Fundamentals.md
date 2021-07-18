---
layout: single
title: "JSnote2: JavaScript Fundamentals"
categories:
  - Dev
tags:
  - JavaScript
  - strict mode
  - Data types
  - null
  - undefined
  - Conversion
  - Strict equality
  - Logical operators
  - Function expression
  - Arrow function
  
---

JavaScript Fundamentals
# Hello, world!
`alert`는 browser-specific command임  
Node.js와 같은 서버 환경에서는 `node my.js`와 같은 command로 script 실행 가능

## The "script" tag
HTML 문서 안에는 `<script>` tag를 이용해서 어디든 JS program을 삽입할 수 있음

#### Example
```html
<script>
  alert('Hello, world!');
</script>
```

- `Hello, world!`라는 메시지를 띄움
- `<script>` 태그가 처리될 때 자동으로 실행됨

## Modern markup
지금은 잘 쓰지 않지만 `<script>` 태그에 attributes가 있음
- `type` attribute
	- `type="text/javascript"`가 HTML4에서는 필수였지만, 현재는 아님
	- 최신의 HTML에서는 `type`의 의미를 아예 바꿔서 JS module을 사용할 때 씀
- `language` attribute
	- script의 언어를 명시하지만, 현재는 JS가 default로 설정되어 있어서 안써도 됨

아주 옛날에는 `<script>`를 아래와 같이 사용함  
```html
<script type="tet/javascript"><!--
  ...
//--></script>
```

- JS code를 처리하지 못하는 browsers에 대해서 코드를 숨기기 위해 사용
- 최근 15년 동안에는 이런 이슈가 없었음
	=> 현재는 사용하지 않음

## External scripts
`src` attribute를 이용해서 script files를 넣을 수도 있음

#### Example
```html
<script src="/path/to/script.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.11/lodash.js"></script>
```

- absolute path, relative path, URL 등을 value로 사용할 수 있음
- 여러 파일을 넣기 위해선 `<script>` 태그를 반복해서 사용해야 함

> 간단한 script만 HTML에 넣고 크기가 큰 script는 파일로 참조하는게 나음!  
> 파일로 참조하면 browser의 cache에 저장되기 때문에, 여러 페이지에서 같은 script를 사용하는 상황에서 script를 한 번만 다운로드하고 cache에서 불러옴, 트래픽 ↓

※ `src` attribute가 선언되어 있으면 script content는 무시됨  
따라서 외부 script 하나를 참조하고 HTML 내부에 script 하나를 넣어야 하면 `<script>` 태그를 두 번 사용해야 함  
e.g.  
```html
<script src="file.js"></script>
<script>
  alert(1);
</script>
```
 

# Code structure
## Statements
```javascript
alert('Hello'); alert("World");
```

- string에 대해서 single quotes, double quotes, backtick 모두 사용 가능함

## Semicolons
JS는 line break를 implicit semicolon으로 인식함!(automatic semicolon insertion에 의해)  
따라서 아래의 코드도 정상적으로 작동함:  
```javascript
alert('Hello')
alert('World')
```

하지만 모든 line break가 semicolon으로 interpet되는 건 아님  
e.g.  
```javascript
alert(3+
1
+2)
```

- `6`이 출력됨
- statement가 완전하지 않기 때문에 semicolon을 넣지 않음

#### Example
```javascript
alert("Hello")

[1, 2].forEach(alert);
```

- 예상되는 결과는 `Hello`, `1`, `2`를 순서대로 출력하는 것이지만, 실제로는 세미콜론이 첫 번째 줄의 끝에 삽입되지 않기 때문에 `Hello`까지만 나옴

웬만하면 세미콜론을 넣는게 나음

## Comments
One-line comments : `// Comments`  
Multiline comments : `/* comments */`  
multiline comments의 경우 중첩될 수 없음

대부분 `Ctrl + /`, `Ctrl + Shift + /`로 단축키가 지정되어 있음

주석은 대부분 production server로 publish되기 전에 지워짐

# The modern mode, "use strict"
전에는 호환성 문제가 없어 JS가 업데이트 되어도 이전의 코드를 수정할 필요가 없었음  
하지만 2009년 ECMAScript 5(ES5)에서 기능을 추가하면서 기존의 것을 수정함  
=> 오래된 코드들의 동작을 위해서 수정된 내용들은 기본적으로 반영되지 않은 채로 남겨놓음  
`"use strict"`를 이용해서 변경된 내용을 활성화 해줘야 함

## "use strict"
`"use strict"`도 큰따옴표, 작은따옴표 상관없음  
e.g.  
```javascript
"use strict";
// this code works the modern way
...
```
- 함수 안에서 사용하면 그 함수에서만 strict mode가 적용됨
- 보통 코드의 맨 윗부분에 적어놓음
	- 중간에 적어놓으면 무시되기 때문에 맨 위에 적거나 적지 않거나 둘 중 하나임
- 한 번 적용하면 다시 해제할 수 없음

## Browser console
browser console에서는 strict mode를 바로 적용하면 페이지 전체에 영향을 미침  
아래와 같이 console에서 입력할 내용에만 적용하는게 나음  
```javascript
'use strict'; <Shift + Enter for a newline>
// codes
<Enter to run>
```

오래된 브라우저에서는 아래와 같이 wrapper를 이용해야 함  
```javascript
(function() {
  'use strict';
  
  // codes
})()
```

## Should we "use strict"?
최신의 JS는 class와 module을 지원함  
class, module은 자동으로 strict mode를 활성화시키기 때문에 이것들을 사용하면 `'use strict'`를 코드 안에 쓰지 않아도 됨

이후의 챕터들에서 다양한 기능들에 대해 strict와 old modes에서의 차이점을 알아볼 예정임(수가 적고 웬만하면 strict모드가 더 나음)  
앞으로의 예제들은 모두 strict mode라고 가정함

# Variables
## A variable
변수 : 이름을 가진 저장 공간  
`let` keyword를 이용해서 변수 선언 가능  

#### Example
```javascript
let message='Hello';

alert(message);
```
- `let` 빼고는 cpp와 비슷함
- 오래된 코드에서는 `let` 대신 `var`을 사용함

**multiline styles**  
```javascript
let user='John, age=25, message='Hello';

let user='John';
let age=25;
let message='Hello';

let user='John',
  age=25,
  message='Hello';

let user='John'
  , age=25
  , message='Hello';
```
- JS도 스페이스 2번으로 인덴팅을 하기 때문에 comma-first style이 보기 좋은 듯

## A real-life analogy
변수 개념에 대한 설명

※ 함수형 언어에서는 변수의 재사용이 금지됨

## Variable naming
JS의 변수 이름에 대한 2가지 제한:
1. 변수 이름에는 알파벳, 숫자, `$`, `_`만 들어가야 함
2. 첫 번째 글자는 숫자가 될 수 없음

변수 이름이 길면 보통 camelCase로 지음  
`$`, `_`도 각각 변수 이름으로 사용할 수 있음  
대소문자 구별함!  
non-latin 문자도 가능은 함  
예약된 keyword 존재(`let`, `class`, `return`, `function` 등)

strict mode를 사용하지 않으면 `num=5;`라는 대입 자체가 변수 선언도 해줬음  
strict mode를 사용하면 저 문장은 에러가 남

## Constants
`const`를 사용해서 선언  

### Uppercase constants
대문자와 underscore를 이용해서 상수 이름을 짓고 사용하는게 관행임(진짜 상수값들에 대해서 alias 느낌으로)  
e.g. `const COLOR_RED="F00";`

상수 중에 프로그램이 실행되면서 계산되는 상수들은 정상적으로 이름지음  
e.g. `cost pageLoadTime=/*calculated time*/;`

## Name things right
프로젝트를 진행하면 기존의 코드를 수정하고 확장하는게 대부분이기 때문에 변수 이름을 잘 짓는게 중요함  
몇 가지 명명 규칙들:
- 읽을 수 있는 이름을 사용
- 약칭이나 `a`, `b`, `c`같은 짧은 이름을 피해야 함
- 이름이 최대한 변수에 대해 설명하도록 지어야 함(`data`는 너무 광범위함)
- 사전에 약속된 언어를 사용(사이트에 방문한 사람들을 user라고 정했으면 `currentUser`와 같이)

※ JS minifiers, browser에 의해 코드가 최적화되기 때문에 변수를 재사용하기 보다는 새 변수를 선언하는게 나음(최적화할 때도 코드의 목적을 더 쉽게 파악할 수 있음)

# Data types
JS에는 8가지 기초적인 자료형이 있음  

변수는 여러 자료형을 저장할 수 있음  
아래 코드도 정상적으로 작동함  
```javascript
let message="hello";
message=1234;
```
- 위와 같은 것이 허용되는 것을 dynamically typed라고 함(data type이 존재하지만 변수는 어느 자료형에도 속하지 않음)

## Number
integer, float 등의 값들  
special numeric values도 존재함:
- `Infinity`, `-Infinity` : `1/0`, `Infinity` 등으로 선언 가능
	- 출력하면 Infinity라는 글자 그대로 나옴
- `NaN` : 틀리거나 정의되지 않은 연산을 할 때 나옴
	- `"asdf"/2` 등
	- `NaN`과의 모든 연산 결과는 `NaN`으로 출력됨

※ divide by zero, non-numeric에 대한 수학적 연산 등 일반적으로 허용되지 않는 연산을 하더라도 JS에서는 에러가 나오지 않음

## BigInt
`Number`은 `2^53 - 1` 초과의 수나 `-(2^53 -1)` 미만의 정수를 표현하지 못함  
`BigInt`는 임의의 길이의 정수를 표현하기 위해 최근에 추가된 자료형임  
정수의 끝에 `n`을 붙여서 `BigInt` value를 만들 수 있음

```javascript
const bigInt=123456486974865416541658654648541n;
```
- 현재는 Firefox, Chrome, Edge, Safari 등이 지원함(IE는 지원하지 않음)

## String
JS에서의 `string` 표현 방법 3가지:
1. Double quotes: `"Hello"`
2. Single quotes: `'Hello'`
3. Backticks: `` `Hello` ``

backtick을 이용하면 다른 변수나 식을 string 안에 넣을 수 있음  
아래와 같이 `${...}`의 형식을 이용  
```javascript
let name="John";
alert(`Hello, ${name}!`);
alert(`1+2=${1+2}`);
```

> JS에는 character type이 없음!

## Boolean(logical type)
`boolean`은 `true`, `false` 두 가지 값만 가질 수 있음  
비교문의 결과가 boolean으로 출력됨  
```javascript
let isGreater=4>1;
alert(isGreater);
```

## The "null" value
`null`은 그 자체로 하나의 type임  
JS에서 `null`은 '존재하지 않는 객체로의 참조', 'null pointer'등의 의미를 가지지 않음  
'empty', 'value unknown' 등의 의미로 생각하면 됨  
```javascript
let age=null;
```

## The "undefined" value
`undefined`도 이 자체로 하나의 type임
'값이 대입되지 않음'의 의미로 생각하면 됨  
```javascript
let age;
alert(age);

/* separated scripts */

let age=100;
age=undefined;
alert(age);
```
- 두 script 모두 `undefined`를 출력함
- `null`은 비었다는 뜻으로 사용되고 `undefined`는 선언되었지만 아무런 값이 대입되지 않았을 때 사용하는 default initial value임

## Objects and Symbols
위의 모든 type들은 한 가지의 값만 저장하는 **primitive**임  
반면 `object`는 더 복잡한 데이터를 저장함

`symbol`은 `object`의 identifier를 생성할 때 사용함

## The typeof operator
`typeof` operator를 사용해서 특정한 변수의 type을 알 수 있음  
두 가지 syntax가 존재:
1. operator로 사용 : `typeof x`
2. function으로 사용 : `typoef(x)`

`typeof x`는 type name을 string으로 반환함

#### Example
```javascript
typeof undefined // "undefined"
typeof 0 // "number"
typeof 10n // "bigint"
typeof Math // "object"
typeof null // "object"
typeof alert // "function"
```

- `null`은 *object가 아니지만* 예전의 코드와 호환성을 위해서 `"object"`가 반환됨
- function은 `object` 타입에 속해있지만 `typeof`에서는 `"function"`을 리턴함(마찬가지로 이전의 JS와 호환성을 위해)

# Interaction: alert, promprt, confirm
`alert`, `prompt`, `confirm` 총 3가지의 browser-specific functions를 소개함

## alert
메시지를 띄우고 OK버튼을 누를 때까지 기다림  
메시지를 띄우는 창을 *modal window*라 부름
- *modal*이라는 단어는 유저들이 정해진 조건을 만족하기 전에는 다른 요소들과 상호작용할 수 없는 것을 말함(이 경우에는 OK버튼)

## prompt
```javascript
result=promprt(title, [default]);
```
와 같이 사용됨  
input field, OK, Cancel 두 버튼이 있는 modal window를 띄움  
- `title` : prompt(사용자에게 보여지는 메시지)
- `default` : input field의 초기값, optional

※ `[...]`는 parameter가 optional하다는 것을 나타냄

#### Example
JS:  
```javascript
let age = prompt('How old are you?', 100);

alert(`You are ${age} years old!`); // You are 100 years old!
```

|Result:|
|:---|
|![js-prompt](https://github.com/siriyaoff/MDN-note/blob/master/images/js-prompt.PNG?raw=true)|

- cancel을 누르거나 Esc로 창을 닫으면 `null`이 반환됨
- IE의 경우 default를 선언해놓지 않으면 default가 반환되어야 할 때 `"undefined"`를 반환함
	따라서 `''`와 같은 것을 default로 넣어놓는게 좋음

## confirm
```javascript
result=comfirm(question);
```
와 같이 사용됨  
`question`과 OK, Cancel 두 버튼이 있는 modal window를 띄움  
OK를 누르면 `true`, 아니면 `false` 반환

## Summary
`alert`, `prompt`, `confirm` 모두 modal window에 대해서 위치, 스타일을 수정할 수 없음  
browser에 따라 모두 다름  
chrome에서는 `prompt`가 제대로 작동하지 않음

# Type Conversions
대부분 operator, function들은 자동으로 값을 알맞은 타입으로 변환해줌  
예를 들어 `alert`는 자동으로 값을 `string`으로 바꿔서 출력하고 mathematical operations는 값을 숫자로 바꿈

`object`에 대해서는 나중에 object to primitive conversion을 살펴볼 예정

## String Conversion
`String(value)` function을 이용해서 `string`으로 conversion 가능

## Numeric Conversion
수학 함수나 식 안에서 자동으로 numeric conversion이 일어남  
예를 들어 `alert("6"/"2");`는 3을 출력함(`/`가 수학 연산자이기 때문)

`Number(value)`를 이용해서 명시적으로 conversion 가능  
Numeric conversion rules:

|Value|Becomes...|
|:---|:---|
|`undefined`|`NaN`|
|`null`|`0`|
|`true` and `false`|`1` and `0`|
|`string`|Trim 후 남아있는 string이 없으면 `0`, 아니면 숫자를 읽음, string에 숫자만 있는게 아니면 `NaN` 리턴|

#### Example
```javascript
alert(Number("   123   ")); // 123
alert(Number("123z")); // NaN
alert(Number(true)); //1
```

- `undefined`일 때 `NaN`, `null`일 때 `0`으로 리턴이 다른 것에 주의!!
- 대부분의 수학 연산자도 이러한 conversion을 수행함

## Boolean Conversion
논리 연산자에서 자동으로 일어나고, `Boolean(value)`를 이용해서 명시적으로 가능  
Boolean conversion rules:
- `0`, empty string(`""`), `null`, `undefined`, `NaN`과 같은 비었다는 의미의 값들은 `false`로 변환됨
- 다른 값들은 모두 `true`로 변환됨

※ `"0"`, `" "` 등 unempty string은 모두 `true`로 변환됨에 주의!(PHP에서는 `"0"`이 `false`로 반환됨)

# Basic operators, maths
## Terms: "unary", "binary", "operand"
- *operand* : 피연산자
- *unary* operator : 단항연산자
- *binary* operator : 이항연산자

## Maths
`+`, `-`, `*`, `/`, `%`, `**` 등이 지원됨  
`**` : exponentiation(python이랑 동일, 제곱근도 지원됨)

## String concatenation with binary `+`
```javascript
alert(2 + 2 + '1' ); // "41" and not "221"
alert('1' + 2 + 2); // "122" and not "14"
alert( 6 - '2' ); // 4, converts '2' to a number
alert( '6' / '2' ); // 3, converts both operands to numbers
```
- `+` 연산자의 경우 `string`, `number` type이 둘 다 존재하면 concatenation으로 취급됨
	- 계산 순서에 유의(교환법칙 성립하지 않음!)
- 다른 수학 연산자들은 모두 `string`이 `number`로 conversion된 후 계산됨

## Numeric conversion, unary `+`
`number` 타입에 대해서는 아무런 영향이 없지만, 다른 타입들에 대해선 `number`로 conversion을 실행함  
`Number(...)` function과 동일  
아래와 같이 `string`으로 표현된 숫자들을 더할 때 유용함  
```javascript
let apples="2";
let oranges="3";

alert( apples + oranges ); // "23", the binary plus concatenates strings

// both values converted to numbers before the binary plus
alert( +apples + +oranges ); // 5
```
- unary +가 연산자 우선순위가 더 높음

## Operator precedence
JS에서는 precedence number가 클 수록 우선순위가 큼

## Assignment
### Assignment `=` returns a value
`x=value`는 `x`에 `value`를 대입한 후 그걸 반환함  
따라서 `let c=3-(a=b+1);`와 같은 statement도 가능

### Chaining assignments
대입 연산자를 연속으로 사용하면 오른쪽에서 왼쪽으로 계산됨  
(`a=b=c=2+2;`와 같이)

## Modify-in-place
`+=`, `*=`와 같은 것들

## Increment/decrement
`++`, `--`  

## Bitwise operators
`&`, `|`, `^`, `~`, `<<`, `>>`, `>>>`  
`>>>` : zero-fill right shift

## Comma
`,`를 이용해서 여러 식을 쓸 수 있지만, 모두 계산된 뒤 맨 마지막 것만 리턴됨  
예를 들어 `let a=(1+2, 3+4);`를 실행하면 `a`에는 `7`이 저장됨  
`a=1, b=3, c=a*b`도 가능함(위에는 괄호때문에 대입이 가장 나중에 실행됨)

comma가 우선순위 값이 가장 낮음

## Tasks
- `" \t \n" -2`는 `-2`가 반환됨
	- space characters는 모두 trim되고 난 후 conversion이 일어나기 때문

# Comparisons
## Boolean is the result
모든 비교 연산자들은 `boolean`을 리턴함

## String comparison
string도 비교가 가능함(사전 순으로)  
사전 순으로 뒤쪽에 올수록 더 큰 것으로 취급됨  
`Bee` > `Be` : `true`

※ 사실 unicode 순서이기 때문에 알파벳의 경우 소문자가 더 크다고 판단함

## Comparison of different types
비교하는 값들의 type이 다를 경우 값들을 `number`로 변환해서 비교함  

#### Example
```javascript
alert('2'>1); // true;
alert('01'==1); // true;
alert(false==0); // true;

let a=0;
alert(Boolean(a)); // false

let b='0';
alert(Boolean(b)); // true

alert(a==b); // true
```
- `"0"==0`의 결과가 `true`임에 유의!

## Strict equality
equal 연산자 `==`는 아래의 조건문처럼 `0`, `false`, `''`을 구별하지 못함  
```javascript
alert( 0 == false ); // true
alert( '' == false ); // true
```

A **strict equality operator** `===` checks the equality **without type conversion**  
=> `===`는 비교하는 두 operand의 type이 다를 경우 바로 `false`를 반환함

strict non-equality operator `!==`도 존재함

## Comparison with null and undefined
`null`이나 `undefined`가 비교 대상으로 사용될 때는 예상치 못한 결과가 나올 수도 있음

```javascript
alert(null===undefined); // false
alert(null==undefined); // true
alert(null==0); // false
alert(undefined==0); // false
```
- `==` 연산자를 사용할 때 `null`와 `undefined`는 둘을 비교할 때만 `true`를 반환, 다른 값과 비교할 때는 모두 `false`를 반환함!!
- `<`, `>`, `<=`, `>=`와 같은 다른 비교 연산자들에서는 `null`은 `0`으로, `undefined`는 `NaN`으로 변환되어 비교됨

### Strange result: `null` vs `0`
```javascript
alert( null > 0 );  // (1) false
alert( null == 0 ); // (2) false
alert( null >= 0 ); // (3) true
```
- 위에서 설명했듯이 `null`은 `undefined` 이외의 값과 `==` 연산을 하면 `false`가 나오기 때문에 (2)의 결과가 나옴
- 나머지 비교 연산자에서는 `0`으로 변환된 후 비교되기 때문에 (3)의 결과가 나옴

### An incomoparable undefined
`undefined`는 다른 값들과 비교하면 안됨

```javascript
alert( undefined > 0 ); // false (1)
alert( undefined < 0 ); // false (2)
alert( undefined == 0 ); // false (3)
```
- (3)은 `null`을 넣었을 때와 같은 이유임
- (1), (2)는 `undefined`가 `NaN`으로 변환되어 비교되기 때문임

### Avoid problems
- `undefined` 또는 `null`과의 비교문은 주의해서 사용해야 함!(`===` 제외)
- `null` 또는 `undefined`가 들어갈 수 있는 변수에 대해선 case를 나눠서 처리

## Tasks
- `"2">"12"` : `true`
	- 둘 다 string이기 때문에 number로 변환되어 비교되지 않음
	- `2>"12"`로 바꾸면 `false`가 됨

# Conditional branching: if, '?'
## The "if" statement
## Boolean conversion
## The "else" clause
## Several conditions: "else if"
## Conditional operator `?`
C와 같음

## Multiple `?`
```javascript
let age = prompt('age?', 18);

let message = (age < 3) ? 'Hi, baby!' :
  (age < 18) ? 'Hello!' :
  (age < 100) ? 'Greetings!' :
  'What an unusual age!';
```
- if else와 같음

## Non-traditional use of `?`
```javascript
let company = prompt('Which company created JavaScript?', '');

(company == 'Netscape') ?
   alert('Right!') : alert('Wrong.');
```
- 아예 statement를 넣을 수도 있음
- 가독성이 떨어지기 때문에 알고만 있으면 될 듯

## Tasks
- `string`과 `number`를 비교할 때는 `string`을 캐스팅하지 않아도 됨(`number`이 있어서 자동으로 변환됨)

# Logical operators
JS에는 `||`, `&&`, `!`, `??`, 총 4개의 논리 연산자가 존재함  
이 article에서는 앞 3개만 다룸  
모든 type에 적용 가능하고, 결과 또한 모든 타입이 될 수 있음

## `||`(OR)
모든 operand가 조건문이거나 `boolean`일 경우 계산 후 `true` / `false`를 반환함  
=> C에서와 동일하게 논리 연산자로 사용 가능

## OR `||` finds the first truthy value
```javascript
result = value1 || value2 || value3;
```
- 위의 코드에서 `||`는 다음과 같이 작동함
	- 왼쪽에서 오른쪽 순서로 operand를 계산
	- 순서대로 계산하면서 결과가 `true`면 그 operand의 원래 값을 반환
	- 만약 모든 operand가 계산되면(모든 결과가 `false`면), 마지막 operand를 반환
- operand가 다른 type일지라도 위의 설명처럼 논리 연산자로 사용 가능

boolean-only OR과 비교해서 다음과 같은 용도로 사용됨
1. 변수들과 식들의 리스트 중 첫 번째 truthy를 구함
	```javascript
	let firstName = "";
	let lastName = "";
	let nickName = "SuperCoder";

	alert( firstName || lastName || nickName || "Anonymous"); // SuperCoder
	```
	- 만약 모든 값들이 falsy라면 `"Anonymous"`가 출력됨
2. Short-circuit evaluation
	각 operand들을 순서대로 계산하면서 truthy를 마주치면 바로 그 operand를 반환하는 성질을 이용  
	```javascript
	true || alert("not printed");
	false || alert("printed");
	```
	- 왼쪽 operand가 falsy일 때만 실행되는 statement를 만들 때 사용됨

## `&&`(AND)
operand가 모두 boolean일 때는 C와 동일하게 사용됨

## AND `&&` finds the first falsy value
```javascript
result = value1 && value2 && value3;
```
- 위의 코드에서 `&&`는 위의 `||`와 반대로 작동함
	- 왼쪽에서 오른쪽 순서로 operand를 계산
	- 순서대로 계산하면서 결과가 `false`이면 그 operand의 원래 값을 반환
	- 만약 모든 operand가 계산되면(모든 결과가 `true`면), 마지막 operand를 반환
- 마찬가지로 논리연산자로 사용 가능(애초에 falsy가 반환되면 조건문 실행이 안됨)

#### Example
```javascript
alert( 1 && 2 && null && 3 ); // null
alert( 1 && 2 && 3 ); // 3, the last one
```

※ `&&`가 `||`보다 우선순위가 높음

아래 코드와 같이 `if`문을 `||`, `&&`를 이용해서 대신할 수 있지만 각각의 목적에 맞게 사용하는게 가독성을 높여줌  
```javascript
let x = 1;

(x > 0) && alert( 'Greater than zero!' );
```

## `!`(NOT)
C와 같음

아래와 같이 `!!`를 이용해서 값을 `boolean`으로 바꿀 수 있음  
```javascript
alert( !!"non-empty string" ); // true
alert( !!null ); // false
```
- `Boolean(...)`를 사용하는 것보다 간단함
- `!`는 모든 논리 연산자 중에 가장 우선순위가 높음

## Tasks
- `alert` 함수는 아무런 값을 반환하지 않기 때문에 operand로 사용될 경우 `undefined`를 반환함

```javascript
alert(alert(1) && alert(2));
```
- 위 코드를 실행하면 `1`이 출력된 후 `undefined`가 출력됨
	- `alert`가 `undefined`를 반환하기 때문에 `1`을 출력하고 `alert`의 리턴값을 반환하고 끝남

# Nullish coalescing operator `??`
`null`이나 `undefined`가 아니면 "defined"라고 부름  
`a ?? b`:
- `a`가 defined => `a` 리턴
- `a`가 defined가 아님 => `b` 리턴

⇔  
```javascript
(a !== null && a !== undefined) ? a : b
```

아래 코드와 같이 사용 가능함  
```javascript
let firstName = null;
let lastName = null;
let nickName = "Supercoder";

// shows the first defined value:
alert(firstName ?? lastName ?? nickName ?? "Anonymous"); // Supercoder
```

## Comparison with `||`
위 예시는 `||`로 대체 가능함  
`||`는 JS가 개발될 때부터 존재했고, `??`는 최근에 생김  
두 연산자의 차이점은 아래와 같음
- `||` : first *truthy* value 리턴
	- `false`, `0`, `""`, `null/undefined`를 구별하지 않음
- `??` : first *defined* value 리턴
	- `null/undefined`를 다른 falsy value와 구별함  
		=> 값이 진짜로 비어있거나 설정되지 않았는지 판단 가능

#### Example
```javascript
let height = 0;

alert(height || 100); // 100
alert(height ?? 100); // 0
```
- 위 예시에서는 값이 0으로 설정되었기 때문에 `??`를 이용하는 것이 올바른 결과를 출력함
- `a=a?100;`와 같이 변수의 초깃값을 설정할 때 사용 가능

## Precedence
`??`(5)는 `||`(6)보다 우선순위가 1단계 낮음(괄호 안은 precedence value, 높을수록 우선순위가 높음)  
=> `=`, `?` 전에, `+`, `*` 등 보다 뒤에 계산됨  
=> `let area = (height ?? 100) * (width ?? 50)`와 같이 사용해야 함

### Using `??` with `&&` or `||`
JS에서는 `??`를 `&&`나 `||`와 같이 사용하는 것을 금지함(괄호로 묶어서 명시적으로 우선순위를 정했을 때는 가능)  
즉 아래의 두 번째 줄과 같이 사용해야 함

```javascript
let x = 1 && 2 ?? 3; // Syntax error
let x = (1 && 2) ?? 3; // Works
```
- `??`가 생기고나서 사람들의 혼동을 방지하기 위해 이런 제한이 생김

# Loops: while and for
## The "while" loop
## The "do...while" loop
## The "for" loop
C와 같음

## Continue to the next iteration
ternary operator `?`를 사용할 때 expression이 아닌 것들은 사용할 수 없음  
예를 들어 아래와 같은 코드는 syntax error가 남

```javascript
(i > 5) ? alert(i) : continue; // continue isn't allowed here
```
- `if`대신 `?`를 사용하면 안되는 이유 중 한가지임

## Labels for break/continue
label을 이용해서 break/continue할 위치를 정할 수 있음

```javascript
outer:
for (let i = 0; i < 3; i++) {

  for (let j = 0; j < 3; j++) {

    let input = prompt(`Value at coords (${i},${j})`, '');

    // if an empty string or canceled, then break out of both loops
    if (!input) break outer; // (*)

    // do something with the value...
  }
}
alert('Done!');
```
- `break <labelName>`을 사용하면 `<labelName>`이 붙어 있는 곳을 빠져나감
- `continue`도 label을 동일하게 적용 가능

label이 모든 곳으로 jump시켜주는 것은 아님!  
아래 코드와 같이 사용할 수 없음  
```javascript
break label; // jump to the label below (doesn't work)

label: for (...)
```

`break`는 code block안에 위치해야 함!!  
아래와 같이 반복문 없이도 사용 가능함(`continue`는 불가능)  
```javascript
label: {
  // ...
  break label; // works
  // ...
}
```

## Tasks
- `(i%2)||alert(i);`로 짝수 출력 가능
- `prompt`에서 사용자가 cancel을 누르면 `null`이 들어감  
	=> `null`은 `0`으로 변환되는 것에 주의!!!

# The "switch" statement
## The syntax
## Grouping of "case"
C와 같음

## Type matters
`case a:`와 같이 equality check일 때는 항상 strict equality operator `===`를 사용함!!  
따라서 `switch`의 변수와 `case`의 비교 대상의 type이 다르면 실행이 되지 않음(`3`, `"3"`과 같이)

# Functions
## Function Declaration
## Local variables
## Outer variables
C와 같음

## Parameters
Call by value가 적용됨

## Default values
아래와 같이 parameter에 default value 선언 가능  
```javascript
function showMessage(from, text = anotherFunction()) {
  // anotherFunction() only executed if no text given
  // its result becomes the value of text
}
```

### Alternative default parameters
함수 내부에서 아래와 같이 기본값을 선언 가능함  
```javascript
// 1
if (text === undefined) {
  text = 'empty';
}

// 2
text = text || 'empty';

// 3
text = count ?? 'empty'
```
- 상황에 따라 맞게 설정해야 함  
	예를 들어 0도 입력으로 들어올 수 있는 경우 `||` 대신 `??` 사용해야 함

## Returning a value
`return;` 또는 `return`을 사용하지 않으면 `undefined`를 리턴함

```javascript
return
 (some + long + expression + or + whatever * f(a) + f(b))
```
- 위와 같이 `return` 다음 줄을 바꾸고 리턴값을 적으면 `return;`와 같음
	=> 긴 식을 리턴하고 싶을 때는 `return (`와 같이 괄호를 `return`과 같은 줄에 적어야 함

## Naming a function
## Functions == Comments
함수가 복잡한 구현이 필요할 경우 여러 함수들로 세분화하는게 나음  
=> 재사용이 많이 되지 않아도 코드의 가독성을 위해서 함수를 선언하는게 나음

## Tasks
- `?`에는 `return`이 operand로 들어갈 수 없음
- `**` 연산자는 소숫점도 지원됨

# Function expressions
JS에서 function은 값으로도 취급됨  
Function Expression : `function() {...}`을 expression처럼 사용하는 것  
아래와 같이 *Function Expression*을 사용 가능함  
```javascript
let sayHi = function() {
  alert( "Hello" );
};

alert(sayHi); // shows the function code
```
- 다른 값들처럼 출력도 가능함
	- `sayHi()`로 함수가 호출된게 아니기 때문에 함수가 실행되지 않음
- function declaration이 아니라 function expression으로 선언할 경우 맨 뒤에 `;`가 붙는 것에 주의!!

함수가 값으로 취급될 수 있기 때문에 아래와 같이 복사도 가능함  
```javascript
function sayHi() {   // (1) create
  alert( "Hello" );
}

let func = sayHi;    // (2) copy

func(); // Hello     // (3) run the copy (it works)!
sayHi(); // Hello    //     this still works too (why wouldn't it)
```
- `func==sayHi`는 `true`임
	(∵ sayHi의 코드를 다 복사하기 때문에 같음)

## Callback functions
함수 a의 parameter로 다른 함수 b, c를 넣어서  
함수 b, c가 함수 a에서 필요시 실행될 수 있음  
이 때 함수 b, c를 함수 a의 *callback functions* 또는 *callbacks*라고 함

#### Example
```javascript
function ask(question, yes, no) {
  if (confirm(question)) yes()
  else no();
}

function showOk() {
  alert( "You agreed." );
}

function showCancel() {
  alert( "You canceled the execution." );
}

// usage: functions showOk, showCancel are passed as arguments to ask
ask("Do you agree?", showOk, showCancel);
```
- JS에서 function은 *action*을 나타낸다고 생각하면 됨(cf. value는 *data*를 나타냄)  
	=> variable처럼 다루고 필요할 때 실행시킬 수 있음

아래와 같이 function expression으로 구현할 수도 있음

```javascript
function ask(question, yes, no) {
  if (confirm(question)) yes()
  else no();
}

ask(
  "Do you agree?",
  function() { alert("You agreed."); },
  function() { alert("You canceled the execution."); }
);
```
- 더 간단한게 구현 가능
- `ask`를 호출하는 statement 안에서 선언되었고, 이름도 없기 때문에 밖에서 사용할 수 없음(*anonymous*라고 부름)

## Function Expression vs Function Declaration
syntax 이외에도 JS engine에 의한 함수 생성 시점이 다름
- Function Expression은 statement를 실행할 때 생성되고 그때부터 호출 가능함
- Function Declaration은 선언부보다 위에서도 호출 가능함

Function Declaration은 명시된 scope도 중요함
- block scope 선언되었다면 그 block 안에서만 호출 가능함

Function Expression을 이용해서 아래와 같이 Declaration으로는 불가능한 것을 구현할 수 있음  
```javascript
let welcome = (age < 18) ?
  function() { alert("Hello!"); } :
  function() { alert("Greetings!"); };

welcome(); // ok now
```

> Function Declaration과 Function Expression을 선택하는 방법:  
> 가능하면 Function Declaration을 사용하는게 나음(가독성 ↑, 가용성 ↑)  
> conditional declaration이 필요하거나 다른 이유로 Function Declaration이 맞지 않을 경우 Function Expression을 사용하면 됨

# Arrow functions, the basics
아래 두 코드는 같은 함수를 만듦  
```javascript
let func = (arg1, arg2, ..., argN) => expression;

let func = function(arg1, arg2, ..., argN) {
  return expression;
};
```
- parameter가 없어도 `()`로 표시해줘야 함
- Function Expression처럼 value로 사용 가능
	```javascript
	let welcome = (age < 18) ?
	  () => alert('Hello') :
	  () => alert("Greetings!");

	welcome();
	```
- one-line action을 구현할 때 효율적임

## Multiline arrow functions
curly braces로 expression을 여러 줄로 늘릴 수 있음  
```javascript
let sum = (a, b) => {  // the curly brace opens a multiline function
  let result = a + b;
  return result; // if we use curly braces, then we need an explicit "return"
};
```
- `{}`를 사용하면 반드시 `return`이 있어야 함