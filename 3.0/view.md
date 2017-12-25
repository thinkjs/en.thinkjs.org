## View

由于某些项目下并不需要 View 的功能，所以 3.0 里并没有直接内置 View 的功能，而是通过 Extend 和 Adapter 来实现的。

### Extend 来支持 View

配置 `src/config/extend.js`，添加如下的配置，如果已经存在则不需要再添加：

```js
const view = require('think-view');
module.exports = [
  view
]
```

通过添加视图的扩展，让项目有渲染模板文件的能力，视图扩展是通过模块[think-view](https://github.com/thinkjs/think-view) 实现的。

### 配置 View Adapter

在 `src/config/adapter.js` 中添加如下的配置，如果已经存在则不需要再添加：

```js
const nunjucks = require('think-view-nunjucks');
const path = require('path');

// 视图的 adapter 名称为 view
exports.view = {
  type: 'nunjucks', // 这里指定默认的模板引擎是 nunjucks
  common: {
    viewPath: path.join(think.ROOT_PATH, 'view'), //模板文件的根目录
    sep: '_', //Controller 与 Action 之间的连接符
    extname: '.html' //模板文件扩展名
  },
  nunjucks: {
    handle: nunjucks,
    beforeRender: () => {}, // 模板渲染预处理
    options: { // 模板引擎额外的配置参数

    }
  }
}
```

这里用的模板引擎是 `nunjucks`，项目中可以根据需要修改。

### 具体使用

配置了 Extend 和 Adapter 后，就可以在 Controller 里使用了。如：

```js
module.exports = class extends think.Controller {
  indexAction(){
    this.assign('title', 'thinkjs'); //给模板赋值
    return this.display(); //渲染模板
  }
}
```

#### assign

给模板赋值。

```js
//单条赋值
this.assign('title', 'thinkjs');

//多条赋值
this.assign({
  title: 'thinkjs',
  name: 'test'
});

//获取之前赋过的值，如果不存在则为 undefined
const title = this.assign('title');

//获取所有赋的值
const assignData = this.assign();
```

#### render

获取渲染后的内容，该方法为异步方法，需要通过 async/await 处理。

```js
//根据当前请求解析的 controller 和 action 自动匹配模板文件
const content1 = await this.render();

//指定文件名
const content2 = await this.render('doc');
const content3 = await this.render('doc/detail');
const content4 = await this.render('doc_detail');

//不指定文件名但切换模板类型
const content5 = await this.render(undefined, 'ejs');

//指定文件名且切换模板类型
const content6 = await this.render('doc', 'ejs');

//切换模板类型，并配置额外的参数
//切换模板类型时，需要在 adapter 配置里配置对应的类型
const content7 = await this.render('doc', {
  type: 'ejs',
  xxx: 'yyy'
});
```

#### display

Render and output content, which is actually called a `render` method, then the contents of the rendered assigned to `ctx.body` property. This method is asynchronous and needs to be handled by async / await.


```js
// Automatically match the template file based on the controller and action being parsed for the current request
await this.display();

// specify the file name
await this.display('doc');
await this.display('doc/detail');
await this.display('doc_detail');

// don't specify the file name switch template type
await this.display(undefined, 'ejs');

// specify the file name and switch the template type
await this.display('doc', 'ejs');

// switch the template type and configure additional parameters
await this.display('doc', {
  type: 'ejs',
  xxx: 'yyy'
});
```


### Template pretreatment

Sometimes it is necessary to preprocess the template, the more common operation is to add `Filter` to `nunjucks` engine. Then you can use `beforeRender` method.

```js
const nunjucks = require('think-view-nunjucks');
const path = require('path');

exports.view = {
  type: 'nunjucks',
  common: {
    viewPath: path.join(think.ROOT_PATH, 'view'), // the root directory of the template file
    sep: '_', // connector between Controller and Action
    extname: '.html' // file extension
  },
  nunjucks: {
    handle: nunjucks,
    beforeRender(env, nunjucks, config) {
      env.addFilter('utc', time => (new Date(time)).toUTCString());
    }
  }
}
```
The parameters passed to the `beforeRender ()` method of the different template engines may be different, and the corresponding template engine view can be found in the https://github.com/thinkjs/think-awesome#view project.

### Modify the default parameters of the template engine

If you want to modify some of the parameters of the template engine, such as: modify the left and right delimiters, you can do it through `options`:

```js
const nunjucks = require('think-view-nunjucks');
const path = require('path');

exports.view = {
  type: 'nunjucks',
  common: {
    viewPath: path.join(think.ROOT_PATH, 'view'), // the root directory of the template file
    sep: '_', // connector between Controller and Action
    extname: '.html' // file extension
  },
  nunjucks: {
    handle: nunjucks,
    options: {
      tags: { // modify the delimiter-related parameters
        blockStart: '<%',
        blockEnd: '%>',
        variableStart: '<$',
        variableEnd: '$>',
        commentStart: '<#',
        commentEnd: '#>'
      }
    }
  }
}
```

### Default injected parameters

In addition to manually registering some variables to the template via the `assign` method, the system automatically injects the `controller`, `config` and `ctx` variables when rendering the template so that it can be used directly in the template.

#### controller

The current controller instance, you can directly call the properties and methods on the controller in the template.

```
{{ if controller.type === 'xx' }}
  <p>current type is xx</p>
{{ endif }}
```

For example, using the `nunjucks` template engine, if you want to call the method in the controller, then the method must be a **synchronization method**.

#### config

All configuration, in the template can be directly through the `config.xxx` to get the configuration information, if the attribute does not exist, then return `undefined`.

#### ctx

Context object of the current request. In the template, you can call its properties directly through `ctx.xxx` or call its methods through `ctx.yyy()`.

If it is to call the method, then the method must be a **synchronization method**.

### Supported template engines

Currently officially supported template engine are: [pug](https://github.com/thinkjs/think-view-pug)、[nunjucks](https://github.com/thinkjs/think-view-nunjucks)、[handlebars](https://github.com/thinkjs/think-view-handlebars)、[ejs](https://github.com/thinkjs/think-view-ejs)。

If you implement the new template engine support, welcome to submit here: <https://github.com/thinkjs/think-awesome#view>。

### FAQ

#### Why after calling the display method is still 404 error?

Occasionally, the `display` method is called in Action, but the page still shows a 404 error:

```
NotFoundError: url `/index/page` not found.
```

This is because the `display` method is an asynchronous method with no await or no return in front of it. The correct usage is:

```js
module.exports = class extends think.Controller {
  indexAction() {
    return this.display(); // return the display asynchronously by return
  }
}
```

```js
module.exports = class extends think.Controller {
  async indexAction() {
    await this.display(); // wait for the display method to return via await
  }
}
```

If the `display` method is called in an async method, you need to wrap the async method to Promise and return it.

#### How to close the view function?

Some projects just provide API interface function, don't need template rendering. When you create a project, the default view expansion is loaded. If you don't need the view function, you can modify `src/config/extend.js` to remove the view expansion. Modify `src/config/adapter.js` to remove the view adapter configuration.

#### How to use the session or cache function in the template?

Sometimes need to get the session or cache related information in the template, but because the operation of the session and cache are asynchronous, so you can't directly call `controller.session` to operate, you need to get the data in the Action and then assign the value in the template, Such as:

```js
module.exports = class extends think.Controller {
  async indexAction() {
    const userInfo = await this.session('userInfo');
    this.assign('userInfo', userInfo);
  }
}
```
After getting `userInfo` and assigning it, you can get the corresponding value from `userInfo.xxx` in the template.
