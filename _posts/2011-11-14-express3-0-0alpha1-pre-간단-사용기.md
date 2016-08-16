---
layout: post
title: express@3.0.0alpha1-pre 간단 사용기
date: 2011-11-14 01:22:02.000000000 +09:00
type: post
published: true
status: publish
categories:
- Node.js
- Programming
- JavaScript
---
node 0.6.x로 업하고 기존 Express와 Socket.IO를 사용했던 프로젝트들이 시작조차 안되서 좀 당황스러웠다.

기존 프로젝트에 있는 Express 모듈이 node 0.6.x를 지원하지 않는다는 메시지가 나와서...

이왕 Express 업그레이드 시키는거 express 2.5.0이 아닌 3.0.0alpha1-pre를 사용해서 써볼까 싶어서 내려받아 띄워봤는데

역시 바로 뜨지 않았다.

하지만 간단한 수정 끝에 서버를 띄울 수 있었는데...

기존 Express + Socket.io 예제 코드는 대체로 아래와 같이 작성되어 있는데,

```js
//...

var app = module.exports = express.createServer()

, io = require('socket.io').listen(app);

...

app.listen(3000);

io.sockets.on('connection', function (socket) {

//...

});
```

Express master tree를 내려받고, Socket.IO를 0.8.7로 업그레이드 시킨 후, 다음과 같이 서버를  실행해보니 서버를 실행할 수 있었다.

```js
// ...

var http = require('http')
  , webApp = module.exports = express()
  , SocketIO = require('socket.io');

// ...

var app = http.createServer(webApp).listen(3000)
  , io = SocketIO.listen(app);

io.sockets.on('connection', function (socket) {

// ...

});
```

하지만 기존에 작성된 코드를 사용할 수 있도록 조만간 app.listen 형태로 바뀔 것 같다.
계속 써볼까 했는데 view engine쪽에 뭔가가 버그가 있는지, EJS를 사용할 수가 없었다. 그래서 결국..

2.5.0으로 걍 내렸다. 흑흑.
