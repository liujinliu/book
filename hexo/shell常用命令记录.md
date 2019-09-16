---
title: shell常用命令记录
date: 2019-09-16
categories: 运维
---

####  shell命令杀死进程和所有子进程
```
pstree pid -p| awk -F"[()]" '{print $2}'| xargs kill -9
```

#### shell获取上一条启动命令的进程号
```
nohup {some command} &
echo $!
```
