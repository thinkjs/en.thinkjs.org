## RESTful API

There is often provide an API for third parties to use in project, a common API design specification is RESTful API. RESTful API use HTTP request type to identify the operation of resources. Such as:

* `GET /ticket`  get ticket list
* `GET /ticket/:id` check out a specific ticket
* `POST /ticket`  create a ticket
* `PUT /ticket/:id` update the ticket with id equal to 12
* `DELETE /ticket/:id` delete the ticket with id equal to 12

### Create RESTful Controller

The REST Controller can be created with the parameter `-r`. Such as:

```
thinkjs controller user -r
```
The following files will be created:
```
create : src/controller/rest.js
create : src/controller/user.js
create : src/logic/user.js
```

`src/controller/user.js` will inherit `src/controller/rest.js` class，`rest.js` is the base class of RESTful Controller, and the specific logic can be modified according to the project.

### Add a custom router

RESTful Controller created and can not be immediately accessed, you need to add the corresponding [custom router](/doc/3.0/router.html). Modify the router configuration file `src / config / router.js`, and add the following configuration:

```js
module.exports = [
  [/\/user(?:\/(\d+))?/, 'user?id=:1', 'rest'], // the first way
  ['/user/:id?', '/user', 'rest'], // the second way
  ['/user/:id?', 'rest'], // the third way
]
```
Note: The third way requires the version `>= 1.0.17` of [think-router](https://github.com/thinkjs/think-router).

The meaning of custom router above is:

* `/\/user(?:\/(\d+))?/` URL regular
* `user?id=:1` Map to parse the route, :1 means take the regular (\d+) value.
* `rest` represented as REST API

By custom router, the request for `/user/:id` is specified as a REST Controller and then accessed.


* `GET /user` get user list and performing `getAction`
* `GET /user/:id` get a user's detail by performing `getAction`
* `POST /user` add a user by performing `postAction`
* `PUT /user/:id` update a user by performing `putAction`
* `DELETE /user/:id` delete a user by performing `deleteAction`

If there is a series of routes are RESTful route, adding custom router each time is very troublesome, then you can modify the custom router configuration file, for example:

```js
module.exports = [
  [/\/api\/(\w+)(?:\/(\d+))?/, 'api/:1?id=:2', 'rest']
];
```
This means that all secondary routes starting with `/ api` will be designated as RESTful routes.

### Data validation

The method in Controller doesn't check the data passed in. The data check can be processed in Logic. The file is `src/logic/user.js`, and the corresponding Action and Controller are in one-to-one correspondence. Specific instructions see the [Logic](/doc/3.0/logic.html).

### Sub-RESTful API

Sometimes there need sub-RESTful API, such as interface of comments on an article, this time can be done through the following custom router:

```js
module.exports = [
  [/\/post\/(\d+)\/comments(?:\/(\d+))?/, 'comment?postId=:1&id=:2', 'rest']
]
```

So in the corresponding Action, you can get the article's id by `this.get("postId")`, and then you can handle it by placing the id in the filter.

```js
const Rest = require('./rest.js');
module.exports = class extends Rest {
  async getAction() {
    const postId = this.get('postId');
    const commentId = this.get('id');
    const comment = this.model('comment');
    if(commentId) { // get detail message
      const data = await comment.where({post_id: postId, id: commentId}).find();
      return this.success(data);
    } else { // get comment list
      const list = await comment.where({post_id: postId}).select();
      return this.success(list);
    }
  }
}
```

### Multi-version RESTful API

Sometimes, some REST APIs are not fully compatible with the previous versions or later versions, so you need to have multiple versions. You can also customize router management at this time, for example:

```js
module.exports = [
  [/\/v1\/user(?:\/(\d+))?/, 'v1/user?id=:1', 'rest'], //v1 version
  [/\/v2\/user(?:\/(\d+))?/, 'v2/user?id=:1', 'rest']  //v2 version
]
```

This time as long as create subdirectory `v1/` and `v2/` inside `src/controller/`, the implementation will automatically find. Specific instructions see the [multi-level controller](/doc/3.0/controller.html#toc-04e).

### RESTful API of Mongo

Because Mongo's id isn't purely numeric, you only need to change the corresponding regular expression when dealing with Mongo's RESTful API (change `\d` to `\w`):

```js
module.exports = [
  [/\/user(?:\/(\w+))?/, 'user?id=:1', 'rest']
]
```

### FAQ

#### How to check RESTful API's custom router has taken effect?

Sometimes after adding the RESTful Controller and custom router, the access doesn't take effect. In this case, you can start the service through `DEBUG=think-router npm start` to check whether the parsed controller and action take effect. For details, see the [How to know what the controller and action are for the current address after the resolution?](/doc/3.0/router.html#toc-54f).

