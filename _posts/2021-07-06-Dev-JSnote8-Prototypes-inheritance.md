---
layout: single
title: "JSnote8: Prototypes, inheritance"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - JavaScript
  - prototype
  - F.prototype
  - Object.prototype
  - Native prototype
  - __proto__
  - very plain object
  - associative array
---

Prototypes, inheritance
# Prototypal inheritance
`user`라는 객체를 만든 상태에서, 이것을 조금 변형해 `admin`, `guest`를 만들고 싶을 때와 같을 때 *prototypal inheritance*가 사용됨

## `[[Prototype]]`
specification에 명시된 hidden property인 `[[Prototype]]`은 `null` 또는 다른 object(prototype이라 불림)의 reference를 저장함  
한 객체로부터 property를 읽는데 없다면, JS는 자동으로 prototype 객체로부터 그것을 찾음  
⇔ prototypal inheritance(프로토타입 상속)

| ![js-prototype1](https://github.com/siriyaoff/MDN-note/blob/master/images/js-prototype1.PNG?raw=true) |
| :---------------------------------------------------------------------------------------------------: |
|                                         javascript.info 참고                                          |

`[[Prototype]]`은 숨겨져 있지만, 몇 가지 방법으로 설정 가능함  
그 중 하나가 `__proto__`를 사용하는 것임:  
```javascript
let animal = {
  eats: true,
  walk() {
    alert("Animal walk");
  }
};
let rabbit = {
  jumps: true
};

rabbit.__proto__ = animal; // (*)

alert( rabbit.eats );
alert( rabbit.jumps ); // true
rabbit.walk(); // Animal walk
```
- `(*)`와 같이 prototypal inheritance를 설정함  
	`animal`은 `rabbit`의 prototype임  ⇔ `rabbit`은 `animal`을 상속받음

prototypal inheritance의 제한 사항:
1. inheritance의 reference는 cycle을 만들면 에러남!
2. `__proto__`의 값은 `Object`이거나 `null`이어야 함!
3. `[[Prototype]]`에는 하나의 객체만 들어감  
	prototype chain이 길어질 수는 있음

> #### `__proto__`는 `[[Prototype]]`을 위한 getter/setter임
> `__proto__`는 `[[Prototype]]`와 동일한게 아니라, `[[Prototype]]`의 getter/setter 역할을 함   
> `__proto__`는 현재 잘 사용되지 않고, 호환성을 위해서 존재함  
> 최신의 JS에서는 `Object.getPrototypeOf`/`Object.setPrototypeOf`를 사용하길 권장함  
> 
> `__proto__`는 specification에 의하면 browser에서만 지원되어야 하지만, 실제로는 서버를 포함해서 모든 환경에서 지원됨  
> `__proto__`가 더 직관적이기 때문에 이 article에서는 이것을 사용함

## Writing doesn't use prototype
prototype은 property를 읽을 때만 사용됨  
property를 수정하거나 삭제하는 것은 직접 object를 사용해야 함:  
```javascript
let animal = {
  eats: true,
  walk() {
    /* this method won't be used by rabbit */
  }
};

let rabbit = {
  __proto__: animal
};

rabbit.walk = function() {
  alert("Rabbit! Bounce-bounce!");
};

rabbit.walk(); // Rabbit! Bounce-bounce!
```
- `rabbit`의 prototype인 `animal`에 `walk()` method가 있지만, `rabbit.walk`에 함수를 넣으면 `rabbit`에 `walk()` method가 추가됨  
	=> `rabbit.walk()`를 호출하면 prototype 대신 `rabbit`에 있는 메소드가 호출됨

Accessor property는 getter/setter 안에 들어가는 property의 존재 여부에 따라서 다른 결과가 나옴:  
```javascript
let user = {
  name: "John",
  surname: "Smith",

  set fullName(value) {
    [this.name, this.surname] = value.split(" ");
  },

  get fullName() {
    return `${this.name} ${this.surname}`;
  }
};

let admin = {
  __proto__: user,
  isAdmin: true
};

alert(admin.fullName); // John Smith (*)

// setter triggers!
admin.fullName = "Alice Cooper"; // (**)

alert(admin.fullName); // Alice Cooper, state of admin modified
alert(user.fullName); // John Smith, state of user protected
```
- `(*)`에서는 `admin`에 `name/surname`이 없기 때문에 prototype의 그것이 호출됨
- `(**)`에서 setter에 의해 `admin`에 `name/surname`이 추가됨  
	=> 마지막에는 `this`의 값이 다르기 때문에 서로 다른 property가 호출됨!

## The value of "this"
`this`는 prototype에 영향을 받지 않음  
method가 object에 선언되든지 prototype에 선언되든지 상관없이, `this`는 항상 점 이전의 객체를 가리킴!  
=> `admin.fullName`에서는 `this`가 `admin`을 가리킴

상속 관계가 복잡해도 이로 인해 편리하게 inherited method 실행 가능  
즉, 상속하면 method는 공유되지만 object state는 공유되지 않음(method 내에서 property를 수정하면 `this`에 해당하는 객체의 상태가 변화함)

#### Example
```javascript
let animal = {
  walk() {
    if (!this.isSleeping) {
      alert(`I walk`);
    }
  },
  sleep() {
    this.isSleeping = true;
  }
};

let rabbit = {
  name: "White Rabbit",
  __proto__: animal
};

rabbit.sleep();

alert(rabbit.isSleeping); // true
alert(animal.isSleeping); // undefined (no such property in the prototype)
```
- prototype에 정의된 메소드 `sleep()`을 `rabbit`에서 호출했으므로 `this`가 `rabbit`을 가리켜 `rabbit`에 `isSleeping`이라는 property가 생김

| ![js-object-state1](https://github.com/siriyaoff/MDN-note/blob/master/images/js-object-state1.PNG?raw=true) |
| :---------------------------------------------------------------------------------------------------------: |
|                               위 예시의 object state<br>javascript.info 참고                                |

## `for...in` loop
`for...in`은 상속받은 property도 순회함  
`obj.hasOwnProperty(key)`로 `obj`가 소유한 key인지 판별할 수 있음:  
```javascript
let animal = {
  eats: true
};

let rabbit = {
  jumps: true,
  __proto__: animal
};

alert(Object.keys(rabbit)); // jumps

for(let prop in rabbit) alert(prop); // jumps, then eats

for(let prop in rabbit) {
  let isOwn = rabbit.hasOwnProperty(prop);

  if (isOwn) {
    alert(`Our: ${prop}`); // Our: jumps
  } else {
    alert(`Inherited: ${prop}`); // Inherited: eats
  }
}
```
- `Object.keys(obj)`는 `obj`의 key만 리턴(key는 method도 포함됨!!)
- `for...in`은 inherited key도 리턴
- 모든 literal object들은 `Object.prototype`을 상속받음  
	`Object.prototype`은 `[[Prototype]]`이 `null`임
- `Object.prototype`의 모든 property들은 `enumerable: false`이기 때문에 `for...in`에 나열되지 않음

> #### key나 value를 가져오는 거의 모든 method들은 상속받는 property를 제외함  
> `Object.keys/values`와 같은 method들은 inherited property를 무시함  
> 즉, 호출된 객체 안의 key에 대해서만 동작하고, prototype의 key들은 가져오지 않음

## Summary

| code                       | description                            |
| :------------------------- | :------------------------------------- |
| `obj.hasOwnProperty(prop)` | `obj`가 `prop`라는 key를 가지는지 판별 |

- hidden property `[[Prototype]]`에 prototype의 reference가 저장됨
	- `__proto__`를 사용해서 설정 가능
- 상속받은 property들은 수정할 수 없음(대입 연산을 하면 현재 객체에 새 property가 생김)
- 상속받은 method에서도 `this`는 항상 점 이전의 객체를 가리킴
- 모든 literal object들은 `Object.prototype`을 상속받음
- `Object.prototype`의 property, method들은 모두 `enumerable: false`임
- key나 value를 가져오는 거의 모든 method들은 상속받는 property를 제외함

## Tasks
- 최신의 엔진에서 property를 호출할 때 object에서 가져오나 prototype에서 가져오나 성능 차이가 없음  
	∵ 첫 번째 가져올 때 탐색하고 위치를 저장해놓기 때문에 다음 요청때는 바로 가져올 수 있음

# F.prototype
constructor function을 이용해서 `new F()`와 같이 객체를 생성할 수 있음  
만약 `F.prototype`이 객체라면, `new` operator로 생성되는 새 객체의 `[[Prototype]]`에 `F.prototype`의 reference를 저장함  
`F.prototype`은 단순히 `F`의 property `prototype`을 의미하는 것이지, 다른 뜻은 없음!!!

#### Example
```javascript
let animal = {
  eats: true
};

function Rabbit(name) {
  this.name = name;
}

Rabbit.prototype = animal;

let rabbit = new Rabbit("White Rabbit"); //  rabbit.__proto__ == animal

alert( rabbit.eats ); // true
```
- `Rabbit.prototype = animal`은 `new Rabbit`이 생성될 때 그것의 `[[Prototype]]`에 `animal`을 대입해라는 의미임  

 | ![js-f-prototype1](https://github.com/siriyaoff/MDN-note/blob/master/images/js-f-prototype1.PNG?raw=true) |
 | :-------------------------------------------------------------------------------------------------------: |
 |                                           javascript.info 참고                                            |
	
	가로 화살표는 평범한 property를 뜻하고 세로 화살표는 inheritance를 뜻함

> #### `F.prototype`은 `new F`로 호출될 때만 사용됨
> `new F`로 객체가 하나 생성된 다음 `F.prototype`이 바뀐다해도 생성된 객체의 `[[Prototype]]`은 기존의 것으로 유지됨  
> reference를 복사하는 것이기 때문

## Default F.prototype, constructor property
모든 함수들은 우리가 선언해주지 않아도 `prototype` property를 가짐  
default `prototype` property는 `{ constructor: F }`와 같은 객체임(`F`는 함수 자기 자신):  
```javascript
function Rabbit() {}

/* default prototype
Rabbit.prototype = { constructor: Rabbit };
*/

alert( Rabbit.prototype.constructor == Rabbit ); // true
```

따라서 `new F`로 생성된 객체들은 `[[Prototype]]` property를 통해서 `F.constructor`에 접근 가능함:  
```javascript
function Rabbit() {}
let rabbit = new Rabbit(); // inherits from {constructor: Rabbit}

alert(rabbit.constructor == Rabbit); // true (from prototype)
```

| ![js-f-prototype2](https://github.com/siriyaoff/MDN-note/blob/master/images/js-f-prototype2.PNG?raw=true) |
| :-------------------------------------------------------------------------------------------------------: |
|                                           javascript.info 참고                                            |

아래와 같이 기존의 객체의 constructor를 생성자로 사용할 수도 있음:  
```javascript
function Rabbit(name) {
  this.name = name;
  alert(name);
}

let rabbit = new Rabbit("White Rabbit");

let rabbit2 = new rabbit.constructor("Black Rabbit");
```
- 객체의 생성자를 모르는데(e.g. 써드파티의 객체) 같은 종류의 객체를 생성하고 싶을 때 유용함

`F.prototype`이 `constructor` property를 가지는 객체이긴 하지만, 우리가 수정할 수 있기 때문에 없어질 수도 있음:  
```javascript
function Rabbit() {}
Rabbit.prototype = {
  jumps: true
};

let rabbit = new Rabbit();
alert(rabbit.constructor === Rabbit); // false
```

`constructor`를 유지하면서 `F.prototype`을 수정하는 방법:  
```javascript
// (1)

function Rabbit() {}
Rabbit.prototype.jumps = true

// (2)

Rabbit.prototype = {
  jumps: true,
  constructor: Rabbit
};
```
1. `F.prototype` 전체를 덮어쓰지 않고 property를 하나씩 추가
2. 덮어쓰는 대신 `constructor: Rabbits`를 넣어줘야 함

## Summary
- `F.prototype`은 `new F()`로 생성된 객체들의 `[[Prototype]]`을 자신이 저장하고 있는 레퍼런스로 설정함
- `F.prototype`의 값은 객체이거나 `null`이어야 함(`[[Prototype]]`과 마찬가지)
- `prototype` property는 생성자 함수의 property이고, 생성자 함수로 객체가 생성될 때만 위와 같은 기능을 함  
	cf. 보통의 객체에서는 기본적으로 생성되지도 않으며, 아무런 영향이 없음
- 생성자 함수를 포함한 *모든 함수*들은 `F.prototype = { constructor: F }`를 가짐  
	=> 객체가 있다면 해당 객체의 생성자에도 접근할 수 있음
	- property를 추가할 때 덮어쓰지 않도록 주의해야 함!

## Tasks
```javascript
function Rabbit() {}
Rabbit.prototype = {
  eats: true
};

let rabbit = new Rabbit();

Rabbit.prototype = {};

alert( rabbit.eats ); // ?
```
- `true`  
	기존의 `Rabbit.prototype` 객체는 `rabbit`에서 reference를 사용하기 때문에 `Rabbit.prototype`에서 참조하지 않아도 garbage collected되지 않음  
	`Rabbit.prototype`은 `{}`의 reference로 바뀜  
	=> 이후에 `new Rabbit`으로 만들어지는 객체들은 `{}`의 reference를 `[[Prototype]]`에 저장함
- inherited property는 읽기만 가능, 수정, 삭제는 inherited property의 key를 호출하더라도 점 앞의 객체의 property로 간주함(e.g. `delete rabbit.eats;`를 하더라도 `prototype`의 property가 지워지지 않음, 대입도 마찬가지로, `rabbit.eats`가 만들어지고 거기에 값이 대입됨)
- `rabbit.prototype`는 존재하지 않음!!  
	∵ `F.prototype`만 위와 같이 사용되기 때문!  
	
	따라서, `Rabbit.prototype.eats`를 상속받은 객체 `rabbit`에서 접근하기 위해선 `rabbit.__proto__.eats`와 같이 접근해야 함!!
- `Rabbit.prototype={};`를 실행해서 지운 다음 `rabbit.constructor()`를 호출하면 `Rabbit`의 prototype인 `Object.prototype`의 constructor가 실행됨!  
	이 constructor는 `new Object(...)`와 같이 사용하지 않음!!  
	`new Object()`로 empty object를 생성하기는 함

# Native prototypes
모든 내장된 생성자 함수들은 `prototype` property를 사용함

## Object.prototype
```javascript
let obj = {};
alert( obj ); // "[object Object]"
```
- `obj`는 empty object인데 `"[object Object]"`로 바꿔주는 `toString` method이 어디서 나온 걸까?  
	`obj = {}`은 `obj = new Object()`의 shorthand이기 때문에 `Object.prototype`에 선언된 `toString`을 사용한 것임  
	즉, `obj.toString`과 `obj.__proto__.toString`과 `Object.prototype.toString`은 모두 같은 레퍼런스임  
	
 | ![js-object-prototype1](https://github.com/siriyaoff/MDN-note/blob/master/images/js-object-prototype1.PNG?raw=true) |
 | :-----------------------------------------------------------------------------------------------------------------: |
 |                                                javascript.info 참고                                                 |

`Object.prototype`의 `[[Prototype]]`은 `null`임!

## Other built-in prototypes
`Array`, `Date`, `Function` 등의 다른 내장 객체들도 prototype에 method를 저장함  
예를 들어, `[1, 2, 3]`으로 생성한 array도 `new Array()`를 사용하는데, constructor function과 다른 method들은 `Array.prototype`에 정의되어 있음  
=> 모든 객체에 각각 method가 정의되어 있는 것보다 메모리를 절약할 수 있음

specification에 의하면, 모든 내장 prototype들은 `Object.prototype`을 상속받음  

| ![js-built-in-prototypes1](https://github.com/siriyaoff/MDN-note/blob/master/images/js-built-in-prototypes1.PNG?raw=true) |
| :-----------------------------------------------------------------------------------------------------------------------: |
|                                                   javascript.info 참고                                                    |

```javascript
let arr = [1, 2, 3];

alert( arr.__proto__ === Array.prototype ); // true

alert( arr.__proto__.__proto__ === Object.prototype ); // true

alert( arr.__proto__.__proto__.__proto__ ); // null

alert(arr.toString === Array.prototype.toString); // true

alert(arr.toString === Object.prototype.toString); // false
```
- `Array.prototype`, `Object.prototype` 둘 다 `toString` method를 가지고 있지만, `arr`에서는 더 가까운 `Array.prototype.toString`을 사용함

`console.dir(obj)`를 이용하면 `obj`의 prototype chain을 알 수 있음:  

| ![js-console-dir](https://javascript.info/article/native-prototypes/console_dir_array.png) |
| :----------------------------------------------------------------------------------------: |
|                                    javascript.info 참고                                    |

`Function`도 마찬가지로, `call`, `apply` 같은 method들은 `Function.prototype`에 정의되어 있음

## Primitives
`String`, `Number`, `Boolean`은 객체가 아니라 primitive지만, property에 접근할 때 임시적으로 wrapper object가 생성됨(`String`, `Number`, `Boolean`의 생성자 사용해서)  
=> 그 wrapper object가 method를 제공한 다음 사라짐

위의 과정이 specification에 명시되어 있는 내용이고, 실제로 wrapper object들은 우리가 볼 수 없고, 대부분의 엔진들이 최적화함  
이 wrapper object들의 method들도 prototype(`String.prototype`, `Number.prototype`, `Boolean.prototype`)에 정의되어 있음

> #### `null`, `undefined`는 wrapper object가 없음
> 따라서 사용할 수 있는 method나 property, prototype도 존재하지 않음

## Chainging native prototypes
Native prototype을 수정할 수 있음  
예를 들어, `String.prototype`에 method를 추가하면, 그 method는 모든 `String`에서 사용 가능:  
```javascript
String.prototype.show = function() {
  alert(this);
};

"BOOM!".show(); // BOOM!
```
- 수정할 수는 있지만, 좋은 생각이 아님!

> #### 주의해야 할 점
> prototype은 global하기 때문에 충돌이 일어나기 쉬움  
> 예를 들어 두 라이브러리가 `String.prototype.show`라는 method를 정의하면, 둘 중 하나는 덮어씌워짐  
> 따라서, 일반적으로 native prototype을 수정하는 것은 좋지 않음

최근에는 polyfill을 목적으로 하는 native prototype 수정은 권장됨  
(Remind: polyfill은 JS specification에는 명시되어 있지만, 특정한 엔진에서 지원되지 않는 내용을 위한 대체제를 만드는 것임)

#### Example
```javascript
if (!String.prototype.repeat) {
  String.prototype.repeat = function(n) {
    // repeat the string n times
    return new Array(n + 1).join(this);
  };
}

alert( "La".repeat(3) ); // LaLaLa
```
- method가 지원되지 않으면 polyfill을 넣음

## Borrowing from prototypes
native prototype의 method를 주로 빌림:  
```javascript
let obj = {
  0: "Hello",
  1: "world!",
  length: 2,
};

obj.join = Array.prototype.join;

alert( obj.join(',') ); // Hello,world!
```
- 이전 article에서는 empty array를 이용해서 `[].join`과 같이 method를 빌렸지만, native prototype을 이용해서 method를 빌릴 수도 있음
	- `join`이 index, `length`만 사용하기 때문에 array-like object에도 사용할 수 있음  
		이렇게 data type이 맞지 않아도 사용할 수 있는 경우가 많음
- `obj.__proto__`를 `Array.prototype`으로 설정해서 `obj.join`으로 바로 사용해도 됨  
	이 방법은 이미 `obj`가 다른 객체를 상속하고 있을 경우 사용할 수 없음  
	∵ 한 객체는 동시에 하나의 객체로부터만 상속받을 수 있기 때문!

## Summary
- 모든 내장 객체들은 같은 패턴을 따름
	- method들은 모두 prototype에 저장됨
	- 객체 자체는 값(array item, date 등)이나 property만 저장함
- primitive도 method를 wrapper object의 prototype에 저장함  
	cf. `null`, `undefined`는 wrapper object가 없음
- built-in prototype도 변경할 수 있지만, polyfill을 할 때만 권장됨

## Tasks
### Decorator 역할을 하는 method를 prototype에 선언
```javascript
function f() {
  alert("Hello!");
}

Function.prototype.defer = function(n) {
  setTimeout(this, n);
};

f.defer(1000); // shows "Hello!" after 1 second
```
- `Function`에서 `this`가 함수 자체이기 때문에 가능함

### decorating defer를 prototype에 선언
```javascript
// (1)
Function.prototype.defer = function(ms) {
  let cthis=this;
  return function() {
    setTimeout(() => cthis.apply(cthis, arguments), ms);
  };
};

// (2)
Function.prototype.defer = function(ms) {
  return (function() {
    setTimeout(() => this.apply(this, arguments), ms);
  }).bind(this);
};

// (3)
Function.prototype.defer = function(ms) {
  return (...args) => setTimeout(() => this.apply(this, args), ms);
};

function f(a, b) {
  alert( a + b );
}

f.defer(1000)(1, 2); // shows 3 after 1 second
```
- `(1)`은 정석대로 wrapper를 function expression으로 정의하고, `cthis`로 context를 넘겨줌
- `(2)`는 context를 bind해서 따로 넘겨주지 않아도 됨
- `(3)`은 wrapper를 rest parameter를 받는 arrow function으로 구현함  
	arrow function들은 outer context를 사용하기 때문에 nested arrow function은 모두 같은 context를 사용하는 것을 알 수 있음

# Prototype methods, objects without __proto__
`__proto__` 대신에 아래와 같은 최신 method들을 사용해야 함:
- `Object.create(proto[, descriptors])` : `proto`를 `[[Prototype]]`으로 하고 `descriptor`를 적용한 객체 생성
- `Object.getPrototypeOf(obj)` : `obj`의 `[[Prototype]]` 리턴
- `Object.setPrototypeOf(obj, proto)` : `obj`의 `[[Prototype]]`을 `proto`로 설정

#### Example
```javascript
let animal = {
  eats: true
};

let rabbit = Object.create(animal);
/*
let rabbit = Object.create(animal, {
  jumps: {
    value: true
  }
});
*/

alert(rabbit.eats); // true

alert(Object.getPrototypeOf(rabbit) === animal); // true

Object.setPrototypeOf(rabbit, {}); // change the prototype of rabbit to {}
```
- 주석과 같이 `Object.create`에 descriptor를 넣어서 적용 가능함

아래와 같이 `for...in`으로 property를 복사하는 것보다 더 효과적인 복사를 수행할 수 있음:  
```javascript
let clone = Object.create(Object.getPrototypeOf(obj), Object.getOwnPropertyDescriptors(obj));
```
- shallow copy지만, `[[Prototype]]`, setter/getter, property flag까지 복사함

## Brief history
역사적인 이유로, `[[Prototype]]`에 접근하는 방법은 많음:
- 생성자 함수의 `F.prototype`은 오래 전부터 사용되어 왔음
- 2012년에 `Object.create`가 표준으로 정의되어 prototype을 사용해서 객체를 생성할 수 있게 됨  
	하지만 prototype에 접근하거나 설정할 수 있는 기능을 제공하지 않음  
	=> browser들이 비표준인 `__proto__`를 구현함
- 2015년에 `Object.setPrototypeOf`, `Object.getPrototypeOf`가 표준에 추가됨  
	`__proto__`와 같은 역할이지만, 이미 `__proto__`가 사실표준으로 굳어진 상태라 `__proto__`도 표준의 부록에 추가됨

> #### 현재 생성된 객체의 `[[Prototype]]`은 변경하지 않는게 좋음
> `[[Prototype]]`을 언제든지 get/set할 수 있지만, 보통 객체가 생성될 때 한 번만 설정한 후 건드리지 않음  
> 
> JS 엔진들은 이렇게 한 번만 설정하는 것에 최적화되어 있음  
> `Object.setPrototypeOf`나 `obj.__proto__`를 사용해서 prototype을 바꾸는 것은 최적화되지 않아서 매우 느림  
> 따라서 속도가 중요할 때는 prototype을 변경하지 않는게 좋음!

## "Very plain" objects
computed property는 `[]`를 사용해서 호출할 수 있음  
하지만 이 방법은 `"__proto__"`를 제외한 나머지 property에 대해 적용 가능함:  
```javascript
let obj = {};

let key = prompt("What's the key?", "__proto__");
obj[key] = "some value";

alert(obj[key]); // [object Object], not "some value"!
```
- `obj["__proto__"]`는 대입 연산이 무시됨
	- `__proto__` property는 `null`이나 객체를 참조해야 하기 때문
	
	원래 의도는 `__proto__`라는 property에 `"some value"`를 저장하는 것이지만, JS가 이미 `__proto__`를 사용하고 있기 때문에 의도한대로 실행되지 않은 것임 + 객체가 대입된다면 대입 연산이 적용되어 오류가 발생함  
	현재는 이미 `__proto__`가 property가 아닌 `Object.prototype`에 관한 accessor property로 구현되어 있으므로 객체를 대입해도 잘못된 대입을 피할 수 있음
	
	
	`toString`과 같은 내장 method도 위와 같이 잘못된 대입이 일어날 수 있음  
	=> 그냥 객체 대신에 `Map`을 사용하면 해결됨  

또는 아래 코드와 같이 `[[Prototype]]`을 `Object.prototype`이 아닌 `null`로 명시해서 위 문제를 해결할 수 있음:  
```javascript
let obj = Object.create(null);

let key = prompt("What's the key?", "__proto__");
obj[key] = "some value";

alert(obj[key]); // "some value"
```
- `obj`는 `[[Prototype]]`에 `null`이 들어간 상태임  
	=> `__proto__` getter, setter가 상속되지 않았으므로 `__proto__`도 평범한 property로 사용됨
- 이런 객체를 very plain object 또는 pure dictionary object라고 부름
- 상속받은 것이 없기 때문에 `toString`같은 다른 내장 method도 사용할 수 없음  
	=> `alert(obj)`가 에러를 반환함
- very plain object를 associative array로 사용할 때는 문제가 되지 않음
- `Object.f()`와 같은 method들은 prototype안에 선언된게 아니기 때문에 very plain object에도 사용할 수 있음!

## Summary

| code                                  | description                                                        |
| :------------------------------------ | :----------------------------------------------------------------- |
| `Object.create(proto[, descriptors])` | `proto`를 `[[Prototype]]`으로 하고 `descriptor`를 적용한 객체 생성 |
| `Object.getPrototypeOf(obj)`          | `obj`의 `[[Prototype]]` 리턴                                       |
| `Object.setPrototypeOf(obj, proto)`   | `obj`의 `[[Prototype]]`을 `proto`로 설정                           |

- 사용자의 입력을 key로 사용하는 경우 `__proto__`가 호출될 수도 있기 때문에 위험함  
	=> 이런 경우 very plain object를 사용하거나 `Map`을 사용하는게 좋음
- `__proto__`는 `[[Prototype]]`의 getter/setter이고, `Object.prototype`에 선언되어 있음

다른 method들:
- `Object.keys/values/entries(obj)` : 각각의 enumerable의 array 리턴
- `Object.getOwnPropertySymbols(obj)` : symbolic key의 array 리턴
- `Object.getOwnPropertyNames(obj)` : string key의 array 리턴
- `Reflect.ownKeys(obj)` : 모든 key의 array 리턴
- `obj.hasOwnProperty(key)` : `key`라는 `obj` 소유의 property가 있는지 판별

object property를 리턴하는 메소드들은 자신 소유의 property만 탐색함  
상속받은 property까지 보려면 `for...in`을 사용하는게 좋음

## Tasks
- descriptor를 사용해서 property를 생성할 때 생략된 flag들은 모두 `false`로 초기화됨!

```javascript
function Rabbit(name) {
  this.name = name;
}
Rabbit.prototype.sayHi = function() {
  alert( this.name );
}

let rabbit = new Rabbit("Rabbit");

rabbit.sayHi();                        // Rabbit
Rabbit.prototype.sayHi();              // undefined
Object.getPrototypeOf(rabbit).sayHi(); // undefined
rabbit.__proto__.sayHi();              // undefined
```