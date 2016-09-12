---
layout: post
title:  ansible 入门坑爹指南
date:   2016-09-09 10:11:43
categories: ansible
---

### 命令行批量操作：
``` bash
/data/hbaseadmin/bigsong/ansible-2.1.0.0/bin/ansible    \
            -i "${ANSIBLE_HOST_FILE}" all           \
            -f 20                                       \
            -m copy                                     \
            -a "src=/data/hbaseadmin/bigsong/blockedupdate/timegrep.sh
                dest=/tmp/bigsong-timegrep.sh"


/data/hbaseadmin/bigsong/ansible-2.1.0.0/bin/ansible    \
            -i "${ANSIBLE_HOST_FILE}" all           \
            -f 20                                       \
            -e "${ANSIBLE_ARGV}"                        \
            -m command                                  \
            -a "${COMMAND}"
```
其中，ANSIBLE_HOST_FILE 为带操作的文件列表。ANSIBLE_ARGV 为可以传递到每台机器的 command 执行的变量，多个变量要放在一起，例如：
``` bash
ANSIBLE_ARGV="start_time='${STARTTIME}'
              stop_time='${STOPTIME}'
              grep_pattern='${GREPPATTERN}'
              print_pattern='${PRINTPATTERN}'"
```

上面两个不同的 ansible 分别用了两个不同的模块来实现不同的功能（通过 -m 参数来指定），其中 copy 用来完成文件分发的功能，而 command 用来执行命令操作。另外就是如果想用 command 来执行具体的 shell 命令，需要直接指定命令的全路径，或者是用 `bash -c` 的方式来指定

### PlayBook
playbook 可以允许执行一些列的剧本：
```yaml
---
- hosts: all
  gather_facts: no
  remote_user: hbaseadmin
  tasks:
  - copy: src=/data/hbaseadmin/bigsong/blockedupdate/batchgrep.sh dest=/tmp/batchgrep-bigsong.sh
    register: ignore

  - command: /bin/bash /tmp/batchgrep-bigsong.sh "{{ start_time }}" "{{ stop_time }}" "{{ grepargv }}"
    register: grepresult

  - debug: msg="{{ grepresult.stdout.split('\n') }}"
```
tasks 会依次执行，playbook 有个坑爹的地方在于没有合适的输出信息，需要用类似与上面的 debug 模块来进行执行结果打印

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com
