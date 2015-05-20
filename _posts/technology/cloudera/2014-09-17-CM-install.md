---
layout: post
title:  "Cloudera Manager 安装详解"
date:   2014-09-17 21:29:02
categories: cloudera
---

Cloudera Manager（以下简称 CM）安装主要是针对 CM Server 及 CM Agent。其他诸如 Service Monitor、Host Monitor 等这些服务是在安装 CDH 的时候一并安装的

安装的方式主要有三种：

1. 通过 Cloudera 公司提供的 bin 文件来安装
   这种方式只能用来安装 CM Server，节点机器上的 Agent 只能再另外通过 Web 页面等其他方式来安装
   采用 bin 文件的安装方式本质上也是用 yum 来安装的，主要是会安装 CM Server、JDK、Deamons Tools、PostgreSQL，并且会自动帮忙配置好，这一点从 CM 的 yum 源就能看出来

2. 通过 yum 来安装
   这种方式对比第一种来说其实就是将其中的安装步骤拆分下来，并且可以弃用默认提供的 PostgreSQL 自己选择一个数据库，如果选择的是 MySQL，还需要再提供额外的 JDBC 库。JDK 也需要自己提供

3. 通过 tar 文件来离线安装
   其实就是将一个已有的 tar 包解压缩，修改下配置，然后起服务。对比上面两种方式的优点是：
    * 完全离线
    * 一切自己定制，包括 JDK、数据库、文件路径，由于 yum 方式安装最终的程序是放在 ROOT 分区下的，日志也是打在 ROOT 分区下，所以有将 ROOT 分区打满的危险

下面分别介绍 CM Server 及 CM Agent 安装的具体方法：

### CM Server
#### 一、bin 文件方式安装
通过 bin 文件方式安装，你可能需要通过配置一个本地 yum 源来提高安装的速度及可靠性，具体的方法可以参见：[如何为 Cloudera 建立本地 yum 源](/blog/2014/09/18/how-to-create-yum-source-for-cloudera/)

然后就是运行 bin 文件安装就行了，安装完成之后可以通过 7180 端口来访问服务，安装过程中出现的错误，可以通过查看弹窗上面给出的日志来进行排查

#### 二、yum 方式安装
通过 yum 方式安装，你可能需要通过配置一个本地 yum 源来提高安装的速度及可靠性，具体的方法可以参见：[如何为 Cloudera 建立本地 yum 源](/blog/2014/09/18/how-to-create-yum-source-for-cloudera/)

之后就可以通过 yum 命令来安装了，可以通过 yum search cloudera-manager 命令来列出所有提供的包

其中 Server 端需要安装包为：

   * cloudera-manager-server.x86_64   
   * cloudera-manager-daemons.x86_64

如果采用 Cloudera Manager 提供的 PostgreSQL 来作为数据库，也可以安装

   * cloudera-manager-server-db.x86_64   

这个包，当然你也可以自己弄一个 MySQL 来代替 PostgreSQL

安装之后可以用命令 service cloudera-scm-server start 来启动服务
可以通过查看 /var/log/cloudera-scm-server/cloudera-scm-server.log 日志来排查可能出现的问题
/var/run/cloudera-scm-server.pid 里面放的是 Server 进程的 pid
/etc/cloudera-scm-server/db.properties 里面是同数据库的连接配置，注意，如果数据库采用的是非默认端口，应该在 com.cloudera.cmf.db.host 配置项里面采用 hostname:port 的方式来配置数据库端口

如果是采用 MySQL 数据库，则还需要自己弄一个 JDBC 库，方法很简单，copy 一个 mysql-connector-java-X.X.XX.jar 包到 /usr/share/cmf/lib/ 路径下即可

#### 三、tar 方式安装
4.8 版本 Cloudera Manager 压缩包地址：
[http://archive.cloudera.com/cm4/cm/4/cloudera-manager-el6-cm4.8.1_x86_64.tar.gz](http://archive.cloudera.com/cm4/cm/4/cloudera-manager-el6-cm4.8.1_x86_64.tar.gz)

Down 下来之后解压到自己想要的目录，解压后得到的文件夹的目录结构如下：

```
cm-4.8.1-+-etc-+-cloudera-scm-agent
         |     |-cloudera-scm-server
         |     |-default
         |     |-init.d
         |     `security
         |-lib
         |-lib64
         |-log
         |-run
         |-sbin
         `-share
```

解压后的目录同系统的目录一一对应关系是：

```
etc     /etc
lib     /usr/lib
lib64   /usr/lib64
log     /var/log
run     /var/run
sbin    /usr/sbin
share   /usr/share
```

启动脚本就放在 etc/init.d 咯，如果将脚本考本到 /etc/init.d 就可以通过 service XXX start 的方式来启动

### CM Agent
#### 一、通过 CM Server 的 Web 界面安装
通过 Web 界面方式安装，你可能需要通过配置一个本地 yum 源来提高安装的速度及可靠性，具体的方法可以参见：[如何为 Cloudera 建立本地 yum 源](/blog/2014/09/18/how-to-create-yum-source-for-cloudera/)

如果是一个全新的系统，没有添加任何集群，则 "服务" --> "所有服务" --> "添加集群"
如果已有集群，则点击 "主机" 然后 "向群集添加新机器"

记得在 "选择存储库" 的时候选择 "自定义存储库" 然后添加本地 yum 库地址

需要注意的是，这种 Web 方式的安装也会同时将 CDH 给装上
安装 CDH 的步骤及注意事项参考：[CDH 安装手册](/blog/2014/09/18/CDH-install/)

#### 二、yum 方式安装
通过 yum 方式安装，你可能需要通过配置一个本地 yum 源来提高安装的速度及可靠性，具体的方法可以参见：[如何为 Cloudera 建立本地 yum 源](/blog/2014/09/18/how-to-create-yum-source-for-cloudera/)

之后就可以通过 yum 命令来安装了，可以通过 yum search cloudera-manager 命令来列出所有提供的包

其中 Agent 端需要安装的包为：

   * cloudera-manager-server.x86_64   
   * cloudera-manager-daemons.x86_64

安装好了之后，需要修改 /etc/cloudera-scm-agent/config.ini 配置文件来配置 Server 的地址
然后可以用命令 service cloudera-scm-agent start 来启动服务
/var/log/cloudera-scm-agent/cloudera-scm-agent.log 里面是日志
/var/run/cloudera-scm-agent.pid 里面放的是 agent 的 pid
/var/run/cloudera-scm-agent/process/ 里面是 Hadoop 的各个 component 的配置及启动日志

#### 三、tar 方式安装
Agent 的 tar 包同 Server 的是一样的

同样，也需要修改 ./etc/cloudera-scm-agent/config.ini 文件来设置 Server 服务器

./etc/init.d/cloudera-scm-agent start 启动服务

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

