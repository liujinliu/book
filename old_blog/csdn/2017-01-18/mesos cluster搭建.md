### 部署结构
192.168.1.100 (master)
192.168.1.101 (slave)
192.168.1.102 (slave)
在三台服务器上创建非root账号grid，同时配置三台服务器ssh免密码登录。
软件版本：mesos-0.24.0-1.0.27.centos65.x86_64.rpm
软件下载地址以及jdk安装方法可以参考我的另外一篇博客
http://blog.csdn.net/github_25679381/article/details/52034995

### 安装mesos
```
rpm -ivh mesos-0.24.0-1.0.27.centos65.x86_64.rpm
```
### 配置mesos
配置文件位于/usr/etc/mesos下面
各个文件内容配置如下：
masters
```
192.168.1.100
```
slaves
```
192.168.1.101
192.168.1.102
```
mesos-deploy-env.sh
```
#!/usr/bin/env bash
 
# This file contains environment variables that modify how the deploy
# scripts are run. For example, it can be used to configure SSH
# options (see below).
 
# Options for SSH.
export SSH_OPTS="-o StrictHostKeyChecking=no -o ConnectTimeout=2"
 
# Use sudo for launching masters and slaves.
#export DEPLOY_WITH_SUDO=1
```
mesos-master-env.sh
```
# This file contains environment variables that are passed to mesos-master.
# To get a description of all options run mesos-master --help; any option
# supported as a command-line option is also supported as an environment
# variable.
 
# Some options you're likely to want to set:
# export MESOS_log_dir=/var/log/mesos
export MESOS_ip=10.185.81.195
export MESOS_work_dir=/home/grid/working/meso
```
mesos-slave-env.sh
```
# This file contains environment variables that are passed to mesos-slave.
# To get a description of all options run mesos-slave --help; any option
# supported as a command-line option is also supported as an environment
# variable.
 
# You must at least set MESOS_master.
 
# The mesos master URL to contact. Should be host:port for
# non-ZooKeeper based masters, otherwise a zk:// or file:// URL.
export MESOS_master=10.185.81.195:5050
export MESOS_work_dir=/home/grid/working/mesos
 
# Other options you're likely to want to set:
# export MESOS_log_dir=/var/log/mesos
# export MESOS_work_dir=/var/run/mesos
# export MESOS_isolation=cgroups
```
### 配置hosts文件
/etc/hosts
```
192.168.1.100  hostname-master
192.168.1.101  hostname-slave0
192.168.1.101  hostname-slave1
```
### 启动mesos集群
```
mesos-start-cluster.sh
```
### 停止mesos集群
```
mesos-stop-cluster.sh
```