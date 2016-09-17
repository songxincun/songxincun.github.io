---
layout: "post"
title: "HBase compaction 机制详解"
date: "2016-09-16 17:54"
categories:
    - hbase
---

先写内存，当写到一定程度满足一定条件之后会触发 flush，flush 到磁盘，生成一个 HFile。由此 HFile 会越来越多，会影响读性能

另外，HBase 里面 delete 掉的数据、TTL 过期的数据、冗余版本的数据是不会立即删除的，只有在 compaction 重新生成新的 HFile 的时候才会被真正物理删除

RS 如果和 DN 混布，会有数据本地化的优化。当个别 RS 挂掉，region 发生迁移后，通过对 region 进行 Major Compaction 重新本地写数据，可以恢复数据本地化

基于以上原因，需要对 HFile 定期的做合并（compaction），减少 HFile 数量、删除冗余数据、实现数据本地化

## Compaction 种类
HFile 在持续不断的刷下来，越来越多，那么在什么时候做合并比较合适呢？一种是定时的去做，比如说每天、每周或者是手动对所有的 HFile 做一次合并，合并成一个大而干净的 HFile，这种一劳永逸的合并称之为 *Major Compaction*。这种合并的缺点是很难把我合并的时机，比如说有些业务刷 HFile 比较频繁，就有可能来不及合，导致 HFile 数量太多影响读性能。另外就是每一次的 Major Compaction 都会把所有的数据重新写一遍，对于有些大的并且并不怎么更新的 Region 来说过于频繁的合并非常浪费 IO 和网络

另外一种合并策略是当 HFile 的个数达到一定指定的数量的时候，就会将若干个 HFile 合并成一个大一点的 HFile，从而保证 HFile 的个数保持在一个可控的数量。这种出于控制 HFile 数量而对部分 HFile 做合并的策略称之为 *Minor Compaction*。有一点值得注意的是：Minor Compaction 会选取部分 HFile 做合并，而如果系统发现这个 region 的所有的 HFile 都在选取的 HFile 里面的话，则这个 Minor Compaction 会升级成为 Major Compaction（反正所有的 HFile 都要合，和 Major Compaction 的性质一样，干脆升级）。同时为了避免数据反复的被重写（老的 HFile 里的数据会不断的参与下一次的 compaction 从而不断的被反复重写）导致性能的浪费，Minor compaction 可以配置一个参数，过大的 HFile 不会参与合并

总结：
* Major Compaction：甭管有多少 HFile，甭管这些 HFile 有多大，统统一股脑合并成一个大的 HFile，该删的多余的数据都删了
* Minor Compaction：由系统在 HFile 达到一定个数的时候触发，会仔细的选取要合并的 HFile，对满足条件的比较小的 HFile 做合并，从而在尽可能的不影响系统的性能的基础上控制 HFile 的个数

## Major Compaction 详解
系统默认的 Major Compaction 是每天做一次，这个值是由参数 *hbase.hregion.majorcompaction* 来配置的。也可以通过 major_compact 命令来手动对某个 region 执行 Major Compaction

由于 Major Compaction 比较粗暴简单，在一些比较大的业务的情况下会对 IO 和网络造成极大的影响和浪费。因此一般对线上业务会将系统自动的 Major Compaction 关掉，转而通过一些业务定制化的 Compaction 方案来对 HFile 进行合并

在需要对数据进行本地化的时候，可以操作 Major Compaction 来实现

## Minor Compaction 详解

> 核心思路：尽可能的对新写入的比较小的 HFile 做合并，从而控制整个 HFile 的个数。同时尽可能的避免对老的比较大的 HFile 做合并，以避免对性能造成太大的影响

Minor Compaction 里面最核心的算法就是如何选取合适的 HFile 来进行 compact，需要排除掉太大的 HFile，HFile 太大会影响系统性能，导致 IO、网络浪费。HFile 的个数也不宜太多。同时还要考虑多个 compact 任务之间的优先级

#### 最简单的实现
CompactionChecker 定期（threadWakeFrequency）的检查 HFile，设定一个 maxCompactSize，将大于该阈值的 HFile 排除在外，当剩下的 HFile 个数减去正在做 compaction 的文件个数大于 minFilesToCompact 时就开始对这些 HFile 做合并，每次最多合并 maxFilesToCompact 个文件

优先级方面，根据每次 CompactionRequest 要合并的总的 HFile 大小，对比配置的 throttle 的值，大的则提交到 largeCompactions 线程池，小的则提交到 smallCompactions 线程池。从而避免大任务阻塞小任务

#### 基本配置：

| 参数名                | 配置项                                         | 默认值             | 线上值            | 说明                                      |
| -------------------- | ---------------------------------------------- | ----------------- | ---------         | --------------------------------------- |
| minFilesToCompact    | hbase.hstore.compaction.min                    | 3                 | 10                | 至少有这么多个满足条件的 StoreFile，Minor Compaction 才会启动     |
| maxFilesToCompact    | hbase.hstore.compaction.max	                | 10                | 25                | 一次 Minor Compaction 最多选这么多个 StoreFile                         |
| maxCompactSize       | hbase.hstore.compaction.max.size               | Long.MAX_VALUE    | 4294967296 (4G)   | 文件大小大于该值的 StoreFile 一定会被 Minor Compaction排除            |
| minCompactSize       | hbase.hstore.compaction.min.size               | memstoreFlushSize | 默认值             |文件大小小于该值的 StoreFile 一定会加入到 Minor compaction              |
| threadWakeFrequency  | hbase.server.thread.wakefrequency              | 10000             | 100（秒）          | Compaction Checker 周期                                      |
|                      | hbase.regionserver.thread.compaction.large     | 1                 | 1                 | 配置 largeCompactions 线程池的线程个数                                |
|                      | hbase.regionserver.thread.compaction.small     | 1                 | 1                 | 配置 smallCompactions 线程池的线程个数                                |
|                      | hbase.regionserver.thread.compaction.throttle  |                   | 8589934592 (8G)   | 待补充                                     |
|                      | hbase.hstore.compaction.ratio                  |                   | 3                 | 待补充                                     |


#### Off-Peak 配置：

| 参数名                | 配置项                                         | 默认值             | 线上值            | 说明                                      |
| -------------------- | ---------------------------------------------- | ----------------- | ---------         | --------------------------------------- |
|                      | hbase.offpeak.start.hour                       |                   | 1                 | 待补充                                      |
|                      | hbase.offpeak.end.hour                         |                   | 6                 | 待补充                                      |
|                      | hbase.hstore.compaction.ratio.offpeak          |                   | 5.0               | 待补充                                      |
|                      | hbase.hstore.majorCompaction.peak.enable       |                   | true              | 待补充                                      |



#### Peak 配置：

| 参数名                | 配置项                                         | 默认值             | 线上值            | 说明                                      |
| -------------------- | ---------------------------------------------- | ----------------- | ---------         | --------------------------------------- |
|                      | hbase.peak.start.hour                          |                   | -1                | 待补充                                      |
|                      | hbase.peak.end.hour                            |                   | -1                | 待补充                                      |
|                      | hbase.regionserver.compaction.peak.maxspeed    |                   | 31457280          | 待补充                                      |


## 监控及问题排查
compaction 的目的是为了控制 HFile 个数，减少数据冗余。所以我们重点需要监控：
1. Region 下 HFile 个数
2. RS compaction 队列长度

另一方面，在排查问题的时候，可以通过查看 hdfs 上面的 HFile 个数及时间戳来判断实际 compaction 的工作情况，优化 compaction 实现


## 业务优化
> 核心思路：compaction 所起的作用是合并小文件、执行物理删除、删除 TTL 过期数据、数据本地化。compaction 会对文件进行重新读写合并，会对 IO 及网络造成负担。在能够保证业务需求的情况下，应当尽可能的避免进行 compaction，或者尽可能的减少单个 compaction 任务的规模

#### 优化经验
1. 关闭 Major Compaction
   * 系统默认的 Major Compaction 是每天执行一次，这个频率可能太频繁了。可以改成手动在业务低峰期执行
   * 由于配置优化好的 Minor Compaction 就已经可以很好的起到合并小文件的作用，所以 Major Compaction 可能并不是那么紧迫。综合考虑业务的情况（是否只有 append 操作，没有 update 和 delete 操作，TTL 是否永久保存）来确定 Major Compaction 是否可以配置的周期很长甚至是关闭，或者是优化实现

2. 数据冷热分离

   核心思路：对于按照时间顺序写入的数据（监控数据、流水数据、日志数据等），这些数据的特点是，写入都是集中在最新的时间（热数据），而之前写入的老的数据是只读的，基本不会再更新（冷数据），因此可以通过按时间分表来将冷热数据进行分离，从而区分对待。

   数据冷热分离的好处有：
   * TTL 优化：对于 TTL 过期的数据，可以直接删表，从而避免 Compaction 对系统 IO、网络造成的浪费
   * 对于冷数据可以定期在业务低峰期做一次 Major Compaction，然后后面就不需要管它。既可以获得最优读性能，又不需要反复的对不更新的数据做 Compaction
   * 建的表的规模更好控制，从而可以更准确的进行预分区，避免出现线上 region split 进而导致 Major Compaction 的情况


## 注意事项
#### meta 表的 compaction
meta 表由于 region 发生迁移，部分 RS 宕机，系统宕机等诸多复杂的原因，可能会出现各种不一致，各种脏数据。因此强烈建议定期对 meta 表做 Major Compaction。同时在集群故障恢复之后，也建议手动对 meta 表进行 Major Compaction 操作
