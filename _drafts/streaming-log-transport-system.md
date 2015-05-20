---
layout: post
title:  "基于 Flume + HBase + Impala 的实时流式日志收集系统介绍"
date:   2014-11-25 15:14:42
categories: flume hbase impala hue
---

### 背景介绍
国际化需要弄一个日志实时分析的系统，包括运营数据统计，监控等等
所以首先就需要一个实时的日志收集系统，来实时的将线上集群的数据实时的传输到集群上去

### 解决方案
底层的 DB 存储采用 HBase 来实现，采用 HBase 的优势在于存储的格式比较灵活，可以存储一些统计信息

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

