## 运行流程

Node.js provide [http](https://nodejs.org/api/http.html) module to create HTTP service, used to respond to user requests, such as Node.js official website provide examples of creating HTTP services:

```js
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```
ThinkJS also use `http.createServer` to create service, so the whole process contains two parts to start service and respond to user requests.

### Start System Service

* Run `npm start` or `node development.js`.
* Instantiate ThinkJS [Application](https://github.com/thinkjs/thinkjs/blob/3.0/lib/application.js) clas，and call `run` method.
* Depending on the environment (Master process, Worker process, command-line calls) to deal with different logic.
* If it is Master process
    - Load config file and produce `think.config`  `think.logger` object.
    - Load `src/bootstrap/master.js` file.
    - If config to watch service, then start to listen files changes within folder `src/`.
    - After file changed, if config to compile source code, it will do the compilation and output to `app/` folder.
    - According to `workers` configuration to decide the number of Worker to fork. When the worker process has started, the `appReady` event is fired. (Can be captured via `think.app.on("appReady")`).
    - If the file has a new modification, it will trigger the compilation, and then kill all Worker processes and re-fork.
* If it is Worker process
    - Load the configuration file to generate the `think.config` and` think.logger` objects.
    - Extend is loaded to provide more functionality for the framework, the configuration file is `src/config/extend.js`.
    - Get the list of modules for the current project, on `think.app.modules`, or empty array if single module.
    - Load the controller file (`src/controller/*.js`) in the project and place it on the` think.app.controllers` object.
    - Load the logic file (`src/logic/*.js`) in the project and place it on the `think.app.logics` object.
    - Load the project's model file (`src/model/*.js`) and place it on the `think.app.models` object.
    - load the service file (`src/service/*.js`) in the project and place it on the `think.app.services` object.
    - Load the route configuration file `src/config/router.js` on the `think.app.routers` object.
    - Load the validation configuration file `src/config/validator.js` on the `think.app.validators` object.
    - Load the middleware configuration file `src/config/middleware.js` and register it with the `think.app.use` method.
    - Loads the timing task configuration file `src/config/crontab.js` and registers the timing task service.
    - Load the `src/bootstrap/worker.js` startup file.
    - Listen for the `onUncaughtException` and` onUnhandledRejection` errors in the process and process them. You can customize these two erroneous handlers by configuring `src/config.js`.
    - Wait for the pre-launch handler for `think.beforeStartServer` registration, where you can register some transactions before starting the service.
    - If you have customized the create service configuration `createServer`, execute this function `createServer(port, host, callback)` to create the service.
    - If there is no customization, start the service with `think.app.listen`.
    - The appReady event is fired when service startup is complete, elsewhere can listened to this event via `think.app.on("appReady")`.
    - The created service is assigned to the `think.app.server` object.
   
After the service starts, the following log is printed:

```sh
[2017-07-02 13:36:40.646] [INFO] - Server running at http://127.0.0.1:8360
[2017-07-02 13:36:40.649] [INFO] - ThinkJS version: 3.0.0-beta1
[2017-07-02 13:36:40.649] [INFO] - Enviroment: development  #current running environment
[2017-07-02 13:36:40.649] [INFO] - Workers: 8   #worker process number
```

### User reqeust processing

When the user requests service, it will be processed through the following steps.
* The request arrives at the webserver (eg: nginx) and forwards the request to the node service via the reverse proxy. If you access the node service directly through the port, then there is no such step.
* Node service to receive user requests, the Master process will be forwarded to the corresponding Worker process.
* Worker process through the registration of the middleware to handle the user's request:
    - [meta](https://github.com/thinkjs/think-meta) to handle some common information such as: setting request timeout, sending ThinkJS version number, sending processing time, etc.
    - [resource](https://github.com/thinkjs/think-resource) Processing static resource requests, static resources are placed under `www/static/`, if a static resource is hit, this middleware will response resource and stop the following middleware.
    - [trace](https://github.com/thinkjs/think-trace) Handles some error messages, prints detailed error messages in the development environment, and the production environment simply reports a generic error.
    - [payload](https://github.com/thinkjs/think-payload) Handling user-uploaded data, including: form data, files, etc. After the analysis is completed, put the data on `request.body` object for later accessing.
    - [router](https://github.com/thinkjs/think-router) Parse the route to match a Controller's Action, the result is saved on `ctx.controller` and` ctx.action` for subsequent processing . If the project is a multi-module structure then there is `ctx.module`.
    - [logic](https://github.com/thinkjs/think-logic) According to the controller and action parsed out, call the corresponding method in logic.
        - Instantiate the logic class and pass `ctx` into it. If there is no then skip this.
        - Execute `__before` method, if it returns` false`, it will not execute all subsequent logic (end prematurely)
        - If the `xxxAction` method exists then it will be executed. If the result is` false`, then all subsequent logic will not be executed
        - If the `xxxAction` method does not exist, then try the` __call` method
        - Execute the `__after` method, and if it returns `false`, no subsequent logic will execute
        - By the method returns false to block the implementation of subsequent logic
    - [controller](https://github.com/thinkjs/think-controller) According to the resolution of the controller and action, call the corresponding method in the controller.
        - Specific call strategy is exactly the same with logic
        - If not, then the current request returns 404
    - When the action is done, you can put the result on the `this.body` property and return it to the user.
* When Worker error, trigger `onUncaughtException` or `onUnhandledRejection` event, or Worker abnormal exit, Master will capture an error, re-fork a new Worker process, and kill the current process.

We can see that all the user requests are handled through middleware. In specific projects, we add assemble more middleware to handle user's request accordingly.

