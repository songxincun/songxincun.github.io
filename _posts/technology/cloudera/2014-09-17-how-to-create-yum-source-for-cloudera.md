---
layout: post
title:  "如何为 Cloudera 建立本地 yum 源"
date:   2014-09-17 17:43:02
categories: cloudera yum
---

通过 yum 或者 bin 的方式来安装 Cloudera Manager 的时候，由于默认的官方源在国外，网络因素等很不稳定，因此最好是自己在内网搭建一个自己的 Cloudera yum 源

搭建 yum 源的方式有很多，本文只介绍其中一种好使的

1. 首先搭建一个 Web 服务器，过程略
2. 从 Cloudera 官网上下载 rpm 包，地址：[http://archive.cloudera.com/cm4/redhat/6/x86_64/cm/4/](http://archive.cloudera.com/cm4/redhat/6/x86_64/cm/4/)
   需要下载的内容包括全部的 RPMS 和 repodata 文件夹，也可以用下面的命令来生成一个 repodata

    ```
    createrepo -d -p ./RPMS/ -o ./
    ```
   其中 createrepo 工具可能需要额外安装
3. 将下载好的 RPMS 和 repodata 目录放置到 Web 服务器的某个访问目录下面，例如：cm
   则访问 URL: http://XXX.XXX.XXX/cm 可以看到 RPMS 和 repodata 两个目录
4. 构造 repo 文件，一个典型的例子如下：

    ```
    [cloudera-manager]
    name = Cloudera Manager, Version 4
    baseurl = http://XXX.XXX.XXX/cm/
    enabled = 1
    gpgcheck = 0
    ```
   将该文件放置到需要通过 yum 来安装 Cloudera Manager 的机器的 /etc/yum.repos.d/ 目录下面，然后执行`yum clean all && yum update`
   之后就可以通过 yum 来安装 Cloudera Manager 的各个 component 了

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

