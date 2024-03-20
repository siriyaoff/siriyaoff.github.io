---
layout: single
title: "JSnote12: Generators, advanced iteration"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - JavaScript
  - Generator
  - iterable
  - Generator composition
  - yield
  - Async iteration
  - Async/await
  - Async generator
  - Pagination
---

Generators, advanced iteration
# Generators
regular function은 하나의 결과만 리턴함  
generator는 여러 개의 값을 필요에 따라 차례대로 리턴(yield)할 수 있음  
iterable과 함께 사용하면 좋음

## Generator functions
`function*`으로 generator function을 생성할 수 있음:  
```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}

let generator = generateSequence();
alert(generator); // [object Generator]
```
- generator function을 실행하면 내부 코드가 실행되지 않고, generator object를 반환함

generator의 주요 메소드는 `next()`임  
`next()`가 호출되면 가장 가까운 `yield <value>`까지 코드를 실행함  
`yield <value>`는 `value`를 리턴하고, 다시 함수 실행을 멈춤  
(`<value>`가 생략되면 `undefined`가 리턴됨)

`next()`의 결과는 항상 두 개의 property를 가지는 객체임:
- `value` : yielded value
- `done` : 함수 코드가 끝났으면 `true`, 아니면 `false`

#### Example
```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}

let generator = generateSequence();

let one = generator.next();
alert(JSON.stringify(one)); // {value: 1, done: false}
let two = generator.next();
alert(JSON.stringify(two)); // {value: 2, done: false}
let three = generator.next();
alert(JSON.stringify(three)); // {value: 3, done: true}
```
- generator가 종료된 후에는 `generator.next()`를 실행하면 `{done: true}` 객체가 계속 반환됨

> #### `function* f(...)` or `function *f(...)`?
> 둘 다 사용 가능하지만, 전자가 많이 사용됨

## Generators are iterable
generator가 `next()` method를 가지기 때문에 iterable임!  
따라서 `for...of`로 반복할 수 있음:  
```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}

let generator = generateSequence();

for(let value of generator) {
  alert(value); // 1, then 2
}
```
- `1`, `2`만 출력되고 `3`은 출력되지 않음에 주의!!!  
	`for...of`는 `done: true`인 마지막 객체를 무시함  
	따라서 `for...of`로 결과를 모두 출력하고 싶으면 마지막 값도 `yield`로 리턴해야 함!:  
	```javascript
	function* generateSequence() {
	  yield 1;
	  yield 2;
	  yield 3;
	}

	let generator = generateSequence();

	for(let value of generator) {
	  alert(value); // 1, then 2, then 3
	}
	```

generator가 iterable이기 때문에 spread syntax `...`을 적용할 수 있음:  
```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  yield 3;
}

let sequence = [0, ...generateSequence()];

alert(sequence); // 0, 1, 2, 3
```

## Using generators for iterables
`range` object는 `from`부터 `to`까지의 값을 반환하는 iterable임:  
```javascript
let range = {
  from: 1,
  to: 5,

  [Symbol.iterator]() {
    return {
      current: this.from,
      last: this.to,

      next() {
        if (this.current <= this.last) {
          return { done: false, value: this.current++ };
        } else {
          return { done: true };
        }
      }
    };
  }
};

alert([...range]); // 1,2,3,4,5
```
- `[Symbol.iterator]()` method는 `range`가 iteration을 시작할 때 한 번만 호출됨  
	e.g. `for...of range`가 시작될 때
	
	iterator 객체를 리턴함
	- 이 객체를 통해서 `for...of`와 같은 반복문이 작동함
	- `next()`를 가지고 있어야 함
- `next()`는 매 반복마다 호출됨  
	e.g. `for...of`의 매 iteration 마다  
	
	`{done: .., value: ..}`인 객체를 리턴함           
 
`Symbol.iterator()`를 generator function으로 바꿀 수 있음:  
```javascript
let range = {
  from: 1,
  to: 5,

  *[Symbol.iterator]() {
    for(let value = this.from; value <= this.to; value++) {
      yield value;
    }
  }
};

alert( [...range] ); // 1,2,3,4,5
```
- `*[Symbol.iterator]()`는 `[Symbol.iterator}: function*()`의 shorthand임
- generator function으로 변환된 `Symbol.iterator()`는 정확히 iterator의 역할을 수행함
	- `.next()` method가 존재함
	- `{done: .., value: ..}` 객체를 리턴함
- generator 자체가 JS에서 iterator를 쉽게 다루기 위해 만들어진 것임  
	=> iterable code가 더 간결해짐

> #### Generators may generate values forever
> 위 예시에서는 유한한 수열을 생성했지만, 무한하게 출력하는 것도 가능함  
> => generator를 사용하는 반복문에서 종료시켜야 함

## Generator composition
generator composition을 사용해서 generator 안에 generator를 embed할 수 있음  
아래와 같이 수열을 생성하는 함수가 있다 하자:  
```javascript
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) yield i;
}
```

이 함수를 사용해서 아래와 같은 수열을 만들어야 함:  
- 첫 번째로 `'0'..'9'`(`48..57`) 출력
- 그 뒤에 `A..Z`(`65..90`) 출력
- 그 뒤에 `a..z`(`97..122`) 출력

regular function에서 위와 같은 작업을 수행하려면 변수 하나에 각각의 함수 호출 결과를 저장하고 그것을 출력해야 함  
generator에서는 `yield*`를 이용해서 generator의 결과를 합칠 수 있음:  
```javascript
function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) yield i;
}

function* generatePasswordCodes() {
  yield* generateSequence(48, 57);
  yield* generateSequence(65, 90);
  yield* generateSequence(97, 122);
}

let str = '';

for(let code of generatePasswordCodes()) {
  str += String.fromCharCode(code);
}

alert(str); // 0..9A..Za..z
```
- `yield* generateSequence(48, 57);`는 `for (let i = 48; i <= 57; i++) yield i;`와 같음
- `yield*`는 다른 generator에게 실행을 위임함  
	즉, `yield* gen`이 `gen`의 반복을 수행하고 그 결과를 바깥의 generator의 `yield`로 전달함
- generator composition은 generator를 다른 generator 안으로 넣기 위한 방법임  
	이를 이용하면 generator의 값을 옮기기 위해 따로 변수를 선언하지 않아도 되기 때문에 메모리가 절약됨

## "yield" is a two-way street
`yield`는 generator의 결과를 밖으로 리턴할 뿐만 아니라, 밖에서 generator 안으로 값을 가져올 수도 있음  
`generator.next(arg)`를 사용해서 `arg`를 `yield`의 결과로 만들 수 있음:  
```javascript
function* gen() {
  // Pass a question to the outer code and wait for an answer
  let result = yield "2 + 2 = ?"; // (*)

  alert(result);
}

let generator = gen();

let question = generator.next().value; // <-- yield returns the value

generator.next(4); // --> pass the result into the generator
```
- `(*)`의 `"2 + 2 = ?"`가 바깥의 `question`으로 들어가고, `generator.next(4)`에 의해 `gen()`의 `result`에 `4`가 대입됨  
	
 | ![js-generator1](https://github.com/siriyaoff/MDN-note/blob/master/images/js-generator1.PNG?raw=true) |
 | :---------------------------------------------------------------------------------------------------: |
 |                                         javascript.info 참고                                          |
	
	1. `generator.next()`에 의해 실행이 시작되고 `yield "2+2=?"`의 결과가 리턴됨  
		이 시점에서 함수 흐름은 `(*)`에 멈춰있음
	2. `yield`의 결과가 `gen`을 호출한 코드의 `question`에 저장됨
	3. `generator.next(4)`가 실행되면서 generator가 다시 실행되고 `4`가 `result`에 들어감
- 첫 번째 `generator.next()`는 항상 argument 없이 호출되어야 함!  
	(있으면 무시됨)

outer code에서 즉시 `next(4)`를 호출할 필요는 없음!  
시간을 두고 실행해도 generator가 입력을 기다리다가 받음:  
```javascript
setTimeout(() => generator.next(4), 1000);
```

보통의 함수들과 다르게, generator와 calling code는 `yield/next`로 값을 서로 전달할 수 있음:  
```javascript
function* gen() {
  let ask1 = yield "2 + 2 = ?";

  alert(ask1); // 4

  let ask2 = yield "3 * 3 = ?"

  alert(ask2); // 9
}

let generator = gen();

alert( generator.next().value ); // "2 + 2 = ?"
alert( generator.next(4).value ); // "3 * 3 = ?"
alert( generator.next(9).done ); // true
```
- 첫 번째를 제외한 나머지 `next(value)`는 `value`를 현재 `yield`의 결과값이 되도록 전달하고, 다음 `yield`의 값을 가져옴

| ![js-generator2](https://github.com/siriyaoff/MDN-note/blob/master/images/js-generator2.PNG?raw=true) |
| :---------------------------------------------------------------------------------------------------: |
|                                         javascript.info 참고                                          |

## generator.throw
outer code도 `yield`의 결과가 되도록 generator 안으로 값을 전달할 수 있음  
에러도 하나의 결과값이기 때문에 에러를 전달할 수도 있음  
에러를 `yield`로 전달하기 위해선 `generator.throw(err)`를 사용하면 됨  
=> 에러가 `yield`가 있는 줄에서 발생하게 됨:  
```javascrpt
function* gen() {
  try {
    let result = yield "2 + 2 = ?"; // (1)
    alert("The execution does not reach here, because the exception is thrown above");
  } catch(e) {
    alert(e);
  }
}

let generator = gen();
let question = generator.next().value;

generator.throw(new Error("The answer is not found in my database")); // (2)
```
- `(2)`에서 generator 안으로 던진 에러는 `(1)`에서 exception이 됨  
	generator 안의 `try...catch`로 에러를 처리함
- generator 안에서 잡지 않으면 에러는 밖으로 떨어져나옴  
	따라서 아래와 같이 `(2)`에서 에러를 잡을 수도 있음:  
	```javascript
	function* generate() {
	  let result = yield "2 + 2 = ?"; // Error in this line
	}

	let generator = generate();
	let question = generator.next().value;

	try {
	  generator.throw(new Error("The answer is not found in my database"));
	} catch(e) {
	  alert(e); // shows the error
	}
	```
- 에러를 아예 잡지 않으면 다른 에러와 마찬가지로 스크립트를 멈춤

## generator.return
`generator.return(value)`는 강제로 generator 실행을 끝내고 `value`를 리턴함:  
```javascript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

const g = gen();

g.next();        // { value: 1, done: false }
g.return('foo'); // { value: "foo", done: true }
g.next();        // { value: undefined, done: true }
```
- 이미 `return`을 사용한 상태에서 한 번 더 사용하면 이전의 값을 다시 리턴함
- `return`을 사용한 후에는 generator가 끝난 상태이기 때문에 `next`를 호출하면 위와 같이 `undefind`가 리턴됨

## Summary

| code                        | description                                                                           |
| :-------------------------- | :------------------------------------------------------------------------------------ |
| `function* f(...) { ... }`  | generator function `f`를 선언                                                         |
| `yield value`               | generator 내부에서 외부와 값을 전달할 때 사용됨                                       |
| `f.next(value)`             | generator `f`의 현재 `yield`의 값으로 `value`를 전달하고 다음 `yield`의 결과값을 받음 |
| `yield* <generator object>` | `<generator object>`의 결과값을 outer generator object의 `yield`의 결과값으로 사용    |
| `generator.throw(err)`      | `generator`에 `err`를 던짐                                                            |
| `generator.return(value)`   | `generator`를 `value`를 가진 객체를 리턴함과 함께 끝냄                                |

- generator function을 실행하면 내부 코드가 실행되는게 아니라 generator object가 반환됨
- generator는 iterable임  
	∵ `next()` method를 가짐
	=> `for...of`에 generator를 사용해서 반복 가능, spread syntax 적용 가능
- 아예 `range`의 `Symbol.iterator`를 generator function으로 바꿀 수도 있음
- generator 내부에서 `yield`, `return`으로 함수 실행을 조절할 수 있음
- generator composition
	- 한 generator object의 결과를 다른 generator의 `yield`의 결과값으로 사용하는 것
	- `yield*`를 이용
- `generator.return()`을 호출하면 가장 최근의 값을 리턴함

## Tasks
### Pseudo-random generator
"seeded pseudo-random generator"는 시드값과 점화식을 이용해서 랜덤 수열을 생성하는 것을 말함

```javascript
function* pseudoRandom(seed){
  while(true){
    seed=seed*16807%2147483647;
    yield seed;
  }
}

let generator = pseudoRandom(1);

alert(generator.next().value); // 16807
alert(generator.next().value); // 282475249
alert(generator.next().value); // 1622650073
```

아래와 같이 함수로도 구현 가능함:  
```javascript
function pseudoRandom(seed) {
  let value = seed;

  return function() {
    value = value * 16807 % 2147483647;
    return value;
  }
}

let generator = pseudoRandom(1);

alert(generator()); // 16807
alert(generator()); // 282475249
alert(generator()); // 1622650073
```
- 하지만 이렇게 구현하면 `for...of`로 반복하거나 generator composition을 사용하지 못함

# Async iteration and generators
## Recall iterables
아래와 같이 `range` iterable을 구현 가능함:  
```javascript
let range = {
  from: 1,
  to: 5,

  [Symbol.iterator]() { // called once, in the beginning of for..of
    return {
      current: this.from,
      last: this.to,

      next() { // called every iteration, to get the next value
        if (this.current <= this.last) {
          return { done: false, value: this.current++ };
        } else {
          return { done: true };
        }
      }
    };
  }
};

for(let value of range) {
  alert(value); // 1 then 2, then 3, then 4, then 5
}
```

## Async iterables
시간차를 두는 등의 이유로 값들이 비동기적으로 들어올 때 비동기적 반복이 필요함  
가장 흔한 케이스는 네트워크 통신임

iterable을 비동기적으로 만들기 위해서는 아래와 같은 과정을 거쳐야 함:
1. `Symbol.asyncIterator`를 사용해야 함(`Symbol.iterator` 대신)
2. `next()`는 promise를 리턴해야 함(다음 값으로 fulfilled)  
	=> `async next()`로 비동기적 반복을 처리함
3. 이러한 iterable은 `for await (let item of iterable)`를 이용해서 반복을 수행해야 함

#### Example
```javascript
let range = {
  from: 1,
  to: 5,

  [Symbol.asyncIterator]() {
    return {
      current: this.from,
      last: this.to,

      async next() {
        await new Promise(resolve => setTimeout(resolve, 1000));

        if (this.current <= this.last) {
          return { done: false, value: this.current++ };
        } else {
          return { done: true };
        }
      }
    };
  }
};

(async () => {
  for await (let value of range) {
    alert(value); // 1,2,3,4,5
  }
})()
```

iterator, async iterator의 차이:

|                                   | Iterators         | Async iterators        |
| :-------------------------------- | :---------------- | :--------------------- |
| Object method to provide iterator | `Symbol.iterator` | `Symbol.asyncIterator` |
| `next()` return value is          | any value         | `Promise`              |
| to loop, use                      | `for...of`        | `for await ...of`      |

> #### The spread syntax `...` doesn't work asynchronously
> asynchronous iterator에 대해서는 spread syntax를 사용할 수 없음!

## Recall generators
generator는 `function*`로 선언된 generator function을 통해 생성됨  
`yield`, `return`으로 값을 리턴함

iterable을 generator로 구현하기 위해선 아래와 같이 `Symbol.iterator`을 generator function으로 만들어야 함:  
```javascript
let range = {
  from: 1,
  to: 5,

  *[Symbol.iterator]() { // a shorthand for [Symbol.iterator]: function*()
    for(let value = this.from; value <= this.to; value++) {
      yield value;
    }
  }
};

for(let value of range) {
  alert(value); // 1, then 2, then 3, then 4, then 5
}
```

## Async generators (finally)
asynchronous generator를 이용하면 비동기적으로 값을 생성하는 객체를 만들 수 있음  
`function*`에 `async`를 적용시켜 generator function이 `await`를 사용할 수 있게 만들어주면 됨  
반복할 때는 `for await (...)`을 사용함:  
```javascript
async function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) {
    await new Promise(resolve => setTimeout(resolve, 1000));
    yield i;
  }
}

(async () => {
  let generator = generateSequence(1, 5);
  for await (let value of generator) {
    alert(value); // 1, then 2, then 3, then 4, then 5 (with delay between)
  }
})();
```

> #### Under-the-hood difference
> regular generator의 경우 `result = generator.next()`를 사용해서 값을 받을 수 있음  
> 하지만 async generator의 경우 `generator.next()`가 비동기적이고, promise를 리턴함  
> 따라서 아래와 같이 `await`를 추가해야 함:  
> ```javascript
> result = await generator.next(); // result = {value: ..., done: true/false}
> ```

### Async iterable range
async iterable range도 `Symbol.asyncIterator`에 `await`를 추가해서 구현할 수 있음:  
```javascript
let range = {
  from: 1,
  to: 5,

  async *[Symbol.asyncIterator]() {
    for(let value = this.from; value <= this.to; value++) {
      await new Promise(resolve => setTimeout(resolve, 1000));
      yield value;
    }
  }
};

(async () => {
  for await (let value of range) {
    alert(value); // 1, then 2, then 3, then 4, then 5
  }
})();
```

> #### Please note:
> 이론적으로는 `Symbol.iterator`, `Symbol.asyncIterator` 둘 다 object에 추가 가능하기 때문에 한 객체가 동기적(`for...of`), 비동기적(`for await...of`) iterable이 될 수 있음  
> 하지만 실제로는 그렇게 사용하지 않음

## Real-life example: paginated data
한 페이지에 100개의 유저 정보만 출력하는 경우 async generator가 필요함  
이 패턴은 유저 정보 뿐만이 아닌 아주 흔한 패턴임  
예를 들어 Github에서도 커밋을 페이지 방식으로 조회하게 만듦:
- `http://api.github.com/repos/<repo>/commits`에 `fetch`를 요청
- 30개의 JSON과 `Link`헤더에 다음 페이지의 링크를 제공함
- 그 링크를 사용해서 더 많은 커밋 정보를 조회함

이 예제에서는 더 간편한 방법을 사용해서 커밋 정보를 얻을 것임  
`fetchCommits(repo)`는 필요할 때마다 요청해서 커밋을 가져오는 함수라 하자  
페이지화된 방법을 사용하기 위해 우리는 `for await...of`를 사용해보자  
그렇다면 아래와 같이 `fetchCommits`를 사용할 수 있음:  
```javascript
for await (let commit of fetchCommits("username/repository")) {
  // process commit
}
```

함수의 구현부:  
```javascript
async function* fetchCommits(repo) {
  let url = `https://api.github.com/repos/${repo}/commits`;

  while (url) {
    const response = await fetch(url, { // (1)
      headers: {'User-Agent': 'Our script'},
    });

    const body = await response.json(); // (2) response is JSON (array of commits)

    // (3) the URL of the next page is in the headers, extract it
    let nextPage = response.headers.get('Link').match(/<(.*?)>; rel="next"/);
    nextPage = nextPage?.[1];

    url = nextPage;

    for(let commit of body) { // (4) yield commits one by one, until the page ends
      yield commit;
    }
  }
}
```
- `(1)`에서 Github에서는 인증 등을 위해 다른 헤더도 필요하면 추가할 수 있도록 `User-Agent`를 요구함
- `(2)`에서 JSON으로 받은 커밋을 복호화함
- `(3)`에서 `Link` 헤더에서 다음 페이지의 링크를 추출함
- `(4)`에서 커밋을 하나씩 yield함

사용 예시:  
```javascript
(async () => {
  let count = 0;
  for await (const commit of fetchCommits('javascript-tutorial/en.javascript.info')) {
    console.log(commit.author.login);
    if (++count == 100) { // let's stop at 100 commits
      break;
    }
  }
})();
```

## Summary
- 비동기적으로 데이터를 입력받는 것은 async generator를 사용하면 됨
- async iteration을 사용해서 async generator에 대해 iteration을 수행할 수 있음

regular iterator, async iterator 구현 차이:

|                            | Iterable                | Async Iterable                                     |
| :------------------------- | :---------------------- | :------------------------------------------------- |
| Method to provide iterator | `Symbol.iterator`       | `Symbol.asyncIterator`                             |
| `next()` return value is   | `{value:..., done:...}` | `Promise` that resolves to `{value:..., done:...}` |

regular generator, async generator 구현 차이:

|                          | Generators              | Async Generators                                   |
| :----------------------- | :---------------------- | :------------------------------------------------- |
| Declaration              | `function*`             | `async function*`                                  |
| `next()` return value is | `{value:..., done:...}` | `Promise` that resolves to `{value:..., done:...}` |

- browser 환경에서는 Stream API를 사용하면 이런 paginated data를 쉽게 다룰 수 있음