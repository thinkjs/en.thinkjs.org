## Extend

Although the frame provide a lot of features, but it is far from enough in actual project development. 3.0 provide extend mechanism to allow extend framework. Supported extend type is: `think`、`application`、`context`、`request`、`response`、`controller`、`logic` and `service`.

Some framework build-in feature is also implemented with extend, such as `Session`, `Cache`.

### Extend Configuration

Extend config file path is `src/config/extend.js` (or `src/common/config/extend.js` in [multi-module](/doc/3.0/multi_module.html) project), in array format:

```js
const view = require('think-view');

module.exports = [
  view //make application support view
];
```


As above, through the view extend the framework support template rendering functions, Controller class has a `assign`,` display` and other methods.

### Project Extend

In addition to extend framework funtionality by introducing an external framework, it is also possible to add extend object in project, of which file places in `src/extend/` directory (or `src/common/extend/` for [multi-module project](/doc/3.0/multi_module.html)).

* `src/extend/think.js` extend think object，think.xxx
* `src/extend/application.js` extend Koa's app object（think.app）
* `src/extend/request.js` extend Koa's request object（think.app.request）
* `src/extend/response.js` extend Koa's response object（think.app.response）
* `src/extend/context.js` extend ctx object（think.app.context）
* `src/extend/controller.js` extend Controller class（think.Controller）
* `src/extend/logic.js` extend Logic class - Logic inherit Controller class so that it has all controller methods
* `src/extend/service.js` extend Service class（think.Service）

If we want to add `isMobile` method to `ctx` to determine if current request is send from mobile, as follows:

```
// src/extend/context.js
module.exports = {
  isMobile(){
    const userAgent = this.userAgent.toLowerCase();
    const mList = ['iphone', 'android'];
    return mList.some(item => userAgent.indexOf(item) > -1);
  }
}
```

So that we can use `ctx.isMobile()` to judge whether it is a mobile request. of cause, this method has no parameters, we can make a `getter`.

```
// src/extend/context.js
module.exports = {
  get isMobile(){
    const userAgent = this.userAgent.toLowerCase();
    const mList = ['iphone', 'android'];
    return mList.some(item => userAgent.indexOf(item) > -1);
  }
}
```

Now we can use `this.isMobile` in ctx, or `ctx.isMobile`, such as: user `this.ctx.isMobile` in controller.
If we also want to use `this.isMobile` in controller, just extend controller to an `isMobile` property.

```
// src/extend/controller.js
module.exports = {
  get isMobile(){
    return this.ctx.isMobile;
  }
}
```

By doing this, we can use `this.isMobile` in controller.
If we want this extend to be available in other project, you can create a npm module and export the extend object in module's entry file.

```
const controllerExtend = require('./controller.js');
const contextExtend = require('./context.js');

// module entry file
module.exports = {
  controller: controllerExtend,
  context: contextExtend
}
```

### Use app in extend
Some Extend need to access `app` object, for that we can export the extend as function, so that we can pass the `app` instance on configuration.

```
// src/config/extend.js
const model = require('think-model');
module.exports = [
  model(think.app) //pass think.app into model extend
];
```

当然除了传 `app` 对象，也可以根据需要传递其他对象。

### Supported extend list

Refer to <https://github.com/thinkjs/think-awesome#extends> for supported Extend list, of cause if you create a good Extend, Pull Request is welcome.

### FAQ

#### How to deal with methods from multiple extends are duplicated？

If more than one extension provides a duplicate name, the latter extension overrides the previous extension, so you can decide how to override it based on the order of the adjustments. Another: use the meaningful method name as soon as you create the extension, do not use too simple method names.

The name of the extension can not be the same as the name of a method built into the framework, which can cause some strange problems.