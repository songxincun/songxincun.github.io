---
layout: post
title:  "HBase 安装注意事项"
date:   2014-09-11 12:18:22
categories: hbase
---

### 硬件

#### 一、CPU
无

#### 二、内存
据说一个 Java 的进程不应该分配过多的内存（堆），否则内存碎片过多引发堆重写的时候，会引发灾难

#### 三、磁盘
应该保证一个磁盘上面至少一个 CPU 核


### 软件

#### 一、文件系统
在挂载文件系统时应该设置 noatime 属性来禁止记录文件访问时间戳以减少内核的管理开销。HBase 不需要这个功能，禁用这个据说可以大幅提高磁盘的读取性能

```
设置 noatime 选项：编辑 /etc/fstab 文件
/dev/sdd1    /data    ext3    defaults,noatime  0  0
```

`tune2fs -m 1 <device-name>`可以减少磁盘的保留块的数目，从而提高磁盘利用率

#### 二、Hadoop
HDFS 需要支持 sync 功能，否则 HBase 可能在灾难性场景中丢失数据。sync 特性是干什么用的？支持 append 么？

#### 三、域名服务
检查 /etc/hosts 文件，看环回 localhost 配置是否正确

hbase.regionserver.dns.interface 配置项可以配置？

hbase.regionserver.dns.nameserver 配置项可以配置与系统默认配置不同的域名服务器地址

#### 四、同步时间
集群节点的时间不同步可能会产生一些奇怪的结果

#### 五、文件句柄和进程限制
Too many open files

`lsof -p <pid-of-regionserver>`可以看到当前的 regionserver 所占用的文件描述符数

HBase 会在第一行日志打印 ulimit 信息

修改文件最大描述符上限方法：

```
修改文件：/etc/security/limits.conf
添加下面一行
hadoop  -    nofile  655360

然后就是 nproc 的值，how？

修改文件：/etc/pam.d/common-session
添加下面一行
session required pam_limits.so

重新登录，配置生效
```

#### 六、DataNode 处理线程数
HDFS 的 DataNode 会设置服务时可处理的文件数上限，这个参数叫做 xcievers

```xml
<property>
  <name>dfs.datanode.max.xcievers</name>
  <value>4096</value>
</property>
```

不配置这个选项可能会导致一些奇怪的问题。用户最终会在数据节点的日志中发现 xcievers 使用超限的异常，这个异常是以块丢失的异常抛出来的

```
10/12/08 20:10:31 INFO hdfs.DFSClient: Cloud not obtain block
    blk_XXXXXXXXXXXXXX_YYYYYYY from any node: java.io.IOException:
    No live nodes contain current block. Will get new block locations from
    namenode and retry...
```

#### 七、交换区
关掉 swap 分区

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

