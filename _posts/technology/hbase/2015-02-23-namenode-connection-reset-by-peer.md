---
layout: post
title:  "DataNode 异常：Connection reset by peer"
date:   2015-02-23 21:25:32
categories: hadoop
---

> 这个参数之前很早就明确过了，设为1时，连接繁忙时容易找出 reset 的情况，不适合 master 的应用场景
> 期望是，所有的 master 节点配置为0，比如 NameSpace、FMS、Adapter 等
>
> 刚刚发现 ecomoff 集群上某些 master 上配置为1，从用户日志上看，有 client 连接 FMS04 出现 reset
>
> 请 OP 同学集中检查一下该配置项的配置
>
> 另外 /proc/sys/net/ipv4/tcp_max_syn_backlog 参数也是配置不一，1024和10240的都有，是否可以统一一下？


DataNode 中出现大量异常：

```
2014-12-09 22:46:41,492 WARN org.apache.hadoop.hdfs.server.datanode.DataNode: java.io.IOException: Call to nj01-opensource-hdfs.rp.baidu.com/10.214.63.53:54310 failed on local exception: java.io.IOException: Connection reset by peer
```

最终的解决方法是所有的 master 节点执行：

```
echo 0 > /proc/sys/net/ipv4/tcp_abort_on_overflow
```

关于 TCP/IP 协议参数的详细文章可以参见：
[http://www.cnblogs.com/OnlyXP/archive/2007/09/29/911269.html](http://www.cnblogs.com/OnlyXP/archive/2007/09/29/911269.html)

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

