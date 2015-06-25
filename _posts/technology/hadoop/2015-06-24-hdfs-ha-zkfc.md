---
layout: post
title:  "HDFS HA 控制 zkfc 判断 NameNode 响应超时的两个重要的参数"
date:   2015-06-24 20:35:52
categories: hadoop
---

### ZKFC: Zookeeper failover controller
HDFS HA 切换有点频繁及莫名其妙，实际上去看，NameNode 并没有异常

ha.failover-controller.graceful-fence.rpc-timeout.ms
ha.zookeeper.session-timeout.ms

这两个参数用于控制 ZKFC 的超时判断，默认好像是 5s，调大以避免 HA 过于敏感


[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

