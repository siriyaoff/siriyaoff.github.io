---
layout: single
title: "JSnote6: Advanced working with functions"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - JavaScript
  - Tail recursion
  - Tail call optimization
  - Execution context
  - Rest parameter & spread syntax
  - arguments
  - Closure
  - Lexical Environment
  - var
  - IIFE
  - Function objects
  - NFE
  - new Function
  - setTimeout & setInterval
  - Decorators
  - Call forwarding
  - Method borrowing
  - Binding
  - Partials
---

Advanced working with functions
# Recursion and stack
## Two ways of thinking
- Iterative
- Recursive

> ※ JS는 maximal recursion depth가 대부분 `100000` 미만으로 제한되어 있음  
> tail call optimization을 사용해서 속도를 향상시킬 수 있음(safari만 지원하고 나머지는 지원하지 않음)

### Tail call optimization
tail call을 최적화하는 것  
tail recursion을 잘 설계해야 함!
- `return`에서 스택에 메모리를 쓰는 연산자를 사용하면 안됨  
	ternary operator `?`는 스택 메모리를 사용하지 않는 연산자임

#### Example
```javascript
// 1
let facto = (x, acc = 1) => {
  return (x <= 1 ? acc : facto(x - 1, x * acc));
};

// 2
let facto = x => {
  return (x <=1 ? 1 : x * facto(x-1));
};
```
- `// 1`은 `return`에 메모리를 사용하는 연산자가 존재하지 않아서 tail recursion이 실행되고, 최적화될 경우 스택이 쌓이지 않을 수도 있음  
	`// 2`는 `*`를 사용해서 메모리를 사용하기 때문에 스택이 계속 쌓임(일반적인 재귀)

## The execution context and stack
실행되고 있는 함수의 프로세스에 관한 정보는 *execution context*에 저장됨  
Execution context
- 함수의 실행에 관한 detail을 포함하는 내부 자료 구조임
	- control flow가 어디 있는지, 현재 변수, `this`의 정보 등

하나의 함수 호출은 하나의 execution context를 가짐  
함수가 중첩 호출을 가지면 다음과 같이 작동함:
1. 현재 함수가 멈춤
2. 현재 함수와 관련된 execution context가 *execution context stack*에 저장됨
3. 중첩된 호출이 실행됨
4. 중첩된 호출이 끝난 후 원래 execution context가 stack에서 회수된 다음 원래 함수가 재개됨

recursion depth는 stack 안의 최대 context의 개수와 같음

n 번 재귀 호출이 일어나는 경우 n 개의 context가 필요하지만, 반복문 기반으로 바꾸면 한 개의 context만 필요함  
=> 메모리가 절약됨

모든 재귀는 반복문으로 옮길 수 있고, 보통 반복문을 사용하는게 더 효과적임  
하지만 재귀가 복잡할 경우 옮겨도 효율성이 높아지지 않는 경우가 존재함

재귀가 더 짧고 직관적으로 읽히기 때문에 성능이 떨어짐에도 불구하고 많이 사용됨

## Recursive traversals
재귀가 잘 사용되는 곳 중 하나가 순회임  
자식이 있을 경우 재귀를 사용하면 됨

#### Example
```javascript
let company = { // the same object, compressed for brevity
  sales: [{name: 'John', salary: 1000}, {name: 'Alice', salary: 1600 }],
  development: {
    sites: [{name: 'Peter', salary: 2000}, {name: 'Alex', salary: 1800 }],
    internals: [{name: 'Jack', salary: 1300}]
  }
};

// The function to do the job
function sumSalaries(department) {
  if (Array.isArray(department)) { // case (1)
    return department.reduce((prev, current) => prev + current.salary, 0); // sum the array
  } else { // case (2)
    let sum = 0;
    for (let subdep of Object.values(department)) {
      sum += sumSalaries(subdep); // recursively call for subdepartments, sum the results
    }
    return sum;
  }
}

alert(sumSalaries(company)); // 7700
```

## Recursive structures
Recursive structure(재귀 구조)는 자기 자신과 같은 구조를 부분으로 포함하는 구조임  
⇔ 재귀의 정의

### Linked list
`Array`는 임의의 위치에 원소 삽입, 삭제가 `O(n)`이 걸림  
Linked list는 `O(1)` 만에 수행 가능

JS에서 linked list는 아래와 같이 재귀적으로 구현함:  
```javascript
let list = {
  value: 1,
  next: {
    value: 2,
    next: {
      value: 3,
      next: {
        value: 4,
        next: null
      }
    }
  }
};
```
- 임의의 원소에 대한 접근이 `O(n)`이 걸림
- `prev` property를 추가해서 DLL으로 개선할 수 있음  
	`tail` object를 추가해서 끝에서의 접근을 개선할 수 있음

## Summary
Tail call recursion은 `return`문에 메모리를 사용하는 연산자를 사용하지 않는게 중요함

# Rest parameters and spread syntax
## Rest parameters `...`
함수의 definition에 상관없이 여러 개의 arguments를 사용해서 호출 가능  
=> 필요한 만큼만 사용되고 에러가 나지 않음!  
`...`를 이용하면 나머지 arguments도 버리지 않고 저장 가능:  
```javascript
function sumAll(...args) {
  let sum = 0;

  for (let arg of args) sum += arg;

  return sum;
}

alert( sumAll(1) ); // 1
alert( sumAll(1, 2) ); // 3
alert( sumAll(1, 2, 3) ); // 6
```
- 항상 마지막에 와야 하고 `...` 뒤에 arguments가 저장될 `Array`의 이름을 적어야 함

## The "arguments" variable
`argumets`는 함수에 존재하는 array-like object임:  
```javascript
function showName() {
  alert( arguments.length );
  alert( arguments[0] );
  alert( arguments[1] );
}

showName("Julius", "Caesar");
// shows: 2, Julius, Caesar

showName("Ilya");
// shows: 1, Ilya, undefined (no second argument)
```
- 모든 arguments를 인덱스를 key로 저장함
- `length` property 존재
- array-like object이면서 iterable이지만, `Array`가 아님  
	=> `Array`의 methods를 사용할 수 없기 때문에 불편함  
	=> 현재는 rest parameter `...`가 더 많이 사용됨

> ※ Arrow functions는 "arguments"가 없음!  
> arrow function에서 `arguments` 객체를 호출하면 바깥의 일반 함수로부터 가져옴  
> ```javascript
> function f() {
>   let showArg = () => alert(arguments[0]);
>   showArg();
> }
>
> f(1); // 1
> ```
> 즉 arrow function은 `this`와 `arguments`를 outer normal function으로부터 가져옴

## Spread syntax
`Array`의 원소들을 `...`를 지원하는 함수에 arguments로 넣기 위해선, rest parameter와 반대로 `Array`를 값 하나하나로 바꿔야 함  
e.g.  
```javascript
alert( Math.max(3, 5, 1) ); // 5, works

let arr = [3, 5, 1];
alert( Math.max(arr) ); // NaN, doesn't work
```
- `arr[0], arr[1], ...`과 같이 인덱스를 이용하면 함수를 호출할 때 일일히 쳐야 함

spread syntax `...`을 사용해서 iterable object를 나열할 수 있음:  
```javascript
let arr = [3, 5, 1];
let arr2 = [8, 9, 15];

let merged = [0, ...arr, 2, ...arr2];
alert(merged); // 0,3,5,1,2,8,9,15
alert(Math.max(10, ...arr, ...arr2)); // 15

// (*)
let str = "Hello";
alert( [...str] ); // H,e,l,l,o
```
- 다른 인자들과 혼합해서 사용 가능
- iterable만 적용 가능
- `(*)`는 iterable을 `Array`로 바꾸는 것으로, `Array.from(obj)` 메소드를 사용할 수도 있음
	- `Array.from(obj)`는 array-like도 지원하기 때문에 `Object`를 `Array`로 바꾸는 작업은 이 메소드를 사용하는게 좋음

## Copy an array/object
`Object.assign(dest[, src1, src2, src3...])`를 사용해서 `Array`나 `Object`를 복사할 수 있지만, spread syntax로도 할 수 있음:  
```javascript
let arr = [1, 2, 3];
let arrCopy = [...arr];

alert(JSON.stringify(arr) === JSON.stringify(arrCopy)); // true
alert(arr === arrCopy); // false (not same reference)

let obj = { a: 1, b: 2, c: 3, d: {name: 'another'} };
let objCopy = { ...obj };
objCopy.d.name='dother';

alert(obj.d.name); // dother
alert(objCopy.d.name); // dother
alert(JSON.stringify(obj) == JSON.stringify(objCopy)); // true
```
- `JSON.stringify`로 객체를 비교할 수 있음!
- `Object.assign`과 마찬가지로 shallow copy되기 때문에 조심해야 함!

> ※ `Object.assign([], src...)`, `Object.assign({}, src...)` 둘 다 `src`의 properties를 `dest`로 추가하는 것임  
> `Array`도 `Object`에 속함(build-in object 중에 하나임)!

## Summary
`...`는 rest parameter, spread syntax로 사용됨
- L-value로 사용되면 rest parameter로 사용된 것임
- R-value로 사용되면 spread syntax로 사용된 것임

함수 안에서 `arguments`로도 인자들에 접근할 수 있지만, `Array` type이 아니기 때문에 잘 쓰이지 않음

# Variable scope, closure
JS는 function-oriented language로, 어디서든 함수를 만들고 호출할 수 있음  
만약 함수가 생성된 다음 전역변수의 값이 바뀌면? - 호출 시의 값 이용  
함수가 argument로 전달된 다음 실행된다면 외부 변수에 접근할 수 있나? - 호출 시의 위치 이용

> ※ `let/const` 변수들을 다룸  
> `var`은 `let/const`로 대체되었기 때문에 여기선 다루지 않음

## Code blocks
변수가 code block `{...}` 안에서 선언되었다면, 그 안에서만 사용 가능함:  
```javascript
{
  // show message
  let message = "Hello";
  alert(message);
}

{
  // show another message
  let message = "Goodbye";
  alert(message);
}
```
- 같은 블록 안에서 이미 선언된 변수를 `let`으로 다시 선언하는 경우 에러가 남
- `for(...){...}`은 `()`안의 변수도 `{}` 안에서 사용 가능

## Nested functions
JS에서 nested fucntion은 return될 수 있음  
=> 어디서 사용되든 선언된 곳에서의 outer variable에 대해 접근 가능:  
```javascript
function makeCounter() {
  let count = 0;
  alert(count);
  return function() {
    return count++;
  };
}

let counter = makeCounter(); // 0

alert(counter()); // 0
alert(counter()); // 1
alert(counter()); // 2

makeCounter(); // 0
alert(counter()); // 3
```
- `counter`를 여러 개 만들면 각각 독립적인가?  
	=> 독립적임

## Lexical Environment
### Step 1. Variables
JS에서 모든 실행중인 함수, code block `{...}`, script 전체는 내부에 숨겨진 object인 *Lexical Environment*를 가짐  
Lexical Environment는 두 부분으로 나뉨:
1. *Environment Record*(환경 레코드) : 모든 local variables를 properties로 저장하는 object
2. A reference to the *outer Lexical Environment* : 바깥의 코드와 관련됨

**Variable** : property of **Environment Record**  
변수를 사용하거나 바꾸는 것은 환경 레코드의 property를 사용하거나 바꾸는 것을 의미함

전체 script와 관련된 Lexical Environment를 *global* Lexical Environment라고 함  
global Lexical Environment는 outer reference가 없음!

script가 실행되면 문장이 실행되기 전에 Lexical Environment가 먼저 준비됨  
이때 Environment Record에는 변수들이 미리 등록되어 있지만, 초기화되지 않은 상태임

| ![js-lexical-environment1](https://github.com/siriyaoff/MDN-note/blob/master/images/js-lexical-environment1.PNG?raw=true) |
| :-----------------------------------------------------------------------------------------------------------------------: |
|                                                   javascript.info 참고                                                    |

- 직사각형이 Environment Record, 화살표가 outer reference를 뜻함
	- global Lexical Environment는 outer reference가 `null`로 향함
- `let` 이전까지 변수가 초기화되지 않은 상태였다가 `let`으로 선언된 뒤에는 `undefined`가 들어감

> ※ Lexical Environment는 specification object로, 이론적으로만(language specification 안에서) 존재함  
> 따라서 이 객체에 접근하거나 수정할 수 없음  
> JS의 엔진들은 specification을 준수하면서 고유한 방법으로 Lexical Environment를 최적화함(사용하지 않는 변수를 버려서 메모리를 절약하는 등)

### Step 2. Function Declarations
함수도 변수와 같이 하나의 값이지만, function declaration으로 선언된 함수들은 Lexical Environment가 생성되는 동시에 초기화됨(`<uninitialized>`가 들어가는게 아니라 `function`이 들어감)  
=> 함수의 정의 위에서도 함수 호출이 가능함  
cf. `let`으로 선언되는 변수들은 선언문이 실행되기 전까지 사용할 수 없음

Function expression은 이에 해당하지 않음에 주의!

### Step 3. Inner and outer Lexical Environment
함수가 호출되면 새로운 Lexical Environment가 자동으로 생성됨  
=> 지역 변수와 파라미터를 저장하고, 함수를 호출한 Lexical Environment가 (outer) reference로 설정됨

코드에서 변수에 접근할 때, **inner Lexical Environment를 먼저 살피고 점점 outer로 나가면서 조사함**  
=> 끝에는 global Lexical Environment를 조사함

만약 변수가 어디에도 없다면 strict mode에서는 에러가 발생함  
(non-strict에서는 존재하지 않는 변수로의 대입 연산이 새로운 global variable을 생성함, 오래된 코드와의 호환을 위해)

### Step 4. Returning a function
모든 **함수**들은 `[[Environment]]`라는 숨겨진 property가 존재함
- 함수가 만들어진 Lexical Environment으로의 reference가 저장됨
- 함수가 생성될 때 한 번 저장되고 변하지 않음

```javascript
function makeCounter() {
  let count = 0;
  
  return function() { // (*)
    return count++;
  };
}

let counter = makeCounter();

alert( counter() ); // 0
alert( counter() ); // 1
alert( counter() ); // 2
```
- `counter`의 `[[Environment]]`는 `let counter = makeCounter();`가 실행될 때 호출된 `makeCounter()`의 Lexical Environment으로의 reference임  
- outer reference는 항상 `[[Environment]]`에 저장된 reference임!!  
	=> `counter()`가 호출될 때마다 생기는 Lexical Environment의 outer reference는 global이 아니라 `counter.[[Environment]]`에 저장된 reference임!  
	
	이 예제에서는 `makeCounter()`의 Lexical Environment이기 때문에 그 안의 변수 `count`를 공유함!

> ※ Closure  
> - outer Lexical Environment의 위치를 기억하고 접근할 수 있는 함수를 뜻함
> - 프로그래밍에서 전반적으로 사용되는 용어
> - 일부 언어에서는 이 자체가 불가능하지만, JS에서는 `new Function` syntax를 제외하면 모든 함수들은 기본적으로 closure임!
> 	- `[[Environment]]` property가 자동으로 생성되기 때문

## Garbage collection
보통 Lexical Environment는 함수 호출이 끝난 다음 모든 변수들과 함께 지워짐  
∵ 다른 JS의 객체와 같이, reachable 할 때만 메모리에 보존되기 때문

하지만, 함수의 종료 후에도 reachable한 nested function이 존재한다면 nested function의 `[[Environment]]` property에 원래 함수의 Lexical Environment로의 레퍼런스가 저장됨  
=> 함수가 끝난 후에도 함수의 Lexical Environment가 reachable하기 때문에 살아있게 됨!

#### Example
```javascript
function f() {
  let value = 123;

  return function() {
    alert(value);
  }
}

let g = f(); // (1)
```
- `g.[[Environment]]`는 `// (1)`에서 호출된 `f()`의 Lexical Environment의 reference가 저장됨
- Lexical Environment도 unreachable하게 되었을 때 지워짐  
	e.g. `g`에 `null`을 저장해서 reference를 없앴을 경우

### Real-life optimizations
이론적으로, 함수가 살아있는 동안, 모든 외부의 변수들도 유지됨  
실제로는 JS 엔진들이 코드를 읽어서 사용되지 않는 외부 변수들은 지워버리는 최적화를 실행함  
=> 최적화에 의해 제거된 변수들은 디버깅 모드에서도 사용하지 못한다는 부작용이 존재함(Chrome, Edge, Opera 등의 V8 엔진에서)

```javascript
let value = "Surprise!";

function f() {
  let value = "the closest value";

  function g() {
    debugger; // in console: type alert(value); Surprise!
  }

  return g;
}

let g = f(); // (1)
g();
```
- 외부 변수 `g`가 실행되면서 만들어진 `f()`의 Lexical Environment는 `// (1)`이 끝난 후에도 유지됨  
	(∵ `g`에 들어가는 `g()`의 outer reference가 참조하고 있기 때문)  
	하지만, `f()`의 변수 `value`는 함수 `g()`에서 사용되지 않기 때문에 최적화에 의해 삭제됨  
	
	따라서 마지막에 `g();`를 실행해서 나온 디버깅 모드에서 `alert(value)`를 실행하면 `f()`의 Lexical Environment에 저장된 `value`인 `"the closest value"`가 출력되지 않고 `"Surprise!"`가 출력됨!

> ※ V8 엔진에서만 나타나는 현상임!

## Summary
함수, code block, script 전체는 Lexical Environment를 가짐
- 내부에 숨겨진 object로, 두 부분으로 나뉨:
	- Environment Record : local variables, function들을 property로 저장
		- 변수들은 선언되어 있지만, 초기화되지 않은 상태
		- function declaration으로 정의된 함수들은 초기화된 상태
	- A reference to the outer Lexical Environment

함수가 호출될 때 그 함수의 Lexical Environment가 생성됨(code block도 마찬가지)

모든 함수들의 Lexical Environment들은 `[[Environment]]`라는 숨겨진 property를 가짐
- 함수가 만들어진, 외부 Lexical Environment로의 reference가 저장됨
- 함수가 생성될 때(외부 Lexical Environment의 Environment record에 등록될 때) 한 번 저장되고 변하지 않음
- 이후 그 함수가 호출될 때마다 Lexical Environment의 outer reference는 `[[Environment]]`에 저장된 reference가 들어감
	- JS의 대부분의(`new Function` syntax 제외) 함수들이 closure인 이유임

Lexical Environment는 다른 객체들과 마찬가지로 reachable 하지 않으면 메모리에서 제거됨  
reachable하더라도 JS 엔진의 최적화에 의해, 살아있지만 내부에서도 사용되지 않는 외부 변수가 존재한다면 지워질 수 있음(V8 엔진의 최적화 방법임)

## Tasks
```javascript
function makeWorker() {
  let name = "Pete";

  return function() {
    alert(name);
  };
}

let name = "John";

// create a function
let work = makeWorker();

// call it
work(); // what will it show?
```
- `work`의 outer reference는 `makeWorker()`의 Lexical Environment로의 reference기 때문에 `"Pete"`를 참조함

```javascript
let x = 1;

function func() {
  alert(x); // (1)
  let x = 2;
}

func();
```
- "uninitialized"와 "non-existing"의 차이임!
	- `(1)`을 실행하는 시점에서 `func()`의 Lexical Environment에 `x`가 `<uninitialized>` 상태로 존재하기 때문에 사용하려 하면 에러가 발생함!
	- 뒤의 `let x=2;`를 지우면 `func()`의 Lexical Environment에는 `x`가 존재하지 않기 때문에 outer reference인 global으로 가서 `x`(= `1`)을 참조함

"ready to use" filters 만들기:  
```javascript
function inBetween(a, b) {
  return function(v) {
    if(a<=v && v<= b) return true;
    else return false;
  };
}

function inArray(arr) {
  return function(v) {
    return arr.includes(v);
  };
}

let arr = [1, 2, 3, 4, 5, 6, 7];

alert( arr.filter(inBetween(3, 6)) ); // 3,4,5,6

alert( arr.filter(inArray([1, 2, 10])) ); // 1,2
```

Army of functions:  
```javascript
function makeArmy() {
  let shooters = [];

  let i = 0;
  while (i < 10) {
    let shooter = function() { // create a shooter function,
      alert( i ); // that should show its number
    };
    shooters.push(shooter); // and add it to the array
    i++;
  }

  // ...and return the array of shooters
  return shooters;
}

let army = makeArmy();

// all shooters show 10 instead of their numbers 0, 1, 2, 3...
army[0](); // 10 from the shooter number 0
army[1](); // 10 from the shooter number 1
army[2](); // 10 ...and so on.
```
- `army`의 원소들은 모두 `makeArmy()`의 Lexical Environment를 outer reference로 가지기 때문에 실행이 끝나 `i`에 `10`이 들어간 상태로 `i`를 출력해서 같은 숫자가 나옴
- `while` 안에서 local variable `j`를 추가해서 해결 가능
- 반복문을 `for`로 바꾸기만 해도 해결됨:  
	```javascript
	function makeArmy() {
	  let shooters = [];

	  for(let i=0;i<10;i++) {
		let shooter = function() {
		  alert( i );
		};
		shooters.push(shooter);
	  }

	  return shooters;
	}
	```
	- `while`의 loop control variable인 `i`는 `makeArmy()`의 Lexical Environment에 속해있지만, `for`의 loop control variable은 `for` 안에 들어가기 때문에 `for`의 code block의 Lexical Environment 안에 `i`가 저장됨!!

# The old "var"
`var` declaration은 `let`과 비슷함  
대부분의 경우 `var`, `let`을 서로 바꿔서 사용 가능

하지만 `var`은 내부적으로 매우 다르게 동작함

## "var" has no block scope
```javascript
if (true) {
  var test = true; // use "var" instead of "let"
}

alert(test); // true, the variable lives after if
```

function-level로만 scope가 존재함:  
```javascript
function sayHi() {
  if (true) {
    var phrase = "Hello";
  }

  alert(phrase); // works
}

sayHi();
alert(phrase); // ReferenceError: phrase is not defined
```
- 오래 전에는 code block이 Lexical Environment를 생성하지 않았기 때문  
	`var` 자체가 이런 환경에서 사용되었기 때문에 code block은 상관없이 선언됨

## "var" tolerates redeclarations
`var`로 정의된 변수는 `var`로 재정의해도 에러가 나지 않음:  
```javascript
var user = "Pete";
var user = "John";

alert(user); // John
```
- 값은 정상적으로 대입됨

## "var" variables can be declared below their use
`var`을 사용하는 정의는 함수(global일 경우 script가) 시작될 때 처리됨  
=> `var`은 definition의 위치에 상관없이 호출할 수 있음:  
```javascript
function sayHi() {
  phrase = "Hello";

  alert(phrase);

  var phrase;
}
sayHi();

/*------------------------------*/

function sayHi() {
  phrase = "Hello"; // (*)

  if (false) {
    var phrase;
  }

  alert(phrase);
}
sayHi();
```
- 두 번째 예제도 정상적으로 실행됨  
	∵ code block은 무시되고 함수가 호출될 때 `phrase`의 선언이 이루어지기 때문
- *Hoisting*이라고도 함
	declaration은 호이스팅이 되지만, assignment는 그렇지 않음!  
	```javascript
	function sayHi() {
	  alert(phrase);
	  var phrase = "Hello";
	}
	
	sayHi();
	```
	- 선언은 되었지만, 값이 없는 상태이기 때문에 `undefined`가 출력됨

> ※ definition == declaration + assignment라고 생각하면 됨

## IIFE
Immediately-invoked function expressions : block-level visibility가 지원되지 않던 옛날에 이를 구현하기 위해서 만듦:  
```javascript
(function() {
  alert("Parentheses around the function");
})();

(function() {
  alert("Parentheses around the whole thing");
}());

!function() {
  alert("Bitwise NOT operator starts the expression");
}();

+function() {
  alert("Unary plus starts the expression");
}();
```
- 함수 레벨의 scope는 존재했기 때문에 `{...}` 대신 사용한 것임
	- JS는 function이라는 코드를 읽으면 function declaration의 시작이라고 인식하기 때문에 앞에 `(`나 `!`, `+` 등을 붙여서 function expression이라고 인식시킴  
		∵ function declaration으로 정의할 경우 function name이 반드시 필요하고, 정의하면서 바로 호출할 수 없기 때문
- 현재는 사용하지 않는 legacy code임!!

## Summary
`var`의 특징:
1. block scope가 존재하지 않고, 함수, global scope만 존재
2. `var`의 declaration은 함수나 script가 시작할 때 처리됨

block scope가 존재하지 않아서 현재는 잘 사용하지 않음

# Global object
global object를 사용하면 어디서든 사용 가능한 변수와 함수를 만들 수 있음  
global object는 보통 언어에 내장되어 있음  
브라우저에서는 `window`, Node.js에서는 `global` 등으로 명명됨  
최근에는 `globalThis`가 global object의 표준화된 이름으로 추가됨(대부분의 browser에서 지원됨)  
이 article에서는 `window`를 사용함

```javascript
alert("Hello");
// is the same as
window.alert("Hello");

var gVar = 5;
alert(window.gVar); // 5
```
- browser 환경에서, `var`을 사용해서 정의된 전역 함수와 변수들은 global object의 property가 됨
	- function declaration으로 global scope에서 정의된 함수도 동일하게 global object의 property가 됨
	- 모듈을 사용하는 최신의 JS에서는 일어나지 않고, 호환성을 위해서 존재하는 기능임

global available하게 만드려면 아래와 같이 global object의 property가 되도록 추가하면 됨:  
```javascript
window.currentUser = {
  name: "John"
};

alert(currentUser.name);  // John
alert(window.currentUser.name); // John
```
- 되도록 사용하지 않는게 좋음!
- 사용하게 될 경우 `window`를 붙여서 global object의 property임을 확실히 명시하는게 좋음

## Using for polyfills
최신 기능을 지원하는지 테스트하기 위해 global object를 사용할 수 있음:  
```javascript
if (!window.Promise) {
  window.Promise = ... 
}
```
- 지원하지 않을 경우 직접 polyfills를 추가할 수 있음

# Function object, NFE
JS에서 함수는 `Object` 타입으로 취급됨
- 호출, property 추가/제거, 레퍼런스로 만들어 전달 등이 가능함
- built-in properties가 존재함

## The "name" property
`name` property는 함수의 이름을 저장함:  
```javascript
let sayHi = function() {
  alert("Hi");
};

alert(sayHi.name); // sayHi

/*--------------------------------------*/

function f(sayHi = function() {}) {
  alert(sayHi.name); // sayHi
}

f();

/*--------------------------------------*/

let arr = [function() {}];

alert( arr[0].name ); // <empty string>
```
- `name` property는 specification에서 "contextual name"이라고 불림
	- 함수의 이름이 직접 제공되지 않아도, context에서 이름을 가져옴
	- 맨 마지막 예시처럼 이름을 추론할 수 없는 상황도 존재하지만, 대부분의 경우 함수는 이름을 가짐

## The "length" property
`length` property는 함수의 parameter의 개수를 리턴함:  
```javascript
function f1(a) {}
function f2(a, b) {}
function many(a, b, ...more) {}

alert(f1.length); // 1
alert(f2.length); // 2
alert(many.length); // 2
```
- rest parameter는 `length`의 개수에 포함되지 않음

아래 예제와 같이 호출할 함수의 파라미터 개수에 따라서 알맞는 함수를 호출할 때(= type introspection) 사용 가능함:  
```javascript
function ask(question, ...handlers) {
  let isYes = confirm(question);

  for(let handler of handlers) {
    if (handler.length == 0) {
      if (isYes) handler();
    } else {
      handler(isYes);
    }
  }

}

// for positive answer, both handlers are called
// for negative answer, only the second one
ask("Question?", () => alert('You said yes'), result => alert(result));
```
- polymorphism의 예시임
	- arguments를 그들의 type(이 예시의 경우, `length`)에 따라서 다르게 처리하는 것

## Custom properties
custom property도 추가 가능:  
```javascript
function sayHi() {
  alert("Hi");

  sayHi.counter++;
}
sayHi.counter = 0;

sayHi();
sayHi();
alert( `Called ${sayHi.counter} times` ); // Called 2 times
sayHi.counter = 10;
alert( `Called ${sayHi.counter} times` ); // Called 10 times
```
- function property와 variable은 연관이 없음
	- `sayHi.counter = 0`은 local variable이 아님
- function property `counter`는 함수 바깥에 저장됨  
	안에 저장하면 호출될 때마다 초기화되기 때문
- closure는 function property로 대체 가능함

Closure을 이용한 버전:  
```javascript
function makeCounter() {
  function counter() {
    return counter.count++;
  };

  counter.count = 0;

  return counter;
}

let counter = makeCounter();
alert( counter() ); // 0
alert( counter() ); // 1
counter.count = 10;
alert( counter() ); // 10
```
- `makeCounter()`의 Lexical Environment가 변수 `counter`에 의해 유지되면서 `count` property가 증가함
- `count`가 완전히 외부가 아닌 중간의 `makeCounter()`의 Lexical Environment에 저장됨  
	=> 외부에서 `counter.count`를 접근하지 못하지만, 이 경우에는 `makeCounter()`가 `counter()`를 반환하기 때문에 외부의 변수 `counter`에 `counter()`가 저장되고 이 변수로 `counter()`의 property를 접근 가능!

`count`에 대한 외부의 접근을 제한한 버전:  
```javascript
function makeCounter() {

  function counter() {
    return count++;
  };

  count = 0;

  return counter;
}

let counter = makeCounter();

alert( counter() ); // 0
alert( counter() ); // 1
alert( counter() ); // 2
```
- 외부에서는 `count`에 접근할 수 없음  
	오직 `counter()`으로만 접근 가능  
	cf. function property를 이용하면 외부에서도 함수에 대한 reference만 있으면 접근 가능  
	
	=> 목적에 따라 구현 방법을 선택하면 됨

## Named Function Expression
```javascript
let sayHi = function func(who) {
  if (who) {
    alert(`Hello, ${who}`);
  } else {
    func("Guest"); // use func to re-call itself
  }
};

sayHi(); // Hello, Guest

func(); // Error, func is not defined (not visible outside of the function)
```
- function expression에 이름을 추가한 것임
	- 이름을 추가한다고 function declaration이 되진 않음
	- 정상적으로 호출 가능함
- NFE의 기능:
	- 함수가 내부에서 자신을 호출 가능하게 만듦
	- 함수 바깥에서는 호출할 수 없음
- 함수의 이름으로 호출하는 경우:  
	```javascript
	let sayHi = function(who) {
	  if (who) {
		alert(`Hello, ${who}`);
	  } else {
		sayHi("Guest"); // Error: sayHi is not a function
	  }
	};

	let welcome = sayHi;
	welcome(); // Hello, Guest
	
	sayHi = null;
	welcome(); // Error, the nested sayHi call doesn't work any more!
	```
	- 외부에서 sayHi가 수정될 경우 에러가 발생할 수 있음!  
		=> NFE를 사용해서 해결

> ※ function declaration에서 함수 이름을 사용하는건 recursion  
> NFE 같이 internal name이 필요하지 않음(확실한 함수 이름이 존재함)

## Summary
- 함수는 `name`, `length`(parameter의 개수) 등의 built-in property를 가짐
- custom property를 추가할 수도 있음
	- closure으로 대체 가능  
		closure으로 구현하는 경우 외부에서의 접근 불가
- function expression에 이름을 추가할 수 있음(NFE)  
	=> function expression으로 선언되는 함수의 property 호출, 재귀 등을 지원 가능  
	
	lodash library에서는 `_` 함수 안에 helper function들을 선언해서 global space의 낭비를 줄이고 이름 충돌의 가능성을 줄임

## Tasks
`sum` 함수를 아래 예시와 같이 구현하기:  
```
alert( sum(1)(2) ); // 3
alert( sum(5)(-1)(2) ); // 6
alert( sum(6)(-1)(-2)(-3) ); // 0
alert( sum(0)(1)(2)(3)(4)(5) ); // 15

// (1) using closure + function property
function sum(val) {
  function f(cur) {
    f.cursum+=cur;
    return f;
  }
  
  f.cursum=val;
  f.toString = function(){
    return f.cursum;
  };
  return f;
}

// (2) using closure
function sum(val) {
  let cursum = val;
  function f(cur) {
    cursum+=cur;
    return f;
  }
  
  f.toString = function(){
    return cursum;
  };
  
  return f;
}
```
- function property는 항상 함수 안이나 함수가 선언된 영역, 두 곳 모두에서 선언 가능
	- closure로 사용된 함수도 function property를 선언해서 사용 가능함
		- closure 안에서 선언하면 호출될 때마다 새로 선언됨  
			outer Lexical Environment에서 선언되면 초기화되지 않음
		- 아예 밖에서는 closure의 property 호출 불가
- `f`를 재귀로 만들기 위해선 parameter `acc`를 넣어야 하기 때문에 예시와 맞지 않음
- `f`가 더 이상 호출되지 않을 때 출력을 위해 `f`의 `toString` method를 정의해야 함!
- `(1)`의 `f.cursum`은 `f`의 reference가 있으면 외부에서도 수정할 수 있지만, `(2)`의 `cursum`은 외부에서 접근 불가능

> ※ closure에서 자신을 리턴하는 것은 재귀가 아님!!  
> 위의 코드에서 `f()` 안에서 `return f`는 재귀가 아님  
> cf. `return f()`였으면 재귀지만, 호출하지 않았으므로 그냥 함수를 반환하는 것 뿐임!

# The "new Function" syntax
`new Function`으로도 함수를 생성할 수 있음  
잘 쓰이지 않지만, 이 방법으로만 해결할 수 있는 상황이 존재함

## Syntax
```javascript
let func = new Function ([arg1, arg2, ...argN], functionBody);

new Function('a', 'b', 'return a + b'); // basic syntax
new Function('a,b', 'return a + b'); // comma-separated
new Function('a , b', 'return a + b'); // comma-separated with spaces

/*-------------example--------------*/

let sum = new Function('a', 'b', 'return a + b');
alert( sum(1, 2) ); // 3

let sayHi = new Function('alert("Hello")');
sayHi(); // Hello
```
- `functionBody`에 있는 string 그대로 함수를 구현함  
	=> run time에 함수를 정의할 수 있음  
	e.g. 서버로부터 함수 코드를 받는 등의 상황에서 사용

## Closure
`new Function`으로 생성된 함수들은 `[[Environment]]`가 global Lexical Environment로 설정됨  
=> closure으로 사용되어도 outer Lexical Environment에 존재하는 properties를 사용할 수 없음:  
```javascript
function getFunc() {
  let value = "test";
  let func = new Function('alert(value)');

  return func;
}

getFunc()(); // error: value is not defined
```
- 구조적으로 더 안전함  
	∵ `new Function`으로 생성되는 함수들은 프로그램이 실행 중에 정의되는데, 이 때는 이미 minifier에 의해 변수들 이름이 내부적으로 단축된 이후이기 때문에 변수명을 알 수 없음  
	
	=> `new Function`은 외부 변수에 접근하면 안됨  
	`[arg1, arg2, ...argN]`과만 상호작용 해야 함

## Summary

| code                                                             | description                       |
| :--------------------------------------------------------------- | :-------------------------------- |
| `let func = new Function ([arg1, arg2, ...argN], functionBody);` | `new Function`을 이용한 함수 정의 |

- `new Function`들은 `[[Environment]]`가 global Lexical Environment로 설정됨  
	=> 외부 변수들을 건드릴 수 없음  
	
	global variables는 사용 가능함

# Scheduling: setTimeout and setInterval
함수를 일정 시간 이후에 실행하는 것을 "scheduling a call"이라고 함  
두 가지 method로 구현할 수 있음:
- `setTimeout` : 일정 시간 후에 함수를 실행시킬 수 있음
- `setInterval` : `setTimeout`을 반복하는 것과 같음(일정 시간 후에 함수 실행을 반복)

JS specification에 명시되어 있지 않지만, 대부분의 browser와 Node.js 등의 환경에서 지원됨

## setTimeout
```javascript
let timerId = setTimeout(func|code[, delay[, arg1, arg2, ...]]);

/*-------------example--------------*/

function sayHi(phrase, who) {
  alert( phrase + ', ' + who );
}
setTimeout(sayHi, 1000, "Hello", "John"); // Hello, John

setTimeout(() => alert('Hello'), 1000);
```
- `func|code` : 실행시킬 함수나 코드
	- 코드도 허용은 되지만 보안상 사용하지 않는게 좋음
- `delay` : 실행시키기 전에 delay, `ms` 단위
	- default : 0ms
- `arg1`, `arg2`... : 함수에 필요한 arguments
- 타이머를 식별하기 위해 만들어진 ID(숫자)가 반환됨  
	=> `clearTimeout` method로 실행을 취소시킬 수 있음:  
	```javascript
	let timerId = setTimeout(...);
	clearTimeout(timerId);
	```
	
	cf. Node.js에서는 `timeId`가 timer object를 반환함

## setInterval
```javascript
let timerId = seInterval(func|code[, delay[, arg1, arg2, ...]]);

/*-------------example--------------*/

let timerId = setInterval(() => alert('tick'), 2000);

setTimeout(() => { clearInterval(timerId); alert('stop'); }, 5000);
```
- `setTimeout`과 문법, 사용 방법이 같음
- `setTimeout`은 한 번만 실행하지만, `setInterval`은 interval을 계속 반복함
- `clearInterval(timerID)`로 취소시킬 수 있음
- 위 예시에서는 2초마다 `alert`를 반복하지만, 5초가 지나면 중지하도록 구현됨  
	Scheduling과 관련된 method들은 function declaration처럼 미리 초기화되고 한꺼번에 실행되는 듯

> ※ `alert`가 실행되고 있을 때도 timer의 시간은 계속 계산됨  
> => `setInterval(() => alert('tick'), 2000);`을 실행하면 사용자가 확인을 누르는 시간도 `2000ms`에 포함되기 때문에 사용자가 느끼는 `alert`가 반복되는 시간은 더 짧음!

## Nested setTimeout
반복적으로 무언가를 실행하는 것은 두 가지 방법으로 구현 가능함  
`setInterval`을 사용하거나 `setTimeout`을 중첩하는 것임:  
```javascript
let timerId = setInterval(() => alert('tick'), 1000);

let timerId = setTimeout(function tick() {
  alert('tick');
  timerId = setTimeout(tick, 1000);
}, 1000);
```
- `setTimeout`을 중첩해서 사용하는 것이 더 유용한 코드를 만들 수 있음  
	- interval을 늘려야 할 경우 안에서 수정 가능함  
	- 정확하게 `delay` 만큼 지연되는게 보장됨
		- `setInterval`은 함수를 실행시킨 시점부터 시간을 재지만, nested `setTimeout`은 함수가 끝난 시점부터 시간을 재기 때문  
			(확실하지 않음)

> ※ `setInterval`, `setTimeout`의 Garbage collection  
> `clearInterval`, `clearTimeout`이 호출되기 전까지 `func`의 reference도 유지되기 때문에 garbage collect되지 않음  
> outer Lexical Environment를 참조하는 함수가 존재하면 `setInterval`이 실행되는 동안에는 그 Lexical Environment도 garbage collect되지 않기 때문에 주의해야 함  
> 사용이 끝나면 `setInterval`으로 취소하는게 메모리 부하를 막음

## Zero delay setTimeout
`setTimeout(func,0)`이나 `setTimeout(func)`로 `delay` 없이 사용하면 현재 script가 끝난 직후 `func`을 실행하게 됨  
browser에서는 zero-delay timeout을 활용해서 event loop를 설정 가능

#### Example
```javascript
setTimeout(() => alert("World"));
alert("Hello");
alert("Hello");
```
- `"Hello"`가 두 번 출력된 다음 `"World"`가 출력됨

> ※ zero delay가 실제로 지연 시간이 없지는 않음  
> HTML5의 경우, timer가 5번 이상 중첩될 경우, 반복되는 구간은 최소 4ms 이상으로 조정됨  
> `setTimeout` 뿐만 아니라 `setInterval`에도 적용됨  
> 지연 없이 구현해놓아도 처음에만 지연 없이 실행되다가 나중에는 지연시간이 생김  
> e.g. `0, 0, 0, 0, 8, 6, 5 ms`  
> 예전에 생긴 specification에 명시되어 있는데, legacy code 중에서 이 조항을 고려해서 구현된게 많음  
> 서버에서는 이런 제한이 없고, 즉각적인 비동기 작업을 구현하는 다른 방법들이 존재함  
> 즉, 이 제한은 browser에 한정됨

## Summary

| code                     | description                            |
| :----------------------- | :------------------------------------- |
| `setTimeout(func         | code[, delay[, arg1, arg2, ...]])`     | `delay` ms 만큼 기다린 후에 `func`를 실행시킴<br>현재 timerId를 반환함(숫자)               |
| `clearTimeout(timerId)`  | `timerId`에 해당하는 타이머를 취소시킴 |
| `seInterval(func         | code[, delay[, arg1, arg2, ...]])`     | `delay` ms 만큼 기다린 후에 `func`를 실행시키는 작업을 반복<br>현재 timerId를 반환함(숫자) |
| `clearInterval(timerId)` | `timerId`에 해당하는 타이머를 취소시킴 |

- scheduling method들은 현재 script가 끝난 다음 한꺼번에 실행됨!  
	e.g. 2초마다 출력, 5초에 2초 타이머 취소가 있으면, 출력이 2번 되고 1초 후에 2초의 타이머가 취소됨  
	cf. `Date`의 method(`Date.now()` 등)들은 다른 statements와 같이 순서대로 실행됨(`alert`가 앞에 있으면 확인이 눌러진 다음 실행됨)
- nested `setTimeout`이 `delay` 만큼 지연되는 것이 보장되고 더 유연한 구현이 가능함  
	cf. `setInterval`은 `func`을 실행하는데 소요되는 시간도 timer에 포함됨
- zero-delay scheduling은 browser 환경에서 5번의 호출 이후 지연 시간이 최소 4ms 이상으로 조절됨
- timer의 최소 지연 시간은 300ms ~ 1000ms까지 늘어남(browser나 OS의 설정에 따라 다름)

## Tasks
1초마다 증가하는 숫자 출력:  
```javascript
function printNumbers(from, to) {
  let current = from;
  setTimeout(function go(){
    alert(current);
    if(current<to) setTimeout(go, 1000);
    current++;
  }, 1000);
}

function printNumbers2(from, to) {
  let current = from, tid=setInterval(function() {
    alert(current);
    if(current == to) clearInterval(tid);
    current++;
  }, 1000);
  
}
```
- `clearInterval`도 `function()`의 다른 script가 끝난 다음 비교되기 때문에 이렇게 구현 가능

# Decorators and forwarding, call/apply
JS는 함수를 다루는데 유연함  
함수를 인자로 넘기거나 객체로 사용할 수 있고, *forwarding*, *decorating*도 가능함

## Transparent caching
CPU를 많이 사용하지만, 결과가 stable한(항상 일정한 결과가 나옴) 함수 `slow(x)`가 있다고 가정하자  
이 함수가 자주 호출된다면, 결과를 캐싱하는게 나음  
하지만 `slow()` 안에 캐시를 추가하는 것보다, wrapper function을 추가하는게 더 좋음

#### Example
```javascript
function slow(x) {
  // there can be a heavy CPU-intensive job here
  alert(`Called with ${x}`);
  return x;
}

function cachingDecorator(func) {
  let cache = new Map();

  return function(x) {
    if (cache.has(x)) {    // if there's such key in cache
      return cache.get(x); // read the result from it
    }

    let result = func(x);  // otherwise call func

    cache.set(x, result);  // and cache (remember) the result
    return result;
  };
}

slow = cachingDecorator(slow);

alert( slow(1) ); // slow(1) is cached and the result returned
alert( "Again: " + slow(1) ); // slow(1) result returned from cache

alert( slow(2) ); // slow(2) is cached and the result returned
alert( "Again: " + slow(2) ); // slow(2) result returned from cache
```
- `Map`을 이용해서 caching 가능
- `slow(x)`를 호출하면, `slow()`가 호출되는 대신, `cachingDecorator`에 의해 반환된 caching wrapper가 호출됨  
	=> `cachingDecorator`가 *decorator*임!
	- 다른 함수를 가져와서 그것의 작동을 바꿈(이 예시에서는 caching을 추가)  
		decorator 안에 cache를 만들어서 함수의 code도 더 간단해짐
	- `cachingDecorator(func)` 안의 closure `function(x)`는 `func(x)`의 wrapper라고도 함
	- `slow()`의 기능은 동일하고, caching만 추가됨
- caching decorator를 사용하면 아래와 같은 장점이 있음:
	- `cachingDecorator`는 reusable함 => 어느 함수에든 적용 가능
	- caching logic이 분리되어 `slow()` 자체의 복잡성이 증가하지 않음
	- 필요하면 여러 개의 decorator를 조합할 수 있음

## Using "func.call" for the context
object method는 decorator를 사용하기에 적합하지 않음  
(∵ method 안에서 `this`를 호출하는 부분이 존재하면, decorator로 옮겨졌을 때 `this`가 `undefined`를 반환하기 때문)

#### Example
```javascript
let worker = {
  someMethod() {
    return 1;
  },

  slow(x) {
    // scary CPU-heavy task here
    alert("Called with " + x);
    return x * this.someMethod(); // (*)
  }
};

function cachingDecorator(func) {
  let cache = new Map();
  return function(x) {
    if (cache.has(x)) {
      return cache.get(x);
    }
    let result = func(x); // (**)
    cache.set(x, result);
    return result;
  };
}

alert( worker.slow(1) ); // the original method works

worker.slow = cachingDecorator(worker.slow); // now make it caching

alert( worker.slow(2) ); // Whoops! Error: Cannot read property 'someMethod' of undefined
```
- `(**)`에서 원래 method인 `slow()`를 호출  
	=> `slow()`가 실행되고 `(*)`에서 `this`를 호출  
	=> `undefined`가 반환되어 `undefined.someMethod()`를 호출하기 때문에 에러가 남
- caching decorator가 아닌 다른 wrapper로 옮겨서 호출해도 같은 결과가 나옴:  
	```javascript
	let func = worker.slow;
	func(2);
	```
	∵ `this`는 함수가 실행될 때 값이 계산됨

`Array`의 method들은 `thisArg`를 사용해서 해결 가능하지만, 일반적인 method들은 `thisArg`가 parameter에 없는 경우가 많음  
=> 내장 method인 `func.call([context[, ...args]])`를 사용해서 해결 가능
- `context` : `thisArg`와 같은 역할로, 함수 내 `this`가 가리키는 것을 지정함

#### Example
```javascript
function sayHi() {
  alert(this.name);
}

let user = { name: "John" };

sayHi.call( user ); // John
```
- argument 없이 실행할 수도 있음

위의 decoration에 `func.call(thisArg)`를 사용:  
```javascript
function cachingDecorator(func) {
  let cache = new Map();
  return function(x) {
    if (cache.has(x)) {
      return cache.get(x);
    }
    let result = func.call(this, x); // "this" is passed correctly now
    cache.set(x, result);
    return result;
  };
}
```
- `worker.slow = cachingDecorator(worker.slow);`가 실행될 때 method `worker.slow`가 `func`로 들어감  
	=> `func` 자체가 `worker.slow`라는 method이기 때문에 `func.call(this, x)`를 호출하면 `this`가 점 앞의 객체인 `worker`가 됨

즉, decorator를 사용할 때 decorate할 `func`가 `this`를 사용한다면, `wrapper` 안에서 `func`를 호출할 때 `func.call(this, ...args)`로 호출하면 됨!

## Going multi-argument
argument가 여러 개인 함수도 caching을 시킬 수 있을까?  
argument가 하나일 때는 `(x, res)`를 `Map`에 넣어서 caching을 구현했지만, `Map`이 key로 하나의 값만 저장 가능하기 때문에 인자가 여러 개일 경우 이 방법이 불가능함

가능한 방법들:
1. multi-key가 허용되는 map-like 자료 구조를 구현
2. nested map을 사용  
	e.g. `cache.get(arg1).get(arg2)`
3. hashing 등으로 두 개의 값을 하나로 만듦

세 번째 방법이 주로 사용됨

`func.call()`에 여러 인자를 넣기 위해서 모든 함수에 존재하는 array-like object `arguments`와 spread syntax `...` 이용:  
```javascript
let worker = {
  slow(min, max) {
    alert(`Called with ${min},${max}`);
    return min + max;
  }
};

function cachingDecorator(func, hash) {
  let cache = new Map();
  return function() {
    let key = hash(arguments); // (*)
    if (cache.has(key)) {
      return cache.get(key);
    }

    let result = func.call(this, ...arguments); // (**)

    cache.set(key, result);
    return result;
  };
}

function hash(args) {
  return args[0] + ',' + args[1];
}

worker.slow = cachingDecorator(worker.slow, hash);

alert( worker.slow(3, 5) ); // works
alert( "Again " + worker.slow(3, 5) ); // same (cached)
```
- 이제 `worker.slow`는 임의의 개수의 인자를 넣어도 작동함(`hash`는 수정해야 함)
- `(*)`에서 `hash()`를 사용해서 `key`를 만듦  
	`(**)`에서 `this`와 `...arguments`를 사용해서 `result`를 만들고 cache에 넣음

## func.apply
`func.call([this[, ...arguments]])` 대신 `func.apply(this[, arguments])`를 사용 가능
`func.call`과의 차이점:  
- `func.apply`에서는 array-like `args`만 허용됨  
	cf. `func.call`에서는 argument로 iterable `args`가 `...`로 인해 나열되어서 들어감

=> `Array` 같이 array-like이면서 iterable인 객체의 경우 `apply`와 `func` 두 곳 모두에 들어갈 수 있지만, array-like가 더 최적화가 잘 되어있기 때문에 `apply`에 넣는게 실행 속도가 더 빠름

context(`this`)와 모든 arguments를 같이 다른 함수로 넘기는 것을 *call forwarding*이라고 함:  
```javascript
let wrapper = function() {
  return func.apply(this, arguments);
};
```
- decorator는 closure 안에서 call forwarding을 사용함
	주로 closure와 함께 사용될 듯(single responsibility principle + 보안)
- `wrapper`를 호출해서 `func`를 실행하는 것과 `func`를 호출하는 것은 구분할 수 없음

## Borrowing a method
위의 hashing function은 두 개의 arguments에 대해서만 작동함  
*method borrowing*을 이용해서 이것을 개선할 수 있음:  
```javascript
function hash() {
  alert( [].join.call(arguments) ); // 1,2
}
```
- `arguments`는 iterable이면서 array-like이지만, `Array`가 아니기 때문에 `Array`의 method인 `arr.join()`을 사용할 수 없음  
	=> `func.call()`을 사용해서 method `join`을 빌릴 수 있음!
- `call()`의 `this`에 `arguments`를 넣은 것임  
	`join()`이 `this[0...n]`을 추가하도록 구현되어있기 때문에 가능함
- 이 경우에는 `Array.from()`을 사용해도 될 듯

## Decorators and function properties
original function이 function property를 가지고 있을 경우 decorator로 함수를 바꾸면 function property는 지원되지 않음  
function property를 유지하는 decorator를 만들기 위해선 `Proxy` object를 사용해야 함!

## Summary

| code                               | description                                                                                 |
| :--------------------------------- | :------------------------------------------------------------------------------------------ |
| `func.call([context[, ...args]])`  | `this`를 `context`로 설정하고 `...args`를 argument로 받는 `func` 실행                       |
| `func.apply(context[, arguments])` | `this`를 `context`로 설정하고 array-like `arguments`를 argument의 리스트로 받는 `func` 실행 |
| `tempobj.method.call(arguments)`   | method borrowing                                                                            |

- argument가 많을 경우 `call` 보다는 `apply`가 더 빠름  
	∵ array-like가 최적화가 더 잘 되어있기 때문
- call forwarding : context와 arguments를 다른 함수로 넘기는 것  
	call forwarding의 기본적인 구조:  
	```javascript
	let wrapper = function() {
	  return original.apply(this, arguments);
	};
	```
- method borrowing : method를 사용하기 위해 빈 객체를 임시로 만들고 method를 사용하는 것
- decorator는 어떤 함수를 argument로 받아서 그 함수의 동작을 바꾸는 wrapper임
	- 함수 자체(주 기능과 복잡성, 코드)는 그대로 유지되어야 함
	
	caching decorator의 기본적인 구조:
	```javascript
	function cachingDecorator(func, hash) {
	  let cache = new Map();
	  return function() { // (1)
		let key = hash(arguments); // (2)
		if (cache.has(key)) {
		  return cache.get(key);
		}

		let result = func.apply(this, arguments); // (3)

		cache.set(key, result);
		return result;
	  };
	}

	function hash(args) {
	  return [].join.call(args); // (4)
	}
	```
	- `(1)`에서 closure 사용  
		cache storage를 구현하기 위함
	- `(2)`에서 object `arguments` 사용  
		`func`의 arguments가 많을 때 key의 생성을 위함
	- `(3)`에서 call forwarding  
		`func`가 `this`를 사용하고 arguments가 많은 method일 경우 대비
	- `(4)`에서 method borrowing  
		array-like `arguments`는 `join()` method를 호출할 수 없기 때문
- array-like `arguments`를 다룰 때 method borrowing을 많이 사용함  
	`Array`인 rest parameter를 대안으로 사용 가능

## Tasks
### Spy decorator
```javascript
function spy(func) {
  function wrapper(...args) {
    wrapper.calls.push(args);
    return func.apply(this, args);
  }

  wrapper.calls = [];

  return wrapper;
}

work = spy(work);

work(1, 2); // 3
work(4, 5); // 9

for (let args of work.calls) {
  alert( 'call:' + args.join() ); // "call:1,2", "call:4,5"
}
```
- function expression에는 function property를 추가할 수 없음

### Delaying decorator
```javascript
function delay(f, ms) {
  return function() {
	// return setTimeout(f.apply(this, arguments), ms); // (1)
	return setTimeout(() => f.apply(this, arguments), ms); // (2)
  };
}

let f1000 = delay(f, 1000);
let f1500 = delay(f, 1500);

f1000("test"); // shows "test" after 1000ms
f1500("test"); // shows "test" after 1500ms
```
- `setTimeout`에는 실행시킬 함수가 들어가야 함  
	expression이 들어갈 경우 계산된 결과가 `setTimeout`의 argument로 들어감
	- `(1)`은 `f.apply`를 실행한 반환값을 호출하는 것이기 때문에 의도한대로 작동하지 않음  
		`(2)`는 arrow function이 하나의 함수이기 때문에 `ms` ms가 지난 후 `f.apply`가 호출됨  
		(+ arrow function에서 `this`를 사용하면 outer function의 `this`가 사용되기 때문에 function expression의 `this`가 제대로 들어감)  
		=> `f`가 method여도 정상적으로 실행됨
	
	아래와 같이 구현해도 됨:  
	```javascript
	function delay(f, ms) {
	  return function(...args) {
		let savedThis = this;
		setTimeout(function() {
		  f.apply(savedThis, args);
		}, ms);
	  };
	}
	```

### Debounce decorator
`f`가 호출되면, 가장 마지막 호출 기준으로 `ms` ms 이후에 그 호출만 실행되게 만듦:  
```javascript
let f = debounce(alert, 1000);

f("a");
setTimeout( () => f("b"), 200);
setTimeout( () => f("c"), 500);

function debounce(func, ms) {
  let timeout;
  return function() {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, arguments), ms);
  };
}
```
- 이 예제에서는 `f()`를 on time, 200ms, 500ms에 호출하기 때문에, 1500ms 이후에 `f("c")`를 호출해야 함  
	=> `f`가 호출될 때마다 이전의 `setTimeout`을 취소하고 시간을 바꿔서 새로 호출함
- `f("a")`가 `alert`를 직접 호출하는게 아님에 주의!!  
	이것도 바로 실행하지 않고 `ms` ms 이후에 실행되도록 예약됨

### Throttle decorator
`f`가 호출되면, `ms` ms 이내에 최대 한 번만 호출되게 만듦:  
```javascript
function f(a) {
  alert(a);
}

let f1000 = throttle(f, 1000);

f1000(1); // shows 1
f1000(2); // (throttling, 1000ms not out yet)
f1000(3); // (throttling, 1000ms not out yet)
// when 1000 ms time out...
// ...outputs 3, intermediate value 2 was ignored

function throttle(func, ms) {
  let isthrottled=false, cargs, cthis;
  function wrapper() {
    if(isthrottled) {
      cargs=arguments;
      cthis=this;
      return;
    }
    isthrottled=true;
    
    func.apply(this, arguments);
    setTimeout(function() {
      isthrottled=false;
      if(cargs) {
        wrapper.apply(cthis, cargs);
        cargs=cthis=null;
      }
    }, ms);
  }
  return wrapper;
}
```
- `isthrottled`가 `true`이면 현재 throttling되고 있음(`ms` ms 이내의 시간이므로 무시되는 중)  
	
	`false`라면 첫 실행이거나 다시 작동시켜야 하는 상황이기 때문에 `true`로 바꾸고 `func` 실행  
	이후 `isthrottled`를 `false`로 바꾸고 `wrapper`를 다시 호출하는 함수를 `ms` ms 이후에 호출  
	이 때 `cargs`가 `null`이 아니면 중간에 무시된 호출이 있다는 뜻이기 때문에 가장 마지막 인자들로 `wrapper`를 호출함(`func`를 호출하지 않고 `wrapper`를 호출하는 이유는 이 호출도 `ms` 안에 최대 한 번 실행되어야 하는 제한이 적용되어야 하기 때문)

> #### debounce와 throttle의 차이
> debounce는 모든 호출을 무시한 뒤 마지막 호출을 `ms` 뒤에 실행되게 만듦  
> throttle은 `ms` 이내에 최대 한 번 실행되게 만듦

# Function binding
method를 callback으로 보낼 경우, 실행될 때 `this`를 제대로 찾지 못한다는 문제가 있음

## Losing "this"
```javascript
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

setTimeout(user.sayHi, 1000); // Hello, undefined!
```
- `user.sayHi`가 object와 떨어져서 호출되어 `this.firstName`을 찾지 못했기 때문에 `undefined`가 출력됨
- 브라우저에 구현된 `setTimeout`는 `this`를 `window`로 설정함(Node.js는 `this`가 timer object가 됨)  
	=> `this.firstName`이 `undefined`가 됨  
	cf. `setTimeout` 이외의 다른 경우에는 `this` 자체가 아예 `undefined`가 되어버림
- wrapper나 binding을 사용해서 context에 맞게 함수를 호출할 수 있음

## Solution 1: a wrapper
```javascript
setTimeout(function() {
  user.sayHi(); // Hello, John!
}, 1000);
```
- `user.sayHi`를 `setTimeout`의 argument로 넘기면 이 함수의 코드만 넘어가지만, wrapper를 이용하면 `user`의 method `sayHi()`가 실행됨  
	```javascript
	setTimeout(() => user.sayHi(), 1000); // Hello, John!
	```
	도 같은 이유로 정상적으로 실행됨
	
	`setTimeout`이 실행되기 전에 `user`의 정보가 바뀐다면 예상과 다른 결과가 나올 수 있음!  
	=> bind로 해결 가능

## Solution 2: bind
```javascript
let boundFunc = func.bind(context);

/*-------------example--------------*/

let user = {
  firstName: "John",
  say(phrase) {
    alert(`${phrase}, ${this.firstName}!`);
  }
};

let say = user.say.bind(user); // (*)

say("Hello"); // Hello, John

setTimeout(() => say("Hello"), 1000); // Hello, John!

user = {
  say() { alert("Another user in setTimeout!"); }
};
```
- `(*)`에서 `user.say()`를 `user`에 bind함  
	=> `say`가 bound function가 됨  
	=> 어디서 호출되든, `user`의 정보가 바뀌든 context가 bind될 당시의 `user`로 고정됨
- `user.say()`가 바뀌더라도 bind될 당시의 `user.say()`가 실행됨  
	`setTimeout`안의 arrow function은 `say()`에 argument를 넣기 위한 것임(wrapper를 이용한 것이 아님)

> Binding all methods  
> 한 객체의 모든 method를 bind해야 할 때 아래와 같이 구현 가능:  
> ```javascript
> for (let key in user) {
>   if (typeof user[key] == 'function') {
>     user[key] = user[key].bind(user);
>   }
> }
> ```
> 
> lodash의 `_.bindAll(obj, methodNames)`와 같이 함수를 제공하기도 함

## Partial functions
The full syntax of bind:  
```javascript
let bound = func.bind(context, [arg1], [arg2], ...);

/*-------------example--------------*/

function mul(a, b) {
  return a * b;
}

let double = mul.bind(null, 2);

alert( double(3) ); // = mul(2, 3) = 6
alert( double(4) ); // = mul(2, 4) = 8
alert( double(5) ); // = mul(2, 5) = 10
```
- `mul`의 parameter `a`를 `2`로 bind함  
	`context`가 필수기 때문에 사용하지 않는 경우 `null`을 넣어줌
- 이렇게 기존의 함수의 parameter를 조절해서 새로운 함수를 만드는 것을 *partial function application*이라고 함
	- 함수의 이름을 다시 지을 수 있음  
	- bind한 parameter는 호출할 때마다 넣어주지 않아도 됨  
		=> generic한 함수를 고쳐서 편의성을 높일 수 있음  
		
		e.g. `send(from, to, text)`를 이용해서 `sendTo(to, text)`를 만들 수 있음

## Going partial without context
`bind`를 사용하면 context도 반드시 넣어야 하므로 `partial` 함수를 따로 구현해놓는 것도 좋음:  
```javascript
function partial(func, ...argsBound) {
  return function(...args) {
    return func.call(this, ...argsBound, ...args);
  }
}

/*-------------example--------------*/

let user = {
  firstName: "John",
  say(time, phrase) {
    alert(`[${time}] ${this.firstName}: ${phrase}!`);
  }
};

user.sayNow = partial(user.say, new Date().getHours() + ':' + new Date().getMinutes());

user.sayNow("Hello"); // [10:00] John: Hello!
```
- `bind`를 사용하지 않고 wrapper로 구현
- lodash에는 `_.partial` 함수가 따로 존재

## Summary

| code                                                   | description                                                                   |
| :----------------------------------------------------- | :---------------------------------------------------------------------------- |
| `let bound = func.bind(context, [arg1], [arg2], ...);` | `func`의 context와 arguments를 `context`, `arg1...`로 bind한 새로운 함수 반환 |

- context와 arguments 일부를 bind한 함수를 bound function이라고 부름
	- 주로 method를 callback으로 전달하기 위해 사용
- context 말고 arguments만 bind한 함수를 partial function이라고 부름
	- 따로 wrapper로 구현해야 함
- 둘 다 lodash에 `_.bindAll(obj, methodNames)`, `_.partial`로 존재함

## Tasks
```javascript
function f() {
  alert(this.name);
}

f = f.bind( {name: "John"} ).bind( {name: "Ann" } );

f(); // John
```
- 한 번 bound된 함수는 다시 bound될 수 없음!!  
	처음 bound된 context로 고정됨

```javascript
function sayHi() {
  alert( this.name );
}
sayHi.test = 5;

let bound = sayHi.bind({
  name: "John"
});

alert( bound.test ); // undefined
```
- `bind`는 아예 새로운 함수를 반환하기 때문에 function property가 공유되지 않음
- strict mode가 아닐 때 `this`가 정의되지 않으면 `globalThis`를 가리키는 듯

# Arrow functions revisited
arrow function은 단지 shorthand가 아니라 유용한 기능을 가지고 있음  
JS는 `arr.forEach(func)`와 같이 함수가 인자로 들어가서, 정의된 환경과 다른 곳에서 실행되는 경우가 많음  
이런 경우에 `func`의 context를 그대로 두고 싶은 경우가 많은데, arrow function이 이것을 도와줌

## Arrow functions have no "this"
arrow function은 `this`가 없고, 외부에서 가져옴  
e.g. object method 내에서 반복을 위해 arrow function을 사용 가능:  
```javascript
let group = {
  title: "Our Group",
  students: ["John", "Pete", "Alice"],

  showList() {
    this.students.forEach(
      student => alert(this.title + ': ' + student) // (*)
    );
  }
};

group.showList();
```
- `(*)`에서 `forEach`안에서 사용된 arrow function은 `forEach`의 outer method인 `showList()`의 `this`를 사용하기 때문에 `group.title`이 들어감
- `(*)`을 `alert(this.title + ': ' + student);`로 바꾸면 에러남  
	`forEach`는 `this`가 `undefined`이기 때문  
	cf. arrow function은 `this` 자체가 없기 때문에 outer `this`를 사용해서 에러가 나지 않음

> ※ Arrow functions은 `new`와 같이 실행할 수 없음  
> ∵ `this`가 없기 때문  
> 따라서 arrow function은 생성자로 사용될 수도 없음(`super`도 가질 수 없음)

> ※ Arrow functions VS bind  
> - `.bind(this)`는 bound function을 생성함  
> - `=>`는 binding을 생성하지 않고 변수를 찾는 것과 같이 outer lexical environment의 `this`를 사용하는 것임

## Arrows have no "arguments"
arrow function은 `arguments`도 존재하지 않음  
=> decorator와 같이 call forwarding을 해야 할 때 `this`와 `arguments`를 편하게 사용할 수 있음  
e.g. `defer(f, ms)`는 `func`를 `ms` ms만큼 지연시킴:  
```javascript
function defer(f, ms) {
  return function() {
    setTimeout(() => f.apply(this, arguments), ms);
  };
}

function sayHi(who) {
  alert('Hello, ' + who);
}

let sayHiDeferred = defer(sayHi, 2000);
sayHiDeferred("John"); // Hello, John after 2 seconds
```
- arrow function의 `this`와 `arguments`는 wrapper의 것을 가져옴

`defer`는 아래와 같이 구현할 수도 있음

```javascript
function defer(f, ms) {
  return function(...args) {
    let ctx = this;
    setTimeout(function() {
      return f.apply(ctx, args);
    }, ms);
  };
}
```
- context를 일일히 저장해야 하고, arguments도 rest parameter를 써서 가져와야 함