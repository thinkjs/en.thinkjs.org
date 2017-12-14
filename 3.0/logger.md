## Logger

ThinkJS implements powerful logging capabilities via the [think-logger3](https://npmjs.com/think-logger3) module and provides adapter extensions that easily extend the built-in logging module. By default, the [log4js](https://github.com/nomiddlename/log4js-node) module is used as the underlying logging module. It has many features such as log classification, log splitting and multi-process.

### How To Use

System has been globally injected the logger object `think.logger`, it provides `debug`, `info`, `warn` and `error` four methods to output log, the default behaviour is to output to console.

```javascript
think.logger.debug('debug message');
think.logger.info('info message');
think.logger.warn('warning');
think.logger.error(new Error('error'));
```

### Confguration

The system coms with `Console`, `File`, `DateFile` three adapter. The default is to use `Controle` to output logs to console.

#### File

If you want to output log to file, add the following configurations into `src/config/adapter.js` file:

```javascript
const path = require('path');
const {File} = require('think-logger3');

exports.logger = {
  type: 'file',
  file: {
    handle: File,
    backups: 10,
    absolute: true,
    maxLogSize: 50 * 1024,  //50M
    filename: path.join(think.ROOT_PATH, 'logs/xx.log')
  }
}

```

Then initial log would create a file called `debug.log`. After this file reached `maxLogSize`, a new file named `debug.log.1` will be created. After log file number reached `backups`, old log chunk file will be removed.

- `filename`: log filename
- `maxLogSize`: The maximum size (in bytes) for a log file, if not provided then logs won't be rotated.
- `backups`: The number of log files to keep after logSize has been reached (default 5)
- `absolute`: If `filename` is a absolute path, the `absolute` value should be `true`.
- `layout`: Layout defines the way how a log record is rendered. More layouts can see [here](https://log4js-node.github.io/log4js-node/layouts.html).

### DateFile

This adapter will log to a file, moving old log messages to timestamped files according to a specified pattern. For example:

```javascript
const path = require('path');
const {DateFile} = require('think-logger3');

exports.logger = {
  type: 'dateFile',
  dateFile: {
    handle: DateFile,
    level: 'ALL',
    absolute: true,
    pattern: '-yyyy-MM-dd',
    alwaysIncludePattern: false,
    filename: path.join(think.ROOT_PATH, 'logs/xx.log')
  }
}

```

Then initial log would create a file called `debug.log`. At midnight, the current `debug.log` file would be rename to `debug.log-2017-03-12`(for example), and a new `debug.log` file created.

- `level`: log level
- `filename`: log base filename
- `pattern`: date filename would append to filename. A new file is started whenever the pattern for the current log entry differs from that of the previous log entry. The following strings are recognised in the pattern:
  - yyyy - the full year, use yy for just the last two digits
  - MM - the month
  - dd - the day of the month
  - hh - the hour of the day (24-hour clock)
  - mm - the minute of the hour
  - ss - seconds
  - SSS - milliseconds (although I'm not sure you'd want to roll your logs every millisecond)
  - O - timezone (capital letter o)
- `alwaysIncludePattern`: If `alwaysIncludePattern` is true, then the initial file will be `filename.2017-03-12` and no renaming will occur at midnight, but a new file will be written to with the name `filename.2017-03-13`.
- `absolute`: If `filename` is a absolute path, the `absolute` value should be `true`.
- `layout`: Layout defines the way how a log record is rendered. More layouts can see [here](https://log4js-node.github.io/log4js-node/layouts.html).


### Advance

If those adapter configuration can't satisfy your need, you can use this adapter and set config like log4js. For example:

```js
const path = require('path');
const {Basic} = require('think-logger3');

exports.logger = {
  type: 'advanced',
  advanced: {
    handle: Basic,
    appenders: {
      everything: { 
        type: 'file', 
        filename: path.join(think.ROOT_PATH, 'logs/all-the-logs.log') 
      },
      emergencies: {  
        type: 'file', 
        filename: path.join(think.ROOT_PATH, 'logs/oh-no-not-again.log') 
      },
      'just-errors': { 
        type: 'logLevelFilter', 
        appender: 'emergencies', 
        level: 'error' 
      }
    },
    categories: {
      default: { 
        appenders: ['just-errors', 'everything'], 
        level: 'debug' 
      }
    }
  }
};
```
 
另外需要注意的是，考虑到分类的用处比较少，我们不支持自定义日志分类，所以在 `categories` 中配置其它分类是无效的，但是默认分类需要存在。

This configuration means that logs level above `error` will be output to` oh-no-not-again.log` and all logs will be output to `all-the-logs.log`. All properties are as same as log4js except `handle` property. You can see more configure properties on [log4js documentation](https://log4js-node.github.io/log4js-node/api.html#configuration-object).


### Level

We support the following log level:

- ALL
- ERROR
- WARN
- INFO
- DEBUG

### Layout


`Console`, `File` and `DateFile` type supports `layout` paramter，to decide the output log format, of which value is and object, bellowing is an example:

```javascript
const path = require('path');
const {File} = require('think-logger3');

module.exports = {
  type: 'file',
  file: {
    handle: File,
    backups: 10,
    absolute: true,
    maxLogSize: 50 * 1024,  //50M
    filename: path.join(think.ROOT_PATH, 'logs/xx.log'),
    layout: {
      type: 'pattern',
      pattern: '%[[%d] [%p]%] - %m',
    }
  }
}
```
Default `Console` output format is `%[[%d] [%z] [%p]%] - %m`, which is `[time] [pid] [log level] - log content`:

- `type`：It supports the fellowing types:
    - basic
    - coloured
    - messagePassThrough
    - dummy
    - pattern
    - customize output type refers to [Adding your own layouts](https://log4js-node.github.io/log4js-node/layouts.html)
    - `pattern`：output format string, supports the following format parameters
    - `%r` - `.toLocaleTimeString()` which look like `下午5:13:04`。
    - `%p` - log level
    - `%h` - host name
    - `%m` - log mesage
    - `%d` - time，default use ISO8601 format，available formats including `ISO8601`, `ISO8601_WITH_TZ_OFFSET`, `ABSOUTE`, `DATE` or other supported format string, such as `%d{DATE}` or `%d{yyyy/MM/dd-hh.mm.ss}`。
    - `%%` - output `%` string
    - `%n` - new line
    - `%z` - get process ID from `process.pid`
    - `%[` - begin color block
    - `%]` - end color block

### Customize Handle

如果觉得提供的日志输出类型不满足大家的需求，可以自定义日志处理的 `handle`。自定义 handle 需要实现以下几个方法：
If you feel that the type of log output provided does not meet your needs, you can implement handle yourself. Custom handle need to achieve the following methods:

```javascript
module.exports = class {
  /**
   * @param {Object}  config  {}  configurations passed in
   */
  constructor(config) {

  }

  debug() {

  }

  info() {

  }

  warn() {

  }

  error() {

  }
}
```

