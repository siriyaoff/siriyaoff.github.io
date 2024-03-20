---
layout: single
title: "FMETP Control flow with WebSocket on Express.js"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - Unity
  - C#
  - FMETP
  - Express.js
  - WebSocket
  - WebSocket Protocol
  - uuid
  - Array
---

## App configuration

- unity 2021.3.6f1
- Test server : `Assets/FMETP_STREAM/FMWebSocket/TestServer_v2.2.4`
  - express 서버
  - FMWebSocket과 통신하기 위해서 Socket.io를 사용하지 않고 `ws` 모듈 사용함
- Unity app  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Ctrl-flow-1.png)
  - FaceCamera 객체 내부의 OnDataByteReadyEvent에서 웹캠 피드를 전송할 방식 선택 가능
    - `Assets/FMETP_STREAM/FMWebSocket/Scripts/WebSocket/FMWebSocketManager`에 전송 방식이 정의되어 있음
      - SendToAll, SendToServer, SendToOthers, SendToTarget  
        ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Ctrl-flow-2.png)  
        ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Ctrl-flow-3.png)
    - `_meta[]`도 아래의 `_byteReg[]`와 같이 `ws.send()`로 보내는 argument임

## WebSocket Base Framing Protocol

- socket.io, FMWebSocket 모두 [WebSocket Protocol RFC 6455](https://datatracker.ietf.org/doc/rfc6455/) 프로토콜을 사용함

![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Ctrl-flow-4.png)

### Express 서버의 `index.html`에서의 예시

```jsx
ws.onmessage = function (evt) {
  FMHTML_SetElementById("StatusTextConnection", "Status" + (isServer ? "(server)" : "(client)") + ": " + new Date().getHours() + ":" + new Date().getMinutes() + ":" + new Date().getSeconds());
  FMHTML_SetElementById("BtnServerText", "Disconnect");
  FMHTML_SetElementById("BtnClientText", "Disconnect");

  var data = evt.data;

  if(typeof evt.data === "string")
  {
    //console.log('string data!');
  }

  if(evt.data instanceof ArrayBuffer)
  {
    var _byteRaw = new Uint8Array(evt.data);

    // FMETP: first byte defines the whole data type..., 0 is raw, 1 is string
    if(_byteRaw[0] === 0)
    {
      var _byteData = _byteRaw.slice(4, _byteRaw.length);
      // document.getElementById("StatusTextBytes").innerHTML = "(byte)" + _byteData.length;
      FMHTML_SetElementById("StatusTextBytes", "(byte)" + _byteData.length);

      if(_byteData.length > 18)
      {
        label_img = FMHTML_GetElementById("LabelVideo", label_img);
        label_aud = FMHTML_GetElementById("LabelAudio", label_aud);

        var _label = ByteToInt16(_byteData, 0);
        // console.log(_byteData.length + ': ' + _label); //Debug label

        if (_label == label_img)
        {
          ...
				}
      }
    }
    if(_byteRaw[0] === 1)
    {
      var _byteData = _byteRaw.slice(4, _byteRaw.length);
      var stringData = '';
      //----conver byte[] to Base64 string----
      var len = _byteData.byteLength;
      for (var i = 0; i < len; i++)
      {
          stringData += String.fromCharCode(_byteData[i]);
      }
      //----conver byte[] to Base64 string----
      document.getElementById("StatusTextString").innerHTML = "(string)" + stringData;
    }
  }
};
```

- payload 길이에 따라서 보낼 ArrayBuffer의 크기가 정해짐
- `_byteRaw[0]`
  - `0` : raw byte
  - `1` : string
- `_byteRaw[1]`
  - `[0, 1, 2, 3]` : To [All, Server, Others, Target]
- `_byteRaw[3]`
  - `3` : Register as server
  - `4` : Register as client

## Express server에서의 room 구조 구현

- socket.io에는 room이 지원되지만, ws에는 없기 때문에 직접 구현해야 함

### FMETP_STREAM에서의 전송 방식 분석

- 처음 접속할 때 접속 방식(server, client)을 설정함  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Ctrl-flow-5.png)  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Ctrl-flow-6.png)
  - 특별히 다른 기능을 하진 않고, 전송 방식을 위해서 구별된 것 같음(all/server/others)
    - FMWebSocket에서 말하는 server를 room 개념으로 이용할 것임
- 각 소켓의 uuid는 아래와 같은 방법으로 생성됨
  ```jsx
  function uuidv4() {
    function s4() {
      return Math.floor((1 + Math.random()) * 0x10000)
        .toString(16)
        .substring(1);
    }
    return s4() + s4() + "-" + s4();
  }
  ```
  - `'-'`을 제외하면 다 합쳐서 12 Bytes임을 알 수 있음

### Express 서버에서의 소켓 저장 구조 변경

- 기존 `index.js`에서의 구현 방법

  ```jsx
  ws = new WS_MODULE.Server({ server });
  ...

  const clients = new Map();

  ...

  ws.on("connection", function connection(ws) {
    const wsid = uuidv4();
    const networkType = "undefined";
    const metadata = { ws, networkType, wsid };
  ```

  - server, client 구별 없이 모든 소켓들을 하나의 `Map`인 `clients`에 저장함
  - 우선 생성 후 `networkType`은 첫 번째 메시지에 따라 나중에 부여함(바꾼 코드에도 그대로 적용됨)

- 저장 방식 수정 후:

  ```jsx
  const servers = new Map(); // (server's uuid, clients' uuid)
  const allwss = new Map(); // (uuid, metadata)

  ...

  ws.on("connection", function connection(ws) {
    const wsid = uuidv4();
    const networkType = "undefined";
    const serverid = "undefined";
    const metadata = { ws, networkType, wsid, serverid };
  ```

  - `servers` : (server uid, client uid의 Array)로 서버-클라이언트 연결 정보를 저장함
  - `allwss` : 이전 코드의 `clients`와 동일하게 wsid와 소켓 정보를 매핑하지만, 소켓 정보에 serverid가 추가됨
    - server일 경우 `"undefined"`가 저장됨

- 연결 구조에 맞게 아래 이벤트들도 변경함
  - server 소켓의 close : client들을 먼저 다 닫은 뒤 `ws.close()` 실행
  - client 소켓의 close : `servers.get(serveruid)`에서 wsid 제거하도록 추가
  - server, client 소켓의 message : 전송 방식에 맞게 처리

### Payload 수정

- 홈페이지의 connect as server/client 버튼을 누르면 `FMWebSocketConnect` 메소드가 실행됨
  - `FMWebSocketConnect`가 아래와 같이 EventListener를 추가함
    ```jsx
    ws.addEventListener("open", (event) => {
      RegisterNetworkType();
      console.log("**connected to server");
    });
    ```

```jsx
function RegisterNetworkType() {
  isServer = _regServer;
  var _byteReg = new Uint8Array(16);

  _byteReg[2] = 9;
  if (_regServer) {
    _byteReg[3] = 3;
    console.log("Register as Server");
  } else {
    // client
    var roomid = document.getElementById("RoomNumber").value; // string now
    _byteReg[3] = 4;
    console.log("Register as Client");
    for (let i = 4; i < 12; i++) {
      _byteReg[i] = parseInt(roomid[i - 4], 16);
    }
    for (let i = 13; i < 17; i++) {
      _byteReg[i - 1] = parseInt(roomid[i - 4], 16);
    }
  }
  ws.send(_byteReg);
}
```

- 원래는 `Uint8Array(4)`로 선언되었지만, server의 uuid도 포함시키기 위해 `16`으로 수정함
  - 프로토콜과 관련된 내용은 그대로 두고, client로 연결할 경우 server의 uuid를 사용자로부터 입력받아서 `'-'`을 뺀 나머지 id를 payload에 추가함

### 수정한 payload 처리

- `index.js`에서는 소켓으로부터 메시지를 받았을 때 아래와 같이 처리함

```jsx
ws.on("message", function incoming(message) {
  if (message.length === 16) {
    if (
      message[0] === 0 &&
      message[1] === 0 &&
      message[2] === 9 &&
      message[3] === 3 // SERVER
    ) {
      servers.set(wsid, new Array());
      serverID = wsid;
      serverWS = ws;
      console.log("regServer: " + wsid + "[Server] " + serverID);
      allwss.get(wsid).networkType = "server";

			// create invitation link
      ws.send(
        "http://" +
          ip.address() +
          "/join/" +
          serverID.slice(0, 8) +
          serverID.slice(9)
      );
    } else if (
      message[0] === 0 &&
      message[1] === 0 &&
      message[2] === 9 &&
      message[3] === 4 // CLIENT
    ) {
      var Oserveruid = "";
      for (let i = 4; i < 16; i++) {
        Oserveruid += message[i].toString(16);
      }
      Oserveruid = Oserveruid.slice(0, 8) + "-" + Oserveruid.slice(8);
      console.log("received server uid : " + Oserveruid);

      var serverws = allwss.get(Oserveruid).ws;
      console.log("regClient: " + wsid + "[Server] " + Oserveruid);
      allwss.get(wsid).networkType = "client";
      allwss.get(wsid).serverid = Oserveruid;
      servers.get(Oserveruid).push(wsid);

      //tell server about the new connected client
      serverws.send("OnClientConnectedEvent(" + wsid + ")");

      //tell client about the existing server
      ws.send("OnFoundServerEvent(" + Oserveruid + ")");
    }
  }
  // if(message.length > 4 && message[0] === 0)
  else if (message.length > 4) {
...
```

- 원래는 `message.length==4`였지만, 연결할 때의 payload를 16으로 늘렸기 때문에 바꾸고 적절히 처리해줌
- server일 경우 invitation 링크를 만들어서 string으로 server websocket에 다시 전송함

### `FMWebSocket.cs` 에서의 payload 수정

- `RegisterNetworkType()` express와 동일하게 수정
- 진행 중

### 발생했던 문제들

- payload type을 `Uint8Array` → `Uint32Array`로 바꿨을 때 제대로 입력받지 못함  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Ctrl-flow-7.png)  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Ctrl-flow-8.png)  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Ctrl-flow-9.png)
  - ws에서 메시지 받을때는 원래 데이터 타입과 상관없이 1B로 쪼개서 받아서 이렇게 배치되는 듯
  - `Uint32Array(4)` 대신 `Uint8Array(16)`으로 바꿈
- JS의 `Array`는 asisgn 연산을 실행하면 shallow copy됨

  ```jsx
  var mmap = new Map();
  mmap.set("a", new Array());
  mmap.get("a").push(1);

  var tmp = mmap.get("a");
  console.log(tmp); // ['1']

  mmap.get("a").push(2);
  console.log(tmp); // ['1', '2']

  mmap.delete("a");
  console.log(tmp); // ['1', '2']
  ```

- unity, local 둘 다 server로 연결한 상태에서 local의 disconnect 버튼을 누르면 둘 다 종료됨
  - chrome 창을 닫거나 유니티를 멈추면 각자의 세션만 종료됨
  - `serverWS.close()`로 웹소켓을 닫아버리기 때문
  - `servers`, `allwss`로 웹소켓 인스턴스를 관리해서 종료해야 하는 서버 ws만 종료함
