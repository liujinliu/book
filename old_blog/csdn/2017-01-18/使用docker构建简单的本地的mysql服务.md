### 环境
```
liujinliu@liujinliu-OptiPlex-7010:~$ uname -a
Linux liujinliu-OptiPlex-7010 4.4.0-57-generic #78-Ubuntu SMP Fri Dec 9 23:50:32 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```
首先参考https://docs.docker.com/engine/installation/linux/ubuntulinux/#/install-the-latest-version安装docker服务
### 下载mysql镜像
参照https://hub.docker.com/_/mysql/的步骤下载mysql
```
docker pull mysql:5.6
```
### 启动mysql
```
sudo docker run --name mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.6
```
### 连接mysql
找到启动的msql容器并找到容器的ip
```
liujinliu@liujinliu-OptiPlex-7010:~$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
7a1400622109        mysql:5.6           "docker-entrypoint.sh"   About an hour ago   Up 35 minutes       3306/tcp            mysql
liujinliu@liujinliu-OptiPlex-7010:~$ sudo docker inspect 7a1400622109 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
```
接下来使用root用户连接mysql
```
liujinliu@liujinliu-OptiPlex-7010:~$ mysql -h 172.17.0.2 -u root -proot -t -P 3306
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.35 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
mysql> create database if not exists test default charset utf8 collate utf8_general_ci;
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)

mysql> 
```
### 一个简单的select for update的示例
```
mysql> select * from active_records;
+-----------+-------+-------+------+
| user_id   | utime | stime | snum |
+-----------+-------+-------+------+
| abcde1230 |     0 |     0 |    0 |
| abcde1231 |     0 |     0 |    0 |
| abcde1232 |     0 |     0 |    0 |
+-----------+-------+-------+------+
3 rows in set (0.00 sec)

mysql> set autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> begin work;
Query OK, 0 rows affected (0.00 sec)

# 此时其他连接执行下面的命令会被阻塞住(即这一行被锁住了，只有commit之后会解锁)
mysql> select snum from active_records where user_id='abcde1230' for update;
+------+
| snum |
+------+
|    0 |
+------+
1 row in set (0.00 sec)

mysql> update active_records set snum=1 where user_id='abcde1230';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> commit;
Query OK, 0 rows affected (0.03 sec)

mysql>
```