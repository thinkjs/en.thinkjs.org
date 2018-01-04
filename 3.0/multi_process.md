## Multi-process

Node provides `cluster` module to create multi-process applications, so as to avoid a single process hangs cause service exceptions. The framework runs the multi-process model from the [think-cluster](https://github.com/thinkjs/think-cluster) module.


### Configuration

Set `workers` number to specify child process number, default is `0` (the current host cpu number).

```js
//src/config/config.js
module.exports = {
  workers: 0 // can be set according to actual situation, 0 means the number of cpu
}
``` 

### Multi-process model

In multi-process model, Master will `fork` `workers` number of child processes, the user request is handle by Worker. If Worker process runs into exception, Master is notified to Fork a new Worker process and stop the current Worker from receiving user request.

### Inter-process communication

Sometimes Workers processes need to communicate to each other, to exchange some data. But Worker process can't communicate directly, it need to use Master process to transit.
Framework provide `think.messenger` to handle inter-process communication, there are several ways:


* `think.messenger.broadcast` will brocas message to all Worker processes
  
  ```js
  //listen to event  src/bootstrap/worker.js
  think.messenger.on('test', data => {
    //All child process capture this this event and runs to here, including current process.
  });

  // src/controller/xxx.js
  // trigger broadcast event
  think.messenger.broadcast('test', data);
  ```

* `think.messenger.consume` Task consumer, only execute in one process. (Sometimes some tasks need to start on system bootstrap, and only want it to run once in one of children process)

  ```js
  // src/bootstrap/worker.js
  think.messenger.on('consumeEvent', (data) => {
    // The callback only run in one child process
  });

  // trigger event, it will be capture by one process
  think.messenger.consume('consumeEvent', data);
  ```

* `think.messenger.map`  execute task in all child processes, and return the task result(the result needs to call JSON.stringify before transmit, which should not be too large, for large dataset transmit can use other approachs, such as file.).

  ```js
  // src/bootstrap/worker.js
  think.messenger.on('testMap', (data) => {
    return Math.random();
  });

  // src/controller/xxx.js
  if(xxx) {
    // Get return result from all child process task, the value is Array.
    // On execution only the first registered callback will be called.
    const data = await think.messenger.map('testMap', data);
  }
  ```

Note: consume and map method needs [think-cluster](https://github.com/thinkjs/think-cluster) version `>=1.4.0`.

### Custom inter-process communication

Sometimes built-in communication methods can not meet all the needs, time time you can customize it. For Master process execution calls `src/bootstrap/master.js`, while Worker processes call `src/bootstrap/worker.js`, by which the inter-process handling will be more easier.

```js
// src/bootstrap/master.js
const cluster = require('cluster');
cluster.on('message', (worker, message) => {
  // Receive specific message for handling
  if(message && message.act === 'xxx'){

  }
})

// src/bootstrap/worker.js
process.send({act: 'xxxx', ...args}); //send data to Master process

```

### FAQ

#### How child process to master process to restart service?

Sometime a general task system needs to be able to auto update itself (such as: blog system update feature), after code is changed, it needs to restart service to make it take effect, instead to manually restart the system, framework provide `think-cluster-reload-workers` directive to allow child process notify master process to restart. Such as:

```js
async upgrade() {
  await downloadCodeFromRemote(); // download update package from remote
  await unzipCode(); // unzip the package
  await installDependencies(); // need dependencies may be added so to reinstall dependencies here
  process.send('think-cluster-reload-workers'); // send master process restart directive
}
```