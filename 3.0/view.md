## View

Because some projects don't need the View function, so in 3.0 doesn't directly built-in View function, but through Extend and Adapter to achieve.

### Extend to support View

Configure `src/config/extend.js`, add the following configuration, you don't need to add if it already exists:

```js
const view = require('think-view');
module.exports = [
  view
]
```

By adding the view's extension, the project has the ability to render the template file, the view is extended through the [think-view](https://github.com/thinkjs/think-view) module.

### Configure View Adapter

Add the following configuration to `src/config/adapter.js` and don't need to be added if it already exists:

```js
const nunjucks = require('think-view-nunjucks');
const path = require('path');

// the view adapter's name is view
exports.view = {
  type: 'nunjucks', // the default template engine specified here is nunjucks
  common: {
    viewPath: path.join(think.ROOT_PATH, 'view'), // the root directory of the template file
    sep: '_', // connector between Controller and Action
    extname: '.html' // template file extension
  },
  nunjucks: {
    handle: nunjucks,
    beforeRender: () => {}, // template rendering preprocessing
    options: { // template engine additional configuration parameters

    }
  }
}
```

The template engine here is `nunjucks`, which can be modified as needed.

### Instructions for use

After configuring Extend and Adapter, it can be used in Controller. Such as:

```js
module.exports = class extends think.Controller {
  indexAction(){
    this.assign('title', 'thinkjs'); // assign the template
    return this.display(); // render template
  }
}
```

#### assign

Assign the template.

```js
// single assignment
this.assign('title', 'thinkjs');

// Multiple assignment
this.assign({
  title: 'thinkjs',
  name: 'test'
});

// get the value assigned before, or undefined if it does not exist
const title = this.assign('title');

// get all the assigned values
const assignData = this.assign();
```

#### render

Get the rendered content, which is an asynchronous method that needs to be handled by async / await.

```js
// automatically match the template file based on the controller and action being parsed for the current request
const content1 = await this.render();

// specify the file name
const content2 = await this.render('doc');
const content3 = await this.render('doc/detail');
const content4 = await this.render('doc_detail');

// don't specify the file name switch template type
const content5 = await this.render(undefined, 'ejs');

// specify the file name and switch the template type
const content6 = await this.render('doc', 'ejs');

// switch the template type and configure additional parameters
// when switching the template type, you need to configure the corresponding type in the adapter configuration
const content7 = await this.render('doc', {
  type: 'ejs',
  xxx: 'yyy'
});
```

#### display

Render and output content, which is actually called a `render` method, then the contents of the rendered assigned to `ctx.body` property. This method is asynchronous and needs to be handled by async / await.


```js
// automatically match the template file based on the controller and action being parsed for the current request
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

Project sometimes need to get the session or cache related information in the template, but because the operation of the session and cache are asynchronous, so you can't directly call `controller.session` to operate, you need to get the data in the Action and then assign the value in the template, Such as:

```js
module.exports = class extends think.Controller {
  async indexAction() {
    const userInfo = await this.session('userInfo');
    this.assign('userInfo', userInfo);
  }
}
```
After getting `userInfo` and assigning it, you can get the corresponding value from `userInfo.xxx` in the template.
