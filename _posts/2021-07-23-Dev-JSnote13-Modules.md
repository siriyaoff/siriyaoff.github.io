---
layout: single
title: "JSnote13: Modules"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - JavaScript
  - Module
  - import.meta
  - export
  - import
  - Default export
  - Re-export
  - import()
  - Async/await
---

Modules
# Modules
# Modules, introduction
프로젝트의 크기가 커지면 여러 개의 모듈로 분리해서 관리하는게 효율적임  
모듈은 보통 하나의 클래스나 함수들로 구성된 라이브러리 하나임  
아래와 같은 모듈 시스템들이 있었음:
- AMD : require.js를 사용해서 개발됨
- CommonJS : Node.js 서버를 위해 개발됨
- UMD : 모듈 시스템들을 함께 사용하기 위해 개발됨

브라우저와 Node.js 등이 모듈 시스템을 지원하기 때문에 요즘은 사용하지 않는 추세임

## What is module?
module은 하나의 파일임  
즉 하나의 script가 하나의 module임  
module은 `export`와 `import`를 사용해서 다른 module의 함수를 호출하는 등의 기능 공유가 가능함
- `export` : 현재 모듈의 외부에서 접근 가능하게 만듦
- `import` : 다른 모듈의 기능을 불러옴

**Example**

```javascript
// 📁 sayHi.js
export function sayHi(user) {
  alert(`Hello, ${user}!`);
}

// 📁 main.js
import {sayHi} from './sayHi.js';

alert(sayHi); // function...
sayHi('John'); // Hello, John!
```
- `import`는 `./sayHi.js`에 있는 모듈을 현재 파일로 불러오고, `sayHi.js`에서 export된 `sayHi`를 `{sayHi}`에 대입함

browser에서는 `<script type="module">`과 같이 attribute를 사용해서 다뤄야 함:  
```javascript
<!doctype html>
<script type="module">
  import {sayHi} from './say.js';

  document.body.innerHTML = sayHi('John');
</script>
```
- 브라우저가 자동으로 fetch, import해서 script를 실행시킴

> **Modules work only via HTTP(s), not locally**  
> `file://`를 이용하면 `import/export`가 제대로 실행되지 않음  
> local web-server를 사용하거나 VS Code의 Live Server Extension을 사용해야 함

## Core module features
모듈과 regular script의 차이는 무엇일까?  
browser, server-side JS 모두에서 허용되는 기능들은 아래와 같음

### Always "use strict"
module은 항상 strict mode에서 동작함:  
```javascript
<script type="module">
  a = 5; // error
</script>
```
- 선언되지 않은 변수에 값을 대입하는 것은 에러를 발생시킴

### Module-level scope
module은 각각 top-level scope를 가짐  
즉 한 모듈에서의 top-level code는 다른 module에서 보이지 않음

```javascript
// 📁 user.js
let user = "John";

// 📁 hello.js
alert(user); // no such variable (each module has independent variables)

// 📁 index.html
<!doctype html>
<script type="module" src="user.js"></script>
<script type="module" src="hello.js"></script>
```
- 에러가 발생함  
	`user`는 `user.js`에서 선언된 top-level variable이기 때문에 `hello.js`에서 사용할 수 없음

외부에서 접근 가능하게 만드려면 `export`로 내보내야 하고, `import`로 필요한 내용을 가져와야 함
- `user.js`에서는 `user`를 `export`해야 함
- `hello.js`에서는 `user.js` module을 `import`해야 함

모듈을 사용하면 global variable 대신 `import/export`를 사용해야 함  
위 코드를 올바르게 구현하면 아래와 같음:  
```javascript
// 📁 user.js
export let user = "John";

// 📁 hello.js
import {user} from './user.js';

document.body.innerHTML = user; // John

// 📁 index.html
<!doctype html>
<script type="module" src="hello.js"></script>
```
- `<script type="module">`마다 독립적인 top-level scope가 존재함:  
	```javascript
	<script type="module">
	  let user = "John";
	</script>

	<script type="module">
	  alert(user); // Error: user is not defined
	</script>
	```

> **Please note:**  
> `window.user="John"`과 같이 명시적으로 `window` property에 값을 대입하면 window-level global하게 됨  
> => 모든 script에서 사용 가능  
> 되도록 사용을 자제해야 함

### A module code is evaluated only the first time when imported
같은 모듈이 다른 모듈들에서 여러 번 import되면 처음 한 번만 실행됨  
만약 모듈안에 `alert`와 같은 내용이 있을 경우, 여러 번 import되어도 처음 한 번만 출력됨:  
```javascript
// 📁 alert.js
alert("Module is evaluated!");

// 📁 1.js
import `./alert.js`; // Module is evaluated!

// 📁 2.js
import `./alert.js`; // (shows nothing)
```
- top-level module code는 모듈 안에서 사용될 데이터의 초기화를 위해 사용되어야 함  
	이게 여러 번 호출될 수 있도록 만드려면 함수로 만들어 export해야 함

모듈이 객체를 export한다 해보자:  
```javascript
// 📁 admin.js
export let admin = {
  name: "John"
};

// 📁 1.js
import {admin} from './admin.js';
admin.name = "Pete";

// 📁 2.js
import {admin} from './admin.js';
alert(admin.name); // Pete
```
- 여러 파일에 import되어도 한 번만 evaluate되기 때문에, `admin`은 처음 생성된 후 모든 importer들에게 전달됨  
	=> `1.js`, `2.js`에서 같은 `admin` 객체를 사용함!!

이러한 특징으로 인해 모듈을 configure할 수 있음  
i.e. 처음 설정한 대로 다른 곳에서도 사용할 수 있도록 모듈을 설정할 수 있음  
e.g. 인증과 같은 경우 한 번 인증하면 다른 곳에서도 다시 할 필요 없음

아래와 같은 classical pattern이 있음:
1. 모듈이 configuration을 위한 수단을 export함  
	e.g. configuration object
2. 모듈의 첫 실행에서 그것을 초기화함  
	e.g. top-level application script에서 그것의 property를 수정
3. 이후의 import들이 설정된 모듈을 사용

예를 들어 `admin.js`가 인증과 같은 어떤 기능을 제공하지만 외부로부터 `config` 객체에 credential을 입력받는다고 가정하자:  
```javascript
// 📁 admin.js
export let config = { };

export function sayHi() {
  alert(`Ready to serve, ${config.user}!`);
}

// 📁 init.js
import {config} from './admin.js';
config.user = "Pete";

// 📁 another.js
import {sayHi} from './admin.js';

sayHi(); // Ready to serve, Pete!
```
- `admin.js`는 `config` 객체를 export함(처음엔 empty)
- `init.js`에서 `config`를 import해서 `config.user`를 설정함
- `admin.js`가 configure되고, 다른 importer(`another.js`)들에서 사용됨

### import.meta
`import.meta` object는 현재 모듈에 관한 정보를 가지고 있음  
저장하는 내용은 환경에 따라 달라짐  
browser에서는 script의 URL, inline script일 경우 현재 페이지의 URL 등을 가지고 있음:  
```javascript
<script type="module">
  alert(import.meta.url); // script URL
</script>
```

### In a module, "this" is undefined
module 안에서 top-level `this`는 `undefined`임  
cf. non-module script에서 top-level `this`는 global object임:  
```javascript
<script>
  alert(this); // window
</script>

<script type="module">
  alert(this); // undefined
</script>
```

## Browser-specific features
> JS browser part부터 읽어야 할 듯

## Build tools
browser 환경에서 모듈만 사용하는 경우는 거의 없음  
보통 Webpack과 같은 툴을 이용해서 bundling한 다음 서버에 올림  
bundler를 사용하면 모듈들의 의존성에 따라서 실행할 수 있고 CSS/HTML 모듈들을 사용할 수 있음

Build tool은 아래와 같은 동작을 함:
1. `<script type="module">`에 들어갈 main module을 선택함
2. 의존성을 분석함
3. 모든 모듈을 하나의 파일로 만듦  
	이 과정에서 `import`를 bundler function으로 대체함  
	=> HTML/CSS module과 같은 모듈들도 지원됨
4. 다른 변환과 최적화들이 수행됨  
	- unreachable code가 지워짐
	- unused exports가 지워짐("tree-shaking"이라고 함)
	- Development-specific statements(e.g. `console`, `debugger`)가 지워짐
	- 최신의 JS 문법이 transpile됨(babel 등을 이용)
	- 결과로 나오는 파일을 minify함(크기를 줄임)

따라서 bundle tool을 사용하면 `import/export`를 포함하지 않고, `type="module"`을 필요로 하지 않음  
=> 아래와 같이 regular script처럼 추가할 수 있음:  
```javascript
<script src="bundle.js"></script>
```

## Summary

| code                                   | description                              |
| :------------------------------------- | :--------------------------------------- |
| `import {feature} from './module.js';` | `'./module.js'`에서 `feature`을 import함 |
| `export let feature = { };`            | `feature`을 export함                     |

- 모듈을 참조할 때는 `<script type="module">`와 같이 attribute를 넣어야 함
- 모듈의 특징
	- 항상 strict mode를 사용
	- 모듈마다 독립적인 top-level scope를 가짐
	- 여러 번 import되면 처음 한 번만 실행됨
	- `import.meta`에 현재 모듈에 관한 정보가 저장됨
	- 모듈 안에서 top-level `this`는 `undefined`임
- Bundle과 같은 build tool을 이용하면 여러 개의 모듈들을 편하게 관리할 수 있음  
	`import/export`가 bundler function으로 대체되고 모듈을 문서에 참조할 때 `type="module"`을 쓰지 않아도 됨

# Export and Import
export와 import는 많은 syntax를 가짐

## Export before declarations
`export`를 declaration 앞에 넣어서 export할 수 있음:  
```javascript
export let months = ['Jan', 'Feb', 'Mar','Apr', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];

export const MODULES_BECAME_STANDARD_YEAR = 2015;

export class User {
  constructor(name) {
    this.name = name;
  }
}
```

> **No semicolons after export class/function**  
> `export`를 붙였다고 클래스나 함수가 expression이 되진 않으므로 세미콜론을 붙이지 않아도 됨

## Export apart from declarations
먼저 선언하고 export할 수 있음:  
```javascript
// 📁 say.js
function sayHi(user) {
  alert(`Hello, ${user}!`);
}

function sayBye(user) {
  alert(`Bye, ${user}!`);
}

export {sayHi, sayBye};
```
- `export`를 declaration 위에 적어도 정상적으로 작동함

## Import *
보통 import할 것들을 curly braces 안에 넣음:  
```javascript
// 📁 main.js
import {sayHi, sayBye} from './say.js';

sayHi('John'); // Hello, John!
sayBye('John'); // Bye, John!
```

하지만 import할게 많으면 `import * as <obj>`와 같이 모든 내용을 하나의 객체로 import할 수 있음:  
```javascript
// 📁 main.js
import * as say from './say.js';

say.sayHi('John');
say.sayBye('John');
```

`import *` 대신 명시적으로 import할 것들을 지정하는 이유는 아래와 같음
1. 최신의 build tool들은 모듈들을 bundling해서 최적화함  
	즉 사용하지 않는 기능은 제외하고 bundling함
2. import하는 것의 이름을 간결하게 쓸 수 있음
3. import하는 것을 명시하는게 코드 전체의 구조를 쉽게 읽는데 좋음

## Import "as"
`as`를 사용해서 다른 이름으로 import할 수 있음:  
```javascript
// 📁 main.js
import {sayHi as hi, sayBye as bye} from './say.js';

hi('John'); // Hello, John!
bye('John'); // Bye, John!
```

## Export "as"
`export`에도 `as`를 사용할 수 있음:  
```javascript
// 📁 say.js
...
export {sayHi as hi, sayBye as bye};

// 📁 main.js
import * as say from './say.js';

say.hi('John'); // Hello, John!
say.bye('John'); // Bye, John!
```

## Export default
모듈에는 크게 두 가지의 종류가 있음:
1. library나 함수들을 포함하는 모듈
2. 하나의 개체만 선언되어 있는 모듈

대부분 두 번째 접근으로 모듈을 만드는 것을 선호함  
∵ 하나의 모듈 안에 그 개체에 관련된 모든 것이 들어있기 떄문

이렇게 모듈을 만들면 많은 모듈들이 필요하지만, 문제가 되지 않음  
∵ 파일 naming이 잘되기 때문에 code navigation으로 찾기가 더 쉬움

`export default`를 사용해서 두 번째 접근으로 만든 모듈의 가독성을 높일 수 있음:  
```javascript
// 📁 user.js
export default class User { // just add "default"
  constructor(name) {
    this.name = name;
  }
}

// 📁 main.js
import User from './user.js'; // not {User}, just User

new User('John');
```
- `export default`로 export하면 curly braces를 제외하고 import할 수 있음!

| Named export              | Default export                    |
| :------------------------ | :-------------------------------- |
| `export class User {...}` | `export default class User {...}` |
| `import {User} from ...`  | `import User from ...`            |

default와 named export를 한 모듈에 같이 사용할 수 있지만, 보통 그렇게 사용하지 않음  
하나의 모듈은 named exports를 가지거나 default 하나만을 가짐

파일 하나에 최대 하나의 default export가 존재하기 때문에, exported entity에는 이름이 없음:  
```javascript
// (1)
export default class { // no class name
  constructor() { ... }
}

// (2)
export default function(user) { // no function name
  alert(`Hello, ${user}!`);
}

// (3)
// export a single value, without making a variable
export default ['Jan', 'Feb', 'Mar','Apr', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
```
- `(1)`, `(2)`, `(3)` 모두 다른 파일이라 가정
- `default` 없이 위와 같이 export하면 에러가 남

### The "default" name
`default`가 default export를 참조하는 용도로 사용되기도 함:  
```javascript
function sayHi(user) {
  alert(`Hello, ${user}!`);
}

export {sayHi as default};
```
- `export default function sayHi...`와 같음

또는 default export와 named가 같이 있을 때 아래처럼 둘을 같이 export할 수 있음:  
```javascript
// 📁 user.js
export default class User {
  constructor(name) {
    this.name = name;
  }
}

export function sayHi(user) {
  alert(`Hello, ${user}!`);
}

// 📁 main.js
import {default as User, sayHi} from './user.js';

new User('John');
```
- `import *`으로 모두 import할 경우 아래와 같이 default export를 가져올 수 있음:  
	```javascript
	// 📁 main.js
	import * as user from './user.js';

	let User = user.default; // the default export
	new User('John');
	```

### A word against default exports
named export는 명시적임  
=> named export를 import하기 위해선 정확한 이름을 알아야 함

default export의 경우에는 import할 때 이름을 정할 수 있음:  
```javascript
import User from './user.js'; // works
import MyUser from './user.js'; // works too
```
- `User`를 import할 때 어떤 이름으로 import하든 상관없음  
	=> 팀원끼리 다른 이름을 사용할 수도 있음
	
	이런 혼란을 없애기 위해 보통 파일 이름과 관련되게 import한 것의 이름을 정함:  
	```javascript
	import User from './user.js';
	import LoginForm from './loginForm.js';
	import func from '/path/to/func.js';
	...
	```

## Re-export
`export ... from ...`을 이용해서 어떤 내용을 import하는 즉시 export할 수 있음(다른 이름으로도 가능):  
```javascript
export {sayHi} from './say.js'; // re-export sayHi

export {default as User} from './user.js'; // re-export default
```

배포용 코드에 프로젝트 내부 구조를 건드릴 수 있도록 모듈의 주소를 넣어놓는 것은 위험할 수도 있기 때문에 필요한 것만 export하고 나머지는 숨기는 용도로 사용할 수 있음

**Example**

패키지 구조는 아래와 같음:  
```
auth/
    index.js
    user.js
    helpers.js
    tests/
        login.js
    providers/
        github.js
        facebook.js
        ...
```

아래와 같이 내부 주소를 외부에 노출시키면 위험함:  
```javascript
import {login, logout} from 'auth/tests/login.js'
```

따라서 re-export를 통해 `auth/index.js`에서 패키지를 이용하는데 필요한 것들을 export하도록 만들어야 함:  
```javascript
// 📁 auth/index.js
// re-export login/logout
export {login, logout} from './helpers.js';

// re-export the default export as User
export {default as User} from './user.js';
...
```

### Re-exporting the default export
re-exporting을 할 때 default export는 따로 처리해줘야 함

```javascript
// 📁 user.js
export default class User {
  // ...
}
```
- `export User from './user.js'`는 syntax 에러가 남  
	default export를 re-export할 때는 `export {default as User}`와 같이 적어야 함
- `export * from './user.js'`는 named exports만 re-export하고, default export는 무시함  
	둘 다 re-export하려면 따로 적어야 함:  
	```javascript
	export * from './user.js'; // to re-export named exports
	export {default} from './user.js'; // to re-export the default export
	```
	
	이런 default export의 귀찮은 점 때문에 잘 사용하지 않기도 함

## Summary
### `export` syntax
- Before declaration of a class/function/…:  
	`export [default] class/function/variable ...`
- Standalone export:  
	`export {x [as y], ...}`
- Re-export:  
	`export {x [as y], ...} from "module"`  
	`export * from "module"` (doesn’t re-export default)  
	`export {default [as y]} from "module"` (re-export default)

### `import` syntax
- Importing named exports:  
	`import {x [as y], ...} from "module"`
- Importing the default export:  
	`import x from "module"`  
	`import {default as x} from "module"`
- Import all:  
	`import * as obj from "module"`
- Import the module (its code runs), but do not assign any of its exports to variables:  
	`import "module"`

- default export를 import할 때는 `{}`를 쓰지 않아도 됨
- `import/export`는 script의 어디든지 넣을 수 있음  
	code block안에 넣으면 안됨!  
	`import/export`는 top-level scope에 있어야 함!

# Dynamic imports
위에서 다뤘던 export/import statements는 static임:
1. `import`의 parameter로 `String` 이외에 다른 것들을 넣을 수 없음:  
	```javascript
	import ... from getModuleName(); // Error, only from "string" is allowed
	```
2. `import` statement를 조건문 안에 사용할 수 없음:  
	```javascript
	if(...) {
	  import ...; // Error, not allowed!
	}

	{
	  import ...; // Error, we can't put import in any block
	}
	```

위와 같은 제약이 존재하는 이유는 `import/export`에 의해 코드 실행에 사용할 것들이 모두 준비되어 있어야 하기 때문  
=> bundler에 의해 최적화가 가능함

## The import() expression
`import(module)`은 module을 로드하고, 모든 export를 포함하는 module object를 resolve하는 promise를 리턴함  
이것은 코드의 어느 부분에서는 호출할 수 있음:  
```javascript
let modulePath = prompt("Which module to load?");

import(modulePath)
  .then(obj => <module object>)
  .catch(err => <loading error, e.g. if no such module>)
```

async function 안이라면 아래와 같이 `await`를 사용할 수도 있음:  
```javascript
let module = await import(modulePath);

/*-------------example--------------*/

// 📁 say.js
export function hi() {
  alert(`Hello`);
}

export function bye() {
  alert(`Bye`);
}

// 📁 main.js
let {hi, bye} = await import('./say.js');

hi();
bye();

/*-------------example--------------*/

// 📁 say.js
export default function() {
  alert("Module loaded (export default)!");
}

// 📁 main.js
let obj = await import('./say.js');
let say = obj.default;
// or, in one line: let {default: say} = await import('./say.js');

say();
```

> **Please note:**  
> dynamic import는 regular script에서 동작하기 때문에, `type="module"` attribute가 필요 없음

> **Please note:**  
> `import()`가 함수처럼 생겼지만, 이는 `super()`처럼 그냥 괄호를 사용하는 문법임  
> 따라서 `import`를 다른 변수에 복사하거나 `call/apply`로 call forwarding 할 수 없음

