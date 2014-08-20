---
layout: post
title:  "Bash 重定向之我的理解"
date:   2014-08-20 08:21:42
categories: bash
---

首先够造一个测试用的脚本：test.sh 
该脚本作用很简单：
向标准输出打印：This is the standard output.
向标准错误打印：This is the standard error output.

```bash
#!/bin/bash

# This is the test.sh

echo This is the standard output.
echo This is the standard error output. >&2
```

为了达到一下目的：将 test.sh 的标准输出定向到文件：output 而将其标准错误定向到 rev 程序以便将标准错误翻转。

正常的思路是：

```bash
./print.sh > output 2>&1 | rev
```

而执行结果却是：无输出，而 output 的内容为：

```bash
This is the standard output.
This is the standard error output.
```

这是因为 shell 处理重定向的方式是：**从右向左.**

**也就是说 shell 先执行 ./print.sh 得到标准错误和标准输出，然后呢
从右向左执行重定向，即先 2>&1 把标准错误定向到标准输出
再执行 > output 把标准输出定向到 output 文件。所以会出现上面的结果**

要想得到预期的结果应该这样：

```bash
./print.sh 2>&1 > output | rev
```

按照上面的理解，先 > output 即将标准输出定向到 output
再 2>&1 将标准错误定向到标准输出然后通过管道送给 rev
结果就是：
output 文件里的内容为：This is the standard output.
而输出为：
    
```bash
.tuptuo rorre dradnats eht si sihT
```

如果是要同时重定向标准输出和标准错误到某个文件，最简单的方法就是用 `&>`, 如下所示：

```bash
./print.sh &> output
```

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

