## Config
We have all kinds of configure in application, including framework configuration and project defined ones. ThinkJS manage all configurations and arrange them into `src/config/` folder (or in `src/common/config/` in multi-module project), these configs are categorized and divided into following files.

* `config.js` common config
* `adapter.js` adapter config
* `router.js` customized router config
* `middleware.js` middlware config
* `validator.js` data validation config
* `extend.js` extend config

### format


```js
// src/config.js

module.exports = {
  port: 1234,
  redis: {
    host: '192.168.1.2',
    port: 2456,
    password: ''
  }
}
```

Configuration value can be as simple as a string or a complex object, depends on the specify requirement.

### multi-environment config

Some configurations need to be set according to the environment, like we want the database configuration be different between production and develop environment, this can be acheived by using multi-environment config.

multi-environment config defines as`[name].[env].js`, for example: `config.development.js`, `config.production.js`.
Note: only `config.js` and `adapter.js` support multi-environment config file.

### System Default Config

These are some default configurations need to be aware:

* [config.js](https://github.com/thinkjs/thinkjs/blob/3.0/lib/config/config.js) common config

  ```js
  {
    port: 8360, // server port
    // host: '127.0.0.1', // server host, removed from 3.1.0
    workers: 0, // server workers num, if value is 0 then get cpus num
    createServer: undefined, // create server function
    startServerTimeout: 3000, // before start server time
    reloadSignal: 'SIGUSR2', // reload process signal
    stickyCluster: false, // sticky cluster, add from 3.1.0
    onUnhandledRejection: err => think.logger.error(err), // unhandledRejection handle
    onUncaughtException: err => think.logger.error(err), // uncaughtException handle
    processKillTimeout: 10 * 1000, // process kill timeout, default is 10s
    jsonpCallbackField: 'callback', // jsonp callback field
    jsonContentType: 'application/json', // json content type
    errnoField: 'errno', // errno field
    errmsgField: 'errmsg', // errmsg field
    defaultErrno: 1000, // default errno
    validateDefaultErrno: 1001 // validate default errno
  };
  ```

### Config Merging

When system starts up, all configurations will be merged and provided to users. Below is the merging process:

* load `[ThinkJS]/lib/config/config.js`
* load `src/config/config.js`
* load `src/config/config.[env].js`
* load `[ThinkJS]/lib/config/adapter.js`
* load `[ThinkJS]/lib/config/adapter.[env].js`
* load `src/config/adapter.js`
* load `src/config/adapter.[env].js`

`[env]` is the runtime environment. At last these configurations will be merged together, configs comes later with the same name will override previous ones.
Configs are loaded by [think-loader](https://github.com/thinkjs/think-loader/) module, and the merged configs are represent by [think-config](https://github.com/thinkjs/think-config/), of which instance is accessible through `think.config`.

### Use Config

To quick access config in different environmens:

* for ctx, use `ctx.config(key)`
* for controller, use `controller.config(key)`
* else, use `think.config(key)`

actually, `ctx.config` and `controller.config` is based on [think.config](/doc/3.0/think.html#toc-929).

```js
const redis = ctx.config('redis'); //get redis config
```

```js
module.exports = class extends think.Controller {
  indexAction() {
    const redis = this.config('redis'); // in controller through this.config
  }
}
```

### Dynamic Set Config

Sometimes we need to dynamically set config, like configurations store in database, we can load them on application start and set them.
The Framework provides a dynamic way of set configurations, by using `think.config(key, value)`:

```
// src/bootstrap/worker.js

// Before HTTP service start
think.beforeStartServer(async () => {
  const config = await think.model('config').select();
  think.config('userConfig', config); // read form database and set
})
```

### FAQ

#### Can I set user related value into config?

`No`. config setting is global and will be visible in all request. Different user settings will interfere with each other.

#### Can congiguration in config.js and adapter.js use the same key？

`No`. Because configurations from config.js and adapter.js will be merged, configuration with the same key will be overridden.

#### How to view the merged config？

System will merge config.js and adapter.js configurations on startup, the final config will be save into `runtime/config/[env].json` file, for example if the current env is `development`, then the config file will be `runtime/config/development.json`.

Config will be convert to string using `JSON.stringify` and save as file, becuase JSON.stringify doesn't support regular expression or function, so these value will not be visible.

#### Config file in multi-module project？

In multi-module project, config file is save at `src/common/config/`, the format and the name is the same as single module one, for  example: `src/common/config/config.js`, `src/common/config/adapter.js`, `src/common/config/middleware.js`.

There are config can be put into each module folder, the path is `src/[module]/config/`, `[module]` is the specific module name.

#### How to check the config file loading detail?

Sometime we want to check the config loading detail, we can show loading detail in cli by using `DEBUG=think-loader-config-* npm start` to boot application.

```text
think-loader-config-40322 load file: //demo/app/config/adapter.js +3ms
think-loader-config-40323 load file: /demo/node_modules/thinkjs/lib/config/adapter.js +5ms
think-loader-config-40320 load file: /demo/app/config/adapter.js +4ms
think-loader-config-40323 load file: /demo/app/config/adapter.js +3ms
think-loader-config-40325 load file: /demo/app/config/config.js +0ms
think-loader-config-40325 load file: /demo/node_modules/thinkjs/lib/config/adapter.js +5ms
think-loader-config-40325 load file: /demo/app/config/adapter.js +3ms
think-loader-config-40321 load file: /demo/app/config/config.js +0ms
think-loader-config-40321 load file: /demo/node_modules/thinkjs/lib/config/adapter.js +5ms
think-loader-config-40321 load file: /demo/app/config/adapter.js +3ms
think-loader-config-40324 load file: /demo/app/config/config.js +0ms
think-loader-config-40319 load file: /demo/app/config/config.js +0ms
think-loader-config-40319 load file: /demo/node_modules/thinkjs/lib/config/adapter.js +6ms
think-loader-config-40324 load file: /demo/node_modules/thinkjs/lib/config/adapter.js +5ms
think-loader-config-40319 load file: /demo/app/config/adapter.js +7ms
think-loader-config-40324 load file: /demo/app/config/adapter.js +8ms
```

Because service will start in Master and each Workers, the debug info will display more than once, we add a pid to distinguish: `think-loader-config-40322` is the config loading detail of process `40322`.
