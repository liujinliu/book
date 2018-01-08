原文:http://www.kongxx.info/blog/?p=120
Zookeeper介绍 Zookeeper 分布式服务框架是用来解决分布式应用中经常遇到的一些数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等。本文主要从使用者角度来介绍一下Zookeeper的安装，配置及应用。
 <!--more-->  
### 单机模式
Zookeeper可以单机安装，这种应用模式主要用在测试或demo的情况下，在生产环境下一般不会采用。
1.  首先可以从Zookeeper的官方网站下载最新的安装包http://mirrors.cnnic.cn/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
2. 解压zookeeper到指定目录下，这里假定为/opt/zookeeper
3. 进入zookeeper目录下的conf目录，复制zoo_sample.cfg为zoo.cfg，并将内容修改如下
```
tickTime=2000
dataDir=/opt/zookeeper-3.4.6/data
clientPort=2181
```
tickTime：Zookeeper 服务器之间或客户端与服务器之间心跳的时间间隔。
dataDir：Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。
clientPort：Zookeeper 服务器监听端口，用来接受客户端的访问请求。
4.  配置完以后，就可以启动zookeeper服务了，进入Zookeeper/bin目录，运行下面的命令来启动Zookeeper服务
```
$ ./zkServer.sh start
JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```
可以采用下面的命令查看启动状态
```
$ ./zkServer.sh status
JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Mode: standalone
```
