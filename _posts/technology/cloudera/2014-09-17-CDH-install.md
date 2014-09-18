---
layout: post
title:  "CDH 安装手册"
date:   2014-09-17 21:35:02
categories: cloudera
---

Cloudera 的产品主要有两个：Cloudera Manager 和 CDH 

## Cloudera Manager
### 简介
用于管理基于 CDH 的 Hadoop 家族集群，提供了安装、监控、管理等操作
主要是分为 Server(Master) 和 Agent(Slave) 两部分
其中 Server 安装在中控机上，用于汇总数据及提供给用户 Web 界面
Agent 安装集群的各个节点上面，用于收集节点数据及执行 Server 给出的操作

### 安装
Cloudera Manager 安装方式可以有以下三种：

1. 通过 Cloudera 公司提供的 bin 文件来安装
   这种方式只能用来安装 Cloudera Manager Server，节点机器上的 Agent 只能再另外通过 Web 页面等其他方式来安装
   采用 bin 文件的安装方式本质上也是用 yum 来安装的，主要是会安装 CM-Server、JDK、Deamons-Tool、PostgreSQL，并且会自动帮忙配置好，这一点从 Cloudera Manager 的 yum 源就能看出来

2. 通过 yum 来安装
   这种方式对比第一种来说其实就是将其中的安装步骤拆分下来，并且可以弃用默认提供的 PostgreSQL 自己选择一个数据库，如果选择的是 MySQL，还需要再提供额外的 JDBC 库。JDK 也需要自己提供

3. 通过 tar 文件来离线安装
   其实就是将一个已有的 tar 包解压缩，修改下配置，然后起服务。对比上面两种方式的优点是：
        * 完全离线
        * 一切自己定制，包括 JDK、数据库、文件路径，由于 yum 方式安装最终的程序是放在 ROOT 分区下的，日志也是打在 ROOT 分区下，所以有将 ROOT 分区打满的危险

下面介绍各自方式的具体安装步骤：

#### 一、bin 文件方式安装
通过 bin 文件方式安装，你可能需要通过配置一个本地 yum 源来提高安装的速度及可靠性，具体的方法可以参见：[如何为 Cloudera 建立本地 yum 源](http://localhost:4000/cloudera/yum/2014/09/18/how-to-create-yum-source-for-cloudera/)

然后就是运行 bin 文件安装就行了，安装完成之后可以通过 7180 端口来访问服务

#### 二、yum 方式安装
通过 yum 方式安装，你可能需要通过配置一个本地 yum 源来提高安装的速度及可靠性，具体的方法可以参见：[如何为 Cloudera 建立本地 yum 源](http://localhost:4000/cloudera/yum/2014/09/18/how-to-create-yum-source-for-cloudera/)

之后就可以通过 yum 命令来安装了，可以通过`yum search cloudera-manager`命令来列出所有提供的包

其中 Server 端需要安装包为：

   * cloudera-manager-server.x86_64   

如果采用 Cloudera Manager 提供的 PostgreSQL 来作为数据库，也可以安装

   * cloudera-manager-server-db.x86_64   

这个包，当然你也可以自己弄一个 MySQL 来代替 PostgreSQL
   

其中，通过 bin 文件方式安装其实也是需要利用 yum 来进行安装的，可以通过搭建本地 yum 源的方式来提高安装的速度和稳定性，具体方法参见：[如何为 Cloudera 建立本地 yum 源](http://localhost:4000/cloudera/yum/2014/09/18/how-to-create-yum-source-for-cloudera/)

通过 bin 文件的安装方式和 yum 的安装方式的区别在于


## CDH
CDH 包含了一套 Hadoop 家族的各个软件，包括：Hadoop, Hive, HBase, Zookeeper, Pig, Oozie 等等。
整个包比较大，700多M

Cloudera Manager 5.1.1
[下载 Cloudera Manager 5.1.1](http://www.cloudera.com/content/support/en/downloads/cloudera_manager/cm-5-1-1.html)

得到一个名为：cloudera-manager-installer.bin 的文件
run it!
安装文件会分别在机器上安装包括 Web Server、JDK、PostSql 等工具
然后访问主机的 7180 端口就能访问 cloudera manager 了
用户名和密码都是 admin

通过 cloudera 来安装 Hadoop 家族的工具

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

