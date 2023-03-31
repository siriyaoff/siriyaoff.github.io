---
layout: single
title: "Socket.io 소개"
categories:
  - Dev
tags:
  - JavaScript
  - Socket.IO
  - Socket
  - Namespace
  - Room
---

## Client-side

- `socket.on(eventName, callback)` : `eventName` 이벤트 수신
- `socket.emit(eventName, msg)` : `eventName` 이벤트 전송

## Server-side

- `socket.on(eventName, callback)` : `eventName` 이벤트 수신
  - `socket.join(roomID)` : `roomID` 방에 접속(roomID 대신 array로 여러 방 접속 가능
  - `socket.leave(roomID)` : `roomID`에서 떠남
- `socket.emit(eventName, msg)` : `eventName` 이벤트 전송
- 모두에게 broadcast
  `io.emit()`, `io.sockets.emit()`
- `socket.broadcast.emit()` : 나를 제외한 전체
- `io.to(socketID).emit()` : 특정 ID만

### Namespace

```jsx
var chat=io.of('/chat');
chat.on('news', ()=>());
```

- `chat` namespace에 대하여 수신
- client에선 `io.connect('address/chat', ...)`과 같이 연결할 때 설정해야 함

### Room

- `io.to(roomID).emit()` : 방 전체에 전송
- `socket.broadcast.to(roomID).emit()` : 나를 제외한 방 전체

### Miscellaneous

```jsx
io.adapter.rooms;
io.of(네임스페이스).adapter.rooms;
socket.adapter.rooms;
```

## Namespace, Room 차이

- Room$$\subset$$Namespace
  - 한 namespace 안에 속한 socket들이 다른 room에 속할 수 있음
- namespace는 나갈 수 없음  
  cf. room은 소켓이 연결된 상태에서 자유롭게 출입 가능

## Room 활용 예시

- 입력한 값이 1이면 room join, 11이면 11을 룸 1에 있는 클라이언트들에게 전송
- `views/index.ejs`

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <title>Socket.IO 예제</title>
  </head>
  <body>
    <ul id="messages" type="none">
      <li id="usercount"></li>
    </ul>

    <form id="msgform">
      <input id="msginput" autocomplete="off" type="text" />
      <button type="submit">전송</button>
    </form>
    <script src="/socket.io/socket.io.js"></script>
    <script>
      var socket = io.connect("http://localhost:3000", {
        path: "/socket.io",
        transports: ["websocket"],
      });
      var msgform = document.getElementById("msgform");
      // user count notice
      socket.on("usercount", (count) => {
        var userCounter = document.getElementById("usercount");
        userCounter.innerText = "현재 " + count + "명이 서버에 접속해있습니다.";
      });
      // message notice
      socket.on("news", (msg) => {
        var messageList = document.getElementById("messages");
        var messageTag = document.createElement("li");
        messageTag.innerText = msg;
        messageList.appendChild(messageTag);
      });
      // send message
      msgform.onsubmit = (e) => {
        e.preventDefault();
        var msginput = document.getElementById("msginput");
        // if(socket.adapter.)
        if (msginput.value == "1") {
          socket.emit("join", msginput.value);
        }
        if (msginput.value == "11") {
          socket.emit("whisper", msginput.value);
        } else socket.emit("reply", msginput.value);
        msginput.value = "";
      };
    </script>
  </body>
</html>
Footer
```

- `socket.js`

```jsx
var socketio = require("socket.io");

module.exports = (server) => {
  const io = socketio(server, { path: "/socket.io" });

  io.on("connection", (socket) => {
    const req = socket.request;

    const ip = req.headers["x-forwarded-for"] || req.connection.remoteAddress;
    console.log("New client in", ip, socket.id, req.ip);
    io.emit("usercount", io.engine.clientsCount);

    socket.on("disconnect", () => {
      console.log("Client disconnected", ip, socket.id);
      clearInterval(socket.interval);
    });

    socket.on("error", (error) => {
      console.log(error);
    });

    socket.on("reply", (data) => {
      console.log("client sent: " + data);
      io.emit("news", data);
    });

    socket.on("join", (data) => {
      console.log("join room");
      socket.join(data);
    });

    socket.on("whisper", (data) => {
      console.log("whisper");
      io.in("1").emit("news", data);
    });
  });
};
```
