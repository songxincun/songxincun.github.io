---
layout: post
title:  "HDFS 各种操作失败原因及解决方案"
date:   2014-09-16 18:32:52
categories: hadoop
---

### 安装 HDFS 时，Format 失败
多半是由于之前已经装过一次集群，因而 dfs.name.dir 文件夹非空导致的
如果确认之前的集群在 dfs.name.dir 下遗留的元数据已经不再需要了的话，可以直接把这个文件夹清空了，然后再次 Format HDFS

### 启动 HDFS、DataNode 或 SecondaryNameNode 时失败，日志里面提示说 VERSION 不一致
每个集群在 Format 之后都会有



[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

