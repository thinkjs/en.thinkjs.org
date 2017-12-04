## Adapter

`Adapter` pattern allows the interface of an existing class to be used as another interface. It is often used to make existing classes work with others without modifying their source code. For example, to allow easy switch databases and template engines by just change the adapter configuration. `Adapter` often used with `Extend`.

ThinkJS provide rich Adapters, covering functions like `View`, `Model`, `Cache`, `Session` and `Websocket`. You can extend you project and choose adapter on need, or even implement one.

### Adapter Configuration

Adapter‘s configuration file is `src/config/adapter.js`（or `src/common/config/adapter.js` is multi-module mode），example usage as follows:

```js
const nunjucks = require('think-view-nunjucks');
const ejs = require('think-view-ejs');
const path = require('path');

exports.view = {
  type: 'nunjucks', // default template engine 'nunjucks'
  common: { // generall settings
    viewPath: path.join(think.ROOT_PATH, 'view'),
    sep: '_',
    extname: '.html'
  },
  nunjucks: { // nunjucks settings
    handle: nunjucks
  },
  ejs: { // ejs settings
    handle: ejs,
    viewPath: path.join(think.ROOT_PATH, 'view/ejs/'),
  }
}

exports.cache = {
  ...
}
```

* `type` define the default Adapter, can be changed on runtime by param
* `common` common settings which will be merged into each adapter type
* `nunjucks` `ejs` specific Adapter settings，the final settings would be each settings merged with common
* `handle` the handler function for a specific type

Adapter supports runtime environment configuration. We normally use different database in production and development environment, to achieve that by create `adapter.development.js` and `adapter.production.js` with different settings, ThinkJS will load the corresponding configure file and apply merging strategy on system start.

Suppose in production environment, configuration file `adapter.production.js` and `adapter.js` will be merged into the final adapter settings.

Adapter configure will be load and merge on application start, the above configure will produce settings as bellow:

```js
exports.view = {
  type: 'nunjucks', // default nunjucks
  nunjucks: { 
    handle: nunjucks,
    viewPath: path.join(think.ROOT_PATH, 'view'),
    sep: '_',
    extname: '.html'
  },
  ejs: {
    handle: ejs,
    viewPath: path.join(think.ROOT_PATH, 'view/ejs/'),
    viewPath: path.join(think.ROOT_PATH, 'view'),
    sep: '_',
    extname: '.html'
  }
}
```

We can see common settings is merged into numjucks and ejs, on runtime we just get adapter configure by type and no merge is needed.

### Adapter Parsing

We need to parse adapter configure and choose one to work with. Take above as example, we have adapter setting with nunjucks and ejs, but we only use one of them.

Adapter configure is parse by `parseAdapterConfig` method in [think-helper](https://github.com/thinkjs/think-helper) module，as bellow：

```js
const helper = require('think-helper');
const viewConfig = think.config('view'); // get view adapter configure

const nunjucks = helper.parseAdatperConfig(viewConfig); // to get the default adapter configure
/**
{
  type: 'nunjucks',
  handle: nunjucks,
  viewPath: path.join(think.ROOT_PATH, 'view'),
  sep: '_',
  extname: '.html'
}
*/

const ejs = helper.parseAdatperConfig(viewConfig, 'ejs') 
/**
{
  handle: ejs,
  type: 'ejs',
  viewPath: path.join(think.ROOT_PATH, 'view/ejs/'),
  viewPath: path.join(think.ROOT_PATH, 'view'),
  sep: '_',
  extname: '.html'
}
*/
```

You can call `parseAdapterConfig` to get configure and use `handle` to run.
But most of the time you don't need to do this by yourself, the Extend method normally will do the job.

### Adapter Usage

Adapter usually work with Extend and implement a type of interface with different implementation. For example: view Adapter（think-view-nunjucks、think-view-ejs）work with [think-view](https://github.com/thinkjs/think-view) Extend。
Extend think-view provide neccesary method for template rendering, inside the render method will call Adapter's `handle` to readlly do the job for a specific template engine.

### Create Project Adapter
Besides third-party Adapter, we can create Adapter in project. Put your implementation file in `src/adapter/` folder (or `src/common/adapter/` for multi-module project), take `src/adapter/cache/xcache.js` for instance，ThinkJS will accept it as cache Adapter of type `xcache`，this file should implement the cache interface。

The last thing is to turn it on:
```js
exports.cache = {
  type: 'file',
  xcache: {
    handle: 'xcache', // set handle to string, so that system will match to src/adapter/cache/xcache.js file as implementation.
    ...
  }
}
```

### Recommended Adapter
Bellow link show all available framework adapter list <https://github.com/thinkjs/think-awesome#adapters>。
