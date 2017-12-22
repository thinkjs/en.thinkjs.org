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

Template configuration from the original `src/common/config/view.js` migrated to `src/config/config.js`, the configuration method is basically the same as before.

The older version of the `preRender()` method is deprecated and the new method is named `beforeRender()`. `nunjucks` template engine parameter order from the original `preRender(nunjucks, env, config)` to `beforeRender(env, nunjucks, config)`.

#### Prevent subsequent execution

Removed methods such as `think.prevent` that prevented subsequent execution, replaced with return false in `__before`、`xxxAction`、`__after` to prevent the subsequent code to continue execution.

#### Error handling

When 2.x creates a project, a corresponding error.js file is created to handle the error. 3.0 instead use middleware [think-trace](https://github.com/thinkjs/think-trace) processing.

### Upgrade recommendations

Due to 3.0 changes a lot of things, it is not easy to upgrade based on the simple modification of the original project code. It is recommended to create the project with the new scaffolding tool and then copy the previous code one by one into the new project.

