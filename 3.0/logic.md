## Logic

When handling the user's request in Action, it is often necessary to obtain the user submit data and verify them. If there is no problem in the verification, the subsequent operations can be performed. Sometime it also check authorization, etc. If you put these codes in an Action, it is bound to make Action's code very complex and lengthy.

In order to solve this problem, ThinkJS adds a layer of `Logic` in front of the controller. The Action in Logic corresponding with the Action in the controller. System will automatically invoke the Action in Logic before calling controller Action.

### Logic

The Logic directory is located at `src/[module]/logic`. Command `thinkjs controller test` will create test controller and test logic.

Logic code similar to the following:

```js
module.exports = class extends think.Logic {
 __before() {
    // todo
 }
 indexAction() {
    // todo
 }
 __after() {
    // todo
 }
}
```

Note: Logic files and Controller file names must be the same if you create them manually

Among them, Action in Logic and action in Controller is one-to-one correspondence. Logic also supports `__before` and` __after` magic methods.

#### Request type verification

Sometimes a specific Action need to be limited to certain types of requests, and to reject other request types. You can filter the request by configuring a specific request type.

```js
module.exports = class extends think.Logic {
 indexAction() {
    this.allowMethods = 'post'; //  only POST 
 }
 detailAction() {
    this.allowMethods = 'get,post'; // allow GET or POST
 }
}
```

#### Verification rules format

Data validation configuration format is `field`: `JSON config object`, as follows:

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      username: {
        string: true,       // data type is String
        required: true,     // username field is required
        default: 'thinkjs', // default value is 'thinkjs'
        trim: true,         // data will be trim
        method: 'GET'       // request method
      },
      age: {
        int: {min: 20, max: 60} // integer between 20 to 60
      }
    }
    let flag = this.validate(rules);
  }
}
```

#### Basic data type

Supported data types including: `boolean`, `string`, `int`, `float`, `array` and `object`, one data field only allows one basic data type, default is `string`.

#### Manual set data value

Sometime it can't automatically get the value (such as: value from the header), then you can manually get the value and then set the configuration. Such as:
```js
module.exports = class extends think.Logic {
  saveAction(){
    let rules = {
      username: {
        value: this.header('x-name') // get value from header
      }
    }
  }
}
```

#### Specify data source

If you validate `version` parameter, its value is obtained from `version` field by specific request type. If the current request type is` GET`, it will get `version` Field; if the request type is` POST`, the value of the field is retrieved via `this.post('version')`, and if the current request type is `FILE`, then `this.file('version')` to get the value.

Sometimes in the `POST` type, you may get the uploaded file or get the parameters on the URL, this time you need to specify the way to get the data. Supported data acquisition methods are `GET`, `POST`, and `FILE`.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      username: {
        required: true,
        method: 'GET'       // data acquisition methods
      }
    }
    let flag = this.validate(rules);
  }
}
```

#### default field value

Use `default:value` to set a field's default value, if field value is empty, default value will be set and apply the following validation logic.

#### trim

Use `trim:true` if current field suports `trim` operation.

#### Data validation method

After validation rule is configured, you can call `this.validate` to validate. Such as:

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      username: {
        required: true
      }
    }
    let flag = this.validate(rules);
    if(!flag){
      return this.fail('validate error', this.validateErrors);
      // return is validation is failed
      // {"errno":1000,"errmsg":"validate error","data":{"username":"username can not be blank"}}
    }
  }
}
```
If you use `this.isGet` or` this.isPost` in the controller's action to determine the request, you would also need to include the `this.isGet` or` this.isPost` in the code above, for example:

```js
module.exports = class extends think.Logic {
  indexAction(){
    if(this.isPost) {
      let rules = {
        username: {
          required: true
        }
      }
      let flag = this.validate(rules);
      if(!flag){
        return this.fail('validate error', this.validateErrors);
      }
    }

  }
}
```

If the return value is `false`, then you can get a detailed error message by `this.validateErrors`. And you can output the error message in `JSON` format via the` this.fail` method, or you can output a page via `this.display`. Logic inherits all Controller methods.

#### auto-validation

In most cases, we just output JSON error message after validation failed. If you don't want call `this.validate` manually each time, you can verify it automatically by setting the validation rule into `this.rules` attribute. For example:

```js
module.exports = class extends think.Logic {
  indexAction(){
    this.rules = {
      username: {
        required: true
      }
    }
  }
}
```

Equivalent to

``` js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      username: {
        required: true
      }
    }
    let flag = this.validate(rules);
    if(!flag){
      return this.fail(this.config('validateDefaultErrno') , this.validateErrors);
    }
  }
}
```

Assigning the validation rule to the `this.rules` automatically validates the execution of the Action, and outputs an error message in JSON format if there is an error.


#### multiple actions reuse check rules

For multiple actions Sometimes we want to re-use some of the validation rules, for example `indexAction` and `homeAction` in `logic` have to verify the `app_id` field is required, you can move `app_id` validation to `scope`.

```js
module.exports = class extends think.Logic {
  get scope() {
    return {
      app_id: {
        required: true
      }
    }
  }

  indexAction(){
    let rules = {
      email: {
        required: true
      }
    }

    // custom app_id error message
    let msgs = {
      app_id: '{name} can not be null',
    }

    if(!this.validate(rules, msgs)) {
      return this.fail(this.validateErrors);
    }
  }

  homeAction() {
    // email shorthand
    // app_id use default error message
    this.rules = {
      email: {
        required: true
      }
    }
  }

}
```

#### Array validation

Data validation supports Array type, but it only supports simple arrays, and does not support nested arrays. `children` specifies an identical check rule for all array elements.


``` js
module.exports = class extends think.Logic {
  let rules = {
    username: {
      array: true,
      children: {
        string: true,
        trim: true,
        default: 'thinkjs'
      },
      method: 'GET'
    }
  }
  this.validate(rules);
}

```

#### Object validation

Data validation supports Object type, note nested arrays are not supported. `children` specifies an identical check rule for all object attributes.


``` js
module.exports = class extends think.Logic {
  let rules = {
    username: {
      object: true,
      children: {
        string: true,
        trim: true,
        default: 'thinkjs'
      },
      method: 'GET'
    }
  }
  this.validate(rules);
}
```

#### convert data before validation

For `boolean` type field, value in one of `'yes'`,`'on'`,`'1'`,`'true'`,`true` will be converted to `true`, and in other cases `false`, and then performs the follow-up rule validation;

For `array` type field, if the field itself is an array, it will not be processed. If the field is a string, it will be `split (',')`, otherwise it will be directly converted into `[field value]` , And then perform the follow-up rule validation.

#### convert data after validation

For fields specified as `int` or `float` data type is automatically `parseFloat` converted after the data is verified.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      age: {
        int: true,
        method: 'GET'
      }
    }
    let flag = this.validate(rules);
  }
}
```

If there is an parameter of `age=26` in url, typeof this.param('age') is of type number after logic level validation.

#### custom error rules

```js
module.exports = class extends think.Logic {
  indexAction(){
    this.rules = {
      username: {
        required: true
      }
    }
  }
}
```

For the above rules this.validateErrors would be {username: 'username can not be blank'} in the case of validation failure. But sometimes you want to customize the error to 'username can not be null'. Need to do the following:

First in `src/config/validator.js` overwrite the default `required` error message:

```js
module.exports = {
  messages: {
    required: '{name} can not be null',
  }
}

```

```js
module.exports = class extends think.Logic {
  indexAction(){
    this.rules = {
      username: {
        required: true,
        aliasName: 'username'
      }
    }
  }
}
```


#### global validation rule

Create a `validator.js` file under the `config` directory in single-module project and `validator.js` under `common/config` directory in multi-module project. Add a custom verification method in `validator.js`:

For example, we want to verify that the `name1` parameter in `GET` request is equal to the string `lucy`. You can add a validation rule as follows; `[server address]/index/?Name1=jack`


```js
// logic index.js
module.exports = class extends think.Logic {
  indexAction() {
    let rules = {
      name1: {
        eqLucy: 'lucy',
        method: 'GET'
      }
    }
    let flag = this.validate(rules);
    if(!flag) {
      console.log(this.validateErrors); // name1 shoud eq lucy
    }
  }
  }
}

// src/config/validator.js
module.exports = {
  rules: {
    eqLucy(value, { argName, validName, validValue, ctx, currentQuery, rule, rules, parsedValidValue }) {
      return value === validValue;
    }
  },
  messages: {
    eqLucy: '{name} should eq {args}'
  }
}

```
Custom validation method parameters is as follow:
```js
(
  value: ,                // The value of the parameter in the corresponding request, here is ctx['param']['name1']
  {
    argName,              // parameter name,here is name1
    validName,            // validation method name, here is 'eqLucy'
    validValue,           // validate value, here is 'lucy'
    parsedValidValue,     // return value of _eqLucy method, if _eqLucy is not provied, it is validValue
    currentQuery,         // curreny request query, equal to ctx['param']
    ctx,                  // ctx object
    rule,                 // validation rule, here is  {eqLucy: 'lucy', method: 'GET'}
    rules,                // all validation rules, here is the value of let rules
  }
)
```


#### parse validation rule parameter

Sometimes we want to parse the parameters of the validation rules, only need to create new parse method of the same name with an underscore at the beginning, and the results of the parse can be returned.

For example, if we want to verify that the `name1` parameter in `GET` request is equal to `name2` parameter, we can add the following verification method: Visit `[server address]/index/?name1=tom&name2=lily`

```js
// logic index.js
module.exports = class extends think.Logic {
  indexAction() {
    let rules = {
      name1: {
        eqLucy: 'name2',
        method: 'GET'
      }
    }
    let flag = this.validate(rules);
    if(!flag) {
      console.log(validateErrors); // name1 shoud eq name2
    }
  }
}

// src/config/validator.js
module.exports = {
  rules: {
    _eqLucy(validValue, { argName, validName, currentQuery, ctx, rule, rules }){
      let parsedValue = currentQuery[validValue];
      return parsedValue;
    },

    eqLucy(value, { argName, validName, validValue, parsedValidValue, currentQuery, ctx, rule, rules }) {
      return value === parsedValidValue;
    }
  },
  messages: {
    eqLucy: '{name} should eq {args}'
  }
}
```

The first parameter in `_eqLucy` is the value of the current validation rule (for this example, the validValue is 'name2'), and the other parameters have the same meanings as above.

#### custom error message

There are three interpolation variables `{name}`, `{args}`, `{pargs}` in the error message. `{name}` will be replaced by the name of the field being checked, `{args}` will be replaced by the value of the checkout rule and `{pargs}` will be replaced by the value returned by the parsing method. If `{args}`, `{pargs}` is not a string, `JSON.stringify` will be processed.


For `Object: false` fields, three custom error formats are supported: Rule1: `Rule: Error Message`; Rule2: `Field Name: Error Message`; Rule 3: `Field Name: {Rule: Error Message}`.

For the case of multiple error messages specified at the same time, priority rule 3 > rule 2 > rule 1.

``` js
module.exports = class extends think.Logic {
  let rules = {
    username: {
      required: true,
      method: 'GET'
    }
  }
  let msgs = {
    required: '{name} can not be blank',         // rule 1
    username: '{name} can not be blank',         // rule 2
    username: {
      required: '{name} can not be blank'        // rule 3
    }
  }
  this.validate(rules, msgs);
}
```

For `Object: true` fields,it supports the following custom error message. priority si rule 5 > (4 = 3) > 2 > 1 .

``` js
module.exports = class extends think.Logic {
  let rules = {
    address: {
      object: true,
      children: {
        int: true
      }
    }
  }
  let msgs = {
    int: 'this is int error message for all field',             // rule 1
    address: {
      int: 'this is int error message for address',             // rule 2
      a: 'this is int error message for a of address',          // rule 3
      'b,c': 'this is int error message for b and c of address' // rule 4
      d: {
        int: 'this is int error message for d of address'       // rule 5
      }
    }
  }
  let flag = this.validate(rules, msgs);
}
```

### supported validation types

#### required

`required: true` field is required,default is `required: false`. `undefined`、`[empty string]` 、`null`、`NaN` will fail `required: true`.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      name: {
        required: true
      }
    }
    this.validate(rules);
    // todo
  }
}
```

`name` is required.

#### requiredIf

This is required when the value of one item is one of some values. Such as:

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      name: {
        requiredIf: ['username', 'lucy', 'tom'],
        method: 'GET'
      }
    }
    this.validate(rules);
    // todo
  }
}
```

For the above example, the value of `name` is required when `username` in `GET` request is `lucy` or `tom`.

#### requiredNotIf

This is required when the value of one item is not one of some values. Such as:

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      name: {
        requiredNotIf: ['username', 'lucy', 'tom'],
        method: 'POST'
      }
    }
    this.validate(rules);
    // todo
  }
}
```

For the above example, the value of `name` is required when `username` in `GET` request is not `lucy` or `tom`.

#### requiredWith

This item is required when there is a value in several other items.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      name: {
        requiredWith: ['id', 'email'],
        method: 'GET'
      }
    }
    this.validate(rules);
    // todo
  }
}
```

For the above example, the value of `name` is required when `id` or `email` in `GET` request has value.

#### requiredWithAll

This item is required when several other values exist.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      name: {
        requiredWithAll: ['id', 'email'],
        method: 'GET'
      }
    }
    this.validate(rules);
    // todo
  }
}
```

For the above example, the value of `name` is required when `id` and `email` in `GET` request has value.

#### requiredWithOut

This item is required when there is a value that does not exist in several other items.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      name: {
        requiredWithOut: ['id', 'email'],
        method: 'GET'
      }
    }
    this.validate(rules);
    // todo
  }
}
```

For the above example, the value of `name` is required when any of `id`, `email` does not exist in` GET` request.

#### requiredWithOutAll

This value is required when several other values do not exist.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      name: {
        requiredWithOutAll: ['id', 'email'],
        method: 'GET'
      }
    }
    this.validate(rules);
    // todo
  }
}
```
For the above example, the value of `name` is required when all the `id` and `email` values in `GET` request do not exist.

#### contains

The value needs to contain a specific value.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      name: {
        contains: 'ID-',
        method: 'GET'
      }
    }
    this.validate(rules);
    // todo
  }
}
```
For the above example, `name` value in `GET` request needs to container `ID-` string.

#### equals

equals to another filed.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      name: {
        equals: 'username',
        method: 'GET'
      }
    }
    this.validate(rules);
    // todo
  }
}
```
For the above example, `name` value must be equal to `username` value in `GET` request.

#### different

not equal to another filed value.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      name: {
        different: 'username',
        method: 'GET'
      }
    }
    this.validate(rules);
    // todo
  }
}
```

For the above example, the values of `name` and `username` fields in `GET` request are not equal.

#### before

The value needs to be before a date, by default it needs to be before the current date.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      time: {
        before: '2099-12-12 12:00:00', // before: true means before the current date.
        method: 'GET'
      }
    }
    this.validate(rules);
    // todo
  }
}
```
For the above example, `time` field in `GET` request needs to be before `2099-12-12 12:00:00`.

#### after

The value needs to be after a date, the default is after the current date, `after: true | time string`.

#### alpha

The value can only be [a-zA-Z], `alpha: true`.

#### alphaDash

The value can only be [a-zA-Z_], `alphaDash: true`.

#### alphaNumeric

The value can only be [a-zA-Z0-9], `alphaNumeric: true`.

#### alphaNumericDash

The value can only be [a-zA-Z0-9_], `alphaNumericDash: true`.

#### ascii

The value can only be ascii character, `ascii: true`.

#### base64

The encoding must be base64, `base64: true`.

#### byteLength

The byte length must be in a range, `byteLength: options`.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      field_name: {
        byteLength: {min: 2, max: 4} // byte length is between 2 - 4
        // byteLength: {min: 2} // minimum byte length is 2
        // byteLength: {max: 4} // maximum byte length is 4
        // byteLength: 10 // byte length need to be equal to 10
      }
    }
  }
}
```

#### creditCard

The value is a credit card numbers, `creditCard: true`.

#### currency

The value needs to be currency, `currency: true | options`, `options` refer to `https://github.com/chriso/validator.js`.

#### date

The value needs to be date, `date: true`.

#### decimal

Need to be decimal, for example: 0.1, .3, 1.1, 1.00003, 4.0, `decimal: true`.

#### divisibleBy

Need to be divisible by a number, `divisibleBy: number`.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      field_name: {
        divisibleBy: 2 //can be divided by 2
      }
    }
  }
}
```

#### email

Need to be email format, `email: true | options`, `options` refer to `https://github.com/chriso/validator.js`.

#### fqdn

Need to be a valid domain, `fqdn: true | options`, `options` refer to `https://github.com/chriso/validator.js`.

#### float

Need to be a float number, `float: true | options`, `options` refer to `https://github.com/chriso/validator.js`.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      money: {
        float: true, // float number 
        // float: {min: 1.0, max: 9.55} // Need to be a float,,with a minimum of 1.0 and maximum of 9.55
      }
    }
    this.validate();
    // todo
  }
}
```

#### fullWidth

Need to include wide byte characters, `fullWidth: true`.

#### halfWidth

Need to include nibble characters, `halfWidth: true`.

#### hexColor

Need to be a hexadecimal color value, `hexColor: true`.

#### hex

Need to be hexadecimal, `hex: true`.

#### ip

Need to be ip format, `ip: true`.

#### ip4

Need to be ip4 format,`ip4: true`.

#### ip6

Need to be ip6 format, `ip6: true`.

#### isbn

Need for ISBN, `isbn: true`.

#### isin

Need to identify the code for the securities, `isin: true`.

#### iso8601

Need to iso8601 date format, `iso8601: true`.

#### issn

International Standard Serial Number, `issn: true`.

#### uuid

Need to UUID (3,4,5 version), `uuid: true`.

#### dataURI

Need to be dataURI format, `dataURI: true`.

#### md5

Need to be md5,`md5: true`.

#### macAddress

Need to be mac address, `macAddress: true`.

#### variableWidth

Need to include both nibble and full-byte characters, `variableWidth: true`.

#### in

in some values, `in: [...]`.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      version: {
        in: ['2.0', '3.0'] //need to be one of 2.0, 3.0
      }
    }
    this.validate();
    // todo
  }
}
```

#### notIn

Can not be in some value, `notIn: [...]`.

#### int

Need to be int, `int: true | options`, `options` refer to `https://github.com/chriso/validator.js`.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      field_name: {
        int: true, 
        //int: {min: 10, max: 100} //need to be between 10 - 100
      }
    }
    this.validate();
    // todo
  }
}
```

#### length

The length needs to be within a certain range, `length: options`.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      field_name: {
        length: {min: 10}, //length can not be less than 10
        // length: {max: 20}, //length can not be greater than 10
        // length: {min: 10, max: 20}, //length needs to be between 10-20
        // length: 10 //length needs to be equal to 10
      }
    }
    this.validate();
    // todo
  }
}
```

#### lowercase

Need to be lowercase letters, `lowercase: true`.

#### uppercase

Need to be capital letters, `uppercase: true`.

#### mobile

Need to phone number, `mobile: true | options`,`options` refer to `https://github.com/chriso/validator.js`.

```js
module.exports = class extends think.Logic {
  indexAction(){
    let rules = {
      mobile: {
        mobile: 'zh-CN' //Must be China's cell phone number
      }
    }
    this.validate();
    // todo
  }
}
```

#### mongoId

Need to be ObjectID for MongoDB, `mongoId: true`.

#### multibyte

Need to include multi-byte characters, `multibyte: true`.

#### url

Need to be url, `url: true|options`,`options` refer to `https://github.com/chriso/validator.js`.

#### order

Need to be database query order, such as: name DESC, `order: true`.

#### field

Need to be database query field, such as: name,title, `field: true`.

#### image

```js
let rules = {
  file: {
    required: true, // required defaults to false
    image: true,
    method: 'file' // File submitted through the post, verify the file needs to specify the method as `file`
  }
}
```
Upload files need to be pictures, `image: true`.

#### startWith

Need to start with some characters, `startWith: string`.

#### endWith

Need to end with some characters, `endWith: string`.

#### string

Need to be string, `string: true`.

#### array

Array is required, `array: true`, if the value of the field is an array, it will not be processed; `split (,)` will be executed if the value of the field is a string; In other cases is converted to `[field value]`.

#### boolean

Needs to be boolean type. `'yes'`, `'on'`, `'1'`, `'true'`, `true` is converted to `true`.

#### object

Needs to be object type, `object: true`.

#### regexp

The field should match the given regular expression.

```js
module.exports = class extends think.Logic {
  indexAction(){
    this.rules = {
      name: {
        regexp: /thinkjs/g
      }
    }
    this.validate();
    // todo
  }
}
```
