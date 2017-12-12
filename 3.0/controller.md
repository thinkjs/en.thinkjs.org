## Controller / 控制器

In the MVC model, the controller is the logical processing part of the user's request. For example: the user-related operations are placed in `user.js`, each operation is inside an Action.

### Create Controller

Project controller needs to inherit from the think.Controller class, so that it can use some built-in methods. Of course, the project can create some common base class, and then the actual controller inherits from this base class.

When the project is created, it automatically creates a base class named `base.js`, which other controllers can inherit from.

```js
//src/controller/user.js

const Base = require('./base.js');
module.exports = class extends Base {
  indexAction(){
    this.body = 'hello world!';
  }
}
```

When done creation, framework will watch file changes to restart service. Access `http://127.0.0.1:8360/user/index` will see messag `hello word!`.

### Action Execution

Action execution is done via [think-controller](https://github.com/thinkjs/think-controller) middleware, According to `ctx.action` value to match controller `xxxAction` mehtod, call this method and related hook method, ithe specific order as bellow:

* Instantiate the Controller class, pass in the `ctx` object.
* Called if the method [__before](/doc/3.0/controller.html#toc-083) exists, and stop it if the return value is false.
* Called  if `xxxAction` exist, and stop it if return value is `false`.
* If `xxxAction` is not exist but [__call](/doc/3.0/controller.html#toc-fcb) is there， call `__call`, and stop it if return value is `false`.
* Execute [__after](/doc/3.0/controller.html#toc-e16) if it exists.

### Pre-operation __before

Project sometimes need to do something in a unified place, such as: to determine whether it has been logged in, if not logged in can not continue the behavior behind. In this case, this can be done with the built-in `__before`.

`__before` is called before calling the concrete Action, so that you can do something in it.

```js
module.exports = class extends think.Controller {
  async __before(){
    const userInfo = await this.session('userInfo');
    // Get the user's session information, if null, return false to prevent subsequent behavior to continue
    if(think.isEmpty(userInfo)){
      return false;
    }
  }
  indexAction(){
    // indexAction is call after the completion of __before
  }
}
```

If the class inheritance needs to call the parent `__before` method, it can be done by` super.__ before`, such as:

```js
module.exports = class extends Base {
  __before(){
    // Use Promise.resolve to wrap return value into Promise
    // if hte return value already is Promise, you can skip this.
    return Promise.resolve(super.__before()).then(flag => {
      // IF you want to stop subsequent execution will return false, here to judge flag as false no longer continue.
      if(flag === false) return false;

      // other code logic
    })
  }
}
```

If Babel is not used, bellow is a more concise way:

```js
module.exports = class extends Base {
  async __before(){
    const flag = await super.__before();
    if(flag === false) return false;
    ...
  }
}
```

### Post operation __after

Post-operation `__after` corresponds with` __before`, but only after specific Action is executed. If a specific Action execution returns `false`,` __after` will not be executed again.

```js
module.exports = class extends think.Controller {
  indexAction(){

  }
  __after(){
    // will execute after indexAction, if indexAction returns false then no longer executed
  }
}
```

### Magic method __call

When the parsed url corresponding to the controller exists, but the Action does not exist, it will try to call the controller magic method `__call`. Here you can deal with non-existent methods.

```js
module.exports = class extends think.Controller {
  indexAction(){

  }
  __call(){
    // this method will be called if action not exist
  }
}
```

### ctx object

Controller instantiation will be passed [ctx](/doc/3.0/context.html) object, Controller can get ctx object using `this.ctx`, and many of the Controller methods is also done by calling the ctx method.

If subclasses need to override the constructor method, you need to call the constructor in the parent class and pass in the ctx parameter:

```js
const Base = require('./base.js');
module.exports = class extends Base {
  constructor(ctx){
    super(ctx); // Invoke parent constructor and pass ctx
    // rest
  }
}
```

### Multi-level controller

Sometimes the project is complex with lots of files, we need to divide them by feature. Such as: to put user-side on a piece, admin-side on a piece.

At this point you can do this with a multilevel controller by creating the `user/` and `admin/` directories in the `src/controller/` directory and then the user-side functional files in the `user/` directory, and admin-side files in `admin/` directory. When access with the corresponding directory name, routing parsing will first try to match the directory for controller.

If there is a console controller subdirectory, there is a user.js file, that is: `src/controller/console/user.js`. When the access request is `/console/user/login`, route will first locate `console/user`, and Action name is `login`.

### Prevent subsequent logic from executing

Controller process in the order of `__before`, `xxxAction`,` __after`, and in some specific scenarios, the need to end the request ahead of time, to prevent the follow-up logic to continue execution. We can use `return false` to acheive that.

```js
module.exports = class extends think.Controller {
  __before() {
    if(!user.isLogin) {
      return false; // return false，then xxxAction and __after will not longer be executed
    }
  }
  xxxAction() {
    // if return false will prevent __after from executing
  }
  __after() {

  }
}
```

### Get params and form values

The parameters on the URL or form upload value, have been been resolved directly by framework, you can directly get them through the corresponding method.
Use [get](/doc/3.0/controller.html#toc-b4e) in Action for parameters on URL. The fields or files submitted by form can be obtained by [post](/doc/3.0/controller.html#toc-3d4) nad [file](/doc/3.0/controller.html#toc-88b) method. The parsing of the form data is done through [think-payload] (https://github.com/thinkjs/think-payload) middleware. The parsed data is placed on the object `ctx.request.body` and finally packaged as post and file methods for use.

### Passing data

由于用户的请求处理经过了中间件、Logic、Controller 等多层的处理，有时候希望在这些环节中透传一些数据，这时候可以通过 `ctx.state.xxx` 来完成。
Because the user's request processing through the middleware, Logic, Controller and other middleware layers, and sometimes hope to pass data between them, we can use `ctx.state.xxx` for that.


```js
// middleware set state
(ctx, next) => {
  ctx.state.userInfo = {};
}

// get state in Logic and Controller
indexAction() {
  const userInfo = this.ctx.state.userInfo;
}
```

Avoid directly adding properties to `ctx` objects when passing data, which may overwrite existing properties and cause some weird issues.

### FAQ

#### How to get req and res object?

Sometime we need to access Node's `req` and `res` object, we can use `this.ctx.req` and `this.ctx.res`:

```js
module.exports = class extends think.Controller {
  indexAction() {
    const req = this.ctx.req;
    const res = this.ctx.res;
    // do something with req & res
  }
}
```

#### Why would use async/await with super reports error？

Currently Babel stable version is `6.x`, if use async/await with super, the code transpiled will cause error, we need Babel version 7.0 to fix that, refer to <https://github.com/babel/babel/issues/3930> for detail.

Workaround is don't use aysnc/await with super, or use Promise directly:

```js
module.exports = class extends Base {
  aaa () {
    // use Promise.resolve to wrap parent method return value as Promise，so that we can use then
    return Promise.resolve(super.aaa()).then(data => {
      ...
    })
  }
}
```

### API


#### controller.ctx

`ctx` object.

#### controller.body

Get or set return valuek, equivalent to [ctx.body](/doc/3.0/context.html#toc-688).

#### controller.ip

* `return` {String}

Get current user ip, equivalent to [ctx.ip](/doc/3.0/context.html#toc-5d1).

```js
module.exports = class extends think.Controller {
  indexAction() {
    const ip = this.ip; // get user IP
  }
}
```


#### controller.ips

Get current request chain's all ip, equivalent to [ctx.ips](/doc/3.0/context.html#toc-f4e).

#### controller.method

Get request type, equivalent to [ctx.method](/doc/3.0/context.html#toc-972).

```js
module.exports = class extends think.Controller {
  indexAction() {
    const method = this.method; // get request method
    if(method === 'OPTIONS') {

    }
  }
}
```

#### controller.isGet

Judge wehther it is GET request, equivalent to [ctx.isGet](/doc/3.0/context.html#toc-15d).

```js
module.exports = class extends think.Controller {
  indexAction() {
    if(this.isGet) {

    }
  }
}
```

#### controller.isPost

Judge whether it is POST request, equivalent to [ctx.isPost](/doc/3.0/context.html#toc-056).

```js
module.exports = class extends think.Controller {
  indexAction() {
    if(this.isPost) {

    }
  }
}
```

#### controller.isCli

* `return` {Boolean}

Judge whether it is call from cli, equivalent to [ctx.isCli](/doc/3.0/context.html#toc-e64).


```js
module.exports = class extends think.Controller {
  indexAction() {
    if(this.isCli) {

    }
  }
}
```

#### controller.userAgent

Get current request's userAgent, equivalent to [ctx.userAgent](/doc/3.0/context.html#toc-125).

```js
module.exports = class extends think.Controller {
  indexAction() {
    const userAgent = (this.userAgent || '').toLowerCase();
    if(userAgent.indexOf('spider') > -1) {

    }
  }
}
```

#### controller.isMethod(method)

Determine whether the current request type is the specified type, equivalent to [ctx.isMethod](/doc/3.0/context.html#toc-dd7).

```js
module.exports = class extends think.Controller {
  indexAction() {
    const isDelete = this.isMethod('DELETE'); 
  }
}
```


#### controller.isAjax(method)

Judge whether it is an Ajax request. If you specify method, then the request type must be the same, equivalent to [ctx.isAjax](/doc/3.0/context.html#toc-677).

```js
module.exports = class extends think.Controller {
  indexAction(){
    let isAjax = this.isAjax('post');
  }
}
```

#### controller.isJsonp(callback)

Equivalent to [ctx.isJsonp](/doc/3.0/context.html#toc-178).

#### controller.get(name)

Equivalent to [ctx.param](/doc/3.0/context.html#toc-f5e). because ctx.get is use by Koa, so we can't add ctx.get.

#### controller.post(name)

Equivalent to [ctx.post](/doc/3.0/context.html#toc-29b).

#### controller.file(name)

Equivalent to [ctx.file](/doc/3.0/context.html#toc-322).

#### controller.header(name, value)

* `name` {String} header name
* `value` {String} header value

Get or set header.

```js
module.exports = class extends think.Controller {
  indexAction(){
    let accept = this.header('accept'); //get header
    this.header('X-NAME', 'thinks'); //set header
  }
}
```

#### controller.expires(time)

Equivalent to [ctx.expires](/doc/3.0/context.html#toc-f99).

#### controller.referer(onlyHost)

Equivalent to [ctx.referer](/doc/3.0/context.html#toc-38c).

#### controller.referrer(onlyHost)

Equivalent to controller.referer.

#### controller.cookie(name, value, options)

Equivalent to [ctx.cookie](/doc/3.0/context.html#toc-a67).

#### controller.redirect(url)

Equivalent to [ctx.redirect](/doc/3.0/context.html#toc-3e0).

#### controller.jsonp(data, callback)

Equivalent to [ctx.jsonp](/doc/3.0/context.html#toc-45f).

#### controller.json(data)

Equivalent to [ctx.json](/doc/3.0/context.html#toc-77f).

#### controller.status

Equivalent to [ctx.status](/doc/3.0/context.html#toc-606).

#### controller.success(data, message)

Equivalent to [ctx.success](/doc/3.0/context.html#toc-526).


#### controller.fail(errno, errmsg, data)

Equivalent to [ctx.fail](/doc/3.0/context.html#toc-c4f).

#### controller.download(filepath, filename)

Equivalent to[ctx.download](/doc/3.0/context.html#toc-b4e).

#### controller.controller(name, m)

* `name` {String} controller name
* `m` {String} module name, only for multi-module project
* `return` {Object} controller instance

Get another controller instance, will throw error if not exist.

```js
module.exports = class extends think.Controller {
  indexAction() {
    // get another controller instance and call its method
    const userController = this.controller('user');
    userController.xxx();
  }
  index2Action() {
    // get sub-controller instance and call its method
    const userController = this.controller('console/user');
    userController.xxx();
  }
  index3Action() {
    // get admin module controller instance and invoke its method
    const userController = this.controller('console/user', 'admin');
    userController.xxx();
  }
}
```

#### controller.action(controller, name, m)

* `controller` {String | Object} controller name
* `name` {String} Action name
* `m` {String, Optional} module name, only for multi-module project
* `return` {Mixed}

Call anthoer controller's action method, also invoke `__before`, `__after` magic methods.

```js
module.exports = class extends think.Controller {
  indexAction() {
    // call user controller's loginAction
    const ret = this.action('user', 'login');
  }
  index2Action() {
    // call front/user controller(sub controller) loginAction
    const ret = this.action('front/user', 'login');
  }
  index3Action() {
    // call admin module's (multi-modul project) user controller loginAction
    const ret = this.action('user', 'login', 'admin');
  }
}
```
#### controller.service(name, m, ...args)

Instaitate Service class, equivalent to [think.service](/doc/3.0/think.html#toc-014).
