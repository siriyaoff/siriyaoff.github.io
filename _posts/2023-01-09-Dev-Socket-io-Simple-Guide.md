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

- Room$\sub$Namespace
  - 한 namespace 안에 속한 socket들이 다른 room에 속할 수 있음
- namespace는 나갈 수 없음
  cf. room은 소켓이 연결된 상태에서 자유롭게 출입 가능
