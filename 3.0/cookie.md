## Cookie

Because the HTTP(S) protocol is a stateless protocol, multiple requests do not know from the same user. This will bring a lot of problems, such as: some page users can access after logging in, page content based on user-related.

In the early days, the solution was to generate a random token, which will be carried on with every request to identify the user. This requires inserting a hidden field containing the token in the form, or on the parameters of the URL request.

Although this approach can solve the problem, but to bring great inconvenience to development, it is also not conducive to the spread of page addresses. To solve this problem, [RFC 2965](https://tools.ietf.org/html/rfc2965) Introduced cookie mechanism that carries `Cookie` headers when requested and sets cookies in response to the` Set-Cookie` field.

### Cookie Format

Request cookie format:

```
Cookie: name1=value1; name2=value2; name3=value3 //multiple Cookie seperate by `; `
```

Response Cookie format:

```
Set-Cookie: key1=value1; path=path; domain=domain; max-age=max-age-in-seconds; expires=date-in-GMTString-format; secure; httponly
Set-Cookie: key2=value2; path=path; domain=domain; max-age=max-age-in-seconds; expires=date-in-GMTString-format; secure; httponly
```

* `key=value` name, value pair
* `path=path` specify effective path, most of the time is `/`, which can take effect inall paths
* `domain=domain` specify effctive domain, validation will be applied
* `max-age=max-age-in-seconds` Suvival time, generally used with expires
* `expires=date-in-GMTString-format` expires time
* `secure` only take effect in `HTTPS`
* `httponly` only carried in HTTP, can't fetch by JS

If you do not set `max-age` and` expires`, cookies will be destroyed as the browser exits. For those who do not want JS to get a cookie, generally set `httponly` attribute, such as: the user Cookie corresponding Session.

Although there are no cookie size restrictions in the standard, browsers generally have limitations, so you can not save too much text in cookies (typically no more than 4K).

### Configuration

Framework use [cookies](https://github.com/pillarjs/cookies) module to manipulate Cookie, support the following configurations:

* `maxAge`: The cookie's timeout, which indicates the number of milliseconds after the current time (`Date.now()`).
* `expires`: `Date` object, which means the expiration time of the cookie (default is expired at the end of the session if not specified).
* `path`: String, indicating the path to the cookie (default `/`).
* `domain`: String, indicating the cookie's domain (no default).
* `secure`: A Boolean value that indicates whether the cookie is only sent over HTTPS (the default is sent via HTTP when `false` is set, and the default is sent over HTTPS when `true` is set).
* `httpOnly`: Boolean value indicating whether the cookie is to be sent over HTTP(S) only, but not from the client's JavaScript (default is `true`).
* `sameSite`: Boolean or string indicating whether the cookie is a `Homologous` cookie (default is `false`). It can be set to `'strict'`,`'lax'`, or `true` (equivalent to `strict`).
* `signed`: A Boolean value that indicates whether to sign the cookie (default is `false`). If set to true, another cookie of the same name with the .sig suffix will be sent as a 27-byte url-safe base64 SHA1 value indicating _cookie-name _ = _ cookie-value_ Hash value, relative to the first [Keygrip](https://www.npmjs.com/package/keygrip) key. This signing key is used to detect tampering the next time a cookie is received.
* `overwrite`: Boolean value that indicates whether to overwrite the cookie of the same name previously set (defaults to false). If set to true, all cookies of the same name (regardless of path or domain) set in the same request will be filtered out of the Set-Cookie header when this cookie is set.


If you need to modify the above configuration, you can modify the configuration file `src / config / config.js`. Such as:

```js
module.exports = {
  cookie: {
    domain: '', 
    path: '/',
    maxAge: 10 * 3600 * 1000, // 10 hours
    signed: true,
    keys: [] // When signed is ture, the key used by keygrip library
  }
}
```

### manipulate cookie

ctx、controller、logic provide `cookie` method to manipulate cookie.

#### get cookie

```js
const theme = this.cookie('theme')
```

#### set cookie

```js
this.cookie('theme', 'gray'); 
this.cookie('theme', 'yellow', { // set other configuration
  maxAge: 10 * 1000,
  path: '/theme'
})
```

#### delete cookie

```js
this.cookie('theme', null)
this.cookie('theme', null, {
  domain: '',
  path: ''
})
```

Deleting a cookie requires the same domain and path configuration as when setting a cookie, otherwise cookies will not be successfully deleted because of a mismatch.

### FAQ

#### Can I send cookie after content is output?

Because the sending cookie is done through the `Set-Cookie` header field, the HTTP protocol specifies that the header information must be sent before the content, so no cookie information can be sent after the content is output.

If you force the header information such as a cookie to be sent after the content is output, something like the following error appears:

```
[ERROR] - Error: Can't set headers after they are sent.
    at ServerResponse.OutgoingMessage.setHeader (_http_outgoing.js:346:11)
    at Cookies.set (think-demo/node_modules/thinkjs/node_modules/cookies/index.js:115:13)
    at Object.cookie (think-demo/node_modules/thinkjs/lib/extend/context.js:260:21)
    at IndexController.cookie (think-demo/node_modules/thinkjs/lib/extend/controller.js:181:21)
    at Timeout._onTimeout (think-demo/src/controller/index.js:10:12)
    at tryOnTimeout (timers.js:224:11)
    at Timer.listOnTimeout (timers.js:198:5)

```
