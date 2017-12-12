## Context

Context, an instance provided by Koa, it is used across the whole lifecycle of user request. It is used within middleware, controller and logic, named `ctx` for short.


```js
// use ctx in middleware
module.exports = options => {
  // ctx use be passed in as the first params
  return (ctx, next) => {
    ...
  }
}
```

```js
// use ctx in controller
module.exports = class extends think.Controller {
  indexAction() {
    const ip = this.ctx.ip;
  }
}
```

Framework inhert ctx and use Extend to add many useful propertis and methods.

### Koa builtin API

#### ctx.req

Node's [request](https://nodejs.org/api/http.html#http_class_http_incomingmessage) object.

#### ctx.res

Node's [response](https://nodejs.org/api/http.html#http_class_http_serverresponse) object.

**No Supported** to use response bypass Koa, avoid to use the following properties:

- `res.statusCode`
- `res.writeHead()`
- `res.write()`
- `res.end()`

#### ctx.request

Koa‘s [Request](http://koajs.com/#request) class.

#### ctx.response

Koa's [Response](http://koajs.com/#response) class.

#### ctx.state
The suggest namespace for sharing data between middleware or sending message to template. We should avoid to put fields into ctx directly which may override existing properties and cuase weired issues.

```js
ctx.state.user = await User.find(id);
```

We can get value in controller through `this.ctx.state.user`.

```js
module.exports = class extends think.Controller {
  indexAction() {
    const user = this.ctx.state.user;
  }
}
```

#### ctx.app

Application instance, the same as `think.app`.

#### ~~ctx.cookies.get(name, [options])~~

Get cookie, deprecated, use [ctx.cookie(name)](#toc-a67) instead.

#### ~~ctx.cookies.set(name, value, [options])~~

Set cookie, deprecated, use [ctx.cookie(name, value, options)](#toc-a67) instead.

#### ctx.throw([msg], [status], [properties])

Helper method, throw error including `.status`, default is `500`. This method allows Koa to response accordingly which support the following combination:

```js
ctx.throw(403)
ctx.throw('name required', 400)
ctx.throw(400, 'name required')
ctx.throw('something exploded')
```

`this.throw('name required', 400)` is equivalent as bellow:

```js
let err = new Error('name required');
err.status = 400;
throw err;
```

Note, this is user scope error, `err.expose` is marked, so these message can be used to response client request. Apparently you won't use it to expose error message if you don't want to leak error detail.

You can passe `properties` object, which will be merged to error, to pass message to other middlewares with a nice defined error message.

```js
ctx.throw(401, 'access_denied', { user: user });
ctx.throw('access_denied', { user: user });
```
Koa use [http-errors](https://github.com/jshttp/http-errors) to create error object.

#### ctx.assert(value, [msg], [status], [properties])

Helper function to throw error When `!value` equals `true`, similar to `.throw()`. Also similar to node's [assert()](http://nodejs.org/api/assert.html) method.

```
this.assert(this.user, 401, 'User not found. Please login!');
```

Koa use [http-assert](https://github.com/jshttp/http-assert) to assert.

#### ctx.respond

If you don't want to use Koa's buildin response, just set `ctx.respond = false`. And then you can use the origin `res` object to response.

Note Koa __doesn't__ suuport this, becuase it may break Koa's middleware and Koa itself. It is a hack way, for those want to use traditional `fn(req, res)` method with middleware.

#### ctx.header

Get all header message, equivalent to `ctx.request.header`.

```js
const headers = ctx.headers;
```

#### ctx.headers

Get all header information, equivalent to `ctx.header`.

#### ctx.method

Get request type, uppercae. Like: `GET`, `POST`, `DELETE`.

```js
const method = ctx.method;
```

#### ctx.method=

Set the request type (will not actually change HTTP request's type), may be useful for some middlewares, example: `methodOverride()`.

```js
ctx.method = 'COMMAND';
```

#### ctx.url

Get request url.

#### ctx.url=

Set request url, for URL rewrite.

#### ctx.originalUrl

Get original request URL.

#### ctx.origin

Get origin of URL, include protocol and host.

```js
ctx.origin
// => http://example.com
```


#### ctx.href

Get full request URL, include protocol, host and url.

```js
ctx.href
// => http://example.com/foo/bar?q=1
```

#### ctx.path

Get request pathname.

#### ctx.path=

Set request pathname and retain query-string when present.

#### ctx.query

Get parsed query-string, returning an empty object when no query-string is present. Note that this getter does not support nested parsing.

For example "color=blue&size=small":

```js
{
  color: 'blue',
  size: 'small'
}
```
#### ctx.query=

Set query-string to the given object. Note that this setter does not support nested objects.

```js
ctx.query = { next: '/login' }
```
#### ctx.querystring

Get raw query string void of ?.

#### ctx.querystring=

Set raw query string.

#### ctx.search

Get raw query string with the ?.

#### ctx.search=

Set raw query string.

#### ctx.host

Get host (hostname:port) when present. Supports X-Forwarded-Host when app.proxy is true, otherwise Host is used.

#### ctx.hostname

Get hostname when present. Supports X-Forwarded-Host when app.proxy is true, otherwise Host is used.

#### ctx.type

Get request Content-Type void of parameters such as "charset".

```js
const ct = ctx.type
// => "image/png"
```

#### ctx.charset

Get request charset when present, or undefined:

```js
ctx.charset
// => "utf-8"
```
#### ctx.fresh

Check if a request cache is "fresh", aka the contents have not changed. This method is for cache negotiation between If-None-Match / ETag, and If-Modified-Since and Last-Modified. It should be referenced after setting one or more of these response headers.

```js
// freshness check requires status 20x or 304
ctx.status = 200;
ctx.set('ETag', '123');

// cache is ok
if (ctx.fresh) {
  ctx.status = 304;
  return;
}

// cache is stale
// fetch new data
ctx.body = await db.find('something');
```
#### ctx.stale

Inverse of ctx.fresh.

#### ctx.socket

Return the request socket.

#### ctx.protocol

Get request protocal, value is `https` or `http`, when `app.proxy` is ture then protocal value is retreive from `X-Forwarded-Proto` header.

Specific details, if `req.socket.encrypted` is true, then return `https`, otherwise if `app.proxy` is true, return `X-Forwarded-Proto` header value, default value is `http`.

Usually we don't want Node.js to serve client directly, but using an extrat web server layout (like nginx). Web server provide HTTP(S) service, and communicate with Node.js with HTTP.

In this situation, Node.js always use `https` as protocal, the actual protocal only known to web server. So we need to defined a special header for it, recommand `X-Forwarded-Proto`. For safty, only if `app.proxy` if true then protocal will read from this header (`production.js` default value is true).


```sh
ssl on;
# SSL certificate
ssl_certificate /usr/local/nginx/ssl/domain.crt;
ssl_certificate_key /usr/local/nginx/ssl/domain.key;

location = /index.js {
  proxy_http_version 1.1;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header Host $http_host;
  proxy_set_header X-Forwarded-Proto "https"; # current Node.js protocol https
  proxy_set_header X-NginX-Proxy true;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
  proxy_pass http://127.0.0.1:$node_port$request_uri;
  proxy_redirect off;
}
```

#### ctx.secure

Shorthand for ctx.protocol == "https" to check if a request was issued via TLS.


#### ctx.ip

Request remote address. Supports X-Forwarded-For when app.proxy is true.


#### ctx.ips

When X-Forwarded-For is present and app.proxy is enabled an array of these ips is returned, ordered from upstream -> downstream. When disabled an empty array is returned.


#### ctx.subdomains

Return subdomains as an array.

Subdomains are the dot-separated parts of the host before the main domain of the app. By default, the domain of the app is assumed to be the last two parts of the host. This can be changed by setting app.subdomainOffset.

For example, if the domain is "tobi.ferrets.example.com": If app.subdomainOffset is not set, ctx.subdomains is ["ferrets", "tobi"]. If app.subdomainOffset is 3, ctx.subdomains is ["tobi"].


#### ctx.is(...types)

Check if the incoming request contains the "Content-Type" header field, and it contains any of the give mime types. If there is no request body, null is returned. If there is no content type, or the match fails false is returned. Otherwise, it returns the matching content-type.

```js
// With Content-Type: text/html; charset=utf-8
ctx.is('html'); // => 'html'
ctx.is('text/html'); // => 'text/html'
ctx.is('text/*', 'text/html'); // => 'text/html'

// When Content-Type is application/json
ctx.is('json', 'urlencoded'); // => 'json'
ctx.is('application/json'); // => 'application/json'
ctx.is('html', 'application/*'); // => 'application/json'

ctx.is('html'); // => false
```

For example if you want to ensure that only images are sent to a given route:

```js
if (ctx.is('image/*')) {
  // process
} else {
  ctx.throw(415, 'images only!');
}
```
#### ctx.accepts(types)

Check if the given type(s) is acceptable, returning the best match when true, otherwise false. The type value may be one or more mime type string such as "application/json", the extension name such as "json", or an array ["json", "html", "text/plain"].

```js
// Accept: text/html
ctx.accepts('html');
// => "html"

// Accept: text/*, application/json
ctx.accepts('html');
// => "html"
ctx.accepts('text/html');
// => "text/html"
ctx.accepts('json', 'text');
// => "json"
ctx.accepts('application/json');
// => "application/json"

// Accept: text/*, application/json
ctx.accepts('image/png');
ctx.accepts('png');
// => false

// Accept: text/*;q=.5, application/json
ctx.accepts(['html', 'json']);
ctx.accepts('html', 'json');
// => "json"

// No Accept header
ctx.accepts('html', 'json');
// => "html"
ctx.accepts('json', 'html');
// => "json"
```

You may call ctx.accepts() as many times as you like, or use a switch:

```js
switch (ctx.accepts('json', 'html', 'text')) {
  case 'json': break;
  case 'html': break;
  case 'text': break;
  default: ctx.throw(406, 'json, html, or text only');
}
```
#### ctx.acceptsEncodings(encodings)

Check if encodings are acceptable, returning the best match when true, otherwise false. Note that you should include identity as one of the encodings!

```js
// Accept-Encoding: gzip
ctx.acceptsEncodings('gzip', 'deflate', 'identity');
// => "gzip"

ctx.acceptsEncodings(['gzip', 'deflate', 'identity']);
// => "gzip"
```

When no arguments are given all accepted encodings are returned as an array:

```js
// Accept-Encoding: gzip, deflate
ctx.acceptsEncodings();
// => ["gzip", "deflate", "identity"]
```
Note that the identity encoding (which means no encoding) could be unacceptable if the client explicitly sends identity;q=0. Although this is an edge case, you should still handle the case where this method returns false.

#### ctx.acceptsCharsets(charsets)

Check if charsets are acceptable, returning the best match when true, otherwise false.

```js
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
ctx.acceptsCharsets('utf-8', 'utf-7');
// => "utf-8"

ctx.acceptsCharsets(['utf-7', 'utf-8']);
// => "utf-8"
```

When no arguments are given all accepted charsets are returned as an array:

```js
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
ctx.acceptsCharsets();
// => ["utf-8", "utf-7", "iso-8859-1"]
```
#### ctx.acceptsLanguages(langs)

Check if langs are acceptable, returning the best match when true, otherwise false.

```js
// Accept-Language: en;q=0.8, es, pt
ctx.acceptsLanguages('es', 'en');
// => "es"

ctx.acceptsLanguages(['en', 'es']);
// => "es"
```

When no arguments are given all accepted languages are returned as an array:

```js
// Accept-Language: en;q=0.8, es, pt
ctx.acceptsLanguages();
// => ["es", "pt", "en"]
```

#### ctx.get(field)

Return request header.

```js
const host = ctx.get('host');
```

#### ctx.body

Get response body.

#### ctx.body=

Set response body to one of the following:

* string written

  The Content-Type is defaulted to text/html or text/plain, both with a default charset of utf-8. The Content-Length field is also set.

* Buffer written

  The Content-Type is defaulted to application/octet-stream, and Content-Length is also set.

* Stream piped

  The Content-Type is defaulted to application/octet-stream.

  Whenever a stream is set as the response body, .onerror is automatically added as a listener to the error event to catch any errors. In addition, whenever the request is closed (even prematurely), the stream is destroyed. If you do not want these two features, do not set the stream as the body directly. For example, you may not want this when setting the body as an HTTP stream in a proxy as it would destroy the underlying connection.

  See: https://github.com/koajs/koa/pull/612 for more information.

  Here's an example of stream error handling without automatically destroying the stream:

  ```js
  const PassThrough = require('stream').PassThrough;

  app.use(function * (next) {
    ctx.body = someHTTPStream.on('error', ctx.onerror).pipe(PassThrough());
  });
  ```

* Object || Array json-stringified

  The Content-Type is defaulted to application/json. This includes plain objects { foo: 'bar' } and arrays ['foo', 'bar'].

* null no content response

If ctx.status has not been set, Koa will automatically set the status to 200 or 204.

#### ctx.status

Get response status. By default, response.status is set to 404 unlike node's res.statusCode which defaults to 200.

#### ctx.status=

Set response status via numeric code:

* 100 "continue"
* 101 "switching protocols"
* 102 "processing"
* 200 "ok"
* 201 "created"
* 202 "accepted"
* 203 "non-authoritative information"
* 204 "no content"
* 205 "reset content"
* 206 "partial content"
* 207 "multi-status"
* 208 "already reported"
* 226 "im used"
* 300 "multiple choices"
* 301 "moved permanently"
* 302 "found"
* 303 "see other"
* 304 "not modified"
* 305 "use proxy"
* 307 "temporary redirect"
* 308 "permanent redirect"
* 400 "bad request"
* 401 "unauthorized"
* 402 "payment required"
* 403 "forbidden"
* 404 "not found"
* 405 "method not allowed"
* 406 "not acceptable"
* 407 "proxy authentication required"
* 408 "request timeout"
* 409 "conflict"
* 410 "gone"
* 411 "length required"
* 412 "precondition failed"
* 413 "payload too large"
* 414 "uri too long"
* 415 "unsupported media type"
* 416 "range not satisfiable"
* 417 "expectation failed"
* 422 "unprocessable entity"
* 423 "locked"
* 424 "failed dependency"
* 426 "upgrade required"
* 428 "precondition required"
* 429 "too many requests"
* 431 "request header fields too large"
* 500 "internal server error"
* 501 "not implemented"
* 502 "bad gateway"
* 503 "service unavailable"
* 504 "gateway timeout"
* 505 "http version not supported"
* 506 "variant also negotiates"
* 507 "insufficient storage"
* 508 "loop detected"
* 510 "not extended"
* 511 "network authentication required"

NOTE: don't worry too much about memorizing these strings, if you have a typo an error will be thrown, displaying this list so you can make a correction.


#### ctx.message

Get response status message. By default, response.message is associated with response.status.

#### ctx.message=

Set response status message to the given value.

#### ctx.length=

Set response Content-Length to the given value.

#### ctx.length

Return response Content-Length as a number when present, or deduce from ctx.body when possible, or undefined.


#### ctx.type

Get response Content-Type void of parameters such as "charset".

```js
const ct = ctx.type;
// => "image/png"
```

#### ctx.type=

Set response Content-Type via mime string or file extension.

```js
ctx.type = 'text/plain; charset=utf-8';
ctx.type = 'image/png';
ctx.type = '.png';
ctx.type = 'png';
```

Note: when appropriate a charset is selected for you, for example response.type = 'html' will default to "utf-8", however when explicitly defined in full as response.type = 'text/html' no charset is assigned.

#### ctx.headerSent

Check if a response header has already been sent. Useful for seeing if the client may be notified on error.


#### ctx.redirect(url, [alt])

Perform a [302] redirect to url.

The string "back" is special-cased to provide Referrer support, when Referrer is not present alt or "/" is used.

```js
ctx.redirect('back');
ctx.redirect('back', '/index.html');
ctx.redirect('/login');
ctx.redirect('http://google.com');
```

To alter the default status of 302, simply assign the status before or after this call. To alter the body, assign it after this call:

```js
ctx.status = 301;
ctx.redirect('/cart');
ctx.body = 'Redirecting to shopping cart';
```
#### ctx.attachment([filename])
Set Content-Disposition to "attachment" to signal the client to prompt for download. Optionally specify the filename of the download.

#### ctx.set(fields)

Set several response header fields with an object:

```js
ctx.set({
  'Etag': '1234',
  'Last-Modified': date
});
```

#### ctx.append(field, value)

Append additional header field with value val.

```js
ctx.append('Link', '<http://127.0.0.1/>');
```

#### ctx.remove(field)

Remove header field.

#### ctx.lastModified=

Set the Last-Modified header as an appropriate UTC string. You can either set it as a Date or date string.

```js
ctx.lastModified = new Date();
```
#### ctx.etag=

Set the ETag of a response including the wrapped "s. Note that there is no corresponding response.etag getter.

```js
ctx.etag = crypto.createHash('md5').update(ctx.body).digest('hex');
```
### Framework Extend API

#### ctx.module

Get module name base on route parsing, this value is always empty on single module project. Default parsing logic is use [think-router](https://github.com/thinkjs/think-router) module.

```js
module.exports = class extends think.Controller {
  __before() {
    // get module
    // Variable name module is used by node, use m instead
    const m = this.ctx.module;
  }
}
```

#### ctx.controller

Get controller name base on route parsing, parse by [think-router](https://github.com/thinkjs/think-router).

```js
module.exports = class extends think.Controller {
  __before() {
    // get controller
    const controller = this.ctx.controller;
  }
}
```

#### ctx.action

Get action name base on route parsing, parse by [think-router](https://github.com/thinkjs/think-router) .

```js
module.exports = class extends think.Controller {
  __before() {
    // get action
    const action = this.ctx.action;
  }
}
```

#### ctx.userAgent

Get user agent.

```js
const userAgent = ctx.userAgent;
if(userAgent.indexOf('spider')){
  ...
}
```

#### ctx.isGet

Judge whether current request method is `GET`.

```js
const isGet = ctx.isGet;
if(isGet){
  ...
}
```

#### ctx.isPost

Judge whether current request method is `POST`.

```js
const isPost = ctx.isPost;
if(isPost){
  ...
}
```

#### ctx.isCli

Judge whether current request type is `CLI` (call from command line).

```js
const isCli = ctx.isCli;
if(isCli){
  ...
}
```

#### ctx.referer(onlyHost)

* `onlyHost` {Boolean} only return host
* `return` {String}

get request referer.

```
const referer1 = ctx.referer(); // http://www.thinkjs.org/doc.html
const referer2 = ctx.referer(true); // www.thinkjs.org
```

#### ctx.referrer(onlyHost)

equal to `referer`.

#### ctx.isMethod(method)

* `method` {String} type
* `return` {Boolean}

Judge whether current request method equals to method value.

```js
const isPut = ctx.isMethod('PUT');
```

#### ctx.isAjax(method)

* `method` {String} request type
* `return` {Boolean}

Judge whether it is ajax request (by `x-requested-with` header value equals to `XMLHttpRequest`), if method is passed, this method will also do compare the request type equals to method value.

```js
const isAjax = ctx.isAjax();
const isPostAjax = ctx.isAjax('POST');
```

#### ctx.isJsonp(callbackField)

* `callbackField` {String} callback field name，default value is `this.config('jsonpCallbackField')`
* `return` {Boolean}

Judge whether it is jsonp request.

```js
const isJsonp = ctx.isJson('callback');
if(isJsonp){
  ctx.jsonp(data);
}
```

#### ctx.jsonp(data, callbackField)

* `data` {Mixed} output data
* `callbackField` {String} callback field name，default value is `this.config('jsonpCallbackField')`
* `return` {Boolean} false

Output jsonp format data, the return value is false. The `Content-Type` returned can be specified by configuring` jsonContentType`.

```js
ctx.jsonp({name: 'test'});

//output
jsonp111({
  name: 'test'
})
```

#### ctx.json(data)

* `data` {Mixed} output data
* `return` {Boolean} false

Output json format data, the return value is false. The `Content-Type` returned can be specified by configuring` jsonContentType`.

```js
ctx.json({name: 'test'});

//output
{
  name: 'test'
}
```

#### ctx.success(data, message)

* `data` {Mixed} output data
* `message` {String} errmsg field data
* `return` {Boolean} false

Output data with `errno` and` errmsg` formats. Where `errno` is 0 and` errmsg` is message.

```js
{
  errno: 0,
  errmsg: '',
  data: ...
}
```
The field names `errno` and` errmsg` can be modified by configuring `errnoField` and` errmsgField`.

#### ctx.fail(errno, errmsg, data)

* `errno` {Number} error number
* `errmsg` {String} error message
* `data` {Mixed} extra error data
* `return` {Boolean} false

```js
{
  errno: 1000,
  errmsg: 'no permission',
  data: ''
}
```

The field names `errno` and` errmsg` can be modified by configuring `errnoField` and` errmsgField`.


#### ctx.expires(time)

* `time` {Number} cache time，unit is miliseconds. Support time format like `1s` or `1m`.
* `return` {undefined}

set `Cache-Control` and `Expires` cache header.

```js
ctx.expires('1h'); //cache 1 hour
```

#### ctx.config(name, value, m)

* `name` {Mixed} config name
* `value` {Mixed} confg value
* `m` {String} module name, for multi-module project
* `return` {Mixed}


Get, set configuration items, internal call `think.config` method.

```js
ctx.config('name'); //get config
ctx.config('name', value); // get config
ctx.config('name', undefined, 'admin'); //get admin module config, for multi-module project
```

#### ctx.param(name, value)

* `name` {String} param name
* `value` {Mixed} param value
* `return` {Mixed}

Get, set the parameter value on the URL. Since the names get, query, etc. have been used by Koa, param can only be used here.

```js
ctx.param('name'); //will undefined if 'name' is not exist
ctx.param(); // Get all the parameter values, including dynamically added parameters
ctx.param('name1,name2'); // Get the specified number of parameter values, separated by commas
ctx.param('name', value); // Reset the parameter value
ctx.param({name: 'value', name2: 'value2'}); // Reset multiple parameter values
```

#### ctx.post(name, value)

* `name` {String} 
* `value` {Mixed} 
* `return` {Mixed}

Get, Set post value.

```js
ctx.post('name'); //Get the POST value, or undefined if it does not exist
ctx.post(); //Get all the POST values, including dynamically added data
ctx.post('name1,name2'); // Get the specified number of POST values, separated by commas
ctx.post('name', value); // Reset the POST value
ctx.post({name: 'value', name2: 'value2'}); //Reset multiple POST values
```

Sometimes submitted data is a composite data, this time to get the data format is the following format:

```
{ action: 'create',
  'data[0][username]': '',
  'data[0][nickname]': '',
  'data[0][password]': '' 
}
```

In fact, we want the data field data to be an array, which we can support using [think-qs] (https://github.com/thinkjs/think-qs) middleware.

#### ctx.file(name, value)

* `name` {String} 
* `value` {Mixed} 
* `return` {Mixed}

Get, set the file data, the file will be saved in a temporary directory, for security, the request will be deleted after the end. If you need to use the corresponding file, you can use the `fs.rename` method to move to other places.
```js
ctx.file('name'); // Get the FILE value, or undefined if it does not exist
ctx.file(); // Get all the FILE values, including dynamically added data 
ctx.file('name', value); // Reset the FILE value
ctx.file({name: 'value', name2: 'value2'}); // Reset multiple FILE values
```

文件的数据格式为：

```js
{
  "size": 287313, // file size
  "path": "/var/folders/4j/g57qvmmd1lb_9h605w_d38_r0000gn/T/upload_fa6bf8c44179851f1cfec99544b4ef22", //temp location
  "name": "An Introduction to libuv.pdf", // file name
  "type": "application/pdf", // type
  "mtime": "2017-07-02T07:55:23.763Z" // last modify time
}
```

File uploads are resolved by the [think-payload] (https://github.com/thinkjs/think-payload) module, which allows you to configure parameters such as file size restrictions.

```js
const fs = require('fs');
const path = require('path');
const rename = think.promisify(fs.rename, fs); // The promisify method renames the method to a Promise interface
module.exports = class extends think.Controller {
  async indexAction(){
    const file = this.file('image');
    // If you upload png format image file, move to another directory
    if(file && file.type === 'image/png') {
      const filepath = path.join(think.ROOT_PATH, 'runtime/upload/a.png');
      think.mkdir(path.dirname(filepath));
      await rename(file.path, filepath)
    }
  }
}
```



#### ctx.cookie(name, value, options)

* `name` {String} Cookie name
* `value` {mixed} Cookie value
* `options` {Object} Cookie config
* `return` {Mixed}

获取、设置 Cookie 值。

```js
ctx.cookie('name'); //get Cookie
ctx.cookie('name', value); //set Cookie
ctx.cookie(name, null); //delete Cookie
ctx.cookie(name, null, {
  path: '/'
})
```

设置 Cookie 时，如果 value 的长度大于 4094，则触发 `cookieLimit` 事件，该事件可以通过 `think.app.on("cookieLimit")` 来捕获。
删除 Cookie 时，必须要设置 `domain`、`path` 等参数和设置的时候相同，否则因为浏览器的同源策略无法删除。
When setting a cookie, if the length of value is greater than 4094, a `cookieLimit` event is fired, which can be captured via `think.app.on ("cookieLimit")`.

When Delete cookie, you must set `domain`,`path` and other parameters and set the same time, otherwise the browser's homologous strategy will reject the delete action.

#### ctx.service(name, m, ...args)

* `name` {String} service name
* `m` {String} module name, only for multi-module project
* `return` {Mixed}

Get service, if the class is instantiated, or directly return. Equivalent to [think.service](/doc/3.0/think.html#toc-014).

```js
// get src/service/github.js module
const github = ctx.service('github');
```

#### ctx.download(filepath, filename)

* `filepath` {String} download file path
* `filename` {String} download file name, if not exist will get from `filepath`.

Download file will set `Content-Disposition` header base on [content-disposition](https://github.com/jshttp/content-disposition) module.

```js
const filepath = path.join(think.ROOT_PATH, 'a.txt');
ctx.download(filepath);
```

If the file name contains Chinese cause garbled, then you can manually specify the Content-Disposition header information, such as:

```js
const userAgent = this.userAgent().toLowerCase();
let hfilename = '';
if (userAgent.indexOf('msie') >= 0 || userAgent.indexOf('chrome') >= 0) {
  hfilename = `=${encodeURIComponent(filename)}`;
} else if(userAgent.indexOf('firefox') >= 0) {
  hfilename = `*="utf8''${encodeURIComponent(filename)}"`;
} else {
  hfilename = `=${new Buffer(filename).toString('binary')}`;
}
ctx.set('Content-Disposition', `attachment; filename${hfilename}`)
ctx.download(filepath)
```
