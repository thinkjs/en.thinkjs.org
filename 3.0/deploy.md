## Production Deployment

When development and the testing is done, we need to deploy to online machine and then provide services.

### Compile


如果修改了 [babel preset](/doc/3.0/babel.html#toc-2cb)，那么需要把 package.json 里的 compile 命令（babel src/ --presets think-node --out-dir app/）作对应的修改。

如果代码不需要转译，那么直接上线 `src/` 目录的代码即可。

If project code needs to be compiled, though it outputs `app/` directory from `src/`, we suggest to run command `npm run compile` during deployement, to avoid some unexpected impact. Or pull the latest code in a clean directory and execute the compile command.

If you modify the [babel preset](/doc/3.0/babel.html#toc-2cb), then you need to modify the compile command (babel src/ --presets think-node --out-dir app/） in package.json  the corresponding changes.

If the code does not need translation, then deploy `src/` directory can be.

### Production Environment

项目创建时，会自动在项目根目录下创建一个名为 `production.js` 的文件，该文件为生产环境运行的入口文件，定义的 `env` 为 `production`。切不可在生产环境把 `development.js` 作为入口文件来启动服务。

When the project is created, a file named `production.js` is automatically created in root directory, which is the entry file for the production environment and defines` env` as `production`. Never use `development.js` in the production environment as an entry file to start the service.

### Service Management

#### PM2

PM2 is specific designed module for Node.js service management. PM2 need to be install globally:
```
sudo npm install -g pm2
```
After installation, we have `pm2` command line command.

When project is created, a file named `pm2.json` is created in root directory, of which content is as bellow:

```json
{
  "apps": [{
    "name": "demo",
    "script": "production.js",
    "cwd": "/Users/welefen/Develop/git/thinkjs/demo",
    "max_memory_restart": "1G",
    "autorestart": true,
    "node_args": [],
    "args": [],
    "env": {}
  }]
}
```

Change `name` to your project name, and modify `cwd` field to online project path.

##### Start Project

Run `pm2 start pm2.json` on project root directory to start application, and you will see the following information:

![](https://p5.ssl.qhimg.com/t011347d36ca082a2e4.jpg)

##### Restart Project

When there is code changed, we need to restart service to make it effective.

The easiest way to restart service is through `pm2 restart pm2.json`, but this approch can result in temporary service interruptions. (restarting the service takes time and restarting can result in inability to process user requests which cause service interruption). If you don't want service interruption, then you can send a signal to restart, the specific code:

```
pm2 sendSignal SIGUSR2 pm2.json
```
pm2 will send `SIGUSR2` singal to framework, after the main process capture this signal, it will fork a new child process for incomming service, and then gradually close previous child process so as to achiveve none-interrupt service restart.

##### cluster mode

Framework will force the use of cluster, and provide service in master/worker way, so you can't open cluster mode in `pm2` (if you open, service will report errors and exist on start up).

#### Manually manage process

##### project start

If you don't want to use PM2 to manage service in production environment, you can manually manage it through a script, , first run `node production.js' to start service.

When service is start up and accessible, start service with `nohub node production.js &`, `nohub` and `&` means run service in background, after that you can see a log similar to the following:

```
$ nohup node production.js &
[2] 1114
appending output to nohup.out
``` 

After seeing the output, press Enter, execute the `exit` command to exit the current terminal, so the service is running in the background.

After the start is complete, you can see the specific node process by `ps aux | grep node`:

```text
welefen           3971   0.0  0.3  3106048  46244 s001  S+   11:14AM   0:00.65 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3970   0.0  0.3  3106048  46064 s001  S+   11:14AM   0:00.64 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3969   0.0  0.3  3106040  46248 s001  S+   11:14AM   0:00.65 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3968   0.0  0.3  3106048  46400 s001  S+   11:14AM   0:00.65 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3967   0.0  0.3  3106048  46608 s001  S+   11:14AM   0:00.65 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3966   0.0  0.3  3106048  46432 s001  S+   11:14AM   0:00.65 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3965   0.0  0.3  3106040  46828 s001  S+   11:14AM   0:00.65 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3964   0.0  0.3  3106048  46440 s001  S+   11:14AM   0:00.64 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3963   0.0  0.2  3135796  40960 s001  S+   11:14AM   0:00.31 node production.js
```

The first few are fork process, the last one is master process.

##### Restart Service

当代码修改后，需要重启服务，最简单的办法就是找到主进程的 pid，然后通过 `kill -9 PID` 杀死进程然后重新启动。如果不想中断服务，那么可以给主进程发送 `SIGUSR2` 信号来完成：

When the code changes requires restart the service, the easiest way is to find the main process pid, kill the process by `kill -9 PID` and then restart. If you do not want to interrupt the service, you can send `SIGUSR2` signal to the main process to finish:

```
kill -s USR2 PID
```

比如上面打印出来的日志中主进程的 pid 为 3963，那么可以通过 `kill -s USR2 3963` 来无中断重启服务。当然每次这么执行比较麻烦，可以包装成一个简单的脚本来执行。
For example, the master process's pid being printed in log above is 3963, you can `kill -s USR2 3963` to restart the service without interruption. Of course, it is troublesome to do so everytime, you can create a simple script for that.
```sh
#!/bin/sh
cd PROJECT_PATH; # enter root directory
nodepid=`ps auxww | grep node | grep production.js | grep -v grep | awk '{print $2}' `
if [ -z "$nodepid" ]; then
    echo 'node service is not running'
    nohup node production.js > ~/file.log 2>&1 & 
else
    echo 'node service is running'
    kill -s USR2 $nodepid 2>/dev/null
    echo 'gracefull restart'
fi
```

### use nginx

虽然 Node.js 自身可以直接创建 HTTP(S) 服务，但生产环境不建议直接把 Node 服务可以对外直接访问，而是在前面用 WebServer（如：nginx） 来挡一层，这样有多个好处：

* 可以更好做负载均衡，比如：同一个项目，启动多个端口的服务，用 nginx 做负载
* 静态资源使用 nginx 直接提供服务性能更高
* HTTPS 服务用 nginx 提供性能更高

创建项目时，会在项目根目录下创建了一个名为 `nginx.conf` 的配置文件：

Although Node.js itself can directly create HTTP(S) services, it is not recommand so in production environment, instead uses WebServer (such as: nginx) in front, which has several benefits:

* Can do better load balancing, such as: the same project, start the service of multiple ports, using nginx to do the load
* Static resources using nginx has higher performance
* HTTPS service performance is higher

When creating a project, a configuration file named `nginx.conf` is created in the project root directory:

```
server {
    listen 80;
    server_name example.com www.example.com;
    root /Users/welefen/Downloads/demo/www;
    set $node_port 8360;

    index index.js index.html index.htm;
    if ( -f $request_filename/index.html ){
        rewrite (.*) $1/index.html break;
    }
    if ( !-f $request_filename ){
        rewrite (.*) /index.js;
    }
    location = /index.js {
        proxy_http_version 1.1;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass http://127.0.0.1:$node_port$request_uri;
        proxy_redirect off;
    }

    location ~ /static/ {
        etag         on;
        expires      max;
    }
}
```

`server_name`,` root`, `port` field value is configured according to the actual situation, and then the configuration file is soft link to the nginx configuration file directory, and finally restart the nginx service (you can `nginx -s reload` to reload the configuration file).


--------

If you still want to provide services directly through Node.js, it is also possible to listen directly to port 80 or port 443 (some environments require `sudo` to listen on these two ports).
### HTTPS

现代网站强制建议使用 HTTPS 访问，这样可以提供网站内容的安全性，避免内容被监听、篡改等问题。如果不愿意支付证书的费用，可以使用 [Let's Encrypt](https://letsencrypt.org/) 提供的免费 SSL/TLS 证书，可以参见文章 [Let's Encrypt，免费好用的 HTTPS 证书](https://imququ.com/post/letsencrypt-certificate.html)。

Modern websites highly suggest using HTTPS to provide the security of the website content and prevent the content from being intercepted and tampered with. If you do not want to pay for a certificate, you can use the free SSL/TLS certificate provided by [Let's Encrypt](https://letsencrypt.org/), which can be found in the article [Let's Encrypt，免费好用的 HTTPS 证书](https://imququ.com/post/letsencrypt-certificate.html).

### FAQ

#### Why static resource is not accessible in production environment？

创建项目时，会自动生成中间件配置 `src/config/middleware.js`（多模块项目文件为 `src/common/config/middleware.js`），里面有个 resource 中间件用来处理静态资源的请求，但这个中间件默认只在开发环境下开启的，线上环境是关闭的，所以会看到上线后静态资源访问不了的情况。

线上环境静态资源请求推荐用 nginx 来处理，这样性能会更高，对 Node 服务的压力也会小一些。如果非要框架处理静态资源请求，那么可以把 `src/config/middleware.js` 里的配置打开即可。

On project initiation, it will auto-generate middleware config file `src/config/middleware.js` (multi-module project `src/common/config/middleware.js`). There is a middleware for handling static resources request, but this middleware by default only for development environment, so you will see static resources can not access after going online.

It is recommanded to use nginx to handle static resource online, of which performance will be higher and lower load for Node service. If you still want to use framework to deal with static resource requests, modify `src/config/middleware.js` to enable this middleware can do so.

```js
module.exports = [
  ...
  {
    handle: 'resource',
    enable: true // always open, default is `enable: isDev` which means only open in development environment
  },
  ...
]
```