---
layout: single
title: "JSnote4: Objects: the basics"
categories:
  - Dev
tags:
  - JavaScript
  - Object literal
  - Computed property
  - Constructor function
  - new.target
  - Garbage collection
  - this
  - new
  - Optional chaining
  - Symbol
  - Object conversion
  - Symbol.toPrimitive
  - toString
---

Objects: the basics
# Objects
object는 `{...}` 안에 `key: value`와 같이 properties를 선언함으로써 만들어짐  
empty object는 object constructor, object literal, 두 가지 방법으로 만들 수 있음:  
```javascript
let user = new Object(); // "object constructor" syntax
let user = {};  // "object literal" syntax
```
- 보통 object literal로 객체를 선언함

## Literals and properties
처음 선언할 때 아래와 같이 property를 선언할 수 있음:  
```javascript
let user = {     // an object
  name: "John",  // by key "name" store value "John"
  age: 30,        // by key "age" store value 30
  "likes birds": true,
};
```
- 따옴표를 사용하면 property 이름으로 여러 단어를 사용할 수 있음!
- `user.isAdmin=true;`와 같이 선언한 이후에도 property를 추가할 수 있음
- 마지막 줄에도 `,`를 남기는 것을 "trailing" or "hanging" comma라고 함  
	=> properties를 쉽게 수정할 수 있음

`delete user.age;`와 같이 `delete` operator를 이용해서 property를 삭제할 수 있음

## Square brackets
multiword properties의 경우 `user.likes birds`로 호출할 수 없음  
(∵ `.`으로 호출하기 위해선 valid variable identifier이어야 함(공백 없음, 숫자로 시작 x, 특수문자 x)  
=> `user.["likes birds"]`와 같이 square bracket notation을 이용해서 호출해야 함

key도 string이기 때문에 아래와 같이 string을 이용해서 호출도 가능함  
```javascript
let key = "likes birds";

// same as user["likes birds"] = true;
user[key] = true;
```
- `user.key`로는 호출할 수 없음에 주의!!
- python의 dictionary처럼 property를 추가할 수 있다고 생각하면 될 듯

### Computed properties
object literal으로 객체를 선언할 때 `[]`를 사용할 수 있음  
이렇게 선언된 property를 *computed property*라고 함  
```javascript
let fruit = prompt("Which fruit to buy?", "apple");

let bag = {
  [fruit]: 5, // the name of the property is taken from the variable fruit
};

alert( bag.apple ); // 5 if fruit="apple"
```
- `[fruit+'Computers']`와 같이 property를 선언할 때 concatenation을 해도 상관없음

## Property value shorthand
```javascript
function makeUser(name, age) {
  return {
    name: name,
    age: age,
    // ...other properties
  };
}

let user = makeUser("John", 30);
alert(user.name); // John
```

위 코드의 object literal 부분을 아래와 같이 shorthand를 사용해서 바꿀 수 있음

```javascript
function makeUser(name, age) {
  return {
    name, // same as name: name
    age,  // same as age: age
    // ...
  };
}
```
- property name(key)와 value가 같을 경우 사용 가능

## Property names limitations
object는 "for", "let", "return"등의 reserved words도 property name으로 사용 가능함  
※ `0`과 같은 숫자도 이름으로 사용 가능!!  
```javascript
let obj = {
  0: "test" // same as "0": "test"
};

// both alerts access the same property (the number 0 is converted to string "0")
alert( obj["0"] ); // test
alert( obj[0] ); // test (same property)
```
- `0`이 자동으로 `"0"`(`string`)으로 바뀜
- `obj["0"]`, `obj[0]` 두 가지 방식으로 호출할 수 있음  

> 객체 배열을 선언하면 그것과는 구별을 어떻게 하나?  
> - Integer property와 배열은 코드에서 바로 구분할 수는 없을 듯  
>	꼭 알아야 할 상황이면 `Array.isArray()`를 사용하면 됨

`__proto__`는 non-object value로 설정할 수 없음!  
`obj.__proto__`와 같이 호출하면 아예 object가 리턴됨

## Property existence test, "in" operator
다른 언어들과 비교해서 object에 관한 JS의 가장 큰 특징은 어떤 property라도 접근할 수 있다는 것임  
property가 존재하지 않아도 가능  
=> 존재하지 않는 property를 호출하면 `undefined`를 리턴함

`in` operator를 사용해서 object에 property가 있는지 확인할 수 있음  
`"key" in object` => `key`라는 property가 존재하면 `true`, 아니면 `false` 리턴  
`"key"` 대신 변수를 넣으면 해당 변수의 value가 object의 property인지 확인함
- property가 `undefined`를 저장하고 있을 때 유용(`undefined`를 명시적으로 대입할 일이 별로 없기 때문에 이런 상황은 잘 일어나지 않음)

## The "for...in" loop
```javascript
let user = {
  name: "John",
  age: 30,
  isAdmin: true
};

for (let key in user) {
  // keys
  alert( key );  // name, age, isAdmin
  // values for the keys
  alert( user[key] ); // John, 30, true
}
```
- `in` 뒤에 object가 오면 iterator에 property name이 들어감

### Ordered like an object
object의 poperties가 *integer properties*인지 아닌지에 따라서 나열되는 기준이 다름

Integer property : `"1"`, `"41"`과 같이 property name을 Integer로 변환했다가 다시 `string`으로 변환해도 값이 같은 property(`"+41"`, `"1.2"`는 같지 않음)
⇔ `name == String(Math.trunc(Number(name)))`

"for...in"으로 key를 나열하면, 
- Integer property인 경우 크기 순으로 나열됨
- 아닌 경우 생성된 순으로 나열됨

※ 숫자들을 key로 사용하고 싶지만 생성된 순으로 나열되기 하고 싶을 때는 `"+49"`와 같이 선언하고 출력할 때 `number`로 변환해서 출력하면 됨!!

## Summary

|code|description|
|:---|:---|
|`let user = {};`|object literal|
|`user[key]=1;`|computed properties<br>object literal에서도 사용 가능|
|`"key" in obj`|이름이 `"key"`인 property 존재 여부|
|`for(let key in user) {...}`|properties 탐색<br>숫자 property만 오름차순, 나머지는 생성 시간 기준으로 나열됨|

- property 선언
	- object literal 안에서 : `name: "John",`
	- 선언 후 : `user.name="John";`
- `Array`, `Date`, `Error` 등의 다양한 `object`들이 존재함(나중에 배울 예정)

## Tasks
객체의 isEmpty를 아래와 같이 구현할 수 있음:  
```javascript
function isEmpty(obj) {
  for (let key in obj) {
    // if the loop has started, there is a property
    return false;
  }
  return true;
}
```

# Object references and copying
object는 저장, 복사될 때 "call by reference"로 처리됨  
(cf. primitive는 "call by value"로 처리됨)  
```javascript
let user = { name: 'John' };

let admin = user;

admin.name = 'Pete'; // changed by the "admin" reference

alert(user.name); // 'Pete', changes are seen from the "user" reference
```

## Comparison by reference
아예 선언부터 독립적으로 해야 독립적인 두 개의 객체가 생성됨

## Cloning and merging, Object.assign
object cloning을 위한 내장 함수는 없음  
=> 두 가지 방법 존재:
1. for...in을 사용해서 properties를 복사  
	```javascript
	let user = {
	  name: "John",
	  age: 30
	};

	let clone = {}; // the new empty object

	// let's copy all user properties into it
	for (let key in user) {
	  clone[key] = user[key];
	}

	// now clone is a fully independent object with the same content
	clone.name = "Pete"; // changed the data in it

	alert( user.name ); // still John in the original object
	```
2. `Object.assign` method 이용  
	```javascript
	Object.assign(dest, [src1, src2, src3...])
	```
	- `src1, ..., srcN`은 source **objects**임
	- source objects의 모든 properties는 `dest`로 복사됨
	- `dest`를 리턴함
	
	```javascript
	// (1)
	let user = { name: "John" };

	let permissions1 = { canView: true };
	let permissions2 = { canEdit: true };

	Object.assign(user, permissions1, permissions2, { name: "Pete" });

	// now user = { name: "Pete", canView: true, canEdit: true }
	
	// (2)
	let user = {
	  name: "John",
	  age: 30
	};

	let clone = Object.assign({}, user);
	```
	
	- (2) : 위의 `for...in`을 이용한 방법이랑 같은 방법임

## Nested cloning
```javascript
let user = {
  name: "John",
  sizes: {
    height: 182,
    width: 50
  }
};
```
- 위와 같은 nested object의 경우 위 방법(`for...in`, `object.assign`)만으로는 복사가 완벽하게 되지 않음  
	`object` type인 property가 reference로 복사되기 때문  
	=> "shallow copy"라고 함
- 따라서 properties의 type을 확인하고 `object`일 경우 따로 복제해줘야 함  
	=> "deep cloning"이라고 함
	
	재귀로 구현하거나 [lodash](https://lodash.com/)의 `_.cloneDeep(obj)` 이용

※ object는 `const`로 선언해도 properties는 수정할 수 있음(`name=...`와 같이 객체 자체를 수정하려 할 때만 에러남)  
properties를 constant로 만들기 위해선 Property flags를 사용해야 함!(나중에 다룸)

## Summary

|code|description|
|:---|:---|
|`Object.assign(dest, [src1, src2...]);`|object cloning(shallow copy)|

- `object`는 호출될 때 항상 call by reference로 처리됨
- deep copy는 lodash의 `_.cloneDeep(obj)` 이용

# Garbage collection
## Reachability
*Reachability*(*도달 가능성*)이 JS memory management의 핵심 개념임  
"reachable" values는 memory에 저장될 수 있게 보장됨
1. 명확한 이유로 reachable values인 값들을 *roots*라고 부름
	- 현재 실행중인 함수와 그 안의 지역 변수, 인자들
	- 중첩된 호출에 속한 함수들과 그 지역 변수, 인자들
	- 전역 변수들  
		...
2. root를 참조할 경우 reachable이라 판단됨  
	예를 들어, object A가 전역 변수고 다른 object B를 property로 참조하고 있으면 B도 reachable임

garbage collector가 unreachable한 objects, primitives를 제거함

※ garbage collection은 자동적으로 일어나고, 우리가 강제할 수 없음

## Two references
unreachable해야 garbage collect됨

## Interlinked objects
```javascript
function marry(man, woman) {
  woman.husband = man;
  man.wife = woman;

  return {
    father: man,
    mother: woman
  }
}

let family = marry({
  name: "John"
}, {
  name: "Ann"
});
```

|Result:|
|:---|
|![js-garbage-collection1](https://github.com/siriyaoff/MDN-note/blob/master/images/js-garbage-collection1.PNG?raw=true)|

- 모든 object가 reachable함

두 개의 references 제거:  
```javascript
delete family.father;
delete family.mother.husband;
```

|Result:|
|:---|
|![js-garbage-collection2](https://github.com/siriyaoff/MDN-note/blob/master/images/js-garbage-collection2.PNG?raw=true)|

- 위의 경우 `name: john`인 object가 garbage collect됨
- incoming reference만 object를 reachable하게 만들 수 있음

## Unreachable island
위의 예시에서 아예 global variable로부터의 reference를 끊으면(`family=null`) 모든 object가 unreachable하게 됨  
서로 incoming references가 존재하지만 reachable value로부터의 reference가 없기 때문에 unreachable함  
=> 참조되었다고 reachable한건 아님!!

## Internal algorithms
garbage collection은 "mark-and-sweep"이라는 algorithm을 기반으로 함:  
1. garbage collector가 roots를 찾고 mark해놓음(방문한 사실도 저장함)
2. mark된 객체들을 방문해서 그것들이 참조하는 객체들을 다시 mark
3. 모든 reachable references를 방문할 때까지 2를 반복
4. mark된 객체를 제외한 객체들을 없앰

bfs랑 비슷  
JS engine에서는 아래와 같은 최적화를 실행함:
- Generational collection  
	object를 new, old로 분류함  
	old들은 오랫동안 유지된 객체들이기 때문에 자주 조사하지 않음
- Incremental collection  
	전체를 한꺼번에 처리하는 대신, 조각으로 나눠서 조금씩 처리함  
	visible delay가 발생하지 않음
- Idle-time collection  
	CPU가 idle일 때만 garbage collector를 실행해서 실행 시간에 영향을 주지 않게 만듦

garbage collection에 관한 지식은 low-level optimization을 수행할 때 요구됨

# Object methods, "this"
object의 property로 선언된 function을 object의 *method*라고 함

## Method examples
```javascript
let user = {
  name: "John",
  age: 30
};

user.sayHi = function() {
  alert("Hello!");
};

user.sayHi(); // Hello!
```
- 위 예시에서는 `sayHi`가 `user`의 method임

아래와 같이 미리 선언된 함수를 method로 사용할 수도 있음:  
```javascript
let user = {
  // ...
};

// first, declare
function sayHi() {
  alert("Hello!");
};

// then add as a method
user.sayHi = sayHi;

user.sayHi(); // Hello!
```

> In Object-oriented programming, we use objects to represent entities

### Method shorthand
아래 두 가지 방법으로 method를 선언할 수 있음:  
```javascript
user = {
  sayHi: function() {
    alert("Hello");
  }
};

// method shorthand looks better, right?
user = {
  sayHi() { // same as "sayHi: function(){...}"
    alert("Hello");
  }
};
```

## "this" in methods
object에 접근하기 위해, method 내에서 `this` keyword를 이용함:  
```javascript
let user = {
  name: "John",
  age: 30,

  sayHi() {
    // "this" is the "current object"
    alert(this.name);
  }
};

user.sayHi(); // John
```

`this`를 사용하지 않고 위의 method 구현부에서 `alert(user.name);`으로 적어도 작동은 됨  
하지만 아래와 같이 `user`을 overwrite하면 문제가 생김:  
```javascript
let user = {
  name: "John",
  age: 30,

  sayHi() {
    alert( user.name ); // leads to an error
  }
};


let admin = user;
user = null; // overwrite to make things obvious

admin.sayHi(); // TypeError: Cannot read property 'name' of null
```

## "this" is not bound
`this`를 method가 아닌 함수에서도 사용 가능함  
=> run-time에 `this`의 값이 계산됨

#### Example
```javascript
let user = { name: "John" };
let admin = { name: "Admin" };

function sayHi() {
  alert( this.name );
}

// use the same function in two objects
user.f = sayHi;
admin.f = sayHi;

// these calls have different this
// "this" inside the function is the object "before the dot"
user.f(); // John  (this == user)
admin.f(); // Admin  (this == admin)

admin['f'](); // Admin (dot or square brackets access the method – doesn't matter)
```
- method도 square brackets와 string으로 호출할 수 있다는 것에 유의!
- `sayHi`를 method가 아닌 일반 함수로 호출하면(`sayHi();`):
	- strict mode에서는 `this`가 `undefined`로 바뀜  
		=> `this.name`을 접근할 때 에러가 남
	- non-strict mode에서는 `this`가 *global object*를 가리킴  
		strict mode로 고친 예전의 에러임
	
	보통 `this`를 사용하는 함수는 method로 사용되기 때문에 object를 통해서 호출되지 않으면 에러인 경우가 많음

> JS에서의 `this`의 취급  
> 다른 언어들에서는 보통 `this`를 method가 정의된 객체만을 참조함("bound this")  
> 하지만 JS에서는 `this`가 method가 정의된 객체만을 참조하지 않고, method를 호출한 객체를 참조함  
> 장점 : 재사용성 ↑, 단점 : 실수가 잦아질 수 있음

## Arrow functions have no "this"
arrow function에서 `this`를 사용하면, arrow function이 구현된 함수(outer normal function)를 호출한 객체를 참조함:  
```javascript
let user = {
  firstName: "Ilya",
  sayHi() {
    let arrow = () => alert(this.firstName);
    arrow();
  }
};

user.sayHi(); // Ilya


let usera = {
  firstName: "Ilya",
  userb: {
    firstName: "Ilya2",
    sayHi : () => alert(this.firstName)
  }
};

let userc={
  firstName: "Ilya3",
  sayHi: ()=> alert(this.firstName)
};

usera.userb.sayHi(); // undefined
userc.sayHi(); // undefined
```
- outer normal function 없이 바로 method로 사용되고 `this`를 사용할 경우 `this`는 `undefined`로 바뀜!!  
	위의 경우에도 `this`를 사용하지 않으면 arrow function을 바로 method로 사용할 수 있음

## Summary
- method에서 property를 참조하기 위해서는 `this`를 이용해야 함(`this.prop`)
- property만 있는 객체는 object literal을 리턴하는 것으로 간단하게 constructor를 구현 가능  
	(constructor function syntax를 사용하지 않고)  
	
	`this`를 사용해야 하는 경우 constructor function을 사용  
	(호출할 때 `new`와 같이 사용되기 때문에 일반 함수와 분명한 차이가 있음)
- arrow function으로 method를 구현할 때 `this`를 사용하면 arrow function을 사용한 method를 소유한 객체를 리턴함  
	※ 반드시 method안에서 arrow function을 function expression으로 사용해야 함!!  
	arrow function 자체를 method로 정의하면 `this`가 `undefined`를 반환함  
	이런 특징때문에 arrow function은 method를 구현할 때 사용하지 않는 것이 좋음!!

## Tasks
```javascript
function makeUser() {
  return {
    name: "John",
    ref: this
  };
}

let user = makeUser();

alert( user.ref.name ); // Error: Cannot read property 'name' of undefined
```
- `ref`에는 `undefined`가 저장됨!  
	∵ `this`의 값은 호출 시점에 결정되는데, `makeUser()`가 method가 아닌, 일반 함수로 호출되었기 때문에 전체 함수인 `undefined`가 들어감(code block과 object literal은 영향을 주지 않음)  
	따라서 `ref: this`는 현재 `this`의 값인 `undefined`가 저장되고, `user.ref`가 `undefined`가 됨
	
	아래 코드와 동치임:  
	```javascript
	function makeUser(){
	  return this; // 이번엔 객체 리터럴을 사용하지 않았습니다.
	}

	alert( makeUser().name ); // Error: Cannot read property 'name' of undefined
	```
	
	원래 의도대로 구현한 코드:  
	```javascript
	function makeUser() {
	  return {
		name: "John",
		ref() {
		  return this;
		}
	  };
	};

	let user = makeUser();

	alert( user.ref().name ); // John
	alert( user.ref().ref().name ); // John
	```
	- `user.ref()`가 method가 되기 때문에 `this`는 `.`앞의 객체로 선정됨

```javascript
let ladder = {
  step: 0,
  up() {
    this.step++;
    return this;
  },
  down() {
    this.step--;
    return this;
  },
  showStep() { // 사다리에서 몇 번째 단에 올라와 있는지 보여줌
    alert( this.step );
    return this;
  }
};

ladder.up();
ladder.up();
ladder.down();
ladder.showStep(); // 1

ladder.up().up().down().showStep(); // 2
```
- method의 리턴을 `this`로 설정해서 호출을 chainable하게 만들 수 있음

# Constructor, operator "new"
비슷한 객체를 많이 만들어야 하는 경우에는 object literal로 하기에 불필요한 반복이 많음  
=> constructor function과 `new` operator를 사용

## Constructor function
생성자 함수는 기능적으로는 보통의 함수와 같음  
생성자 함수의 규칙 2가지:
1. 대문자로 시작(common agreement)
2. `new` operator와 같이 사용되어야 함

#### Example
```javascript
function User(name) {
  // this = {};  (implicitly)

  // add properties to this
  this.name = name;
  this.isAdmin = false;

  // return this;  (implicitly)
}

let user = new User("Jack");

alert(user.name); // Jack
alert(user.isAdmin); // false
```
- 생성자 함수에서는 주석과 같이 `this`에 이미 객체가 선언되어 있다고 생각하면 됨(`undefined`가 리턴되지 않음!)
- 모든 함수들이 생성자로 사용될 수 있음 => `new`와 같이 사용되면 모든 함수들이 위와 같이 `this`를 리턴함

`let user=new function() { ... };`와 같이 사용하면 constructor는 재사용할 수 없는 대신 하나의 객체만을 생성하도록 코드를 encapsulate하는 효과가 있음

## Constructor mode test: new.target
`new.target` property를 사용하면 함수 내부에서 이 함수가 `new`와 함께 호출되었는지 알 수 있음  
- `new`와 함께 호출된 함수에서 사용하면 함수 자체(`function User() { ... }`)가 리턴됨
- `new`없이 호출된 함수에서 사용하면 `undefined` 리턴

아래와 같이 `new`가 있던 없던 constructor mode로 동작하게 구현할 수 있음:  
```javascript
function User(name) {
  if (!new.target) { // if you run me without new
    return new User(name); // ...I will add new for you
  }

  this.name = name;
}

let john = User("John"); // redirects call to new User
alert(john.name); // John
```
- 문법을 더 유연하게 만들지만, `new`를 생략해서 코드의 가독성을 떨어뜨릴 수 있음

## Return from constructors
보통 constructor는 `return` 문장이 없지만, 있을 경우 아래와 같이 처리됨:
- `return`이 object와 같이 사용되면 `this` 대신 그 object를 리턴
- `return`이 primitive와 같이 사용되거나 혼자 사용되면 무시됨

#### Example
```javascript
function BigUser() {

  this.name = "John";

  return { name: "Godzilla" };  // <-- returns this object
}

alert( new BigUser().name );  // Godzilla, got that object
```
- `new` operator가 `.`보다 우선순위가 높은듯

※ `new`를 사용하면 constructor의 괄호를 생략할 수 있음(인자가 없을 때)  
하지만 이것 또한 가독성을 떨어뜨리기 때문에 좋은 코딩 스타일이 아님

## Methods in constructor
method도 `this`를 사용해서 정의 가능함:  
```javascript
function User(name) {
  this.name = name;

  this.sayHi = function() {
    alert( "My name is: " + this.name );
  };
}

let john = new User("John");

john.sayHi(); // My name is: John
```
- class를 이용하면 더 복잡한 object를 생성할 수 있음(지금 다루는 object는 구조체 느낌인듯)

## Summary

|code|description|
|:---|:---|
|`let user = new User([param...]);`|Constructor function|
|`new.target`|현재 함수가 `new`와 함께 호출되었는지 판별|

- `new` operator가 `.`보다 우선순위가 높음  
	`new User().name`을 실행하면 만들어진 객체의 `name` property가 정상적으로 출력됨
- object literal으로 선언할 때는 `:`, constructor function에서 선언할 때는 일반 statement처럼 `=` 사용

## Tasks
- constructor로 method를 정의할 때는 무조건 `this.method= function expression;`을 사용해야 하는 듯(function declaration을 사용하면 method로 추가되지 않음)  
	아니면 function declaration으로 정의한 뒤에 `this.method=func_name;`으로 정의해도 됨

# Optional chaining `?.`
property를 호출하는 안전한 방법임(특히 A.B.C와 같이 중첩된 property 호출에서 B가 `null`일 수도 있을 때)

## The "non-existing property" problem
```javascript
let user = {}; // a user without "address" property

alert(user.address.street); // Error!
```
- 위의 상황처럼 `user`가 `address` property를 가지지 않음  
	=> `user.address`가 `undefined`이기 때문에 `user.address.street`을 호출하면 에러가 남

web development에서도 `document.querySelector('.elem')`함수를 사용할 때 해당하는 element가 없으면 `null`이 반환됨:  
```javascript
let html = document.querySelector('.elem').innerHTML; // error if it's null
```

### 해당 element가 업을 때 `html`에 `null`을 대입하려면?
`if`나 삼항연산자 `?`를 사용  
e.g. `alert(user.address ? user.address.street : undefined);`

중첩이 많아지면 코드에 불필요한 반복이 너무 많아짐  
=> `&&` 이용:  
`alert(user.address && user.address.street && user.address.street.name );`

AND'ing 또한 반복이 너무 많음  
이런 반복을 줄이기 위해서 optional chaining `?.`가 추가됨

## Optional chaining
`?.`는 왼쪽 대상이 `undefined`거나 `null`이면 계산을 멈추고 `undefined`를 리턴함  
아래에서는 편의를 위해 `null`이나 `undefined`가 아니면 객체가 존재한다고 가정함

즉, `value?.prop`는 아래와 같이 작동함:
- `value`가 존재한다면 `value.prop` 리턴
- `value`가 존재하지 않는다면(`undefined` 또는 `null`일 경우) `undefined` 리턴

위의 예제에 optional chaining을 적용하면 아래와 같음:  
```javascript
let user = {}; // user has no address

alert( user?.address?.street ); // undefined (no error)
```
- `user = null`일 경우에도 작동함(당연히 `undefined` 반환)

> optional chaining은 존재하지 않아도 괜찮은 대상에만 사용해야 함  
> 모든 대상에 사용하면 항상 존재해야 하는 대상이 실수로 정의되지 않아도 에러가 나지 않기 때문에 디버깅하기 어려워짐!

> `?.` 전의 값은 반드시 선언되어 있어야 함!  
> optional chaining은 선언된 변수에 대해서만 동작함

## Short-circuiting
`?.`는 왼쪽 대상이 존재하지 않으면 아예 멈춤  
=> 아래와 같이 함수나 증가식도 계산되지 않음:  
```javascript
let user = null;
let x = 0;

user?.sayHi(x++); // no "sayHi", so the execution doesn't reach x++

alert(x); // 0, value not incremented
```

## Other variants: `?.()`, `?.[]`
method를 호출할 때 `method()`대신 `method?.()`로 호출해서 optional chaining과 같은 효과를 낼 수 있음:  
```javascript
let userAdmin = {
  admin() {
    alert("I am admin");
  }
};

let userGuest = {};

userAdmin.admin?.(); // I am admin

userGuest.admin?.(); // nothing (no such method)
```

변수를 이용한 property 호출도 원래는 `obj[var]`과 같이 호출하지만 `obj?.[var]`로 optional chaining과 같은 효과를 낼 수 있음:  
```javascript
let key = "firstName";

let user1 = {
  firstName: "John"
};

let user2 = null;

alert( user1?.[key] ); // John
alert( user2?.[key] ); // undefined
```

`delete` operator도 `?.`과 함께 사용할 수 있음:  
```javascript
delete user?.name; // delete user.name if user exists
```

> `?.`를 이용해서 읽기, 삭제를 안전하게 수행할 수 있지만 수정은 안됨!  
> `user?.name = "john";`은 `user`가 존재하지 않으면 `undefined`에 RHS를 대입하는 꼴이 되기 때문에 어차피 에러남

## Summary

|code|description|
|:---|:---|
|`user?.address`|optional chaining<br>연산자 이전의 값은 반드시 선언되어 있어야 함|
|`obj.method?.()`|method에 대해 optional chaining 적용|
|`obj?.[var]`|computed property에 대해 optional chaining 적용|

# Symbol type
specification에 따르면, object property key는 `string` type이거나 `symbol` type임  

## Symbols
`Symbol()`을 사용해서 unique identifier을 만들 수 있음:  
```javascript
// id is a symbol with the description "id"
let id = Symbol("id");

let id1 = Symbol("id");

alert(id == id1); // false
```
- 같은 description을 써서 생성하더라도 둘은 다른 `symbol`들임

※ symbol들은 자동으로 `string`으로 변환되지 않음!!  
```javascript
let id = Symbol("id");
alert(id); // TypeError: Cannot convert a Symbol value to a string
alert(id.toString()); // Symbol(id), now it works
alert(id.description); // id
```
- `symbol`과 `string`은 근본적으로 다르기 때문에 `symbol`은 아예 다른 형으로 변환되는 것이 막혀있음
- `symbol`을 출력하기 위해서는 `toString()`을 이용하거나 description을 출력해야 함

## "Hidden" properties
`symbol`을 key로 이용해서 숨겨진 property를 생성할 수 있음  
=> `symbol`을 이용하지 않고는 접근할 수 없음

#### Example
```javascript
let user = { // belongs to another code
  name: "John"
};

let id = Symbol("id");

user[id] = 1;

alert( user[id] ); // we can access the data using the symbol as the key
```
- `symbol`은 항상 다르기 때문에, 다른 곳에서 똑같은 `id`를 `symbol`로 사용하더라도 `user[id]`는 서로 다른 properties가 됨  
	cf. `"id"`를 property name으로 사용할 경우 충돌이 일어남

## Symbols in an object literal
square brackets `[]`를 이용해서 object literal에서도 `symbol`을 사용할 수 있음:  
```javascript
let id = Symbol("id");

let user = {
  name: "John",
  [id]: 123 // not "id": 123
};
```

## Symbols are skipped by for...in
`symbol`로 선언된 property는 심지어 `for...in` 반복문에서도 배제됨:  
```javascript
let id = Symbol("id");
let user = {
  name: "John",
  age: 30,
  [id]: 123
};

for (let key in user) alert(key); // name, age (no symbols)

// the direct access by the symbol works
alert( "Direct: " + user[id] );
```

`object.assign`을 이용해서 object를 복제할 때는 symbol properties도 복사됨

## Global symbols
global symbol을 이용해서 다른 곳에서 같은 `symbol`을 사용할 수 있음  
`Symbol.for(key)`를 사용해서 *global symbol registry*에 *global symbol*을 등록하거나 등록되어 있는 것을 읽음
- global symbol registry에 `key`에 해당하는 global symbol이 등록되어 있지 않으면 생성, 등록하고 반환
- 등록되어 있는 경우 해당 global symbol을 반환함

```javascript
// read from the global registry
let id = Symbol.for("id"); // if the symbol did not exist, it is created

// read it again (maybe from another part of the code)
let idAgain = Symbol.for("id");

// the same symbol
alert( id === idAgain ); // true
```

> Ruby에서의 symbol이 JS에서의 global symbol과 비슷함

### `Symbol.keyFor`
`Symbol.keyFor(sym)`을 사용해서 global symbol 값에 해당하는 `key`를 찾을 수 있음  
```javascript
let globalSymbol = Symbol.for("name");
let localSymbol = Symbol("name");

alert( Symbol.keyFor(globalSymbol) ); // name, global symbol
alert( Symbol.keyFor(localSymbol) ); // undefined, not global

alert( localSymbol.description ); // name
alert( globalSymbol.description ); // name
```
- `Symbol.keyFor(sym)`은 global symbol registry에서 `sym`에 해당하는 key를 찾아서 반환함  
	=> non-global symbol에 대해서는 key값을 찾을 수 없기 때문에 `undefined` 반환
- 모든 `symbol`들은 `description` property를 가짐!  
	global symbol도 key가 description이기 때문에 `.description`으로 출력 가능함

## System symbols
system symbols가 존재함:
- Symbol.hasInstance
- Symbol.isConcatSpreadable
- Symbol.iterator
- Symbol.toPrimitive
	- object to primitive conversion에 필요함

## Summary

|code|description|
|:---|:---|
|`let id = Symbol("description");`|symbol 생성|
|`sym.description`|symbol의 `"description"` 반환|
|`let id = Symbol.for("description");`|global symbol 생성|
|`Symbol.keyFor(sym)`|global symbol의 key(`"description"`) 반환|

- symbolic property는 `for...in`에서도 배제됨
- global symbol은 `sym.description`을 사용하든 `Symbol.keyFor()`을 사용하든 생성될 때 사용한 description을 리턴함
- `Object.getOwnPropertySymbols(obj)`를 사용하면 properties 뿐만 아니라 symbol들까지 알 수 있음  
- `Reflect.ownKeys(obj)`를 사용하면 symbolic properties를 포함해서 모든 properties의 key를 알 수 있음

# Object to primitive conversion
`obj1 + obj2` 등 객체끼리 연산될 때는 자동으로 primitives로 변환됨:
1. 모든 객체는 boolean으로 변환될 때 `true`로 취급됨 => 객체를 변환할 때는 numeric, string으로의 변환만 사용함
2. 객체끼리 뺄셈이나 수학적 함수를 적용할 때 numeric conversion이 일어남  
	e.g. `Date` 객체의 연산 : `date1 - date2`
3. string conversion은 보통 `alert(obj)`와 같이 객체를 출력할 때 일어남

## ToPrimitive
`"string"`, `"number"`, `"default"` 3개의 hint를 이용해서 객체의 변환을 조절할 수 있음  
hint는 목표로 하는 자료형으로 이해하면 됨

### `"string"`
`alert()`와 같이 문자열이 들어가는 연산을 수행하면 hint가 string이 됨:  
```javascript
// output
alert(obj);

// using object as a property key
anotherObj[obj] = 123;
```

### `"number"`
계산할 때는 hint가 number가 됨:  
```javascript
// explicit conversion
let num = Number(obj);

// maths (except binary plus)
let n = +obj; // unary plus
let delta = date1 - date2;

// less/greater comparison
let greater = user1 > user2;
```

### `"default"`
`+`, `==`와 같이 숫자, 문자 모두에서 사용되는 연산자에서는 hint가 default가 됨:  
```javascript
// binary plus uses the "default" hint
let total = obj1 + obj2;

// obj == number uses the "default" hint
if (user == 1) { ... };
```
- `<`, `>`와 같은 비교 연산자들도 숫자, 문자 모두 피연산자로 사용 가능하지만 hint가 number로 됨  
	`Date` 객체를 제외하면 대부분의 내장된 객체에서 default도 number처럼 동작하기 때문에 다 외울 필요는 없음

※ hint는 3개만 존재함!(number, string, default)  
boolean은 hint가 아님!!

conversion을 할 때 JS는 아래 순서대로 작동함:
1. 객체에 `obj[Symbol.toPrimitive](hint)` method를 찾고, 있으면 호출함  
	`Symbol.toPrimitive`는 system symbol으로, 따로 만드는게 아님
2. 위의 경우에 해당하지 않고 hint가 string일 때  
	`obj.toString()`, `obj.valueOf()` 순서대로 찾으면서 존재하는 것을 실행함
3. 위의 경우에 해당하지 않고 hint가 number 또는 default일 때  
	`obj.valueOf()`, `obj.toString()` 순서대로 찾으면서 존재하는 것을 실행함  

## Symbol.toPrimitive
`Symbol.toPrimitive`는 내장 symbol로, conversion method로 사용됨

#### Example
```javascript
let user = {
  name: "John",
  money: 1000,

  [Symbol.toPrimitive](hint) {
    alert(`hint: ${hint}`);
    return hint == "string" ? `{name: "${this.name}"}` : this.money;
  }
};

// conversions demo:
alert(user); // hint: string -> {name: "John"}
alert(+user); // hint: number -> 1000
alert(user + 500); // hint: default -> 1500
```

## toString/valueOf
위의 symbolic key와 다르게 string-named method임  
반드시 primitive value를 리턴해야 함  
(`object`를 리턴할 경우 아예 무시됨)

각 함수의 default:
- `toString`은 `"[object Object]"`를 리턴함
- `valueOf`는 object 자체를 리턴함

```javascript
let user = {name: "John"};

alert(user); // [object Object]
alert(user.valueOf() === user); // true
```

- 재정의하고 사용하기 때문에 큰 의미는 없음

#### Example
```javascript
let user = {
  name: "John",
  money: 1000,

  // for hint="string"
  toString() {
    return `{name: "${this.name}"}`;
  },

  // for hint="number" or "default"
  valueOf() {
    return this.money;
  }

};
```
- `[Symbol.toPrimitive](hint)`와 동일한 기능임
- 객체를 변환하는 순서에 따라서, `Symbol.toPrimitive`와 `valueOf`가 없으면 `toString`이 모든 변환을 담당함  
	

## Return types
primitive-conversion method들이 반드시 hint와 같은 primitive를 리턴할 필요는 없음  
primtivie만 리턴하면 됨

> `toString`과 `valueOf`는 객체를 리턴해도 에러가 나지 않고 method가 존재하지 않는 것처럼 무시됨  
> 예전에는 JS에 에러에 관한 명확한 정의가 없었기 때문  
> 반면, `Symbol.toPrimitive`는 primitive를 리턴하지 않으면 에러남

## Further conversions
객체가 인자로 사용될 때 두 단계에 거쳐서 conversion이 일어남:
1. 위 규칙에 따라서 객체가 primitive로 변환됨
2. 변환된 primitive가 적절한 type이 아니면 변환됨

#### Example
```javascript
let obj = {
  // toString handles all conversions in the absence of other methods
  toString() {
    return "2";
  }
};

alert(obj * 2); // 4
alert(obj + 2); // 22
```

## Summary

|code|description|
|:---|:---|
|`[Symbol.toPrimitive](hint)`|conversion method|
|`toString()`|conversion method|
|`valueOf()`|conversion method|

- 실무에서는 `obj.toString()`가 모든 변환을 담당하는 "catch-all" method로 구현되는 경우가 많음
- 호환성을 위해서 `toString()`, `valueOf()`을 `return this[Symbol.toPrimitive]('string or number')`과 같이 구현할 수 있음
