---
layout: post
title: http的get接口和post接口性能压测
categories: [test ]
tags: [performance]
description: http服务的性能测试
---

## 概要
http的get接口使用http_load,  post接口使用ab压测


## http_load

##### 环境安装 
  
 http_load 非常轻量级， 只需要安装http_load的包即可 。 
 
```
yum install http_load
``` 
##### 压测方法
 

```
$http_load
usage:  http_load [-checksum] [-throttle] [-proxy host:port] [-verbose] [-timeout secs] [-sip sip_file]
            [-cipher str]
            -parallel N | -rate N [-jitter]
            -fetches N | -seconds N
            url_file
One start specifier, either -parallel or -rate, is required.
One end specifier, either -fetches or -seconds, is required.
```
**参数说明**

- url_file : 准备好压测的url文件  
- -p  / -r  ： 并发数 / qps (两者取其一，一个是指定并发，一个是指定qps) 
- -f / -s :  请求数目/压测时间 （两者取其一， 一个是请求总条目数，一个是指定压测的时间） 

**结果说明 **

```
$http_load -p 10  -s 600 query_general 2>/tmp/err
213609 fetches, 10 max parallel, 7.98346e+09 bytes, in 600 seconds
37374.2 mean bytes/connection
356.015 fetches/sec, 1.33058e+07 bytes/sec
msecs/connect: 0.304634 mean, 11.828 max, 0.141 min
msecs/first-response: 27.0774 mean, 1067.21 max, 0.598 min
80899 bad byte counts
HTTP response codes:
  code 200 -- 141690
  code 400 -- 122
  code 500 -- 149
  code 501 -- 253
  code 502 -- 71395
```

这里只是说明比较关注的几个指标： 

- qps : 356.015 fetches/sec  
- rt : msecs/first-response 

**其他说明**

1. http_load在处理时会去关注每次访问同一个URL返回结果（即字节数）是否一致，若不一致就会抛出byte count wrong。 若被测服务返回就是有变化，可以忽略该错误，同时标准错误重定向。
2. http_load并不会统计机器信息，也不会产出中间结果，只会在压测完成之后，返回汇总信息。 如果我们想要压测某服务查看是否有内存溢出的情况的时候，除了使用tsar 来 查看系统信息，还可以在压测的同时，对服务进程所占的机器资源做监控， 用一个命令行实现 ：

```
op -d 60 -p 13648 -b  > aa

使用top名领，实时将某条进程对内存的占用 记录到文件中， -d为时间间隔， -p为进程编号
```


## ab
ab，全称是 apache benchmark.  也是apache服务器工具里面的压测工具。 
和http_load相比， ab除了支持http的get接口，**也同时支持 http的post接口**

#### 安装

apache的包中就会有这个压测的工具， 所以只需要安装apache即可 

```
yum install -b current httpd
```

#### 压测过程
ab的参数较多， 这里只说明比较常用的几个  

http get接口的压测。

```
/usr/bin/ab -h
Usage: ab [options] [http[s]://]hostname[:port]/path
Options are:
    -n requests     压测的请求条数
    -c concurrency  并发
    -t timelimit    最大压测时长， 设置了这个时间的时候，还有个默认的 -n 50000 ,哪个条件先达到性能测试就结束
```

可以看到和http_load相比， ab 无法将文件作为url压测文件，对于多条url的压测，推荐http_load.   


##### post接口

举例我现在有一个post接口 ： 

```
curl -H "Content-Type:application/json" -d '{ "key1":"value1", "key2":"value2"} http://union.m.taobao.org/convert
```
要想对其进行压测，命令为 ：

```
ab -n 10 -c 10 -T "application/json" -p postData.txt  http://union.m.taobao.org/convert

-n  requests 压测请求的条数
-c  并发  
-H   Content-Type
-p  post提交的数据 ， 比如说上面接口 { "key1":"value1", "key2":"value2"} ，多条请求的数据以换行分割
```


**结果说明**

这里直接引用网上的说明 ：

```
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests


Server Software:        Apache/2.2.25 (服务器软件名称及版本信息)
Server Hostname:        localhost (服务器主机名)
Server Port:            80 (服务器端口)
Document Path:          /index.php (供测试的URL路径)
Document Length:        10 bytes (供测试的URL返回的文档大小)
Concurrency Level:      100 (并发数)
Time taken for tests:   0.247 seconds (压力测试消耗的总时间)
Complete requests:      1000 (压力测试的总次数)
Failed requests:        0 (失败的请求数)
Write errors:           0 (网络连接写入错误数)
Total transferred:      198000 bytes (传输的总数据量)
HTML transferred:       10000 bytes (HTML文档的总数据量)
Requests per second:    4048.34 [#/sec] (mean) (平均每秒的请求数)
Time per request:       24.701 [ms] (mean) (所有并发用户(这里是100)都请求一次的平均时间)
Time per request:       0.247 [ms] (mean, across all concurrent requests) (单个用户请求一次的平均时间)
Transfer rate:          782.78 [Kbytes/sec] received (传输速率，单位：KB/s)
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.3      0       1
Processing:     6   23   4.2     24      30
Waiting:        5   20   5.3     21      29
Total:          6   23   4.2     24      30

Percentage of the requests served within a certain time (ms)
  50%     24
  66%     25
  75%     26
  80%     26
  90%     27
  95%     27
  98%     28
  99%     29
 100%     30 (longest request)

```

以上各种指标中，关注的比较多是 ： 

- qps :Requests per second:    4048.34 [#/sec] (mean) (平均每秒的请求数) 
- rt: Time per request:       0.247 [ms] (mean, across all concurrent requests) (单个用户请求一次的平均时间)






