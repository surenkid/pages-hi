---
layout: post
title: Apache Benchmark(AB)Web网站性能压力测试工具简单使用
date: 2017-10-19 00:39:00
category: 软件应用
permalink: /672.html
tags:
- apache
- web
- test
---

<!--markdown-->网站上线前往往需要做一些压力测试，防止高并发场景时服务器挂掉。压力测试工具有不少，但是最简单的应该算是Apache附带的Apache Benchmark（下面简称AB）了，这里我就对AB的安装使用以及分析做一个简单的介绍。

![Apache Benchmark][1]

# 安装Apache Benchmark(AB)

## Windows安装

其中Windows下安装ab比较简单，直接去[Apache官网][2]找对应的编译好的Windows包，解压后找到ab.exe，就是这个工具了，

下载地址：https://www.apachehaus.com/cgi-bin/download.plx

## Linux安装

接着我们看看Linux下安装，一般的教程都是需要安装httpd这个包，包含了apche服务器以及相关的工具，这里由于我们只需要用到ab这个附带的测试工具，因此只需要单独安装工具包就可以了

CentOS下Yum直接安装：

    yum install httpd-tools 

Ubuntu下也差不多：

    apt-get install apache2-utils

# 命令参数解释

安装好之后直接使用如下命令即可简单测试：

    ab http://hi.ktsee.com/

当然每个测试的场景不同，这些必要的参数还是需要设置的：

- `-n`参数用来设置模拟请求的总次数
- `-c`参数用来设置模拟请求的并发数
- `-t`参数用来设置模拟请求的时间

简单举例，如果要模拟1个用户访问1000次：

    ab -n 1000 http://hi.ktsee.com/

如果要模拟500个用户访问1000次（表示每个用户访问2次）

    ab -c 500 -n 1000 http://hi.ktsee.com/

如果要模拟500个用户同时访问，并且每个用户访问停留时间10秒：

    ab -c 500 -t 10 http://hi.ktsee.com/

以上是常用的参数，其他参数如下，可以根据实际测试场景使用：


- `-A`	采用base64编码向服务器提供身份验证信息，用法: -A 用户名:密码
- `-C`	cookie信息，用法: -C key=value
- `-d`	不显示pecentiles served table
- `-e`	保存基准测试结果为csv格式的文件
- `-g`	保存基准测试结果为gunplot或TSV格式的文件
- `-h`	显示ab可选参数列表
- `-H`	采用字段值的方式发送头信息和请求
- `-i`	发送HEAD请求，默认发送GET请求
- `-k`	设置ab命令允许1个http会话响应多个请求
- `-p`	通过POST发送数据，用法： -p page=1&key=value
- `-P`	采用base64编码向服务器提供身份验证信息，用法: -A 用户名:密码
- `-q`	执行多余100个请求时隐藏掉进度输出
- `-s`	使用Https协议发送请求，默认使用Http
- `-S`	隐藏中位数和标准偏差值
- `-v`	-v 2 及以上将打印警告和信息，-v 3 打印http响应码，-v 4 打印头信息
- `-V`	显示ab工具的版本号
- `-w`	采用HTML表格打印结果
- `-x`	HTML标签属性，使用 -w 参数时，将放置在`<table>`标签中
- `-X`	设置代理服务器，用法 -X 192.168.1.1:80
- `-y`	HTML标签属性，使用 -w 参数时，将放置在`<tr>`标签中
- `-z`	HTML标签属性，使用 -w 参数时，将放置在`<td>`标签中

# 测试用例与输出结果分析

使用上面的第二个例子，得到的结果如下：

    This is ApacheBench, Version 2.3 <$Revision: 655654 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/
    
    Benchmarking hi.ktsee.com (be patient)
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
    
    
    Server Software:        
    Server Hostname:        hi.ktsee.com                        
    Server Port:            80                                    
    
    Document Path:          /
    Document Length:        2208 bytes
    
    // 以上是你测试的host, port等一部分的信息
    
    Concurrency Level:      500
    // 这个标志了这1000个请求完成了总共消耗了这些时间
    Time taken for tests:   10.117 seconds 
    Complete requests:      1000
    // 这1000个请求中失败的次数,如果又失败,它也有输出非2XX的失败
    // 如果测试中出现了错误,最好看下这些错误是不是你所期望的
    Failed requests:        320
    Write errors:           0
    // 这1000个请求总共传输了这些数据
    Total transferred:      587848 bytes
    HTML transferred:       435294 bytes
    // 每秒执行了334个请求,这个可以看到你服务器每秒能够处理多少个这样的url请求
    // 是一个很重要的性能指标
    Requests per second:    98.85 [#/sec] (mean)
    // 第一个时间是以你的并发数为一组,这里我是设置了并发100,那么
    // 就是这一组请求所能消耗的总时间
    Time per request:       5058.355 [ms] (mean)
    // 这个就是平均下来每个请求的消耗的时间了
    Time per request:       10.117 [ms] (mean, across all concurrent requests)
    // 测试的时候通过网络的传输的速率
    Transfer rate:          56.74 [Kbytes/sec] received
    
    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:      132  613 1214.6    148    8311
    Processing:   132 1638 2024.0    704    8337
    Waiting:      132 1417 1927.7    558    6691
    Total:        264 2251 2314.2   1269    8472
    // 以上这段数据标志了一个请求从连接,发送数据,接收数据这个三个大致的时间,最小以及平均值
    
    
    Percentage of the requests served within a certain time (ms)
      50%   1269
      66%   2385
      75%   3354
      80%   4192
      90%   6475
      95%   7041
      98%   7930
      99%   8355
     100%   8472 (longest request)
    
    // 以上的数据标志了所有的api请求的消耗时间分布的区间
    // 可以看到, 50%的请求是在1269MS以下, 66%的请求实在2385MS以下
    // 这个数据其实可以看出很多问题来
    //       如果这个数据很平均,即第二列的数据的值几乎都差不多,其实可以说明,至少
    //       你的服务器在这个并发的情况下对于各个请求所能提供的能力是均衡的
    //       在面对这种压力的并发的情况下,对资源没有明显的使用短缺
    //       如果这个数据的差距非常大,这个情况就要结合自身的应用分析了
    //       就像现在这种情况,可以发现,明显有一部分请求在很久之后才得到响应
    //       说明,在并发的情况下,CPU或者IO或者其它资源没有被均衡的使用


  [1]: https://static.ktsee.com/s1/2017/10/ab.png
  [2]: http://httpd.apache.org/