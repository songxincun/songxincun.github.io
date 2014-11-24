---
layout: post
title:  "HBase Shell 命令"
date:   2014-11-08 17:20:00
categories: hbase
---

### HBase Shell 命令
#### 集群相关
| HBase Shell 命令 | 描述                    |
| ---------------- | ----------------------- |
| status           | 返回集群状态信息        |
| version          | 返回集群版本信息        |
| shutdown         | 关闭集群!!!             |
| tools            | 列出 HBase 所支持的工具 |

#### 表相关
| HBase Shell 命令 | 描述                         |
| ---------------- | ---------------------------- |
| list             | 列出集群所有的表             |
| describe         | 列出一个表的信息             |
| create           | 创建一个表                   |
| count            | 统计一个表的行数             |
| exists           | 判断一个表是否存在           |
| scan             | 遍历一个表                   |
| disable          | 禁用一个表                   |
| drop             | 删除一个表                   |
| enable           | 启用一个表                   |
| alert            | 修改列族模式                 |
| truncate         | 重新创建一张表（表会被清空） |

#### 数据操作
| HBase Shell 命令 | 描述                   |
| ---------------- | ---------------------- |
| get              | 获取行或单元的值       |
| put              | 插入或更新行           |
| delete           | 删除某个单元的值       |
| deleteall        | 删除某个行全部单元的值 |
| incr             | 增加某个单元的值       |

#### 其他
| HBase Shell 命令 | 描述       |
| ---------------- | ---------- |
| exit             | 退出 Shell |


[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

