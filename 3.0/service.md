## Service

In the project, in addition to query the database and other operations, sometimes also need to call some remote interfaces, such as: call GitHub interface, call sending text messages interface and so on.

These functions aren't appropriate inside the model, so the framework provides Service to solve such problems.

### Create Service document

Service files stored in `src/service/`(`src/common/service/` in multi-module project) directory, the contents of the file as follows:

```js
module.exports = class extends think.Service {
  constructor() {

  }
  xxx() {

  }
}
```

Service inherits `think.Service` base class, but the base class does not provide any function, you can use Extend to expand.

You can create the service file in the root directory of the project through `thinkjs service xxx` command, which supports multi-level directories.

### Instantiate the Service class

The `think.service` method can be used to instantiate a Service class. In the controller, ctx also has a corresponding `service` method, such as `ctx.service`, `controller.service`, which are shortcuts to think.service .

When the project starts, it will scan all the services under the project file, and stored in the object `think.app.services`. Instantiation from the object to find the corresponding class file, if not found an error.

#### Instantiation without parameter class

```js
// src/service/sms.js
module.exports = class extends think.Service {
  xxx() {

  }
}

// instantiation with no parameter
const sms = think.service('sms');
sms.xxx();
```

#### Instantiation with parameter class

```js
// src/service/sms.js
module.exports = class extends think.Service {
  constructor(key, secret) {
    super();
    this.key = key;
    this.secret = secret;
  }
  xxx() {

  }
}

// instantiation with parameter
const sms = think.service('sms', key, secret);
sms.xxx();
```

#### Instantiation in multi-module project

```js
// src/home/service/sms.js
module.exports = class extends think.Service {
  constructor(key, secret) {
    super();
    this.key = key;
    this.secret = secret;
  }
  xxx() {

  }
}

// specified from home to find the service class
const sms = think.service('sms', 'home', key, secret);
```

#### Instantiation of multi-level directory

```js
// src/service/aaa/sms.js
module.exports = class extends think.Service {
  xxx() {

  }
}

const sms = think.servie('aaa/sms');
```

### Extending Service class method

The base class `think.Service` doesn't provide any method, but you need to use many common methods in fact, such as: get data from the remote interface module, update data to the database after the data processing operations. At this point you can enhance the `think.Service` class through the corresponding extensions, such as:

* [think-fetch](https://github.com/thinkjs/think-fetch) module allows `think.Service` class to have a `fetch` method so it's easy to get remote data.
* [think-model](https://github.com/thinkjs/think-model) module makes `think.Service` class have a `model` method, which allows for quick manipulation of the database.

These modules are Extend that enhance the ability of the `think.Service` class.

Of course, the project can also extend `think.Service` class according to the need, such as:

```js
// src/extend/service.js
module.exports = {
  getDataFromApi() {

  }
}
```

Enhance the ability of the `think.Service` class by adding the corresponding method in the extension file `src/extend/service.js` (`src/common/extend/service.js` in multi-module project). Then this can be used directly in `src/service/xxx.js`.

```js
// src/service/sms.js
module.exports = class extends think.Service {
  async xxx() {
    const data = await this.getDataFromApi(); //Here to visit extended method in extended/service.js
  }
}
```

If these extension methods are more general, then you can organize them into a `Extend` module release, other projects want to use just introduce this module. For details, see the [Extend/扩展](/doc/3.0/extend.html).
