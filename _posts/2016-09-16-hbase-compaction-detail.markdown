---
layout: "post"
title: "HBase compaction 机制详解"
date: "2016-09-16 17:54"
categories:
    - hbase
---

先写内存，当写到一定程度满足一定条件之后会触发 flush，flush 到磁盘，生成一个 HFile。由此 HFile 会越来越多，会影响读性能

另外，HBase 里面 delete 掉的数据、ttl 过期的数据、冗余版本的数据是不会立即删除的，只有在 compaction 重新生成新的 HFile 的时候才会被真正物理删除

基于以上原因，需要对 HFile 定期的做合并（compaction），减少 HFile 数量并且删除冗余数据

## Compaction 种类
HFile 在持续不断的刷下来，越来越多，那么在什么时候做合并比较合适呢？一种是定时的去做，比如说每天、每周或者是手动对所有的 HFile 做一次合并，合并成一个大而干净的 HFile，这种一劳永逸的合并称之为 *Major Compaction*。这种合并的缺点是很难把我合并的时机，比如说有些业务刷 HFile 比较频繁，就有可能来不及合，导致 HFile 数量太多影响读性能。另外就是每一次的 Major Compaction 都会把所有的数据重新写一遍，对于有些大的并且并不怎么更新的 Region 来说过于频繁的合并非常浪费 IO 和网络

另外一种合并策略是当 HFile 的个数达到一定指定的数量的时候，就会将若干个 HFile 合并成一个大一点的 HFile，从而保证 HFile 的个数保持在一个可控的数量。这种出于控制 HFile 数量而对部分 HFile 做合并的策略称之为 *Minor Compaction*。有一点值得注意的是：Minor Compaction 会选取部分 HFile 做合并，而如果系统发现这个 region 的所有的 HFile 都在选取的 HFile 里面的话，则这个 Minor Compaction 会升级成为 Major Compaction（反正所有的 HFile 都要合，和 Major Compaction 的性质一样，干脆升级）。同时为了避免数据反复的被重写（老的 HFile 里的数据会不断的参与下一次的 compaction 从而不断的被反复重写）导致性能的浪费，Minor compaction 可以配置一个参数，过大的 HFile 不会参与合并

总结：
* Major Compaction：甭管有多少 HFile，甭管这些 HFile 有多大，统统一股脑合并成一个大的 HFile，该删的多余的数据都删了
* Minor Compaction：由系统在 HFile 达到一定个数的时候触发，会仔细的选取要合并的 HFile，对满足条件的比较小的 HFile 做合并，从而在尽可能的不影响系统的性能的基础上控制 HFile 的个数

## Major Compaction 优化
系统默认的 Major Compaction 是每天做一次，这个值是由参数 *hbase.hregion.majorcompaction* 来配置的。也可以通过 major_compact 命令来手动对某个 region 执行 Major Compaction

由于 Major Compaction 比较粗暴简单，在一些比较大的业务的情况下会对 IO 和网络造成极大的影响和浪费。因此一般对线上业务会将系统自动的 Major Compaction 关掉，转而通过一些业务定制化的 Compaction 方案来对 HFile 进行合并

## Minor Compaction 优化

> 核心思路：定期对 region 下所有的 HFile 做 check，排除掉过大的（算法 1）和正在做 compaction 的 HFile，选取剩下的符合要求的 HFile 里的一部分做合并

Minor Compaction 里面最核心的算法就是如何选取合适的 HFile 来进行 compact，需要排除掉太大的 HFile，HFile 太大会影响系统性能，导致 IO、网络浪费。HFile 的个数也不宜太多。同时还要考虑多个 compact 任务之间的优先级

#### 基本配置：

| 参数名                | 配置项                                         | 默认值             | 线上值            | 说明                                      |
| -------------------- | ---------------------------------------------- | ----------------- | ---------         | --------------------------------------- |
| minFilesToCompact    | hbase.hstore.compaction.min                    | 3                 | 10                | 至少有这么多个满足条件的 StoreFile，Minor Compaction 才会启动     |
| maxFilesToCompact    | hbase.hstore.compaction.max	                | 10                | 25                | 一次 Minor Compaction 最多选这么多个 StoreFile                      |
| maxCompactSize       | hbase.hstore.compaction.max.size               | Long.MAX_VALUE    | 4294967296 (4G)   | 文件大小大于该值的 StoreFile 一定会被 Minor Compaction排除            |
| minCompactSize       | hbase.hstore.compaction.min.size               | memstoreFlushSize | 默认值             |文件大小小于该值的 StoreFile 一定会加入到 Minor compaction            |
| threadWakeFrequency  | hbase.server.thread.wakefrequency              | 10000             | 100               | Compaction Checker 周期                                      |
|                      | hbase.regionserver.thread.compaction.large     | 1                 | 1                 | 配置 largeCompactions 线程池的线程个数                               |
|                      | hbase.regionserver.thread.compaction.small     | 1                 | 1                 | 配置 smallCompactions 线程池的线程个数                               |
|                      | hbase.regionserver.thread.compaction.throttle  |                   | 8589934592 (8G)   | 待补充                                      |
|                      | hbase.hstore.compaction.ratio                  |                   | 3                 | 待补充                                      |

CompactionChecker 是 RS 上的工作线程（Chore），设置执行周期是通过 threadWakeFrequency 指定，大小通过 hbase.server.thread.wakefrequency 配置（默认 10000），然后乘以默认倍数 multiple （1000），毫秒时间转换为秒。因此，在不做参数修改的情况下，CompactionChecker 大概是 2hrs, 46mins, 40sec 执行一次。

首先，对于 HRegion 里的每个 HStore 进行一次判断，needsCompaction() 判断是否足够多的文件触发了 Compaction 的条件。

条件为：HStore 中 StoreFIles 的个数 – 正在执行 Compacting 的文件个数 > minFilesToCompact

操作：以最低优先级提交 Compaction 申请



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


## 注意事项
meta 表的 compaction
