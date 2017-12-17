## Router

When users access an address, you need to have a corresponding logic to deal with. The traditional approach, a request corresponding to a document, such as access `/user/about.php`, then it will be in the project corresponding directory `/user/about.php` this entity file. Although this approach can solve the problem, but will lead to a lot of files, and may be a lot of documents in the logic function is actually relatively simple.

In the current MVC development model, these problems are usually solved by router. The solution is: first map all the user's requests to an entry file (eg: `index.php`), and then the framework parses the address of the current request, parses out the corresponding function to perform according to the configuration or convention, and finally calls and responds User's request.

Because Node.js starts the HTTP(S) service itself, it has naturally aggregated user requests into an entry point, making it easier to handle route mappings.

In ThinkJS, when the user accesses a URL, the last is through the action specific controller to respond. So we need to parse out the URL corresponding to the controller and action, this resolution is through [think-router] (https://github.com/thinkjs/think-router) module.

### Router configuration


`think-router` is a middleware that has been added to the `src/config/middleware.js` file by default when it's created, and `options` supports the following parameters:

* `defaultModule` {String} Multi-module project, the default module name. The default is `home`
* `defaultController` {String} The default controller name, the default is `index`
* `defaultAction` {String} Default operation name, the default value is `index`
* `prefix` {Array} The prefix to ​​removed in pathname. The default value is `[]`
* `suffix` {Array} The suffix to ​​removed in pathname. The default is `['.html']`
* `enableDefaultRouter` {Boolean} Whether to use default route resolution in case of mismatch, the default value is `true`
* `optimizeHomepageRouter` {Boolean} Whether to optimize the homepage, the default value is `true` (If the access address is the home page, no custom router matching will be performed)
* `subdomainOffset` {Number} Offset under subdomain mapping, default is `2`
* `subdomain` {Object | Array} Subdomain mapping list, default is `{}`
* `denyModules` {Array} The list of modules that are forbidden to access in multi-module project, the default is `[]`

The specific default configuration is as follows, it can be modified as needed in the project:

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

When the user accesses the server, the initial `pathname` can be obtained through the `ctx.url` attribute. For example: Visit this page `https://www.thinkjs.org/doc/3.0/router.html`, The initial pathname is `/en/doc/3.0/router.html`.

In order to facilitate the subsequent resolution of the corresponding controller and action by pathname, pathname needs to be preprocessed.

#### prefix & suffix

Sometimes for search engine optimization or for some other reason, there's something more added to the URL. For example, the current page is a dynamic page. For SEO, the `.html` suffix is ​​appended to the URL to pretend that the page is a static page, but `.html` is useless for route resolution and should be removed.

At this time, you can use the prefix and suffix configuration to remove some of the specific value of the front or rear, such as:

```js
{
  prefix: [],
  suffix: ['.html'],
}
```

`prefix` and `subffix` are arrays, each of which can be a string or a regular expression, and stop subsequent matches after the first match. After the above `pathname` is filtered by the default configuration, get the pure pathname as `/en/doc/3.0/router`.

If you visit the URL is `http://www.thinkjs.org/`, then finally get the pure `pathname` is the string `/ `.

#### Subdomain mapping

When the project is more complex, you may want to deploy different functions under different domain names, but the code is still under one project. At this time, you can do this through sub-domain name mapping:

```js
{
  subdomainOffset: 2,
  subdomain: { // subdomain config
    'bbb,aaa': 'aaa'
  }
}
```

When doing sub-domain mapping, the need to resolve the sub-domain name of the current domain specific What is? For example, for the domain name aaa.bbb.example.com, the parsed subdomain name list is `['bbb', 'aaa']`. When the domain name offset is 3, the parsed sub-domain name list is `["aaa"]`, and the parsed value is stored in the `ctx.subdomains` attribute. If the current domain name is an IP, the parsed ctx.subdomains is always an empty array.

When making a subdomain match, `ctx.subdomains` is converted to a string (`join(",")`) and then matched with the `subdomain` configuration. If the configuration in `subdomain` is matched then the corresponding value prefix is ​​appended to the ` pathname` value. For example, when accessing `http://aaa.bbb.example.com/api_lib/inbox/123`, the resulting pathname will be `/aaa/api_lib/inbox/123` because `'bbb, aaa': 'aaa'` is configured. The matching order is backward matching according to the configuration, and if it matches, the subsequent matching will be terminated.

If the `subdomain` configuration is an array, then the array will be automatically converted into objects for later matching.

```js
subdomain: ['admin', 'user']

// 转化为
subdomain: {
  admin: 'admin',
  user: 'user'
}
```

### Router analysis

After preprocessing by `prefix & suffix` and `subdomain`, we get the `pathname` to be parsed later. The default route resolution rule is `/controller/action`. If it is a multi-module project, the rule is `/module/controller/action`, and parse out the corresponding `module`, `controller`, `action` values ​​according to this rule.

If the controller has children, then it will match the child controller first, and then match the action.

| pathname | Item Type | Child Controller | module | controller | action | Notes |
--- --- --- --- --- --- --- --- --- --- --- --- --- |
| | | Single module | None | | index | index | controller, action is the default configuration value |
| / user | Single Module | None | | user | index | action is the default value of configuration |
| / user / login | Single Module | None | | user | login | |
| / console / user / login | Single Module | Yes | | console / user | login | Yes Child Controller console / user |
/ console / user / login / aaa / bbb | Single module | Yes | | console / user | login | Remaining aaa / bbb no longer parses |
| / admin / user | Multi-module | None | admin | user | index | Multi-module project with module named admin |
| / admin / console / user / login | Multi-module | Yes | admin | console / user | login | | |


Resolve the module, controller, action, respectively, on `ctx.module`, `ctx.controller`, `ctx.action` on the follow-up call processing. If you do not want the default route resolution, you can turn it off by configuring `enableDefaultRouter: false`.

### Custom router rules

Although the default route resolution to meet the demand, but sometimes leads to the URL does not look elegant enough, we hope that the URL is relatively short, which will be more conducive to memory and dissemination. The framework provides custom router to handle this requirement.

Custom router rules configuration file for `src/config/router.js` (multi-module project on`src/common/config/router.js`), router rules for the two-dimensional array:

```js
module.exports = [
  [/libs\/(.*)/i, '/libs/:1', 'get'],
  [/fonts\/(.*)/i, '/fonts/:1', 'get,post'],
];
```
Each router rule is also an array, the items in the array correspond to: `match`, `pathname`, `method`, `options`:

* `match` {String | RegExp} pathname matching rules, can be a string or regular. If it is a string, it will be reverted to regular via the [path-to-regexp] (https://github.com/pillarjs/path-to-regexp) module
* `pathname` {String} After matching the mapped pathname, follow-up will be based on the mapping of the pathname to resolve the corresponding controller, action
* `method` {String} The request type supported by this route rule, defaults to all. Multiple request types are separated by commas, such as: get, post
* `options` {Object} Additional options, such as: specifying statusCode on jump

Custom routes Read the `think.app.routers` object at service startup. The matching rules for a route are: Matches from front to back, and no match to back if hit.

#### Get the match in the match value

When configuring a rule, sometimes it is required to obtain the matched value in pathname in pathname. In this case, you can obtain it through string matching or regular grouping.

##### String router


```js
module.exports = [
  ['/user/:name', 'user']
]
```
The string matching format is `:name`. After matching this route, it will get the value corresponding to `:name`, and finally it will be converted to the corresponding parameter for later retrieval.

For the above router, if the access path is `/user/thinkjs`,`:name` matches the value of `thinkjs` and then appends a parameter named name which can be passed to` this.get("name") `to get this parameter. Of course, `pathname` can refer to`: name`, such as:

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

For the above router, if the access path is `/user/thinkjs`, then the regular group `(\w+) `matches the value `thinkjs` so that the second parameter can be passed `:1` Get this value. For multiple groups in the regular, then you can get the corresponding matching value by `:1`, `:2`, `:3`.


#### Redirect

Sometimes after the project has been refactored several times, some changes may occur to the URL address. In order to be compatible with the previous URL, it is generally required to jump to the new URL before the URL. This can be done by setting `method` to `redirect`.

```js
module.exporst = [
  ['/usersettings', '/user/setting', 'redirect', {statusCode: 301}]
]
```
When the access address is `/usersettings`, it will automatically jump to `/user/setting` and specify the statusCode of this request as 301.

#### RESTful

Sometimes want to provide RESTful API, this time can also be done with custom Router, the relevant documents, please go to [RESTful API] (/doc/3.0/rest.html).

### Add custom Router dynamically

Sometimes we need to develop some highly customized systems, such as: Universal CMS system, these systems can generally configure some page access rules. At this time some custom Router can not write dead, but need to configure the rules of the background stored in the database, and then dynamically configure custom Router rules.

This time you can use `think.beforeStartServer` method to read the latest custom Router rules from the database before the service starts, and then through the `routerChange` event.

```js
// src/bootstrap/worker.js

think.beforeStartServer(async () => {
  const config = think.model('config');
  // Save all custom routes on the data for the router
  const data = await config.where({key: 'router'}).find();
  const routers = JSON.parse(data.value);
  // Trigger the routerChange event to set the new custom route to the think.app.routers object
  // The format of routers is the same as the custom Router format, two-dimensional array
  think.app.emit('routerChange', routers);
})

```

### FAQ

#### How to know what the controller and action are for the current address after the resolution?

Analytic controller and action were placed on the `ctx.controller` and `ctx.action`, and sometimes we want to quickly know the path of the current visit last resolved controller and action what, this time with the help of `debug` Quickly see.

```bash
# windows cmd
set DEBUG=think-router && npm start

# windows powershell
$env:DEBUG="think-router"
npm start

# Linux and Mac
DEBUG=think-router npm start
```

[think-router] (https://github.com/thinkjs/think-router) In the analysis of Router print the relevant debugging information, through `DEBUG=think-router` to open, will be opened in the console look To the following debugging information:

```
think-router matchedRule: {"match":{"keys":[]},"path":"console/service/func","method":"GET","options":{},"query":{}} +53ms
think-router RouterParser: path=/console/service/func, module=, controller=console/service, action=func, query={} +0ms
```

`matchedRule` hit which custom route, `RouterParser` parse out the value.

Of course, through the debug information can also quickly locate some custom Router sometimes fails to take effect.

#### How to optimize custom route matching performance?

As the custom Router is matched from front to back, until the rule hits to stop matching the future, if the rules are very dependent, then you need to go ahead of the rules again, this may be a bit slow. This time can be combined with the traffic of each interface, the important route on the front, not on the back of the route to improve performance.

#### Regular Router suggestions

For regular routes, the default is not strict matching, so there may be regular performance problems, and may easily affect other routes, this time by `^` and `$` for a strict match.

```js
module.exports = [
  [/^\/user$/, 'user']
]
```
For the above Router, this rule will be hit only if the access address is `/user`, which will reduce the impact on other routes. If you remove `^` and `$`, accessing `/console/user/thinkjs` will also hit the above route. In fact, we may have written another route to match this address, but it is hit early by this rule, This has brought some difficulties for development.

#### Can I use a third-party Router parser?

The default route resolution of the framework is done via [think-router] (https://github.com/thinkjs/think-router). If you want to replace the third-party route resolver, you can use `src/config/middleware.js` in the Router configuration replaced by the corresponding module, and then parse the module, controller, action value stored in the `ctx` object for subsequent middleware processing.


```js
// Third-party Router analysis module example, the specific code can refer to https://github.com/thinkjs/think-router
module.exports = (options, app) => {
  return (ctx, next) => {
    const routers = app.routers; // Get all the custom Router configuration
    ...
    ctx.module = ''; // Parse the module, controller, action saved in ctx
    ctx.controller = '';
    ctx.action = '';
    return next();
  }
}
```
