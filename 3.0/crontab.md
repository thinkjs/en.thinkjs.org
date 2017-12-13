## Crontab / Timing tasks

When the project is online, it is often necessary to periodically perform certain functions (for example, timing to pull some data remotely and summarizing some data in the database regularly). In this case, the task needs to be processed using a scheduled task.

Framework provides two ways to perform scheduled tasks, one is executed in the currently started child process, the other is executed with a new process (command line execution).

### Execute in current child process

Some of the timing task is to pull some data in memory for use in the user request process, this timing task you want to call in the current process (because the framework starts the service is multi-process architecture, so the timing task only in the child process), the function is through [think-crontab](https://github.com/thinkjs/think-crontab) module to complete.

#### Configuration

The configuration file for the scheduled task is `src/config/crontab.js` (or `src/common/config/crontab.js` under multi-module project, also supports configure for each module with `src/[module]/config/crontab.js`), the configuration item is an array. Such as:

```javascript
module.exports = [{
  interval: '10s',
  immediate: true,
  handle: () => {
    //do something
  }
}, {
  cron: '0 */1 * * *',
  handle: 'crontab/test',
  type: 'all'
}]
```

The parameters supported by each configuration item are as follows:

* `interval` {String | Number} execute interval

  Support number and string formats, in miliseconds. If it is string format, it will be parsed to number using [think.ms](/doc/3.0/think.html#toc-35a) method.

* `cron` {String} crontab format, like `0 */1 * * *`

  crontab format，see specific <http://crontab.org/>. If `interval` attribute is configured, this config will be ignore.

* `type` {String} task execution method, *one* or *all*, default is *one*

  Child processes in which the task will be executing, default it is in one child process, `all` means in all child precesses. Even it is configured to run in one process, it can only guarantee the rule in one machine, multiple machines will execute machine number times. If we want to run once across idc and machine, It can be done by `enable` parameter control or command line execution.

* `handle` {Function | String} Execute the task, the implementation of the corresponding function or routing address, such as: `crontab/test`

  Scheduled task handler, it can be a specific function, or route address in string format (it will be parse by route and run conresponding Action).

* `immediate` {Boolean} Whether execute immediately, default is `false`

  If immediately execute task once.

* `enable` {Boolean} enable scheduled task, default is `true`

  If enable sacheduled task, set to `false` to disable this task rule. For example: we only want to execute once among multiple machines, we can decide by machine host name.


  ```js
  const hostname = require('os').hostname();
  module.exports = [{
    interval: '10s',
    enable: hostname === 'host name',
    handle: () => {
      //do something
    }
  }]
  ```

#### Debug

If you want to see the scheduled task are sunning successfully, you can view the debugging information by start project with `DEBUG=think-crontab npm start`.

### Commandl ine execution

If there are regular tasks across multiple idcs and machines want to perform once, or the task is time-consming, then we execute it in command line. Command line execution need to be done in conjunction wiht system crontab task.

Command line execution use script with route address, such as: `node production.js crontab/test`, where `crontab/test` is route address, so combined with the system crontab can be scheduled execution.

We can edit scheduled task with command `crontab -e`, such as:

```sh
0 */1 * * * /bin/sh (cd projectpath; node production.js crontab/test) # execute every one hour
```

### FAQ

#### How to limit Action is for scheduled task only?

By default, Action doesn't has access control, so schuduled task action can also access with URL. If we don't want this, we can use `this.isCli` to stop accordingly:

```js
module.exports = class extends think.Controller {
  testAction() {
    // reject access if it is not from cli
    if(!this.isCli) return this.fail(1000, 'deny');
    ...
  }
}
```
Action invoke by scheduled task is not a normal user request, but by simulating a request to complete, the simulation set request type to `CLI`, `isCli` is to determine whether the request type is `CLI`.