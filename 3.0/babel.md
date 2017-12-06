## Babel Transpile


Framework requires the minimum Node.js version to `6.0.0`, which doesn't support `async/await`, so we need to use [Babel](https://babeljs.io/) transpiler to enable this feature.

Babel will transpile `src/` and output to `app/`, adding `sourceMap` file respectively.

### Babel Transpile Rule

Default preset is [babel-preset-think-node](https://github.com/thinkjs/babel-preset-think-node), to configure in `development.js` entry file:

```js
const Application = require('thinkjs');
const babel = require('think-babel');
const watcher = require('think-watcher');
const notifier = require('node-notifier');

const instance = new Application({
  ROOT_PATH: __dirname,
  watcher: watcher,
  transpiler: [babel, {
    presets: ['think-node'] // default preset babel-preset-think-node
  }],
  notifier: notifier.notify.bind(notifier),
  env: 'development'
});

instance.run();

```

babel-preset-think-node only transpile [es2015-modules-commonjs](http://babeljs.io/docs/plugins/transform-es2015-modules-commonjs/)、[exponentiation-operator](http://babeljs.io/docs/plugins/transform-exponentiation-operator/)、[trailing-function-commas](http://babeljs.io/docs/plugins/syntax-trailing-function-commas/)、[async-to-generator](http://babeljs.io/docs/plugins/transform-async-to-generator/)、[object-rest-spread](http://babeljs.io/docs/plugins/transform-object-rest-spread/), if above transpile rules can't satisfy your requirement, you can create your own babel preset.


### Use without Babel

If your Node version is greater than `7.6.0`(suggest 8.x.x LTS), which already supports `async/await`, you can turn off Babel transplile.

#### By CLI command

You can use `-w` param to turn off Babel.

```sh
thinkjs new demo -w;
```

#### By code

Existing project can turn off Babel by delete some code:

* Entry file configure with Babel (development.js)

    ```js
    const Application = require('thinkjs');
    const babel = require('think-babel');
    const watcher = require('think-watcher');
    const notifier = require('node-notifier');

    const instance = new Application({
      ROOT_PATH: __dirname,
      watcher: watcher, //watch file change
      transpiler: [babel, {  // transpiler
        presets: ['think-node']
      }],
      notifier: notifier.notify.bind(notifier), //通知器，当转译报错时如何通知
      env: 'development'
    });

    instance.run();
    ```

* Entry file without Babel（development.js）

    ```js
    const Application = require('thinkjs');
    const watcher = require('think-watcher');
    const instance = new Application({
      ROOT_PATH: __dirname,
      watcher: watcher,
      env: 'development'
    });

    instance.run();
    ```

The non-babel version just remove the `transpiler` and `notifier` configure.