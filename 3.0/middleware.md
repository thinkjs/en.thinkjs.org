## Middleware

Middleware is a very important concept in Koa, the use of middleware, users can easily handle the request. Since ThinkJS 3.0 was built on the Koa @ 2 release, it is fully compatible with Koa's middleware.

### define middleware

```js
module.exports = options => {
  return (ctx, next) => {
    // do something
  }
}
```

The middleware is a higher-order function, and the external function receives an `options` parameter, which makes it easy for the middleware to provide some configuration information to turn on / off some functions. The function returns `ctx`, `next`, and `ctx` is a shorthand for` context`, which is an object of the current request's life cycle and stores some information about the current request, `next` to call the subsequent middleware, the return value is Promise, so you can easily handle the post-logic.


The middleware execution process is an onion model, similar to the following picture:

![](https://p0.ssl.qhimg.com/t014260f4c3e6e64daa.png)

If you want to implement a middleware to print the current request execution time, you can use something like the following:

```js
const defaultOptions = {
  consoleExecTime: true // to print or not
}
module.exports = (options = {}) => {
  // merge options
  options = Object.assign({}, defaultOptions, options);
  return (ctx, next) => {
    if(!options.consoleExecTime) {
      return next(); // If print the execution time is not needed, call the subsequent execution logic directly
    }
    const startTime = Date.now();
    let err = null;
    // Call next statistics all the time the logic of the follow-up
    return next().catch(e => {
      err = e; // Here first save the error on a Error object, to facilitate the statistics of the execution time under the error circumstances
    }).then(() => {
      const endTime = Date.now();
      console.log(`request exec time: ${endTime - startTime}ms`);
      if(err) return Promise.reject(err); // If subsequent execution logic has an error, return it
    })
  }
}
```

In Koa, you can use middleware by calling `app.use`, such as:

```js
const app = new Koa();
const execTime = require('koa-execTime'); // statistical execution time module
app.use(execTime({}));  // this middle should be use first, and then introduce the rest middle
```

The use of middleware through `app.use` is not conducive to the unified maintenance of middleware.

#### extend app parameters

The default middleware pass `options` parameter, and some middleware need to read the app related information, the framework extend it to auto-pass app object to the middleware.

```js
module.exports = (options, app) => {
  // here the app is think.app object
  return (ctx, next) => {

  }
}
```

The you need to use properties or methods in think object, you can get through `app.think.xxx`.

### configuration

In order to facilitate the management and use of middleware, the framework uses a unified configuration file to manage the middleware, the configuration file is `src/config/middleware.js` (multi-module project configuration file is `sr/common/config/middleware.js`) .
```js
const path = require('path')
const isDev = think.env === 'development'

module.exports = [
  {
    handle: 'meta', // middleware handler
    options: {   // configurations of the middleware
      logRequest: isDev,
      sendResponseTime: isDev,
    },
  },
  {
    handle: 'resource',
    enable: isDev, // whether to enable the current middleware
    options: {
      root: path.join(think.ROOT_PATH, 'www'),
      publicPath: /^\/(static|favicon\.ico)/,
    },
  }
]
```

Configuration items are middlewares list used in the project, each support  `handle`，`enable`，`options`，`match` and other attributes.

#### handle

Middleware processing function, it can be system built-in, or introduced outside or middleware in the project.

handle function signature:

```js
module.exports = (options, app) => {
  return (ctx, next) => {

  }
}
```

The parameters received by the middleware in addition to options, but also a extra `app` object, which is Koa Application instance.

#### enable

Whether to enable current middleware, such as: a middleware only in the development environemnt to take effect.

```js
{
  handle: 'resouce',
  enable: think.env === 'development' //only in development environment to take effect
}
```

#### options

Configuration item passed to middleware, formatted as an object, get this configuration form middlewrae.

```js
module.exports = [
  {
    options: {
      key: value
    } 
  }
]
```

Sometimes the configuration items need to be obtained from remote, such as: the configuration item is stored in database which need to be obtained asynchronously, this time define options as a function.

```js
module.exports = [
  {
    // to define options as async function and return fetched configuration.
    options: async () => {
      const config = await getConfigFromDb();
      return {
        key: config.key,
        value: config.value
      }
    }
  }
]
```

#### match

To enable this middleware when specific rules are matched, support two ways, one is path matching, one is a custom funcion matching. Such as:

```js
module.exports = [
  {
    handle: 'xxx-middleware',
    match: '/resource' // The middleware is enabled when URL is /resource
  }
]
```

```js
module.exports = [
  {
    handle: 'xxx-middleware',
    match: ctx => { // match is a function passed ctx, is the result is true, enable the middleware
      return true;
    }
  }
]
```

### framework built-in middleware

Framework comes with a few middlewares that can be configured by string name.

```js
module.exports = [
  {
    handle: 'meta', // built-in middleware can be confirued by string name.
    options: {}
  }
]
```

* [meta](https://github.com/thinkjs/think-meta) Show some meta information, such as: send ThinkJS version number, interface processing time and so on.
* [resource](https://github.com/thinkjs/think-resource) handle static resouces, suggest disabled it and use webserver in production environment.
* [trace](https://github.com/thinkjs/think-trace) handle error, development environment will print detailed error message, you can also customize the display error page.
* [payload](https://github.com/thinkjs/think-payload) Handle form submission and file upload, similar to middleware like koa-bodyparser.
* [router](https://github.com/thinkjs/think-router) route parsing, including custom route parsing
* [logic](https://github.com/thinkjs/think-logic) data validation.
* [controller](https://github.com/thinkjs/think-controller) invoke controller and action.

### project defined middleware

Sometimes we need to add middleware according to specific project requirement, it can put them into `src/middleware` direcotry, by which you can reference them through file name.

Such as: Add `src/middleware/csrf.js`, then you can reference this middleware directly through `csrf` string.

```js
module.exports = [
  {
    handle: 'csrf',
    options: {}
  }
]
```


### Introduce external middleware

The introduction of external middleware is every simple, just need to require it.

```js
const csrf = require('csrf'); 
module.exports = [
  ...,
  {
    handle: csrf,
    options: {}
  },
  ...
]
```

### FAQ

#### Do I need to consider the order of middleware configuration?

Middleware execution is performed in the order of configuration, so developers need to consider the order of configuration.

#### How do I know which middleware in the current environment to take effect?

You can view the infomation using `DEBUG=koa:application node development.js` to start the application, in console message as bellow will be printed: `koa:application use ...`.

Note: if multiple workers are started then middleware info will be printed multiple times.

#### How to pass data between Logic and Controller?

Sometimes we want to configure some data in middleware and access it in the subseuent Logic, Controller middleware, this can be done use `ctx.state`, for detail refers to [passing data](/doc/3.0/controller.html#toc-247).


#### How to set data in GET/POST?

In the middleware, we can get the parameters of the query or the data submitted by the form through `ctx.param`, `ctx.post`, etc. However, in some middleware, we hope to set some parameter values and form values to be used in subsequent Logic, Controller in this time can be set by `ctx.param`,`ctx.post`:

```js
// set param name=value，so that to get value through this.get('name') in Logic, Controller
// it will override the value if it already exists.
ctx.param('name', 'value');

// set post value，later in Logic or Controller to get value through this.post('name2')
ctx.post('name2', 'value');
```

#### Can I put middleware configuration into config.js?

`Inappropriate`, the middleware provides `options` parameter to set configuration, no need to configure additional parameters in config.js.

```js
module.exports = [
  {
    handle: xxxMiddleware,
    options: { // middleware configuration
      key1: value1,
      key2: think.env === 'development' ? value2 : value3
    }
  }
]
```

If some configuration are `env` related, then you can make judgments here.