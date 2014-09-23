---
layout: post
title:  "HBase 安装记录"
date:   2014-09-23 17:31:12
categories: hbase
---

HBase 组件主要有：zookeeper、hadoop（HDFS + MapRed）、hbase
zookeeper 和 hadoop 是完全独立的（对于 hadoop 1.0 而言），可以独立安装及配置，详见：

   * [Zookeeper 安装记录](http://songxincun.github.io/zookeeper/2014/09/23/zookeeper-install-how-to/)
   * [Hadoop 安装记录](http://songxincun.github.io/hadoop/2014/09/23/hadoop-install-how-to/)

hbase 的配置中则需要依赖 zookeeper 和 hadoop

## 安装前准备
你可能需要为 hbase 创建一个单独的用户，用下面这个命令

```
useradd -d /home/hbase     -m hbase
```

## HBase 安装
1.  hbase-env.sh
    主要是设置以下环境变量

    ```bash
    export JAVA_HOME=/home/work/jdk1.7
    export HBASE_MANAGES_ZK=false
    export HBASE_PID_DIR=/tmp/pids
    ```

2.  backup-masters
    里面配置备用的 master

3.  hbase-site.xml
    
    ```xml
    <property>
        <name>hbase.master</name>
        <value>nj02-rp-2014-q1-01-jh2-211.nj02.baidu.com:60000</value>
    </property>

    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://nj02-rp-2014-q1-01-jh2-210.nj02.baidu.com:54310/hbase</value>
    </property>

    <property>
        <name>dfs.support.append</name>
        <value>true</value>
    </property>

    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>

    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>nj02-rp-2014-q1-01-jh2-210.nj02.baidu.com,nj02-rp-2014-q1-01-jh2-211.nj02.baidu.com,nj02-rp-2014-q1-01-jh2-212.nj02.baidu.com</value>
    </property>
    ```

4.  启动 HBase

    ```bash
    # 启动 master
    bin/hbase-daemon.sh start master

    # 启动 RegionServer
    bin/hbase-daemon.sh start regionserver
    ```

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

