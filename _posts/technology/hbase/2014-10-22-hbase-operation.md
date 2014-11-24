---
layout: post
title:  "HBase 操作记录"
date:   2014-10-22 14:18:22
categories: hbase
---

### 调整 hbase.hregion.max.filesize 配置
hbase.hregion.max.filesize
用于设置 HRegion 的最大大小，当一个 region 大小达到这个值时，就会发生 split
默认值是10G，但当时搭建集群的时候，考虑到 holmes 产品都是大表结构，所以加了个0...
现在的问题是值太大，region 太大，性能太差，所以值要调回去

虽然意识到 split 操作是 RegionServer 主导的，但最开始还是先修改了 master 的配置
重启 master 之后发现是没有生效的

后面通过 noah 上线的方式修改了所有的 RegionServer 的配置之后，接下来就是要滚动重启了

通过下面的命令可以从 zookeeper 上面获取到所有的在线 RegionServer

```bash
zkCli.sh ls /hbase/rs
```

然后就是在中控机遍历这些机器，挨个进行 graceful 重启

```bash
hbase/bin/graceful_stop.sh --restart --reload --debug $host
```

非常蛋疼的一件事情是：这个 graceful 重启在本机针对本机进行的时候，需要输入 ssh 密码...
所以得在中控机利用信任关系进行重启...

整个重启过程异常的慢，一整天时间才重启了90多台机器


[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

