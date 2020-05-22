# Laravel 使用 WebSocket

## 一、搭建laravel-echo-server服务，只包括主流程，其它参考文档，两种使用方式，任选其一
代码地址: [laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server)

服务器安装  
_____

1. 普通安装
```node
npm install laravel-echo-server -g;
```

2. 初始化，会有一些选择，可参考文档，也可后期修改，生成的内容在laravel-echo-server.json中
```node
laravel-echo-server init
```

3. 启动项目
```
laravel-echo-server start
```

4. 生成key
```
 laravel-echo-server client:add
```

docker方式
_____
文件依赖dnmp[地址](https://github.com/cboy868/dnmp)
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
        - ./data/echo:/app
    ports:
        - "6001:6001"
```

2. 生成key
```
 laravel-echo-server client:add
```

3. 执行命令启动docker
```docker
docker-compose up -d
```

检测环境是否ok,访问http://your_host_address:6001/socket.io/socket.io.js,打印出正常的js代码ok  

## 二、laravel后台

1. 定义trait

```php
namespace App\HelpTrait;

use GuzzleHttp\Client;

trait BroadcastHttpPush
{
    public function push($data)
    {
        //docker
        // $baseUrl = 'http://echo:6001/';//docker
        //普通服务
        $baseUrl = 'http://localhost:6001/';//url
        $appId = '6fa6403e606637ce';//appId
        $key = '038fcf9c0b5deb8fb3de1dd1015b65be';//key
        $httpUrl = $baseUrl . 'apps/' . $appId . '/events?auth_key=' . $key;
      
        $client = new Client([
            'base_uri' => $httpUrl,
            'timeout' => 2.0,
        ]);
        $response = $client->post($httpUrl, [
            'json' => $data
        ]);
        $code = $response->getStatusCode();
    }
}
```

2. 在控制器中使用

```php
namespace App\Controllers;

use App\HelpTrait\BroadcastHttpPush;

class TestSocketController
{
    use BroadcastHttpPush;
    
    public function index()
    {
        $broadcastChannel = array(
            "channel" => "private-Message",   // 通道名，`private-`表示私有
            "name" => "sayHello",    // 事件名
            "data" => array(
                "status" => 200, 
                "message" => "hello world!"
            )
        );
        $this->push($broadcastChannel);
    }

    public function front()
    {
        return view("test.front");
    }
}

```

3. web.php 加入 routers

```php
Route::get('/test', "TestSocketController@index")->name('test.websocket');
Route::get('/test/testecho', "TestSocketController@front")->name('test.websocket.echo');
```


## 三、laravel前端

1. resources/js/bootstrap.js中添加代码  

```javascript
import Echo from 'laravel-echo';

window.Echo = new Echo({
    broadcaster: 'socket.io',
    host: 'localhost:6001',
    key: '6fa6403e606637ce',
    auth:
    {
        headers:
        {
            Authorization: 'Bearer 038fcf9c0b5deb8fb3de1dd1015b65be'
        }
    }
});
```

2. 项目目录中编译js

```
npm run dev
```

3. blade模板中使用,编辑resources/views/test/front.blade.php模板代码

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="csrf-token" content="<?php echo csrf_token()?>">
    <script src="http://127.0.0.1:6001/socket.io/socket.io.js"></script>
    <script src="/js/app.js"></script>
</head>

<body>
    <script>
        console.log(Echo);
        Echo.channel('private-Message')
            .listen('.sayHello', (e) => {
                console.log(e);
                console.log(e.message);
            });
    </script>
</body>
</html>
```

## 四、测试

1. 打开页面1 http://your-domain-name:port/test/testecho
2. 发送消息 http://your-domain-name:port/test
3. 查看页面1控制台是否有消息打印
