## TypeScript

[TypeScript](http://www.typescriptlang.org/) is a free and open source programming language developed by Microsoft. It's a superset of JavaScript, adding optional static types to the JavaScript, which is great for large projects.

ThinkJS 3.2 began to support users to create TypeScript type of project, and the development will automatically compile, automatically update, without manual compilation and other complex operations. If you want to know more implementation details, refer to [ThinkJS 3.0 如何实现对 TypeScript 的支持](https://zhuanlan.zhihu.com/p/31057738).

### Create TypeScript project

Think-cli after version 2.1.1 can create a TypeScript project with the following command:

```sh
thinkjs new project-name typescript
```

### Introduced Extend module definition

After generating a TypeScript project template with think-cli (hereinafter collectively referred to as the project template), the `src/index.ts` file is generated automatically. Here you need to configure which Extend modules are used by the project so that TS's IntelliSense will take effect.

``` js
import * as ThinkJS from '../node_modules/thinkjs';

// Extend module in project
import './extend/controller';
import './extend/logic';
import './extend/context';
import './extend/think';
import './extend/service';
import './extend/application';
import './extend/request';
import './extend/response';

// external Extend module
import 'think-view';
import 'think-model';
import 'think-i18n';
// more extend module reference [think-awesome](https://github.com/thinkjs/think-awesome)

export const think = ThinkJS.think;
```

### Get the model and service types
```js
// in controller
import { think } from 'thinkjs';
import SomeService from '../service/someservice';
import SomeModel from '../model/somemodel';

export default class extends think.Controller {
  indexAction() {
    const serviceInstance: SomeService = think.service('someservice');
    const modelInstance: SomeModel = think.model('somemodel');
  }
}
```

### TSLint

TypeScript and JavaScript project writing style is very close, as long as you get used to it after a period of time to adapt. We also configured a set of TSLint rules based on the characteristics of the ThinkJS project to include in the project template. Use TSLint to more quickly enforce specifications and protect code in teams.

### Compile and deploy

In the development environment can use `think-typescript` to compile, also support `tsc` direct compilation, compiled code and JS version is common.
