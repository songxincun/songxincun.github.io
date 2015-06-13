---
layout: post
title:  "HDFS 一次事故"
date:   2015-06-13 17:51:52
categories: hadoop
---

表面现象就是两台 NameNode 都挂了，重启一台之后，如果再起另外一台，前面的那台就会挂
看 NameNode 的日志挂之前报

```
2015-06-13 07:29:23,195 FATAL org.apache.hadoop.hdfs.server.namenode.FSEditLog: Error: flush failed for required journal (JournalAndStream(mgr=QJM to [10.215.174.64:8485, 10.214.218.37:8485, 10.215.182.36:8485, 10.215.187.24:8485, 10.215.29.58:8485], stream=QuorumOutputStream starting at txid 1612800325))
org.apache.hadoop.hdfs.qjournal.client.QuorumException: Got too many exceptions to achieve quorum size 3/5. 4 exceptions thrown:
10.214.218.37:8485: IPC's epoch 39 is less than the last promised epoch 40
        at org.apache.hadoop.hdfs.qjournal.server.Journal.checkRequest(Journal.java:414)
        at org.apache.hadoop.hdfs.qjournal.server.Journal.checkWriteRequest(Journal.java:442)
        at org.apache.hadoop.hdfs.qjournal.server.Journal.journal(Journal.java:342)
        at org.apache.hadoop.hdfs.qjournal.server.JournalNodeRpcServer.journal(JournalNodeRpcServer.java:148)
        at org.apache.hadoop.hdfs.qjournal.protocolPB.QJournalProtocolServerSideTranslatorPB.journal(QJournalProtocolServerSideTranslatorPB.java:158)
        at org.apache.hadoop.hdfs.qjournal.protocol.QJournalProtocolProtos$QJournalProtocolService$2.callBlockingMethod(QJournalProtocolProtos.java:25421)
```

这个日志是说在 NameNode 往 QJM 写数据的时候报的 epoch 号太 low，被拒绝了，详细可参考 QJM 的原理

此时 HDFS ls 很慢，而 put 一个文件则一直报

```
15/06/13 11:03:56 INFO hdfs.DFSClient: Could not complete /tmp/disk-oum15._COPYING_ retrying...
15/06/13 11:04:00 INFO hdfs.DFSClient: Could not complete /tmp/disk-oum15._COPYING_ retrying...
15/06/13 11:04:07 INFO hdfs.DFSClient: Could not complete /tmp/disk-oum15._COPYING_ retrying...
```

同时 Cloudera Manager 上面的监控报警说
![Cloudera Manager监控报警](/images/HDFS-incident.JPG)

RPC 处理延迟时间超时（后面发现超过一秒，put 数据的时候就会报前面的问题了）

### 解决办法
各种无头绪之后，采用如下办法解决：

* 将 DataNode 数量降至 250 台（此前是 950 台）
* 然后启动一台 NameNode，这时发现 HDFS 恢复正常了，put 正常，而且 RPC 延迟降至很低
* 然后逐步将 DataNode 加入 HDFS，每次 50 台左右，加到最后一批的时候，HDFS 异常了，将最后一批下掉，又恢复正常
* 最后一批只能采取每次 10 台的方式加入，最后 HDFS 全部恢复正常

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

