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

## Minor Compaction 优化

## 注意事项
meta 表的 compaction
