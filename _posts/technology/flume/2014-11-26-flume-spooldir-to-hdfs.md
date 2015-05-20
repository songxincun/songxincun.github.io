---
layout: post
title:  "利用 Flume 来将切割后的静态文件上传到 HDFS"
date:   2014-11-26 16:06:52
categories: flume 
---

### 配置

```bash
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = spooldir
a1.sources.r1.ignorePattern = ^$
a1.sources.r1.basenameHeader = true
a1.sources.r1.basenameHeaderKey = filename
a1.sources.r1.spoolDir = /home/work/pi/log/recsys_s_online/recsys_s_online_al/

a1.sources.r1.deserializer.maxLineLength = 40960

a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = static
a1.sources.r1.interceptors.i1.key = hostname
a1.sources.r1.interceptors.i1.value = nj02-rp-5yue-bu-recommend-personal117.nj02.baidu.com

# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.channel = c1
a1.sinks.k1.hdfs.path = hdfs://nj01-rp-arch-namenode00.nj01.baidu.com:54310/user/rp-rd/ouyanlin/recsys_s_online_al/%{hostname}
a1.sinks.k1.hdfs.fileType = DataStream
a1.sinks.k1.hdfs.filePrefix = %{filename}
a1.sinks.k1.hdfs.hdfs.writeFormat= Text
a1.sinks.k1.hdfs.inUsePrefix = .
#a1.sinks.k1.hdfs.filePrefix = events-
a1.sinks.k1.hdfs.rollInterval= 900
a1.sinks.k1.hdfs.rollSize= 0
a1.sinks.k1.hdfs.rollCount= 0
a1.sinks.k1.hdfs.round = true
a1.sinks.k1.hdfs.roundValue = 30
a1.sinks.k1.hdfs.roundUnit = minute
a1.sinks.k1.hdfs.useLocalTimeStamp = true

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

环境变量可以在 conf/flume-env.sh 配置文件里面配置

log 相关的配置可以在 conf/log4j.properties 配置文件里面配置

### flume 启动命令
bin/flume-ng agent --conf ./conf --conf-file ./conf/flume-conf.properties --name a1

之前遇到了程序动不动就报
> File has been modified since being read

然后退出，最终的解决方案参见：[/blog/2014/11/25/flume-problem/](Flume 遇到的问题及修改代码解决方案)

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

