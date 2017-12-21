## think object

Frame built-in global object `think`, easy to use in the project anytime, anywhere.

### API

#### think.app

`think.app` is an instance of Koa [Application](https://github.com/koajs/koa/blob/master/lib/application.js#L61) object, generated at system startup.

In addition, more attributes have been extended for the app.

* `think.app.think` is equivalent to the think object. Make it easy to use other methods on the think object where the app object is passed in.
* `think.app.modules` list of modules, empty array in single module project
* `think.app.controllers` save the controller file in the project, for quick follow-up call
* `think.app.logics` stores logic files in the project
* `think.app.models` stores model files in the project
* `think.app.services` stores service files
* `think.app.routers` stores custom router configuration
* `think.app.validators` stores calibration configuration
* `think.app.server` the server object after creating the HTTP service

If you want to check the value of these properties, you can do it in the `appReady` event.

```js
think.app.on('appReady', () => {
  console.log(think.app.controllers)
})
```

#### think.ROOT_PATH

The root directory of the project, other directories can be generated through this, such as:

```js
const runtimePath = path.join(think.ROOT_PATH, 'runtime/');
const viewPath = path.join(think.ROOT_PATH, 'view/');
```

#### think.APP_PATH

APP root directory, the default is `${think.ROOT_PATH}/app`. If the project doesn't need to translate, the default path is: `${think.ROOT_PATH}/src`.

#### think.env

The current runtime, equivalent to `think.app.env`, is defined in the portal file like `development.js`.

#### think.version

The current ThinkJS version number.

#### think.config(name, value, m)

* `name` {String} key to be configured
* `value` {Mixed} value to be configured
* `m` {String} module name, used in multi-module project

Read or set the configuration, this function is implemented by the [think-config](https://github.com/thinkjs/think-config) module. In the context, controller, logic can be directly through the `this.config` method to operate the configuration.

```js
// get configuration
const value1 = think.config('name');
// get the specified module configuration, valid in multi-module project
const value2 = think.config('name', undefined, 'admin');

// set configuration
think.config('name', 'value');
// specify the module setting configuration value
think.config('name', 'value', 'admin');
```

#### think.Controller

Controller base class, other controller classes inherit this class.

```js
// src/controller/user.js
module.exports = class userController extends think.Controller {
  indexAction() {

  }
}
```

#### think.Logic

Logic base class, inherited from `think.Controller`.

```js
// src/logic/user.js
module.exports = class userLogic extends think.Logic {
  indexAction() {

  }
}
```

#### think.Service

Service base class, other service classes inherit this class.

```js
// src/service/sms.js
module.exports = class extends think.Service {

}
```

#### think.service(name, m, ...args)

* `name` {String} Service name
* `m` {String} module name, valid in multi-module project
* `...args` {Array} instantiate the required parameters for the Service class. In the single-module project, the `m` parameter is supplemented with args.

Instantiate the Service class, if the exported object is not a class, then return directly.

```js
const instance1 = think.service('sms');
const instance2 = think.service('sms', 'admin');
```

#### think.beforeStartServer(fn)

* `fn` {Function} function name to register

Functions registered and executed before the service is started, fn needs to return Promise if there is an asynchronous operation.

#### think.isArray(array)

* `array` {any} determine whether the input is an array
* `return` {Boolean}

Determine whether it is an array, equivalent to `Array.isArray`.

```js
think.isArray([]); // true
think.isArray({}); // false
```

#### think.isBoolean(boolean)

* `boolean` {any}

Determine whether the input is a Boolean value

```js
think.isBoolean(false); // true
```

#### think.isInt(any)

* `any` {any}

Determine whether the input is an integer.

#### think.isNull(any)

* `any` {any}

Determine the input is `null`, you can also directly determine the `xxx == null`.

#### think.isNullOrUndefined(any)

* `any` {any}

Determine the input is `null` or `undefined`.

#### think.isNumber(number)

* `number` {any}

Determine whether the input is a number.

```js
think.isNumber(1); // true
```

#### think.isString(str)

* `str` {any}

Determine whether the input is String.

#### think.isSymbol(any)

* `any` {any}

Determine whether the type of input is Symbol.

#### think.isUndefined(any)

* `any` {any}

Determine whether the input is undefined, you can also directly determine the `xxx == undefined`.

#### think.isRegExp(reg)

* `reg` {any}

Determine whether the input is a RegExp object.

#### think.isDate(date)

* `date` {any}

Judge whether the input is a Date object.

#### think.isError(error)

* `error` {any}

Determining whether the type of input is Error.

#### think.isFunction(any)

* `any` {any}

Determine whether the type of input is Function.

#### think.isPrimitive(any)

* `any` {any}

Determine whether the input is of primitive type, including: `null`, `string`, `boolean`, `number`, `symbol`, `undefined`.

#### think.isIP(ip)

* `ip` {String}

Determine whether a string is ip address, IP v4 or IP v6, equivalent to `net.isIP`.

#### think.isBuffer(buffer)

* `buffer` {any}

Determine whether the input is a Buffer object, equivalent to `Buffer.isBuffer`.

#### think.isIPv4(ip)

* `ip` {String}

Determine whether a string is an IP v4 address, equivalent to `net.isIPv4`.

#### think.isIPv6(ip)

* `ip` {String}

Determine whether a string is an IP v6 address, equivalent to `net.isIPv6`.

#### think.isMaster

Determine whether the current process is the main process, equivalent to `cluster.isMaster`.

#### think.isObject(obj)

* `obj` {any}

Determine whether an input is an Object, by `Object.prototype.toString.call(obj)` is `[Object Object]` to determine.

```js
think.isObject({}); // true
think.isObject([]); // false
think.isObject(null); // false
```

#### think.promisify(fn, receiver)

* `fn` {Function} function will be wrapped
* `receiver` {Object} the object will be bound

This method wraps a callback function as Promise.

```js
let fn = think.promisify(fs.readFile, fs);
let data = await fn(__filename);
```

#### think.extend(target,...any)

* `target` {Object} to extend the target object
* `...any` {Object} there can be any number of objects

Deep copy of the object, if the key is the same, then the value will be overwritten with the previous value.

```js
think.extend({a: 1}, {b: 2});
// return {a:1,b:2};

think.extend({a: 1}, {a: 2});
// return {a: 2}
```

#### think.camelCase(str)

* `str` {String}

Turn snake case into camel case.

```js
think.camelCase('index_index');
// return 'indexIndex'
```

#### think.snakeCase(str)

* `str` {String}

Turn camel case into snake case.

```js
think.snakeCase('indexIndex');
// return 'index_index'
```

#### think.isNumberString(str)

* `str` {String}

Determine whether the input is a string type of number.

```js
think.isNumberString('419');
// return true
```

#### think.isTrueEmpty(any)

* `any` {any}

Determine whether the real empty, `undefined`, `null`, `''`, `NaN` is true, the other is false.

```js
think.isTrueEmpty(null);
// return true
```

#### think.isEmpty(any)

* `any` {any}

Determine whether the object is empty, `undefined`, `null` ,`''`, `NaN`, `[]`, `{}`, `0`, `false` is true, the other is false.

```js
think.isEmpty(null);
// return true
```

#### think.defer()

Generate a Deferred object.

```js
function test() {
  const defer = think.defer();
  setTimeout(function() {
    defer.reslove('1');
  },1000)
  return defer
}

test().then((result)=>{
  resut === '1'
})
```

#### think.omit(obj, props)

* `obj` {Object} the object to manipulate
* `props` {String | Array} attributes to ignore, if a string, separate multiple values with commas

Ignore some of the properties in the object and return a new object.

```js
const value = think.omit({a: 1, b: 2, c: 3}, 'a,b');
// value is {c: 3}
```

#### think.md5(str)

* `str` {String}

Calculates the md5 value of the string.

#### think.timeout(num)

* `num`{Number} time, in milliseconds

Package setTimeout as Promise.

```js
think.timeout(1000).then(()=>{
  ...
})
```


#### think.escapeHtml(str)

* `str` {String}

Escape the string of HTML, escape `<`、`>`、`"`、`'` characters.

#### think.datetime(date, format)

* `data` {Date}
* `format` {String} default 'YYYY-MM-DD HH:mm:ss'

Return a formatted date.

```js
think.datetime(1501406894849)
// return "2017-07-30 17:28:14"
```
#### think.uuid(version)

* `version` {String} v1|v4
* `return` {String}

Generate uuid strings, conforming to [RFC4122](http://www.ietf.org/rfc/rfc4122.txt) specifications, based on [uuid](https://github.com/kelektiv/node-uuid) module.

#### think.ms(str)

* `str` {String}
* `return` {Number}

把一个语义化的时间转成毫秒，如果转换失败则抛异常，使用 [ms](https://github.com/zeit/ms) 库转换。
Turn a semantic time into milliseconds, throw an exception if the conversion fails, and use the [ms](https://github.com/zeit/ms) library conversion.

```js
think.ms('2 days')  // 1d,10h,1y
// return 172800000
```

#### think.isExist(path)

* `path` {String}

Check whether the path exists.

```js
think.isExist('/usr/local/bin/node')
// return true
```

#### think.isFile(filepath)

* `filepath` {String}

Check if it is a file path.

```js
think.isFile('/usr/local/bin/node')
// return true
```

#### think.isDirectory(filepath)

* `filepath` {String}

Check if it is a folder path.

```js
think.isDirectory('/usr/local/bin')
// return true
```

#### think.chmod(path, mode)

* `path` {String}
* `mode` {String} default '0777'

Change the permissions of a file or folder.

```js
think.chmod('/usr/local/bin', '0775')
```

#### think.mkdir(path, mode)

* `path` {String} the directory to create
* `mode` {String} folder permissions, the default is `0777`
* `return` {Boolean}

Create a folder. Return true if created successfully, false otherwise.

```js
think.mkdir('/usr/local/bin/thinkjs', '0775')
```

#### think.getdirFiles(dir, prefix)

* `dir` {String} folder path
* `prefix` {String} path prefix
* `return` {Array} an array containing all files

Get all the files under the folder.

```js
think.getdirFiles('/usr/local/bin')
// return []
```

#### think.rmdir(path, reserve)

* `path` {String}
* `reserve` {Boolean} whether to retain the current folder, delete only the files under the folder

Delete folders and folders under the file, asynchronous operation.

```js
think.rmdir('/usr/local/bin/thinkjs', true).then(()=>{
  console.log('Delete completed.')
})
```

### FAQ

#### Is it recommended to use the think object in the plugin?

It is not recommended to use the think object directly in the middleware, adapter, extend, which will make the plug-in code inconvenient for unit testing. If you want to use the `app` object can be passed in, and then use the `app.think.xxx` attributes or methods on the think object.

```js
// src/config/middleware.js
module.exports = [
  {
    handle: xxx
  }
];


// xxx middleware
module.exports = (options, app) => {
  return (ctx, next) => {
    // get the list of modules for the project via app.think.modules
    const modules = app.think.modules;
    // If it is a multi-module project (single-module project length is always 0)
    if(modules.length) {

    }
  }
}
```

