---
layout: post
title:  "Git 浅尝辄止"
date:   2014-08-16 09:00:43
categories: git
---

1.  不像 SVN, Git 不需要自己搭建服务器便能创建一个仓库

    ```bash
    git init
    ```
    这个命令会在当前目录建立一个 .git 文件夹，相当于是建立了一个本地仓库
    可以往里面修改、发布等各种操作，完全不需要服务器
    当然，如果打算将最终的版本分享出去，还是需要提交到服务器才行

2.  克隆一个远端服务器的仓库

    ```bash
    git clone username@host:/path/to/repository
    ```

3.  克隆一个本地仓库，这也就是说为啥 Git 可以脱机操作吧，不错

    ```bash
    git clone /path/to/repository
    ```

4.  添加某个文件或全部的文件到本地的缓冲区里面（计划改动）

    ```bash
    git add <filename>
    git add *
    git add --all
    ```

5.  将全部的改动发布到本地仓库的 HEAD 版本

    ```bash
    git commit -m "呵呵"
    ```

6. 要想将本地的仓库 push 到服务器上，执行：

    ```bash
    git push origin master
    ```
    master 可以替换成任意分支

7.  当然如果说是一个刚创建的本地仓库，没有同任何远程仓库绑定，可以执行下面命令来绑定：

    ```bash
    git remote add origin <server>
    ```

分支的概念，就是说出了 master 这一坨之外，还可以有其他的很多坨，以后再说

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

