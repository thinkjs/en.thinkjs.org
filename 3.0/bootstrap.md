## Customize Bootstrap

We use `npm start` or `node production.js` to bootstrap application. Though we can add application logic inside these two files, but it isn't recommanded. System proivde other entries for adding bootstrap behavious. 

### bootstrap

System will load file under `src/boostrap/` which are:

* Master process will load `src/bootstrap/master.js`
* Each worker processes will load `src/bootstrap/worker.js`

So we can put bootstrap related code inside these two files respectively.

If there are code need to be called in both Master and Woker process, you can extract it into a seperate file and require it in both master.js and worker.js.

```js
// src/bootstrap/common.js
global.commonFn = function(){

}

// src/bootstrap/master.js
require('./common.js')


// src/boostrap/worker.js
require('./common.js')

```

### think.beforeStartServer

If you want to add logic before node start http server. For example, to read configuration from database, or read a dataset from remote host.
`think.beforeStartServer` is design for this scenario: 

```js
think.beforeStartServer(async () => {
  const data = await getDataFromApi();
  think.config(key, data);
})
```
`think.beforeStartServer` allows to register multiple callbacks, it also accepts a Promise which means you can use `async/await` to perform a asynchronous operation.

System will not start until all registered callbacks are finished. of cause there is a timeout limit. You can set the timeout value by configure `startServerTimeout` which is 3 seconds by default.

```js
//src/config/config.js
module.exports = {
  startServerTimeout: 5000 // change timeout to 5s
}
```

### Customize http service

By default system create http service by calling Koa's `listen` method, you can also customize it by using `createServer` configure.

```js
// src/config/config.js
const https = require('https');
const fs = require('fs');

const options = {
  key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
  cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem')
};

module.exports = {
  // just create service, no listen
  createServer: function(callback){
    return https.createServer(options, callback);
  }
}
```

### think.app.server

You can get http service instance in `think.app.server`.

### appReady event

After http service is created, it will fire `appReady` event. To register it, use `think.app.on("appReady")`.

```js
think.app.on("appReady", () => {
  const server = think.app.server;
})
```
