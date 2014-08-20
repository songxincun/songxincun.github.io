---
layout: post
title:  "DOS 一些概念"
date:   2014-08-18 15:52:33
categories: dos
---

1.  Dos 变量和命令及命令的选项不区分大小写

2.  **注意：Dos 中的路径名是用 '\' 而非 '/' 来分隔的**
    其中 . 和 .. 跟 Shell 下一样分别表示 "当前目录" 和 "父目录"

3.  Shell 下用 '-' 来表示选项，如："-l"，"-R"
    Dos 用 '/' 来表示，比如说："/L"，"/R"，"/?"

4.  '*' 和 '?' 通配符跟 Shell 的意思一样

5.  HELP 命令可以列出常用的命令。或查询给出的命令的帮助信息
    command /? 也是一样的效果

6.  SET 命令列出所有的变量
    SET var 列出变量 var 的值

7.  变量 %PATH% 中保存的值是 Dos 查找命令的路径。相当于 Shell 下的 PATH 环境变量
    区别在于 Shell 用 ':' 来分割路径，而 Dos 用 ';' 分割   

8.  Dos 下文件的执行：
    Dos 下文件的执行跟 UNIX 不一样，Dos 没有 UNIX 可执行权限的概念。所有的文件由扩展名来分类。
    不同的分类关联到不同的开放式命令，举例来说：

    ```bat
    C:\Users\Administrator> ASSOC .html
    .html=WebFile
    C:\Users\Administrator> ASSOC .mht
    .mht=WebFile
    C:\Users\Administrator> ASSOC .txt
    .txt=txtfile
    ```
        
    ASSOC 命令查询给出的扩展名的分类。不同的扩展名可以分在同一类。比如上面的 .html 文件和 .mht 文件都属于 WebFile 类

    ```bat
    C:\Users\Administrator> FTYPE txtfile
    txtfile=C:\windows\notepad.exe %1
    ```
        
    FTYPE 命令查询给出的分类所对应的开放式命令。也就是说执行这类文件时，相当于执行对应的命令。
    Dos 下没有可执行权限的概念，任何文件列出来就会执行对应的开放式命令。比如说：

    ```bat
    C:\Users\Administrator> test.txt
    ```
        
    按照上面 FTYPE 给出的结果就相当于执行：

    ```bat
    C:\Users\Administrator> C:\windows\notepad.exe test.txt
    ```
        
    Dos 下有个变量 "%PATHEXT%" 里面保存了可以直接执行的文件的扩展名。
    这些文件列出名字而无需写全扩展名即可执行，感觉好无聊啊

9.  重定向和管道貌似基本跟 Shell 一样 

10. && 和 || 的短路作用跟 Shell 的一样。
    Shell 采用 ';' 来分隔同一行的命令。而 Dos 采用 '&' 来分隔。用 () 可以将多个命令组合起来   

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

