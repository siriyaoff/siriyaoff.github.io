---
layout: single
title: "JSnote7: Object properties configuration"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - JavaScript
  - property flags
  - descriptor
  - accessor property
  - getter and setter
---

Object properties configuration
# Property flags and descriptors
property는 "key-value" pair 이상의 가치를 가짐  
additional configuration options, getter, setter function이 존재함

## Property flags
object property는 `value` 이외에도 3개의 attribute를 가짐(flag라고 함)
- `writable` : `true`일 경우 값이 바뀔 수 있음
- `enumerable` : `true`일 경우 반복문으로 property를 순회할 때 나열될 수 있음
- `configurable` : `true`일 경우 property가 삭제되거나 attributes가 수정될 수 있음

property를 일반적인 방식으로 선언하면 모든 flag가 `true`로 설정됨  
`Object.getOwnPropertyDescriptor`를 사용하면 모든 flag의 상태를 알 수 있음:  
```javascript
let descriptor = Object.getOwnPropertyDescriptor(obj, propertyName);

/*-------------example--------------*/

let user = {
  name: "John"
};

let descriptor = Object.getOwnPropertyDescriptor(user, 'name');

alert( JSON.stringify(descriptor, null, 2 ) );
/* property descriptor:
{
  "value": "John",
  "writable": true,
  "enumerable": true,
  "configurable": true
}
*/
```
- parameters
	- `obj` : property를 소유한 객체
	- `propertyName` : property의 이름
- "property descriptor"라는 `Object`를 반환함  
	value, flags를 property로 가짐

`Object.defineProperty`로 flag를 수정 가능:  
```javascript
Object.defineProperty(obj, propertyName, descriptor)

/*-------------example--------------*/

let user = {};

Object.defineProperty(user, "name", {
  value: "John"
});

let descriptor = Object.getOwnPropertyDescriptor(user, 'name');

alert( JSON.stringify(descriptor, null, 2 ) );
/*
{
  "value": "John",
  "writable": false,
  "enumerable": false,
  "configurable": false
}
*/
```
- parameters
	- `obj`, `propertyName` : 정의하거나 바꾸고 싶은 property의 정보
	- `descriptor` : 적용할 property descriptor
- property가 존재한다면 `descriptor`에 명시된 flag를 갱신함  
	존재하지 않는다면 property를 새로 만듦  
	(이때 flag를 생략하면 `false`로 초기화됨)

## Non-writable
`writable` flag가 `false`일 경우:  
```javascript
let user = {
  name: "John"
};
Object.defineProperty(user, "name", {
  writable: false
});

user.name = "Pete"; // Error: Cannot assign to read only property 'name'
```
- `user.name`이 `"Pete"`로 바뀌지 않음  
	∵ non-writable하기 때문

> **non-writable한 property를 수정할 때 에러는 strict mode에서만 발생함**  
> non-strict mode에서 flag에 위반하는 코드는 그냥 무시되고 에러를 발생시키진 않음

## Non-enumerable
객체 안에 이미 선언된 `toString()`은 non-enumerable이지만, 직접 추가하면 다른 method와 같이 `for...in`에 나열됨:  
```javascript
let user = {
  name: "John",
  toString() {
    return this.name;
  }
};

for (let key in user) alert(key); // name, toString
alert(Object.keys(user)); // name,toString

Object.defineProperty(user, "toString", {
  enumerable: false
});

for (let key in user) alert(key); // name
alert(Object.keys(user)); // name
```
- `user`의 `toString`은 직접 추가한 것이기 때문에 나열됨
- `enumerable`을 `false`로 바꾸면 더는 나열되지 않음
	- `Object.keys()`에도 나열되지 않음!

## Non-configurable
`configurable`이 `false`면 property를 삭제하거나 flag를 수정할 수 없음  
e.g. `Math.PI`는 non-writable, non-enumerable, non-configurable임:  
```javascript
let descriptor = Object.getOwnPropertyDescriptor(Math, 'PI');

alert( JSON.stringify(descriptor, null, 2 ) );
/*
{
  "value": 3.141592653589793,
  "writable": false,
  "enumerable": false,
  "configurable": false
}
*/

Math.PI = 3; // Error
```
- property를 non-configurable로 만드는 것은 비가역적임  
	`defineProperty`로도 수정이 제한됨:
	- `configurable`을 바꿀 수 없음
	- `enumerable`을 바꿀 수 없음
	- `writable`을 `false`에서 `true`로 바꿀 수 없음(`false` -> `true`는 됨)
	- `get/set`을 바꿀 수 없음(없을 때 추가는 됨)
- `configurable: false`는 flag의 수정과 property 삭제를 제한하고 값을 바꾸는 것만 허용하기 위한 목적으로 사용함
- `writable`과 `configurable`을 `false`로 만들면 상수와 같게 됨

## Object.defineProperties
```javascript
Object.defineProperties(obj, {
  prop1: descriptor1,
  prop2: descriptor2
  // ...
});

/*-------------example--------------*/

Object.defineProperties(user, {
  name: { value: "John", writable: false },
  surname: { value: "Smith", writable: false },
  // ...
});
```
- 여러 property의 flag를 한 번에 설정 가능

## Object.getOwnPropertyDescriptors
`Object.getOwnPropertyDescriptors`와 `Object.defineProperties`를 이용하면 flag를 포함한 object cloning이 가능함:  
```javascript
let clone = Object.defineProperties({}, Object.getOwnPropertyDescriptors(obj));

for (let key in user) {
  clone[key] = user[key]
}
```
- `for...in`은 symbolic property를 무시하지만, `getOwnPropertyDescriptors`는 symbolic을 포함한 모든 property의 descriptor를 반환함

## Sealing an object globally
object에 대한 접근을 제한할 수 있는 method들이 있음:
- `Object.preventExtensions(obj)` : `obj`에 새로운 property가 추가되는 것이 제한됨
- `Object.seal(obj)` : property 추가/삭제가 제한, 모든 property에 대해 `configurable: false`가 적용됨
- `Object.freeze(obj)` : property 추가/삭제/수정이 제한, 모든 property에 대해 `configurable: false, writable: false`가 적용됨
- `Object.isExtensible(obj)` : 새로운 property 추가가 제한되었는지 판별
- `Object.isSealed(obj)` : property 추가/삭제가 제한, 모든 property에 `configurable: false`가 적용되었는지 판별
- `Object.isFrozen(obj)` : property 추가/삭제/수정이 제한, 모든 property에 `configurable: false, writable: false`가 적용되었는지 판별

사실 잘 사용되지 않음

## Summary

| code                                                   | description                                                                                                        |
| :----------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------- |
| `Object.getOwnPropertyDescriptor(obj, propertyName)`   | `obj`의 `propertyName`의 descriptor를 리턴                                                                         |
| `Object.defineProperty(obj, propertyName, descriptor)` | `obj`의 `propertyName`에 `descriptor`를 적용함<br>`obj` 리턴                                                       |
| `Object.defineProperties(obj, descriptors)`            | `obj`의 properties에 descriptors 적용<br>`obj` 리턴                                                                |
| `Object.getOwnPropertyDescriptors(obj)`                | `obj`의 모든 property의 descriptor 리턴                                                                            |
| `Object.preventExtensions(obj)`                        | `obj`에 새로운 property가 추가되는 것을 제한함                                                                     |
| `Object.seal(obj)`                                     | property 추가/삭제가 제한, 모든 property에 대해 `configurable: false`가 적용됨<br>`obj` 리턴                       |
| `Object.freeze(obj)`                                   | property 추가/삭제/수정이 제한, 모든 property에 대해 `configurable: false, writable: false`가 적용됨<br>`obj` 리턴 |
| `Object.isExtensible(obj)`                             | 새로운 property 추가가 제한되었는지 판별                                                                           |
| `Object.isSealed(obj)`                                 | property 추가/삭제가 제한, 모든 property에 `configurable: false`가 적용되었는지 판별                               |
| `Object.isFrozen(obj)`                                 | property 추가/삭제/수정이 제한, 모든 property에 `configurable: false, writable: false`가 적용되었는지 판별         |

- `writable`, `enumerable`, `configurable` 총 3개의 flag가 존재함

# Property getters and setters
object property에는 두 가지 종류가 있음
- *data properties* : 이때까지 다뤘던 properties가 모두 data property임
- *accessor properties* : value가 바뀌거나 참조될 때 실행되는 함수지만, 외부에서 볼 때는 보통의 property로 취급됨

## Getters and setters
accessor property는 "getter", "setter" method로 표현됨  
object literal에서는 `get`, `set`으로 나타냄:  
```javascript
let obj = {
  get propName() {
    // getter, the code executed on getting obj.propName
  },

  set propName(value) {
    // setter, the code executed on setting obj.propName = value
  }
};

/*-------------example--------------*/

let user = {
  name: "John",
  surname: "Smith",

  get fullName() {
    return `${this.name} ${this.surname}`;
  },

  set fullName(value) {
    [this.name, this.surname] = value.split(" ");
  }
};

alert(user.fullName); // John Smith

user.fullName = "Alice Cooper";

alert(user.name); // Alice
alert(user.surname); // Cooper
```
- getter는 `obj.propName`이 읽어질 때 실행됨  
	setter는 `propName`에 값이 대입될 때 실행됨
- `fullName`이라는 가상의 property가 생김

## Accessor descriptors
accessor property의 descriptor에는 `value`와 `writable` 대신 `get`과 `set`이 들어감
- `get` : property가 읽어질 때 실행되는 함수
- `set` : property에 값이 대입될 때 실행되는 함수
- `enumerable` : data property의 그것과 같음
- `configurable` : data property의 그것과 같음

**Example**  
```javascript
let user = {
  name: "John",
  surname: "Smith"
};

Object.defineProperty(user, 'fullName', {
  get() {
    return `${this.name} ${this.surname}`;
  },

  set(value) {
    [this.name, this.surname] = value.split(" ");
  }
});

alert(user.fullName); // John Smith

for(let key in user) alert(key); // name, surname
```
- `get/set`과 `value/writable`은 같이 사용될 수 없음!  
	같이 사용하려 하면 에러남!  
	e.g.  
	```javascript
	// Error: Invalid property descriptor.
	Object.defineProperty({}, 'prop', {
	  get() {
		return 1
	  },

	  value: 2
	});
	```

## Smarter getters/setters
getter와 setter는 wrapper로 사용될 수 있음:  
```javascript
let user = {
  get name() {
    return this._name;
  },

  set name(value) {
    if (value.length < 4) {
      alert("Name is too short, need at least 4 characters");
      return;
    }
    this._name = value;
  }
};

user.name = "Pete";
alert(user.name); // Pete

user.name = ""; // Name is too short...
```
- `name`에 값이 대입되면 accessor property를 거쳐서 `_name`에 저장됨
- setter에서 간단한 filter를 적용할 수 있음  
	위 예시의 경우 글자 개수가 4 이하면 다시 입력하게 만듦
- `_name`에 접근하면 accessor property를 거치지 않고 입력이 가능하지만, 보통 `_`가 붙은 property는 내부에서 사용하는 property로 약속되어 있으므로 외부에서 접근하면 안됨

## Using for compatibility
기존에 `name`, `age` property를 선언하는 constructor `user`에서 `age` 대신 `birthday`를 사용해야 한다고 가정하자:  
```javascript
function User(name, birthday) {
  this.name = name;
  this.birthday = birthday;

  Object.defineProperty(this, "age", {
    get() {
      let todayYear = new Date().getFullYear();
      return todayYear - this.birthday.getFullYear();
    }
  });
}

let john = new User("John", new Date(1992, 6, 1));

alert( john.birthday ); // birthday is available
alert( john.age );      // ...as well as the age
```
- `age`를 accessor property로 만들어서 getter를 이용해 `birthday`를 계산해서 출력할 수 있음  
	=> 이전의 `age` property를 사용하는 코드에서 사용할 수 있도록 `user`를 구현 가능

## Summary
- getter `get`와 setter `set`를 사용해서 accessor property를 만들 수 있음  
	wrapper같은 느낌으로, accessor property가 호출되거나 값이 대입될 때 사용될 값을 직접 설정할 수 있음
	- `get`, `set`, `enumerable`, `configurable` attribute를 가짐

