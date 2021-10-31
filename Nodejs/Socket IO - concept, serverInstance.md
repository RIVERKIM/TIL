# Socket IO - concept, serverInstance

생성일: 2021년 8월 30일 오후 9:52

### WebSocket 란?

- 사용자의 부라우저와 서버 사이의 동적인 양방향 연결 채널을 구성하는 HTML5  프로토콜이다. WebSocket API를 통해 서버로 메시지를 보내고 요청없이 응답을 받아오는 것이 가능하다.
- HTTP는 클라이언트에 의해 초기화되기 때문에 서버가 변경사항을 클라이언트에게 알릴 수 있는 방법이 없지만 Websocket의 연결은 HTTP 통신과는 다르게 클라이언트가 특정 주기를 가지고 Polling 하지 않아도 변경된 사항을 시기 적절하게 전달할 수 있는 지속적이고 완전한 양방향 연결 스트림을 만들어 주는 기술이다.

![Untitled](Socket%20IO%20-%20concept,%20serverInstance%2075927b8d533d47e0806c57e52fa85fa6/Untitled.png)

**Client: index.html**

```java
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Native WebSocket Example</title>
</head>
<body>
<script>
  // 웹소켓 전역 객체 생성
  var ws = new WebSocket("ws://localhost:3000");

  // 연결이 수립되면 서버에 메시지를 전송한다
  ws.onopen = function(event) {
    ws.send("Client message: Hi!");
  }

  // 서버로 부터 메시지를 수신한다
  ws.onmessage = function(event) {
    console.log("Server message: ", event.data);
  }

  // error event handler
  ws.onerror = function(event) {
    console.log("Server error message: ", event.data);
  }
</script>
</body>
</html>
```

**Server: app.js**

```java
var WebSocketS = require("ws").Server;
var wss = new WebSocketServer({ port: 3000 });

// 연결이 수립되면 클라이언트에 메시지를 전송하고 클라이언트로부터의 메시지를 수신한다
wss.on("connection", function(ws) {
  ws.send("Hello! I am a server.");
  ws.on("message", function(message) {
    console.log("Received: %s", message);
  });
});
```

### [Socket.](http://Socket.Io)io 란?

- WebSocket을 기반으로 클라이언트와 서버의 양방향 통신을 가능하게 해주는 모듈이다.
- 클라이언트에서 EventListener로 새로운 정보를 받아 정보를 업데이트 한다.
- 실시간 분석, 메시지, 채팅 등

**설치**

```java
npm install socket.io // cf) client 의 경우 socket.io-client
```

### Initialize with Express

```java
const express = require('express')
const { createServer } = require('http');
const { Server } = require('socket.io');

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, { /*options*/})

io.on('connection', (socket) => {
	socket.on('event-name', (data) => {})
})

httpServer.listen(3000, () => console.log("~"));
```

### Server Instance

**Server#engine** : A reference to the underlying [Engine.IO](http://Engine.IO) server

- fetch the number of currently connected clients

```java
const count = io.engine.clientsCount
// may or may not similary
const count2 = io.of('/').sockets.size;
```

- generate a custom session ID

```java
const uuid = require('uuid')

io.engine.generateId = (req) => {
	return uuid.v4();
}
```

- connection_error: will be emitted when a connection is abnormally closed

```java
io.engine.on("connection_error", (err) => {
  console.log(err.req);      // the request object
  console.log(err.code);     // the error code, for example 1
  console.log(err.message);  // the error message, for example "Session ID unknown"
  console.log(err.context);  // some additional error context
});
```

**Engine Utility method**

- SocketsJoin: makes the matching socket instance join the specified rooms

```java
// make all Socket instances join the "room1" room
io.socketsJoin("room1");

// make all Socket instances in the "room1" room join the "room2" and "room3" rooms
io.in("room1").socketsJoin(["room2", "room3"]);

// make all Socket instances in the "room1" room of the "admin" namespace join the "room2" room
io.of("/admin").in("room1").socketsJoin("room2");

// this also works with a single socket ID
io.in(theSocketId).socketsJoin("room1");
```

- socketsLeave: makes the matching socket instance leave the specified rooms

```java
// make all Socket instances leave the "room1" room
io.socketsLeave("room1");

// make all Socket instances in the "room1" room leave the "room2" and "room3" rooms
io.in("room1").socketsLeave(["room2", "room3"]);

// make all Socket instances in the "room1" room of the "admin" namespace leave the "room2" room
io.of("/admin").in("room1").socketsLeave("room2");

// this also works with a single socket ID
io.in(theSocketId).socketsLeave("room1");
```

- disconnectSockets: makes the matching socket instances disconnect

```java
// make all Socket instances disconnect
io.disconnectSockets();

// make all Socket instances in the "room1" room disconnect (and discard the low-level connection)
io.in("room1").disconnectSockets(true);

// make all Socket instances in the "room1" room of the "admin" namespace disconnect
io.of("/admin").in("room1").disconnectSockets();

// this also works with a single socket ID
io.of("/admin").in(theSocketId).disconnectSockets();
```

- fecthSockets: return the matching socket instances

```java
// return all Socket instances
const sockets = await io.fetchSockets();

// return all Socket instances in the "room1" room of the main namespace
const sockets = await io.in("room1").fetchSockets();

// return all Socket instances in the "room1" room of the "admin" namespace
const sockets = await io.of("/admin").in("room1").fetchSockets();

// this also works with a single socket ID
const sockets = await io.in(theSocketId).fetchSockets();
```

- socket

```java
for (const socket of sockets) {
  console.log(socket.id);
  console.log(socket.handshake);
  console.log(socket.rooms);
  console.log(socket.data);
  socket.emit(/* ... */);
  socket.join(/* ... */);
  socket.leave(/* ... */);
  socket.disconnect(/* ... */);
}
```

- data: arbitrary object that can be used to share information between [Socket.IO](http://Socket.IO) servers

```java
// server A
io.on("connection", (socket) => {
  socket.data.username = "alice";
});

// server B
const sockets = await io.fetchSockets();
console.log(sockets[0].data.username); // "alice"
```