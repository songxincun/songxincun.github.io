---
layout: post
title:  "Cloudera 版本 hadoop 安装手册"
date:   2015-07-18 16:57:02
categories: cloudera
---

之前虽然写过关于 Cloudera 安装的手册，但是比较简陋。随着工作经验的积累，对整个 Cloudera 生态的理解也更加深刻，因此重新总结一下 Cloudera 版本的 hadoop 体系组织结构及安装方法
本文只介绍了最中规中矩的先安装 Cloudera Manager 再通过 Cloudera Manager 提供的 Web 页面安装 hadoop 集群的方式。其他的开挂的比如直接通过 yum 或者干脆手动安装 CDH 的方式没有介绍，但本质上是一样的，后者的可定制性可能会强一点

先来熟悉几个概念：
#### Cloudera Manager
![Cloudera Manager 图示](/images/cloudera-manager-example.png)
Cloudera Server 提供的一个管理集群的 Web 系统，可以通过该 Web 页面来查看、操作集群

#### Cloudera Server
主控节点，对用户提供 Cloudera Manager Web 系统来管理集群。同时负责向 Cloudera Agent 发送操作命令和收集 Cloudera Agent 采集的集群各个节点的信息

#### Cloudera Server DB
配合 Cloudera Server 工作的数据库，默认 Cloudera 自带安装的是 PostgreSql，也可以自己提供一个

#### Cloudera Agent
安装在集群各个节点上面，负责监视节点上的进程，采集主机及进程信息，汇报给 Cloudera Server，执行 Cloudera Server 发过来的操作命令

#### CDH
Cloudera's Distribution Including Apache Hadoop
可以理解为一个 hadoop 的发行版
比如说 CDH 5.3.1 里面就打包了：
     hadoop-2.5.0
     hbase-0.98.6
     hive-0.13.1
等等，具体的某个发行版里面所包含的各个配件组成及其版本对应，可以参考[这里](http://archive.cloudera.com/cdh5/cdh/5/)

#### parcel
我觉得可以理解为一种包管理工具，类似于 RPM
比如说 CDH 5.3.1 对应的 parcel 包名就是：CDH-5.3.1-1.cdh5.3.1.p0.5-el6.parcel
官方下载地址：http://archive.cloudera.com/cdh5/parcels/
下下来，解压缩（tar xzvf），里面就是完整的对应 CDH 发行版的各个组件了
其实你也可以绕过前面那些 Cloudera 的各种管理工具，自己下一个完整的某个发行版本 CDH 对应的 parcel 包，然后安装到各个节点上面，配置集群，安装方式和裸 Apache hadoop 的安装方式类似

Cloudera 版本的 hadoop 安装大致可以分为以下几步：

1. 在某台中控机上安装 Cloudera Server + Cloudera Server DB，完成一个 Web 系统的搭建
2. 在所有的节点上面安装 Cloudera Agent
3. 在所有节点上面下载安装指定 CDH 版本的 parcel 包（通过 Cloudera Manager 的 web 页面操作）
4. 为节点分配角色，拉起服务，完成集群安装

### 安装前的注意事项：
#### yum 源
Cloudera Server、Cloudera Server DB、Cloudera Agent 都是通过 yum 方式来安装的，默认的源是 cloudera 官网的源
如果你集群中的机器无法上外网，或者速度较慢（连得是国外的 cloudera 官网），那么你可能需要自己搭建一个私有的 yum 源
搭建的方法很简单：

1. 搭建一个 web 服务器，过程略
2. 从 Cloudera 官网上下载 Cloudera Manager 对应版本的 rpm 包：[http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/](http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/)
   选择某个版本，将文件夹里的内容都 down 下来
   推荐命令：```wget -r -L -np http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.4/```
3. 将 down 下来的文件夹放置到 web 服务器的某个访问路径下比如：http://XXX.XXX.XXX/5.3
4. 构造 yum 的 repo 文件，一个典型的例子如下：

```
[cloudera-manager]
name = Cloudera Manager, Version 5
baseurl = http://XXX.XXX.XXX/5.3/
enabled = 1
gpgcheck = 0
```
5. 将构造好的 repo 文件放置到 Cloudera Server 机器上的 /etc/yum.repos.d/ 目录下，如果该目录下有其他类似 Cloudera Manager 的 repo 文件，你可能需要将他们请空
6. 执行`yum clean all && yum update`，然后执行 `yum search cloudera`，看有没有正确的 search 结果

#### parcel 源
CDH 的 parcel 包是 Cloudera Server 下载一份到本地缓存后，再分发到集群各个节点的。由于该包较大，目测 1.5G 以上，所以建议同 yum 一道，自己提前弄个私有源，方法也基本类似：

1. 弄个 web server
2. 去官网 copy 某个版本的目录：http://archive.cloudera.com/cdh5/parcels/ 到本地 web server
   推荐命令：```wget -r -L -np http://archive.cloudera.com/cdh5/parcels/5.4/```


同 yum 不一样，parcel 不需要构造 repo 文件，而是在后面通过 Cloudera Manager web 页面安装的时候填写本地 parcel url 地址

下面就具体介绍各个模块的安装步骤

### Clouder Manager（Cloudera Server + Cloudera Server DB）安装
用例版本：Cloudera Manager 5.3
要求：
* CentOS 6.3
* gcc 4.4.6

首先去 Cloudera 官网上面将 cloudera-manager-installer.bin 文件下下来：[http://archive.cloudera.com/cm5/installer/](http://archive.cloudera.com/cm5/installer/)
安装前要注意的是，安装的 Cloudera Server DB 默认数据库的目录是放到 /var/lib/cloudera-scm-server-db 下面，这样很容易就把 ROOT 分区打满了，由于还没有找到配置这个目录的地方，所以就要提前建一个软链来将这个路径链到大分区磁盘
按照前面介绍的设置好 yum 源之后，直接 chmod a+x cloudera-manager-installer.bin  && ./cloudera-manager-installer.bin 
然后各种 yes and 下一步，中间出错可以按照提示看日志去排查，常见的错误参见文档后面的 FAQ
安装完毕后可以通过浏览器访问 Cloudera Server 的 7180 端口来打开我们的 Cloudera Manager web 页面
由于 Baidu 办公网限制只能访问 8000 ~ 9000 端口，因此需要设置反向代理来访问这个 7180 端口

### Cloudera Agent 安装
如果是首次安装 Cloudera 集群，在安装完成 Cloudera Manager 之后，打开 Cloudera Manager 网页则会提示：
![Cloudera 选择许可证](/images/cloudera-manager-edition.png)
选择许可证然后下一步

如果是从 Cloudera 主界面进入的则可以点击导航栏上面的 "主机" -> "向集群添加新主机"
进入下面这个页面：
![Cloudera 查找主机](/images/cloudera-manager-search-hosts.png)

搜索主机然后继续下一步
到了选择存储库的页面，选择 parcel 的时候，点击更多选项
![选择 parcel](/images/cloudera-manager-choice-parcel.png)
将 "本地 Parcel 存储库路径" 和 "Parcel 目录" 改到大一点的分区，避免将 ROOT 分区打满
将前面设置的本地私有 Parcel 库的路径添加到 "远程 Parcel 存储库 URL"
点击确定后会过一会会在页面多出一个这个：
![CDH 版本选择](/images/cloudera-manager-found-cdh.png)
表面 Cloudera Manager 已经找到你本地的 CDH 了
然后是设置 Cloudera Agent 的下载地址：
![Cloudera Agent 地址](/images/cloudera-manager-agent-setting.png)
其中，"自定义存储库" 就填前面 yum 源的地址（同 Cloudera Manager 安装时候配的 yum 地址）
之后的步骤都比较简单了，一路按照提示弄就行。安装过程中遇到任何问题，可以参考文档后面的 FAQ
等到下面这个图出现，就意味着 Cloudera Agent 安装完毕了：
![Cloudera Agent 安装完毕](/images/cloudera-manager-agent-installed.png)
这时候主机就已经是受控了，去到 Cloudera Manager 主页面导航栏 -> "主机" 就能看到这些机器了，不过都没有安装 CDH 也没有分配集群

### CDH 安装（安装选定的 Parcel）
如果是接着前面的 "Cloudera Agent 安装" 继续的话，接下来就到了安装 Parcel 的阶段了，安装 Parcel 共分为 3 步：下载、分配、激活
让 Cloudera 自动完成就好了

### 分配角色拉起服务
默认 Cloudera 会自动为所选择的服务分配角色，如果只是为了体验，基本也就够了，但是显然无法达到最优
而本身分布式系统各个角色如何分配就是很复杂的一个事情，这里也就不单独讲了

### FAQ
* [Cloudera TaskTracker 启动失败问题追查记录](/blog/2014/09/17/cloudera-tasktracker-start-error/)

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

