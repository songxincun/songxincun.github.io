---
layout: post
title:  "Hadoop 运维及使用过程中遇到的各种 case"
date:   2014-11-24 10:33:52
categories: hadoop
---

### 单台 datanode 磁盘打满，导致集群不可用

单台 datanode 打满后，HDFS 写失败，进而导致上层 RegionServer 夯住，即使过一会 datanode 磁盘缓过来了，RS 依旧卡住。但是心跳依旧存在，zookeeper 能检测到 RS 的状态

后来发现时 hdfs 的配置有问题
dfs.replication.min 默认是1，但是线上配置的是3。这也就意味着，原本写成功一个副本即为写成功，现在需要写成功3个才算。
另外一个配置：dfs.replication 则表示默认的副本数（默认为3）


[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

