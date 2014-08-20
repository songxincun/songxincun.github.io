---
layout: post
title:  "虚拟化技术解决方案"
date:   2014-08-18 17:45:43
categories: virtualization
---

1. cgroups
   Linux 内核支持的一个功能，用于限制和隔离一个进程组资源（包括：CPU、内存、磁盘）
   [cgroups 维基百科介绍](http://zh.wikipedia.org/wiki/Cgroups)

2. LXC
   Linux Containers (LXC) 是 Linux 内核的一个容器功能接口，它使得 Linux 用户可以容易的创建和管理系统或应用容器
   [LXC 维基百科介绍](http://zh.wikipedia.org/wiki/Linux_Containers)

3. VirtualBox
   完全虚拟化的一个虚拟机软件，能虚拟各种操作系统，资源利用率应该不会很高（我自己认为的）
   [VirtualBox 维基百科介绍](http://zh.wikipedia.org/wiki/VirtualBox)

4. Openvz
   Linux 下完全虚拟化的一个虚拟机
   [Openvz 维基百科介绍](http://zh.wikipedia.org/wiki/OpenVZ)

4. Vagrant
   基于 VirtualBox 等虚拟机之上的一种虚拟技术，类似于 Ghost?

5. Docker
   集装箱的概念，不完全虚拟化，所以性能会比 Openvz、Vagrant 等要高，好像类似于 Matrix 他们搞的一样

6. Puppet

7. Chef

8. apt-get, yum, pacman 等包管理工具

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

