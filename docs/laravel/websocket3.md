# Laravel 广播之私有频道，使用redis方式

私有频道的处理，比较繁琐，需要有权限认证等一系列判断

## 一、搭建laravel-echo-server服务，见上一篇文章

## 二、laravel后台

1. 创建事件，随便以哪种方式，代码正确就OK，

```php
namespace Modules\Crm\Events;

use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Foundation\Events\Dispatchable;

class OrderUpdateEvent implements ShouldBroadcast
{
    use SerializesModels, InteractsWithSockets, Dispatchable;

    public $data;

    /**
     * Create a new event instance.
     *
     * @return void
     */
    public function __construct($data)
    {
        $this->data = $data;
    }

    /**
     * 定义渠道
     */
    public function broadcastOn()
    {
        //主要是这里创建了私有频道
        return new PrivateChannel("orderUpdate");
    }

    /**
     * 定义事件
     */
    public function broadcastAs()
    {
        return 'update';
    }

    /**
     * 广播的数据格式
     */
    public function broadcastWith(){
        return ['data' => $this->data, 'status'=>'ok'];
    }
}

```

2. 控制器，中加入对私有频道的数据处理

```php
namespace Modules\Crm\Http\Controllers\Admin;
use Illuminate\Http\Request;
/**
 * @name socket测试
 */
class SocketController extends CrmController
{
  /**
   * @name socket测试页面
   */
  public function index(Request $request)
  {
        $user = auth()->user();
        return response()
            ->view('crm::admin.socket.index')
            ->header('Authorization', auth('api')->tokenById($user->id));//头里面包含token
  }
  /**
   * 测试触发事件
   * 根据type判断触发哪个事件
   * msg 为发送的消息
   */
  public function trigger(Request $request)
  {
    if ($request->input('type') == 'private') {
      //todo 私有渠道消息处理
      broadcast(new OrderUpdateEvent(['msg' => $request->input("msg")]));
    } else {
      broadcast(new NoticeEvent(["msg"=>$request->input("msg")]));
    }
  }
}

```

3. routes/channel中加入以下

```php 
Broadcast::channel('orderUpdate', function ($user) {
    return true;
},['guards' => ['api']]);
```

4. 注意，auth("api")使用的是jwt,config/auth.php中对应配置改为如下，其它有关jwt内容参考jwt

```php
'guards' => [
    'api' => [
        'driver' => 'jwt',//这里改下就ok
        'provider' => 'users',
        'hash' => false,
    ],
],
```


## 三、laravel前端

1. bootstrap.js 中的连接需要加入auth_token,如下

```js
let headers = getHeaders();
window.Echo = new Echo({
    broadcaster: 'socket.io',
    host: 'localhost:6001',
    key: '6fa6403e606637ce',//对应laravel-echo-server.json文件中的id
    auth:{headers:{
        "Authorization":"Bearer " + headers["authorization"],
    }}
});

//临时的一个方法，此函数要移动到其它地方
function getHeaders(){
    var req = new XMLHttpRequest();
    req.open('GET', document.location.href, false);
    req.send(null);
    var headerArr = req.getAllResponseHeaders().split('\n');
    var headers = {};
    headerArr.forEach(item=>{
        if(item!==''){
		    var index = item.indexOf(':');
	        var key = item.slice(0,index);
	        var value = item.slice(index+1).trim();
	        headers[key] = value
	    }
	    
    })
    return headers
}

```

2. admin.crm.socket页面代码最终的样子

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="csrf-token" content="<?php echo csrf_token() ?>">
    <script src="http://127.0.0.1:6001/socket.io/socket.io.js"></script>
    <script src="/js/app.js"></script>
</head>

<body>
    <div id="app">
        <div>socket消息框，发送消息后，此处出现内容则ok</div>
        <div style="width:600px;height:200px;border:1px solid #ccc;overflow:scroll" v-html="all">

        </div>
        <form action="">
            <input type="text" v-model="msg">
            <button v-on:click.stop.prevent="sendMsg('public')">公共消息</button>
            <button v-on:click.stop.prevent="sendMsg('private')">私有消息</button>
        </form>
    </div>
    <script>
        
        new Vue({
            el: '#app',
            data: {
                msg: "",
                all: ""
            },
            mounted: function() {
                this.channel();
                this.privateChannel();
            },
            methods: {
                //私有
                privateChannel: function() {
                    console.dir('私有开始')
                    Echo.private('orderUpdate')
                        .listen('.update', (e) => {
                            let msg = "privateMsg:" + e.data.msg + "<br>";
                            this.all += msg;
                            console.log(e.msg);
                        });
                },
                //公有的
                channel: function() {
                    Echo.channel('webNotice')
                        .listen('.task', (e) => {
                            let msg = "公共消息:" + e.data.msg + "<br>";
                            this.all += msg;
                        });
                },
                sendMsg: function(type) {
                    console.dir(type);
                    axios.post('/crm/socket/trigger', {
                            msg: this.msg,
                            type: type
                        })
                        .then(function(response) {
                            console.log(response);
                        })
                        .catch(function(error) {
                            console.log(error);
                        });

                }
            }
        })
    </script>
</body>

</html>
```

## 四、测试

1. 启动队列

```
php artisan queue:listen --tries=1
```

2. 功能测试

```
进入crm/socket页面，点各消息按扭，查看消息框内的内容变化，有变化则表示成功
```


## 五、注意

1. 如果总提示csrf_token验证不通过，可暂时关闭中间件中的验证，App\Http\Middleware\VerifyCsrfToken.php中添加如下代码

```php
protected $except = [
    "broadcasting/auth"
];
```

2. laravel-echo-server 的devMode改为true，否则没有log不容易调试

```js
"devMode": true,
```

3. 私有通道比较繁琐，是因为多了权限校验，权限处理好了，就没什么问题了


