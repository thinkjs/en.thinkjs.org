## Asynchronous Processing
Node.js use event-driven, non-blocking I/O model, a lot of interfaces are asynchronous, like file operations, network request。 though syncchronous methods are provided, we should avoid to use them as much as possible becuase they are blocking operations.

The official asynchronous interface use the callback form:


```js
const fs = require('fs');
fs.readFile(filepath, 'utf8', (err, content) => {
  if(err) return ;
  ...
})
```
While the concept of callbacks is simple, it can easily lead to problem call [callback hell](http://callbackhell.com/) when business logic get complicated. to solve this, methods like event、thunk、Promise、Generator function、Async functions comes up successively, and finally `Async Functions` wins out which also supports by ThinkJS.

### Async functions

Async functions use `async/await` syntax, for example:

```js
async function fn() {
  const value = await getFromApi();
  doSomethimgWithValue();
}
```

* await must be used inside async function，but async doesn't have to use with await
* Async functions can be normal function or Arrow functions
* await accepts a Promise，if not it just run through
* async function returns a Promise

Async/await is base on Promise, if expression follows await is not Promise, we need a way to turn it into a Promise.

#### Usage

ThinkJS 3.0 suggest use Async functions, and all framework interface are base on Promise which is handy to apply Async functions.

```js
module.exports = class extends think.Controller {
  async indexAction() {
    // select return Promise，use await 
    const list = await this.model('user').select();
    return this.success(list);
  }
}
```

Though Async functions is handy for Asynchronous issues, but it requires Node.js version `>=7.6.0`, to use it in lower version of Node.js, we can use Babel to tranform (Framework support Node.js version greater than 6.0, so cli default ship with Babel transform which truns Async functions into a way called Generator functions + co).

#### Compare to Generator

Async functions syntax look very similar to Generator syntax, there are quit some differences between them. Async functions:

* is born for asynchronouse issues，`async/await` has better text semantics。while Generator which is a iterator can be use to handle async problem.
* requires Promise to follow with await，but yield does not.
* does not need extra runtime, Generator needs [co](https://github.com/tj/co).
* can use with Arrow functions，while Generator function can't.
* doesn't have problem like yield or yield *.

### promisify
Async functions is base on Promise, but a lot api is not returning Promise, like Node.js is using callback which signature is `fn(aa, bb, callback(err, data))`, we can convert it into Promise way using framework method `think.promisify`.

```js
const fs = require('fs');
const readFile = think.promisify(fs.readFile, fs);

const parseFile = async (filepath) => {
  const content = await readFile(filepath, 'utf8'); // readFile returns Promise
  doSomethingWithContent();
}
```

If the callback is not `callback(err, data)`, instead of `think.promisify`, we need to handle it seperatly:

```js
const exec = require('child_process').exec;
return new Promise((resolve, reject) => {
  // exec callback params
  exec(filepath, (err, stdout, stderr) => {
    if(err) return reject(err);
    if(stderr) return reject(stderr);
    resolve(stdout);
  })
})
```

### Error Handling

Error handling is very troublesome in Node.js, a little mistake may cause respose failure. we need to take care of each callback's erro separately which is very trivial. 

By using Async functions, error will be converted to Rejected Promise, which will block the following excution.

#### try/catch

Use `try/catch` for Async function is no different then synchronous try/catch:

```js
module.exports = class extends think.Contoller {
  async indexAction() {
    try {
      await getDataFromApi1();
      await getDataFromApi2();
      await getDataFromApi3();
    } catch(e) {
      // capture error
    }
  }
}
```
Wrapper everything inside `try/catch` to catch error. One problem is it is not easy to tell which interface trigger the error in catch, to judge by error type and wrap eash call is too ugly. For this we can use `then/catch` to handle.

#### then/catch

For promise, we know there are then and catch method, each is use to handle resovle and reject. So we can precent the error by using catch and ture Rejected Promise to Resolved Promise, and then handle the error properly.


```js
module.exports = class extends think.Controller {
  async indexAction() {
    // Use catch to convert rejected promise to resolved promise
    const result = await getDataFromApi1().catch(err => {
      return think.isError(err) ? err : new Error(err)
    });
    // handle error type
    if (think.isError(result)) {
      // return error message or return formatted error message
      return this.fail(1000, result.message);
    }

    const result2 = await getDataFromApi2().catch(err => {
      return think.isError(err) ? err : new Error(err)
    });
    if(think.isError(result2)) {
      return this.fail(1001, result.message);
    }

    // return false in catch if error message is not used
    // but the condition is getDataFromApi3 will not return false
    const result3 = await getDataFromApi3().catch(() => false);
    if(result3 === false) {
      return this.fail(1002, 'error message');
    }
  }
}
```
Use catch to trun Rejected Promise to Resolved Promise, you can easily customize the way to output error message.

#### trace
In some situation, it is not convenient to add try/catch neither to turn Rejected Promise to Resolved Promise in catch, then you can use framework middleware [trace](https://github.com/thinkjs/think-trace) to handle error message.

```js
// src/config/middleware.js

module.exports = [
  ...
  {
    handle: 'trace',
    options: {
      sourceMap: false,
      debug: true, // print error messgae or not
      error(err) {
        // particular error handling, like update to monitor system.
        console.error(err);
      }
    }
  }
  ...
];
```
Whenever throws an error, trace module will catch it. In debug mode, it will display detail error information and response data as request content type.

![](https://camo.githubusercontent.com/7fc4d8401b0bae26bae354f70da39e7ad0812af2/68747470733a2f2f70312e73736c2e7168696d672e636f6d2f743031303539383661633764666331633139372e706e67)

### timeout
To delay processing some transactions, the most common way is use `setTimeout` method, but setTimeout itself doesn't return Promise, if error throw inside the function will not be catchable, we can convert it to Promise.
Frame provide `think.timeout` to quickly use a promisify timeout:

```js
return think.timeout(3000).then(() => {
  // 3s later goes here
})
```

or：

```js
module.exports = class extends think.Controller {
  async indexAction() {
    await think.timeout(3000);// wait 3000 miliseconds
    return this.success();
  }
}
```

### FAQ

#### Can I use Generator？
No，ThinkJS 3.x no longer support Generator, all asynchronous handling use Async funtions with Promise, which is also the most elegent solution so far.
