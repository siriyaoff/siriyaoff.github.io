---
layout: single
title: "JSnote5: Data types"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - JavaScript
  - Primitives
  - Object
  - String
  - Array
  - Iterable
  - Map
  - Set
  - WeakMap
  - WeakSet
  - Destructuring
  - Date
  - JSON
---

Data types
# Methods of primitives
primitive와 object의 차이점:
- A primitive
	- primitive type의 값임
	- 7개의 type이 존재 : `String`, `Number`, `Bigint`, `Boolean`, `Symbol`, `null`, `undefined`
- An object
	- 여러 개의 값을 property로 저장할 수 있음
	- `{}`로 생성할 수 있고 JS에는 함수와 같은 여러 종류의 object가 있음
	- 함수도 property로 저장(method)할 수 있음
	- primitive보다 자원을 많이 소모함

## A primitive as an object
primitive를 method와 함께 사용하기 위해 아래와 같은 방법을 사용함:
- `String`, `Number`, `Boolean`, `Symbol`의 method와 property에 접근하는 것을 허용함
- 위 접근을 가능하게 하기 위해, *object wrapper*를 제공함(임시적으로 생성, 소멸됨)

object wrapper는 각각의 primitive type에 따라 다르며, primitive의 이름과 같음(`String`, `Number`, `Boolean`, `Symbol`)  
각각 다른 methods를 제공함

#### Example
string method `str.toUpperCase()`는 `str`를 capitalize한 것을 반환함:  
```javascript
let str = "Hello";

alert( str.toUpperCase() ); // HELLO
```
이 method를 호출했을 때 일어나는 일:
1. `str`은 primitive이기 때문에, property에 접근하는 순간 string의 값과 `toUpperCase()` 등의 method를 가진 객체가 생성됨
2. method가 실행되고 새로운 string이 반환됨
3. 객체가 파괴되고 primitive인 `str`만 남음

이렇게 primitive가 method를 제공하면서 가볍게 유지됨  
JS 엔진은 추가적인 객체를 생성하지 않을 만큼 최적화되어 있지만 명세서에는 그것을 생성하는 것처럼 적혀있음

`Number`에는 `toFixed(n)`과 같은 method가 존재함  
`alert( n.toFixed(2) );` : 소수점 `n`자리까지 남도록 반올림함

> ※ `String/Number/Boolean`을 생성자로는 사용하지 않는 게 좋음  
> Java와 같은 언어에서는 명시적으로 wrapper object를 생성할 수 있음(`new Number(0)`과 같이)  
> JS에서도 가능은 하지만 크게 아래의 이유로 추천되지 않음:  
> - primitive가 아닌 object로 인식되기 때문에 조건문에서 값에 상관없이 참을 반환함
>
> 반면 `new`를 사용하지 않고 `String/Number/Boolean`만 쓰는 것은 값을 원하는 type으로 변환하게 도와주기 때문에 유용함  
> e.g. `let num = Number("123");`

> ※ `null/undefined`는 method가 없음  
> 위 type들은 wrapper object가 없음  
> `alert(null.test);`와 같이 property에 접근하려 하면 에러가 발생함

## Summary
- primitive를 다룰 때 유용한 기능들이 각각의 primitive 안에 있음
	- method로 구현된 기능을 호출하면 임시적으로 object wrapper라는 객체가 생성되고, method가 실행된 뒤 파괴됨

## Tasks
### primitive가 method를 호출하면 객체로 처리되는데, property를 추가할 수 있을까?  
```javascript
let str = "Hello";

str.test = 5;

alert(str.test);
```
- strict mode의 사용 여부에 따라 결과가 다름
	- strict mode를 사용하지 않으면 `undefined`가 출력됨
	- strict mode를 사용하면 error가 발생함

- property에 접근할 때 wrapper object가 생성은 됨!
	- strict mode에서는 값을 수정할 수 없음
	- strict mode가 아닐 경우, 객체가 `test` property를 가지지만, 그 statement가 끝난 후 사라짐!  
		따라서 마지막 줄에서 `str`은 `test`라는 property를 찾을 수 없음!
	
	primitive와 object의 차이점 중 하나임:  
	primitive는 추가적인 데이터를 저장할 수 없음!!

# Numbers
최신의 JS에는 두 가지 타입의 숫자가 존재함:
1. 일반적인 숫자는 64-bit format IEEE-764(double precision floating point number)으로 저장됨
2. 임의의 길이의 숫자를 표현하기 위해서 BigInt가 사용됨  
	일반적인 숫자는 `[-2^53, 2^53]`의 수만 표현 가능하기 때문

이 article에서는 일반적인 숫자에 대해 다룰 예정

## More ways to write a number
underscore `_`를 숫자 사이에 구분자로 넣을 수 있음:  
```javascript
let billion = 1_000_000_000;
```
- 숫자의 가독성을 높여줌
- JS 엔진은 숫자 자리수 사이에 사용된 `_`는 무시함

`"e"`를 지수승 표기로 사용할 수 있음:
```javascript
let billion = 1e-6;  // 0.000001

alert( 7.3e9 );  // 7300000000
```

### Hex, binary and octal numbers
`0x`, `0b`, `0o` for hex, binary, octal numbers  
case doesn't matter  
for other numeral systems, we should use the function `parseInt`

## toString(base)
`num.toString(base)`를 사용하면 숫자를 `base`진법으로 변환한 결과를 string으로 반환함  
```javascript
let num = 255;

alert( num.toString(16) );  // ff
alert( 123456..toString(36) ); // 2n9c
```
- `base`는 `[2, 36]` 내의 숫자만 가능
- 마지막 줄의 `..`은 method를 호출하기 위해 사용한 것임  
	점을 하나만 쓰면 소수점이라 인식하기 때문에 method를 바로 호출하려면 `..`을 사용해야 함  
	`(1234).toString(36)`과 같이 괄호로 묶어도 됨

## Rounding
- `Math.floor` : rounds down
- `Math.ceil` : rounds up
- `Math.round` : rounds to the nearest integer
- `Math.trunc` : removes anything afer the decimal point

위 함수들의 계산 결과는 모두 integer임  
소수점 이하 n자리까지 나오게 계산하고 싶다면?
1. 수학적 조작(2자리까지 나오게 하고싶다면 100을 곱한 값을 함수에 넣음)
2. `toFixed(n)` 사용(자동으로 반올림됨)
	- 결과는 `String`이기 때문에 unary plus로 변환해줘야 함

## Imprecise calculations
64비트 IEEE-754에서 52비트는 가수(fraction), 11비트는 지수(exponent), 1비트가 부호로 사용됨
- 숫자가 너무 크면 `Infinity`로 바뀜
- 정밀도 손실이 일어남(`0.1 + 0.2 == 0.3`은 `false`임)  
	(+ `alert( 9999999999999999 ); // shows 10000000000000000`)  
	부동 소수점으로 저장하기 때문  
	해결 방법:
	1. 소수들을 계산할 동안만 정수로 바꿔서 계산
		- 값이 크지 않을 때 사용해야 함
		- 다시 소수점을 돌려놓는 과정에서 에러가 발생하기 때문에 오차를 줄이는 정도임
	2. `toFixed(n)`을 사용해서 오차를 표현하지 않으면 됨  
		e.g. `alert( sum.toFixed(2) );`

> ※ 부동소수점 덕분에 두 개의 0이 존재함(`0`, `-0`)

## Tests: isFinite and isNaN
`isNaN(value)`는 `value`가 `NaN`이면 `true`를 리턴함:  
```javascript
alert( isNaN(NaN) ); // true
alert( isNaN("str") ); // true

alert( NaN === NaN ); // false
```
- 마지막 줄처럼, `NaN`은 자신을 포함해서 모든 것들과 같지 않기 때문에 `isNaN()` 함수가 필요함

`isFinite(value)`는 `value`를 숫자로 변환하고 일반적인 숫자면 `true`, `NaN/Infinity/-Infinity`면 `false`를 반환함:  
```javascript
alert( isFinite("15") ); // true
alert( isFinite("str") ); // false, because a special value: NaN
alert( isFinite(Infinity) ); // false, because a special value: Infinity
```
- 빈 문자열이나 공백, `null`은 `0`으로 취급되기 때문에 `true`를 반환함

> ※ `Object.is`는 `===`와 비슷한 기능의 내장 메소드지만, 아래 두 가지 edge case에서 더 신뢰할만한 결과를 반환함:
> 1. `Object.is(NaN, NaN) === true`
> 2. `Object.is(0, -0) === false`
> 
> 위와 같은 비교 방식은 specification에서 SameValue라고 불림

## parseInt and parseFloat
인자의 시작부터 변환 가능한 문자까지만 변환해서 리턴, 변환할 수 없다면 `NaN` 리턴:  
```javascript
alert( parseInt('100px') ); // 100
alert( parseFloat('12.5em') ); // 12.5

alert( parseInt('12.3') ); // 12, only the integer part is returned
alert( parseFloat('12.3.4') ); // 12.3, the second point stops the reading

alert( parseInt('a123') ); // NaN, the first symbol stops the process
```
- `parseInt(str, radix)`로도 사용 가능, `radix` 진법으로 `str`를 읽고 숫자로 변환함  
	e.g. `alert( parseInt('2n9c', 36) ); // 123456`

## Other math functions
- `Math.random()` : returns a random number in `[0, 1)`
- `Math.max(a, b, c...)` / `Math.min(a, b, c...)` : returns the max / min
- `Math.pow(n, power)` : returns `n^pow`

## Summary

| code                                                                | description                           |
| :------------------------------------------------------------------ | :------------------------------------ |
| `num.toString(base)`                                                | `num`을 `base`진법의 문자열으로 변환  |
| `Math.floor()`<br>`Math.ceil()`<br>`Math.round()`<br>`Math.trunc()` | 소수점 처리                           |
| `num.toFixed(n)`                                                    | 소수점 이하 `n`자리까지 남도록 반올림 |
| `isNaN(val)`                                                        | `val`이 `NaN`인지 판별                |
| `isFinite(val)`                                                     | `val`이 finite한 수인지 판별          |
| `Object.is(v1, v2)`                                                 | `v1`, `v2`가 SameValue인지 판별       |
| `parseInt(str[, radix])`                                            | `str`을 `radix` 진법으로 파싱함       |
| `parseFloat(str)`                                                   | `str`을 소수점까지 파싱함             |
| `Math.random()`                                                     | `[0, 1)` 범위의 임의의 수 반환        |
| `Math.max(a, b, c...)`<br>`Math.min(a, b, c...)`                    | max, min 반환                         |
| `Math.pow(n, pow)`                                                  | `n^pow` 반환                          |

- SameValue는 `===`과 비슷하지만, `(NaN, NaN)`은 같고, `(0, -0)`은 다름

## Tasks
- `num.toFixed(n)`는 소수점 이하 `n`까지 나오도록 실제 값을 반올림하기 때문에 오차가 발생 가능함  
	```javascript
	alert( 6.35.toFixed(1) ); // 6.3
	alert( 6.35.toFixed(20) ); // 6.34999999999999964473
	
	alert( Math.round(6.35 * 10) / 10); // 6.35 -> 63.5 -> 64(rounded) -> 6.4
	```
	따라서 `n` 번째 자리까지 구하기 위해선 마지막 줄같이 소수점 이하 `n` 번째 자리를 소수점 이하 1 번째 자리로 만든 후 `Math.round()`를 사용하는게 좋음
- `prompt`에서 취소를 누르면 `null`이 입력되는 것에 유의!!!

# Strings
`String`의 포맷은 항상 **UTF-16**임!(page encoding과는 연관이 없음)

## Quotes
quotes 중 backtick(`` ` ` ``)은 안에 변수를 포함(`${}`)하거나 줄바꿈을 할 수 있음  
추가로 backtick을 이용하면 template function을 사용할 수 있음:
- ``func`string` ``으로 사용
- string과 내장된 식을 받아서 처리할 수 있음([tagged template](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#tagged_templates)이라고도 불림)
- 자동완성같은 기능으로, 실무에서는 잘 사용하지 않는 듯

## Special characters

| Character                                   | Description                                                                            |
| :------------------------------------------ | :------------------------------------------------------------------------------------- |
| `\n`                                        | New line                                                                               |
| `\r`                                        | Carriage return<br>window에서는 line break가 `\r\n`으로 표현됨                         |
| `\'`, `\"`                                  | Quotes                                                                                 |
| `\\`                                        | Backslash                                                                              |
| `\t`                                        | Tab                                                                                    |
| `\b`, `\f`, `\v`                            | Backspace, Form feed, Vertical tab<br>호환성을 위해서 유지되지만, 현재는 사용되지 않음 |
| `\xXX`                                      | Unicode character with the given hexadecimal Unicode `XX`<br>e.g. `\x7A` = `'z'`       |
| `\uXXXX`                                    | Unicode symbol with the hex code `XXXX` in UTF-16 encoding                             |
| `\u{X...XXXXXX}`<br>(1 to 6 hex characters) | Unicode symbol with the given UTF-32 encoding                                          |

- escape character `\`를 이용해서 특수문자를 표현

## String length
`length` property를 사용해서 구함:  
```javascript
alert( `My\n`.length ); // 3
```

## Accessing characters
1. square brackets `[]`
	- `str[0]`
	- 해당하는 문자가 없으면 `undefined` 리턴
2. `str.charAt(pos)`
	- `str.charAt(pos)`
	- 해당하는 문자가 없으면 empty string `''` 리턴
3. iterating
	- `for (let char of "Hello")`

## Strings are immutable
`String`의 일부를 수정할 수는 없음  
`String` 전체를 바꿀 수는 있음:  
```javascript
let str = 'Hi';
str[0] = 'h'; // doesn't work
alert( str[0] ); // H

str = 'h' + str[1]; // replace the string
alert( str ); // hi
```

## Changing the case
`toLowerCase()`, `toUpperCase()` method를 사용:  
```javascript
alert( 'Interface'.toUpperCase() ); // INTERFACE
alert( 'Interface'.toLowerCase() ); // interface
alert( 'Interface'[0].toLowerCase() ); // 'i'
```
- 한 문자에 대해서도 사용 가능

## Searching for a substring
### str.indexOf
`str.indexOf(substr[, pos])` : `substr`을 `str`의 `pos` 번째 문자부터 찾아나감  
일치하는 부분이 있을 경우 시작하는 위치를 리턴, 없을 경우 `-1` 리턴:  
```javascript
let str = 'Widget with id';

alert( str.indexOf('Widget') ); // 0
alert( str.indexOf('widget') ); // -1
alert( str.indexOf('id', 2) ) // 12
```
- 대소문자 구별함
- `str.lastIndexOf(substr[, pos])`도 있음
	- `str`의 마지막부터 `substr`을 찾아나감

### The bitwise NOT trick
JS에서 bitwise NOT `~`는 숫자를 32-bit 정수로 변환한 다음 NOT 연산을 실행함  
`~n`이 `-(n+1)`과 같다는 점을 이용해서 `indexOf`로 부분문자열을 찾지 못했을 때 조건을 간단히 할 수 있음  
=> `if (~str.indexOf("Widget"))`

큰 숫자는 잘리고, `~(2^31-1)`도 `0`이기 때문에 긴 문자열에 대해서는 사용할 수 없음  
최신의 JS에서는 `.includes` method를 이용함

### includes, startsWith, endsWith
`str.includes(substr[, pos])`는 `str`에 `substr`이 들어있는지 판별함  
=> `true/false` 리턴

`str.startsWith(substr)`, `str.endsWith(substr)`는 `str`이 `substr`로 시작하거나, 끝나는지 판별함  
=> `true/false` 리턴

```javascript
alert( "Widget".startsWith("Wid") ); // true
alert( "Widget".endsWith("get") ); // true
```

## Getting a substring
1. `str.slice(start[, end])`
	- `str`의 `[start, end)` 부분문자열을 반환  
		`end`가 없을 경우 `start`부터 끝까지 포함한 부분문자열을 반환
	- `start`, `end`에 음수도 넣을 수 있음
2. `str.substring(start[, end])`
	- `str`의 `start`와 `end` 사이를 포함한 부분문자열을 반환  
		`slice`와 비슷하지만, `start`가 `end`보다 클 수 있음:  
		```javascript
		let str = "stringify";

		// these are same for substring
		alert( str.substring(2, 6) ); // "ring"
		alert( str.substring(6, 2) ); // "ring"

		// ...but not for slice:
		alert( str.slice(2, 6) ); // "ring" (the same)
		alert( str.slice(6, 2) ); // "" (an empty string)
		```
	- Negative arguments가 허용되지 않음
3. `str.substr(start[, length])`
	- `str`의 `start`부터 시작하는, 길이가 `length`인 부분문자열을 반환
	- `start`는 음수를 넣을 수 있음

> ※ `substr`은 JS specification에 나와있지 않아서 non-browser 환경에서는 지원하지 않을 수도 있음  
> 웬만하면 다 지원됨  
> `slice`는 음수도 인자로 넣을 수 있기 때문에 `substr`보다 유연함  
> 사실상 `slice`만 기억해도 충분함

## Comparing strings
문자끼리는 알파벳 순서로 비교 가능함  
ASCII를 이용하지 않고 UTF-16을 이용!

`str.codePointAt(pos)` : `str`의 `pos` 번째 문자의 코드 반환  
`String.fromCodePoint(code)` : `code`에 해당하는 문자 반환

#### Example
```javascript
alert( "z".codePointAt(0) ); // 122
alert( "Z".codePointAt(0) ); // 90

alert( String.fromCodePoint(90) ); // Z
alert( '\u005a' ); // Z(0x005a == 90)
```
- `\u` prefix를 사용해서 UTF-16 코드를 직접 입력할 수 있음

### Correct comparisons
다른 언어들끼리 비교하기 위해서 ECMA-402 안의 `str.localeCompare(str2)` 함수를 사용하면 됨
- `str`이 `str2`보다 작으면 음수 리턴
- `str`이 `str2`보다 크면 양수 리턴
- 둘이 같으면 `0` 리턴

> ※ default로 diacritical marks(Ö와 같은 발음 구별 부호)가 알파벳(O와 같이)과 동일하게 처리됨

> ※ IE10 이하는 Intl.js라는 라이브러리가 필요함

## Internals, Unicode
### Surrogate pairs
자주 사용되는 문자들은 2-byte code에 저장되지만, 잘 쓰이지 않는 문자들은 추가적으로 2바이트를 더해 surrogate pair로 인코딩됨:  
```javascript
alert( '𝒳'.length ); // 2, MATHEMATICAL SCRIPT CAPITAL X
alert( '😂'.length ); // 2, FACE WITH TEARS OF JOY
alert( '𩷶'.length ); // 2, a rare Chinese hieroglyph
```

`String.fromCodePoint`와 `str.codePointAt`은 surrogate pairs를 넣어도 정상적으로 작동함  
이 함수들이 개발되기 이전의 함수들인 `String.fromCharCode`와 `str.charCodeAt`은 surrogate pairs를 넣으면 제대로 작동하지 않음

```javascript
alert( '𝒳'[0] ); // strange symbols...
alert( '𝒳'[1] ); // ...pieces of the surrogate pair

alert( '𝒳'.charCodeAt(0).toString(16) ); // d835, between 0xd800 and 0xdbff
alert( '𝒳'.charCodeAt(1).toString(16) ); // dcb3, between 0xdc00 and 0xdfff

alert('𝒳'.codePointAt(0).toString(16)); // 1d4b3
alert('𝒳'.codePointAt(1).toString(16)); // dcb3 (*)

alert(String.fromCodePoint(0x1d4b3)); // 𝒳

alert('\u{1d4b3}'); // 𝒳
```
- 이전의 함수들은 surrogate pair를 분리해서 처리해줘야 함  
	surrogate pair의 첫 번째 문자는 `[0xd800, 0xdbff]` 사이의 코드,  
	두 번째는 `[0xdc00, 0xdfff]` 사이의 코드를 가짐
- 최신의 두 함수(`String.fromCodePoint`, `str.codePointAt`)은 surrogate pair도 정상적으로 한 문자처럼 처리함  
	하지만 `(*)`와 같이 두 번째 문자에 접근은 가능하고, surrogate pair는 두 개의 인덱스를 가지는 건 같음!

### Diacritical marks and normalization
발음 구별 기호 중 자주 사용되는 것은 UTF-16에 등록되어 있지만, 나머지도 표현하기 위해서 문자를 조합 가능함:  
```javascript
alert( 'S\u0307' ); // Ṡ
alert( 'S\u0307\u0323' ); // Ṩ
```
- `\u0307`이 위에 점, `\u0323`이 아래 점임

두 코드의 순서를 바꾸면, 보기에는 똑같지만 `==`를 사용해서 비교할 경우 `false`를 반환함  
이것을 해결하기 위해 Unicode normalization을 사용함  
`str.normalize()`로 구현되어 있음:  
```javascript
alert( "S\u0307\u0323".normalize() == "S\u0323\u0307".normalize() ); // true
alert( "S\u0307\u0323".normalize().length ); // 1

alert( "S\u0307\u0323".normalize() == "\u1e68" ); // true
```
- `str.normalize()`를 사용하면 보기에 똑같은 문자들을 모두 같게 만들어 준다는 것을 알 수 있음
- normalization을 거친 문자는 code 상으로 여러 개가 합쳐졌어도 길이가 `1`이라는 것에 유의!

## Summary

| code                                               | description                                                                                      |
| :------------------------------------------------- | :----------------------------------------------------------------------------------------------- |
| `str.length`                                       | `str`의 길이 리턴                                                                                |
| `str[n]`                                           | `str`의 `n` 번째 문자(없으면 `undefined`) 리턴                                                   |
| `str.charAt(pos)`                                  | `str`의 `pos` 번째 문자(없으면 `''`) 리턴                                                        |
| `for...of`                                         | iterating할 때 사용                                                                              |
| `str.toUpperCase()`<br>`str.toLowerCase()`         | `str`을 대/소문자화                                                                              |
| `str.indexOf(substr[, pos])`                       | `str`의 `pos` 번째 문자부터 오른쪽으로 `substr`을 찾고 첫 번째 위치(없으면 `-1`) 리턴            |
| `str.lastIndexOf(substr[, pos])`                   | `str`의 `pos` 번째 문자부터 왼쪽으로 `substr`을 찾고 첫 번째 위치(없으면 `-1`) 리턴              |
| `str.includes(substr[, pos])`                      | `str`에 `substr`이 있는지 판별<br>`true/false` 리턴                                              |
| `str.startsWith(substr)`<br>`str.endsWith(substr)` | `str`이 `substr`로 시작/끝나는지 판별<br>`true/false` 리턴                                       |
| `str.slice(start[, end])`                          | `str`의 `[start, end)`(`end` 없으면 끝까지) 리턴<br>`start`, `end`는 음수가 허용됨               |
| `str.substring(start[, end])`                      | `start`와 `end` 사이의 부분문자열 리턴<br>`start`가 `end`보다 클 수 있지만, 음수가 허용되지 않음 |
| `str.substr(start[, length])`                      | `str`에서 `start`부터 시작하고 길이가 `length`인 부분문자열 리턴<br>`start`는 음수가 허용됨      |
| `str.codePointAt(pos)`                             | `str`의 `pos`번째 문자의 코드 리턴                                                               |
| `String.fromCodePoint(code)`                       | `code`에 해당하는 문자 리턴                                                                      |
| `str.localeCompare(str2)`                          | `str`과 `str2`를 비교, (`<`, `>`, `==`) 각각의 경우에 (음수, 양수, 0) 리턴                       |
| `str.normalize()`                                  | `str`을 Unicode normalization 처리함                                                             |
| `str.trim()`                                       | `str`을 trim한 결과 리턴                                                                         |
| `str.repeat(n)`                                    | `str`을 `n`번 반복한 결과 리턴                                                                   |

- `str.toUpperCase()`, `str.toLowerCase()`는 문자 하나에도 사용 가능
- 문자열을 비교할 때 기본적으로 발음 구별 기호들은 알파벳으로 취급됨

## Tasks
- `Number(str)` 보다 `+str`가 편리함

# Arrays
## Declaration
```javascript
let arr = new Array();
let arr = [];

let fruits = ["Apple", "Orange", "Plum"];
fruits[3] = 'Lemon';
alert( fruits.length ); // 4
alert( fruits ); // Apple,Orange,Plum,Lemon

let arr = [ 'Apple', { name: 'John' }, true, function() { alert('hello'); } ];
```
- 대부분 두 번째 방법 이용
- `append`같은 method 없이 원소를 바로 추가 가능
- `arr.length`로 길이 반환
- 배열 자체를 출력할 수도 있음(csv같이 나옴)
- pyhton의 list처럼 자료형에 제한이 없음
- object literal처럼 trailing comma를 사용할 수 있음

## Methods pop/push, shift/unshift
JS의 array는 아래 4가지 method를 사용해서 queue나 stack으로 활용 가능함:
- `pop`
	- array의 마지막 원소를 빼서 반환
	- `array.pop();`
- `push`
	- array의 마지막에 원소를 추가, 전체 길이 반환
	- `array.push(elem[, elem2...]);`
- `shift`
	- array의 첫 번째 원소를 빼서 반환
	- `array.shift();`
- `unshift`
	- array의 맨 처음에 원소를 추가, 전체 길이 반환
	- `array.unshift(elem[, elem2...]);`

## Internals
array도 근본적으로는 object이기 때문에, **reference로 복사됨**  
JS에서도 배열이 순서가 있는 데이터를 처리하는데 최적화가 되어있음  
하지만 배열을 아래 예시처럼 일반적인 객체로 사용하면 소용이 없어짐:  
```javascript
let fruits = []; // make an array

fruits[99999] = 5; // assign a property with the index far greater than its length

fruits.age = 25; // create a property with an arbitrary name
```
- array를 잘 못 사용하는 경우:
	- non-numeric property를 추가
	- 인덱스를 불연속적이게 정의
	- 배열을 역순으로 채움

## Performance
`push`, `pop`은 `O(1)`이지만, `unshift`, `shift`는 `O(n)`임!

## Loops
기본 `for`문, `for...of`, `for...in` 세 가지 방법으로 반복문을 돌릴 수 있음:
- `for...in`과 `for...of`를 사용하면 index는 알 수 없음
- `for...in`은 numeric이 아닌 property와 method에도 접근함  
	`for...in`은 일반적인 객체와 사용하는 것에 최적화되어있기 때문에 10-100배 정도 느림  
	=> array-like object와 사용하지 않는게 좋음

## A word about "length"
`length`는 배열 안에 값의 개수를 세는 것이 아닌, *가장 큰 인덱스* + 1로 계산됨:  
```javascript
let fruits = [];
fruits[123] = "Apple";

alert( fruits.length ); // 124
```
- 이런 식으로 쓰면 안됨!

`length`는 수정이 가능함:  
```javascript
let arr = [1, 2, 3, 4, 5];

arr.length = 2; // truncate to 2 elements
alert( arr ); // [1, 2]

arr.length = 5; // return length back
alert( arr[3] ); // undefined: the values do not return
```
- 배열에 들어있는 원소의 개수보다 작게 만들면 뒤쪽의 원소들은 없어짐  
	다시 길이를 늘려도 복구되지 않음  
	따라서 `arr.length = 0;`으로 간단하게 배열을 초기화할 수 있음

## new Array()
```javascript
let arr = new Array("Apple", "Pear", "etc");

let arr = new Array(2); // (*)
alert( arr[0] ); // undefined! no elements.
```
- `(*)`와 같이 선언하면 `vector<int> vi(2);`와 같이 원소가 선언되지만 안에 값이 없음

## Multidimensional arrays
```javascript
let matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

alert( matrix[1][1] ); // 5, the central element
```

## toString
array에는 `toString`만 존재(`Symbol.toPrimitive`, `valueOf`는 구현되어있지 않음):  
```javascript
let arr = [1, 2, 3];

alert( arr ); // 1,2,3
alert( String(arr) === '1,2,3' ); // true

alert( [] + 1 ); // "1"
alert( [1] + 1 ); // "11"
alert( [1,2] + 1 ); // "1,21"
```
- 배열은 csv처럼 변환됨
- `[]`와 같은 비어있는 배열은 `""`로 변환됨

## Don't compare arrays with `==`
`==`은 array를 위한 처리가 구현되어있지 않아서 다른 객체들처럼 처리됨:  
```javascript
alert( [] == [] ); // false, (1)
alert( [0] == [0] ); // false, (2)

alert( 0 == [] ); // true, (3)
alert('0' == [] ); // false, (4)
```
- `(1), (2)`는 operand가 양쪽 다 객체이기 때문에 객체끼리 비교
- `(3), (4)`는 `[]`가 내장 `toString`에 의해 `''`로 변환된 다음 알맞은 type으로 다시 변환됨  
	`(3)`에서는 숫자로 변환되는데 `''`가 `0`으로 변환되기 때문에 `true`  
	`(4)`에서는 `'0'`과 `''`를 비교하기 때문에 `false`

> ※ `===`는 type이 다르면 바로 `false`, 같으면 `==`와 동일하게 비교!!

배열을 비교하기 위해선 원소를 하나하나 비교하거나 iteration methods를 사용해야 함

## Summary

| code                                           | description                   |
| :--------------------------------------------- | :---------------------------- |
| `arr.length`                                   | `arr`의 길이 반환             |
| `arr.push(val[, val2...])`<br>`arr.pop()`      | `arr`의 뒤쪽에 원소 삽입/삭제 |
| `arr.unshift(val[, val2...])`<br>`arr.shift()` | `arr`의 앞쪽에 원소 삽입/삭제 |

- `unshift`, `shift`는 `O(n)`임
- 웬만하면 `for...of` 사용해서 탐색
- `arr.length=0;`으로 간단하게 초기화 가능

## Tasks
- `ar=ar+ar2;`를 실행하면 `ar`들이 `String`으로 변환된 후 합쳐져 `ar`의 type이 `String`으로 바뀜!!  
	`ar[0]`을 출력하면 첫 번째 원소가 아니라 첫 번째 문자가 출력됨
- `ar.push(ar2);`를 실행하면 `ar2`가 `ar`의 원소로 들어감  
	=> 둘 다 길이가 2였다면 실행한 후 `ar`의 길이는 4가 아니라 3이 됨
- method도 배열의 원소로 들어갈 수 있음:  
	```javascript
	let arr = ["a", "b"];

	arr.push(function() {
	  alert( this );
	})

	arr[2](); // a,b,function(){...}
	```
	- `arr[2]`에는 함수 자체가 들어가 있기 때문에 호출하면 `arr`이 리턴됨

# Array methods
## Add/remove items
`push/pop/unshift/shift` 이외에도 아래 method들이 있음

### splice
array도 객체이므로 `delete`를 사용해서 property를 지울 수 있긴 하지만 index는 그대로 남음:  
```javascript
let arr = ["I", "go", "home"];

delete arr[2]; // remove "go"

alert( arr[2] ); // undefined

// now arr = ["I",  , "home"];
alert( arr.length ); // 3
```
- `delete obj.key`는 `key`에 해당하는 값을 지우는 것이기 때문에 index가 줄어들지는 않음

`arr.splice`을 사용하면 됨  
```javascript
arr.splice(start[, deleteCount, elem1, ..., elemN]);
```
- `arr`의 `start` 번째부터 `deleteCount`만큼 지운 다음 `elem1, ..., elemN`을 삽입하고,  
	지운 원소들의 배열을 반환함

#### Example
```javascript
let arr = ["I", "study", "JavaScript", "right", "now"];

// remove 3 first elements and replace them with another
arr.splice(0, 3, "Let's", "dance");
alert( arr ) // now ["Let's", "dance", "right", "now"]

let removed = arr.splice(0, 2);
alert( removed ); // "Let's", "dance"
```
- `deleteCount`에 `0`을 넣어서 삽입만 할 수 있음
- `start`는 음수가 허용됨

### slice
```javascript
arr.slice([start[, end]]);
```
- `String`의 method함수 `str.slice(start[, end])`와 같은 기능임
- 인자 없이 `arr.slice()`를 실행하면 `arr`을 복사할 수 있음
- `arr`이 수정되는게 아니라 새로운 배열이 반환됨

### concat
```javascript
arr.concat(arg1, arg2...);
```
- `arr`의 원소 + `arg...`로 이루어진 배열이 반환됨  
	**`arr`에 원소가 추가되지 않음!!**  
	`arr`에 추가하기 위해선 `arr = arr.concat(arg);`와 같이 사용해야 함
- `arg`로 array, value 둘 다 들어갈 수 있음
- 순서대로 추가됨
- `arg`가 array면 원소들이 반환되는 배열에 추가됨  
	array 이외의 type이면 `arg` 자체가 복사(shallow copy)되어 추가됨  
	
	e.g.  
	```javascript
	let arr = [1, 2];

	let arrayLike = {
	  0: "something",
	  length: 1
	};

	alert( arr = arr.concat(arrayLike) ); // 1,2,[object Object]
	alert( arr.length); // 3
	arr[2][0] = 'other';
	alert( arrayLike[0] ); // other
	
	arr.length = 2;
	
	let stra="asdf";
	alert( arr = arr.concat(stra) ); // 1,2,asdf
	arr[2]="iiii";
	alert(arr[2]); // iiii
	alert(stra); // asdf
	```
	- `.`으로 호출하기 위해선 valid variable identifier이어야 함(공백 없음, 숫자로 시작 x, 특수문자 x)
	- numeric property와 length만 있다해도 `new Array()`나 `[]`으로 선언하지 않으면 array-like object가 됨
	
	array-like object는 array처럼 원소가 추가되도록 만들 수 있음  
	=> `Symbol.isConcatSpreadable` property를 `true`로 설정하면 됨!  
	```javascript
	let arrayLike = {
	  0: "something",
	  1: "else",
	  [Symbol.isConcatSpreadable]: true,
	  length: 2
	};

	alert( [].concat(arrayLike) ); // something,else
	```

## Iterate: forEach
```javascript
arr.forEach(function(item, index, array) { ... });
```
- `item`, `index`, `array`는 필요하면 함수의 parameter로 사용 가능
- parameter name은 `item/index/array`로 고정된게 아니고 순서대로 저렇게 들어가는 듯

#### Example
```javascript
// for each element call alert
["Bilbo", "Gandalf", "Nazgul"].forEach(alert);

["Bilbo", "Gandalf", "Nazgul"].forEach((item, index, array) => {
  alert(`${item} is at index ${index} in ${array}`);
});
```
- 함수의 result는 무시됨

## Searching in array
### indexOf/lastIndexOf and includes
`String`의 method들과 같음

- `arr.indexOf(item[, from])`
- `arr.lastIndexOf(item[, from])`
- `arr.includes(item[, from])`

> ※ 모두 비교할 때 `===`을 사용함  
> => `false`를 넣으면 falsy value가 아닌 `false`만 찾음

`includes` method는 `indexOf/lastIndexOf`와 다르게 `NaN`을 처리할 수 있음:  
```javascript
const arr = [NaN];
alert( arr.indexOf(NaN) ); // -1 (should be 0, but === equality doesn't work for NaN)
alert( arr.includes(NaN) );// true (correct)
```
- 포함 여부를 확인할 때는 `includes`를 사용하는게 좋음

### find and findIndex
```javascript
let result = arr.find(function(item, index, array) { ... });
let result = arr.findIndex(function(item, index, array) { ... });
```
- argument로 들어가는 함수가 `true`를 반환할 경우 탐색을 멈추고 그 `item/index`을 반환  
	만족하는 `item/index`가 없을 경우 `undefined/-1` 반환

#### Example
```javascript
let users = [
  {id: 1, name: "John"},
  {id: 2, name: "Pete"},
  {id: 3, name: "Mary"}
];

let user = users.find(item => item.id == 1);

alert(user.name); // John
```

### filter
```javascript
let results = arr.filter(function(item, index, array) { ... });
```
- 함수가 `true`를 반환할 경우 result array에 item을 추가하고 반복을 계속함  
	반복이 끝난 후 result를 반환(만족하는 item이 없었다면 empty array가 반환됨)
- `find`와 유사하지만, `find`는 만족하는 item 하나만 찾아주는 반면 `filter`는 만족하는 모든 item을 찾아줌

#### Example
```javascript
let users = [
  {id: 1, name: "John"},
  {id: 2, name: "Pete"},
  {id: 3, name: "Mary"}
];

// returns array of the first two users
let someUsers = users.filter(item => item.id < 3);

alert(someUsers.length); // 2
```

## Transform an array
### map
```javascript
let result = arr.map(function(item, index, array) { ... });

/*-------------example--------------*/

let lengths = ["Bilbo", "Gandalf", "Nazgul"].map(item => item.length);
alert(lengths); // 5,7,6
```
- argument로 들어가는 함수를 사용해서 `arr`의 item을 매핑하고, 그 결과를 반환

### sort(fn)
```javascript
arr.sort([function() { ... }]);

/*-------------example--------------*/

let arr = [ 1, 2, 15 ];
arr.sort();
alert( arr );  // 1, 15, 2

function compareNumeric(a, b) {
  if (a > b) return 1;
  if (a == b) return 0;
  if (a < b) return -1;
}

arr.sort(compareNumeric);
alert(arr);  // 1, 2, 15
```
- `sort`도 정렬된 array를 반환하긴 하지만, `arr` 자체도 바뀌기 때문에 보통 리턴값을 사용하지 않음
- 인자를 넣지 않으면 `String`을 기준으로 정렬함
	- 비교함수를 넣어서 정렬 기준을 바꿀 수 있음  
		비교함수는 `(a, b)`를 비교할 때 `a`가 더 크면 양수, `b`가 더 크면 음수를 리턴하도록 구현하면 됨  
		=> `arr.sort( (a, b) => a-b )`로 코드를 간결하게 만들 수 있음

> ※ chrome에서는 항상 stable sort로 정렬됨

string을 비교하는 경우 `localeCompare`을 사용하는게 좋음:  
```javascript
let countries = ['Österreich', 'Andorra', 'Vietnam'];

alert( countries.sort( (a, b) => a > b ? 1 : -1) );
// Andorra, Vietnam, Österreich (wrong)

alert( countries.sort( (a, b) => a.localeCompare(b) ) );
// Andorra,Österreich,Vietnam (correct!)
```

### reverse
```javascript
arr.reverse();

/*-------------example--------------*/

let arr = [1, 2, 3, 4, 5];
arr.reverse();

alert( arr ); // 5,4,3,2,1
```
- `sort`와 마찬가지로 reversed array를 반환함

### split and join
```javascript
str.split([delim[, limit]]);

/*-------------example--------------*/

let names = 'Bilbo, Gandalf, Nazgul';

let arr = names.split(', '); // ['Bilbo', 'Gandalf', 'Nazgul']
```
- `delim`이 생략되면 `str` 전체가 하나의 element로 들어감  
	`delim`에 문자열이 들어갈 수 있음  
	`delim`이 `str`의 시작/끝에 나타나면 empty string이 시작/끝에 element로 들어감
- `limit`이 `0`이면 empty array를 반환  
	배열의 길이가 `limit`가 되기 전에 `str`의 끝에 다다르면 바로 종료됨  
	(배열의 길이가 `limit`가 되도록 채우지 않음)

```javascript
arr.join(glue);

/*-------------example--------------*/

let arr = ['Bilbo', 'Gandalf', 'Nazgul'];

let str = arr.join(';'); // glue the array into a string using ;

alert( str ); // Bilbo;Gandalf;Nazgul
```
- `glue`를 넣지 않으면 원소만 이음

### reduce/reduceRight
```javascript
let value = arr.reduce(function(accumulator, item, index, array) { ... }, [initial]);
let value = arr.reduceRight(function(accumulator, item, index, array) { ... }, [initial]);

/*-------------example--------------*/

let arr = [1, 2, 3, 4, 5];
let result = arr.reduce((sum, current) => sum + current, 0);

alert(result); // 15
```
- array를 기반으로 값을 도출할 때 사용
- `accumulator` : 이전 호출의 결과(`initial`이 있는 경우 처음에는 `initial`)  
	반복이 끝난 뒤 `accumulator`가 `reduce`의 결과로 반환됨
- empty array에서 `initial` 없이 `reduce`를 호출할 경우 에러남!

## Array.isArray
array도 `object` type에 속하기 때문에 `typeof`로 타입을 알 수 없음  
=> `Array.isArray(value)`으로 배열인지 판별

## Most methods support "thisArg"
array method 중 `find`, `filter`, `map`과 같이 함수를 argument로 호출하는 method들은 `thisArg`를 추가적으로 붙일 수 있음:  
```javascript
arr.find(func, thisArg);
arr.filter(func, thisArg);
arr.map(func, thisArg);
```
- `thisArg`는 optional last argument임
- `thisArg`의 값은 `func`의 `this`가 됨

#### Example
```javascript
let army = {
  minAge: 18,
  maxAge: 27,
  canJoin(user) {
    return user.age >= this.minAge && user.age < this.maxAge;
  }
};

let users = [
  {age: 16},
  {age: 20},
  {age: 23},
  {age: 30}
];

// find users, for who army.canJoin returns true
let soldiers = users.filter(army.canJoin, army);

alert(soldiers.length); // 2
alert(soldiers[0].age); // 20
alert(soldiers[1].age); // 23
```
- `army.canJoin`은 method로 호출된게 아니라 `filter`의 argument로 들어간 것이기 때문에 함수의 구현부(코드)만 인자로 들어간 상태임  
	=> 실행될 때 `this`가 가리키는 것은 없는 상태이기 때문에 `thisArg`로 `this`를 표시해줘야 함  
	
	`users.filter(user => army.canJoin(user));`로 `thisArg` 없이 사용할 수 있음

## Summary

| code                                                                                                      | description                                                                                                                                                                              |
| :-------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `arr.splice(start[, deleteCount, elem1, ..., elemN])`                                                     | `arr`의 `start` 번째부터 `deleteCount`만큼 지운 다음 `elem1, ..., elemN`을 삽입하고, 지운 원소들의 배열을 리턴<br>`arr`도 변화함                                                         |
| `arr.slice([start[, end]])`                                                                               | `arr`의 `[start, end)`를 리턴                                                                                                                                                            |
| `arr.concat(arg1, arg2...)`                                                                               | `arr`에 `arg...`을 더한 배열을 리턴                                                                                                                                                      |
| `arr.forEach(function(item, index, array) { ... })`                                                       | `arr`의 원소들을 순회하면서 함수에 대입함                                                                                                                                                |
| `arr.indexOf(item[, from])`<br>`arr.lastIndexOf(item[, from])`                                            | `arr`의 `from` 번째부터 `item`을 찾고 첫 번째 일치하는 원소의 인덱스(없으면 `-1`)을 리턴<br>`arr`의 `from`부터 왼쪽으로 `item`을 찾고 첫 번째 일치하는 원소의 인덱스(없으면 `-1`)을 리턴 |
| `arr.includes(item[, from])`                                                                              | `arr`의 `from` 번째부터 `item`이 존재하는지 판별<br>`true/false` 리턴                                                                                                                    |
| `arr.find(function(item, index, array) { ... })`<br>`arr.findIndex(function(item, index, array) { ... })` | 함수가 `true`를 반환하면 탐색을 멈추고 해당 원소 리턴<br>`arr.find`와 같지만, 해당 index 리턴                                                                                            |
| `arr.filter(function(item, index, array) { ... })`                                                        | 함수를 `true`로 만드는 원소들의 배열 리턴                                                                                                                                                |
| `arr.map(function(item, index, array) { ... })`                                                           | 함수로 `arr`을 매핑하고 result array를 리턴                                                                                                                                              |
| `arr.sort([function() { ... })`                                                                           | 함수를 기준으로 정렬한 결과를 리턴<br>`arr`도 변화함<br>함수를 생략하면 string으로 비교해서 정렬함                                                                                       |
| `arr.reverse()`                                                                                           | `arr`을 역순으로 정렬하고 결과 리턴<br>`arr`도 변화함                                                                                                                                    |
| `str.split([delim[, limit]])`                                                                             | `str`을 `delim`으로 구분하고 `limit` 개까지만 array에 저장 후 리턴<br>아무 것도 넣지 않을 경우 나눠지지 않고, `''`을 넣을 경우 한 글자씩 나눠짐                                          |
| `arr.join(glue)`                                                                                          | `arr`의 원소들을 `glue`를 사이에 넣어서 이은 string을 리턴                                                                                                                               |
| `arr.reduce(function(accumulator, item, index, array) { ... }, [initial])`                                | `arr`의 원소들을 함수에 넣으면서 `accumulator`에 결과를 저장하고 반환<br>`initial`은 `accumulator`의 초기값                                                                              |
| `arr.reduceRight(function(accumulator, item, index, array) { ... }, [initial])`                           | `arr.reduce`와 같은 기능이지만 원소들을 역순으로 처리함                                                                                                                                  |
| `Array.isArray(value)`                                                                                    | `value`가 `Array` type인지 판별                                                                                                                                                          |
| `arr.some(function(item, index, array) { ... })`<br>`arr.every(function(item, index, array) { ... })`     | `arr` 안에 함수를 `true`로 만드는 원소가 있는지 판별<br>모든 원소가 함수를 `true`로 만드는지 판별                                                                                        |
| `arr.fill(value[, start[, end]])`                                                                         | `arr`의 `[start, end)`를 `value`로 채우고 리턴<br>`arr`도 변화함                                                                                                                         |
| `arr.copyWithin(target[, start[, end]])`                                                                  | `arr`의 `[start, end)`를 `target` 번째부터 시작해서 붙여넣고 리턴<br>`arr`도 변화함                                                                                                      |
| `arr.flat([depth])`                                                                                       | `arr`의 원소들을 `depth`만큼 차원을 낮춘 결과를 리턴<br>`depth`를 `Infinity`로 설정할 수 있음                                                                                            |
| `arr.flatMap(function(item, index, array) { ... })`                                                       | `arr`의 원소들을 함수로 매핑 후 한 차원 낮춘 결과를 리턴                                                                                                                                 |

- `arr.concat`은 shallow copy이기 때문에 객체의 배열은 deep copy를 따로 해야함
- `includes`는 `NaN`도 찾을 수 있음  
	cf. `NaN === NaN`이 `false`이기 때문에 `indexOf`로는 찾지 못함
- 함수를 argument로 받는 method들은 `thisArg` parameter를 추가할 수 있음  
	(argument인 함수가 method일 때 `this`가 가리키는 객체 지정)
- `arr.every`를 이용해서 배열 2개가 같은지 판별할 수 있음:  
	```javascript
	function arraysEqual(arr1, arr2) {
	  return arr1.length === arr2.length && arr1.every((value, index) => value === arr2[index]);
	}

	alert( arraysEqual([1, 2], [1, 2])); // true
	```
- 호출 후 `arr`도 변화하는 함수들:
	- `arr.splice`
	- `arr.sort`
	- `arr.reverse`
	- `arr.fill`
	- `arr.copyWithin`

## Tasks
- string을 `-`로 split하고 첫 번째 단어 제외 camelize:  
	```javascript
	function camelize(str) {
	  return str.split('-')
	  .map((word, index) => index == 0 ? word : word[0].toUpperCase() + word.slice(1))
	  .join('');
	}
	```
- `forEach`도 반복문과 비슷하게 인덱스를 기반으로 돌아감  
	=> 안에서 `splice` 등으로 배열의 index가 바뀔 경우 영향을 받음  
	e.g. `splice(i, 1)`과 같이 원소 하나를 뺄 경우 반복문처럼 i 바로 다음 원소는 순회하지 않음!
- `f(ar)`에서 arr을 argument로 넘기면, arr의 레퍼런스가 argument로 들어감  
	=> 레퍼런스를 이용해서 method, property를 호출하면 arr을 수정할 수 있지만, 그냥 ar에 배열을 대입하려 하면 함수의 local variable ar에 대입되기 때문에 함수 밖의 arr은 수정되지 않음
- 확장 가능한 계산기 만들기:  
	operator, expression을 분리할 필요 없이 operator를 property로 사용하는 객체를 property로 사용하면 됨  
	```javascript
	this.methods = {
      "-": (a, b) => a - b,
      "+": (a, b) => a + b
    };
	```
- arrow function에서 object literal을 리턴할 경우 `({ ... })`와 같이 괄호로 감싸야 함  
	∵ arrow function에는 `value => expr`, `value => { ... }`와 같이 두 종류가 있음!
- cpp에서의 string은 객체라서 때문에 문자 하나를 수정 가능하지만,  
	JS의 string은 primitive이기 때문에 문자 하나의 수정은 불가능함
- shuffle an array
	- `arr`에서 하나씩 골라 집어넣는 방법(`O(N^2)`)  
		```javascript
		function shuffle(arr) {
		  let iar=[];
		  for(let i=0;i<arr.length;i++)
			iar.push(arr[i]);
		  for(let i=0;i<arr.length;i++) {
			let t=Math.trunc(Math.random()*iar.length);
			arr[i]=iar[t];
			iar.splice(t, 1);
		  }
		}
		```
	- Fisher-Yates shuffle(`O(N)`)  
		```javascript
		function shuffle(array) {
		  for (let i = array.length - 1; i > 0; i--) {
			let j = Math.floor(Math.random() * (i + 1));
			[array[i], array[j]] = [array[j], array[i]];
		  }
		}
		```
		- `[a, b] = [b, a]`는 destructing assignment로, 나중에 다룰 예정
		- 모든 케이스가 동일한 확률을 가짐
- `arr.reduce`를 사용해서 데이터들을 하나의 객체로 변환할 수 있음

# Iterables
객체가 array가 아니더라도 데이터의 모음(list, set 등)으로 표현되면, *iterable*로 만들어 `for...of`로 효과적으로 순회하도록 만들 수 있음

## Symbol.iterator
```javascript
let range = {
  from: 1,
  to: 5
};

range[Symbol.iterator] = function() {  // 1
  return { // 2
    current: this.from,
    last: this.to,

    // 3
    next() { // 4
      if (this.current <= this.last) {
        return { done: false, value: this.current++ };
      } else {
        return { done: true };
      }
    }
  };
};


for (let num of range) {
  alert(num); // 1, then 2, 3, 4, 5
}
```
- `range`를 iterable로 만들기 위해서 `[Symbol.iterator]()` method를 추가해야 함  
	`Symbol.iterator` : 객체를 iterable하게 만들기 위해 존재하는 내장 symbol  
	1. `for...of`가 처음 실행될 때 이 메소드를 호출함  
		이 메소드는 *iterator*(`next`라는 method를 가지는 객체)를 리턴해야 함  
		이 메소드를 찾지 못하면 에러를 반환함
	2. `for...of`는 리턴된 iterator로만 실행됨
	3. `for...of`가 다음 iteration으로 넘어가려 할 때마다 `next()`를 호출함
	4. `next()`는 `{done: Boolean, value: any}` 형식의 객체를 리턴해야 함  
		`done=true`는 iteration이 끝났다는 것을 의미함  
		그게 아니면 `value`에 다음 값을 넣어야함
- `next()`가 리턴하는 객체의 key는 이름을 바꾸면 에러남
- `range` 자체를 iterator로 만들어도 됨:  
	```javascript
	let range = {
	  from: 1,
	  to: 5,

	  [Symbol.iterator]() {
		this.current = this.from;
		return this;
	  },

	  next() {
		if (this.current <= this.to) {
		  return { done: false, value: this.current++ };
		} else {
		  return { done: true };
		}
	  }
	};
	```
	- `Symbol.iterator` method에서 iterator를 다른 객체로 만들어 리턴하지 않고, `range` 자신을 리턴함  
		=> `range`에 `next()`가 선언되어 있어야 함
- 하나의 `range`로 중첩된 `for...of`를 돌릴 수 없음  
	안쪽 반복이 끝난 후 `current`가 공유되기 때문에 바깥쪽 반복도 바로 종료됨

## String is iterable
`Array`와 `String`이 널리 쓰이는 내장 iterable임  
`String`의 경우 `for...of`를 돌리면 문자 하나씩 순회함:  
```javascript
for (let char of "test") {
  // triggers 4 times: once for each character
  alert( char ); // t, then e, then s, then t
}

let str = '𝒳😂';
for (let char of str) {
    alert( char ); // 𝒳, and then 😂
}
```
- surrogate pair로 이루어진 문자열을 넣어도 정상적으로 작동함!

## Calling an iterator explicitly
iterator을 명시적으로 호출할 수도 있음:  
```javascript
let str = "Hello";

// does the same as
// for (let char of str) alert(char);

let iterator = str[Symbol.iterator]();

while (true) {
  let result = iterator.next();
  if (result.done) break;
  alert(result.value); // outputs characters one by one
}
```
- `for...of`를 호출한 것과 같은 결과임
- string은 method를 호출할 때 wrapper object가 생성되기 때문에 따로 method를 정의해도 그 구문 뒤에 바로 사라짐

## Iterables and array-likes
- iterable : `[Symbol.iterator]()` method를 구현한 객체
	- `for...of` 반복문에 넣을 수 있음
- array-like : indexes, `length`를 key로 가지는 객체
	- numeric indexes를 가짐

string은 iterable이면서(`for...of`에서 작동함) array-like(numeric indexes, `length`가 존재)임  
보통은 iterable이거나 array-like이거나 둘 중에 하나임  
둘 다 `push`, `pop`과 같은 method가 없다는 것은 같음

## Array.from
`Array.from` method를 사용해서 iterable이나 array-like로 진짜 `Array` type으로 만들 수 있음:  
```javascript
Array.from(obj[, mapFn, thisArg]);

/*-------------example--------------*/

let arrayLike = {
  0: "Hello",
  1: "World",
  length: 2
};

let arr = Array.from(arrayLike); // (*)
alert(arr.pop()); // World (method works)

let str = '𝒳😂';

// splits str into array of characters
let chars = Array.from(str);

alert(chars[0]); // 𝒳
alert(chars[1]); // 😂
alert(chars.length); // 2
```
- iterable, array-like 모두에 대해서 작동함
- surrogate pairs에 대해서도 정상적으로 작동함

## Summary

| code                                | description                                                                                                  |
| :---------------------------------- | :----------------------------------------------------------------------------------------------------------- |
| `range.[Symbol.iterator]()`         | 객체 `range`를 iterable로 만들기 위해서 정의해야 함<br>iterator 객체를 리턴해야 함                           |
| `iterator.next()`                   | `iterator`가 다음 반복으로 넘어가기 전에 호출하는 함수<br>`done`, `value` property를 가진 객체를 리턴해야 함 |
| `Array.from(obj[, mapFn, thisArg])` | iterable 또는 array-like 객체인 `obj`를 `Array` type으로 바꾸고 `mapFn`을 사용해서 매핑한 배열을 리턴        |

- `[Symbol.iterator]()` method는 `for...of`가 호출될 때 자동으로 실행됨
- `String`, `Array` 같은 built-in iterables도 위 메소드가 구현되어 있음
- string iterable을 이용하면 surrogate pairs를 수월하게 처리 가능

# Map and Set
object를 사용해서 keyed collection을 저장  
array를 사용해서 ordered collection을 저장

## Map
Map은 object와 비슷하게 keyed data를 저장하지만, **모든 타입의 key를 허용**함  
기본적인 method, property들:
- `new Map([entries])` : entries로 초기화된 map 리턴
- `map.set(key, value)` : `key`와 `value`를 저장하고 `map` 리턴
	- 자신을 리턴하기 때문에 chaining이 가능함
- `map.get(key)` : `key`에 해당하는 값(`key`가 존재하지 않으면 `undefined`)을 리턴
- `map.has(key)` : `key`가 존재하면 `true`, 아니면 `false` 리턴
- `map.delete(key)` : `key`와 해당하는 `value`를 삭제
	- 성공적으로 삭제했다면 `true`, 해당하는 `key`가 었다면 `false` 리턴
- `map.clear()` : `map`을 비움
- `map.size` : `map`의 현재 원소 수를 리턴

> ※ `map[key]`로 해도 property를 추가할 수 있지만, 이 문법은 일반 객체에 적용되는 제한(string/symbol key만 가능)을 적용시킴  
> 따라서 `Map`의 method인 `set`, `get`을 사용하는게 좋음

`Map`은 객체도 key로 사용할 수 있음  
cf. 일반 객체는 객체를 key로 사용하면 모든 객체가 `"[object Object]"`로 변환되기 때문에, 다른 객체를 넣어도 `obj["[object Object]"]`라는 property 하나로 처리됨

> ※ `Map`은 SameValue 알고리즘을 이용해서 key들을 비교함  
> 이 알고리즘은 수정될 수 없음

## Iteration over Map
`Map`을 순회하는 3가지 method:
- `map.keys()` : key의 iterable을 리턴
- `map.values()` : value의 iterable을 리턴
- `map.entries()` : `[key, value]`의 iterable을 리턴
	- `Map`을 `for...of`를 사용하여 순회할 때 default로 호출됨

#### Example
```javascript
let recipeMap = new Map([
  ['cucumber', 500],
  ['tomatoes', 350],
  ['onion',    50]
]);

// iterate over keys (vegetables)
for (let vegetable of recipeMap.keys()) {
  alert(vegetable); // cucumber, tomatoes, onion
}

// iterate over values (amounts)
for (let amount of recipeMap.values()) {
  alert(amount); // 500, 350, 50
}

// iterate over [key, value] entries
for (let entry of recipeMap) { // the same as of recipeMap.entries()
  alert(entry); // cucumber,500 (and so on)
}
```
- `Map`은 값이 삽입된 순서대로 순회함  
	cf. 일반 객체는 반복할 때 Integer property일 경우 정렬한 순서대로 순회함

> ※ `Map`도 `Array`와 비슷하게 `forEach` method를 가지고 있음:  
```javascript
// runs the function for each (key, value) pair
recipeMap.forEach( (value, key, map) => {
  alert(`${key}: ${value}`); // cucumber: 500 etc
});
```

## Object.entries: Map from Object
`Map`을 생성할 때 array나 다른 iterable을 argument로 사용해서 초기화할 수 있음:  
```javascript
// array of [key, value] pairs
let map = new Map([
  ['1',  'str1'],
  [1,    'num1'],
  [true, 'bool1']
]);

alert( map.get('1') ); // str1
```
- 다른 iterable도 key/value pair가 property로 들어가 있어야 하고 `next()`를 정의할 수 있어야 하기 때문에, `Map`을 초기화할 때 사용하기 위해선 해봤자 integer property를 가지는 객체일 듯

`Object.entries(obj)` : `obj` -> entries  
```javascript
let obj = {
  name: "John",
  age: 30
};

let map = new Map(Object.entries(obj));

alert( map.get('name') ); // John
```
- `Object.entries(obj)`는 `Map`의 constructor 안에 들어가는 entries를 `obj`로부터 만들어서 반환함  
	이 코드에서는 `[ ["name","John"], ["age", 30] ]`를 리턴함

## Object.fromEntries: Object from Map
`Object.fromEntries(entries)` : `entries` -> obj  
```javascript
let prices = Object.fromEntries([
  ['banana', 1],
  ['orange', 2],
  ['meat', 4]
]);

// now prices = { banana: 1, orange: 2, meat: 4 }

alert(prices.orange); // 2
```
- parameter가 꼭 `Array`일 필요는 없음  
	iterable object면 됨 => map을 바로 넣어도 됨  
	e.g. `obj = Object.fromEntries(map)`

## Set
`Set`를 이용해서 set of values를 저장  
key가 없고 각 값들은 여러 번 추가해도 처음 한 번만 저장됨  
기본적인 method, property들:
- `new Set([iterable])` : `iterable`의 values로 초기화된 set 리턴
- `set.add(value)` : `value`를 추가하고 자신을 리턴
- `set.delete(value)` : `value`가 존재하면 삭제하고 `true`, 아니면 `false` 리턴
- `set.has(value)` : `value`가 존재하면 `true`, 아니면 `false` 리턴
- `set.clear()` : `set`을 비움
- `set.size` : `set`의 현재 원소 개수 리턴

> uniqueness check가 필요한 자료구조에 효율적

## Iteration over Set
`for...of`, `forEach` 둘 다 사용 가능:  
```javascript
let set = new Set(["oranges", "apples", "bananas"]);

for (let value of set) alert(value);

// the same with forEach:
set.forEach((value, valueAgain, set) => {
  alert(value);
});
```
- `forEach`를 사용할 때 callback function에 parameter가 3개임에 주의!  
	`valueAgain`은 `Map`과의 호환성을 위한 것임  
	`Map`을 `Set`으로 바꿀 때 유용함
- `Map`과 비슷하게 아래의 메소드들을 `for...of`에서 사용 가능
	- `set.keys()` : value의 iterable을 리턴
	- `set.values()` : value의 iterable을 리턴
	- `set.entries()` : `[value, value]`의 iterable을 리턴
	
	`Map`과의 호환을 위해서 구현됨

## Summary

| code                                              | description                                                                                                       |
| :------------------------------------------------ | :---------------------------------------------------------------------------------------------------------------- |
| **Map**                                           |                                                                                                                   |
| `new Map([entries])`                              | `entries`로 초기화된 map 리턴                                                                                     |
| `map.set(key, value)`                             | `key`와 `value`를 저장하고 `map` 리턴                                                                             |
| `map.get(key)`                                    | `key`에 해당하는 값(존재하지 않으면 `undefined`) 리턴                                                             |
| `map.has(key)`                                    | `key`가 존재하면 `true`, 아니면 `false` 리턴                                                                      |
| `map.delete(key)`                                 | `key`, `value`가 존재하면 삭제하고 `true`, 아니면 `false` 리턴                                                    |
| `map.clear()`                                     | `map`을 비움                                                                                                      |
| `map.size`                                        | `map`의 현재 원소 수를 리턴                                                                                       |
| `map.keys()`<br>`map.values()`<br>`map.entries()` | key의 iterable을 리턴<br>value의 iterable을 리턴<br>`[key, value]`의 iterable을 리턴                              |
| `Object.entries(obj)`                             | `obj`로부터 entries의 **array**를 만들어서 리턴                                                                   |
| `Object.fromEntries(entries)`                     | `entries`로부터 object를 만들어서 리턴<br>`entries`는 꼭 array를 이용하지 않아도 entries를 포함한 iterable이면 됨 |
| **Set**                                           |                                                                                                                   |
| `new Set([iterable])`                             | `iterable`의 values로 초기화된 set 리턴                                                                           |
| `set.add(value)`                                  | `value`를 추가하고 `set` 리턴                                                                                     |
| `set.delete(value)`                               | `value`가 존재하면 삭제하고 `true`, 아니면 `false` 리턴                                                           |
| `set.has(value)`                                  | `value`가 존재하면 `true`, 아니면 `false` 리턴                                                                    |
| `set.clear()`                                     | `set`을 비움                                                                                                      |
| `set.size`                                        | `set`의 현재 원소 수를 리턴                                                                                       |
| `set.keys()`<br>`set.values()`<br>`set.entries()` | value의 iterable을 리턴<br>value의 iterable을 리턴<br>`[value, value]`의 iterable을 리턴                          |

- `Object` : collection of keyed values  
	`Map` : collection of keyed values  
	`Array` : collection of ordered values  
	`Set` : collection of unique values
- `Set`에는 `Map`과의 호환성을 위해서 구현된 메소드가 많음

## Tasks
- `Array.from(obj)`는 **모든 Array-like, iterable에 대해 사용 가능**
- `Object.keys/values/entries`는 **array**를 리턴하는 반면,  
	`map/set.keys/values/entries`는 **iterable**을 리턴함
- `Array.from(iterable)`을 `[iterable]`로 간단한게 구현 가능

# WeakMap and WeakSet
`Map`은 객체도 key로 사용할 수 있는데, key로 사용되는 객체는 garbage collect되지 않음  
하지만 `WeakMap`은 key로 사용되는 객체가 garbage collect의 대상이 되는 것을 막아주지 않음

## WeakMap
`WeakMap`의 key는 object만 가능함  
object가 `WeakMap`의 key로 사용되는 상황에서 object로의 reference가 하나도 없다면 그 object는 `WeakMap`과 메모리에서 자동으로 지워짐(garbage collect됨):  
```javascript
let john = { name: "John" };

let weakMap = new WeakMap();
weakMap.set(john, "...");

john = null; // overwrite the reference

// john is removed from memory!
```

추가로, `WeakMap`은 iteration과 `keys/values/entries()`와 같은 method를 지원하지 않고, 아래의 method만 지원함:
- `weakMap.get(key)`
- `weakMap.set(key, value)`
- `weakMap.delete(key)`
- `weakMap.has(key)`

=> 모든 key나 value를 알아낼 수 없음  
∵ 객체가 다른 reference를 가지지 않으면 garbage collect의 대상이 되지만, garbage collection이 언제 일어나는지는 JS engine에 의해 결정되기 때문(바로 지워질지, 기다렸다가 한꺼번에 지워질지)  
즉, `WeakMap`의 현재 원소 개수는 알 수 없음

## Use case: additional data
`WeakMap`은 *additional data storage*로써 유용하게 사용됨  
third-party library에 속하거나 다른 이유로 객체 안에 property를 추가하는게 적합하지 않은 상황에서, 객체가 살아있는 동안에만 유효한 데이터를 저장하고 싶을 때 `WeakMap`이 적합한 자료구조임  
`WeakMap`에 데이터를 넣으면 key인 object가 garbage collect될 때 데이터도 자동으로 삭제되기 때문

#### Example
```javascript
// 📁 visitsCount.js using Map
let visitsCountMap = new Map(); // map: user => visits count

// increase the visits count
function countUser(user) {
  let count = visitsCountMap.get(user) || 0;
  visitsCountMap.set(user, count + 1);
}

// 📁 main.js
let john = { name: "John" };

countUser(john); // count his visits

// later john leaves us
john = null;

// 📁 visitsCount.js using WeakMap
let visitsCountMap = new WeakMap(); // weakmap: user => visits count

// increase the visits count
function countUser(user) {
  let count = visitsCountMap.get(user) || 0;
  visitsCountMap.set(user, count + 1);
}
```
- `Map`을 사용할 경우 `john`이 사라져도 수동으로 `visitsCountMap`에서 데이터를 삭제해야 함  
	프로젝트가 커지면 수동으로 삭제하는데 많은 자원이 들어감  
	=> `WeakMap`으로 해결

## Use case: caching
객체를 parameter로 받는 함수가 객체별로 결과를 저장(cache)해놓고 재사용할 수 있음  
이 때 `Map`보다 `WeakMap`을 사용하는게 적합함:  
```javascript
// 📁 cache.js
let cache = new WeakMap();

// calculate and remember the result
function process(obj) {
  if (!cache.has(obj)) {
    let result = /* calculate the result for */ obj;

    cache.set(obj, result);
  }

  return cache.get(obj);
}

// 📁 main.js
let obj = {/* some object */};

let result1 = process(obj);
let result2 = process(obj);

// ...later, when the object is not needed any more:
obj = null;

// Can't get cache.size, as it's a WeakMap,
// but it's 0 or soon be 0
// When obj gets garbage collected, cached data will be removed as well
```
- memoization과 비슷

## WeakSet
- `Set`과 유사하지만, 객체만 저장 가능
- `WeakMap`과 마찬가지로 객체가 reachable할 때만 `WeakSet`안에서 유지됨
- `WeakMap`과 마찬가지로 iteration 불가
	- `weakSet.add(value)`
	- `weakSet.has(value)`
	- `weakSet.delete(value)`
	
	위 3개의 method만 지원함

`WeakMap`과 마찬가지로 additional storage 역할이지만, 임의의 데이터가 아닌, "yes/no"만 표현하는 데이터를 위한 자료구조임  
e.g. 사용자의 방문 횟수가 아닌, 방문 여부

`WeakMap`과 `WeakSet`의 가장 큰 단점은 반복 작업이 불가능한 것과 현재의 모든 원소에 대한 정보(개수 등)을 알 수 없다는 점임  
다른 곳에서 관리되거나 저장된 객체들에 대한 "additional" storage를 제공하는 역할임을 생각해야 함

## Summary

| code                      | description                                                        |
| :------------------------ | :----------------------------------------------------------------- |
| **WeakMap**               |                                                                    |
| `weakMap.get(key)`        | `weakMap`에 `key`에 해당하는 값 리턴                               |
| `weakMap.set(key, value)` | `weakMap`에 `key`, `value`를 추가하고 `weakMap` 리턴               |
| `weakMap.delete(key)`     | `weakMap`에서 `key`가 존재하면 삭제 후 `true`, 아니면 `false` 리턴 |
| `weakMap.has(key)`        | `weakMap`에 `key`가 존재하는지 판별                                |
| **WeakSet**               |                                                                    |
| `weakSet.add(value)`      | `value`를 추가하고 `weakSet` 리턴                                  |
| `weakSet.has(value)`      | `value`가 존재하는지 판별                                          |
| `weakSet.delete(value)`   | `value`가 존재하면 삭제하고 `true`, 아니면 `false` 리턴            |

- `WeakMap`과 `WeakSet`은 secondary storage로서, primary storage에서 객체가 지워지면 이 자료구조들에서도 자동으로 지워짐

## Tasks
- 객체 안에 property를 추가할 수 있다면, symbolic property를 추가해서 다른 사람들은 접근할 수 없는 property를 만들어서 `WeakSet`, `WeakMap`의 역할을 대신할 수 있음
	- 구조적인 관점에서 보면 `WeakSet`이나 `WeakMap`을 사용하는게 나음(secondary storage라는 semantic role이 있기 때문)

# Object.keys, values, entries
`keys/values/entries()`는 모든 자료 구조들에 대해서 사용 가능하도록 약속되어 있음  
=> 자료구조를 만들게 된다면 이 메소드들도 구현해야 함

`Map`, `Set`, `Array`에 대해서는 이미 배움(iterable을 리턴함)  
`Object`에 대해서도 사용 가능하지만, **array를 리턴**함!
- `Object.keys(obj)` : keys의 array를 리턴
- `Object.values(obj)` : values의 array를 리턴
- `Object.entries(obj)` : `[key, value]`의 array를 리턴
	- 호출 방법이 `map.keys()`와 다른 것에 유의  
		∵ `obj`가 `keys`라는 method를 가질 수도 있기 때문

> ※ `Object.keys/values/entries`는 symbolic properties를 무시함!!
> `Object.getOwnPropertySymbols(obj)`가 symbolic keys만 나열된 array를 리턴  
> `Reflect.ownKeys(obj)`가 모든 key가 나열된 array 리턴

## Transforming objects
객체에는 array의 `map`, `filter`와 같은 method들이 없음  
하지만 아래와 같이 property들을 순회할 수 있는 방법이 존재:
1. `Object.entries(obj)`를 사용해서 `obj`로부터 key/value pair의 배열을 얻음
2. 얻은 배열에 배열의 method를 적용
3. 반환된 배열을 `Object.fromEntries(array)`로 다시 객체로 만듦

```javascript
let prices = {
  banana: 1,
  orange: 2,
  meat: 4,
};

let doublePrices = Object.fromEntries(
  // convert to array, map, and then fromEntries gives back the object
  Object.entries(prices).map(([key, value]) => [key, value * 2])
);

alert(doublePrices.meat); // 8
```

## Summary

| code                                                                | description                                                                                         |
| :------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------------- |
| `Object.keys(obj)`<br>`Object.values(obj)`<br>`Object.entries(obj)` | `obj`의 key들의 array 리턴<br>`obj`의 value들의 array 리턴<br>`obj`의 `[key, value]`들의 array 리턴 |
| `Object.fromEntries(array)`                                         | entry들로 이루어진 `array`를 바탕으로 객체를 생성하고 리턴                                          |

- `Object.keys/values/entries`는 `Map`이나 다른 객체들의 method와 같이 iterable이 아닌 `Array`를 리턴함에 주의!

```javascript
let arr = ['a', , 'c'];
let sparseKeys = Object.keys(arr);
let denseKeys = [...arr.keys()];
alert(sparseKeys); // ['0', '2']
alert(denseKeys);  // [0, 1, 2]

alert(arr[1]);
```
- `Object.keys(arr)`은 arr의 arr[n]의 값이 `undefined`면 무시하지만, `arr.keys()`는 무시하지 않고 모두 출력함!

# Destructuring assignment
*Destructuring assignment*는 배열이나 객체를 변수들로 분리하는 문법임  
Destructuring은 수많은 파라미터, 기본값을 가지는 복잡한 함수에도 알맞음

## Array destructuring
```javascript
// 1
let [firstName, surname] = "John Smith".split(' ');
alert(surname);  // Smith

// 2
let [firstName, , title] = ["Julius", "Caesar", "Consul", "of the Roman Republic"];
alert( title ); // Consul

// 3
let [a, b, c] = "abc"; // ["a", "b", "c"]
let [one, two, three] = new Set([1, 2, 3]);

// 4
let user = {
  name: "John",
  age: 30
};

// loop over keys-and-values
for (let [key, value] of Object.entries(user)) {
  alert(`${key}:${value}`); // name:John, then age:30
}

// 5
[guest, admin] = [admin, guest];

```
1. `"John Smith".split(' ')`와 같이 배열을 리턴하는 method도 "destructurize" 가능함
2. `,`를 사용해서 원소를 무시할 수 있음
3. 오른쪽에는 모든 iterable이 올 수 있음
4. destructuring을 이용해서 entries를 순회할 수 있음
	- `Map`, `Set`도 가능함
5. 손쉽게 swap을 구현할 수 있음

> ※ Destructuring은 destructive와 다른 의미임  
> 객체를 분해하지만, 객체 자체에 영향을 주지는 않음

### The rest `...`
```javascript
let arr = ["Julius", "Caesar", "Consul", "of the Roman Republic"];
let [name1, name2] = arr;

alert(name1); // Julius
alert(name2); // Caesar

let [name1, name2, ...rest] = arr;

alert(rest[0]); // Consul
alert(rest[1]); // of the Roman Republic
alert(rest.length); // 2
```
- 길이가 4인 배열을 `[a1, a2]`로 destructurize하면 배열의 3, 4번째 원소들은 무시됨
- `...`을 사용하면 남는 원소들을 한꺼번에 저장할 수 있음
	- `...` 다음에 남는 원소들을 저장할 변수의 이름을 넣음
	- destructuring assignment에서만 사용 가능
	- 항상 left-side의 마지막에 사용해야 함

### Default values
```javascript
// default values
let [name = "Guest", surname = "Anonymous"] = ["Julius"];

alert(name);    // Julius (from array)
alert(surname); // Anonymous (default used)

// runs only prompt for surname
let [name = prompt('name?'), surname = prompt('surname?')] = ["Julius"];

alert(name);    // Julius (from array)
alert(surname); // whatever prompt gets
```
- destructuring을 사용할 때 left-side에 default value 설정 가능
	- `prompt`를 사용해서 부족한 값을 입력받을 수 있음

## Object destructuring
```javascript
let {var1, var2} = {var1:..., var2:...};

/*-------------example--------------*/

let options = {
  title: "Menu",
  width: 100,
  height: 200
};

// 1
let {title, width, height} = options;

// 2
let {width, title, height} = options;

// 3
let {width: w, height: h, title} = options;

// 4
let {width = 100, height = prompt("height?"), title} = options;

// 5
let {width: w = 100, height: h = 200, title} = options;
```
1. destructurize할 객체의 key를 left-side에 넣어야 함  
	=> `title`, `width`, `height`라는 변수 안에 property의 값이 저장됨
2. left-side의 변수의 순서들은 상관없음
3. left-side에서 `sourceProperty: targetVariable`과 같이 이름을 다시 설정 가능함
4. array destructuring 같이 default value를 설정 가능함
5. `:`와 `=`를 동시에 사용 가능함

### The rest pattern `...`
array destructuring과 같이 남는 property들을 rest pattern `...`을 사용해서 다른 객체에 저장 가능함:  
```javascript
let options = {
  title: "Menu",
  height: 200,
  width: 100
};

let {title, ...rest} = options;

alert(rest.height);  // 200
```
- 나머지 property들은 `rest`라는 객체 안에 저장됨
	- old IE의 경우 Babel 등으로 polyfill 해야함

> ※ `let` 없이 destructuring을 사용하기 위해선 아래와 같이 괄호로 묶어야 함!  
> `({title, width, height} = {title: "Menu", width: 200, height: 100});`  
> ∵ JS는 `{...}`을 code block으로 인식하기 때문

## Nested destructuring
```javascript
let options = {
  size: {
    width: 100,
    height: 200
  },
  items: ["Cake", "Donut"],
  extra: true
};

let {
  size: {
    width,
    height
  },
  items: [item1, item2],
  title = "Menu"
} = options;

alert(title);  // Menu
alert(width);  // 100
alert(height); // 200
alert(item1);  // Cake
alert(item2);  // Donut
```
- left-side pattern을 알맞게 설정해서 중첩된 객체도 destructurize 가능

## Smart function parameters
함수의 parameter에도 destructuring을 적용 가능:  
```javascript
function({
  incomingProperty: varName = defaultValue
  ...
}) { ... }

/*-------------example--------------*/

let options = {
  title: "My menu",
  items: ["Item1", "Item2"]
};

function showMenu({
  title = "Untitled",
  width: w = 100,  // width goes to w
  height: h = 200, // height goes to h
  items: [item1, item2] // items first element goes to item1, second to item2
} = {}) {
  alert( `${title} ${w} ${h}` ); // My Menu 100 200
  alert( item1 ); // Item1
  alert( item2 ); // Item2
}

showMenu(options);
```
- 함수의 parameter가 많을 때 객체 하나로 담아서 보낸 후 분해해서 사용가능하게 만듦
- destructuring은 항상 argument가 존재한다고 가정하기 때문에 destructuring에도 기본값(`= {}`)을 넣어줘야 함!

## Summary

| code                 | description                                                                                  |
| :------------------- | :------------------------------------------------------------------------------------------- |
| `let [...] = array;` | array destructuring                                                                          |
| `let {...} = obj;`   | object destructuring                                                                         |
| `...`                | rest pattern<br>객체일 경우 새로운 객체 안에, 배열일 경우 새로운 배열을 저장 공간으로 사용함 |

- 기존에 존재하는 변수에 대해 수행할 경우 전체 문장을 괄호로 묶어야 함

# Date and time
## Creation
```javascript
new Date()
new Date(milliseconds)
new Date(datestring)
new Date(year, month, date, hours, minutes, seconds, ms)

/*-------------example--------------*/

let now = new Date();
let Jan02_1970 = new Date(24 * 3600 * 1000);
let date = new Date("2017-01-26");
let date = new Date(2011, 0, 1);
```
- argument 없이 호출하면 현재 시간을 저장
- milliseconds는 1970.01.01 UTC 기준, 음수도 가능
- datestring은 GMT 기준 0시로 설정된 후 실행되는 환경의 timezone을 따라서 수정됨
- 직접 입력하는 경우 끝쪽부터 연속으로 0인 부분은 생략 가능

## Access date components
- `date.getFullYear()` : 4자리로 `date`의 연도 리턴
- `date.getMonth()` : `[0, 11]`으로 `date`의 달 리턴
- `date.getDate()` : `[1, 31]`으로 `date`의 일 리턴
- `date.getHours()/.getMinutes()/.getSeconds()/.getMilliseconds()` : `date`의 시/분/초/밀리초 리턴
- `date.getDay()` : `[0, 6]`으로 `date`의 요일 리턴(0이 일요일)

> ※ 위의 메소드들은 모두 실행 환경의 timezone이 기준임  
> get뒤에 UTC를 붙여서 UTC 기준으로 시간을 출력할 수도 있음

- `date.getTime()` : 1970.01.01 UTC으로부터 현재까지 지난 시간을 ms단위로 리턴
- `date.getTimezoneOffset()` : UTC와 현재 timezone의 차이를 분 단위로 리턴  
	timezone이 UTC-1이면 `60`이 리턴됨(UTC+0이 기준임)

> ※ UTC+1이면 UTC보다 1시간 빠름

## Setting date components
- `date.setFullYear(year[, month[, date]])`
- `date.setMonth(month[, date])`
- `date.setDate(date)`
- `date.setHours(hour[, min[, sec[, ms]]])`
- `date.setMinutes(min[, sec[, ms]])`
- `date.setSeconds(sec[, ms])`
- `date.setMilliseconds(ms)`
- `date.setTime(milliseconds)` : `date`의 시간을 1970.01.01 UTC 기준으로 `milliseconds` ms 만큼 지난 시간으로 설정

`setTime`을 제외한 메소드들은 UTC-variant를 가짐  
e.g. `setUTCHours`

> ※ argument 이외의 부분은 수정되지 않음!

## Autocorrection
윤년이나 범위를 넘어선 값들은 알아서 계산해서 대입함:  
```javascript
let date = new Date(2013, 0, 32); // 1 Feb 2013

let date = new Date(2016, 1, 28); // 28 Feb 2016
date.setDate(date.getDate() + 2); // 1 Mar 2016

date.setDate(0); // 최소가 1이기 때문에 이전 달의 말일으로 변경됨 = 29 Feb 2016
```
- 달의 경우 `[0, 11]`으로 표현됨에 주의
- 일은 `[1, 31]`로 표현됨

## Date to number, date diff
`Date` 객체가 `Number`로 변환되면 `date.getTime()`과 같은 결과가 나옴  
=> 1970.01.01부터 현재까지 경과한 ms를 반환함  
=> 아래와 같이 실행시간 측정에 사용 가능  
```javascript
let start = new Date();

// do the job
for (let i = 0; i < 100000; i++) {
  let doSomething = i * i * i;
}

let end = new Date();
alert( `The loop took ${end - start} ms` );
```

## Date.now()
시간을 측정하기 위해서 `Date` 객체를 선언할 필요는 없음  
`Date.now()` method를 사용하면 현재 timestamp(1970.01.01부터 경과한 ms)를 알 수 있음  
`new Date().getTime()`과 같은 기능이지만 객체를 생성하지 않기 때문에 더 빠름  
```javascript
let start = Date.now();

// do the job
for (let i = 0; i < 100000; i++) {
  let doSomething = i * i * i;
}

let end = Date.now();
alert( `The loop took ${end - start} ms` );
```

## Benchmarking
```javascript
function diffSubtract(date1, date2) {
  return date2 - date1;
}

function diffGetTime(date1, date2) {
  return date2.getTime() - date1.getTime();
}
```
- 각 함수를 100000번씩 돌리면 `diffGetTime`이 더 빠름
	- type conversion을 하지 않기 때문
- multi-process OS에서는 병렬로 작업을 처리되는 작업이 존재할 수 있으므로 각 함수들을 번갈아가면서 여러 번 측정해야 더 신뢰할 수 있는 결과를 얻을 수 있음  
	```javascript
	// added for "heating up" prior to the main loop
	bench(diffSubtract);
	bench(diffGetTime);

	// now benchmark
	for (let i = 0; i < 10; i++) {
	  time1 += bench(diffSubtract);
	  time2 += bench(diffGetTime);
	}
	```
	- JS의 engine은 자주 실행되는 'hot code'만을 최적화하기 때문에 벤치마킹할 코드를 미리 실행시켜 heat-up하는게 좋음
- 최신의 JS engine들은 최적화를 많이 하기 때문에 연산자나 내장 함수 등을 측정하는 것은 테스트와 실제 환경에서 많이 다를 수 있음

## Date.parse from a string
`Date.parse(str)`을 사용해서 `String`으로부터 timestamp를 파싱할 수 있음  
`str`의 format은 `YYYY-MM-DDTHH:mm:ss.sssZ`이어야 함:
- `YYYY-MM-DD` : year-month-day
- `"T"` : delimiter(character)
- `HH:mm:ss.sss` : hour-minute-second-millisecond
- `Z` : timezone(`+-hh:mm`)
	- `Z`만 사용하면 UTC+0을 나타냄

#### Example
```javascript
let ms = Date.parse('2012-01-26T13:51:50.417-07:00');
alert(ms); // 1327611110417  (timestamp)

let date = new Date( Date.parse('2012-01-26T13:51:50.417-07:00') );
alert(date);
```
- 입력 포맷이 잘못되면 `NaN` 리턴
- 결과값을 바로 `Date`의 생성자에 넣어서 `Date` 객체 생성 가능

## Summary

| code                                                                                                                                                                                                                                                  | description                                                                                                |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------- |
| `date.getFullYear()`                                                                                                                                                                                                                                  | `date`의 연도를 4자리로 리턴                                                                               |
| `date.getMonth()`                                                                                                                                                                                                                                     | `date`의 달을 `[0, 11]`으로 리턴                                                                           |
| `date.getDate()`                                                                                                                                                                                                                                      | `date`의 일을 `[1, 31]`으로 리턴                                                                           |
| `date.getHours()`<br>`date.getMinutes()`<br>`date.getSeconds()`<br>`date.getMilliseconds()`                                                                                                                                                           | `date`의 시/분/초/밀리초 리턴                                                                              |
| `date.getDay()`                                                                                                                                                                                                                                       | `date`의 요일을 `[0, 6]`으로 리턴<br>0이 일요일                                                            |
| `date.getTime()`                                                                                                                                                                                                                                      | 1970.01.01 UTC+0부터 현재까지 경과한 시간을 ms 단위로 리턴                                                 |
| `date.getTimezoneOffset()`                                                                                                                                                                                                                            | UTC+0과 현재 timezone의 차이를 분 단위로 리턴<br>UTC+0 기준임                                              |
| `date.setFullYear(year[, month[, date]])`<br>`date.setMonth(month[, date])`<br>`date.setDate(date)`<br>`date.setHours(hour[, min[, sec[, ms]]])`<br>`date.setMinutes(min[, sec[, ms]])`<br>`date.setSeconds(sec[, ms])`<br>`date.setMilliseconds(ms)` | `date`의 시간 설정                                                                                         |
| `date.setTime(milliseconds)`                                                                                                                                                                                                                          | `date`의 시간을 1970.01.01 UTC 기준으로 `milliseconds` ms 만큼 지난 시간으로 설정                          |
| `Date.now()`                                                                                                                                                                                                                                          | `new Date().getTime()`과 같음                                                                              |
| `Date.parse(str)`                                                                                                                                                                                                                                     | `YYYY-MM-DDTHH:mm:ss.sssZ` 포맷의 `str`을 파싱해서 timestamp 리턴<br>`str`의 포맷이 잘못된 경우 `NaN` 리턴 |

- 이름에 "Time"이 들어가지 않은 method들은 `get/set` 다음에 `UTC`를 붙여서 UTC+0 기준으로 설정 가능(= UTC-variant)
- `set*` 메소드들은 argument 이외의 정보들은 수정되지 않음  
	리턴값은 설정된 시간의 timestamp임
- Month은 `[0, 11]`, Day는 `[0, 6]`, Date는 `[1, 31]`로 표현됨에 주의
- `Date`를 `Number`로 변환하면 `date.getTime()`과 같은 결과임
- JS는 마이크로초로 시간을 측정해주는 메소드를 지원하지 않음  
	browser가 `performance.now()`로 페이지가 로딩될 때부터 경과된 시간을 마이크로초로 측정하는 메소드 지원  
	Node.js는 `microtime` 모듈로 지원함

## Tasks
- `new Date(year, month+1, 0).getDate()`로 `year.month`의 마지막 날을 알 수 있음

# JSON methods, toJSON
복잡한 객체를 전송/로깅 등의 이유로 `String`으로 바꿔야 한다고 가정하자.  
`toString()` method를 구현하는 방법은 property가 바뀔 때마다 갱신해줘야 하고, 중첩된 객체가 있을 경우도 고려해야 하므로 비효율적임  
=> JSON을 이용해서 표현 가능

## JSON.stringify
JSON(JavaScript Object Notation) : 값과 객체를 나타내는 범용적인 format  
원래는 JS를 위해서 만들어졌지만, 다른 언어들도 JSON을 처리하는 라이브러리를 가짐  
따라서 client가 JS를 사용하면 JSON을 사용해서 데이터를 교환하는게 편리함

JS는 아래 메소드들을 지원함:
- `JSON.stringify(obj)` : `obj`를 JSON으로 변환
- `JSON.parse` : JSON을 객체로 변환

#### Example
```javascript
let student = {
  name: 'John',
  age: 30,
  isAdmin: false,
  courses: ['html', 'css', 'js'],
  wife: null
};

let json = JSON.stringify(student);

alert(typeof json); // string
alert(json);
/*
{
  "name": "John",
  "age": 30,
  "isAdmin": false,
  "courses": ["html", "css", "js"],
  "wife": null
}
*/
```
- JSON은 `typeof`로 조사하면 `String`으로 나옴
- JSON을 사용해서 도출된 string `json`은 *JSON-encoded/serialized/stringified/marshalled* object라고 불림
- JSON-encoded object는 object literal과 몇 가지 차이점이 존재함:
	- `String`은 오직 큰 따옴표로 표현됨
	- property name도 큰 따옴표로 표현됨

```javascript
alert( JSON.stringify(1) ) // 1

alert( JSON.stringify('test') ) // "test"

alert( JSON.stringify(true) ); // true

alert( JSON.stringify([1, 2, 3]) ); // [1,2,3]
```
- `JSON.stringify`는 primitive에도 적용 가능함
- JSON은 아래와 같은 data types를 지원함:
	- Objects `{ ... }`
	- Arrays `[ ... ]`
	- Primitives(strings, numbers, booleans, `null`)

> ※ JSON은 data-only, language-independent하기 때문에 JS-specific한 properties는 `JSON.stringify`가 무시함  
> e.g. methods, symbolic properties, `undefined`를 저장하고 있는 properties

중요한 점은 nested object를 지원한다는 것임!  
하지만 circular reference가 있으면 에러가 남  
```javascript
let room = {
  number: 23
};

let meetup = {
  title: "Conference",
  participants: ["john", "ann"]
};

meetup.place = room;       // meetup references room
room.occupiedBy = meetup; // room references meetup

JSON.stringify(meetup); // Error: Converting circular structure to JSON
```

## Excluding and transforming: replacer
`JSON.stringify`의 full syntax는 아래와 같음:  
```javascript
let json = JSON.stringify(value[, replacer[, space]]);
```
- `value` : 인코딩할 값
- `replacer` : 인코딩할 property가 저장된 배열 또는 mapping function `function(key, value)`
	- mapping function은 인코딩하지 않을 property에 대해서 `undefined`를 리턴하면 됨
- `space` : indent 설정

#### Example
```javascript
let room = {
  number: 23
};

let meetup = {
  title: "Conference",
  participants: [{name: "John"}, {name: "Alice"}],
  place: room // meetup references room
};

room.occupiedBy = meetup; // room references meetup

// 1
alert( JSON.stringify(meetup, ['title', 'participants']) );
// {"title":"Conference","participants":[{},{}]}

// 2
alert( JSON.stringify(meetup, ['title', 'participants', 'place', 'name', 'number']) );
/*
{
  "title":"Conference",
  "participants":[{"name":"John"},{"name":"Alice"}],
  "place":{"number":23}
}
*/

// 3
alert( JSON.stringify(meetup, function replacer(key, value) {
  alert(`${key}: ${value}`);
  return (key == 'occupiedBy') ? undefined : value;
}));
/* key:value pairs that come to replacer:
:             [object Object]
title:        Conference
participants: [object Object],[object Object]
0:            [object Object]
name:         John
1:            [object Object]
name:         Alice
place:        [object Object]
number:       23
occupiedBy: [object Object]
*/
```
- nested object에도 `replacer`가 적용됨  
	=> nested object도 포함시켜야 정상적으로 인코딩됨  
	배열로 나열하기 너무 길 경우 함수를 사용하면 됨
- `replacer`로 사용하는 함수에서의 `this`는 현재 property를 가지는 object를 가리킴
- 첫 번째 `alert`가 key가 없는 이유 : wrapper object이기 때문(key가 없고 value는 object 전체를 가리킴)

## Formatting: space
`JSON.stringify(value, replacer, space)`의 `space`는 인덴팅을 조절하기 위해서 사용됨  
숫자 대신 string을 넣으면 그 string을 사용해서 indentation을 수행함  
오직 logging 또는 출력을 예쁘게 하기 위해서 사용(전송할 때는 indent가 없어도 됨)

## Custom "toJSON"
`toString`을 구현해서 객체가 `String`으로 변환되는 것을 조절하는 것처럼,  
`toJSON`을 구현해서 객체가 JSON으로 변환되는 것을 조절함(`toJSON`이 존재하면 `JSON.stringify`가 자동적으로 `toJSON`을 호출함)

#### Example
```javascript
let room = {
  number: 23,
  toJSON() {
    return this.number;
  }
};

let meetup = {
  title: "Conference",
  date: new Date(Date.UTC(2017, 0, 1)),
  room
};

alert( JSON.stringify(meetup) );
/*
  {
    "title":"Conference",
    "date":"2017-01-01T00:00:00.000Z",  // (1)
    "room": 23              // (2)
  }
*/
```
- (1) `Date` 객체는 날짜를 string으로 반환하는 built-in `toJSON` 메소드가 존재함
- (2) object `room`은 `toJSON` 메소드가 구현되어 있으므로 `meetup`이 `stringify`될 때도 property `room`의 value가 `23`으로 설정됨

## JSON.parse
JSON-string을 decode할 때 사용됨:  
```javascript
let value = JSON.parse(str[, reviver]);
```
- `str` : 파싱할 JSON-string
- `reviver` : `(key, value)`를 parameter로 받아서 property의 value를 계산하는 함수 `function(key, value)`
- JSON은 주석을 허용하지 않음!

## Using reviver
```javascript
let str = '{"title":"Conference","date":"2017-11-30T12:00:00.000Z"}';

// 1
let meetup = JSON.parse(str);
alert( meetup.date.getDate() ); // Error!

// 2
let meetup = JSON.parse(str, function(key, value) {
  if (key == 'date') return new Date(value);
  return value;
});
alert( meetup.date.getDate() ); // now works!
```
- `date`의 value는 `stringify`에서 이미 `String`으로 바뀜  
	=> `// 1`과 같이 `date`를 다른 property와 같이 처리하면 `Date` type으로 들어가지 않음  
	
	`// 2`와 같이 reviver를 사용해서 `date` property는 `Date` 객체로 선언해야 함
- replacer와 같이 reviver도 nested object에 대해서 잘 작동함

## Summary

| code                                         | description                                                                             |
| :------------------------------------------- | :-------------------------------------------------------------------------------------- |
| `JSON.stringify(value[, replacer[, space]])` | `value`를 JSON으로 변환함<br>`replacer`로 포함할 property 선택<br>`space`로 인덴팅 설정 |
| `JSON.parse(str[, reviver])`                 | `str`을 object로 변환함<br>`reviver`로 `Date`와 같은 객체를 decode함                    |
| `obj.toJSON()`                               | `obj`의 JSON으로의 변환 구현(= custom `stringify`)                                      |

- JSON은 plain objects, arrays, strings, numbers, booleans, `null`을 지원함  
	JS-specific values(methods, `undefined`, symbolic properties)는 무시됨
- property에 circular reference가 존재하면 에러남!

## Tasks
- transformer functions(`replacer`, `reviver`)가 호출되면 wrapper object(`('', this)`)도 argument로 들어오는 것에 주의!
	- object가 트리 형태라고 생각하면,  
		`replacer`는 root(object의 wrapper object)부터 argument로 들어옴(= preorder)  
		`reviver`는 leaf부터 argument로 들어옴(= postorder)
	
	e.g.  
	```javascript
	let room = {
	  number: 23
	};

	let meetup = {
	  title: "Conference",
	  occupiedBy: [{name: "John"}, {name: "Alice"}],
	  place: room
	};

	room.occupiedBy = meetup;
	meetup.self = meetup;

	afterstringify=JSON.stringify(meetup, function replacer(key, value) {
	  alert(`${key} and ${value}`);
	  return (key!='' && value===meetup)?undefined:value;
	}, 2);
	/*
	 and [object Object]
	title and Conference
	occupiedBy and [object Object],[object Object]
	0 and [object Object]
	name and John
	1 and [object Object]
	name and Alice
	place and [object Object]
	number and 23
	occupiedBy and [object Object]
	self and [object Object]
	*/

	alert(JSON.parse(afterstringify, function reviver(key, value) {
	  alert(`${key} and ${value}`);
	  return value;
	}));
	/*
	title and Conference
	name and John
	0 and [object Object]
	name and Alice
	1 and [object Object]
	occupiedBy and [object Object],[object Object]
	number and 23
	place and [object Object]
	 and [object Object]
	*/
	```
