## Cache
We often use diffrent kinds of cahce in application. Frame provide [think-cache](https://github.com/thinkjs/think-cache) extend and corresponding adapter to operate cache.

### Configure extend width adapter

Add following configuration in `src/config/extend.js` (or `src/common/config/extend.js` in multi-module project):

```js
const cache = require('think-cache');
module.exports = [
  cache
]
```

Modify `src/config/adapter.js`(or `src/common/config/adapter.js` in multi-module project):

```js
const fileCache = require('think-cache-file');

exports.cache = {
  type: 'file',
  common: {
    timeout: 24 * 60 * 60 * 1000 // ms
  },
  file: {
    handle: fileCache,
    cachePath: path.join(think.ROOT_PATH, 'runtime/cache'), // cache file path
    pathDepth: 1,
    gcInterval: 24 * 60 * 60 * 1000 // Interval to check if cache expires
  }
}
```
Supported cache type refer to <https://github.com/thinkjs/think-awesome#cache>。

### Inject Methods

think-cache extend will inject `think.cache`, `ctx.cache` and  `controller.cache` method, inside ctx.cache and controller.cache will use think.cache, which will read the current request's cache configure.

#### Get Cache

```js
module.exports = class extends think.Controller {
  // get
  async indexAction() {
    const data = await this.cache('name');
  }
  // specify cache type, read from redis(think-cache-redis required)
  async index2Action() {
    const data = await this.cache('name', undefined, 'redis');
  }
}
```

Normally a cache operation involed try read the cache, or get data and set the cache value if cache is missed. As a short hand, wo support the following way:

```js
module.exports = class extends think.Controller {
  // if cache exist, return it.
  // if not exist, exicute the value function, the reutrn value will be resolve and cached.
  // if value function is asynchronous, return Promise
  async indexAction() {
    const data = await this.cache('name', () => {
      return getDataFromApi();
    });
  }
}
```

#### Set Cache

```js
module.exports = class extends think.Controller {
  // set cache
  async indexAction() {
     await this.cache('name', 'value');
  }
  // set cache of redis type
  async index2Action() {
    await this.cache('name', 'value', 'redis');
  }
  // set cache of redis type
  async index2Action() {
    await this.cache('name', 'value', {
      type: 'redis',
      redis: {
        timeout: 24 * 60 * 60 * 1000
      }
    });
  }
}
```

#### delete cache

set the cache valu to `null` means delete a cache.

```js
module.exports = class extends think.Controller {
  // delete
  async indexAction() {
    await this.cache('name', null);
  }
  // delete cache of redis type
  async index2Action() {
    await this.cache('name', null, 'redis');
  }
}
```

### cache gc
Some cache container allows to set expires time, like Mencache and Redis, expired data will be deleted automaticly. But containers like File or Db don't provide this feature, in which case we need to handle the cleaning ourself.

We define `gcInterval` to specify the interval to check if cache expires, the minimum time is 1 hour. It means wo excecute the `gc` method every one hour, the clean logic define by each module and will be call by [think-gc](https://github.com/thinkjs/think-gc).

### FAQ

#### Can I use Node.js menory to cache data?
Theoretically, yes you can, but it is not suggested. Use memory to cache data will cause memory usage grows too large and affects normal user request, the loss outweights the gain.
