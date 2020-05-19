# Laravel 广播之公共频道，的使用redis方式

## 一、搭建laravel-echo-server服务，见上一篇文章

## 二、laravel后台

1. 配置
```
#1 config/app.php 中取消注释
App\Providers\BroadcastServiceProvider::class,

#2 修改.env文件中广播驱动配置
BROADCAST_DRIVER=redis

#3 redis连接配置
REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379

#4 redis key的前缀
REDIS_PREFIX=laravel_database_
```

2. 创建事件，随便以哪种方式，代码正确就OK

```php
namespace Modules\Crm\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;

class NoticeEvent implements ShouldBroadcast
{
    use SerializesModels, InteractsWithSockets, Dispatchable;
    public $data;
    public function __construct($data)
    {
        $this->data = $data;
    }
    /**
     * 定义渠道
     */
    public function broadcastOn()
    {
        return new Channel("webNotice");
    }
    /**
     * 定义事件
     */
    public function broadcastAs()
    {
        return 'task';
    }
    /**
     * 广播的数据格式
     */
    public function broadcastWith(){
        return ['data' => $this->data, 'status'=>'ok'];
    }
}
```

2. 控制器，随便写到什么位置没关系，而对对应好就ok

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
      return view('crm::admin.socket.index');
  }
  /**
   * 测试触发事件
   * 根据type判断触发哪个事件
   * msg 为发送的消息
   */
  public function trigger(Request $request)
  {
    if ($request->input('type') == 'private') {

    } else {
      broadcast(new NoticeEvent(["msg"=>$request->input("msg")]));
    }
  }
}

```

3. routers
```php 
use Illuminate\Support\Facades\Route;

Route::prefix('crm')->group(function () {
    Route::get('/socket', 'SocketController@index')->name('admin.crm.socket');
});

```


## 三、laravel前端

1. bootstrap.js部分查看上一篇文章

2. admin.crm.socket页面代码
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="csrf-token" content="{{ csrf_token() }}">
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
            },
            methods: {
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
                    axios.post('/admin/crm/socket/trigger', {
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
进入crm/socket页面，点公共消息按扭，查看消息框内的内容变化，有变化则表示成功


