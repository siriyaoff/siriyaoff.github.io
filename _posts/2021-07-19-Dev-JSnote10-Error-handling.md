---
layout: single
title: "JSnote10: Error handling"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - JavaScript
  - Error handling
  - Error object
  - throw
  - rethrowing
  - global catch
  - instanceof
  - Wrapping exceptions
---

Error handling
# Error handling, "try...catch"
`try...catch`를 사용하면 에러가 발생했을 때 스크립트가 멈추는 대신 다른 행동을 하도록 제어할 수 있음

## The "try...catch" syntax
`try...catch`는 두 block으로 이루어짐: `try`, `catch`:  
```javascript
try {
  // code...
} catch (err) {
  // error handling
}
```
- 아래와 같이 동작함:
	1. `try {...}`가 실행됨
	2. 에러가 발생하지 않으면 `catch (err)`는 무시됨
	3. 에러가 발생하면 `try`의 실행이 중단되고 제어 흐름이 `catch (err)`로 바뀜  
		`err`은 발생한 에러에 대한 정보를 담고 있는 에러 객체임
	
 | ![js-try-catch1](https://github.com/siriyaoff/MDN-note/blob/master/images/js-try-catch1.PNG?raw=true) |
 | :---------------------------------------------------------------------------------------------------: |
 |                                         javascript.info 참고                                          |
	
즉, `try {...}`에서 에러가 발생해도 스크립트가 멈추지 않고 `catch`에서 그것을 해결할 수 있는 기회가 주어지는 것임

**Example**

errorless example:  
```javascript
try {
  alert('Start of try runs');  // (1) <--
  // ...no errors here
  alert('End of try runs');   // (2) <--
} catch (err) {
  alert('Catch is ignored, because there are no errors'); // (3)
}
```
- `(1)`, `(2)`만 출력됨

example with error:  
```javascript
try {
  alert('Start of try runs');  // (1) <--
  lalala; // error, variable is not defined!
  alert('End of try (never reached)');  // (2)
} catch (err) {
  alert(`Error has occurred!`); // (3) <--
}
```
- `(1)`, `(3)`만 출력됨

> **`try...catch`는 runtime error에 대해서만 작동함!**  
> `try...catch`가 작동하기 위해서는 스크립트가 실행 가능해야 함  
> 즉, 문법적으로 올바른 스크립트여야 함
> 
> JS 엔진은 먼저 코드를 읽은 다음 그것을 실행시킴  
> 읽는 단계에서 일어나는 에러는 "parse-time" error라고 불리며, 코드 내부에서 해결할 수 없음
>
> 따라서 `try...catch`는 올바르게 쓰여진 코드에서 발생하는 runtime error(또는 exception)에 대해서만 작동함

> **`try...catch`는 동기적으로 작동함**  
> `setTimeout`과 같은 scheduled code에서 일어나는 exception은 `try...catch`에서 잡을 수 없음:  
> ```javascript
> try {
>   setTimeout(function() {
>     noSuchVariable; // script will die here
>   }, 1000);
> } catch (err) {
>   alert( "won't work" );
> }
> ```
> - 에러가 포함된 함수가 `try...catch`를 마친 다음 실행되기 때문
>
> scheduled function 내부의 exception을 잡기 위해선 `try...catch`를 함수 내부에 넣어야 함:  
> ```javascript
> setTimeout(function() {
>   try {
>     noSuchVariable; // try...catch handles the error!
>   } catch {
>     alert( "error is caught here!" );
>   }
> }, 1000);
> ```

## Error object
에러가 발생하면, JS는 에러에 관한 정보를 담은 객체를 생성함  
그 객체가 `catch`에 인자로 전달됨  
모든 내장 에러들에 대해, 에러 객체들은 두 개의 주요 property가 있음:
- `name` : 에러 이름  
	e.g. `"ReferenceError"` for an undefined variable
- `message` : 에러에 관한 텍스트 정보  
- `stack` : 현재 call stack  
	비표준이지만, 대부분의 환경에서 지원됨

**Example**

```javascript
try {
  lalala; // error, variable is not defined!
} catch (err) {
  alert(err.name); // ReferenceError
  alert(err.message); // lalala is not defined
  alert(err.stack); // ReferenceError: lalala is not defined at (...call stack)

  // Can also show an error as a whole
  // The error is converted to string as "name: message"
  alert(err); // ReferenceError: lalala is not defined
}
```
- `alert(err)`을 하면 `name`, `message`까지만 나옴

## Optional "catch" binding
에러에 관한 정보가 필요하지 않으면 `catch (err)`에서 에러 객체 `err`을 생략할 수 있음:  
```javascript
try {
  // ...
} catch { // <-- without (err)
  // ...
}
```
- 비교적 최근에 추가된 기능이기 때문에 polyfill이 필요할 수도 있음

## Using "try...catch"
`try...catch`의 실사용 예시를 알아보자.  
JS는 `JSON.parse(str)`으로 JSON-encoded value를 읽을 수 있게 해줌:  
```javascript
let json = '{"name":"John", "age": 30}'; // data from the server
let user = JSON.parse(json); // convert the text representation to JS object

alert( user.name ); // John
alert( user.age );  // 30
```

만약 `json`이 망가졌다면, `JSON.parse`는 에러를 발생시키고 스크립트가 죽음  
이렇게 데이터가 잘못되면 방문자들은 알 수 없음  
=> 에러 메시지 없이 스크립트가 멈춤

`try...catch`로 에러를 처리할 수 있음:  
```javascript
let json = "{ bad json }";

try {

  let user = JSON.parse(json); // <-- when an error occurs...
  alert( user.name ); // doesn't work

} catch (err) {
  // ...the execution jumps here
  alert( "Our apologies, the data has errors, we'll try to request it one more time." );
  alert( err.name );
  alert( err.message );
}
```
- 이 예시에서는 `catch`에서 메시지를 띄우는 것만 했지만, 다양한 다른 행동들도 가능함

## Throwing our own errors
만약 `json`이 문법상 올바르지만, 있어야 하는 `name` property가 없다면 어떻게 할 수 있을까?:  
```javascript
let json = '{ "age": 30 }'; // incomplete data

try {
  let user = JSON.parse(json); // <-- no errors
  alert( user.name ); // no name!

} catch (err) {
  alert( "doesn't execute" );
}
```
- `throw` operator를 사용해서 해결할 수 있음

### "Throw" operator
`throw` operator는 에러를 발생시킴:  
```javascript
throw <error object>
```

primitive나 임의의 객체 등 모든 것들을 에러 객체로 사용 가능함  
하지만 보통은 `name`과 `message` property가 있는 객체를 사용하는게 좋음  
∵ 내장된 에러 객체와의 호환성을 위해

JS는 `Error`, `SyntaxError`, `ReferenceError`, `TypeError` 등의 표준 에러들에 대한 내장 생성자를 가지고 있음  
이런 생성자들을 사용해서 에러 객체를 생성해도 됨:  
```javascript
let error = new Error(message);
let error = new SyntaxError(message);
let error = new ReferenceError(message);
// ...
```

내장 에러들은 `name`가 정확히 생성자의 이름이고 `message`는 생성자의 argument임:  
```javascript
let error = new Error("Things happen o_O");

alert(error.name); // Error
alert(error.message); // Things happen o_O
```

`JSON.parse`는 아래와 같이 `SyntaxError`를 생성함:  
```javascript
try {
  JSON.parse("{ bad json o_O }");
  
} catch (err) {
  alert(err.name); // SyntaxError
  alert(err.message); // Unexpected token b in JSON at position 2
}
```

따라서 아래와 같이 `name` property가 없을 때 우리가 에러를 발생시킬 수 있음:  
```javascript
let json = '{ "age": 30 }'; // incomplete data

try {
  let user = JSON.parse(json); // <-- no errors
  if (!user.name) {
    throw new SyntaxError("Incomplete data: no name"); // (*)
  }
  alert( user.name );
  
} catch (err) {
  alert( "JSON Error: " + err.message ); // JSON Error: Incomplete data: no name
}
```
- `user.name`이 존재하지 않기 때문에 `undefined`를 반환하고 `(*)`가 실행됨
- 위 예시에서는 `catch`가 `JSON.parse`와 `(*)`의 경우 모두에 대한 error handling을 담당함

## Rethrowing
아래와 같이 선언되지 않은 변수를 사용하는 등 다른 에러도 발생 가능함:  
```javascript
let json = '{ "age": 30 }'; // incomplete data

try {
  user = JSON.parse(json); // <-- forgot to put "let" before user
  // ...
  
} catch (err) {
  alert("JSON Error: " + err); // JSON Error: ReferenceError: user is not defined
  // (no JSON Error actually)
}
```
- 이때까지는 `try...catch`를 incorrect data로 인한 에러를 잡을 때만 사용했지만, 실제로 `catch`는 `try` block에서 발생하는 *모든* 에러를 잡음  
	=> incorrect data가 아닌 다른 에러가 발생했을 경우에는 제대로 해결할 수 없음

이런 문제를 해결하기 위해서 "rethrowing" technique를 이용함  
Rethrowing : `catch`는 그것이 목표로 하는 에러만 처리하고 나머지는 "rethrow"해야 함

보통 에러 종류를 확인하기 위해 `instanceof`를 사용함:  
```javascript
try {
  user = { /*...*/ };

} catch (err) {
  if (err instanceof ReferenceError) {
    alert('ReferenceError'); // "ReferenceError" for accessing an undefined variable
  }
}
```
- 또는 `err.name`이나 `err.constructor.name`으로 에러 이름을 확인할 수도 있음

아래 예시는 `catch`가 `SyntaxError`만 해결하고 나머지는 rethrow하도록 구현함:  
```javascript
let json = '{ "age": 30 }'; // incomplete data
try {
  let user = JSON.parse(json);
  if (!user.name) {
    throw new SyntaxError("Incomplete data: no name");
  }
  blabla(); // unexpected error
  alert( user.name );

} catch (err) {
  if (err instanceof SyntaxError) {
    alert( "JSON Error: " + err.message );
  } else {
    throw err; // rethrow (*)
  }
}
```
- `(*)`에서의 error throwing은 `try...catch`에서 떨어져 나옴  
	외부에 `try...catch`가 있다면 그것에 의해 잡히거나 스크립트를 중단함

아래 코드는 `try...catch`를 중첩해서 사용하여 rethrow한 에러를 해결하는 예시임:  
```javascript
function readData() {
  let json = '{ "age": 30 }';

  try {
    // ...
    blabla(); // error!
  } catch (err) {
    // ...
    if (!(err instanceof SyntaxError)) {
      throw err; // rethrow (don't know how to deal with it)
    }
  }
}

try {
  readData();
} catch (err) {
  alert( "External catch got: " + err ); // caught it!
}
```
- `readData`는 `SyntaxError`만 해결하고, outer `try...catch`에서 다른 에러들을 처리함

## try...catch...finally
`try...catch` 뒤에 `finally`도 붙일 수 있음  
`finally`는 아래와 같이 동작함:
- 에러가 발생하지 않으면 `try` 다음 실행됨
- 에러가 발생하면 `catch` 다음 실행됨

**Example**

```javascript
try {
  alert( 'try' );
  if (confirm('Make an error?')) BAD_CODE();
} catch (err) {
  alert( 'catch' );
} finally {
  alert( 'finally' );
}
```

`finally` 절은 에러의 발생 유무에 상관없이 실행되어야 하는 내용이 있을 때 자주 사용됨  
예를 들어 `fib(n)`은 `n` 번째 피보나치 수을 반환하는데, 함수 내부에서 에러가 생기면 음수나 정수가 아닌 숫자를 출력하게 구현할 수도 있음:  
```javascript
let num = +prompt("Enter a positive integer number?", 35)

let diff, result;

function fib(n) {
  if (n < 0 || Math.trunc(n) != n) {
    throw new Error("Must not be negative, and also an integer.");
  }
  return n <= 1 ? n : fib(n - 1) + fib(n - 2);
}

let start = Date.now();

try {
  result = fib(num);
} catch (err) {
  result = 0;
} finally {
  diff = Date.now() - start;
}

alert(result || "error occurred");

alert( `execution took ${diff}ms` );
```
- 추가로, `finally`를 사용하면 에러 유무에 상관없이 시간 측정을 제대로 할 수 있음
- 함수는 `return`이나 `throw`를 만나면 종료됨!

> **`finally` and `return`**  
> `try...catch` 안에 `return`이 있어도 `finally`가 작동함:  
> ```javascript
> function func() {
>   try {
>     return 1;
> 
>   } catch (err) {
>     /* ... */
>   } finally {
>     alert( 'finally' );
>   }
> }
> 
> alert( func() );
> ```
> - `"finally"`가 출력된 다음 `1`이 출력됨
> - `return`이 실행되기 전에 `finally`가 먼저 실행됨

> **`try...finally`**  
> `try...finally`는 에러를 처리하지 않고 넘길 때 유용함:  
> ```javascript
> function func() {
>   // start doing something that needs completion (like measurements)
>   try {
>     // ...
>   } finally {
>     // complete that thing even if all dies
>   }
> }
> ```
> - 위 코드에서 `try`에 에러가 발생하면 에러는 fall out되고 `finally`만 실행된 다음 함수가 종료됨  
> 	에러가 발생하지 않으면 `finally`가 끝난 다음 함수의 나머지 코드도 실행됨

## Global catch
> **Environment-specific**  
> 이 내용은 core JS의 내용이 아님

`try...catch` 밖에서 에러가 나는 상황에 대비해서, specification에는 없지만 대부분의 환경에서 지원하는 것이 있음  
예를 들어 Node.js에서는 `process.on("uncaughtException")`으로 지원함  
브라우저 안에서는 `window.onerror` property에 함수를 대입하면 uncaught error이 생겼을 때 그 함수가 실행됨:  
```javascript
window.onerror = function(message, url, line, col, error) {
  // ...
};
```
- `message` : 에러 메세지
- `url` : 에러가 발생한 스크립트의 URL
- `line`, `col` : 에러가 발생한 위치
- `error` : 에러 객체

**Example**

```javascript
<script>
  window.onerror = function(message, url, line, col, error) {
    alert(`${message}\n At ${line}:${col} of ${url}`);
  };

  function readData() {
    badFunc(); // Whoops, something went wrong!
  }

  readData();
</script>
```
- `window.onerror`의 역할은 보통 디버그를 수월하게 하기 위해 로그를 전송하는 것임  
	스크립트의 에러를 해결하기에는 너무 다양한 에러가 나올 수 있음
- [http://errorception.com](http://errorception.com)  
	[http://www.muscula.com](http://www.muscula.com) 등의 사이트로도 에러 확인이 가능함  
	페이지에 `window.onerror`를 정의하는 스크립트를 넣어서 그 결과를 처리 후 보여줌

## Summary

| code                   | description                                         |
| :--------------------- | :-------------------------------------------------- |
| `throw <error object>` | `<error object>`를 에러 객체로 하는 에러를 발생시킴 |

- `try...catch...finally`를 사용해서 runtime error의 error handling 가능  
	`setTimeout`과 같은 scheduled code 내부의 에러는 잡을 수 없음
- error object는 `message`, `name`, `stack` 등의 property를 가짐
- `throw` operator를 사용해서 에러를 발생시킬 수 있음
- "rethrowing"으로 에러를 선택적으로 해결할 수 있음
- 함수는 `return`이나 `throw`를 만나면 종료됨  
	`finally`가 존재하는 `try` 블록 안에서 함수가 종료될 경우, 종료 전에 무조건 `finally`가 실행됨!
- `window.onerror` property에 함수를 저장하면 global error가 발생했을 때 그 함수가 실행됨

## Tasks
```javascript
// (1)
try {
  work work
} catch (err) {
  handle errors
} finally {
  cleanup the working space
}

// (2)
try {
  work work
} catch (err) {
  handle errors
}

cleanup the working space
```
- `(1)`은 `try`나 `catch`에서 `return`이 실행되거나 다른 에러가 발생해도 정상적으로 cleanup이 실행되지만, `(2)`는 위와 같은 경우 cleanup이 실행되지 않고 script가 멈춤
- `catch`로 에러를 잡지 않으면 `finally`는 실행되지만 `try...finally` 바깥의 스크립트는 실행되지 않고 멈춤

# Custom errors, extending Error
무언가를 개발하다보면 에러가 발생한 특정한 상황을 에러에 반영하기 위해 `HttpError`, `DbError`, `NotFoundError`와 같이 다양한 종류의 에러 객체가 필요하다.  
이 에러 객체들은 기본적으로 `message`, `name`, `stack`과 같은 property를 지원하지만, 자신만의 property를 가질 수도 있다.  
e.g. `HttpError`는 `404`, `403`, `500`와 같은 값을 저장하는 `statusCode`를 property로 가질 수 있다.

JS에서는 `throw`로 어떠한 것이든 넘겨줄 수 있기 때문에 우리가 만드는 에러가 `Error`를 상속받을 필요는 없다.  
하지만 `Error`를 상속하면 `obj instanceof Error`와 같이 에러 객체인지 판별하기 쉬워지기 때문에 상속받는게 좋다.

프로그램이 커질수록 에러도 아래 예시와 같이 계층 구조를 가지게 된다.  
e.g. `HttpTimeoutError`는 `HttpError`를 상속받음

## Extending Error
user data를 저장한 JSON을 읽는 `readUser(json)`이라는 함수를 생각해보자.  
이 함수의 내부에서 우리는 `JSON.parse`를 사용하고, 이 메소드는 `json`이 형식에 맞지 않을 경우 `SyntaxError`가 발생한다.

우리는 user data에 `name`이나 `age`가 없는 경우도 `json`이 올바르지 않은 경우도 에러가 발생한 것처럼 취급한다.  
즉 `readUser(json)`은 JSON을 읽을 뿐만 아니라, `json`이 올바른지 확인하도록 구현해야 한다.  
필요한 필드가 없거나 형식이 잘못되었을 경우 에러라 간주하고 `ValidationError`라는 에러 객체를 반환하게 하면 된다.

`ValidationError`는 문제가 생긴 필드에 관한 정보를 가지고 있어야 한다.  
우리는 `ValidationError` 클래스가 `Error` 클래스를 상속하도록 만들 것이다.  
`Error`는 내장된 클래스지만, 이해를 돕기 위해 아래와 같이 비슷하게 직접 구현했다:  
```javascript
class Error {
  constructor(message) {
    this.message = message;
    this.name = "Error"; // (different names for different built-in error classes)
    this.stack = <call stack>; // non-standard, but most environments support it
  }
}

class ValidationError extends Error {
  constructor(message) {
    super(message); // (1)
    this.name = "ValidationError"; // (2)
  }
}

// Usage
function readUser(json) {
  let user = JSON.parse(json);
  if (!user.age) {
    throw new ValidationError("No field: age");
  }
  if (!user.name) {
    throw new ValidationError("No field: name");
  }
  return user;
}

try {
  let user = readUser('{ "age": 25 }');
} catch (err) {
  if (err instanceof ValidationError) {
    alert("Invalid data: " + err.message); // Invalid data: No field: name
  } else if (err instanceof SyntaxError) { // (*)
    alert("JSON Syntax Error: " + err.message);
  } else {
    throw err;
  }
}
```
- `(*)`는 `err.name == "SyntaxError"`와 같음  
	하지만 `instanceof`가 미래 지향적임  
	∵ 에러가 더욱 세분화되어도 `SyntaxError`를 상속받으면 코드를 수정하지 않아도 판별할 수 있기 때문
- `SyntaxError`, `ValidationError`일 경우 직접 처리하고, 나머지 에러들은 rethrow함

## Further inheritance
`ValidationError`는 너무 포괄적임: property가 없을 수도 있고 포맷이 잘못될 수도 있음  
따라서 property가 없는 경우를 담당하는 `PropertyRequiredError`를 만들어야 함:  
```javascript
class PropertyRequiredError extends ValidationError {
  constructor(property) {
    super("No property: " + property);
    this.name = "PropertyRequiredError";
    this.property = property;
  }
}

// Usage
function readUser(json) {
  let user = JSON.parse(json);
  if (!user.age) {
    throw new PropertyRequiredError("age");
  }
  if (!user.name) {
    throw new PropertyRequiredError("name");
  }
  return user;
}

try {
  let user = readUser('{ "age": 25 }');
} catch (err) {
  if (err instanceof ValidationError) {
    alert("Invalid data: " + err.message); // Invalid data: No property: name
    alert(err.name); // PropertyRequiredError
    alert(err.property); // name
  } else if (err instanceof SyntaxError) {
    alert("JSON Syntax Error: " + err.message);
  } else {
    throw err; // unknown error, rethrow it
  }
}
```
- `super(...)`는 message를 인자로 받음
- `PropertyRequiredError`는 `ValidationError`를 상속받았기 때문에 `err instanceof ValidationError`로 판별 가능함
- `this.name=<class name>`으로 에러 객체를 생성할 때마다 초기화해줘야 하는데, 가장 상위 객체 "basic error"를 만들어서 그 생성자 안에 `this.name=this.constructor.name`을 실행해서 이를 생략할 수 있음:  
	```javascript
	class MyError extends Error {
	  constructor(message) {
		super(message);
		this.name = this.constructor.name;
	  }
	}

	class ValidationError extends MyError { }

	class PropertyRequiredError extends ValidationError {
	  constructor(property) {
		super("No property: " + property);
		this.property = property;
	  }
	}

	alert( new PropertyRequiredError("field").name ); // PropertyRequiredError
	```
	- `super()`를 호출해도 context는 현재 객체이기 때문에 제대로 작동함

## Wrapping exceptions
위의 `readUser`에서는 `SyntaxError`, `ValidationError` 두 가지 에러를 처리할 수 있음  
다른 많은 에러들에 대해서도 처리하게 구현할 수 있지만, 에러의 종류가 너무 많기 때문에 이렇게 하나씩 처리하는 것은 비효율적임  
사실 우리는 "data reading error"가 발생했는지만 알면 됨  
(발생한 이유는 에러 메세지에서 알 수 있기 때문)  
또는 필요할 때만 에러의 세부 정보를 알 수 있는 방법만 있으면 됨

이러한 테크닉을 "wrapping exceptions"라고 부름:
1. `ReadError` 클래스를 생성해서 모든 "data reading" error를 대표하게 함
2. `readUser` 함수는 내부에서 일어나는 data reading error를 잡고, `ValidationError`나 `SyntaxError` 대신 `ReadError`를 생성함
3. `ReadError`는 `cause` property에 원래 에러로의 reference를 저장함

이렇게 되면 `readUser`를 호출한 이후에는 `ReadError`가 발생했는지만 확인하면 됨  
모든 에러들을 일일히 확인할 필요가 없음!

**Example**

```javascript
class ReadError extends Error {
  constructor(message, cause) {
    super(message);
    this.cause = cause;
    this.name = 'ReadError';
  }
}

class ValidationError extends Error { /*...*/ }
class PropertyRequiredError extends ValidationError { /* ... */ }

function validateUser(user) {
  if (!user.age) {
    throw new PropertyRequiredError("age");
  }
  if (!user.name) {
    throw new PropertyRequiredError("name");
  }
}

function readUser(json) {
  let user;
  try {
    user = JSON.parse(json);
  } catch (err) {
    if (err instanceof SyntaxError) {
      throw new ReadError("Syntax Error", err);
    } else {
      throw err;
    }
  }

  try {
    validateUser(user);
  } catch (err) {
    if (err instanceof ValidationError) {
      throw new ReadError("Validation Error", err);
    } else {
      throw err;
    }
  }

}

try {
  readUser('{bad json}');
} catch (e) {
  if (e instanceof ReadError) {
    alert(e);
    // Original error: SyntaxError: Unexpected token b in JSON at position 1
    alert("Original error: " + e.cause);
  } else {
    throw e;
  }
}
```
- `ReadError`를 정의하고, `readUser` 함수는 코드를 실행하면서 우리가 확인해야 할 필요가 있는 에러들(`SyntaxError`, `ValidationError`)이 발생하면 `ReadError`로 rethrow함
- 따라서 `readUser`를 호출했을 때 `ReadError`가 발생하면 알맞게 처리하면 되고, 나머지 에러는 처리할 필요가 없다는 뜻이기 때문에 rethrow함

## Summary
- `Error` 클래스도 상속할 수 있음  
	`name`을 수정하는 것과 `super`를 호출하는 것만 주의하면 됨  
	`name` 수정은 `this.name=this.constructor.name`으로 자동화할 수 있음
- `instanceof` operator나 `name` property를 이용해서 error의 종류를 알 수 있음
- wrapping exception은 널리 사용되는 기술임:  
	함수가 low-level exception을 처리하고 higher-level error를 여러 low-level 에러를 대신해 throw함  
	low-level exception에 관한 정보를 `cause` property에 저장할 수도 있음

## Tasks
### `FormatError` 구현
```javascript
class FormatError extends SyntaxError{
  constructor(message){
    super(message);
    this.name=this.constructor.name;
  }
}

let err = new FormatError("formatting error");

alert( err.message ); // formatting error
alert( err.name ); // FormatError
alert( err.stack ); // stack

alert( err instanceof FormatError ); // true
alert( err instanceof SyntaxError ); // true (because inherits from SyntaxError)
```