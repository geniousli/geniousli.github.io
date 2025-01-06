---
title: abtest
date: 2016-09-07
tags: test
---

## 参考
[性能测试之ab测试工具](http://www.makmong.com/1314.html)

[性能测试之ab测试工具-mht](ab_test.mht)
### ab测试参数详解：
```
-n 测试会话中所执行的请求个数,默认仅执行一个请求
-c 一次产生的请求个数,即同一时间发出多少个请求,默认为一次一个
-t 测试所进行的最大秒数,默认为无时间限制....其内部隐含值是[-n 50000],它可以使对服务器的测试限制在一个固定的总时间以内
-p 包含了需要POST的数据的文件
-T POST数据所使用的Content-type头信息
-v 设置显示信息的详细程度
-w 以HTML表格的形式输出结果,默认是白色背景的两列宽度的一张表
-i 以HTML表格的形式输出结果,默认是白色背景的两列宽度的一张表
-x 设置<table>属性的字符串,此属性被填入<table 这里>
-y 设置<tr>属性的字符串
-z 设置<td>属性的字符串
-C 对请求附加一个Cookie行，其典型形式是name=value的参数对,此参数可以重复
-H 对请求附加额外的头信息,此参数的典型形式是一个有效的头信息行,其中包含了以冒号分隔的字段和值的对(如"Accept-Encoding: zip/zop;8bit")
-A HTTP验证,用冒号:分隔传递用户名及密码
-P 无论服务器是否需要(即是否发送了401认证需求代码),此字符串都会被发送
-X 对请求使用代理服务器
-V 显示版本号并退出
-k 启用HTTP KeepAlive功能,即在一个HTTP会话中执行多个请求,默认为不启用KeepAlive功能
-d 不显示"percentage served within XX [ms] table"的消息(为以前的版本提供支持)
-S 不显示中值和标准背离值,且均值和中值为标准背离值的1到2倍时,也不显示警告或出错信息,默认会显示最小值/均值/最大值等(为以前的版本提供支持)
-g 把所有测试结果写入一个'gnuplot'或者TSV(以Tab分隔的)文件
-e 产生一个以逗号分隔的(CSV)文件,其中包含了处理每个相应百分比的请求所需要(从1%到100%)的相应百分比的(以微妙为单位)时间
-h 显示使用方法
-k 发送keep-alive指令到服务器端
```
### ab测试解读
```
[root@web ~]# ab -n 100 -c 10 -k http://192.168.18.41/forum.php
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/
Benchmarking 192.168.18.41 (be patient).....done
Server Software:        Apache/2.4.4 #被测试Web服务软件名称
Server Hostname:        192.168.18.41 #服务器主机名,即请求的URL中的主机部分名称
Server Port:            80 #被测试Web服务器软件的监听端口
Document Path:          /forum.php #请求URL的绝问文件路径,即请求的资源
Document Length:        12428 bytes #请求文件的大小
Concurrency Level:      10 #并发数(-c属性来设置)
Time taken for tests:   1.014 seconds #执行完所有的请求所花费的时间,即整个测试持续的时间
Complete requests:      100 #完成的请求数量
Failed requests:        96 #失败的请求数量
   (Connect: 0, Receive: 0, Length: 96, Exceptions: 0)
Write errors:           0
Keep-Alive requests:    0
Total transferred:      1243660 bytes #整个场景中的网络传输量,即所有请求的响应数据的总和,包含头信息和正文长度
HTML transferred:       1180016 bytes #整个场景中的HTML内容传输量,即所有请求中响应数据的正文长度,不包含头信息的长度
Requests per second:    98.61 [#/sec] (mean) #吞吐率：即每秒处理的请求数,相当于LR中的每秒事务数,括号中的mean表示这是一个平均值,其值为Complete requests/Time taken for tests
Time per request:       101.407 [ms] (mean) #服务器平均请求处理的时间,即每个请求实际运行时间的平均值,其值为Time per request/Concurrency Level
Time per request:       10.141 [ms] (mean, across all concurrent requests) #平均每秒网络上的流量,即这些请求在单位内从服务器获取的数据长度,其值为(Total transferred/Time taken for tests)/1024
Transfer rate:          1197.65 [Kbytes/sec] received  #这个统计选项可以很好的说明服务器在处理能力达到极限时其出口带宽的需求量,可以帮助排除是否存在网络流量过大导致响应时间延长的问题
Connection Times (ms) #网络上消耗的时间的分解
              min  mean[+/-sd] median   max
Connect:        0    1   0.7      1       4
Processing:    24   99  93.1     68     482
Waiting:       19   92  92.1     59     475
Total:         25  100  93.3     69     482
#整个场景中所有请求的响应情况,在场景中每个请求都有一个响应时间
#下面结果表明,50％的用户响应时间(即请求处理时间,这里处理时间是指Time per request)小于69毫秒,66％的用户响应时间小于97毫秒,而最大的响应时间小于1268毫秒
#对于并发请求,实际上CPU并不是同时处理的,而是按照每个请求获得的时间片而逐个轮转处理的,所以基本上第一个Time per request时间约等于第二个Time per request时间乘以并发请求数
Percentage of the requests served within a certain time (ms)
  50%     69
  66%     97
  75%    117
  80%    123
  90%    224
  95%    330
  98%    452
  99%    482
 100%    482 (longest request)
```
