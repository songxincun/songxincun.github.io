---
layout: post
title:  "hadoop streaming 用法"
date:   2014-10-05 12:18:22
categories: hadoop
---

[http://hadoop.apache.org/docs/r0.19.1/cn/streaming.html](http://hadoop.apache.org/docs/r0.19.1/cn/streaming.html)

```bash
$HADOOP_HOME/bin/hadoop  jar $HADOOP_HOME/hadoop-streaming.jar \
    -input myInputDirs  \
    -output myOutputDir \
    -mapper /bin/cat    \
    -reducer /bin/wc
```

input 作为 mapper 的输入，可以是多个文件或文件夹。然后所有的输入文件一起按照 map 任务的数量进行分块，每个块一起交给 mapper 进行处理

streaming 模式的 map 任务的输入是不分成 \<key, value\> 的，就是一整块赤裸裸的交给 mapper 指定的程序处理

然后 mapper 的输出则是每一行以第一个 \t 为分隔符，前面为 key，后面的为 value。如果没有 \t，则整行为 key，value 为 null

然后用这些 \<key, value\> 对做分桶，同一个 key 会分到同一个桶

然后将一个个的桶会作为 reducer 的输入，注意：同一个 key 的一定是交给同一个 reducer 处理（因为一定在同一个桶里面），而不同的 key 也可能交给同一个 reducer 处理。reducer 的输入是 mapper 输出的 \<key, value\> 对的 key + '\t' + value 形式的输出。

## aggregate
streaming 模式自带了一个 aggregate 的包，可以非常方便的进行统计计算。

使用方法就是，在 map 阶段，每一行都按照

```
function:$key\t$value
```
的格式输出，然后 reduce 阶段，指定 reducer 为 aggregate，则 MapReduce 任务就会对所有相同的 key 的 value 进行 function 计算了。

function 列表包括：

|                |                                                          |
| -------------- | -------------------------------------------------------- |
| DoubleValueSum | double 值序列的求和                                      |
| LongValueMax   | long 值序列求最大值                                      |
| LongValueMin   | long 值序列求最小值                                      |
| LongValueSum   | long 值序列求和                                          |
| StringValueMax | string 值序列的字母序最大值                              |
| StringValueMin | string 值序列的字母序最小值                              |
| UniqValueCount | 求单一值的个数                                           |
| ValueHistogram | 求每个值的个数、最小值、中值、最大值、平均值、标准方差   |

举例：

```bash
$HADOOP_HOME/bin/hadoop  jar $HADOOP_HOME/hadoop-streaming.jar \
    -input myInputDirs                                      \
    -output myOutputDir                                     \
    -mapper  'awk {printf("LongValueMax:%s\t%s", $1, $2)}'  \
    -reducer aggregate
```

## 调试
可以通过将 reduce 设置为 NONE，或者 将 mapred.reduce.tasks 设置为 0，从而免去 reduce 过程，这时候的输出就只是 map 任务的输出，可以看看结果对不对从而得知 map 是否符合预期，以及 reduce 任务接受了什么样的输入。

默认的 reducer 是 IdentityReducer，也就是直接将输入输出，通过设置 mapred.reduce.tasks 的值为非 0，就可以看到洗牌后的结果了

## 注意事项
   1. 在 Java 的 模式中，每个 map() 方法只处理一个键值对，不同的键值对之间很难进行信息交换。而在 streaming 模式中，每个 map 任务处理的是一个数据块，可以直接操作所有的数据块内的键值对。比如说 "wc -l"

   2. 在 Java 的 API 里面，reduce 阶段，所有相同键的键/值对被结组成为一个键和一列值，于是 reduce() 方法就只需要处理这一个键的这一组值。而在 streaming 编程模式里面，送到 reduce 阶段的数据块则是混在一起的（虽然是已经排好序且相同的键都交给同一个 reduce 任务），需要用户自己来分开

   3. 在向 streaming 传递 "Generic options" 的时候，要用 'D' 而不是 '-D'。例如：

    ```bash
    $HADOOP_HOME/bin/hadoop  jar $HADOOP_HOME/hadoop-streaming.jar \
        -input myInputDirs  \
        -output myOutputDir \
        -mapper /bin/cat    \
        -reducer /bin/wc    \
                            \
        D mapred.reduce.tasks=0
    ```

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

