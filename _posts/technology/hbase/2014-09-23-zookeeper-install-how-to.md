---
layout: post
title:  "Zookeeper 安装记录"
date:   2014-09-23 17:31:12
categories: zookeeper
---

## 安装前准备
你可能需要为 zookeeper 创建一个对应的用户，用下面这个命令

```
useradd -d /home/zookeeper -m zookeeper
```

## zookeeper 安装
1.  修改 conf/zoo.cfg 配置文件，

    ```bash
    # The number of milliseconds of each tick
    tickTime=2000
    # The number of ticks that the initial 
    # synchronization phase can take
    initLimit=10
    # The number of ticks that can pass between 
    # sending a request and getting an acknowledgement
    syncLimit=5
    # the directory where the snapshot is stored.
    # do not use /tmp for storage, /tmp here is just 
    # example sakes.
    dataDir=/home/zookeeper/data
    # the port at which the clients will connect
    clientPort=2181
    #
    # Be sure to read the maintenance section of the 
    # administrator guide before turning on autopurge.
    #
    # http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
    #
    # The number of snapshots to retain in dataDir
    #autopurge.snapRetainCount=3
    # Purge task interval in hours
    # Set to "0" to disable auto purge feature
    #autopurge.purgeInterval=1
    server.1=nj02-rp-2014-q1-01-jh2-210.nj02:2888:3888
    server.2=nj02-rp-2014-q1-01-jh2-211.nj02:2888:3888
    server.3=nj02-rp-2014-q1-01-jh2-212.nj02:2888:3888
    ```
    dataDir 所配置的路径确保 zookeeper 用户有权限
    server.1、server.2、server.3 分别对应三个节点及其对应的 id

2.  修改 bin/zkEnv.sh 文件
    主要是配置 Java 路径

    ```bash
    if [ "$JAVA_HOME" != "" ]; then
        JAVA="$JAVA_HOME/bin/java"
    else
        JAVA=/home/work/jdk1.7/bin/java
    fi
    ```

3.  设置 zookeeper 节点的 id
    在 dataDir 所设置的目录下，设置一个名为 myid 文件内容为对应的

    ```bash
    server.1=nj02-rp-2014-q1-01-jh2-210.nj02:2888:3888
    server.2=nj02-rp-2014-q1-01-jh2-211.nj02:2888:3888
    server.3=nj02-rp-2014-q1-01-jh2-212.nj02:2888:3888
    ```
    中，server.X 的 X

4.  起服务
    
    ```bash
    bin/zkServer.sh start
    ```
    查看 bin/zookeeper.out 文件排查错误

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

