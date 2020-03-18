# redis锁

要解决的问题是，避免业务逻辑中对数据重复处理。本文所有代码均为伪代码，只考虑业务逻辑

### 第一种方式
使用redis的incr函数，每次调用此函数时，如果反回值大于1，则表示此锁正在使用中，从而实现加锁
```
public function testRedisLock()
{
    #1进入关键业务逻辑，加锁
    $key = "productId"
    $ttl = 30

    try{
        if ($redis->incr($key)>1) {//大于1表示，已加锁
            return
        }
        //为防止锁无法释放，还要设置过期时间
        $redis->expire($key, $ttl)
        //进行业务逻辑处理...

    } cache(Exception $e) {
        //异常处理
    } finally {
        $redis->del($key)
    }

}

```
缺点：破坏原子性操作，上锁机制，与设置过期时间，是分开的，这中间如果服务器宕机或者其它故障，会造成锁无法释放


### 第二种方式
使用redis的setnx(key, value)方式，如果值不存在，则反回1,如果存在，则反回0，为避免锁不释放，仍需要设置过期时间
 
```
public function testRedisLock()
{
    #1进入关键业务逻辑，加锁
    $key = "productId"
    $ttl = 30

    try{
        if (!$redis->setnx($key, 1)) {//未设置成功，则反回
            return
        }
        //为防止锁无法释放，还要设置过期时间
        $redis->expire($key, $ttl)
        //进行业务逻辑处理...

    } cache(Exception $e) {
        //异常处理
    } finally {
        //删除redis key
        $redis->del($key)
    }

}

```
缺点：破坏原子性操作，上锁机制，与设置过期时间，是分开的，这中间如果服务器宕机或者其它故障，会造成锁无法释放


### 第三种方式
使用redis的set($key, $value, array('nx', 'ex' => $ttl))方式，本方法就是setnx加了个过期时间，但是原子操作
 
```
public function testRedisLock()
{
    #1进入关键业务逻辑，加锁
    $key = "productId"//此值，应该是业务过来的某个值
    $ttl = 30

    try{
        if (!$redis->set($key, $value, array('nx', 'ex' => $ttl))) {//未设置成功，则反回
            return
        }
        //开始进行业务逻辑处理...

    } cache(Exception $e) {
        //异常处理
    } finally {
        //删除redis key
        $redis->del($key)
    }
}

```
缺点：设置的过期时间为ttl，如果ttl<过期时间，那么客户端A业务处理还没完成，锁就已经被解开了，客户端B请求过来后会加新锁，客户端A请求业务处理完成后，删除的是客户端B请求上的锁，造成锁失效。


### 第四种方式

仍使用redis的set($key, $value, array('nx', 'ex' => $ttl))方式，本方法就是setnx加了个过期时间，但是原子操作
 
```
public function testRedisLock()
{
    #1进入关键业务逻辑，加锁
    $key = "productId"//此值，应该是业务过来的某个值
    $ttl = 30

    $value = uuid()

    try{
        if (!$redis->set($key, $value, array('nx', 'ex' => $ttl))) {//未设置成功，则反回
            return
        }
        //开始进行业务逻辑处理...

    } cache(Exception $e) {
        //异常处理
    } finally {
        //判断是自己的锁，再删除redis key
        if ($value == $redis->get($key)) {
            $redis->del($key)
        }
    }
}

```
问题:  
1、 redis发现锁失败了要怎么办？中断请求还是循环请求？  
2、 循环请求的话，如果有一个获取了锁，其它的在去获取锁的时候，是不是容易发生抢锁的可能？  
### 第五种方式
针对问题1：使用循环请求，循环请求去获取锁  
针对问题2：针对第二个问题，在循环请求获取锁的时候，加入睡眠功能，等待几毫秒在执行循环  

```
try {
    do {  //针对问题1，使用循环
        $ttl = 10;
        $roomid = 10001;
        $key = 'room_lock';
        $value = 'room_'.$roomid;  //分配一个随机的值针对问题3
        $isLock = $redis->set($key, $value, array('nx', 'ex' => $ttl));//ex 秒
        if ($isLock) {
            if (redis->get($key) == $value) {  //防止提前过期，误删其它请求创建的锁
                //执行业务代码
                continue;//执行成功删除key并跳出循环
            } else {
                //如果是两次值不同，说明A请示的业务还未处理完，$key超时，B进来了，此时把$isLock设置为false，一段时间后重新请求
                $sleepTime = 1000//此值应该取一个递增值
                $isLock = false 
                sleep($sleepTime)//睡眠后重新请求
            }
        } else {
            usleep(5000); //睡眠，降低抢锁频率，缓解redis压力，针对问题2
        }
    } while(!$isLock);
} catch (Exception $e) {

} finally{
    if (redis->get($key) == $value) {  //防止提前过期，误删其它请求创建的锁
        $redis->del($key);
    }
}

```

仍存在的问题  
redis 主从结构，如果redis主服务器已接收到锁，但在同步从服务时崩溃，此时如何处理？

### 官方提供
[解释](https://www.cnblogs.com/ironPhoenix/p/6048467.html)
```
$servers = [
    ['127.0.0.1', 6379, 0.01],
    ['127.0.0.1', 6389, 0.01],
    ['127.0.0.1', 6399, 0.01],
];

$redLock = new RedLock($servers);

//加锁
$lock = $redLock->lock('my_resource_name', 1000);

//删除锁
$redLock->unlock($lock)
```