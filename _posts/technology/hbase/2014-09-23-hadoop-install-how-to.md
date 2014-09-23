---
layout: post
title:  "Hadoop 安装记录"
date:   2014-09-23 17:31:12
categories: hadoop
---

## 安装前准备
你可能需要为 hadoop 创建一个对应的用户，用下面这个命令

```
useradd -d /home/hadoop    -m hadoop
```

## hadoop 安装
### HDFS
1.  hadoop-env.sh
    主要是配置 JAVA_HOME 环境变量

2. core-site.xml
    
    ```xml
    <property>
        <name>fs.default.name</name>
        <value>hdfs://nj02-rp-2014-q1-01-jh2-210.nj02.baidu.com:54310</value>
    </property> 
    ```

3.  hdfs-site.xml

    ```xml
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/</value>
        <final>true</final>
    </property>

    <property>
        <name>dfs.name.dir</name>
        <value>${hadoop.tmp.dir}/dfs/name</value>
        <final>true</final>
    </property>

    <property>
        <name>dfs.hosts.exclude</name>
        <value>/home/hadoop/hadoop/conf/exclude</value>
    </property>

    <property>
        <name>dfs.data.dir</name>
        <value>${hadoop.tmp.dir}/disk0/dfs/data,${hadoop.tmp.dir}/disk1/dfs/data</vaule>
        <final>true</final>
    </property>
    ```
    确保 hadoop 用户对所配置的目录有权限

4.  启动 HDFS

    ```bash
    # 启动 namenode
    bin/hadoop-daemon.sh start namenode

    # 启动 secondarynamenode
    bin/hadoop-daemon.sh start secondarynamenode

    # 启动 datanode
    bin/hadoop-daemon.sh start datanode
    ```

### MapRed
弄好了 HDFS 之后，就可以弄 MapRed 了

1.  mapred-site.xml
    
    ```xml
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/</value>
        <final>true</final>
    </property>

    <property>
        <name>mapred.job.tracker</name>
        <value>nj02-rp-2014-q1-01-jh2-212.nj02.baidu.com:54311</value>
    </property>

    <property>
        <name>mapred.local.dir</name>
        <value>${hadoop.tmp.dir}/disk0/mapred/local,${hadoop.tmp.dir}/disk1/mapred/local</vaule>
        <final>true</final>
    </property>

    <property>
        <name>mapred.temp.dir</name>
        <value>${hadoop.tmp.dir}/mapred/temp</value>
        <final>true</final>
    </property>
    ```

2.  启动 MapRed

    ```bash
    # 启动 jobtracker
    bin/hadoop-daemon.sh start jobtracker

    # 启动 tasktracker
    bin/hadoop-daemon.sh start tasktracker
    ```

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

