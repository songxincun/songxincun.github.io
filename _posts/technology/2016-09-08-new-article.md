---
layout: post
title:  "新年新气象7"
date:   2016-09-08 14:14:42
categories: c
---

LD_PRELOAD 环境变量可以将某个具体的库加入lib路径里
LD_LIBRARY_PATH 环境变量设置lib搜索路径

libc.so 是 Linux 下的基本库，误删之后，所有的程序都跑不了了...
如果你还在一个终端下，那还好，赶紧设置 LD_PRELOAD 环境变量将实际的 libc.so 文件加进去

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com