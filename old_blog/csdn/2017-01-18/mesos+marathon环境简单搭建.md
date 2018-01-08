
## 环境准备
<!--more-->  
```
centos6
$ uname -a
Linux liujinliu.centos 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
机器ip地址为192.168.211.131
```
## 安装jdk1.8
从以下地址下载对应的rpm包即可安装
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
下载之后，参考下面的命令安装
```
# rpm -ivh jdk-8u101-linux-x64.rpm
安装之后执行下面的命令，查看安装结果
# java -version
java version "1.8.0_101"
Java(TM) SE Runtime Environment (build 1.8.0_101-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.101-b13, mixed mode)
```

## 下载mesos安装包并安装
从以下网址找合适的包进行下载
http://open.mesosphere.com/downloads/mesos/
```
# wget http://repos.mesosphere.com/el-testing/6/x86_64/RPMS/mesos-1.0.0-1.0.79.rc4.centos65.x86_64.rpm
# yum install subversion libevent2-devel
# rpm -ivh mesos-1.0.0-1.0.79.rc4.centos65.x86_64.rpm
```
## 下载marathon
可以从https://mesosphere.github.io/marathon/直接下载
```
# wget http://downloads.mesosphere.com/marathon/v1.1.1/marathon-1.1.1.tgz
# tar zxf marathon-1.1.1.tgz
```
## zookeeper搭建
此步可以参考任意教程，此处略过
## mesos和marathon启动
#### 首先启动mesos(简单起见，首先启动成no-HA模式)
```
mkdir /opt/mesos-master
#启动master
mesos-start --work_dir=/opt/mesos-master --ip=192.168.211.131
mkdir /opt/mesos-slave
#启动agent
GLOG_v=1 mesos-agent --master=192.168.211.131:5050 --isolation=docker/runtime,filesystem/linux --work_dir=/opt/mesos-slave --image_providers=docker --docker_registry=http+socket://192.168.211.131:5000/ --containerizers=mesos,docker
```
通过mesos-excute提交task
```
mesos-execute --master=192.168.211.1131:5050 --name=test1 --command="docker pull 192.168.211.131:5000/letv_es/only_for_test:0.0.2"
```
mesos运行在本机的5050端口，访问http://192.168.211.131:5050/#/可以看到控制面板
启动
#### 启动marathon
```
# cd marathon-1.1.1/bin
# ./start --master 192.168.211.131:5050  --zk zk://127.0.0.1:2181/marathon
```
此时浏览器打开http://192.168.211.131:8080/ui/#/apps可以看到控制台页面
