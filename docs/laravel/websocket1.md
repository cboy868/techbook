# Laravel 广播使用redis方式，基础依赖及配置

## 一、搭建laravel-echo-server服务，使用docker-compose方式

1. docker-compose.yml中添加echo代码，需要redis服务 

```docker-compose
version: "3"
services:
  echo:
    image: oanhnn/laravel-echo-server
    container_name: laravel-echo-server
    depends_on:
        - redis
    links:
        - redis
    volumes:
        - ./services/echo:/app
    ports:
        - "6001:6001"
```

2. 执行命令启动docker

```docker
docker-compose up -d
```

3. 进入docker生成key

```
 laravel-echo-server client:add
```

4. 配置文件在启动时会生成，添加完key以后，app/laravel-echo-server.json文件类似如下内容

```json
{
	"apiOriginAllow": {},
	"authEndpoint": "/broadcasting/auth",
	"authHost": "http://crm.xueda.local",
	"clients": [
		{
			"appId": "6fa6403e606637ce",
			"key": "038fcf9c0b5deb8fb3de1dd1015b65be"
		}
	],
	"database": "redis",
	"databaseConfig": {
		"redis": {
			"host": "redis",
			"port": "6379",
			"keyPrefix": "laravel_database_",
			"options": {
				"db": "0"
			}
		},
		"sqlite": {
			"databasePath": "/database/laravel-echo-server.sqlite"
		},
		"publishPresence": true
	},
	"devMode": true,
	"host": null,
	"port": 6001,
	"protocol": "http",
	"sslCertPath": "",
	"sslKeyPath": "",
	"sslCertChainPath": "",
	"sslPassphrase": "",
	"socketio": {},
	"subscribers": {
		"http": true,
		"redis": true
	}
}
```

检测环境是否ok,访问http://your_host_address:6001/socket.io/socket.io.js,打印出正常的js代码ok  

## 二、laravel前端 bootstrap.js内容处理

1. 需要安装的服务

```
npm install --save laravel-echo
npm install --save socket.io-client
//vue并非广播必须，只是页面中会用到
npm install --save vue
```


2. resources/js/bootstrap.js中加入代码，改完后别忘记编译

```js
window.Vue = require('vue');
import Echo from 'laravel-echo';

window.Echo = new Echo({
    broadcaster: 'socket.io',
    host: 'localhost:6001',
    key: '6fa6403e606637ce'//对应laravel-echo-server.json文件中的id
});
```

3. 主目录页面下编译
```sh
npm run dev
```