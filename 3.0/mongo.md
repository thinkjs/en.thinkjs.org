## MongoDB

If you want to use MongoDB to stroe data in project, framework provide [think-mongo](https://github.com/thinkjs/think-mongo) extend to support MonogDB, this module is based on [mongodb](https://github.com/mongodb/node-mongodb-native).

### MongoDB Extend features

Modify extend configuration file`src/config/extend.js` (or `src/common/config/extend.js`in multi-module project), add the following configuration: 

```js
const mongo = require('think-mongo');

module.exports = [
  mongo(think.app) // To allow framework support model feature
]
```

After extend is configured, it will inject `think.Mongo`、`think.mongo`、`ctx.mongo` and `controller.mongo` methods, where think.Mongo is the base class file for instantiating Mongo models, others are the Mongo model instantiating methods, ctx.mongo and controller.mongo is think.mongo method's wrapper.

### Config MongoDB database

MongoDB database configuration reuse relational database congiguration, it is a adapter configuration in model, file path is `src/config/adapter.js` (or `src/common/config/adapter.js` in multi-module project).

```js
exports.model = {
  type: 'mongo', // The default use type, can be change by parameter on runtime
  common: { // common setting
    logConnect: true, // whether to print database connection information
    logger: msg => think.logger.info(msg) // logger to print message
  },
  mongo: {
    host: '127.0.0.1',
    port: 27017,
    user: '',
    password: '',
    database: '', // database name
    options: ''
  }
}
```

support multiple host and port, such as:

```js
exports.model = {
  type: 'mongo', // The default type
  common: {
    logConnect: true,
    logger: msg => think.logger.info(msg)
  },
  mongo: {
    host: ['127.0.0.1', '10.16.1.2'],
    port: [27017, 27018],
    user: '',
    password: '',
    database: '', // database name
    options: ''
  }
}
```

More configuration refers to <http://mongodb.github.io/node-mongodb-native/2.0/tutorials/urls/>。

### Create model file

The model files are placed in `src/model/` directory (multi-module projects `src/common/model` and `src/[module]/model`), inheriting the model base class `think.Mongo` as bellow:

```js
// src/model/user.js
module.exports = class extends think.Mongo {
  getList() {
    return this.field('name').select();
  }
}
```
 
If the project is more complex, you want to subdirectory management of the model file, you can create subdirectories in the model directory, such as: `src/model/front/user.js`, `src/model/admin/user.js`, so Create the `front` and` admin` directories under the model directory to manage the foreground and background model files separately.
Instantiation of a model with subdirectories requires subdirectories path, such as: `think.mongo('front/user')`, see [here](/doc/3.0/relation_model.html#toc-9d9).


### Instantiate the model

项目启动时, 会扫描项目下的所有模型文件（目录为 `src/model/`, 多模块项目下目录为 `src/common/model` 以及各种 `src/[module]/model`）, 扫描后会将所有的模型类存放在 `think.app.models` 对象上, 实例化时会从这个对象上查找, 如果找不到则实例化模型基类 `think.Mongo`。

When the project starts, it scans for all model files (`src/model/` under the project directory, `src/common/model` and various `src/[module]/model` under the multi-module project) After all the model classes will be stored in `think.app.models` object, from which to search class for instantiation, if not found, `think.Mongo` will be instantiated.

#### think.mongo

Instantiate model class.

```js
think.mongo('user'); // get model instance
think.mongo('user', 'sqlite'); // get model instance and change database type
think.mongo('user', { // get model instance, set type and add other parameters
  type: 'sqlite',
  aaa: 'bbb'
}); 
think.mongo('user', {}, 'admin'); // get model instance, specific admin module (only for multi-module project)
```
#### ctx.mongo

Instantiate the model class, call the `think.mongo` method after obtaining the configuration, and get the configuration for the current module under multi-module project.
```js
const user = ctx.mongo('user');
```

#### controller.mongo

Instantiate the model class, call the `think.mongo` method after obtaining the configuration, and get the configuration for the current module under multi-module project.

```js
module.exports = class extends think.Controller {
  async indexAction() {
    const user = this.mongo('user'); // instantiate model in controller
    const data = await user.select();
    return this.success(data);
  }
}
```

#### service.mongo

Equivalent to `think.mongo`.

#### Model with subdirectories instantiated

If the model directory contains subdirectories, then you need to bring the corresponding subdirectory when instantiating:

```js
const user1 = think.mongo('front/user'); // instantiate front user model
const user2 = think.mongo('admin/user'); // instantiate backend user model

```

### FAQ

#### How to use mongoose in project?

[think-mongoose](https://github.com/thinkjs/think-mongoose) module is provided, You can use some of the Mongoose operations directly in the project.

### API

#### mongo.pk

Get primary key of the data table, the default value is `_id`. If the data table's primary key is not `_id`, you need to configure yourself, such as:

```js
module.exports = class extends think.Mongo {
  get pk() {
    return 'user_id';
  }
}
```

Sometimes do not want to write model files, but in the controller directly instantiated, then want to change the name of the primary key, you can set `_pk` property, such as:

```js
module.exports = class extends think.Controller {
  async indexAction() {
    const user = this.mongo('user');
    user._pk = 'user_id'; // set pk through _pk property
    const data = await user.select();
  }
}
```


#### mongo.tablePrefix

Obtain data table prefix, obtained from the `prefix` field in the configuration. If you want to modify it, you can through the following ways:

```js
module.exports = class extends think.Mongo {
  get tablePrefix() {
    return 'think_';
  }
}
```

#### mongo.mongo

Get data table name, the value is `tablePrefix + modelName`. If you want to modify it, you can through the following ways:
```js
module.exports = class extends think.Mongo {
  get tableName() {
    return 'think_user';
  }
}
```


#### mongo.model(name)

* `name` {String} model to instanitate
* `return` {this} model instance

To instantiate other model, support subdirectory model instantiation.

```js
module.exports = class extends think.Mongo {
  async getList() {
    // if subdirecotry you need to add the subdirectory path, such as: this.mongo('front/article')
    const article = this.mongo('article'); 
    const data = await article.select();
    ...
  }
}
```

#### mongo.db(db)

Get or set db instance, db for Adapter handle instance.
```js
module.exports = class extends think.Mongo {
  async getList() {
    // To allow user reuse current Apdater handle instance, so that to reuse database connection
    const user = this.mongo('user').db(this.db()); 
  }
}
```

#### mongo.modelName

Model name on model instantiation.

```js
const user = think.mongo('user');
```

If instance is instantiate by model name of `user`, then the value of `model.modelName` is `user`.

#### mongo.config

Incoming configuration of the model instantiation, the model will automatically be instantiated, without manual assignment.

```js
{
  host: '127.0.0.1',
  port: 27017,
  ...
}
```

#### mongo.limit(offset, length)

* `offset` {Number} start index(similar to SQL offset)
* `length` {Number} length (similar to SQL length)
* `return` {this}

Set SQL statement `limit`, will be assigned to `this.options.limit` property, for subsequent parsing.

```js
module.exports = class extends think.Mongo() {
  async getList() {
    // Top 10
    const list1 = await this.limit(10).select();
    // 11 ~ 20
    const list2 = await this.limit(10, 20).select();
  }
}
```


#### mongo.page(page, pagesize)

* `page` {Number} Set current page
* `pagesize` {Number} size of page, default value is `this.config.pagesize`
* `return` {this}

Setting the query page will resolve to [limit](/doc/3.0/mongo.html#toc-769) data.

```js
module.exports = class extends think.Mongo() {
  async getList() {
    const list1 = await this.page(1).select(); // query first page, 10 records a page.
    const list2 = await this.page(2, 20).select(); // query second page, 20 records a page
  }
}
```

Record number per page can be set through `pageSize`, such as:

```js
// src/config/adapter.js
exports.model = {
  type: 'mongo',
  mongo: {
    database: '',
    ...
    pageSize: 20, // set default page size to 20
  }
}
```

#### model.where(where)

* `where` {String} set query conditions
* `return` {this}

Set query fields, which is stored as `this.options.where` property, for later parsing.

```js
module.exports = class extends think.Mongo{
  async getList() {
    const data = await this.where(where).select();
  }
}
```

#### model.field(field)

* `field` {String} query field
* `return` {this}

Set query field, which is stored as `this.options.field` propery, for later parsing.

```js
module.exports = class extends think.Mongo{
  async getList() {
    const data1 = await this.field('d_name').select();

    const data2 = await this.field('c_id,d_name').select();
  }
}
```


#### model.table(table, hasPrefix)

* `table` {String} table name, support SELECT query
* `hasPrefix` {Boolean} whether `table` has prefix, default value is `false`.
* `return` {this}

Set the table name for current model, if hasPrefix is false, the table name will be appended `tablePrefix`, the last value will be set to `this.options.table` property.
If this property is not set, the name of the table will be obtained by `model.tableName` property at the time of final parsing.


#### model.parseOptions(options)

* `options` {Object} options to be merged to `this.options` for parsing
* `return` {Promise}

Parse options. The where, limit, group, etc. actions sets the corresponding vlaue the `this.options`, which parse `this.options` and appends corresponding properties for subsequent processing.

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

After parsing through `this.parseOptions`, the value of `this.options` will be set to emtpy object `{}`.


#### model.order(order)

* `order` {String | Array | Object} sorting method
* `return` {this}

Set the sorting method, will add `this.options.order` property.

#### model.group(group)

* `group` {String} group by field
* `return` {this}

Set group inquiry. Will add `this.options.group` property for later analysis.

#### model.distinct(distinct)

* `distinct` {String} 
* `return` {this}

Distinct query. Will add `this.options.distinct` property for later analysis.


#### model.add(data, options)

* `data` {Object} Data to be added, if some fields of data do not exist in data table will automatically be filtered out
* `options` {Object} will be parse by [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} ID of added data

Add one record, return id of inserted data.

The return value may be 0 if data table has no primary key or `auto nincrement` property not being set. If manually set primary key on data insersion, the return value may also be 0.

```js
module.exports = class extends think.Controller {
  async addAction(){
    let model = this.mongo('user');
    let insertId = await model.add({name: 'xxx', pwd: 'yyy'});
  }
}
```


#### model.thenAdd(data, where)

* `data` {Object} data to be added
* `where` {Object} where condition, which will be set by [where](/doc/3.0/relation_model.html#toc-d47) method.
* `return` {Promise}

Only add data when where condition did not hit any data.

```js
module.exports = class extends think.Controller {
  async addAction(){
    const model = this.mongo('user');
    //first parameter is the data to be added, the second parameter is where condition, which mean to add data when there is record with email of 'xxx'
    const result = await model.thenAdd({name: 'xxx', pwd: 'yyy'}, {email: 'xxx'});
    // result returns {id: 1000, type: 'add'} or {id: 1000, type: 'exist'}
  }
}
```

It can also set where condition through `this.where` method, such as:

```js
module.exports = class extends think.Controller {
  async addAction(){
    const model = this.mongo('user');
    const result = await model.where({email: 'xxx'}).thenAdd({name: 'xxx', pwd: 'yyy'});
    // result returns {id: 1000, type: 'add'} or {id: 1000, type: 'exist'}
  }
}
```

#### model.addMany(dataList, options)

* `dataList` {Array} data list to be added
* `options` {Object} options which will be parsed by [parseOptions](/doc/3.0/relation_model.html#toc-d91) method.
* `return` {Promise} return ID list of inserted data

Add multiple records at once.

```js
module.exports = class extends think.Controller {
  async addAction(){
    let model = this.mongo('user');
    let insertIds = await model.addMany([
      {name: 'xxx', pwd: 'yyy'},
      {name: 'xxx1', pwd: 'yyy1'}
    ]);
  }
}
```


#### model.delete(options)

* `options` {Object} options which will be parsed by [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} return the number of rows affected

Delete data.

```js
module.exports = class extends think.Controller {
  async deleteAction(){
    let model = this.mongo('user');
    let affectedRows = await model.where({id: ['>', 100]}).delete();
  }
}
```


#### model.update(data, options)

* `data` {Object} data to update
* `options` {Object} option to be parsed by [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} return the number of rows affected

Update data.

```js
module.exports = class extends think.Controller {
  async updateAction(){
    let model = this.mongo('user');
    let affectedRows = await model.where({name: 'thinkjs'}).update({email: 'admin@thinkjs.org'});
  }
}
```

By default updatte must be added where condition to prevent misuse caused all the data is incorrectly updated. If you are sure to update all data, add `1=1` where condition will do the trick. Such as:

```js
module.exports = class extends think.Controller {
  async updateAction(){
    let model = this.mongo('user');
    let affectedRows = await model.where('1=1').update({email: 'admin@thinkjs.org'});
  }
}
```

Sometime we need to update value according to other fields or database functions, this can be done with the help of `exp`.

```js
module.exports = class extends think.Controller {
  async updateAction(){
    let model = this.mongo('user');
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

Only update data when where condition did not hit any data.

#### model.updateMany(dataList, options)

* `dataList` {Array} data list to  update
* `options` {Object} option to be parsed by [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} return the number of rows affected

Update multiple data, the datalist must contain the primary key, it will be automatically parsed as update condition.

```js
this.mongo('user').updateMany([{
  id: 1, // data must contain primary key
  name: 'name1'
}, {
  id: 2,
  name: 'name2'
}])
```

#### model.increment(field, step)

* `field` {String} field name
* `step` {Number} value to increase, default is 1
* `return` {Promise}

Increase field value.

```js
module.exports = class extends think.Mongo {
  updateViewNums(id){
    return this.where({id: id}).increment('view_nums', 1); //add 1
  }
}
```

#### model.decrement(field, step)

* `field` {String} filed name
* `step` {Number} value to decrease, default is 1
* `return` {Promise}

Decrease field value.

```js
module.exports = class extends think.Mongo {
  updateViewNums(id){
    return this.where({id: id}).decrement('coins', 10); //To descrease gold by 10
  }
}
```


#### model.find(options)

* `options` {Object} options will be parse by [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} return single data

To query single data, the return value type is object. If no data is found, the return value is `{}`.

```js
module.exports = class extends think.Controller {
  async listAction(){
    let model = this.mongo('user');
    let data = await model.where({name: 'thinkjs'}).find();
    //data returns {name: 'thinkjs', email: 'admin@thinkjs.org', ...}
    if(think.isEmpty(data)) {
      // handle whne data is empty
    }
  }
}
```

You can use [think.isEmpty](/doc/3.0/think.html#toc-df2) method to judge whether value is empty.

#### model.select(options)

* `options` {Object} options will be parse by [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise} return multiple data

Query multiple data, return array data type. If no data is found, the return value is `[]`.

```js
module.exports = class extends think.Controller {
  async listAction(){
    let model = this.mongo('user');
    let data = await model.limit(2).select();
    //data returns [{name: 'thinkjs', email: 'admin@thinkjs.org'}, ...]
    if(think.isEmpty(data)){

    }
  }
}
```

You can use [think.isEmpty](/doc/3.0/think.html#toc-df2) method to judge whether value is empty.

#### model.countSelect(options, pageFlag)

* `options` {Number | Object} options will be parse by [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `pageFlag` {Boolean} when the number of pages is not legal, true is amended to the first page, false is amended to the last page, the default does not correct
* `return` {Promise}

Paging queries, in general, need to be combined with the `page` method. Such as:

```js
module.exports = class extends think.Controller {
  async listAction(){
    let model = this.mongo('user');
    let data = await model.page(this.get('page')).countSelect();
  }
}
```


The return data structure is as follows:

```js
{
  pagesize: 10, 
  currentPage: 1,
  count: 100, 
  totalPages: 10, 
  data: [{ 
    name: "thinkjs",
    email: "admin@thinkjs.org"
  }, ...]
}
```

Sometimes the total number is stored in other tables, do not need to check the current table to get the total number, this time can be the first parameter `options` set to the total number of queries.

```js
module.exports = class extends think.Controller {
  async listAction(){
    const model = this.mongo('user');
    const total = 256;
    // set total number to 256
    const data = await model.page(this.get('page')).countSelect(total);
  }
}
```
#### model.sum(field)

* `field` {String} field name
* `return` {Number|Array} return the summation result

If there is no grouping, the default number will be returned. If there is any group, the group information and the summation result will be returned, as shown in the following example:

```js
module.exports = class extends think.Controller {
  async listAction(){
    let model = this.mongo('user');
    // ret1 = 123  no group, return number
    let ret1 = await m.sum('age');		

    // ret2 = [{group:'thinkjs1',total:6},{group:'thinkjs2',total:8}]
    // with group return [{group:xxx,total:xxx}...]
    let ret2 = await m.group('name').sum('age'); 

	  // ret3 = [{group:{name:'thinkjs',version'1.0'},total:6},{group:{name:'thinkjs',version'2.0'},total:8},]
    let ret3 = await m.where({name:'thinkjs'}).order('version ASC').group('name,version').sum('age'); 
  }
}
```

#### model.aggregate(options)
* `options` {Object} options will be parse by [parseOptions](/doc/3.0/relation_model.html#toc-d91) method
* `return` {Promise}

Aggregation operation, see [Aggregation](https://docs.mongodb.com/manual/reference/sql-aggregation-comparison/)

#### model.mapReduce(map,reduce,out)
* `map` {	function | string} mapping method
* `reduce` {	function | string} reduce method
* `out` {Object} other options
* `return` {Promise}

Collestion Map-Reduce operations, see [MapReduce](http://mongodb.github.io/node-mongodb-native/2.2/api/Collection.html#mapReduce)

#### model.createIndex(indexes,options)
* `indexes` {	string | object} index name
* `options` {Object} 
* `return` {Promise}

Create index, see [ensureIndex](http://mongodb.github.io/node-mongodb-native/2.2/api/Db.html#ensureIndex)

#### model.getIndexes()
* `return` {Promise}

Get Index.