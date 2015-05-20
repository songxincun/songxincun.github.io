---
layout: post
title:  "Flume 遇到的问题及修改代码解决方案"
date:   2014-11-25 15:30:42
categories: flume 
---

### 背景介绍
我本来是打算采用 flume 来将线上切割好的日志传输到 HDFS 上面去，采用的就是 flume 提供的 spooldir source
配置如下：

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

实际应用过程中总是出现一个问题，就是老是动不动就抛出异常提示 "File has been modified since being read:"，然后 source 线程就退出了，结果数据就不传输了

我发现代码里面这块的实现逻辑是这样的：

```java
    File fileToRoll = new File(currentFile.get().getFile().getAbsolutePath());

    currentFile.get().getDeserializer().close();

    // Verify that spooling assumptions hold
    if (fileToRoll.lastModified() != currentFile.get().getLastModified()) {
        String message = "File has been modified since being read: " + fileToRoll;
        throw new IllegalStateException(message);
    }
    if (fileToRoll.length() != currentFile.get().getLength()) {
        String message = "File has changed size since being read: " + fileToRoll;
        throw new IllegalStateException(message);
    }
```

程序在判断了一次之后，如果没有符合要求了，就退出了。而我的现实要求，可能需要程序有几次重试，给上游切割程序一点准备的时间，因此我将这块的逻辑修改成这样：

```java
    File fileToRoll = new File(currentFile.get().getFile().getAbsolutePath());

    currentFile.get().getDeserializer().close();

    // Verify that spooling assumptions hold
    long prevModified  = currentFile.get().getLastModified();
    long prevLength    = currentFile.get().getLength();
    int  retryTimes    = 0;
    int  maxRetryTimes = 5;
    while (prevModified != fileToRoll.lastModified()) {
      if (retryTimes > maxRetryTimes) {
        String message = "File has been modified since being read: " + fileToRoll;
        throw new IllegalStateException(message);
      }   

      prevModified = fileToRoll.lastModified();
      prevLength   = fileToRoll.length();
      try{
        Thread.sleep((int)Math.pow(2, retryTimes)*5*1000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }

      retryTimes++;
    }

    if (prevLength != fileToRoll.length()) {
      String message = "File has changed size since being read: " + fileToRoll;
      throw new IllegalStateException(message);
    }
```

程序会重试5次，第一次间隔5秒，第二次10秒....

这样一来，上游就有了充分的时间准备了

然后接下来就是编译了，将 mvn 的源置为公司内部的源，然后执行：

```bash
mvn clean install -Dmaven.test.skip=true
```

后面启动 flume 的时候，提示说找不到库：

```
org.apache.flume.conf.file.AbstractFileConfigurationProvider$FileWatcherRunnable.run(AbstractFileConfigurationProvider.java:207)] Failed to start agent because dependencies were not found in classpath. Error follows.
java.lang.NoClassDefFoundError: org/apache/hadoop/io/SequenceFile$CompressionType
    at org.apache.flume.sink.hdfs.HDFSEventSink.configure(HDFSEventSink.java:214)
    at org.apache.flume.conf.Configurables.configure(Configurables.java:41)
    ... ...
```

后面弄了5个包到 flume 的 lib 下面就好使了：

   * hadoop-core-1.0.4.jar
   * commons-configuration-1.6.jar
   * commons-httpclient-3.0.1.jar
   * jets3t-0.6.1.jar
   * commons-codec-1.4.jar

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

