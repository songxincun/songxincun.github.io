---
layout: post
title:  "HDFS RESTful 接口"
date:   2014-10-24 21:03:02
categories: hadoop
---

### HDFS 支持 RESTful 接口
首先要配置 hdfs-site.xml 添加：

```
<property>
  <name>dfs.webhdfs.enabled</name>
  <value>true</value>
</property>
```

然后就可以访问了，例如获取 HDFS 根目录下的所有文件列表：
http://${NameNode-hostname}:${PORT}/webhdfs/v1/?op=liststatus
其中 PORT 为 NameNode 的 http 端口


[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

