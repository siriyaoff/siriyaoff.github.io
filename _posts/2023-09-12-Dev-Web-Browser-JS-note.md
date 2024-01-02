---
layout: single
title: "웹브라우저 JS"
categories:
  - Dev
tags:
  - JavaScript
  - element
  - jQuery
  - node
  - event
  - Event bubbling
  - Event capturing
  - stopPropagation()
  - Ajax
  - XMLHttpRequest
  - jQuery Ajax
---

- 식별자 API
  - `element.tagName` : 태그 이름 리턴
  - `element.id` : id 리턴
  - `element.className` : 클래스 이름들 리턴
  - `element.classList` : 클래스 이름들 배열로 리턴
- 조회 API
  - `element.getElementsByClassName` : 클래스 이름을 바탕으로 `element` 하위 element들 조회
- 속성 제어 API
  - `element.getAttribute(name)`
  - `element.setAttribute(name, value)`
  - `element.hasAttribute(name)`
  - `element.removeAttribute(name)`
- jQuery에서의 속성 제어
  ```jsx
  var t = $('#target');
  t.attr('title', 'title1');
  console.log(t.attr('title');
  t.removeAttr('title');
  ```
  - `t.attr()` : 속성이나 정적인 정보 관련
  - `t.prop()` : 동적인 상태 관련
  - `t.find()`는 chaining 가능

## Node

- 모든 DOM 객체는 Node 객체를 상속받음
- 관련 필드, 메소드들
  - `node.childNodes`
  - `node.firstChild`
  - `node.lastChild`
  - `node.nextSibling`
  - `node.previousSibling`
  - `node.contains()`
  - `node.hasChildNodes()`
  - `node.nodeType`
    - `ELEMENT_NODE`, `ATTRIBUTE_NODE`, `TEXT_NODE`, …
  - `node.nodeName` : node의 태그명
  - `node.nodeValue`
  - `node.textContent`
  - `node.appendChild()`
  - `node.removeChild()`
  - `node.replaceChild()`
- jQuery 활용
  - `t.before()`
  - `t.prepend()`
  - `t.append()`
  - `t.after()`
  - `t.remove()`
  - `t.empty()` : text node 제거
  - `t.replaceAll()`
  - `t.replaceWith()`
  - `t.clone()`
- 문자열 관련
  - `element.innerHTML` : string 형태로 자식 노드를 읽을 수 있음
  - `element.outerHTML` : 본인 포함
  - `element.innerText`, `outerText` : 값을 읽을 때는 HTML 코드 제외한 나머지 리턴, 값을 대입할 때는 HTML 코드 그대로 추가

## Event

- event target : 이벤트가 일어날 객체
- event type : 이벤트의 종류(click, scroll, mousemove, …)
- event 등록 방법
  - inline : `onClick` 속성에 직접 등록
  - property listener : element 뒤에 script(이벤트 객체 지정, `t.onclick=...`과 같이 구현)
  - `addEventListener(event, handler)`
- Event bubbling : 중복된 태그에 대해서 이벤트가 발생했을 때 하위 원소에서 상위 원소로 이벤트가 전파됨
- Event capturing : 이벤트가 상위 원소에서 하위 원소로 전파됨
  - `addEventListener`의 3번째 파라미터에 capture option을 `true`로 설정
- 두 현상 모두 event 등록할 때 아래처럼 `stopPropagation()` 등록해서 방지 가능
  ```jsx
  const handler(event){
  	event.stopPropagation();
  	console.log("real event");
  }
  ```
- element의 기본 동작을 취소할 수 있음
  - 기본 동작 : submit 누르면 input 데이터 제출 등
  - handler에서 `false`를 리턴하거나 위 `stopPropagation`처럼 `event.preventDefault()`를 호출해서 설정 가능

### Event type

- 폼 태그 관련
  - submit, change, focus, blur(element가 focus 해제되었을 때)
- 로드 관련
  - load, DOMContentLoaded(DOM 요소만 로드되면 실행되기 때문에 이미지까지 모든 자원 로딩을 기다리지 않아도 됨)
- 마우스 관련
  - click, dbclick, mousedown, mouseup(마우스 뗄 때), mousemove(호버링), mouseout, contextmenu
  - `event.shiftKey`, `event.altKey`, `event.ctrlKey` 프로퍼티로 키보드 조합 탐지 가능
  - `clientX`, `clientY`로 포인터 위치 탐지 가능
- jQuery 이벤트 : `.on(event [, selector] [, data], handler(eventObject)`
  - `event` : 이벤트 타입
  - `selector` : 이벤트를 발생시킬 요소 선택
  - `data` : `handler`로 전달될 데이터
  - `handler` : 이벤트 핸들러
  - jQuery를 사용하면 존재하지 않는 element에 대해서도 이벤트를 등록할 수 있음
  - 다중 바인딩 지원(`.on({event:handler,})`)
  - 이벤트 제거 : `.off(event, handler)`

## 네트워크 통신

- Ajax(Asynchronous JavaScript and XML) : JS를 사용해서 비동기적으로 서버와 브라우저가 데이터를 주고 받는 방식

### XMLHttpRequest

```jsx
// GET
var xhr = new XMLHttpRequest();
xhr.open("GET", "./time.php");
xhr.onreadystatechange = function () {
  if (xhr.readyState === 4 && xhr.status == 200) {
    document.querySelector("#time").innerHTML = xhr.responseText;
  }
};
xhr.send();

// POST
var xhr = new XMLHttpRequest();
xhr.open("POST", "./time2.php");
xhr.onreadystatechange = function () {
  document.querySelector("#time").innerHTML = xhr.responseText;
};
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
var data = "";
data = "key=value&key1=value1...";
xhr.send(data);
```

- `xhr.open()` : 요청 설정
- `xhr.onreadystatechange()` : callback 함수 설정
- `xhr.setRequestHeader()` : 요청 헤더 설정(보낼 데이터 타입 등)
- `xhr.send()` : 서버로 요청 보냄

### jQuery Ajax

- jQuery Ajax는 크로스브라우징 문제를 자동으로 해결해줌
- `$.ajax([settings])`
  - `settings` : 옵션을 담고 있는 객체
  - `data` : 서버로 전송할 데이터
  - `dataType` : datatype
  - `success` : 요청 성공했을 때의 callback
  - `type` : 요청 type
