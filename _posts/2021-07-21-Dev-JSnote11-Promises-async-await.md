---
layout: single
title: "JSnote11: Promises, async/await"
categories:
  - Dev
tags:
  - JavaScript
  - Promise
  - Promise chaining
  - Thenable
  - fetch
  - Error handling
  - Rethrowing
  - unhandledrejection
  - API
  - Promisification
  - Microtask
  - Async/await
---

Promises, async/await
# Introduction: callbacks
> #### 이 article에서는 browser method를 사용함
> callback, promise와 다른 추상적인 개념을 소개하기 위해 스크립트 로딩, 페이지 조작과 같은 browser method를 사용함  

많은 함수들은 호스트의 JS에서 제공되는데, 이 함수들을 이용해서 비동기적(asynchronous) 동작을 schedule할 수 있음  
즉 동작이 원할 때 시작되도록 만들 수 있음  
`setTimeout`, 스크립트 로딩 등이 그 예시임  
아래 `loadScript(src)`는 `src`로 주어지는 script를 로드함:  
```javascript
function loadScript(src) {
  let script = document.createElement('script');
  script.src = src;
  document.head.append(script);
}

loadScript('/my/script.js');
```
- `<script>` tag를 페이지에 추가하고 로딩이 완료되면 자동으로 실행함
- `loadScript('/my/script.js');` 아래의 코드들은 스크립트가 로딩될 떄까지 기다리지 않고 실행됨  
	e.g. 아래와 같이 script를 로드하고 그 안에 선언된 함수를 실행하면 작동하지 않음:  
	```javascript
	loadScript('/my/script.js'); // the script has "function newFunction() {…}"

	newFunction(); // no such function!
	```

`loadScript`의 두 번째 인자로 `callback`을 넣어 스크립트가 로드되었을 때 실행되게 만들어보자:  
```javascript
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;

  script.onload = () => callback(script);

  document.head.append(script);
}

loadScript('/my/script.js', function() {
  newFunction();
  ...
});
```
- 이제 `callback` argument를 이용하면 스크립트가 로드된 후 그 안의 함수를 호출할 수 있음  
	`loadScript`가 끝난 다음 `callback`이 실행되는게 아니라 `loadScript` 안에서 sciprt loading을 마치고 `callback`까지 실행하는 것임
- 이는 "callback-based" 이라고 불리는 asynchronous programming의 스타일 중의 하나임  

## Callback in callback
어떻게 두 스크립트를 순차적으로 로드할 수 있을까?  
또 다른 `loadScript`를 `callback`으로 넣으면 됨:  
```javascript
loadScript('/my/script.js', function(script) {
  alert(`Cool, the ${script.src} is loaded, let's load one more`);

  loadScript('/my/script2.js', function(script) {
    alert(`Cool, the second script is loaded`);
  });
});
```
- 바깥의 `loadScript`부터 순차적으로 로드됨
- 순차적으로 로드해야 할 script가 많아질 수록 중첩을 반복해야 함

## Handling errors
script를 로드할 때 에러가 나는 상황을 고려해보자:  
```javascript
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;

  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`Script load error for ${src}`));

  document.head.append(script);
}

loadScript('/my/script.js', function(error, script) {
  if (error) {
    // handle error
  } else {
    // script loaded successfully
  }
});
```
- `callback` 함수에 `error` argument를 추가해서 에러가 났을 때는 처리하도록 구현함  
	"error-first callback" style이라고 부름  
	
	이 방법도 흔히 쓰이는데, 관례가 존재함:
	1. `callback()`의 첫 번째 인자는 에러
	2. 두 번째 인자부터는 성공적으로 로드되었을 때 사용할 것들임
	
	즉, 에러가 있을 경우에는 `callback(err)`이 호출되고, 없으면 `callback(null, ...)`으로 호출하면 됨

## Pyramid of Doom
위에서 언급했듯이, callback을 사용하는 방법은 nested call이 많아질 수록 관리하기가 어려움  
nested call이 너무 많아진 상태를 "callback hell" 또는 "pyramid of doom"이라고 부르기도 함

이를 해결하기 위해 callback을 모두 각각의 함수로 정의하면 됨:  
```javascript
loadScript('1.js', step1);

function step1(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...
    loadScript('2.js', step2);
  }
}

function step2(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...
    loadScript('3.js', step3);
  }
}

function step3(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...continue after all scripts are loaded (*)
  }
}
```
- 콜 스택과 내부 동작은 같지만 코드의 가독성이 증가함  
	하지만 모든 `step*` 함수들은 pyramid of doom을 피하기 위한 일회용 함수이기 때문에 namespace가 복잡해짐(namespace cluttering)  
	그리고 코드의 흐름이 끊김
	
	이런 단점을 보완한게 promise임

## Summary
- "callback-based" style : 스크립트 로딩을 마치고 실행해야 하는 내용을 `callback` 함수 형태로 받아서 스크립트 로드 후 실행하도록 구현하는 방식
- error-first callback : `callback` 함수의 첫 번째 인자를 error object로 둬서 에러 처리까지 추가한 것
- 중첩이 너무 많아질 경우 pyramid of doom을 피하기 위해 `callback`을 모두 다른 함수로 선언해서 중첩을 없앨 수 있음(내부 구조는 같음)

# Promise
JS에도 유튜브 채널을 알림 설정 해놓는 것과 같은 기능이 있음
- "producing code" : 데이터를 로드하는 등의 시간이 소요되는 일을 하는 코드
- "consuming code" : "producing code"의 결과를 이용해서 동작하는 코드
- *promise* : "producing code"와 "consuming code"를 이어주는 JS의 object  
	"subscription list"라고 볼 수 있음  
	"producing code"가 완료되면 결과를 사용할 수 있도록 알려줌

promise의 생성자는 아래와 같음:  
```javascript
let promise = new Promise(function(resolve, reject) {
  // executor (the producing code, "singer")
});
```
- `new Promise`로 전달되는 함수는 *executor*라고 함  
	promise가 생성되었을 때 executor는 자동으로 실행됨  
	executor는 결과를 만드는 "producing code"를 포함함
- executor의 인자 `resolve`와 `reject`는 JS에 의해 제공되는 callback임
	- `resolve(value)` : executor의 작업이 성공적으로 수행되면 그 결과를 이용해서 작동하는 callback
	- `reject(error)` : executor에서 에러가 발생하면 그것을 처리하는 callback
	
	|![js-promise-properties](https://github.com/siriyaoff/MDN-note/blob/master/images/js-promise-properties.PNG?raw=true)|
	|:---:|
	|javascript.info 참고|
	
	우리가 적어야 하는 코드는 executor 안의 코드 밖에 없음
- `promise` 객체는 아래와 같은 내부 property를 가짐:
	- `state` : `"pending"`으로 초기화됨  
		`resolve`가 호출되었을 때는 `"fulfilled"`, `reject`가 호출되었을 때는 `"rejected"`로 바뀜
	- `result` : `undefined`로 초기화됨  
		`resolve(value)`가 호출되었을 때는 `value`, `reject(error)`가 호출되었을 때는 `error`로 바뀜

#### Example
```javascript
// (1)
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve("done"), 1000);
});

// (2)
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => reject(new Error("Whoops!")), 1000);
});
```
- `// (1)`에서는 executor가 자동으로 실행되는데, `resolve("done")`이 호출되기 때문에 `promise` 객체의 property는 아래와 같이 바뀜  
	`state: "fulfilled", result: "done"`
- `// (2)`에서는 `reject(new Error("Whoops!")`를 호출했기 때문에 `promise` 객체의 property는 아래와 같이 바뀜  
	`state: "rejected", result: error`

executor는 작업을 수행하고 `resolve`나 `reject`를 호출해서 현재 promise 객체의 state를 바꿈  
`state`가 `"fulfilled"` 또는 `"rejected"`인 객체는 "settled"라 하고, 아직 callback이 실행되지 않은 객체는 "pending" promise라고 함

> #### 단 하나의 callback만 실행할 수 있음
> executor에서는 `resolve`나 `reject` 중 하나를 한 번만 호출할 수 있음  
> 그 이후의 `resolve`나 `reject`는 무시됨:  
> ```javascript
> let promise = new Promise(function(resolve, reject) {
>   resolve("done");
> 
>   reject(new Error("…")); // ignored
>   setTimeout(() => resolve("…")); // ignored
> });
> ```
>
> executor에 의해 수행되는 작업의 결과는 하나의 결과만을 가져야 하기 때문  
> 또한 `resolve`나 `reject`는 단 하나의 argument(또는 생략)만을 가짐  
> (나머지는 무시됨)

> #### Reject with `Error` objects
> `reject`도 `resolve`처럼 argument로 어떤 것이든 가능하지만, 웬만하면 `Error` 객체를 넣는게 좋음

> #### The `state` and `result` are internal
> `state`와 `result`는 Promise 객체의 내부 property이기 때문에 우리가 직접 접근할 수 없음  
> `.then`, `.catch`, `.finally` method를 사용해서 접근해야 함

## Consumers: then, catch, finally
consuming function은 `.then`, `.catch`, `.finally` method를 이용해서 등록 가능함

### then
```javascript
promise.then(
  function(result) { /* handle a successful result */ },
  function(error) { /* handle an error */ }
);

/*-------------example--------------*/

// (1)
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve("done!"), 1000);
});

// resolve runs the first function in .then
promise.then(
  result => alert(result), // shows "done!" after 1 second
  error => alert(error) // doesn't run
);

// (2)
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => reject(new Error("Whoops!")), 1000);
});

// reject runs the second function in .then
promise.then(
  result => alert(result), // doesn't run
  error => alert(error) // shows "Error: Whoops!" after 1 second
);
```
- `.then`의 첫 번째 argument는 promise가 resolved일 때 실행되는 함수로, `result`를 인자로 받음
- 두 번째 argument는 rejected일 때 실행되는 함수로, `error`를 인자로 받음
- `// (1)`과 `// (2)`에서 볼 수 있듯이, 두 함수 중 하나만 실행됨  
	∵ executor의 결과에 따라 `value`, `error` 둘 중에 하나만 존재할 수 있기 때문

만약 에러가 났을 경우를 고려하지 않아도 되는 상황이면 아래와 같이 구현할 수도 있음:  
```javascript
let promise = new Promise(resolve => {
  setTimeout(() => resolve("done!"), 1000);
});

promise.then(alert); // shows "done!" after 1 second
```

### catch
에러를 처리해야 하는 상황이면, `.then(null, errHandler)` 또는 `.catch(errHandler)`로 구현할 수 있음:  
```javascript
let promise = new Promise((resolve, reject) => {
  setTimeout(() => reject(new Error("Whoops!")), 1000);
});

promise.catch(alert); // shows "Error: Whoops!" after 1 second
```
- `.catch(f)`와 `.then(null, f)`의 shorthand임(기능은 같음)

### finally
`try...catch`의 `finally`와 같음  
`finally`는 cleanup을 수행할 때 좋음  
e.g. loading indicator 중지할 때:  
```javascript
new Promise((resolve, reject) => {
  ...
})
  .finally(() => stop loading indicator)
  .then(result => show result, err => show error)
```
- `.finally`는 executor의 결과에 상관없이 loading indicator를 먼저 중지시킴
- loading indicator가 먼저 중지된 후 `reuslt/error`가 출력됨

위 예시에서 알 수 있듯이, `finally(f)`는 `then(f, f)`와 완전히 동일하지는 않음:
1. `finally`에 들어가는 handler는 argument가 없음  
	∵ executor의 결과에 상관없이 동작하는 코드이기 때문
2. `finally`의 handler는 result와 error를 다음 handler로 넘김  
	```javascript
	new Promise((resolve, reject) => {
	  throw new Error("error");
	})
	  .finally(() => alert("Promise ready"))
	  .catch(err => alert(err));  // <-- .catch handles the error object
	```

> #### We can attach handlers to settled promises
> promise가 pending 상태이면 `.then/catch/finally` handler가 기다리지만, settled일 경우 바로 실행됨:  
> ```javascript
> let promise = new Promise(resolve => resolve("done!"));
> 
> promise.then(alert); // done!
> ```
>
> handler를 언제든지 추가할 수 있음  
> 결과가 나와있는 promise에도 handler를 추가할 수 있음

## Example: loadScript
promise를 사용해서 loadScript를 구현해보자:  
```javascript
function loadScript(src) {
  return new Promise(function(resolve, reject) {
    let script = document.createElement('script');
    script.src = src;

    script.onload = () => resolve(script);
    script.onerror = () => reject(new Error(`Script load error for ${src}`));

    document.head.append(script);
  });
}

let promise = loadScript("https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.11/lodash.js");

promise.then(
  script => alert(`${script.src} is loaded!`),
  error => alert(`Error: ${error.message}`)
);

promise.then(script => alert('Another handler...'));
```
- callback을 사용한 `loadScript`는 `callback`으로 callback 함수를 받아서 내부에서 실행시키는 구조였음  
	따라서 callback을 한 가지를 한 번만 사용 가능했음  
	그에 비해 promise를 사용하면 `loadScript`가 promise를 리턴하고, 리턴된 promise를 이용해서 callback을 필요한 만큼 실행시킬 수 있음

## Summary

|code|description|
|:---|:---|
|`promise.then(f1, f2)`|`promise`가 fulfilled면 `f1`, rejected면 `f2` 실행|
|`promise.catch(f)`|`promise`가 rejected면 `f` 실행<br>`promise.then(null, f)`와 같음|
|`promise.finally(f)`|`promise`가 settled면 `f` 실행|

```javascript
let promise = new Promise(function(resolve, reject) {
  // executor
});
```
- promise의 property
	- `state` : `"pending"`으로 초기화  
		`"fulfilled"` / `"rejected"`로 바뀜
	- `result` : `undefined`로 초기화  
		`value` / `error`로 바뀜
- executor에서 `resolve(value)` 또는 `reject(error)`를 호출해야 함
- `.then/catch/finally`와 같은 handler들은 필요한 만큼 실행 가능
- `.finally(f)`의 `f`는 argument가 존재하지 않음
- `.finally`는 arguments(`value/error`)를 전달함(chaining 가능)

## Tasks
```javascript
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

delay(3000).then(() => alert('runs after 3 seconds'));
```
- executor는 원래 `(resolve, reject)`를 argument로 받지만, `resolve`만 받아도 됨

# Promises chaining
promise chaining을 이용해서 비동기적인 작업을 효율적으로 구현할 수 있음:  
```javascript
new Promise(function(resolve, reject) {

  setTimeout(() => resolve(1), 1000); // (*)

}).then(function(result) { // (**)

  alert(result); // 1
  return result * 2;

}).then(function(result) { // (***)

  alert(result); // 2
  return result * 2;

}).then(function(result) {

  alert(result); // 4
  return result * 2;

});
```
- `promise.then`은 promise를 리턴하기 때문에 chaining이 가능함  
	handler가 값을 리턴하면 그것은 promise의 result가 됨  

> #### chaining 중간에 error를 반환해서 catch하는 것도 가능한가?
> handler 안에서 에러를 reject하면 됨

하나의 promise에 여러 개의 `then`을 추가하는 것은 chaining이 아님:  
```javascript
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve(1), 1000);
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});
```

|chaining|multiple `.then`|
|:---:|
|![js-promise-chaining1](https://github.com/siriyaoff/MDN-note/blob/master/images/js-promise-chaining1.PNG?raw=true)|![js-promise-chaining2](https://github.com/siriyaoff/MDN-note/blob/master/images/js-promise-chaining2.PNG?raw=true)|
|javascript.info 참고|javascript.info 참고|

## Returning promises
`.then(handler)` 안의 `handler`도 promise를 생성하고 반환할 수 있음  
이때 이어지는 handler는 생성된 promise가 settled될 때까지 기다림:  
```javascript
new Promise(function(resolve, reject) {
  setTimeout(() => resolve(1), 1000);

}).then(function(result) {
  alert(result); // 1
  return new Promise((resolve, reject) => { // (*)
    setTimeout(() => resolve(result * 2), 1000);
  });

}).then(function(result) { // (**)
  alert(result); // 2
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(result * 2), 1000);
  });

}).then(function(result) {
  alert(result); // 4
});
```
- 새로운 promise를 생성할 때 1초를 기다린 다음 resolve가 실행되도록 executor를 구현했기 때문에 `alert` 사이에 1초의 지연 시간을 만들 수 있음
- `.then`에서 promise를 생성하고 리턴하는 방법은 비동기적인 chain을 만드는데 사용될 수 있음

## Example: loadScript
이전 article에서 `loadScript`가 promise를 생성해서 `script`를 `value`로 전달했음  
이를 promise chaining을 사용해서 구현해보자:  
```javascript
loadScript("/article/promise-chaining/one.js")
  .then(function(script) {
    return loadScript("/article/promise-chaining/two.js");
  })
  .then(script => loadScript("/article/promise-chaining/three.js"))
  .then(function(script) {
    one();
    two();
    three();
  });
```
- `two.js`를 arrow function을 사용해서 `three.js`처럼 줄일 수 있음
- 중첩을 거듭해도 오른쪽으로 자라지 않음

위 코드에서 `.then` 안의 handler도 arrow function으로 구현할 수 있음:  
```javascript
loadScript("/article/promise-chaining/one.js").then(script1 => {
  loadScript("/article/promise-chaining/two.js").then(script2 => {
    loadScript("/article/promise-chaining/three.js").then(script3 => {
      one();
      two();
      three();
    });
  });
});
```
- 하지만 이렇게 구현하면 오른쪽으로 자라기 때문에 중첩이 많아지면 "pyramid of doom"이 발생함  
	arrow function을 사용하면 `.then`이 닫히기 전에 chaining을 해야 하기 때문에 이렇게 됨

> #### Thenables
> 엄밀히 말하면, handler는 promise를 리턴하는게 아니라, "thenable" object를 리턴함  
> thenable object는 `.then` method를 가지는 객체를 말함  
> 이 객체는 써드파티가 promise와 호환가능한 객체를 구현하는 것에서부터 시작됨  
> `.then`을 가지기 때문에 promise와 호환됨:  
> ```javascript
> class Thenable {
>   constructor(num) {
>     this.num = num;
>   }
>   then(resolve, reject) {
>     alert(resolve); // function() { native code }
>     setTimeout(() => resolve(this.num * 2), 1000);
>   }
> }
> 
> new Promise(resolve => resolve(1))
>   .then(result => {
>     return new Thenable(result);
>   })
>   .then(alert);
> ```
> - `alert(resolve)`가 실행되고 1초 뒤에 `2`가 출력됨
> - 이 기능 덕분에 `Promise`를 상속받지 않고 promise chain을 연결할 수 있음

## Bigger example: fetch
frontend에서 promise는 네트워크 요청을 위해 자주 사용됨  
`fetch`를 서버에서부터 사용자 정보를 받아오는 메소드로 사용할 것임  
이 메소드는 다양한 optional parameter가 존재하지만, 기본 문법은 아래와 같음:  
```javascript
let promise = fetch(url);
```
- `url`으로 네트워크 요청을 보내고 promise를 리턴함  
	서버가 응답을 보내면 promise는 `response` 객체가 다운로드 시작됨과 동시에 `"fulfilled"` 상태가 됨

따라서 다운로드가 완전히 종료되는 것을 확인하기 위해선 `response.text()`를 호출해야 함  
이 메소드는 텍스트 전체가 다운로드되면 이 텍스트를 `result` 값으로 갖는 resolved promise(`"fulfilled"`)를 리턴함:  
```javascript
fetch('/article/promise-chaining/user.json')
  .then(function(response) {
	return response.text();
  })
  .then(function(text) {
	alert(text); // {"name": "iliakan", "isAdmin": true}
  });
```
- `fetch`에서 다운로드 시작, 첫 번째 `.then`에서 다운로드 완료, 두 번째 `.then`에서 데이터를 처리함

`response.json()`를 사용하면 데이터를 JSON으로 parse할 수 있음:  
```javascript
fetch('/article/promise-chaining/user.json')
  .then(response => response.json())
  .then(user => alert(user.name)); // iliakan, got user name
```

이를 활용해서 아래와 같이 github에 요청을 보내서 사용자 이미지를 띄울 수 있음:  
```javascript
fetch('/article/promise-chaining/user.json')
  .then(response => response.json())
  .then(user => fetch(`https://api.github.com/users/${user.name}`))
  .then(response => response.json())
  .then(githubUser => {
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => img.remove(), 3000); // (*)
  });
```
- 유저 정보는 '/article/promise-chaining/user.json'에 저장된 상태임
- 현재는 이미지를 띄운 다음 더이상 chaining이 불가능함  
	∵ `setTimeout`으로 이미지를 없애기 때문에 이미지가 사라지기 전에 이미 script가 종료된 상태이기 때문

chaining이 가능하게 만들기 위해선 아래와 같이 이미지가 사라질 때 `resolve`를 같이 실행할 수 있도록 promise를 사용해야 함:  
```javascript
fetch('/article/promise-chaining/user.json')
  .then(response => response.json())
  .then(user => fetch(`https://api.github.com/users/${user.name}`))
  .then(response => response.json())
  .then(githubUser => new Promise(function(resolve, reject) { // (*)
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => {
      img.remove();
      resolve(githubUser); // (**)
    }, 3000);
  }))
  .then(githubUser => alert(`Finished showing ${githubUser.name}`));
```
- `(*)`의 `.then`에서 `(**)`가 실행될 때 settled되는 promise를 반환함  
	따라서 그다음의 `.then`은 이때까지 기다리게 됨

아래와 같이 코드를 정리할 수 있음:  
```javascript
function loadJson(url) {
  return fetch(url)
    .then(response => response.json());
}

function loadGithubUser(name) {
  return fetch(`https://api.github.com/users/${name}`)
    .then(response => response.json());
}

function showAvatar(githubUser) {
  return new Promise(function(resolve, reject) {
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => {
      img.remove();
      resolve(githubUser);
    }, 3000);
  });
}

// Use them:
loadJson('/article/promise-chaining/user.json')
  .then(user => loadGithubUser(user.name))
  .then(showAvatar)
  .then(githubUser => alert(`Finished showing ${githubUser.name}`));
  // ...
```

## Summary
- handler 안에서도 `resolve`나 `reject`를 사용해서 계속 chaining 가능(argument를 chain 상의 다음 부분으로 전달함)
- handler 안에서 명시적으로 promise를 선언하지 않고 chaining을 사용하면 thenable 객체를 리턴함  
	`.then` 메소드를 가지기 때문에 chaining 가능
- `.then/catch/finally`가 promise를 리턴한다면, chain의 나머지 부분은 그 promise가 settled될 때까지 기다림  
	
	|![js-promise-chaining3](https://github.com/siriyaoff/MDN-note/blob/master/images/js-promise-chaining3.PNG?raw=true)|
	|:---:|
	|javascript.info 참고|

## Tasks
### 아래 두 코드는 같은가?
```javascript
promise.then(f1).catch(f2);

// Versus:

promise.then(f1, f2);
```
- promise가 fulfilled 상태일 때
	- 첫 번째는 `f1`이 실행되고 에러가 나지 않으면 그대로 종료, 에러가 나고 `reject`가 실행되면 `f2`에서 에러 객체를 받아 error handling
	- 두 번째는 `f1`이 실행됨
- promise가 rejected 상태일 때
	- 첫 번째는 `.catch`로 바로 넘어가서 `f2`에서 error handling
	- 두 번째는 `f2`가 실행됨

# Error handling with promises
promise chain은 error handling에 뛰어남  
promise가 rejected되면, 제어 흐름은 체인 상 가장 가까운 `.catch`로 넘어감:  
```javascript
fetch('https://no-such-server.blabla') // rejects
  .then(response => response.json())
  .catch(err => alert(err)) // TypeError: failed to fetch
```
- `.catch`가 떨어져 있어도 정상적으로 작동함

error handling의 가장 편한 방법은 chain의 마지막에 `.catch`를 추가하는 것임:  
```javascript
fetch('/article/promise-chaining/user.json')
  .then(response => response.json())
  .then(user => fetch(`https://api.github.com/users/${user.name}`))
  .then(response => response.json())
  .then(githubUser => new Promise((resolve, reject) => {
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => {
      img.remove();
      resolve(githubUser);
    }, 3000);
  }))
  .catch(error => alert(error.message));
```

## Implicit try...catch
promise executor와 handler는 보이지 않는 `try...catch`를 가지고 있음  
만약 exception이 발생하면 rejection으로 간주됨:  
```javascript
new Promise((resolve, reject) => {
  throw new Error("Whoops!");
}).catch(alert); // Error: Whoops!

// same as

new Promise((resolve, reject) => {
  reject(new Error("Whoops!"));
}).catch(alert); // Error: Whoops!
```
- 이 "Implicit `try...catch`"가 에러를 잡고 rejected promise로 바꿔줌

이 동작은 executor뿐만 아니라 handler에도 적용됨:  
```javascript
// (1)
new Promise((resolve, reject) => {
  resolve("ok");
}).then((result) => {
  throw new Error("Whoops!"); // rejects the promise
}).catch(alert); // Error: Whoops!

// (2)
new Promise((resolve, reject) => {
  resolve("ok");
}).then((result) => {
  blabla(); // no such function
}).catch(alert); // ReferenceError: blabla is not defined
```
- `throw`로 에러를 발생시키는 것 이외에도 그냥 에러가 발생하면 알아서 `.catch`로 넘겨줌!

## Rethrowing
위에서 알아본 것과 같이, chain의 끝에 있는 `.catch`는 `try...catch`와 비슷한 역할을 함  
(위의 어디에서든지 에러가 나면 실행됨)

`try...catch`에서는 에러를 확인하고 처리할 수 없으면 rethrow했음  
promise에서도 비슷한 것이 가능함  
`.catch` 안에 `throw`를 넣으면 제어 흐름이 그 다음의 가장 가까운 error handler로 넘어감  
에러를 처리한 다음에는 chain 상 이후의 handler 중 가장 가까운 `.then`으로 넘어감

#### Example
```javascript
new Promise((resolve, reject) => {
  throw new Error("Whoops!");

}).catch(function(error) {
  alert("The error is handled, continue normally");

}).then(() => alert("Next successful handler runs"));
```
- `.catch`에서 에러를 처리하고 `.then`으로 넘어감

```javascript
new Promise((resolve, reject) => {
  throw new Error("Whoops!");

}).catch(function(error) { // (*)
  if (error instanceof URIError) {
    // handle it
  } else {
    alert("Can't handle such error");
    throw error;
  }

}).then(function() {
  /* doesn't run here */
}).catch(error => { // (**)
  alert(`The unknown error has occurred: ${error}`);

});
```
- `(*)`에서 에러를 잡지만 rethrow해서 바로 `(**)`로 제어 흐름이 넘어감

## Unhandled rejections
chain의 마지막에 `.catch`를 넣지 않거나 다른 이유로 에러가 처리되지 않으면 어떻게 될까?  
`try...catch`에서 이런 상황이 발생하면 script 전체가 멈춰버림  
promise에서도 비슷한 상황이 벌어짐  
JS 엔진이 rejection을 추적해서 global error를 생성함  
이는 console에서 볼 수 있음

browser 상황에서는 `unhandledrejection` event를 사용해서 그런 에러들을 잡을 수 있음:  
```javascript
window.addEventListener('unhandledrejection', function(event) {
  alert(event.promise); // [object Promise]
  alert(event.reason); // Error: Whoops!
});

new Promise(function() {
  throw new Error("Whoops!");
}); // no catch to handle the error
```
- event object는 두 가지의 property를 가짐
	- `promise` : error를 생성하는 promise
	- `reason` : 처리되지 않은 error object
- 처리되지 않은 에러가 발생할 경우 `unhandledrejection` handler가 작동됨
- 이런 에러는 주로 회복할 수 없기 때문에 보통 서버로 로그를 전송함

Node.js와 같은 non-browser 에서는 unhandled error를 추적할 수 있는 다른 방법들이 존재함

## Summary
- `.catch`는 `reject()` 호출, `throw`, 에러 발생 등의 모든 종류의 에러를 잡음
- 제어 흐름은 chain 순서대로 따라가지만, 에러가 발생하면 가장 가까운 `.catch`로 넘어감  
	`.catch`에서 에러를 성공적으로 처리하면 chain 상 가장 가까운 다음 `.then`으로 다시 넘어감
- `unhandledrejection` eent handler를 이용해서 unhandled error를 처리할 수 있음

## Tasks
### Error in setTimeout
```javascript
new Promise(function(resolve, reject) {
  setTimeout(() => {
    throw new Error("Whoops!");
  }, 1000);
}).catch(alert);
```
- 이 코드에서의 에러는 asynchronous error이기 때문에 처리되지 않음!  
	synchronous error들만 처리됨

# Promise API
`Promise` class에는 6개의 static method가 존재함

## Promise.all
여러 개의 promise가 모두 실행된 다음에 해야하는 작업이 있다고 해보자  
e.g. 여러 URL에서 다운받은 후에 한꺼번에 처리

이럴 때 `Promise.all`을 사용할 수 있다:  
```javascript
let promise = Promise.all([...promises...]);

/*-------------example--------------*/

Promise.all([
  new Promise(resolve => setTimeout(() => resolve(1), 3000)), // 1
  new Promise(resolve => setTimeout(() => resolve(2), 2000)), // 2
  new Promise(resolve => setTimeout(() => resolve(3), 1000))  // 3
]).then(alert);
```
- `Promise.all`은 promise의 배열(iterable이면 가능하지만, 보통 배열을 사용)을 인자로 받아서 promise들의 결과의 배열을 리턴함
- 위 예시에서는 3초 이후에 `1,2,3`이 출력됨

작업이 필요한 데이터의 배열을 promise의 배열로 매핑하고, 이것을 `Promise.all`으로 감싸는 방법이 자주 사용됨:  
```javascript
let urls = [
  'https://api.github.com/users/iliakan',
  'https://api.github.com/users/remy',
  'https://api.github.com/users/jeresig'
];

let requests = urls.map(url => fetch(url));

Promise.all(requests)
  .then(responses => responses.forEach(
    response => alert(`${response.url}: ${response.status}`)
  ));
```
- `urls`를 `fetch()`로 매핑해서 `requests`에 저장하고, `Promise.all()`로 감쌈

만약 배열 안의 promise 중 하나라도 rejected되면 `Promise.all`이 리턴하는 promise는 즉시 그 에러로 reject됨:  
```javascript
Promise.all([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Whoops!")), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).catch(alert); // Error: Whoops!
```

> #### In case of an error, other promises are ignored
> 하나의 promise가 reject되면, `Promise.all`은 즉시 reject하고 배열 안의 다른 promise들의 결과는 지움  
> 여러 개의 `fetch`가 있었고 그 중 하나가 실패했다고 가정하면, 나머지 `fetch`가 그대로 진행되어 완료되지만, `Promise.all`이 그 결과를 무시함
>
> promise에는 취소의 개념이 없기 때문에 `Promise.all`이 promise를 취소할 수는 없음  
> `AbortController`를 통해 취소가 가능하지만, 그것은 Promise의 API가 아님

> #### `Promise.all(iterable)` allows non-promise "regular" values in `iterable`
> `Promise.all`은 보통 promise의 iterable을 인자로 받지만, 다른 객체들도 가능함  
> 다른 객체들은 그대로 resulting array로 넘겨짐:  
> ```javascript
> Promise.all([
>   new Promise((resolve, reject) => {
>     setTimeout(() => resolve(1), 1000)
>   }),
>   2,
>   3
> ]).then(alert); // 1, 2, 3
> ```
> - `1,2,3`이 출력됨

## Promise.allSettled
`Promise.all`은 어떤 promise라도 reject되면 전체를 reject함  
이는 모든 결과가 나와야 처리가능한 "all or nothing" case에 알맞음:  
```javascript
Promise.all([
  fetch('/template.html'),
  fetch('/style.css'),
  fetch('/data.json')
]).then(render);
```

`Promise.allSettled`는 promise들의 결과에 상관없이 모든 promise가 settle 되기까지 기다림  
resulting array는 아래와 같은 상태의 객체들을 가질 수 있음
- 성공적으로 수행된 경우 `{status:"fulfilled", value:result}`
- 에러가 발생한 경우 `{status:"rejected", reason:error}`

유저들의 데이터를 받아오는데, 요청 하나가 실패해도 계속 진행하는 fetch를 구현해보자:  
```javascript
let urls = [
  'https://api.github.com/users/iliakan',
  'https://api.github.com/users/remy',
  'https://no-such-url'
];

Promise.allSettled(urls.map(url => fetch(url)))
  .then(results => { // (*)
    results.forEach((result, num) => {
      if (result.status == "fulfilled") {
        alert(`${urls[num]}: ${result.value.status}`);
      }
      if (result.status == "rejected") {
        alert(`${urls[num]}: ${result.reason}`);
      }
    });
  });
```
- 결과는 아래와 같다:  
	```javascript
	[
	  {status: 'fulfilled', value: ...response...},
	  {status: 'fulfilled', value: ...response...},
	  {status: 'rejected', reason: ...error object...}
	]
	```

### Polyfill
브라우저가 `Promise.allSettled`를 지원하지 않으면 쉽게 polyfill할 수 있다:  
```javascript
if (!Promise.allSettled) {
  const rejectHandler = reason => ({ status: 'rejected', reason });

  const resolveHandler = value => ({ status: 'fulfilled', value });

  Promise.allSettled = function (promises) {
    const convertedPromises = promises.map(p => Promise.resolve(p).then(resolveHandler, rejectHandler));
    return Promise.all(convertedPromises);
  };
}
```

## Promise.race
`Promise.all`과 비슷하지만, 먼저 settled된게 먼저 result에 저장됨:  
```javascript
let promise = Promise.race(iterable);

/*-------------example--------------*/

Promise.race([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Whoops!")), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).then(alert); // 1
```

## Promise.any
`Promise.race`와 비슷하지만, 첫 번째 fulfilled promise가 나올 때까지 기다리고 그 결과를 리턴함  
만약 모든 promise가 rejected되면 반환되는 promise는 `AggregateError`(모든 promise의 에러를 `errors` property에 저장함)으로 reject됨:  
```javascript
let promise = Promise.any(iterable);

/*-------------example--------------*/

// (1)
Promise.any([
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Whoops!")), 1000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).then(alert); // 1

// (2)
Promise.any([
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Ouch!")), 1000)),
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Error!")), 2000))
]).catch(error => {
  console.log(error.constructor.name); // AggregateError
  console.log(error.errors[0]); // Error: Ouch!
  console.log(error.errors[1]); // Error: Error
});
```

## Promise.resolve/reject
`Promise.resolve`와 `Promise.reject`는 최근에는 잘 사용되지 않음  
`async/await`로 대체 가능하기 때문

### Promise.resolve
`Promise.resolve(value)`는 `value`를 result로 하는 resolved promise를 생성함:  
```javascript
let promise = new Promise(resolve => resolve(value));

/*-------------example--------------*/

let cache = new Map();

function loadCached(url) {
  if (cache.has(url)) {
    return Promise.resolve(cache.get(url)); // (*)
  }

  return fetch(url)
    .then(response => response.text())
    .then(text => {
      cache.set(url,text);
      return text;
    });
}
```
- `cache`가 존재하면 그 값을 `value`로 하는 promise 리턴

### Promise.reject
`Promise.reject(error)`는 `error`로 reject된 promise 생성함  
아래 코드와 같은 기능임:  
```javascript
let promise = new Promise((resolve, reject) => reject(error));
```
- 실무에서는 거의 사용되지 않음

## Summary

|code|description|
|:---|:---|
|`Promise.all([...promises...])`|`promises`가 모두 resolved되면 그 결과들의 배열 리턴|
|`Promise.allSettled([...promises...])`|`promises`가 모두 settled되면 그 결과를 담은 객체들의 배열 리턴|
|`Promise.race([...promises...])`|`promises` 중에서 가장 먼저 처리된 promise의 결과 또는 에러 리턴|
|`Promise.any([...promises...])`|`promises` 중에서 가장 먼저 fulfilled되는 promise의 결과 리턴<br>`promises`가 모두 rejected면 `AggregateError` 리턴|
|`Promise.resolve(value)`<br>`Promise.reject(error)`|`value`로 resolved된 promise 리턴<br>`error`로 rejected된 promise 리턴|

- `Promise.all`은 하나라도 rejected되면 rejected promise 리턴
- `Promise.allSettled`가 리턴하는 배열은 아래 과정으로 만들어지는 객체들을 저장함:
	- `fulfilled` 상태면 `{status:"fulfilled", value:result}`
	- `rejected` 상태면 `{status:"rejected", reason:error}`
- `Promise.resolve/reject`는 사실상 거의 사용되지 않고, `Promise.all`이 이 중에서 가장 빈번하게 사용됨

> #### `Promise`의 static method와 handler(`.then/catch/finally`)는 다름
> static method들은 promise가 아닌 promises의 결과를 리턴하고, handler는 thenable을 리턴함!

# Promisification
promisfication은 callback을 실행해주는 함수를 promise를 반환하는 함수로 바꾸는 변환을 말함  
많은 함수들이 callback-based이기 때문에 이러한 변환이 자주 필요함

아래와 같이 `loadScript(src, callback)` 함수가 있다 하자:
```javascript
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;

  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`Script load error for ${src}`));

  document.head.append(script);
}

// usage:
// loadScript('path/script.js', (err, script) => {...})
```

이 함수를 `loadScriptPromise(src)`로 promisify 할 수 있음  
callback 대신 같은 기능을 하는 promise을 리턴하게 만들면 됨:  
```javascript
let loadScriptPromise = function(src) {
  return new Promise((resolve, reject) => {
    loadScript(src, (err, script) => {
      if (err) reject(err);
      else resolve(script);
    });
  });
};

// usage:
// loadScriptPromise('path/script.js').then(...)
```
- promise를 리턴하는 wrapper로 구현함  
	`loadScriptPromise`는 promise-based로 구현됨

실제로는 함수를 promisify하는 경우가 많을 수도 있기 때문에 helper를 만드는게 좋음  
`promisify(f)`는 `f`를 promisify하고 wrapper function을 리턴하는 함수임:  
```javascript
function promisify(f) {
  return function (...args) { // return a wrapper-function (*)
    return new Promise((resolve, reject) => {
      function callback(err, result) { // our custom callback for f (**)
        if (err) {
          reject(err);
        } else {
          resolve(result);
        }
      }

      args.push(callback); // append our custom callback to the end of f arguments

      f.call(this, ...args); // call the original function
    });
  };
}

// usage:
let loadScriptPromise = promisify(loadScript);
loadScriptPromise(...).then(...);
```
- `f`는 callback-based 함수로, argument의 마지막에 callback이 들어감
- `(*)`에서는 callback function을 정의하고 `f`에 `args`, `callback`을 넣어서 call forward하는 wrapper function을 리턴함
- callback은 `(**)`와 같이 promise의 executor에서 정의함

위의 `callback`은 argument가 `(err, result)`f로 고정되어 있음  
아래와 같이 `callback`을 확장 가능:  
```javascript
function promisify(f, manyArgs = false) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      function callback(err, ...results) {
        if (err) {
          reject(err);
        } else {
          resolve(manyArgs ? results : results[0]);
        }
      }

      args.push(callback);

      f.call(this, ...args);
    });
  };
}

// usage:
f = promisify(f, true);
f(...).then(arrayOfResults => ..., err => ...);
```
- `promisify`에 `manyArgs`라는 parameter를 추가함:  
	`true`면 `callback`의 인자가 여러 개라는 뜻임

callback에 `err`이 없는 것과 같이 다른 경우도 많은데, 이럴 경우 helper를 사용하지 않고 직접 promisify하면 됨  
좀 더 유연한 promisification을 제공하는 모듈들이 있음  
e.g. es6-promisify, Node.js의 `util.promisify` 등

> #### Please note:
> Promisification은 `async/await`를 사용할 때 좋은 접근이지만, callback을 완전히 대체하지는 않음  
> promise는 하나의 result를 가질 수 있지만 callback은 여러 번 불릴 수 있음  
> 따라서 promisification은 callback을 한 번만 호출하는 경우에 사용해야 함  
> promisify한 함수는 callback을 여러 번 사용할 수 없음  
> (`f(...).then()`과 같이 `f(...)`의 레퍼런스가 유지되지 않기 때문)

# Microtasks
promise handler `.then`/`.catch`/`.finally`는 항상 비동기적임  
promise가 즉시 resolve된다 하더라도 script가 먼저 끝난 후에 promise의 handler들이 실행됨:  
```javascript
let promise = Promise.resolve();

promise.then(() => alert("promise done!"));

alert("code finished");
```
- `"code finished"`가 먼저 출력된 다음에 `"promise done!"`이 출력됨!

## Microtasks queue
asynchronous task는 적절한 관리가 필요함  
이를 위해서 ECAM에서는 `PromiseJobs`라는 내부의 queue를 이용함(V8 엔진에서는 "microtask queue"라고 부름)
- 이 큐는 FIFO구조임
- task의 실행은 다른 것이 실행되고 있지 않을 때 시작됨

promise가 준비되었을 때, `.then/catch/finally` handler는 큐에 들어감  
JS 엔진이 idle 상태일 때 큐에서 task를 빼서 실행함

이때문에 위 예시에서 `"code finished"`가 먼저 출력됨

promise handler는 항상 이 내부 큐를 통해서 실행됨  
만약 promise chain이 존재한다면 각각 비동기적으로 실행됨  
i.e. handler 각각 분리되어서 큐로 들어감

그렇다면 어떻게 `"code finished"`가 `"promise done!"` 다음에 나오게 할 수 있을까?  
=> `"code finished"`도 큐에 넣으면 된다:  
```javascript
Promise.resolve()
  .then(() => alert("promise done!"))
  .then(() => alert("code finished"));
```

## Unhandled rejection
"unhandled rejection"은 promise error가 microtask queue의 마지막에서 처리되지 않았을 때 일어남  
보통 에러를 대비해서 `.catch`를 chian의 끝에 붙여놓음:  
```javascript
let promise = Promise.reject(new Error("Promise Failed!"));
promise.catch(err => alert('caught'));

window.addEventListener('unhandledrejection', event => alert(event.reason));
```
- error가 처리되었기 때문에 `unhandledrejection`이 실행되지 않음

하지만 `.catch`를 추가하는 것을 잊어버린다면, 아래와 같이 microtask queue가 비워진 다음에 엔진이 이벤트를 trigger함:  
```javascript
let promise = Promise.reject(new Error("Promise Failed!"));

window.addEventListener('unhandledrejection', event => alert(event.reason));
```
- event가 실행되어 `"Promise Failed!"`가 출력됨

error handler를 나중에 실행시켜보자:  
```javascript
let promise = Promise.reject(new Error("Promise Failed!"));
setTimeout(() => promise.catch(err => alert('caught')), 1000);

// Error: Promise Failed!
window.addEventListener('unhandledrejection', event => alert(event.reason));
```
- `"Program Failed!"`가 출력된 다음에 `'caught'`가 출력됨!
- `setTimeout`에 의해 `promise.catch`는 나중에 microtask queue에 추가되기 때문에 `window.addEventListener(...)` 줄이 실행되는 시점에서 microtask queue가 비어있어 `unhandledrejection`이 실행됨  
	이후 `.catch`가 추가되고, promise가 rejected이기 때문에 이것 또한 실행됨

## Summary
- promise handling은 microtask queue를 통하여 실행되기 때문에 항상 비동기적임  
	따라서 `.then/catch/finally` handler들은 현재 코드가 끝난 다음 실행됨
- 만약 handler가 실행된 다음 실행되어야 하는 코드가 있으면 `.then`으로 handler 뒤에 추가해야 함
- 브라우저와 Node.js를 포함한 대부분의 JS 엔진에서 microtask의 개념은 event loop, macrotask와 묶여있음

# Async/await
"async/await"를 promise와 같이 사용하면 더 편하게 사용할 수 있음

## Async functions
`async` keyword는 `function` 전에 배치됨:  
```javascript
async function f() {
  return 1;
}

f().then(alert); // 1
```
- `async`를 붙이면 항상 함수가 promise를 리턴하도록 만듦  
	다른 값들이 리턴될 경우 resolved promise로 자동으로 감쌈
- 위 예시에서는 `1`을 리턴하지만 promise로 감싸졌기 때문에 handler를 추가해서 출력 가능함

아래 코드처럼 명시적으로 promise를 리턴해도 됨:  
```javascript
async function f() {
  return Promise.resolve(1);
}

f().then(alert); // 1
```

## Await
`await`는 promise가 settled되고 result를 반환할 때까지 기다리게 만듦:  
```javascript
let value = await promise;

/*-------------example--------------*/

async function f() {

  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("done!"), 1000)
  });

  let result = await promise; // wait until the promise resolves (*)

  alert(result); // "done!"
}

f();
```
- `await`는 항상 async function 안에서 사용해야 함
- 위 예시에서는 `(*)`에서 promise가 resolve 될 때까지 기다렸다가 그 값을 `result`에 저장한 후 출력함  
	원래라면 handler를 이용해야 출력이 가능했음  
	`await`는 promise가 settled 될 때까지 함수 실행을 중지시킴

따라서 `await`는 `promise.then`보다 더 편리하게 promise의 result를 얻을 수 있고 가독성이 좋음

> #### Can't use `await` in regular functions
> non-async function에서 `await`를 사용하려 하면 syntax error가 발생함:  
> ```javascript
> function f() {
>   let promise = Promise.resolve(1);
>   let result = await promise; // Syntax error
> }
> ```

이전에 예로 들었던 `showAvatar()`를 `async/await`를 사용해서 다시 구현해보자:
1. `.then`을 `await`로 바꿔야 함
2. 함수를 `async`로 바꿔야 함

`.then`을 사용한 구현:  
```javascript
function loadJson(url) {
  return fetch(url)
    .then(response => response.json());
}

function loadGithubUser(name) {
  return fetch(`https://api.github.com/users/${name}`)
    .then(response => response.json());
}

function showAvatar(githubUser) {
  return new Promise(function(resolve, reject) {
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => {
      img.remove();
      resolve(githubUser);
    }, 3000);
  });
}

// Use them:
loadJson('/article/promise-chaining/user.json')
  .then(user => loadGithubUser(user.name))
  .then(showAvatar)
  .then(githubUser => alert(`Finished showing ${githubUser.name}`));
  // ...
```

`async/await`를 사용한 구현:  
```javascript
async function showAvatar() {

  // read our JSON
  let response = await fetch('/article/promise-chaining/user.json');
  let user = await response.json();

  // read github user
  let githubResponse = await fetch(`https://api.github.com/users/${user.name}`);
  let githubUser = await githubResponse.json();

  // show the avatar
  let img = document.createElement('img');
  img.src = githubUser.avatar_url;
  img.className = "promise-avatar-example";
  document.body.append(img);

  // wait 3 seconds
  await new Promise((resolve, reject) => setTimeout(resolve, 3000));

  img.remove();

  return githubUser;
}

showAvatar();
```
- 훨씬 간결해짐

> #### `await` won't work in the top-level code
> top-level code에서는 `await`를 사용할 수 없음:  
> ```javascript
> let response = await fetch('/article/promise-chaining/user.json');
> let user = await response.json();
> ```
> - top-level code에서 `await`를 사용하면 syntax error남
> 
> 아래와 같이 임의의 async function으로 감싸면 됨:  
> ```javascript
> (async () => {
>   let response = await fetch('/article/promise-chaining/user.json');
>   let user = await response.json();
>   ...
> })();
> ```
> 
> 8.9버전 이상의 V8 엔진에서는 모듈을 사용하면 top-level에서도 `await`를 사용 가능함

> #### `await` accepts "thenables"
> `promise.then`과 마찬가지로 `await`도 thenable 객체를 인자로 받을 수 있음  
> 따라서 `.then`을 지원하는 객체는 `await`도 적용할 수 있음
>
> `await`가 `.then`을 지원하는 non-promise 객체를 인자로 받으면, `.then`을 내장 함수인 `resolve`, `reject`를 인자로 주고 호출함(executor와 같은 역할)  
> 그다음 `await`는 `resolve`/`reject` 중 하나가 호출되기까지 기다리고, 나온 결과를 가지고 작동됨

> #### Async class methods
> async class method를 선언하려면 앞에 `async`만 붙이면 됨:  
> ```javascript
> class Waiter {
>   async wait() {
>     return await Promise.resolve(1);
>   }
> }
> 
> new Waiter()
>   .wait()
>   .then(alert); // 1
> ```
> - `await`를 메소드 안에서 사용 가능함
> - async method도 async function과 같이 항상 promise를 리턴해야 함!

## Error handling
promise가 정상적으로 resolve되면, `await promise`는 result를 리턴함  
하지만 reject될 경우, error를 반환하고, `throw`가 있는 것처럼 처리됨:  
```javascript
async function f() {
  await Promise.reject(new Error("Whoops!"));
}

// same as

async function f() {
  throw new Error("Whoops!");
}
```
- `await`가 `return`, `throw`의 역할도 함

`try...catch`로 `throw`를 잡는 것처럼 error를 잡을 수 있음:  
```javascript
async function f() {
  try {
    let response = await fetch('http://no-such-url');
  } catch(err) {
    alert(err); // TypeError: failed to fetch
  }
}

f();
```

`try...catch`를 사용하지 않아도 async function으로 생성된 promise에 `.catch`를 더해서 에러를 처리할 수 있음:  
```javascript
async function f() {
  let response = await fetch('http://no-such-url');
}

f().catch(alert); // TypeError: failed to fetch
```

`.catch`마저 사용하지 않아도, 이전에 설명했던 `unhandledrejection` event handler를 사용해서 에러를 잡을 수 있음

> #### `async/await` and `promise.then/catch`
> `async/await`를 사용하면 `.then`은 거의 사용하지 않음  
> ∵ `await`로 이미 promise가 settled 되기 때문  
> `.catch`도 `try...catch`로 대체해서 더 편리하게 사용 가능함
>
> 하지만 top-level code에서는 `async` 함수로 감싸지지 않았기 때문에 문법적으로 `await`를 사용할 수 없음  
> 따라서 `f().catch(alert)`와 같이 `.then/catch`를 사용해서 결과를 처리해야 함

> #### `async/await` works well with `Promise.all`
> 여러 개의 promise를 모두 기다려야 할 때 `Promise.all`으로 감싼 다음 `await`를 사용할 수 있음:  
> ```javascript
> let results = await Promise.all([
>   fetch(url1),
>   fetch(url2),
>   ...
> ]);
> ```
> - 에러가 발생하면 `await`를 사용하지 않을 때처럼 발생한 에러를 throw함

## Summary

|code|description|
|:---|:---|
|`async function f() { ... }`|`f()`를 async function으로 만듦|
|`let value = await promise;`|`promise`가 settled 될 때까지 함수 실행을 중지함|

- async function은 항상 promise를 리턴해야 함  
	primitive일 경우 자동으로 promise로 변환됨
- `await`는 regular function, top-level code에서는 사용할 수 없음  
	∵ async function으로 감싸지지 않았기 때문
- `await` 뒤에 thenable을 사용해도 됨  
	`await`가 `resolve`, `reject`를 제공하고 executor를 자동으로 실행시켜줌
- async method도 만들 수 있음
- `await`는 `return`, `throw`의 역할을 함  
	따라서 `try...catch` 안에서도 사용 가능함  
- `async/await`를 사용하면 `promise.then/catch`를 거의 사용하지 않음  
	`await`로 이미 promise가 settled 되었기 때문에 바로 사용하면 됨
- `await Promise.all(...)`도 가능함

## Tasks
### Rewrite using async/await
```javascript
function loadJson(url) {
  return fetch(url)
    .then(response => {
      if (response.status == 200) {
        return response.json();
      } else {
        throw new Error(response.status);
      }
    });
}

loadJson('no-such-user.json')
  .catch(alert); // Error: 404

// answer
async function loadJson(url) {
  let value=await fetch(url);
  
  if(value.status == 200){
    return value.json(); // (*)
  }
  
  throw new Error(value.status);
}
```
- 위에서 `await`를 사용했기 때문에 `(*)`에서 처리에 주의해야 함  
	`return`을 사용했기 때문에 `await` 이후에 실행되지만  
	만약 아래와 같이 구현하면 `await`를 넣는게 중요함!  
	```javascript
	let json = await response.json();
    return json;
	```

### Rewrite "rethrow" with async/await
```javascript
class HttpError extends Error {
  constructor(response) {
    super(`${response.status} for ${response.url}`);
    this.name = 'HttpError';
    this.response = response;
  }
}

function loadJson(url) {
  return fetch(url)
    .then(response => {
      if (response.status == 200) {
        return response.json();
      } else {
        throw new HttpError(response);
      }
    });
}

function demoGithubUser() {
  let name = prompt("Enter a name?", "iliakan");

  return loadJson(`https://api.github.com/users/${name}`)
    .then(user => {
      alert(`Full name: ${user.name}.`);
      return user;
    })
    .catch(err => {
      if (err instanceof HttpError && err.response.status == 404) {
        alert("No such user, please reenter.");
        return demoGithubUser();
      } else {
        throw err;
      }
    });
}

demoGithubUser();
```

아래와 같이 바꿀 수 있음:  
```javascript
async function loadJson(url) {
  let value = await fetch(url);
  if(value.status == 200){
    return value.json();
  }
  
  throw new HttpError(value);
}

async function demoGithubUser() {
  while(true){
    let name = prompt("Enter a name?", "iliakan");
    try{
      let json = await loadJson(`https://api.github.com/users/${name}`);
      break;
    } catch(err) {
      if(err instanceof HttpError && err.response.status == 404){
        alert("No such user, please reenter");
      }else{
        throw err;
      }
    }
  }

  alert(`Full name: ${user.name}.`);
  return user;
}

demoGithubUser();
```

### Call async from non-async
```javascript
async function wait() {
  await new Promise(resolve => setTimeout(resolve, 1000));

  return 10;
}

function f() {
  // ...what should you write here?
  // we need to call async wait() and wait to get 10
  // remember, we can't use "await"
}
```
- `f()`에서 `wait()`을 호출하고 그 결과를 사용해야 할 때 어떻게 구현할 수 있을까?
- `f()`는 regular function이기 때문에 `await`를 사용할 수 없음!

`wait()`에 `.then`을 붙이면 됨:  
```javascript
async function wait() {
  await new Promise(resolve => setTimeout(resolve, 1000));

  return 10;
}

function f() {
  // shows 10 after 1 second
  wait().then(result => alert(result));
}

f();
```