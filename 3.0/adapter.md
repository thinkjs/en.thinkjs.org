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
const viewConfig = think.config('view'); // 获取 view adapter 的详细配置

const nunjucks = helper.parseAdatperConfig(viewConfig); // 获取 nunjucks 的配置，默认 type 为 nunjucks
/**
{
  type: 'nunjucks',
  handle: nunjucks,
  viewPath: path.join(think.ROOT_PATH, 'view'),
  sep: '_',
  extname: '.html'
}
*/

const ejs = helper.parseAdatperConfig(viewConfig, 'ejs') // 获取 ejs 的配置
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

通过 `parseAdapterConfig` 方法就可以拿到对应类型的配置，然后就可以调用对应的 `handle`，传入配置然后执行了。

当然，配置解析并不需要使用者在项目中具体调用，一般都是在插件对应的方法里已经处理。

### Adapter 使用

Adapter 都是一类功能的不同实现，一般是不能独立使用的，而是配合对应的扩展一起使用。如：view Adapter（think-view-nunjucks、think-view-ejs）配合 [think-view](https://github.com/thinkjs/think-view) 扩展进行使用。

项目安装 think-view 扩展后，提供了对应的方法来渲染模板，但渲染不同的模板需要的模板引擎有对应的 Adapter 来实现，也就是配置中的 `handle` 字段。

### 项目中创建 Adapter

除了引入外部的 Adapter 外，项目内也可以创建 Adapter 来使用。Adapter 文件放在 `src/adapter/` 目录下（多模块项目放在 `src/common/adapter/`），如：`src/adapter/cache/xcache.js`，表示加了一个名为 `xcache` 的 cache Adapter 类型，然后该文件实现 cache 类型一样的接口即可。

实现完成后，就可以直接通过字符串引用这个 Adapter 了，如：

```js
exports.cache = {
  type: 'file',
  xcache: {
    handle: 'xcache', //这里配置字符串，项目启动时会自动查找 src/adapter/cache/xcache.js 文件
    ...
  }
}
```

### 推荐的 Adapter

框架推荐的 Adapter 为 <https://github.com/thinkjs/think-awesome#adapters>。
