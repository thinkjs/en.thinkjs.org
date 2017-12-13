## Quick start

With the scaffolding provided by ThinkJS, you can quickly create a project. In order to be able to use more ES6 features, the framework requires Node.js to be at least `6.x`, and we suggest to use [LTS version](https://nodejs.org/en/download/).

### Install ThinkJS


```sh
$ npm install -g think-cli
```

After the installation is complete, there will be a `thinkjs` command in the system (think-cli's version number can be checked by `thinkjs -V`, which is not the version of thinkjs). If you can not find this command, confirm that the environment variables are correct.

If you are upgrading from `2.x`, you need to delete the previous command and then reinstall it.

### Uninstall old version

```sh
$ npm uninstall -g thinkjs
```

### Create project

Execute `thinkjs new [project_name]` to create a project, such as:

```
$ thinkjs new demo;
$ cd demo;
$ npm install; 
$ npm start; 
```

When done you will see following log message:

```
[2017-06-25 15:21:35.408] [INFO] - Server running at http://127.0.0.1:8360
[2017-06-25 15:21:35.412] [INFO] - ThinkJS version: 3.0.0-beta1
[2017-06-25 15:21:35.413] [INFO] - Enviroment: development
[2017-06-25 15:21:35.413] [INFO] - Workers: 8
```

Open browser to visit `http://127.0.0.1:8360/`, or to replace IP to specific address if project is create on a remote server.


### Project structure

The default project structure is as follows:

```text
|--- development.js   // Development entry file
|--- nginx.conf  //nginx config file
|--- package.json
|--- pm2.json //pm2 config file
|--- production.js // Production entry file
|--- README.md
|--- src
| |--- bootstrap  // Auto-execute directory on start up
| | |--- master.js // Execute in Master process 
| | |--- worker.js // Execute in worker process
| |--- config  // configuration directory
| | |--- adapter.js  // adapter config
| | |--- config.js  // default config file
| | |--- config.production.js  // production environment default config file, will merge into config.js
| | |--- extend.js  //extend config file
| | |--- middleware.js // Middleware config file
| | |--- router.js // Custom route config file 
| |--- controller  // Controller folder
| | |--- base.js
| | |--- index.js
| |--- service  // Service folder
| | |--- **.js // User defined service
| |--- logic //logic directory
| | |--- index.js
| |--- model // Model directory
| | |--- index.js
|--- view  // Template direcotry
| |--- index_index.html
|--- www
| |--- static  // static resource direcotry
| | |--- css
| | |--- img
| | |--- js
```
