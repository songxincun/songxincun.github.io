---
layout: post
title:  "关于解决 Hadoop 磁盘不均的两个重要配置"
date:   2015-06-11 14:35:52
categories: hadoop
---

### DataNode 卷选择策略
![DataNode 卷选择策略说明](/images/dfs.datanode.fsdataset.volume.choosing.policy.png)
这个配置主要是 DataNode 写数据的时候考虑往哪个磁盘的写的时候所采用的策略，有两种：
* 循环法
* 可用空间

循环法就是 DataNode 配置的目录无脑循环的写，如果由于各种原因多个目录配到一个磁盘的时候，就会出现磁盘使用不均，这个配置是默认的
可用空间就是 DataNode 会根据磁盘使用情况挑选要写的目录，不过这个配置需要另外一个配置进行配合：dfs.datanode.available-space-volume-choosing-policy.balanced-space-preference-fraction，这个配置具体说明如下

### 可用空间策略平衡的首选项
![可用空间策略平衡的首选项说明](/images/dfs.datanode.available-space-volume-choosing-policy.balanced-space-preference-fraction.png)
如图所示，这个配置主要是在卷选择策略选择可用空间的时候，配置新的块往非比较空的磁盘偏移的比例，一般配置为 0.5 -- 1，配置成 1 则新块都往空的磁盘写，就可以解决磁盘使用不均的问题了

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

