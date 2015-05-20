---
layout: post
title:  "CDH 安装手册"
date:   2014-09-17 21:35:02
categories: cloudera
---

目前我只尝试过通过 Cloudera Manager 提供的 Web 界面来安装 CDH

为了提高安装时的稳定性和速度，可能需要自己在本地搭建一个 CDH 的 parcels 源

搭建方法是：

1. 搭建一个 http 服务器
2. 将 [http://archive.cloudera.com/cdh4/parcels/4.7.0/](http://archive.cloudera.com/cdh4/parcels/4.7.0/) 下的 parcel 文件及 manifest.json 文件下下来，然后用 sha1sum 工具来为 parcel 文件生成 sha1 文件
3. 将所有的 parcel 及 sha1 及 manifest 文件扔到 http 服务器的某个目录下，则最终这个目录的 URL 地址即为安装 CDH 时的 parcel 地址

通过 Cloudera Manager 提供的 Web 界面来安装 CDH 非常简单，照着提示做就行

已经解决的可能出现的 case 有：

* [Cloudera TaskTracker 启动失败问题追查记录](/blog/2014/09/17/cloudera-tasktracker-start-error/)

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

