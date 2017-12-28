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

Database tables often associated with other data tables, data operations need to operate together with the associated table. For example: A blog post will have categories, tags, reviews, and which user it belongs to. The types of support are: one to one, one to one (belong to), one to many and many to many.

The detailed relationship can be configured via the [model.relation](/doc/3.0/relation_model.html#toc-548) attribute.

#### One to one

One to one association, indicating that the current table contains a subsidiary table. Assuming that the model name of the current table is `user` and the model name of the associated table is `info`, the default value of `key` in the configuration is `id` and the default value of `fKey` is `user_id`.

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

One to one association, belonging to a associated table, as opposed to HAS_ONE. Assuming that the current model name is `info` and the name of the associated table is `user`, the default value of `key` in the configuration is `user_id`, and the default value of the `fKey` field is `id`.

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

One to many association. If the current model name is `post` and the associated table's model name is `comment`, then the configuration field `key` defaults to `id` and the configuration field `fKey` defaults to `post_id`.

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

If the data in the associated table needs to be paged query, it can be done via the [model.setRelation](/doc/3.0/relation_model.html#toc-d7a) method.

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
  One to one (belong to) the default value is a combination of the name of the associated table and id, such as: cate_id
  ```
* `fKey` the associated table corresponding key

  ```
  One to one, one to many, many to many the default value is a combination of table name and id, such as: cate_id
  One to one (Belonging) the default value for the current model's primary key, such as: id
  ```

* `field` field set when the associated table query, the default value is `*`. If you need to set, must contain the value of `fKey`, support function.

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
* `where` 关联表查询时设置的 where 条件，支持函数
* `order` 关联表查询时设置的 order，支持函数
* `limit` 关联表查询时设置的 limit，支持函数
* `page` 关联表查询时设置的 page，支持函数
* `rModel` 多对多关系下，对应的关联关系模型名，默认值为二个模型名的组合，如：`article_cate`

  ```
  多对多关联模型下，一般需要一个中间的关联表维护关联关系，如：article（文章）和 cate（分类）是多对多的关联关系，那么就需要一个文章-分类的中间关系表（article_cate），rModel 为配置的中间关联表的模型名称
  ```
* `rfKey` 多对多关系下，关系表对应的 key
* `relation` 是否关闭关联表的关联关系

  ```
  // 如果关联表还配置了关联关系，那么查询时还会一并查询
  // 有时候不希望查询关联表的关联数据，那么就可以通过 relation 属性关闭
  get relation() {
    return {
      cate: {
        relation: false // 关闭关联表的所有关联关系，可以避免关联死循环等各种问题
      }
    }
  }
  ```

#### model.setRelation(name, value)

设置关联关系后，查询等操作都会自动查询关联表的数据。如果某些情况下不需要查询关联表的数据，可以通过 `setRelation` 方法临时关闭关联关系查询。

##### 全部关闭

通过 `setRelation(false)` 关闭所有的关联关系查询。

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

##### 部分启用

通过 `setRelation('comment')` 只查询 `comment` 的关联数据，不查询其他的关联关系数据。

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

##### 部分关闭

通过 `setRelation('comment', false)` 关闭 `comment` 的关联关系数据查询。

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

##### 重新全部启用

通过 `setRelation(true)` 重新启用所有的关联关系数据查询。

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

##### 动态更改配置项

虽然通过 relation 属性配置了关联关系，但有时候调用的时候希望动态修改某些值，如：设置分页，这时候也可以通过 setRelation 方法来完成。

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
    // 动态设置 comment 的分页
    return this.setRelation('comment', {page}).select();
  }
}
```

#### model.db(db)

获取或者设置 db 的实例，db 为 Adapter handle（如：think-model-mysql） 的实例。事务操作时由于要复用一个连接需要使用该方法。

```js
module.exports = class extends think.Model {
  async getList() {
    // 让 user 复用当前的 Apdater handle 实例，这样后续可以复用同一个数据库连接
    const user = this.model('user').db(this.db());
  }
}
```

#### model.modelName

实例化模型时传入的模型名

```js
const user = think.model('user');
```

实例化时传入的模型名为 `user`，那么 `model.modelName` 值为 `user`。

#### model.config

实例化模型时传入的配置，模型实例化时会自动传递，不用手工赋值。

```js
{
  host: '127.0.0.1',
  port: 3306,
  ...
}
```

#### model.tablePrefix

获取数据表前缀，从配置里的 `prefix` 字段获取。如果要修改的话，可以通过下面的方式：

```js
module.exports = class extends think.Model {
  get tablePrefix() {
    return 'think_';
  }
}
```

#### model.tableName

获取数据表名，值为 `tablePrefix + modelName`。如果要修改的话，可以通过下面的方式：

```js
module.exports = class extends think.Model {
  get tableName() {
    return 'think_user';
  }
}
```

#### model.pk

获取数据表的主键，默认值为 `id`。如果数据表的主键不是 `id`，需要自己配置，如：

```js
module.exports = class extends think.Model {
  get pk() {
    return 'user_id';
  }
}
```

有时候不想写模型文件，而是在控制器里直接实例化，这时候又想改变主键的名称，那么可以通过设置 `_pk` 属性的方式，如：

```js
module.exports = class extends think.Controller {
  async indexAction() {
    const user = this.model('user');
    user._pk = 'user_id'; // 通过 _pk 属性设置 pk
    const data = await user.select();
  }
}
```

#### model.options

模型操作的一些选项，设置 where、limit、group 等操作时最终都会解析到 options 选项上，格式为：

```js
{
  where: {}, // 存放 where 条件的配置项
  limit: {}, // 存放 limit 的配置项
  group: {},
  ...
}
```

#### model.lastSql

获取最近一次执行的 SQL 语句，默认值为空。

```js
const user = think.model('user');
console.log(user.lastSql); // 打印最近一条的 sql 语句，如果没有则为空
```

#### model.model(name)

* `name` {String} 要实例化的模型名
* `return` {this} 模型实例

实例化别的模型，支持子目录的模型实例化。

```js
module.exports = class extends think.Model {
  async getList() {
    // 如果含有子目录，那么这里带上子目录，如： this.model('front/article')
    const article = this.model('article');
    const data = await article.select();
    ...
  }
}
```

#### model.limit(offset, length)

* `offset` {Number} SQL 语句里的 offset
* `length` {Number} SQL 语句里的 length
* `return` {this}

设置 SQL 语句里的 `limit`，会赋值到 `this.options.limit` 属性上，便于后续解析。

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

* `page` {Number} 设置当前页数
* `pagesize` {Number} 每页条数，默认值为 `this.config.pagesize`
* `return` {this}

设置查询分页，会解析为 [limit](/doc/3.0/relation_model.html#toc-47d) 数据。

```js
module.exports = class extends think.Model() {
  async getList() {
    // SQL: SELECT * FROM `test_d` LIMIT 0,10
    const list1 = await this.page(1).select(); // 查询第一页，每页 10 条
    // SQL: SELECT * FROM `test_d` LIMIT 20,20
    const list2 = await this.page(2, 20).select(); // 查询第二页，每页 20 条
  }
}
```

每页条数可以通过配置项 `pageSize` 更改，如：

```js
// src/config/adapter.js
exports.model = {
  type: 'mysql',
  mysql: {
    database: '',
    ...
    pageSize: 20, // 设置默认每页为 20 条
  }
}
```

#### model.where(where)

* `where` {String | Object} 设置查询条件
* `return` {this}

设置 where 查询条件，会添加 `this.options.where` 属性，方便后续解析。可以通过属性 `_logic` 设置逻辑，默认为 `AND`。可以通过属性 `_complex` 设置复合查询。

`注意：where 条件中的值必须要在 Logic 里做数据校验，否则可能会有 SQL 注入漏洞。`

##### 普通条件

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

##### null 条件

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

##### EXP 条件

ThinkJS 默认会对字段和值进行转义，防止安全漏洞。有时候一些特殊的情况不希望被转义，可以使用 EXP 的方式，如：

```js
module.exports = class extends think.Model {
  where1(){
    //SELECT * FROM `think_user` WHERE ( (`name` ='name') )
    return this.where({name: ['EXP', "=\"name\""]}).select();
  }
}
```

##### LIKE 条件

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
  //like 多个值
  where3(){
    //SELECT * FROM `think_user` WHERE ( (`title` LIKE 'welefen' OR `title` LIKE 'suredy') )
    return this.where({title: ['like', ['welefen', 'suredy']]}).select();
  }
  //多个字段或的关系 like 一个值
  where4(){
    //SELECT * FROM `think_user` WHERE ( (`title` LIKE '%welefen%') OR (`content` LIKE '%welefen%') )
    return this.where({'title|content': ['like', '%welefen%']}).select();
  }
  //多个字段与的关系 Like 一个值
  where5(){
    //SELECT * FROM `think_user` WHERE ( (`title` LIKE '%welefen%') AND (`content` LIKE '%welefen%') )
    return this.where({'title&content': ['like', '%welefen%']}).select();
  }
}
```


##### IN 条件

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

##### BETWEEN 查询

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

##### 多字段查询

```js
module.exports = class extends think.Model {
  where1(){
    //SELECT * FROM `think_user` WHERE ( `id` = 10 ) AND ( `title` = 'www' )
    return this.where({id: 10, title: "www"}).select();
  }
  //修改逻辑为 OR
  where2(){
    //SELECT * FROM `think_user` WHERE ( `id` = 10 ) OR ( `title` = 'www' )
    return this.where({id: 10, title: "www", _logic: 'OR'}).select();
  }
  //修改逻辑为 XOR
  where2(){
    //SELECT * FROM `think_user` WHERE ( `id` = 10 ) XOR ( `title` = 'www' )
    return this.where({id: 10, title: "www", _logic: 'XOR'}).select();
  }
}
```

##### 多条件查询

```js
module.exports = class extends think.Model {
  where1(){
    //SELECT * FROM `think_user` WHERE ( `id` > 10 AND `id` < 20 )
    return this.where({id: {'>': 10, '<': 20}}).select();
  }
  //修改逻辑为 OR
  where2(){
    //SELECT * FROM `think_user` WHERE ( `id` < 10 OR `id` > 20 )
    return this.where({id: {'<': 10, '>': 20, _logic: 'OR'}}).select()
  }
}
```

##### 复合查询

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

* `field` {String} 查询字段，支持 `AS`。
* `return` {this}

设置 SQL 语句中的查询字段，默认为 `*`。设置后会赋值到 `this.options.field` 属性上，便于后续解析。

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

* `field` {String} 查询字段，不支持 `AS`。
* `return` {this}

查询时设置反选字段（即：不查询配置的字段，而是查询其他的字段），会添加 `this.options.field` 和 `this.options.fieldReverse` 属性，便于后续分析。

该功能的实现方式为：查询数据表里的所有字段，然后过滤掉配置的字段。


```js
module.exports = class extends think.Model{
  async getList() {
    // SQL: SELECT `id`, `c_id` FROM `test_d`
    const data1 = await this.fieldReverse('d_name').select();
  }
}
```

#### model.table(table, hasPrefix)

* `table` {String} 表名，支持值为一个 SELECT 语句
* `hasPrefix` {Boolean} `table` 里是否已经含有了表前缀，默认值为 `false`
* `return` {this}

设置当前模型对应的表名，如果 hasPrefix 为 false 且 table 不是 SQL 语句，那么表名会追加 `tablePrefix`，最后的值会设置到 `this.options.table` 属性上。

如果没有设置该属性，那么最后解析 SQL 时通过 `mode.tableName` 属性获取表名。

#### model.union(union, all)

* `union` {String} union 查询字段
* `all` {boolean} 是否使用 UNION ALL
* `return` {this}

设置 SQL 中的 UNION 查询，会添加 `this.options.union` 属性，便于后续分析。

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

* `join` {String | Object | Array} 要组合的查询语句，默认为 `LEFT JOIN`
* `return` {this}

组合查询，支持字符串、数组和对象等多种方式。会添加 `this.options.join` 属性，便于后续分析。

##### 字符串

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` LEFT JOIN think_cate ON think_group.cate_id=think_cate.id
    return this.join('think_cate ON think_group.cate_id=think_cate.id').select();
  }
}
```

##### 数组

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

##### 对象：单个表

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` INNER JOIN `think_cate` AS c ON think_user.`cate_id`=c.`id`
    return this.join({
      table: 'cate',
      join: 'inner', //join 方式，有 left, right, inner 3 种方式
      as: 'c', // 表别名
      on: ['cate_id', 'id'] //ON 条件
    }).select();
  }
}
```

##### 对象：多次 JOIN

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


##### 对象：多个表

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
        join: 'left', // 有 left,right,inner 3 个值
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

##### 对象：ON 条件含有多个字段

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` LEFT JOIN `think_cate` ON think_user.`id`=think_cate.`id` LEFT JOIN `think_group_tag` ON think_user.`id`=think_group_tag.`group_id` LEFT JOIN `think_tag` ON (think_user.`id`=think_tag.`id` AND think_user.`title`=think_tag.`name`)
    return this.join({
      cate: {on: 'id, id'},
      group_tag: {on: ['id', 'group_id']},
      tag: {
        on: { // 多个字段的 ON
          id: 'id',
          title: 'name'
        }
      }
    }).select()
  }
}
```

##### 对象：table 值为 SQL 语句

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

* `order` {String | Array | Object} 排序方式
* `return` {this}

设置 SQL 中的排序方式。会添加 `this.options.order` 属性，便于后续分析。

##### 字符串

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



##### 数组

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` ORDER BY id DESC,name ASC
    return this.order(['id DESC', 'name ASC']).select();
  }
}
```

##### 对象

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

* `aliasName` {String} 表别名
* `return` {this}

设置表别名。会添加 `this.options.alias` 属性，便于后续分析。

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM think_user AS a;
    return this.alias('a').select();
  }
}
```

#### model.having(having)

* `having` {String} having 查询的字符串
* `return` {this}

设置 having 查询。会设置 `this.options.having` 属性，便于后续分析。

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` HAVING view_nums > 1000 AND view_nums < 2000
    return this.having('view_nums > 1000 AND view_nums < 2000').select();
  }
}
```


#### model.group(group)

* `group` {String} 分组查询的字段
* `return` {this}

设定分组查询。会设置 `this.options.group` 属性，便于后续分析。

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT * FROM `think_user` GROUP BY `name`
    return this.group('name').select();
  }
}
```

#### model.distinct(distinct)

* `distinct` {String} 去重的字段
* `return` {this}

去重查询。会设置 `this.options.distinct` 属性，便于后续分析。

```js
module.exports = class extends think.Model {
  getList(){
    //SELECT DISTINCT `name` FROM `think_user`
    return this.distinct('name').select();
  }
}
```


#### model.beforeAdd(data)

* `data` {Object} 要添加的数据

添加前置操作。

#### model.afterAdd(data)

* `data` {Object} 要添加的数据

添加后置操作。

#### model.afterDelete(data)

删除后置操作。

#### model.beforeUpdate(data)

* `data` {Object} 要更新的数据

更新前置操作。

有时候希望提交了某值则更新，没有值为空的话就不更新的功能，那么可以通过这个方法来操作：

```js
module.exports = class extends think.Model {
  beforeUpdate(data) {
    for (const key in data) {
      // 如果值为空则不更新
      if(data[key] === '') {
        delete data[key];
      }
    }
    return data;
  }
}
```

#### model.afterUpdate(data)

* `data` {Object} 要更新的数据

更新后置操作。

#### model.afterFind(data)

* `data` {Object} 查询的单条数据
* `return` {Object | Promise}

`find` 查询后置操作。

#### model.afterSelect(data)

* `data` [Array] 查询的数据数据
* `return` {Array | Promise}

`select` 查询后置操作。



#### model.add(data, options)

* `data` {Object} 要添加的数据，如果数据里某些字段在数据表里不存在会自动被过滤掉
* `options` {Object} 操作选项，会通过 [parseOptions](/doc/3.0/relation_model.html#toc-d91) 方法解析
* `return` {Promise} 返回插入的 ID

添加一条数据，返回值为插入数据的 id。

如果数据表没有主键或者没有设置 `auto increment` 等属性，那么返回值可能为 0。如果插入数据时手动设置主键的值，那么返回值也可能为 0。

```js
module.exports = class extends think.Controller {
  async addAction(){
    let model = this.model('user');
    let insertId = await model.add({name: 'xxx', pwd: 'yyy'});
  }
}
```

有时候需要借助数据库的一些函数来添加数据，如：时间戳使用 mysql 的 `CURRENT_TIMESTAMP` 函数，这时可以借助 `exp` 表达式来完成。

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

* `data` {Object} 要添加的数据
* `where` {Object} where 条件，会通过 [where](/doc/3.0/relation_model.html#toc-d47) 方法设置 where 条件
* `return` {Promise}

当 where 条件未命中到任何数据时才添加数据。

```js
module.exports = class extends think.Controller {
  async addAction(){
    const model = this.model('user');
    //第一个参数为要添加的数据，第二个参数为添加的条件，根据第二个参数的条件查询无相关记录时才会添加
    const result = await model.thenAdd({name: 'xxx', pwd: 'yyy'}, {email: 'xxx'});
    // result returns {id: 1000, type: 'add'} or {id: 1000, type: 'exist'}
  }
}
```

也可以把 where 条件通过 `this.where` 方法直接指定，如：

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

* `dataList` {Array} 要添加的数据列表
* `options` {Object} 操作选项，会通过 [parseOptions](/doc/3.0/relation_model.html#toc-d91) 方法解析
* `return` {Promise} 返回插入的 ID 列表

一次添加多条数据。

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

* `fields` {Array | String} 列名
* `table` {String} 表名
* `options` {Object} 操作选项，会通过 [parseOptions](/doc/3.0/relation_model.html#toc-d91) 方法解析
* `return` {Promise} 返回插入的 ID 列表

添加从 options 解析出来子查询的结果数据。

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

* `options` {Object} 操作选项，会通过 [parseOptions](/doc/3.0/relation_model.html#toc-d91) 方法解析
* `return` {Promise} 返回影响的行数

删除数据。

```js
module.exports = class extends think.Controller {
  async deleteAction(){
    let model = this.model('user');
    let affectedRows = await model.where({id: ['>', 100]}).delete();
  }
}
```


#### model.update(data, options)

* `data` {Object} 要更新的数据
* `options` {Object} 操作选项，会通过 [parseOptions](/doc/3.0/relation_model.html#toc-d91) 方法解析
* `return` {Promise} 返回影响的行数

更新数据。

```js
module.exports = class extends think.Controller {
  async updateAction(){
    let model = this.model('user');
    let affectedRows = await model.where({name: 'thinkjs'}).update({email: 'admin@thinkjs.org'});
  }
}
```

默认情况下更新数据必须添加 where 条件，以防止误操作导致所有数据被错误的更新。如果确认是更新所有数据的需求，可以添加 `1=1` 的 where 条件进行，如：

```js
module.exports = class extends think.Controller {
  async updateAction(){
    let model = this.model('user');
    let affectedRows = await model.where('1=1').update({email: 'admin@thinkjs.org'});
  }
}
```

有时候更新值需要借助数据库的函数或者其他字段，这时候可以借助 `exp` 来完成。

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

* `data` {Object} 要更新的数据
* `where` {Object} where 条件
* `return` {Promise}

当 where 条件未命中到任何数据时添加数据，命中数据则更新该数据。

#### updateMany(dataList, options)

* `dataList` {Array} 要更新的数据列表
* `options` {Object} 操作选项，会通过 [parseOptions](/doc/3.0/relation_model.html#toc-d91) 方法解析
* `return` {Promise} 影响的行数

更新多条数据，dataList 里必须包含主键的值，会自动设置为更新条件。

```js
this.model('user').updateMany([{
  id: 1, // 数据里必须包含主键的值
  name: 'name1'
}, {
  id: 2,
  name: 'name2'
}])
```

#### model.increment(field, step)

* `field` {String} 字段名
* `step` {Number} 增加的值，默认为 1
* `return` {Promise}

字段值增加。

```js
module.exports = class extends think.Model {
  updateViewNums(id){
    return this.where({id: id}).increment('view_nums', 1); //将阅读数加 1
  }
}
```

#### model.decrement(field, step)

* `field` {String} 字段名
* `step` {Number} 增加的值，默认为 1
* `return` {Promise}

字段值减少。

```js
module.exports = class extends think.Model {
  updateViewNums(id){
    return this.where({id: id}).decrement('coins', 10); //将金币减 10
  }
}
```


#### model.find(options)

* `options` {Object} 操作选项，会通过 [parseOptions](/doc/3.0/relation_model.html#toc-d91) 方法解析
* `return` {Promise} 返回单条数据

查询单条数据，返回的数据类型为对象。如果未查询到相关数据，返回值为 `{}`。

```js
module.exports = class extends think.Controller {
  async listAction(){
    let model = this.model('user');
    let data = await model.where({name: 'thinkjs'}).find();
    //data returns {name: 'thinkjs', email: 'admin@thinkjs.org', ...}
    if(think.isEmpty(data)) {
      // 内容为空时的处理
    }
  }
}
```

可以通过 [think.isEmpty](/doc/3.0/think.html#toc-df2) 方法判断返回值是否为空。

#### model.select(options)

* `options` {Object} 操作选项，会通过 [parseOptions](/doc/3.0/relation_model.html#toc-d91) 方法解析
* `return` {Promise} 返回多条数据

查询多条数据，返回的数据类型为数组。如果未查询到相关数据，返回值为 `[]`。

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

可以通过 [think.isEmpty](/doc/3.0/think.html#toc-df2) 方法判断返回值是否为空。

#### model.countSelect(options, pageFlag)

* `options` {Number | Object} 操作选项，会通过 [parseOptions](/doc/3.0/relation_model.html#toc-d91) 方法解析
* `pageFlag` {Boolean} 当页数不合法时处理，true 为修正到第一页，false 为修正到最后一页，默认不修正
* `return` {Promise}

分页查询，一般需要结合 `page` 方法一起使用。如：

```js
module.exports = class extends think.Controller {
  async listAction(){
    let model = this.model('user');
    let data = await model.page(this.get('page')).countSelect();
  }
}
```

返回值数据结构如下：

```js
{
  pagesize: 10, //每页显示的条数
  currentPage: 1, //当前页
  count: 100, //总条数
  totalPages: 10, //总页数
  data: [{ //当前页下的数据列表
    name: "thinkjs",
    email: "admin@thinkjs.org"
  }, ...]
}
```

有时候总条数是放在其他表存储的，不需要再查当前表获取总条数了，这个时候可以通过将第一个参数 `options` 设置为总条数来查询。

```js
module.exports = class extends think.Controller {
  async listAction(){
    const model = this.model('user');
    const total = 256;
    // 指定总条数查询
    const data = await model.page(this.get('page')).countSelect(total);
  }
}
```

#### model.getField(field, num)

* `field` {String} 字段名，多个字段用逗号隔开
* `num` {Boolean | Number} 需要的条数
* `return` {Promise}

获取特定字段的值，可以设置 where、group 等条件。

** 获取单个字段的所有列表 **

```js
module.exports = class extends think.Controller {
  async listAction(){
    const data = await this.model('user').getField('c_id');
    // data = [1, 2, 3, 4, 5]
  }
}
```

** 指定个数获取单个字段的列表 **

```js
module.exports = class extends think.Controller {
  async listAction(){
    const data = await this.model('user').getField('c_id', 3);
    // data = [1, 2, 3]
  }
}
```

** 获取单个字段的一个值 **

```js
module.exports = class extends think.Controller {
  async listAction(){
    const data = await this.model('user').getField('c_id', true);
    // data = 1
  }
}
```

** 获取多个字段的所有列表 **

```js
module.exports = class extends think.Controller {
  async listAction(){
    const data = await this.model('user').getField('c_id,d_name');
    // data = {c_id: [1, 2, 3, 4, 5], d_name: ['a', 'b', 'c', 'd', 'e']}
  }
}

```


** 获取指定个数的多个字段的所有列表 **

```js
module.exports = class extends think.Controller {
  async listAction(){
    const data = await this.model('user').getField('c_id,d_name', 3);
    // data = {c_id: [1, 2, 3], d_name: ['a', 'b', 'c']}
  }
}
```

** 获取多个字段的单一值 **

```js
module.exports = class extends think.Controller {
  async listAction(){
    const data = await this.model('user').getField('c_id,d_name', true);
    // data = {c_id: 1, d_name: 'a'}
  }
}
```

#### model.count(field)

* `field` {String} 字段名，如果不指定那么值为 `*`
* `return` {Promise} 返回总条数

获取总条数。

```js
module.exports = class extends think.Model{
  // 获取字段值之和
  getScoreCount() {
    // SELECT COUNT(score) AS think_count FROM `test_d` LIMIT 1
    return this.count('score');
  }
}
```

#### model.sum(field)

* `field` {String} 字段名
* `return` {Promise}

对字段值进行求和。

```js
module.exports = class extends think.Model{
  // 获取字段值之和
  getScoreSum() {
    // SELECT SUM(score) AS think_sum FROM `test_d` LIMIT 1
    return this.sum('score');
  }
}
```

#### model.min(field)

* `field` {String} 字段名
* `return` {Promise}

求字段的最小值。


```js
module.exports = class extends think.Model{
  // 获取最小值
  getScoreMin() {
    // SELECT MIN(score) AS think_min FROM `test_d` LIMIT 1
    return this.min('score');
  }
}
```

#### model.max(field)

* `field` {String} 字段名
* `return` {Promise}

求字段的最大值。

```js
module.exports = class extends think.Model{
  // 获取最大值
  getScoreMax() {
    // SELECT MAX(score) AS think_max FROM `test_d` LIMIT 1
    return this.max('score');
  }
}
```

#### model.avg(field)

* `field` {String} 字段名
* `return` {Promise}

求字段的平均值。

```js
module.exports = class extends think.Model{
  // 获取平均分
  getScoreAvg() {
    // SELECT AVG(score) AS think_avg FROM `test_d` LIMIT 1
    return this.avg('score');
  }
}
```

#### model.query(sqlOptions)

* `sqlOptions` {String | Object} 要执行的 sql 选项
* `return` {Promise} 查询的数据

指定 SQL 语句执行查询，`sqlOptions` 会通过 [parseSql](/doc/3.0/relation_model.html#toc-ec3) 方法解析，使用该方法执行 SQL 语句时需要自己处理安全问题。

```js
module.exports = class extends think.Model {
  getMysqlVersion() {
    return this.query('select version();');
  }
}
```


#### model.execute(sqlOptions)

* `sqlOptions` {String | Object} 要操作的 sql 选项
* `return` {Promise}

执行 SQL 语句，`sqlOptions` 会通过 [parseSql](/doc/3.0/relation_model.html#toc-ec3) 方法解析，使用该方法执行 SQL 语句时需要自己处理安全问题。

```js
module.exports = class extends think.Model {
  xxx() {
    return this.execute('set @b=5;call proc_adder(2,@b,@s);');
  }
}
```


#### model.parseSql(sqlOptions, ...args)

* `sqlOptions` {String | Object} 要解析的 SQL 语句
* `...args` {Array} 解析的数据
* `return` {Object}

解析 SQL 语句，将 SQL 语句中的 `__TABLENAME__` 解析为对应的表名。通过 [util.format](https://nodejs.org/api/util.html#util_util_format_format_args) 将 args 数据解析到 sql 中。

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

* `options` {Object} 要合并的 options，会合并到 `this.options` 中一起解析
* `return` {Promise}

解析 options。where、limit、group 等操作会将对应的属性设置到 `this.options` 上，该方法会对 `this.options` 进行解析，并追加对应的属性，以便在后续的处理需要这些属性。

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

调用 `this.parseOptions` 解析后，`this.options` 属性会被置为空对象 `{}`。


#### model.startTrans()

* `return` {Promise}

开启事务。

#### model.commit()

* `return` {Promise}

提交事务。

#### model.rollback()

* `return` {Promise}

回滚事务。

```js
module.exports = class extends think.Model {
  async addData() {
    // 如果添加成功则 commit，失败则 rollback
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

如果事务操作过程中需要实例化多个模型操作，那么需要让模型之间复用同一个数据库连接，具体见 [model.db](/doc/3.0/relation_model.html#toc-f95)。

#### model.transaction(fn)

* `fn` {Function} 要执行的函数，如果有异步操作，需要返回 Promise
* `return` {Promise}

使用事务来执行传递的函数，函数要返回 Promise。如果函数返回值为 Resolved Promise，那么最后会执行 commit，如果返回值为 Rejected Promise（或者报错），那么最后会执行 rollback。

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
由于事务里的操作需要在同一个连接里执行，如果处理过程中涉及多个模型的操作，需要多个模型复用同一个数据库连接，这时可以通过 `model.db` 方法达到复用数据库连接的效果。

```js
module.exports = class extends think.Model {
  async updateData(data){
    const result = await this.transaction(async () => {
      const insertId = await this.add(data);
      // 通过 db 方法让 user_cate 模型复用当前模型的数据库连接
      const userCate = this.model('user_cate').db(this.db());
      let result = await userCate.add({user_id: insertId, cate_id: 100});
      return result;
    })
  }
}
```

#### model.cache(key, config)

* `key` {String} 缓存 key，如果不设置会获取 SQL 语句的 md5 值作为 key
* `config` {Mixed} 缓存配置
* `return` {this}

设置查询缓存，只在 `select`、`find`、`getField` 等查询相关的方法下有效。会自动合并 cache Adapter、model cache 的配置。

```js
// cache adapter 配置
exports.cache = {
  type: 'file',
  file: {
    handle: fileCache,
    ...
  }
}
// model adapter 配置
exports.model = {
  type: 'mysql',
  mysql: {
    handle: mysqlModel,
    ...
    cache: { // 额外的缓存配置
      type: 'file',
      handle: fileCache
    }
  }
}
```

最终会将 cache adapter 配置、model cache 配置、以及参数里的配置合并起来作为 cache 的配置。

```js
module.exports = class extends think.Controller {
  indexAction() {
    // 设置缓存 key 为 userList，有效期为 2 个小时
    return this.model('user').cache('userList', {timeout: 2 * 3600 * 1000}).select();
  }
}
```

#### model.lock(lock)

* `lock` {Boolean} 是否 lock
* `return` {this}

SELECT 时加锁，在 SELECT 语句后面加上 `FOR UPDATE`。

```js
module.exports = class extends think.Controller {
  async indexAction() {
    const user = this.model('user');
    const data = await user.lock(true).where({id: 1}).find();
    await user.where({id: data}).update({score: 1});
  }
}
```
