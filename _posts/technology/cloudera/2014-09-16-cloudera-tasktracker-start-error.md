---
layout: post
title:  "Cloudera TaskTracker 启动失败问题追查记录"
date:   2014-09-16 16:25:44
categories: cloudera
---

### 问题描述
通过 Cloudera Manager 安装 CDH 时，在启动 MapRed 阶段失败，查看发现 Jobtracker 启动成功，而 TaskTracker 全部失败
错误信息如下图所示：
![ERROR-Message](/images/Cloudera-TaskTracker-Start-ERROR.JPG)
> Error found before invoking supervisord: dictionary update sequence element #78 has length1; 2 is required

实际上这个错误是由于环境变量导致的
执行`env`会发现有这么一个环境变量

```
module=() {  eval `/usr/bin/modulecmd bash $*`
}
```

这是一个函数环境变量，由于 env 显示出来的结果不在一行，而本身 cmf 的程序有个 bug，解析这个变量的时候就会有问题

---

### 修复方法
找到并打开文件 \<CM-PATH\>/lib64/cmf/agent/src/cmf/util.py 
找到函数`source`，里面有一行代码：

```
pipe = subprocess.Popen(['/bin/bash', '-c', ". %s; %s; env" % (path, command)],
                        stdout=subprocess.PIPE, env=caller_env)
```
将这行代码修改成

```
pipe = subprocess.Popen(['/bin/bash', '-c', ". %s; %s; env | grep -v { | grep -v }" % (path, command)],
                        stdout=subprocess.PIPE, env=caller_env)
```
也就是将其中的`env`改成`env | grep -v { | grep -v }`，这样程序在通过`env`获取环境变量的时候就会把那个坑爹的 module 给排除掉了

然后重启 agent，再重试就 OK 了。

---

### 排查过程回顾
由于之前其他的模块出现错误的时候都是通过查看 \<CM-PATH\>/run/cloudera-scm-agent/process 下的对应的日志来解决的（这个文件夹下面包含了当前机器的各个角色对应的启动配置及启动日志）
但是这次的 TaskTracker 对应的目录下面并没有 log，也就是说压根就没来得及执行 MapRed 启动脚本，程序就跪了
于是查看 agent 的日志 \<CM-PATH\>/log/cloudera-scm-agent/cloudera-scm-agent.log 
然后发现里面有跟之前图片吻合的错误日志

```
Traceback (most recent call last):
  File "/home/work/opdir/cm-4.8.1/lib64/cmf/agent/src/cmf/util.py", line 286, in source
    return dict((line.split("=", 1) for line in data.splitlines()))
ValueError: dictionary update sequence element #80 has length 1; 2 is required
```
于是就打开 /home/work/opdir/cm-4.8.1/lib64/cmf/agent/src/cmf/util.py 文件，找到那一行，发现代码是这样写的

```
  try:
    return dict((line.split("=", 1) for line in data.splitlines()))
  except Exception, e:
    LOG.exception("Failed to parse environment variables: " + data)
    raise e
```
然后回到日志文件，找到 Failed to parse environment variables: 这么一行

```
[16/Sep/2014 16:03:18 +0000] 2091 MainThread util         ERROR    Failed to parse environment variables: CDH_HCAT_HOME=/opt/cloudera/parcels/CDH-4.7.0-1.cdh4.7.0.p0.40/lib/hcatalog
HADOOP_LOGFILE=hadoop-cmf-mapreduce1-TASKTRACKER-nj02-rp-2014-q1-01-jh2-213.nj02.baidu.com.log.out
TOMCAT_HOME=/opt/cloudera/parcels/CDH-4.7.0-1.cdh4.7.0.p0.40/lib/bigtop-tomcat
HOSTNAME=nj02-rp-2014-q1-01-jh2-213.nj02.baidu.com
HADOOP_SECURITY_LOGGER=INFO,RFAS
CDH_SOLR_HOME=/usr/lib/solr
HADOOP_LOG_DIR=/var/log/hadoop-0.20-mapreduce
......
```

根据前面的代码可以了解到 Failed to parse environment variables: 这句话之后的一系列的 key=value 就是导致程序异常的 "data" 了
为了找出罪魁祸首的 key=value 组合，创建一个包括所有的 key=vaule 对的文件，然后对其执行 dict((line.split("=", 1) for line in data.splitlines())) 操作，最终找到罪魁祸首：

```
module=() {  eval `/usr/bin/modulecmd bash $*`
}
```
找到祸首之后，接下来考虑的就是如何去解决这个问题了
回溯 data 的来源，可以看到这样的代码

```
pipe = subprocess.Popen(['/bin/bash', '-c', ". %s; %s; env" % (path, command)],
                        stdout=subprocess.PIPE, env=caller_env)
data = pipe.communicate()[0]
```
总而言之，data 就是通过执行 `/bin/bash -c ". XXX; CMD; env"` 这个命令得到的
出去单独执行`env`命令，就能找到

```
module=() {  eval `/usr/bin/modulecmd bash $*`
}
```
这样的变量，因此可以通过修改程序，将`env`命令修改成`env | grep -v { | grep -v }`来修复这一问题

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

