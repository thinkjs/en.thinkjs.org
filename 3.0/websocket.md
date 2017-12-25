## WebSocket

ThinkJS currently supports `socket.io` for WebSocket and has some simple packaging for it, with subsequent additions of [socketjs](https://github.com/sockjs/sockjs-node), [ws](https://github.com/websockets/ws) Library support.

### Turn on the WebSocket

In a clustered environment, WebSocket requires a sticky session to ensure that a given client requests that the same worker be hit, otherwise the handshake mechanism will not work. In order to do this, you need to turn on the `stickyCluster` configuration.

In order to ensure the performance, `stickyCluster` function is disabled by default. If the project needs to be enabled, you can modify the configuration file `src/config/config.js`:

```js
module.exports = {
  stickyCluster: true,
  // ...
};
```

### Configuration WebSocket

WebSocket is integrated into ThinkJS as `extend`, with `src/config/extend.js` first configured:

```js
const websocket = require('think-websocket');

module.exports = [
  // ...
  websocket(think.app),
];
```

Each implementation of WebSocket exists as an `adapter`, taking `socket.io` as an example (encapsulated with [think-websocket-socket.io](https://github.com/thinkjs/think-websocket-socket.io)), configured in `src/config/adapter.js` as follows:

```js
const socketio = require('think-websocket-socket.io');
exports.websocket = {
  type: 'socketio',
  common: {
    // common config
  },
  socketio: {
    handle: socketio,
    allowOrigin: '127.0.0.1:8360',  // all domains are allowed by default
    path: '/socket.io',             // '/socket.io' as default
    adapter: null,                  // no adapter by default
    messages: [{
      open: '/websocket/open',
      addUser: '/websocket/addUser'
    }]
  }
}
```

### Event to Action mapping

Taking `socket.io` as an example, ThinkJS follows the mechanism of interaction between the server and client of `socket.io` through an event, so the server needs to map the event name to the corresponding Action in order to respond to the specific event. The mapping of events is configured in the `messages` field as follows:

```js
exports.websocket = {
  // ...
  socketio: {
    // ...
    messages: {
      open: '/websocket/open',       // when the connection is established, the open Action under the corresponding websocket Controller is processed
      close: '/websocket/close',     // Action to close when closing the connection
      addUser: '/websocket/addUser', // Action for addUser event handling
    }
  }
}
```

The `open` and `close` event names are fixed, which means that the connection is established and disconnected. Other events are self-defined, and can be added as needed in the project.

### Server-side Action Processing

By configuring the mapping of the event to the Action, you can do the corresponding processing in the corresponding Action. Such as:

```js
module.exports = class extends think.Controller {

  constructor(...arg) {
    super(...arg);
  }

  openAction() {
    this.emit('opend', 'This client opened successfully!')
    this.broadcast('joined', 'There is a new client joined successfully!')
  }

  addUserAction() {
    console.log('get the data sent by the client addUser event', this.wsData);
    console.log('get the current WebSocket object', this.websocket);
    console.log('determine if the current request is a WebSocket request', this.isWebsocket);
  }
}
```

#### emit

Action can send the current `socket` event by `this.emit` method, such as:

```js
module.exports = class extends think.Controller {

  constructor(...arg) {
    super(...arg);
  }

  openAction() {
    this.emit('opend', 'This client opened successfully!')
  }
}
```

#### broadcast

Action can give all the `socket` broadcast events by `this.broadcase` method, such as:

```js
module.exports = class extends think.Controller {

  constructor(...arg) {
    super(...arg);
  }

  openAction() {
    this.broadcast('joined', 'There is a new client joined successfully!')
  }
}
```

### Client example

Client sample code is as follows:

```
<script src="http://lib.baomitu.com/socket.io/2.0.1/socket.io.js"></script>
<script type="text/javascript">
  var socket = io('http://localhost:8360');

  $('.send').on('click', function(evt) {
    var username = $.trim($('.usernameInput').val());
    if(username) {
      socket.emit('addUser', username);
    }
  });

  socket.on('opend', function(data) {
    console.log('opend:', data);
  });

  socket.on('joined', function(data) {
    console.log('joined:', data);
  });
</script>
```


### socket.io

`socket.io` wraps both front and back of WebSocket and is very convenient to use.

#### io object

In Action, you can get `io` object via `this.ctx.req.io`, which is an instance of `socket.io`.

io object contains the method see the document [https://socket.io/docs/server-api/#server](https://socket.io/docs/server-api/#server)。

#### Set path

Set the path to be handled by `socket.io`, which defaults to `/socket.io`. If you need to change, you can modify the configuration of `src/config/adapter.js`:

```js
exports.websocket = {
  // ...
  socketio: {
    // ...
    path: '/socket.io',
  }
}
```

For the detailed configuration of path, see the document [https://socket.io/docs/server-api/#server-path-value](https://socket.io/docs/server-api/#server-path-value). Note that if the server modifies the processing path, the client also needs to make corresponding changes.

#### Set allowOrigin

By default, `socket.io` allows all domain names to be accessed. If you need to change, you can modify the configuration of `src/config/adapter.js`:

```js
exports.websocket = {
  // ...
  socketio: {
    // ...
    allowOrigin: '127.0.0.1:8360',
  }
}
```
For the detailed configuration of allowOrigin, see the document
[https://socket.io/docs/server-api/#server-origins-value](https://socket.io/docs/server-api/#server-origins-value). Note that if the server modifies the processing path, the client also needs to make corresponding changes.

#### Set adapter

When using multi-node to deploy WebSocket, multiple nodes can communicate with Redis, then you can set the adapter to achieve.

```js
const redis = require('socket.io-redis');
exports.websocket = {
  // ...
  socketio: {
    // ...
    adapter: redis,
  }
}
```
For the detailed configuration of adapter, see the document [https://socket.io/docs/server-api/#server-adapter-value](https://socket.io/docs/server-api/#server-adapter-value)。
