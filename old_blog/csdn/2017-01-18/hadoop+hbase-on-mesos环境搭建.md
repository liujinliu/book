首先参考我的另外一篇博客搭建好mesos cluster环境
 <!--more-->  
http://blog.csdn.net/github_25679381/article/details/52424022
### 部署结构
192.168.1.100 (master)
192.168.1.101 (slave)
192.168.1.102 (slave)
三台服务器创建grid用户，同时配置ssh免密码登录
hadoop版本采用hadoop-2.5.0-cdh5.2.0
mesos版本为0.24.0
### hadoop-mesos framework源码编译
从下面的地址下载源码
https://github.com/mesos/hadoop
修改pom.xml，与我们使用的mesos版本对应
```
.....
<mesos.version>0.24.0</mesos.version>
....
```
#### 下载maven工具
选择apache-maven-3.3.9，直接从apache官网找就可以，下到本地解压
```export PATH=/home/grid/mesos/hadoop-package/apache-maven-3.3.9/bin:/usr/java/jdk1.8.0_101/bin:$PATH```
#### 进到framework目录下编译
```mvn package```

### hadoop版本下载
```
wget http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.5.0-cdh5.2.0.tar.gz
tar zxf hadoop-2.5.0-cdh5.2.0.tar.gz
```
将上一步编译完成的lib包放入下载的hadoop代码包中
```
cp /home/grid/mesos/hadoop-package/hadoop/target/hadoop-mesos-0.1.0.jar  /home/grid/mesos/hadoop-2.5.0-cdh5.2.0/share/hadoop/common/lib/
```
取消默认调度器(具体解释可以参考github上的源码库)
```
cd hadoop-2.5.0-cdh5.2.0
 
mv bin bin-mapreduce2
mv examples examples-mapreduce2
ln -s bin-mapreduce1 bin
ln -s examples-mapreduce1 examples
 
pushd etc
mv hadoop hadoop-mapreduce2
ln -s hadoop-mapreduce1 hadoop
popd
 
pushd share/hadoop
rm mapreduce
ln -s mapreduce1 mapreduce
popd
```
### 修改/etc/hosts，使得主机之间可以互相访问
这里有一点特别坑人，就是主机名字不能带下划线(xxx_xxx这样的hostname是不行的)
```
192.168.1.100  hostname-master
192.168.1.101  hostname-slave0
192.168.1.101  hostname-slave1
```
### hadoop配置
core-site.xml
```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
 
<!-- Put site-specific property overrides in this file. -->
 
<configuration>
  <property>
   <name>hadoop.tmp.dir</name>
   <value>/home/grid/working/hadoop/ljl_hadoop_data</value>
 </property>
  <property>
   <name>fs.defaultFS</name>
   <value>hdfs://192.168.1.100:9000</value>
 </property>
   <!-- This value should match the value in fs.defaultFS -->
 <property>
   <name>fs.default.name</name>
   <value>hdfs://192.168.1.100:9000</value>
 </property>
</configuration>
```
hdfs-site.xml
```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
 
<!-- Put site-specific property overrides in this file. -->
 
<configuration>
  <property>
  <name>dfs.permissions</name>
  <value>false</value>
  </property>
  <property>
  <name>dfs.replication</name>
  <value>1</value>
  </property>
</configuration>
```
mapred-site.xml
```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
 
<!-- Put site-specific property overrides in this file. -->
 
<configuration>
 <property>
  <name>mapred.job.tracker</name>
  <value>192.168.1.100:9001</value>
 </property>
 <property>
   <name>mapred.jobtracker.taskScheduler</name>
   <value>org.apache.hadoop.mapred.MesosScheduler</value>
 </property>
 <property>
   <name>mapred.mesos.taskScheduler</name>
   <value>org.apache.hadoop.mapred.JobQueueTaskScheduler</value>
 </property>
 <property>
   <name>mapred.mesos.master</name>
   <value>192.168.1.100:5050</value>
 </property>
 <property>
   <name>mapred.mesos.executor.uri</name>
   <value>hdfs://192.168.1.100:9000/hadoop-2.5.0-cdh5.2.0.tar.gz</value>
 </property>
</configuration>
```
masters
```
hostname-master
```
slaves
```
hostname-slave0
hostname-slave1
```
hadoop-env.sh
```
export JAVA_HOME=/usr/java/jdk1.7.0_76
export MESOS_JAR="/usr/share/java/mesos-0.24.0.jar"
export PROTOBUF_JAR="/home/grid/hadoop-pro/hadoop-2.5.2/share/hadoop/mapreduce1/lib/protobuf-java-2.5.0.jar"
export MESOS_NATIVE_JAVA_LIBRARY="/usr/local/lib/libmesos.so"
export MESOS_NATIVE_LIBRARY="/usr/local/lib/libmesos.so"
export HADOOP_CLASSPATH="/usr/share/java/mesos-0.24.0.jar:$HADOOP_CLASSPATH"
export HADOOP_HEAPSIZE=2000
```
### mesos额外配置
这个配置是为了可以在每台slave上可以找到hadoop命令
mesos-master-env.sh
```
export PATH=/home/grid/hadoop-package/hadoop-2.5.0-cdh5.2.0/bin:$PATH
export MESOS_log_dir=/var/log/mesos
```
mesos-slave-env.sh
```
export PATH=/home/grid/hadoop-package/hadoop-2.5.0-cdh5.2.0/bin:$PATH
export MESOS_log_dir=/var/log/mesos
```
配置更新后，需要重启mesos
```
mesos-stop-cluster.sh
mesos-start-cluster.sh
```
### 启动hdfs
在master节点操作
```
export PATH=/home/grid/hadoop-package/hadoop-2.5.0-cdh5.2.0/bin:$PATH
hadoop namenode -format
hadoop namenode &
hadoop datanode &
```
将上面的hadoop打包后传到hdfs
```
cd /home/grid/hadoop-package
tar zcf hadoop-2.5.0-cdh5.2.0.tar.gz hadoop-2.5.0-cdh5.2.0
hadoop fs -put hadoop-2.5.0-cdh5.2.0.tar.gz /hadoop-2.5.0-cdh5.2.0.tar.gz
```
### 启动jobtracker
```hadoop jobtracker &```
### 测试
采用hadoop自带的wordcount进行测试
```
echo "Hello World Bye World" > /tmp/file0
hadoop fs -mkdir /data
hadoop fs -copyFromLocal /tmp/file0 /data
hadoop jar /home/grid/hadoop-package/hadoop-2.5.0-cdh5.2.0/share/hadoop/mapreduce1/hadoop-examples-2.5.0-mr1-cdh5.2.0.jar wordcount /data /out
hadoop fs -cat /out/part-r-00000
```
hadoop在反复启动过程可能会遇到下面的错误
```Writing to HDFS could only be replicated to 0 nodes instead of minReplication (=1)```
这个问题可以参考下面的解答
http://stackoverflow.com/questions/15571584/writing-to-hdfs-could-only-be-replicated-to-0-nodes-instead-of-minreplication
暴力一点的解决方法是
```
1.停止所有hadoop进程(可以通过ps aux | grep hadoop查看)
2.rm -rf /home/grid/working/hadoop/ljl_hadoop_data/*
3.hadoop namenode -format
4.hadoop namenode &
5.hadoop datanode &
6.此时hdfs中的文件也全没了，需要重新放入
```
### Hbase搭建
搭建完hadoop on mesos后，下面在这个基础上搭建hbase
hbase采用Apache官网最新稳定版本，下载并解压到/home/grid/hbase-package路径下
http://mirrors.cnnic.cn/apache/hbase/stable/hbase-1.2.2-bin.tar.gz
#### zookeeper搭建
在三台服务器分别启动zookeeper，我操作时候是按照公司内部搭建zookeeper方法搭建的，其实完全可以参考任意教程搭建zookeeper集群。
#### hbase配置
hbase-env.sh
```
export JAVA_HOME=/usr/java/jdk1.7.0_76
export HBASE_LOG_DIR=/home/grid/working/hadoop/hbase/logs
export HBASE_MANAGES_ZK=false ###使用自己创建的zookeeper，true表示使用hbase自带的zookeeper
```
hbase-site.xml
```
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://192.168.1.100:9000/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.master</name>
    <value>hdfs://192.168.1.100:60000</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>192.168.1.100,192.168.1.101,192.168.1.102</value>
  </property>
</configuration>
```
hbase.rootdir指定Hbase数据存储目录
                                        hbase.cluster.distributed 指定是否是完全分布式模式，单机模式和伪分布式模式需要将该值设为false
                                         hbase.master指定Master的位置
                                        hbase.zookeeper.quorum 指定zooke的集群，多台机器以逗号分隔
修改conf下面的regionservers文件
```
192.168.1.100
192.168.1.101
192.168.1.102
```
### 将hase打包分发到其他节点
```
scp hbase-1.2.2.tar.gz grid@192.168.1.101:/home/grid/hbase-package
scp hbase-1.2.2.tar.gz grid@192.168.1.102:/home/grid/hbase-package
```
### hbase启动
```
cd hbase-1.2.2/bin
./start-hbase.sh
```
### 验证
```
[grid@hadoop-mesos-ljl-2 bin]# ./hbase shell
2016-09-03 16:21:40,657 WARN  [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/grid/hbase-package/hbase-1.2.2/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/grid/hadoop-package/hadoop-2.5.0-cdh5.2.0/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/grid/hadoop-package/hadoop-2.5.0-cdh5.2.0/share/hadoop/mapreduce1/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.2.2, r3f671c1ead70d249ea4598f1bbcc5151322b3a13, Fri Jul  1 08:28:55 CDT 2016
 
hbase(main):001:0> list
TABLE                                                                                                      
t1                                                                                                         
1 row(s) in 0.3070 seconds
 
=> ["t1"]
hbase(main):002:0> get 't1','rowkey001'
COLUMN                       CELL                                                                          
 f1:col1                     timestamp=1472887978064, value=value01                                        
1 row(s) in 0.2740 seconds
 
hbase(main):003:0>
```
