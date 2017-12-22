## Upgrade guide

This document is from 2.x to 3.x and can not be smoothly upgraded due to the large interface changes in this upgrade. This document is more about the interface change guide.

### Change list
#### Core changes

3.0 abandoned the core architecture of 2.x, built on Koa 2.x and compatible with all the features in Koa. The main changes are:

* the previous `http` object was changed to` ctx` object
* execution is done entirely by calling `middleware`
* many of the built-in features in the framework are no longer built-in by default and can be extended with extensions

#### Project begining

When the 2.x project starts, all the files in the `src/bootstrap/` directory are automatically loaded. 3.0 no longer automatically load all the files, but instead:

* load the `src/boostrap/master.js` file in the Master process
* load the `src/boostrap/worker.js` file in the Worker process

If you want to load other files, you can use the `require` method in the corresponding file to import it.

#### Configuration

2.x will automatically load all files in the `src/config/` directory, 3.0 to load the corresponding file according to the function.

#### hook and middleware

Remove hooks and middleware in 2.x, change to middleware in Koa, manage middleware in `src/config/middleware.js` config file.

#### Controller

Change the base class `think.controller.base` to `think.Controller` and remove the `think.controller.rest` class.

#### Model

Change the base class `think.model.base` to `think.Model`。

#### View

模板的配置由原来的 `src/common/config/view.js` 迁移至 `src/config/config.js` 中，配置方法和之前基本一致。

其中老版本的 `preRender()` 方法已经废弃，新方法名为 `beforeRender()`。`nunjucks` 模板引擎的参数顺序由原来的 `preRender(nunjucks, env, config)` 修改为 `beforeRender(env, nunjucks, config)`。

#### 阻止后续执行

移除了 `think.prevent` 等阻止后续执行的方法，替换为在 `__before`、`xxxAction`、`__after` 中返回 `false` 来阻止后续代码继续执行。

#### 错误处理

2.x 创建项目时，会创建对应的 error.js 文件用来处理错误。3.0 里改为使用中间件 [think-trace](https://github.com/thinkjs/think-trace) 处理。

### 升级建议

由于 3.0 改动了很多东西，所以不太容易基于原有项目代码简单修改来升级。建议使用新的脚手架工具创建项目，然后一一将之前的代码拷贝到新项目中进行修改。

