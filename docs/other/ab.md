# ab压力测试

## 关键概念

### PV是什么

PV是page view的简写。PV是指页面的访问次数，每打开或刷新一次页面，就算做一个pv。

计算模型：  
每台服务器每秒处理请求的数量=((80%*总PV量)/(24小时*60分*60秒*40%)) / 服务器数量 。  
其中关键的参数是80%、40%。表示一天中有80%的请求发生在一天的40%的时间内。24小时的40%是9.6小时，有80%的请求发生一天的9.6个小时当中（很适合互联网的应用，白天请求多，晚上请求少）。

简单计算的结果：  
((80%*500万)/(24小时*60分*60秒*40%))/1 = 115.7个请求/秒  
((80%*100万)/(24小时*60分*60秒*40%))/1 = 23.1个请求/秒  

最终结论：  
如果你的服务器一秒能处理231.4--347.1个请求/秒，就可以应对平均500万PV/每天。  
如果你的服务器一秒能处理46.2--69.3个请求，就可以应对平均100万PV/每天。  

## 指标说明

这里说明每秒N个请求，就是QPS。因为我关心的是应用程序处理业务的能力。
重要指标：  
Concurrency Level: 1000　　// 1000个并发请求  
Time taken for tests: 2.296 seconds　　// 整个测试持续时间  
Complete requests: 1000　　// 完成了1000个请求  
Failed requests: 170 (Connect: 0, Receive: 0, Length: 170,  Exceptions: 0)　　// 失败的请求数170  
Non-2xx responses: 170　　// 170个不是2开头的响应码，比如缓存的304  
Total transferred: 496990 bytes　　// 整个场景中的网络传输量  
HTML transferred: 103360 bytes　　// 整个场景中的HTML内容传输量  
Requests per second: 435.62 [#/sec] (mean)  

_____
1.吞吐率,大家最关心的指标之一

服务器并发处理能力的量化描述，单位是reqs/s，指的是在某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率。  

吞吐率是基于并发用户数的。这句话代表了两个含义：  
a、吞吐率和并发用户数相关  
b、不同的并发用户数下，吞吐率一般是不同的  
计算公式：总请求数/处理完成这些请求数所花费的时间，即  
Request per second=Complete requests/Time taken for tests

必须要说明的是，这个数值表示当前机器的整体性能，值越大越好  
Time per request: 2295.555 [ms] (mean)  
_____

2.用户平均请求等待时间，大家最关心的指标之二

计算公式：处理完成所有请求数所花费的时间/（总请求数/并发用户数）  
Time per request: 2.296 [ms] (mean, across all concurrent requests)  
_____

3.服务器平均请求处理时间，大家最关心的指标之三

计算公式：处理完成所有请求数所花费的时间/总请求数  
Transfer rate: 211.43 [Kbytes/sec] received  
平均每秒网络上的流量，可以帮助排除是否存在网络流量过大导致响应时间延长的问题  

## 命令

-n 在测试会话中所执行的请求个数（总数）

-c 一次产生的请求个数（单次请求次数）

-t 测试所进行的最大秒数。其内部隐含值是-n 50000，它可以使对服务器的测试限制在一个固定的总时间以内。默认时，没有时间限制。

-p 包含了需要POST的数据的文件。

-P 对一个中转代理提供BASIC认证信任。用户名和密码由一个:隔开，并以base64编码形式发送。无论服务器是否需要(即, 是否发送了401认证需求代码)，此字符串都会被发送。

-T POST数据所使用的Content-type头信息。

-v 设置显示信息的详细程度-4或更大值会显示头信息，3或更大值可以显示响应代码(404,200等),2或更大值可以显示警告和其他信息。

-V 显示版本号并退出。

-w 以HTML表的格式输出结果。默认时，它是白色背景的两列宽度的一张表。

-i 执行HEAD请求，而不是GET。

-x 设置<table>属性的字符串。

-X 对请求使用代理服务器。

-y 设置<tr>属性的字符串。

-z 设置<td>属性的字符串。

-C 对请求附加一个Cookie:行。其典型形式是name=value的一个参数对，此参数可以重复。

-H 对请求附加额外的头信息。此参数的典型形式是一个有效的头信息行，其中包含了以冒号分隔的字段和值的对(如,"Accept-Encoding:zip/zop;8bit")。

-A 对服务器提供BASIC认证信任。用户名和密码由一个:隔开，并以base64编码形式发送。无论服务器是否需要(即,是否发送了401认证需求代码)，此字符串都会被发送。

-h 显示使用方法/帮助信息。

-d 不显示"percentage served within XX [ms] table"的消息(为以前的版本提供支持)。

-e 产生一个以逗号分隔的(CSV)文件，其中包含了处理每个相应百分比的请求所需要(从1%到100%)的相应百分比的(以微妙为单位)时间。由于这种格式已经“二进制化”，所以比'gnuplot'格式更有用。

-g 把所有测试结果写入一个'gnuplot'或者TSV(以Tab分隔的)文件。此文件可以方便地导入到Gnuplot,IDL,Mathematica,Igor甚至Excel中。其中的第一行为标题。

-i 执行HEAD请求，而不是GET。

-k 启用HTTP KeepAlive功能，即在一个HTTP会话中执行多个请求。默认时，不启用KeepAlive功能。

-q 如果处理的请求数大于150，ab每处理大约10%或者100个请求时，会在stderr输出一个进度计数。此-q标记可以抑制这些信息。

## 示例说明

一次请求1000个，总共请求10000次，请求的地址:http://xdzx.xueda.com/api/system/external-pub/get-user-info?uid=358327

```ab
ab -c 1000 -n 10000 http://xdzx.xueda.com/api/system/external-pub/get-user-info?uid=358327
```

ab软件版本和官方信息声明

```ab
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/
```

//执行进度

```ab
Benchmarking xdzx.xueda.com (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests
```

web服务名称和版本+URL+端口号

```ab
Server Software:        nginx/1.14.0
Server Hostname:        xdzx.xueda.com
Server Port:            80
```

测试结果

```ab
Document Path:          /api/system/external-pub/get-user-info?uid=358327  #(测试URL路径)
Document Length:        104 bytes #(测试URL返回的文档大小)
Concurrency Level:      1000 #（并发数）
Time taken for tests:   16.575 seconds #（压力测试消耗的总时间）
Complete requests:      10000 #（压力测试的总次数）
Failed requests:        1312 #（失败的请求数）
   (Connect: 0, Receive: 0, Length: 1312, Exceptions: 0)  
Write errors:           0 #（网络连接写入错误数）
    :      1312 #（1312个不是2开头的响应码，比如缓存的304等）
Total transferred:      4975376 bytes #（所有请求的响应数据长度总和。包括每个HTTP响应数据的头信息和正文数据的长度）
HTML transferred:       1242048 bytes #（所有请求的响应数据中正文数据的总和，也就是减去了Total transferred中HTTP响应数据中的头信息的长度）
Requests per second:    603.33 [#/sec] (mean) #（平均每秒的请求数，务器并发处理能力的量化描述，单位是reqs/s，指的是在某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率。计算公式：总请求数/处理完成这些请求数所花费的时间，即Request per second=Complete requests/Time taken for tests必须要说明的是，这个数值表示当前机器的整体性能，值越大越好。）
Time per request:       1657.462 [ms] (mean) #（所有并发用户(这里是1000)都请求一次的平均时间）
Transfer rate:          293.14 [Kbytes/sec] received #(传输速率，单位：KB/s，表示这些请求在单位时间内从服务器获取的数据长度，计算公式：Total trnasferred/ Time taken for tests，这个统计很好的说明服务器的处理能力达到极限时，其出口宽带的需求量。)

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        6    8   2.3      7      44
Processing:    10  923 1584.6    593   13317
Waiting:       10  922 1584.6    593   13317
Total:         16  931 1584.8    601   13329

Percentage of the requests served within a certain time (ms)
  50%    601 #（50%的请求在601ms内返回）
  66%    804
  75%    969
  80%   1077
  90%   1382 #（90%的请求在1382ms内返回）
  95%   1675
  98%   8673
  99%  10144
 100%  13329 (longest request)
```
