## Relation Model

During project development, you always need to manipulate database tables, thus involes CRUD operations, but Spelling SQL statements manually is very troublesome. Meanwhile，you also need pay attention to the security issue like SQL injection. ThinkJS provides the model function to facilitate the operation of the database.

### Extend model

The default framework does not provide the model function, you need to load the corresponding extension to support, the corresponding module is [think-model](https://github.com/thinkjs/think-model).Modify the extended configuration file `src/config/extend.js` ( `src/common/config/extend.js` in multi-module project) and add the following configuration:

```js
const model = require('think-model');

module.exports = [
  model(think.app) // let the framework support the model function
]
```

After adding the model's extension, the method [think.Model](/doc/3.0/relation_model.html#toc-c4c)、[think.model](/doc/3.0/relation_model.html#toc-3f0)、[ctx.model](/doc/3.0/relation_model.html#toc-876)、[controller.model](/doc/3.0/relation_model.html#toc-7ff)、[service.model](/doc/3.0/relation_model.html#toc-af8) is added here。


### Configure the database

Since the model will support multi-types of database, so the format of the configuration file by the Adapter way. The file path is `src/config/adapter.js` (`src/common/config/adapter.js` in multi-module project).

```js
const mysql = require('think-model-mysql');
exports.model = {
  type: 'mysql', //default type, can call the specified parameters to switch
  common: { // common configuration
    logConnect: true, // whether to print database connection information
    logSql: true, // whether to print SQL statement
    logger: msg => think.logger.info(msg) // the logger for print information
  },
  mysql: { // mysql configuration
    handle: mysql
  },
  mysql2: { // another mysql configuration
    handle: mysql
  },
  sqlite: {  // sqlite configuration

  },
  postgresql: { // postgresql configuration

  }
}
```

if the project need to use mutiple configurations of the same database, you can distinguish between different `types`.

```js
const user1 = think.model('user'); // use default database configuration, default type is mysql
const user2 = think.model('user', 'mysql2'); // use mysql2 configuration
const user3 = think.model('user', 'sqlite'); // use sqlite configuration
const user4 = think.model('user', 'postgresql'); // use postgresql configuration
```

As you can call the specified `type`, in theory, ThinkJS support an unlimited number of types of configuration, the project can be configured as needed.

#### Mysql

Adapter of Mysql is [think-model-mysql](https://github.com/thinkjs/think-model-mysql), the bottom is based on the [mysql](https://github.com/mysqljs/mysql) library, using the connection pool way to connect to the database, the default connection number is 1.

```js
const mysql = require('think-model-mysql');
exports.model = {
  type: 'mysql',
  mysql: {
    handle: mysql, // Adapter handle
    user: 'root', // username
    password: '',
    database: '',
    host: '127.0.0.1',
    port: 3306,
    connectionLimit: 1, // connection number of connection pool, the default is 1
    prefix: '', // data sheet prefix，, if there is more than one item in a database, then the data sheet between the items can be distinguished by the prefix
  }
}
```

In addition to using the host and port to connect to the database, but also through `socketPath` to connect. More configuration options see <https://github.com/mysqljs/mysql#connection-options>

#### SQLite

Adapter of SQLite is [think-model-sqlite](https://github.com/thinkjs/think-model-sqlite), the bottom is based on the [sqlite3](https://github.com/mapbox/node-sqlite3) library, using the connection pool way to connect to the database, the default connection number is 1.

```js
const sqlite = require('think-model-sqlite');
exports.model = {
  type: 'sqlite',
  sqlite: {
    handle: sqlite, // Adapter handle
    path: path.join(think.ROOT_PATH, 'runtime/sqlite'), // directory for saving sqlite
    database: '', // database name
    connectionLimit: 1, // connection number of connection pool, the default is 1
    prefix: '', // data sheet prefix，, if there is more than one item in a database, then the data sheet between the items can be distinguished by the prefix
  }
}
```

#### PostgreSQL


Adapter of PostgreSQL is [think-model-postgresql](https://github.com/thinkjs/think-model-postgresql), the bottom based on [pg](https://github.com/brianc/node-postgres) library, using the connection pool way to connect to the database, the default connection number is 1.

```js
const postgresql = require('think-model-postgresql');
exports.model = {
  type: 'postgresql',
  postgresql: {
    handle: postgresql, // Adapter handle
    user: 'root', // username
    password: '',
    database: '',
    host: '127.0.0.1',
    port: 3211,
    connectionLimit: 1, // connection number of connection pool, the default is 1
    prefix: '', // data sheet prefix，, if there is more than one item in a database, then the data sheet between the items can be distinguished by the prefix
  }
}
```

In addition to using the host and port to connect to the database, but also through `connectionString` to connect. More configuration options see <https://node-postgres.com/features/connecting>

### Create model file

The model files are placed in the `src/model/` directory (``src/common/model` and `src/[module]/model` in multi-module project), inheriting the model base class `think.Model` with the file format:

```js
// src/model/user.js
module.exports = class extends think.Model {
  getList() {
    return this.field('name').select();
  }
}
```

You can also quickly create model files in the project root via `thinkjs model modelName`.

------

If the project is complex and you want to catalog your model files, you can create subdirectories under the model directory, such as `src/model/front/user.js`, `src/model/admin/user.js`. Create the `front` and` admin` directories under the model directory to manage the front-end and back-end model files separately.

Model instantiation with subdirectories requires subdirectories like `think.model('front/user')`, see [here](/doc/3.0/relation_model.html#toc-9d9).

### Instantiate the model

When the project starts, it scans for all model files (`src/model/` under the project directory, `src/common/model` and various `src/[module]/model` under the multi-module project). After that, all the model classes will be stored in the `think.app.models` object, and will be looked up from this object upon instantiation. If it is not found, the model base class `think.Model` will be instantiated.

#### think.model

Instantiate the model class.

```js
think.model('user'); // get the instance of the model
think.model('user', 'sqlite'); // get the instance of the model, modify the type of database
think.model('user', { // get the instance of the model，modify the type of database and add other arguments
  type: 'sqlite',
  aaa: 'bbb'
});
think.model('user', {}, 'admin'); // get the instance of the model，specified as admin module (valid under multi-module project)
```
#### ctx.model

Instantiate the model class, call the `think.model` method after getting the configuration, and get the configuration under the current module in a multi-module project.

```js
const user = ctx.model('user');
```

#### controller.model

Instantiate the model class, call the `think.model` method after getting the configuration, and get the configuration under the current module in a multi-module project.

```js
module.exports = class extends think.Controller {
  async indexAction() {
    const user = this.model('user'); // instantiate the model in the controller
    const data = await user.select();
    return this.success(data);
  }
}
```

#### service.model

Instantiate the model class, equivalent to `think.model`.

#### Instantiate a model that contains subdirectories

If the model directory contains subdirectories, you need to add the corresponding subdirectory when instantiating, for example:

```js
const user1 = think.model('front/user'); // instantiate the user model for front-end
const user2 = think.model('admin/user'); // instantiate the user model for back-end
```

### CRUD operation

The base class `think.Model` provides a rich way of CRUD operation, the following one by one to introduce.

#### Retrieve data

The model provides several ways to retrieve data, such as:

* [find](/doc/3.0/relation_model.html#toc-a74) query a single data
* [select](/doc/3.0/relation_model.html#toc-3ad) query multiple data
* [count](/doc/3.0/relation_model.html#toc-274) The total number of query
* [countSelect](/doc/3.0/relation_model.html#toc-a39) paging query data
* [max](/doc/3.0/relation_model.html#toc-df2) query the maximum value of the field
* [avg](/doc/3.0/relation_model.html#toc-8d2) query the average value of the field
* [min](/doc/3.0/relation_model.html#toc-1d7) query the minimum value of the field
* [sum](/doc/3.0/relation_model.html#toc-c11) sum the field values
* [getField](/doc/3.0/relation_model.html#toc-f0a) query the value of the specified field

At the same time the model supports the following methods to specify specific conditions in the SQL statement, such as:

* [where](/doc/3.0/relation_model.html#toc-d47) specify the where condition in the SQL statement
* [limit](/doc/3.0/relation_model.html#toc-47d) / [page](/doc/3.0/relation_model.html#toc-a43) specify the limit in the SQL statement
* [field](/doc/3.0/relation_model.html#toc-68b) / [fieldReverse](/doc/3.0/relation_model.html#toc-ad6) specify the field in the SQL statement
* [order](/doc/3.0/relation_model.html#toc-973) specify the order in the SQL statement
* [group](/doc/3.0/relation_model.html#toc-55a) specify the group in the SQL statement
* [join](/doc/3.0/relation_model.html#toc-48b) specify the join in the SQL statement
* [union](/doc/3.0/relation_model.html#toc-ad1) specify the union in the SQL statement
* [having](/doc/3.0/relation_model.html#toc-be2) specify the having in the SQL statement
* [cache](/doc/3.0/relation_model.html#toc-fb8) set the query cache

#### Create data

The model provides the following methods to create data:

* [add](/doc/3.0/relation_model.html#toc-c73) create a single data
* [thenAdd](/doc/3.0/relation_model.html#toc-3e2) add when where condition doesn't exist
* [addMany](/doc/3.0/relation_model.html#toc-a55) add multiple data
* [selectAdd](/doc/3.0/relation_model.html#toc-a56) add result data for subquery

#### Update data

The model provides the following methods to update data:

* [update](/doc/3.0/relation_model.html#toc-b86) update a single data
* [updateMany](/doc/3.0/relation_model.html#updatemany-datalist-options) update multiple data
* [thenUpdate](/doc/3.0/relation_model.html#toc-1b0) conditional update
* [increment](/doc/3.0/relation_model.html#toc-990) the value added to the field
* [decrement](/doc/3.0/relation_model.html#toc-41c) the value to reduce the field

#### Delete data

The model provides the following methods to delete data:

* [delete](/doc/3.0/relation_model.html#toc-866) delete data

#### Manually execute the SQL statement

Sometimes the model packaging method can't meet all the circumstances, this time need to manually specify the SQL statement, you can through the following methods:

* [query](/doc/3.0/relation_model.html#toc-89d) handwritten SQL statement to query
* [execute](/doc/3.0/relation_model.html#toc-a1e) handwritten SQL statement to execute


### Transaction

For data security demanding business (such as: order system, banking system) operation requires the use of transaction, so as to ensure the atomicity of data, consistency, isolation and durability, the model provides a method of operating the transaction.

#### Manally manipulate transaction

You can manipulate transactions manually using the [model.startTrans](/doc/3.0/relation_model.html#toc-0ae), [model.commit](/3.0/relation_model.html#toc-9fc), and [model.rollback](/3.0/relation_model.html#toc-0f2) methods.

#### transaction

每次操作事务时都手工执行 startTrans、commit 和 rollback 比较麻烦，模型提供了 [model.transaction](/doc/3.0/relation_model.html#toc-e30) 方法快速操作事务。
The manual execution of startTrans, commit, and rollback is cumbersome for every transaction, and the model provides the [model.transaction](/doc/3.0/relation_model.html#toc-e30) method for quickly manipulating transactions.

### Set the primary key

The primary key of the datasheet can be set via the `pk` attribute as described in [model.pk](/doc/3.0/relation_model.html#toc-c88).

### Set schema

The data table structure can be set via the `schema` attribute, as described in [model.schema](/doc/3.0/relation_model.html#toc-2d3).

### Related query

Database tables often associated with other data tables, data operations need to operate together with the association table. For example: A blog post will have categories, tags, reviews, and which user it belongs to. The types of support are: one to one, one to one (belong to), one to many and many to many.

The detailed relationship can be configured via the [model.relation](/doc/3.0/relation_model.html#toc-548) attribute.

#### One to one

One to one association, indicating that the current table contains a subsidiary table. Assuming that the model name of the current table is `user` and the model name of the association table is `info`, the default value of `key` in the configuration is `id` and the default value of `fKey` is `user_id`.

```js
module.exports = class extends think.Model {
  get relation() {
    return {
      info: think.Model.HAS_ONE
    };
  }
}
```

When you execute a query, you get data similar to the following:

```js
[
  {
    id: 1,
    name: '111',
    info: { // data in the association table
      user_id: 1,
      desc: 'info'
    }
  }, ...]
```

#### One to one (belong to)

One to one association, belonging to a association table, as opposed to HAS_ONE. Assuming that the current model name is `info` and the name of the association table is `user`, the default value of `key` in the configuration is `user_id`, and the default value of the `fKey` field is `id`.

```js
module.exports = class extends think.Model {
  get relation() {
    return {
      user: think.Model.BELONG_TO
    }
  }
}
```

When you execute a query, you get data similar to the following:

```js
[
  {
    id: 1,
    user_id: 1,
    desc: 'info',
    user: {
      name: 'thinkjs'
    }
  }, ...
]
```

#### One to many

One to many association. If the current model name is `post` and the association table's model name is `comment`, then the configuration field `key` defaults to `id` and the configuration field `fKey` defaults to `post_id`.

```js
module.exports = class extends think.Model {
  get relation() {
    return {
      comment: {
        type: think.Model.HAS_MANY
      }
    }
  }
}
```

When you execute a query, you get data similar to the following:

```js
[{
  id: 1,
  title: 'first post',
  content: 'content',
  comment: [{
    id: 1,
    post_id: 1,
    name: 'welefen',
    content: 'first comment'
  }, ...]
}, ...]
```

If the data in the association table needs to be paged query, it can be done via the [model.setRelation](/doc/3.0/relation_model.html#toc-d7a) method.

#### Many to many

Many to many association. Assuming the current model name is `post` and the associated model name is `cate`, then a corresponding relational table is needed. The configuration field `rModel` defaults to `post_cate` and the configuration field `rfKey` defaults to `cate_id`.


```js
module.exports = class extends think.Model {
  get relation() {
    return {
      cate: {
        type: think.Model.MANY_TO_MANY,
        rModel: 'post_cate',
        rfKey: 'cate_id'
      }
    }
  }
}
```

When you execute a query, you get data similar to the following:

```js
[{
  id: 1,
  title: 'first post',
  cate: [{
    id: 1,
    name: 'cate1',
    post_id: 1
  }, ...]
}, ...]
```

### Distributed / separation of read and write

Sometimes the database needs to use a distributed database, or read and write separation, this time can add `parser` to the configuration to complete, such as:

```js
exports.model = {
  type: 'mysql',
  mysql: {
    user: 'root',
    password: '',
    parser: sql => {
      // here will pass in the current SQL to be executed
      const sqlLower = sql.toLowerCase();
      if(sql.indexOf('select ') === 0) {
        return {
          host: '',
          port: ''
        }
      } else {
        return {
          host: '',
          port: ''
        }
      }
    }
  }
}
```

`parser` can return different configurations based on sql and will merge the returned configuration with the default configuration.

### FAQ

#### What is the maximum number of connections to the database?

Assuming the project has two clusters, each cluster has ten machines, each machine has four workers enabled, and the number of connections in the connection pool of database configuration is five, then the overall maximum number of connections is: `2 * 10 * 4 * 5 = 400`

#### How to check related debugging information?

The debug name used by the model is `think-model`, which can be started with `DEBUG = think-model npm start` and checked for debugging information.

### API

#### model.schema

Set the table structure, the default access from the data table, you can also configure additional configuration items.

```js
module.exports = class extends think.Model {
  get schema() {
    return {
      id: { // field name
        type: 'int(11)',
        ...
      }
    }
  }
}
```
Supported fields are:

* `type` {String} the type of field, including the length attribute
* `required` {Boolean} required or not
* `default` {mixed} the default value, can be a value or a function

  ```js
  module.exports = class extends think.Model {
    get schema() {
      return {
        type: { // field name
          type: 'varchar(10)',
          default: 'small'
        },
        create_time: {
          type: 'datetime',
          default: () => think.datetime() // default is a function
        },
        score: {
          type: 'int',
          default: data => { // data is added / updated data
            return data.grade * 1.5;
          }
        }
      }
    }
  }
  ```
* `primary` {boolean} is the primary key or not
* `unique` {boolean} whether the field is unique
* `autoIncrement` {boolean} whether or not `auto increment`
* `readonly` {boolean} whether the field is read-only, can only be added when creating, can't update the field
* `update` {boolean} whether the default value is valid also when updating. If `readonly` is set, then this field is invalid.


#### model.relation

Configure the association of data tables.

```js
module.exports = class extends think.Model {
  // configure the association
  get relation() {
    return {
      cate: { // configure the relationship with the classification
        type: think.Model.MANY_TO_MANY,
        ...
      },
      comment: { // configure the association with comments

      }
    }
  }
}
```

The configuration supported by each association is as follows:

* `type` association type, default is `think.Model.HAS_ONE`

  ```
  One to one: think.Model.HAS_ONE
  One to one (belong to): think.Model.BELONG_TO
  One to many: think.Model.HAS_MANY
  Many to many: think.Model.MANY_TO_MANY
  ```
* `model` the model name of the association table, the default is the key of configuration

  ```
  When instantiating the corresponding relational model, the relational model is instantiated via const relationModel = this.model(item.model)
  ```
* `name` the corresponding data field name, the default is the key of configuration, after querying the data, save the field name.

  ```
  // origin data
  const originData = {
    id: 1,
    email: ''
  }
  // set the corresponding data field named cate
  // then the final generated data is
  const targetData = {
    id: 1,
    email: '',
    cate: {

    }
  }
  ```
* `key` the current model's associated key

  ```
  One to one, one to many, many to many the default value is the primary key of the current model, such as: id
  One to one (belong to) the default value is a combination of the name of the association table and id, such as: cate_id
  ```
* `fKey` the association table corresponding key

  ```
  One to one, one to many, many to many the default value is a combination of table name and id, such as: cate_id
  One to one (Belonging) the default value for the current model's primary key, such as: id
  ```

* `field` field set when the association table query, the default value is `*`. If you need to set, must contain the value of `fKey`, support function.

  ```
  // set the field field
  get relation() {
    return {
      cate: {
        field: 'id,name' // only query id, name field
      }
    }
  }

  // set the field to function
  get relation() {
    return {
      cate: {
        // rModel is an instance of the associated model and model is an instance of the current model
        field: (rModel, model) => {
          return 'id,name'
        }
      }
    }
  }
  ```
* `where` where conditions need to be set in the association table query, support function
* `order` the order need to be set in the association table query, support function
* `limit` the limit need to be set in the association table query, support function
* `page` the page need to be set in the association table query, support function
* `rModel` many to many association, the corresponding associated model name, the default value is a combination of two model names, such as: `article_cate`

  ```

  In the many to many association model, an intermediate relational table is generally required to maintain the association. For example, the article and the cate （category） are many to many association, then you need an article-category intermediate relational table (article_cate), RModel is the model name of the intermediate relation table in the configuration.

  ```
* `rfKey` the corresponding key of association table in many to many association
* `relation` whether to close the relation of the association table

  ```
  // if the association table is configured with the relationship and query together while doing the query
  // sometimes don't want to query the association table associated data, then you can close the relation property
  get relation() {
    return {
      cate: {
        relation: false // close all the relation of the association table, to avoid problems such as death cycle
      }
    }
  }
  ```

#### model.setRelation(name, value)

After setting the association, the query and other operations will automatically query the data of the association table. If you do not need to query the data of the association table in some cases, you can temporarily close the relational query through the `setRelation` method.

##### All disabled

Disable all the relational queries via `setRelation(false)`.

```js
module.exports = class extends think.Model {
  constructor(...args){
    super(...args);
    this.relation = {
      comment: think.Model.HAS_MANY,
      cate: think.Model.MANY_TO_MANY
    };
  }
  getList(){
    return this.setRelation(false).select();
  }
}
```

##### Partially enabled

Only query the relevant data of the `comment` by `setRelation('comment')`, don't query other relational data.

```js
module.exports = class extends think.Model {
  constructor(...args){
    super(...args);
    this.relation = {
      comment: think.Model.HAS_MANY,
      cate: think.Model.MANY_TO_MANY
    };
  }
  getList2(){
    return this.setRelation('comment').select();
  }
}
```

##### Partially disabled

Disable the relational data query for `comment` via `setRelation('comment', false)`.

```js
module.exports = class extends think.Model {
  constructor(...args){
    super(...args);
    this.relation = {
      comment: think.Model.HAS_MANY,
      cate: think.Model.MANY_TO_MANY
    };
  }
  getList2(){
    return this.setRelation('comment', false).select();
  }
}
```

##### All re-enabled

Re-enable all relational data query via `setRelation(true)`.

```js
module.exports = class extends think.Model {
  constructor(...args){
    super(...args);
    this.relation = {
      comment: think.Model.HAS_MANY,
      cate: think.Model.MANY_TO_MANY
    };
  }
  getList2(){
    return this.setRelation(true).select();
  }
}
```

##### Dynamically modify configuration

Although the relation is configured by the relation attribute, but sometimes when you want to dynamically modify some of the values, such as: set the paging, this time can also be done through the `setRelation` method.

```js
module.exports = class extends think.Model {
  constructor(...args){
    super(...args);
    this.relation = {
      comment: think.Model.HAS_MANY,
      cate: think.Model.MANY_TO_MANY
    };
  }
  getList2(page){
    // dynamically set the comment page
    return this.setRelation('comment', {page}).select();
  }
}
```

#### model.db(db)

Get or set an instance of db, db is an instance of Adapter handle (such as `think-model-mysql`). This method is required for transactional operations because of the reuse of a connection.

```js
module.exports = class extends think.Model {
  async getList() {
    // let user reuse current Apdater handle instance, so follow-up can reuse the same database connection
    const user = this.model('user').db(this.db());
  }
}
```

#### model.modelName

The model name that is passed in when the model is instantiated.

```js
const user = think.model('user');
```

The model passed in instantiation is named `user`, so `model.modelName` is `user`.

#### model.config

Incoming configuration when instantiating the model, configuration will automatically transfer without manual assignment.

```js
{
  host: '127.0.0.1',
  port: 3306,
  ...
}
```

#### model.tablePrefix

Obtain the data table prefix, obtained from the `prefix` field in the configuration. If you want to modify it, you can through the following ways:

```js
module.exports = class extends think.Model {
  get tablePrefix() {
    return 'think_';
  }
}
```

#### model.tableName

Get the data table name, the value is `tablePrefix + modelName`. If you want to modify it, you can through the following ways:

```js
module.exports = class extends think.Model {
  get tableName() {
    return 'think_user';
  }
}
```

#### model.pk

Get the primary key of the data table, the default is `id`. If the data table's primary key isn't id, you need to configure it, such as:

```js
module.exports = class extends think.Model {
  get pk() {
    return 'user_id';
  }
}
```

If you didn't write the model file but instantiated directly in the controller, then you want to change the name of the primary key, you can set the `_pk` property, such as:

```js
module.exports = class extends think.Controller {
  async indexAction() {
    const user = this.model('user');
    user._pk = 'user_id'; // set the primary key via the _pk property
    const data = await user.select();
  }
}
```

#### model.options

Some options for model operation, setting `where`, `limit`, `group` and other operations will eventually be resolved to the `options`, the format is:

```js
{
  where: {}, // the configuration to store where condition
  limit: {}, // the configuration to store limit
  group: {},
  ...
}
```

#### model.lastSql

Get the most recent execution of the SQL statement, the default value is empty.

```js
const user = think.model('user');
console.log(user.lastSql); // print a recent sql statement, if not return empty.
```

#### model.model(name)

* `name` {String} name of the model to be instantiated
* `return` {this} model instance

Instantiate other models that support sub-directory model instantiation.

```js
module.exports = class extends think.Model {
  async getList() {
    // If you have subdirectories, add subdirectories here, such as: this.model('front/article')
    const article = this.model('article');
    const data = await article.select();
    ...
  }
}
```

#### model.limit(offset, length)

* `offset` {Number} offset in the SQL statement
* `length` {Number} length in the SQL statement
* `return` {this}

Setting the `limit` in the SQL statement will be assigned to the `this.options.limit` attribute for subsequent parsing.

```js
module.exports = class extends think.Model() {
  async getList() {
    // SQL: SELECT * FROM `test_d` LIMIT 10
    const list1 = await this.limit(10).select();
    // SQL: SELECT * FROM `test_d` LIMIT 10,20
    const list2 = await this.limit(10, 20).select();
  }
}
```

#### model.page(page, pagesize)

* `page` {Number} set the current page number
* `pagesize` {Number} the size of each page, the default is `this.config.pagesize`
* `return` {this}

Set query paging, will be parsed to [limit](/doc/3.0/relation_model.html#toc-47d) data.

```js
module.exports = class extends think.Model() {
  async getList() {
    // SQL: SELECT * FROM `test_d` LIMIT 0,10
    const list1 = await this.page(1).select(); // query the first page, 10 per page
    // SQL: SELECT * FROM `test_d` LIMIT 20,20
    const list2 = await this.page(2, 20).select(); // query the second page, 20 per page
  }
}
```

The size of each page can be modified by the configuration item `pageSize`, such as:

```js
// src/config/adapter.js
exports.model = {
  type: 'mysql',
  mysql: {
    database: '',
    ...
    pageSize: 20, // set the default to 20 per page
  }
}
```

#### model.where(where)

* `where` {String | Object} set query conditions
* `return` {this}

Set the where query condition and add `this.options.where` attribute for subsequent analysis. The logic can be set via the property `_logic`, which defaults to `AND`. The compound query can be set via the property `_complex`.

`Note: Values in the where condition must be verified in Logic, or there may be SQL injection vulnerabilities.`

##### Common conditions

```js
module.exports = class extends think.Model {
  where1(){
    //SELECT * FROM `think_user`
    return this.where().select();
  }
  where2(){
    //SELECT * FROM `think_user` WHERE ( `id` = 10 )
    return this.where({id: 10}).select();
  }
  where3(){
    //SELECT * FROM `think_user` WHERE ( id = 10 OR id < 2 )
    return this.where('id = 10 OR id < 2').select();
  }
  where4(){
    //SELECT * FROM `think_user` WHERE ( `id` != 10 )
    return this.where({id: ['!=', 10]}).select();
  }
}
```

##### null condition

```js
module.exports = class extends think.Model {
  where1(){
    //SELECT * FROM `think_user` where ( title IS NULL );
    return this.where({title: null}).select();
  }
  where2(){
    //SELECT * FROM `think_user` where ( title IS NOT NULL );
    return this.where({title: ['!=', null]}).select();
  }
}
```

##### EXP condition

ThinkJS defaults to escaping fields and values to prevent security holes. Sometimes some special circumstances don't want to be escaped, you can use the EXP approach, such as:

```js
module.exports = class extends think.Model {
  where1(){
    //SELECT * FROM `think_user` WHERE ( (`name` ='name') )
    return this.where({name: ['EXP', "=\"name\""]}).select();
  }
}
```

##### LIKE condition

```js
module.exports = class extends think.Model {
  where1(){
    //SELECT * FROM `think_user` WHERE ( `title` NOT LIKE 'welefen' )
    return this.where({title: ['NOTLIKE', 'welefen']}).select();
  }
  where2(){
    //SELECT * FROM `think_user` WHERE ( `title` LIKE '%welefen%' )
    return this.where({title: ['like', '%welefen%']}).select();
  }
  //like multiple values
  where3(){
    //SELECT * FROM `think_user` WHERE ( (`title` LIKE 'welefen' OR `title` LIKE 'suredy') )
    return this.where({title: ['like', ['welefen', 'suredy']]}).select();
  }
  // multiple fields OR Like a value
  where4(){
    //SELECT * FROM `think_user` WHERE ( (`title` LIKE '%welefen%') OR (`content` LIKE '%welefen%') )
    return this.where({'title|content': ['like', '%welefen%']}).select();
  }
  // multiple fields AND Like a value
  where5(){
    //SELECT * FROM `think_user` WHERE ( (`title` LIKE '%welefen%') AND (`content` LIKE '%welefen%') )
    return this.where({'title&content': ['like', '%welefen%']}).select();
  }
}
```


##### IN condition

```js
module.exports = class extens think.Model {
  where1(){
    //SELECT * FROM `think_user` WHERE ( `id` IN ('10','20') )
    return this.where({id: ['IN', '10,20']}).select();
  }
  where2(){
    //SELECT * FROM `think_user` WHERE ( `id` IN (10,20) )
    return this.where({id: ['IN', [10, 20]]}).select();
  }
  where3(){
    //SELECT * FROM `think_user` WHERE ( `id` NOT IN (10,20) )
    return this.where({id: ['NOTIN', [10, 20]]}).select();
  }
}
```

##### BETWEEN query

```js
module.exports = class extens think.Model {
  where1(){
    //SELECT * FROM `think_user` WHERE (  (`id` BETWEEN 1 AND 2) )
    return this.where({id: ['BETWEEN', 1, 2]}).select();
  }
  where2(){
    //SELECT * FROM `think_user` WHERE (  (`id` BETWEEN '1' AND '2') )
    return this.where({id: ['between', '1,2']}).select();
  }
}
```

##### Multi-field query

```js
module.exports = class extends think.Model {
  where1(){
    //SELECT * FROM `think_user` WHERE ( `id` = 10 ) AND ( `title` = 'www' )
    return this.where({id: 10, title: "www"}).select();
  }
  // change to OR
  where2(){
    //SELECT * FROM `think_user` WHERE ( `id` = 10 ) OR ( `title` = 'www' )
    return this.where({id: 10, title: "www", _logic: 'OR'}).select();
  }
  //change to XOR
  where2(){
    //SELECT * FROM `think_user` WHERE ( `id` = 10 ) XOR ( `title` = 'www' )
    return this.where({id: 10, title: "www", _logic: 'XOR'}).select();
  }
}
```

##### Multiple conditions query

```js
module.exports = class extends think.Model {
  where1(){
    //SELECT * FROM `think_user` WHERE ( `id` > 10 AND `id` < 20 )
    return this.where({id: {'>': 10, '<': 20}}).select();
  }
  // change to OR
  where2(){
    //SELECT * FROM `think_user` WHERE ( `id` < 10 OR `id` > 20 )
    return this.where({id: {'<': 10, '>': 20, _logic: 'OR'}}).select()
  }
}
```

##### Compound query

```js
module.exports = class extends think.Model {
  where1(){
    //SELECT * FROM `think_user` WHERE ( `title` = 'test' ) AND (  ( `id` IN (1,2,3) ) OR ( `content` = 'www' ) )
    return this.where({
      title: 'test',
      _complex: {id: ['IN', [1, 2, 3]],
        content: 'www',
        _logic: 'or'
      }
    }).select()
  }
}
```


#### model.field(field)

* `field` {String} query field, support `AS`。
* `return` {this}

Set the query field in the SQL statement, the default is `*`. The value will be assigned to the `this.options.field` property, for subsequent analysis.

```js
module.exports = class extends think.Model{
  async getList() {
    // SQL: SELECT `d_name` FROM `test_d`
    const data1 = await this.field('d_name').select();

    // SQL: SELECT `c_id`,`d_name` FROM `test_d`
    const data2 = await this.field('c_id,d_name').select();

    // SQL: SELECT c_id AS cid,`d_name` FROM `test_d`
    const data3 = await this.field('c_id AS cid, d_name').select();
  }
}
```

#### model.fieldReverse(field)

* `field` {String} query field, `AS` is not supported。
* `return` {this}

If you set an anti-election field (ie, don't query the configured fields but query other fields), the `this.options.field` and `this.options.fieldReverse` properties are added to facilitate subsequent analysis.

The implementation of this function: Query all the fields in the data table, and then filter out the configuration field.


```js
module.exports = class extends think.Model{
  async getList() {
    // SQL: SELECT `id`, `c_id` FROM `test_d`
    const data1 = await this.fieldReverse('d_name').select();
  }
}
```

#### model.table(table, hasPrefix)

* `table` {String} table name, support for a SELECT statement
* `hasPrefix` {Boolean} whether the `table` already contains a table prefix, the default value is `false`
* `return` {this}

Set the table name corresponding to the current model. If `hasPrefix` is false and `table` is not an SQL statement, the table name will be appended with `tablePrefix`, and the last value will be set to the `this.options.table` attribute.

If this property is not set then the table name is taken from the `mode.tableName` property when the SQL is last parsed.

#### model.union(union, all)

* `union` {String} union query fields
* `all` {boolean} whether to use UNION ALL
* `return` {this}

Set SQL UNION query and add `this.options.union` attribute, for subsequent analysis.

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` UNION (SELECT * FROM think_pic2)
    return this.union('SELECT * FROM think_pic2').select();
  }
  getList2(){
    //SELECT * FROM `think_user` UNION ALL (SELECT * FROM `think_pic2`)
    return this.union({table: 'think_pic2'}, true).select();
  }
}
```

#### model.join(join)

* `join` {String | Object | Array} query statement to be combined, the default is `LEFT JOIN`
* `return` {this}

Combination of queries, support for strings, arrays and objects, and many other ways. And add the `this.options.join` property for subsequent analysis.

##### String

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` LEFT JOIN think_cate ON think_group.cate_id=think_cate.id
    return this.join('think_cate ON think_group.cate_id=think_cate.id').select();
  }
}
```

##### Array

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` LEFT JOIN think_cate ON think_group.cate_id=think_cate.id RIGHT JOIN think_tag ON think_group.tag_id=think_tag.id
    return this.join([
      'think_cate ON think_group.cate_id=think_cate.id',
      'RIGHT JOIN think_tag ON think_group.tag_id=think_tag.id'
    ]).select();
  }
}
```

##### Object: a single table

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` INNER JOIN `think_cate` AS c ON think_user.`cate_id`=c.`id`
    return this.join({
      table: 'cate',
      join: 'inner', // join way, there are left, right, inner 3 ways.
      as: 'c', // table alias
      on: ['cate_id', 'id'] //ON condition
    }).select();
  }
}
```

##### Object: JOIN multiple times

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM think_user AS a LEFT JOIN `think_cate` AS c ON a.`cate_id`=c.`id` LEFT JOIN `think_group_tag` AS d ON a.`id`=d.`group_id`
    return this.alias('a').join({
      table: 'cate',
      join: 'left',
      as: 'c',
      on: ['cate_id', 'id']
    }).join({
      table: 'group_tag',
      join: 'left',
      as: 'd',
      on: ['id', 'group_id']
    }).select()
  }
}
```


##### Object: multiple tables

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` LEFT JOIN `think_cate` ON think_user.`id`=think_cate.`id` LEFT JOIN `think_group_tag` ON think_user.`id`=think_group_tag.`group_id`
    return this.join({
      cate: {
        on: ['id', 'id']
      },
      group_tag: {
        on: ['id', 'group_id']
      }
    }).select();
  }
}
```

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM think_user AS a LEFT JOIN `think_cate` AS c ON a.`id`=c.`id` LEFT JOIN `think_group_tag` AS d ON a.`id`=d.`group_id`
    return this.alias('a').join({
      cate: {
        join: 'left', // there left, right, inner 3 values
        as: 'c',
        on: ['id', 'id']
      },
      group_tag: {
        join: 'left',
        as: 'd',
        on: ['id', 'group_id']
      }
    }).select()
  }
}
```

##### Object: The ON condition contains more than one field

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` LEFT JOIN `think_cate` ON think_user.`id`=think_cate.`id` LEFT JOIN `think_group_tag` ON think_user.`id`=think_group_tag.`group_id` LEFT JOIN `think_tag` ON (think_user.`id`=think_tag.`id` AND think_user.`title`=think_tag.`name`)
    return this.join({
      cate: {on: 'id, id'},
      group_tag: {on: ['id', 'group_id']},
      tag: {
        on: { // Multiple fields ON
          id: 'id',
          title: 'name'
        }
      }
    }).select()
  }
}
```

##### Object: table value is a SQL statement

```js
module.exports = class extends think.Model {
  async getList(){
    let sql = await this.model('group').buildSql();
    //SELECT * FROM `think_user` LEFT JOIN ( SELECT * FROM `think_group` ) ON think_user.`gid`=( SELECT * FROM `think_group` ).`id`
    return this.join({
      table: sql,
      on: ['gid', 'id']
    }).select();
  }
}
```

#### model.order(order)

* `order` {String | Array | Object} sort method
* `return` {this}

Set the sort method in SQL and add `this.options.order` attribute, easy to follow-up analysis.

##### String

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` ORDER BY id DESC, name ASC
    return this.order('id DESC, name ASC').select();
  }
  getList1(){
    //SELECT * FROM `think_user` ORDER BY count(num) DESC
    return this.order('count(num) DESC').select();
  }
}
```



##### Array

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` ORDER BY id DESC,name ASC
    return this.order(['id DESC', 'name ASC']).select();
  }
}
```

##### Object

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` ORDER BY `id` DESC,`name` ASC
    return this.order({
      id: 'DESC',
      name: 'ASC'
    }).select();
  }
}
```


#### model.alias(aliasName)

* `aliasName` {String} table alias
* `return` {this}

Set table alias and add the `this.options.alias` attribute for subsequent analysis.

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM think_user AS a;
    return this.alias('a').select();
  }
}
```

#### model.having(having)

* `having` {String} having query string
* `return` {this}

Set having query and set the `this.options.having` attribute for subsequent analysis.

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` HAVING view_nums > 1000 AND view_nums < 2000
    return this.having('view_nums > 1000 AND view_nums < 2000').select();
  }
}
```


#### model.group(group)

* `group` {String} Fields for grouping query
* `return` {this}

Set grouping query and set the `this.options.group` attribute for subsequent analysis.

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` GROUP BY `name`
    return this.group('name').select();
  }
}
```

#### model.distinct(distinct)

* `distinct` {String} the field need deduplicate
* `return` {this}

Deduplicate query and set `this.options.distinct` attribute for subsequent analysis.

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT DISTINCT `name` FROM `think_user`
    return this.distinct('name').select();
  }
}
```


#### model.beforeAdd(data)

* `data` {Object} the data to be added

Pre-operation of add.

#### model.afterAdd(data)

* `data` {Object} the data to be added

Follow-up operation of add.

#### model.afterDelete(data)

Follow-up operation of delete.

#### model.beforeUpdate(data)

* `data` {Object} the data to be updated

Pre-operation of update.

Sometimes you need a function to update the value when it is submitted and not update when it is null, then you can use this method to operate:

```js
module.exports = class extends think.Model {
  beforeUpdate(data) {
    for (const key in data) {
      // Not updated if value is empty
      if(data[key] === '') {
        delete data[key];
      }
    }
    return data;
  }
}
```

#### model.afterUpdate(data)

* `data` {Object} data to update

Post operation of update。

#### model.afterFind(data)

* `data` {Object} a single data to query
* `return` {Object | Promise}

Follow-up operation of `find` query.

#### model.afterSelect(data)

* `data` [Array] data to query
* `return` {Array | Promise}

Follow-up operation of `select` query.



#### model.add(data, options)

* `data` {Object} the data to be added, if some of the data in the field doesn't exist in the data table will automatically be filtered out
* `options` {Object} Operation options are resolved via the [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} return the ID inserted

Add a piece of data, the return value is insert data id.

The return value may be 0 if the data table has no primary key or no attribute such as `auto increment` is set. If you manually set the value of the primary key when inserting data, the return value may also be 0.

```js
module.exports = class extends think.Controller {
  async addAction(){
    let model = this.model('user');
    let insertId = await model.add({name: 'xxx', pwd: 'yyy'});
  }
}
```

Sometimes project need to use some functions of the database to add data, such as: timestamp using mysql `CURRENT_TIMESTAMP` function, then you can use `exp` expression to complete.

```js
module.exports = class extends think.Controller {
  async addAction(){
    let model = this.model('user');
    let insertId = await model.add({
      name: 'test',
      time: ['exp', 'CURRENT_TIMESTAMP()']
    });
  }
}
```

#### model.thenAdd(data, where)

* `data` {Object} the data to be added
* `where` {Object} where condition, where condition is set by [where](/doc/3.0/relation_model.html#toc-d47) method
* `return` {Promise}

Add data when the where condition hasn't hit any data.

```js
module.exports = class extends think.Controller {
  async addAction(){
    const model = this.model('user');
    // the first parameter is the data to be added, the second parameter is added condition. Added only when no related records are queried based on the condition of the second parameter
    const result = await model.thenAdd({name: 'xxx', pwd: 'yyy'}, {email: 'xxx'});
    // result returns {id: 1000, type: 'add'} or {id: 1000, type: 'exist'}
  }
}
```

Where conditions can also be directly specified by `this.where` method, such as:

```js
module.exports = class extends think.Controller {
  async addAction(){
    const model = this.model('user');
    const result = await model.where({email: 'xxx'}).thenAdd({name: 'xxx', pwd: 'yyy'});
    // result returns {id: 1000, type: 'add'} or {id: 1000, type: 'exist'}
  }
}
```


#### model.addMany(dataList, options)

* `dataList` {Array} the list of data to be added
* `options` {Object} Operation options are resolved via the [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} return the list of inserted IDs

Add multiple pieces of data at once.

```js
module.exports = class extends think.Controller {
  async addAction(){
    let model = this.model('user');
    let insertIds = await model.addMany([
      {name: 'xxx', pwd: 'yyy'},
      {name: 'xxx1', pwd: 'yyy1'}
    ]);
  }
}
```

#### model.selectAdd(fields, table, options)

* `fields` {Array | String} field name
* `table` {String} table name
* `options` {Object} Operation options are resolved via the [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} return the ID inserted

Add the result data of the subquery parsed from options.

```js
module.exports = class extends think.Controller {
  async addAction(){
    let model = this.model('user');
    let insertIds = await model.selectAdd(
      'xxx,xxx1,xxx2',
      'tableName',
      {
        id: '1'
      }
    );
  }
}
```


#### model.delete(options)

* `options` {Object} Operation options are resolved via the [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} return the number of rows affected

Delete data.

```js
module.exports = class extends think.Controller {
  async deleteAction(){
    let model = this.model('user');
    let affectedRows = await model.where({id: ['>', 100]}).delete();
  }
}
```


#### model.update(data, options)

* `data` {Object} data to update
* `options` {Object} Operation options are resolved via the [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} return the number of rows affected

Update data.

```js
module.exports = class extends think.Controller {
  async updateAction(){
    let model = this.model('user');
    let affectedRows = await model.where({name: 'thinkjs'}).update({email: 'admin@thinkjs.org'});
  }
}
```

By default, the WHERE condition must be added to update the data to prevent any misoperation causing all data to be incorrectly updated. If it is confirmed that you need to update all the data, you can add where `1=1` conditions, such as:

```js
module.exports = class extends think.Controller {
  async updateAction(){
    let model = this.model('user');
    let affectedRows = await model.where('1=1').update({email: 'admin@thinkjs.org'});
  }
}
```

Sometimes update values need to rely on the database function or other fields, this time can be done with the help of `exp`.

```js
module.exports = class extends think.Controller {
  async updateAction(){
    let model = this.model('user');
    let affectedRows = await model.where('1=1').update({
      email: 'admin@thinkjs.org',
      view_nums: ['exp', 'view_nums+1'],
      update_time: ['exp', 'CURRENT_TIMESTAMP()']
    });
  }
}
```


#### model.thenUpdate(data, where)

* `data` {Object} data to update
* `where` {Object} where condition
* `return` {Promise}

Add data when the where condition hasn't hit any data, else update the data.

#### updateMany(dataList, options)

* `dataList` {Array} the list of data to uodate
* `options` {Object} Operation options are resolved via the [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} return the number of rows affected

Update multiple data, the `dataList` must contain the value of the primary key, which will be automatically set to update the conditions.

```js
this.model('user').updateMany([{
  id: 1, // the data must contain the value of the primary key
  name: 'name1'
}, {
  id: 2,
  name: 'name2'
}])
```

#### model.increment(field, step)

* `field` {String} field name
* `step` {Number} increment value, default is 1
* `return` {Promise}

Increment field value.

```js
module.exports = class extends think.Model {
  updateViewNums(id){
    return this.where({id: id}).increment('view_nums', 1); //view number plus one
  }
}
```

#### model.decrement(field, step)

* `field` {String} field name
* `step` {Number} decrement value, default is 1
* `return` {Promise}

Decrement field value.

```js
module.exports = class extends think.Model {
  updateViewNums(id){
    return this.where({id: id}).decrement('coins', 10); //coins minus 10
  }
}
```


#### model.find(options)

* `options` {Object} Operation options are resolved via the [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} return a single piece of data

Query a single data, the data type returned as an object. If no relevant data is found, the return value is `{}`.

```js
module.exports = class extends think.Controller {
  async listAction(){
    let model = this.model('user');
    let data = await model.where({name: 'thinkjs'}).find();
    //data returns {name: 'thinkjs', email: 'admin@thinkjs.org', ...}
    if(think.isEmpty(data)) {
      // when the content is empty
    }
  }
}
```

The [think.isEmpty](/doc/3.0/think.html#toc-df2) method can be used to determine if the return value is empty.

#### model.select(options)

* `options` {Object} Operation options are resolved via the [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} return multiple data

Query multiple data, the data type returned is an array. If no relevant data is found, the return value is `[]`.

```js
module.exports = class extends think.Controller {
  async listAction(){
    let model = this.model('user');
    let data = await model.limit(2).select();
    //data returns [{name: 'thinkjs', email: 'admin@thinkjs.org'}, ...]
    if(think.isEmpty(data)){

    }
  }
}
```

The [think.isEmpty](/doc/3.0/think.html#toc-df2) method can be used to determine if the return value is empty.

#### model.countSelect(options, pageFlag)

* `options` {Number | Object} Operation options are resolved via the [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `pageFlag` {Boolean} when the number of pages isn't legal, true is amended to the first page, false is amended to the last page, the default doesn't do it
* `return` {Promise}

Paging queries, in general, need to be combined with the `page` method. Such as:

```js
module.exports = class extends think.Controller {
  async listAction(){
    let model = this.model('user');
    let data = await model.page(this.get('page')).countSelect();
  }
}
```

The return value data structure is as follows:

```js
{
  pagesize: 10, // the size of each page
  currentPage: 1, // current page
  count: 100, // total number
  totalPages: 10, // total pages
  data: [{ // the data list in the current page
    name: "thinkjs",
    email: "admin@thinkjs.org"
  }, ...]
}
```

Sometimes the total number is stored in other tables, don't need to check the current table to get the total number, this time can be the first parameter `options` set to the total number of queries.

```js
module.exports = class extends think.Controller {
  async listAction(){
    const model = this.model('user');
    const total = 256;
    // specify the total number of queries
    const data = await model.page(this.get('page')).countSelect(total);
  }
}
```

#### model.getField(field, num)

* `field` {String} field name, multiple fields are separated by commas
* `num` {Boolean | Number} the number of need
* `return` {Promise}

Get the value of a specific field, you can set `where`, `group` and other conditions.

** Get all the list of individual fields **

```js
module.exports = class extends think.Controller {
  async listAction(){
    const data = await this.model('user').getField('c_id');
    // data = [1, 2, 3, 4, 5]
  }
}
```

** Specified number to get a list of individual fields **

```js
module.exports = class extends think.Controller {
  async listAction(){
    const data = await this.model('user').getField('c_id', 3);
    // data = [1, 2, 3]
  }
}
```

** Gets a single value of a single field **

```js
module.exports = class extends think.Controller {
  async listAction(){
    const data = await this.model('user').getField('c_id', true);
    // data = 1
  }
}
```

** Get all the list of multiple fields **

```js
module.exports = class extends think.Controller {
  async listAction(){
    const data = await this.model('user').getField('c_id,d_name');
    // data = {c_id: [1, 2, 3, 4, 5], d_name: ['a', 'b', 'c', 'd', 'e']}
  }
}

```


** Gets all the lists for the specified number of multiple fields **

```js
module.exports = class extends think.Controller {
  async listAction(){
    const data = await this.model('user').getField('c_id,d_name', 3);
    // data = {c_id: [1, 2, 3], d_name: ['a', 'b', 'c']}
  }
}
```

** Get a single value of multiple fields **

```js
module.exports = class extends think.Controller {
  async listAction(){
    const data = await this.model('user').getField('c_id,d_name', true);
    // data = {c_id: 1, d_name: 'a'}
  }
}
```

#### model.count(field)

* `field` {String} field name, default is `*`
* `return` {Promise} return the total number

Get total number.

```js
module.exports = class extends think.Model{
  // get the sum of the field values
  getScoreCount() {
    // SELECT COUNT(score) AS think_count FROM `test_d` LIMIT 1
    return this.count('score');
  }
}
```

#### model.sum(field)

* `field` {String} field name
* `return` {Promise}

Sum the field values.

```js
module.exports = class extends think.Model{
  // get the sum of the field values
  getScoreSum() {
    // SELECT SUM(score) AS think_sum FROM `test_d` LIMIT 1
    return this.sum('score');
  }
}
```

#### model.min(field)

* `field` {String} field name
* `return` {Promise}

Find the minimum value of the field.


```js
module.exports = class extends think.Model{
  // get the minimum value
  getScoreMin() {
    // SELECT MIN(score) AS think_min FROM `test_d` LIMIT 1
    return this.min('score');
  }
}
```

#### model.max(field)

* `field` {String} field name
* `return` {Promise}

Find the maximum value of the field.

```js
module.exports = class extends think.Model{
  // get the maximum value
  getScoreMax() {
    // SELECT MAX(score) AS think_max FROM `test_d` LIMIT 1
    return this.max('score');
  }
}
```

#### model.avg(field)

* `field` {String} field name
* `return` {Promise}

Find the average of the field.

```js
module.exports = class extends think.Model{
  // get the average value
  getScoreAvg() {
    // SELECT AVG(score) AS think_avg FROM `test_d` LIMIT 1
    return this.avg('score');
  }
}
```

#### model.query(sqlOptions)

* `sqlOptions` {String | Object} SQL option to execute
* `return` {Promise} data to query

Specifying a SQL statement to execute the query, `sqlOptions` is resolved via the [parseSql](/doc/3.0/relation_model.html#toc-ec3) method, which requires that you handle your own security issues when executing SQL statements.

```js
module.exports = class extends think.Model {
  getMysqlVersion() {
    return this.query('select version();');
  }
}
```


#### model.execute(sqlOptions)

* `sqlOptions` {String | Object} SQL option to operate
* `return` {Promise}

Specifying a SQL statement to execute the query, `sqlOptions` is resolved via the [parseSql](/doc/3.0/relation_model.html#toc-ec3) method, which requires that you handle your own security issues when executing SQL statements.

```js
module.exports = class extends think.Model {
  xxx() {
    return this.execute('set @b=5;call proc_adder(2,@b,@s);');
  }
}
```


#### model.parseSql(sqlOptions, ...args)

* `sqlOptions` {String | Object} SQL option to parse
* `...args` {Array} data to parse
* `return` {Object}

Parsing SQL statements, the SQL statement `__TABLENAME__` resolve to the corresponding table name. Args data is parsed into sql via [util.format](https://nodejs.org/api/util.html#util_util_format_format_args).

```js
module.exports = class extends think.Model {
  getSql(){
    const sql = 'SELECT * FROM __GROUP__ WHERE id=10';
    const sqlOptions = this.parseSql(sql);
    //{sql: "SELECT * FROM think_group WHERE id=10"}
  }
  getSql2(){
    const sql = 'SELECT * FROM __GROUP__ WHERE id=10';
    const sqlOptions = this.parseSql({sql, debounce: false});
    //{sql: SELECT * FROM think_group WHERE id=10", debounce: false}
  }
}
```

#### model.parseOptions(options)

* `options` {Object} the options to be merged, and will be combined into `this.options` for parsing
* `return` {Promise}

Resolution options. The where, limit, group, and the other operations set the corresponding property to `this.options`, which parses `this.options` and appends the corresponding properties so they are needed for subsequent processing.

```js
const options = await this.parseOptions({limit: 1});
/**
options = {
  table: '',
  tablePrefix: '',
  pk: '',
  field: '',
  where: '',
  limit: '',
  group: '',
  ...
}
*/
```

After calling `this.parseOptions`, the `this.options` property will be set to empty object `{}`.

#### model.startTrans()

* `return` {Promise}

Start the transaction.

#### model.commit()

* `return` {Promise}

Commit the transaction.

#### model.rollback()

* `return` {Promise}

Rollback the transaction.

```js
module.exports = class extends think.Model {
  async addData() {
    // commit if commit is succsessful, rollback if failed
    try {
      await this.startTrans();
      const result = await this.add({});
      await this.commit();
      return result;
    } catch(e){
      await this.rollback();
    }
  }
}
```

If you need to instantiate multiple model operations during a transaction, you need to reuse the same database connection between models, as described in [model.db](/doc/3.0/relation_model.html#toc-f95).

#### model.transaction(fn)

* `fn` {Function} function to be executed, if there is asynchronous operation, you need to return a Promise
* `return` {Promise}

Use the transaction to perform the function passed, the function needs to return a Promise. If the function return the value of Resolved Promise, then the final implementation of the commit, if the return value is Rejected Promise (or error), then the final implementation of rollback.

```js
module.exports = class extends think.Model {
  async updateData(data){
    const result = await this.transaction(async () => {
      const insertId = await this.add(data);
      return insertId;
    })
  }
}
```

Because the operations in the transaction need to be executed in the same connection, if multiple models are involved in the process, multiple models are required to reuse the same database connection. In this case, the database connection can be reused through the `model.db` method.

```js
module.exports = class extends think.Model {
  async updateData(data){
    const result = await this.transaction(async () => {
      const insertId = await this.add(data);
      // through the db method for the user_cate model reuse the current model of the database connection
      const userCate = this.model('user_cate').db(this.db());
      let result = await userCate.add({user_id: insertId, cate_id: 100});
      return result;
    })
  }
}
```

#### model.cache(key, config)

* `key` {String} cache key, if not set will get the SQL statement md5 value as the key
* `config` {Mixed} cache configuration
* `return` {this}

Set the query cache, only valid in `select`, `find`, `getField` and other methods related to the query. And combine the cache adapter and the model cache configuration automatically.

```js
// cache adapter configuration
exports.cache = {
  type: 'file',
  file: {
    handle: fileCache,
    ...
  }
}
// model adapter configuration
exports.model = {
  type: 'mysql',
  mysql: {
    handle: mysqlModel,
    ...
    cache: { // extra cache configuration
      type: 'file',
      handle: fileCache
    }
  }
}
```

The cache adapter configuration, the model cache configuration, and the arguments configuration will eventually be combined as the cache configuration.

```js
module.exports = class extends think.Controller {
  indexAction() {
    // set the cache key to userList, valid for 2 hours
    return this.model('user').cache('userList', {timeout: 2 * 3600 * 1000}).select();
  }
}
```

#### model.lock(lock)

* `lock` {Boolean} whether to lock
* `return` {this}

Add the lock when SELECT, `FOR UPDATE` after the SELECT statement.

```js
module.exports = class extends think.Controller {
  async indexAction() {
    const user = this.model('user');
    const data = await user.lock(true).where({id: 1}).find();
    await user.where({id: data}).update({score: 1});
  }
}
```
