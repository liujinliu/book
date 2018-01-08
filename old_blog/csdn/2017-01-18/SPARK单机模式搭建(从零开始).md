因为是从零开始，所以本文将spark部署成单机模式，并且有些文件放到home的个人目录中，不过看下设置的环境变量的就可以知道，这些文件放那里其实是无所谓的

服务器环境为cenos，并且JDK已经正确安装，可通过jar命令是否可用来判断
```
$ jar
Usage: jar {ctxui}[vfmn0PMe] [jar-file] [manifest-file] [entry-point] [-C dir] files ...
Options:
    -c  create new archive
    -t  list table of contents for archive
...
```
### SCALA安装
```
wget http://www.scala-lang.org/files/archive/scala-2.10.1.tgz
tar zxf scala-2.10.1.tgz
sudo mv scala-2.10.1 /usr/local
```
在/etc/profile.d/下创建文件scala.sh，内容如下
```
$ cat /etc/profile.d/scala.sh 
#!usr/bin/env bash
export SCALA_HOME=/usr/local/scala-2.10.1
export PATH=$PATH:$SCALA_HOME/bin
```
使环境变量生效
```
cd /etc
source profile
```
### SBT安装
```
wget https://repo.typesafe.com/typesafe/ivy-releases/org.scala-sbt/sbt-launch/0.13.8/sbt-launch.jar
$cd ~/
$mkdir bin --如果已经存在这个目录则不需要重复创建
$mv sbt-launch.jar ~/bin
```
在~/bin目录下创建文件sbt，内容如下
```
$ cat sbt
SBT_OPTS="-Xms512M -Xmx1536M -Xss1M -XX:+CMSClassUnloadingEnabled -XX:MaxPermSize=256M"
java $SBT_OPTS -jar `dirname $0`/sbt-launch.jar "$@"
```
按如下方式测试，看是否OK
```
$ sbt
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=256M; support was removed in 8.0
[info] Set current project to liujinliu (in build file:/home/liujinliu/)
```
### 安装SPARK
```
wget http://mirrors.hust.edu.cn/apache/spark/spark-1.3.1/spark-1.3.1-bin-hadoop2.6.tgz
tar zxf spark-1.3.1-bin-hadoop2.6.tgz
```
启动SPARK
```
./sbin/start-master.sh
```
登录localhost:8080可以看到UI界面，上面会显示当前节点的信息
其中在标题处会显示当前master所在的URL信息，这个参数需要作为入参在创建worker时候使用(spark://IP:PORT)
启动一个worker
```
./bin/spark-class org.apache.spark.deploy.worker.Worker spark://IP:PORT
```
此时localhost:8080的页面可以看到worker的信息

下面以一个git上的工程为例讲解如何提交一个scala的driver
从下面的地址下载源代码
https://github.com/alvinj/ScalaApacheAccessLogParser

加载之后解压，目录结构如下
```
base/
    build.sbt
    README.md
    src/
        main/
            scala/
                 AccessLogParser.scala
                 AccessLogRecord.scala
                 SimpleTestApp.scala --这个文件为自己创建
```
在src下面有一个和main同级的目录test，删除，下面将讲解如何将这个工程编译打包然后提交给spark运行
SimpleTestApp.scala内容如下
```
package com.alvinalexander.accesslogparser

import java.util.Calendar

object SimpleTestApp {
    def main(args: Array[String]) {
        val records = """
124.30.9.161 - - [21/Jul/2009:02:48:11 -0700] "GET /java/edu/pj/pj010004/pj010004.shtml HTTP/1.1" 200 16731 "http://www.google.co.in/search?hl=en&client=firefox-a&rlz=1R1GGGL_en___IN337&hs=F0W&q=reading+data+from+file+in+java&btnG=Search&meta=&aq=0&oq=reading+data+" "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.9.0.11) Gecko/2009060215 Firefox/3.0.11 GTB5"
89.166.165.223 - - [21/Jul/2009:02:48:12 -0700] "GET /favicon.ico HTTP/1.1" 404 970 "-" "Mozilla/5.0 (Windows; U; Windows NT 5.1; de; rv:1.9.0.11) Gecko/2009060215 Firefox/3.0.11"
94.102.63.11 - - [21/Jul/2009:02:48:13 -0700] "GET / HTTP/1.1" 200 18209 "http://www.developer.com/net/vb/article.php/3683331" "Mozilla/4.0 (compatible; MSIE 5.01; Windows NT 5.0)"
124.30.7.162 - - [21/Jul/2009:02:48:13 -0700] "GET /images/tline3.gif HTTP/1.1" 200 79 "http://www.devdaily.com/java/edu/pj/pj010004/pj010004.shtml" "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.9.0.11) Gecko/2009060215 Firefox/3.0.11 GTB5"
122.165.54.17 - - [21/Jul/2009:02:48:12 -0700] "GET /java/java_oo/ HTTP/1.1" 200 32579 "http://www.google.co.in/search?hl=en&q=OO+with+java+standalone+example&btnG=Search&meta=&aq=f&oq=" "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.9.0.7) Gecko/2009021910 Firefox/3.0.7"
217.32.108.226 - - [21/Jul/2009:02:48:13 -0700] "GET /blog/post/perl/checking-testing-perl-module-in-inc-include-path/ HTTP/1.1" 200 18417 "http://www.devdaily.com/blog/post/perl/perl-error-cant-locate-module-in-inc/" "Mozilla/5.0 (X11; U; SunOS i86pc; en-US; rv:1.7) Gecko/20070606"
122.165.54.17 - - [21/Jul/2009:02:48:15 -0700] "GET /java/java_oo/java_oo.css HTTP/1.1" 200 1235 "http://www.devdaily.com/java/java_oo/" "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.9.0.7) Gecko/2009021910 Firefox/3.0.7"
""".split("\n").filter(_ != "")
        val parser = new AccessLogParser
        val rec = parser.parseRecord(records(0))
        println("IP ADDRESS: " + rec.get.clientIpAddress)
    }
}
```
这个文件的测试内容是参考源代码test目录的两个文件写的

修改build.sbt内容:
```
name := "ScalaApacheAccessLogParser"
version := "1.0"
scalaVersion := "2.10.4"  --主要是这行，与当前使用的scala版本匹配
resolvers += "Typesafe Repository" at "http://repo.typesafe.com/typesafe/releases/"
scalacOptions += "-deprecation"
libraryDependencies += "org.scalatest" % "scalatest_2.10" % "2.1.0" % "test"
```

在base目录下执行编译和打包
```
sbt  compile
sbt  package
```
需要注意的是sbt 0.13.x版本一定要和scala 2.10.x版本配套使用，使用scala 2.11.x版本会报错(can't expand macros compiled by previous versions of Scala)

编译成功会在base目录下生成target文件夹，并且可以在这个文件夹的子文件夹内找到scalaapacheaccesslogparser_2.10-1.0.jar，路径为
target/scala-2.10/scalaapacheaccesslogparser_2.10-1.0.jar

将target文件夹整体拷贝到与
spark-1.3.1-bin-hadoop2.6文件夹平行的位置
```
spark/
      spark-1.3.1-bin-hadoop2.6/
      target/
```
### 提交driver到spark
```
./spark-1.3.1-bin-hadoop2.6/bin/spark-submit --class "com.alvinalexander.accesslogparser.SimpleTestApp" --master spark://IP:PORT target/scala-2.10/scalaapacheaccesslogparser_2.10-1.0.jar
```
IP和PORT跟起worker时候的一样
可以在shell看到如下输出
``` 
IP ADDRESS: 124.30.9.161
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties

```


###另外安利我写的两个模块，欢迎使用或贡献代码：
[一个命令行版本的zookeeper cli](https://pypi.python.org/pypi/zoo_cmd)
[一个tornado web app框架快速生成工具](https://pypi.python.org/pypi/twapp)
