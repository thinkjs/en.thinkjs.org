## Router

When a user accesses an address, you need a corresponding logic to process it. The traditional approach, a request corresponding to a file, such as access is `/user/about.php`, then in the corresponding directory of the project there must have an entity file named `/user/about.php`. Although this approach can solve the problem, but will lead to a lot of files in the project, and may be a lot of files logical function is relatively simple.

In the current MVC development model, these problems are usually solved by router. The solution is: first map all the user's requests to an entry file (eg: `index.php`), and then the framework parses the address of the current request, parses out the corresponding function to perform according to the configuration or convention, and finally calls and responds user's request.

Since Node.js is self-starting HTTP(S) service, requests from users have been naturally aggregated into an entry, making it easier to handle router mappings.

In ThinkJS, when a user accesses a URL, it finally responds by the specific action in the controller. So we need to parse out the controller and action of the URL, this parsing is achieved through [think-router](https://github.com/thinkjs/think-router) module.

### Router configuration

`think-router` is a middleware, which has been added to the `src/config/middleware.js` file by default when the project is created, where `options` supports the following parameters:

* `defaultModule` {String} the default module name in multi-module project, the default value is `home`
* `defaultController` {String} the default controller name, the default value is `index`
* `defaultAction` {String} the default operation name, the default value is `index`
* `prefix` {Array} the pathname default prefix to remove, the default value is `[]`
* `suffix` {Array} the pathname default suffix to remove, the default value is `['.html']`
* `enableDefaultRouter` {Boolean} whether or not to use the default router parsing in the case of mismatch, the default value is `true`
* `optimizeHomepageRouter` {Boolean} whether to optimize the home page, the default value is true (if the access address is the home page, then no custom router match)
* `subdomainOffset` {Number} subdomain mapping offset, the default value is `2`
* `subdomain` {Object|Array} subdomain mapping list, the default value is `{}`
* `denyModules` {Array} a list of modules that are forbidden to access in multi-module project, the default value is `[]`

The specific default configuration is as follows, in the project can be modified as needed:

```js
module.exports = [
  {
    handle: 'router',
    options: {
      defaultModule: 'home',
      defaultController: 'index',
      defaultAction: 'index',
      prefix: [],
      suffix: ['.html'],
      enableDefaultRouter: true,
      subdomainOffset: 2,
      subdomain: {},
      denyModules: []
    }
  }
];
```

### Path preprocessing

When the user accesses the service, the initial `pathname` can be obtained through the` ctx.url` attribute. For example, to access the current page `https://www.thinkjs.org/en/doc/3.0/router.html` , The initial pathname is `/zh-cn/doc/3.0/router.html`.

To facilitate the parsing of the corresponding controller and action by pathname, pathname needs to be preprocessed.

#### prefix & suffix

Sometimes for search engine optimization or for some other reason, there's something more to add to the URL. For example, the current page is a dynamic page. For SEO, the `.html` suffix is appended to the URL to pretend that the page is a static page, but` .html` is useless for router parsing and needs to be removed.

At this time, you can use the `prefix` and `suffix` configuration to remove some of the specific value of the front or rear, such as:

```js
{
  prefix: [],
  suffix: ['.html'],
}
```

If you visit the URL is `http://www.thinkjs.org/`, then finally get the pure `pathname` is the string `/`.

#### Subdomain mapping

When the project is more complex, you may want to deploy different functions under different domains, but the code is still in the same project. At this time, you can do this through subdomain mapping:

```js
{
  subdomainOffset: 2, // domain offset
  subdomain: { // subdomain mapping detailed configuration
    'bbb,aaa': 'aaa'
  }
}
```

When making a subdomain mapping, you need to parse out the subdomain of the current domain name. This time you need to use the domain offset `subdomainOffset`, which the default value is 2. For example: if the domain is `aaa.bbb.example.com`, the parsed subdomain list is `["bbb", "aaa"]`. When the domain offset is 3, the parsed subdomain list is `["aaa"]`. The parsed value saved in the `ctx.subdomains` attribute, the parsed `ctx.subdomains` is always an empty array if the current domain is an IP.

When making a subdomain match, `ctx.subdomains` is converted to a string (use `join(",")`) and then matched with the `subdomain` configuration. If the configuration in `subdomain` is matched then the corresponding value prefix is appended to the `pathname` value. For example, when accessing `http://aaa.bbb.example.com/api_lib/inbox/123`, the resulting pathname will be`/aaa/api_lib/inbox/123`, because `'bbb,aaa': 'aaa'` is configured. The matching order is backward matching according to the configuration, and if it matches, the subsequent matching will be terminated.

If the subdomain configuration is an array, then the array will be automatically converted into objects for later matching.

```js
subdomain: ['admin', 'user']

// convert to
subdomain: {
  admin: 'admin',
  user: 'user'
}
```

### Router analysis

After preprocessing by `prefix & suffix` and `subdomain`, we get the `pathname` to be parsed later. The default router parsing rules is `/controller/action`. If it is a multi-module project, the rule is `/module/controller/action`, and parse out the corresponding `module`,` controller`, `action` values according to this rule.

If the controller has children, then it will match the child controller first, and then match the action.

| pathname  | project type  | child controller  |  module | controller  | action | note |
|---|---|---|---|---|---|---|
| / | single module | no | | index | index | controller, action is the default configuration |
| /user | single module | no | | user | index | action is the default configuration |
| /user/login | single module | no | | user | login |  |
| /console/user/login | single module | yes | | console/user | login | have child controller console / user |
| /console/user/login/aaa/bbb | single module | yes | | console/user | login | the remaining aaa / bbb no longer parse |
| /admin/user | multi-module | no | admin | user | index | multi-module project, there is a module named admin |
| /admin/console/user/login | multi-module | yes | admin | console/user | login | | |

module, controller, action parsed content are stored in `ctx.module`, `ctx.controller`, `ctx.action`, to facilitate the follow-up call processing. If you don't want the default router resolution, you can turn it off by configuring `enableDefaultRouter: false`.

### Custom router rules

Although the default route resolution to meet the requirement, but sometimes it will lead to the URL doesn't look elegant enough. We also prefer URL to be shorter, which will be more conducive to memory and dissemination. The framework provides custom router to handle this requirement.

Custom router rule configuration file is `src/config/router.js` (`src/common/config/router.js` in multi-module project), and the router rule is a two-dimensional array:

```js
module.exports = [
  [/libs\/(.*)/i, '/libs/:1', 'get'],
  [/fonts\/(.*)/i, '/fonts/:1', 'get,post'],
];
```

Each router rule is also an array, the items in the array correspond to: `match`, `pathname`, `method`, `options`:

* `match` {String | RegExp} pathname matching rules, can be a string or regular. If it is a string, it will be reverted to regular via the [path-to-regexp](https://github.com/pillarjs/path-to-regexp) module
* `pathname` {String} match the mapped pathname, follow-up will be based on the mapping pathname to resolve the corresponding controller, action
* `method` {String} the type of request supported by this router rule, defaults to all. Multiple request types are separated by commas, such as: `get, post`

* `options` {Object} additional options, such as: jump to specify statusCode

Custom routes are saved to the `think.app.routers` object when the service is started, and the matching rule of the route is: matching one by one from the front to the back, and not matching backwards if the rule is hit.

#### Get the matched value from the match

When configuring a rule, you sometimes need to obtain the matched value from the match in the pathname. In this case, you can obtain it through string matching or regular grouping.

##### String router


```js
module.exports = [
  ['/user/:name', 'user']
]
```

The string matching format is `:name`. After matching this router, it will get the value corresponding to `:name`, and finally it will be converted to the corresponding parameter for later retrieval.

For the above router, if the access path is `/user/thinkjs`, `:name` matches the value of `thinkjs` and then appends a parameter named `name` which can be passed to `this.get("name")` to get this parameter. Of course, `pathname` can refer to `:name`, such as:

```js
module.exports = [
  ['/user/:name', 'user/info/:name']
]
```

##### Regular router

```js
module.exports = [
  [\/user\/(\w+)/, 'user?name=:1']
]
```

For the above router, if the access path is `/user/thinkjs`, then the regular group `(\w+)` matches the value `thinkjs` so that the second parameter can be passed `:1` to get this value. For multiple groups in the regular, then you can get the corresponding matching value by `:1`, `:2`, `:3`.

#### Redirect

Sometimes after the project has been refactored several times, some changes may occur to the URL address. In order to be compatible with the previous URL, it is generally required to make the previous URL jump to the new URL. This can be done by setting `method` to `redirect`.

```js
module.exporst = [
  ['/usersettings', '/user/setting', 'redirect', {statusCode: 301}]
]
```

When the access address is `/usersettings`, it will automatically jump to `/user/setting` and specify the statusCode of this request as 301.

#### RESTful

When you need to provide RESTful API, you can do it by using a custom router. For related documents, see [RESTful API](/doc/3.0/rest.html).

### Add custom router dynamically

Sometimes we need to develop some highly customized systems, such as: universal CMS system, these systems need generally configure some page access rules. At this time some custom router shouldn't be hard-code, but need to save the background configuration rules in the database, and then dynamically configure custom router rules.

At this point you can read the latest custom router rules from the database before starting the service via the `think.beforeStartServer` method and then through the `routerChange` event processing.

```js
// src/bootstrap/worker.js

think.beforeStartServer(async () => {
  const config = think.model('config');
  // save all custom routes on the data of the field named router
  const data = await config.where({key: 'router'}).find();
  const routers = JSON.parse(data.value);
  // trigger the routerChange event to set the new custom router to the think.app.routers object
  // the format of routers is the same as the custom router format, a two-dimensional array
  think.app.emit('routerChange', routers);
})

```

### FAQ

#### How to check the current address resolution controller and action corresponding to what?

The parsed controller and action are stored in `ctx.controller` and `ctx.action` respectively. Sometimes we want to know the controller and action which are finally parsed by the current access path. In this case, we can use `debug` to quickly see.

```bash
# windows cmd
set DEBUG=think-router && npm start

# windows powershell
$env:DEBUG="think-router"
npm start

# Linux and Mac
DEBUG=think-router npm start
```

[think-router](https://github.com/thinkjs/think-router) print the relevant debugging information in the router resolution, you need to open through the `DEBUG=think-router`. Then you can see the following debugging information in the console:

```
think-router matchedRule: {"match":{"keys":[]},"path":"console/service/func","method":"GET","options":{},"query":{}} +53ms
think-router RouterParser: path=/console/service/func, module=, controller=console/service, action=func, query={} +0ms
```

`matchedRule` is the hit custom router, and `RouterParser` is the parsed value.

Of course, through the debug information can quickly find out the reasons for sometimes some custom router doesn't take effect.

#### How to optimize the performance of custom router matching?

As the custom router is matched from front to back, until the rule hit to stop matching.If the rules are at the end, you need to match the rules in front, which may be a bit slow. In this case, we can combine the traffic of each interface, put the important router rules in front, and the unimportant ones in the back to improve the performance.

#### Regular router suggestion

For regular router, the default is not strict matching, so there may be regular performance problems, and may easily affect other router, this time through `^` and `$` for a strict match.

```js
module.exports = [
  [/^\/user$/, 'user']
]
```

For the above router, this rule will be hit only if the access address is `/ user`, which will reduce the impact on other router. If you remove `^` and `$`, accessing `/console/user/thinkjs` will also hit the above route. In fact, we may have written another rule to match this address, but it is hit early by this rule, This has brought some difficulties for development.

#### Can I use a third-party router parser?

The default router parser of ThinkJS is [think-router](https://github.com/thinkjs/think-router), if you want to use a third-party router parser instead, you can replace the router configuration in `src/config/middleware.js` with the corresponding module, and then store the parsed value of module, controller and action in the `ctx` object for subsequent middleware processing.

```js
// example of third-party router parser module, specific code can refer to  https://github.com/thinkjs/think-router
module.exports = (options, app) => {
  return (ctx, next) => {
    const routers = app.routers; // get all the custom routing configuration
    ...
    ctx.module = ''; // store the parsed value of module, controller and action in the `ctx`
    ctx.controller = '';
    ctx.action = '';
    return next();
  }
}
```
