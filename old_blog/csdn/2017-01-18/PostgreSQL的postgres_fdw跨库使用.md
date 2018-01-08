# postgres_fdw简介
 <!--more-->  
fdw是foreign-data wrapper的一个简称，可以叫外部封装数据。
postgres_fdw实现的是各个postgresql数据库及远程数据库之间的跨库操作。
### 下载mysql_fdw
在安装之前，首先需要确认本地已经安装了mysql的客户端。
从以下网站下载mysql_fdw安装包
http://pgxn.org/dist/mysql_fdw/
上面的网站同时很清楚的介绍了如何安装，需要注意的是在安装之前根据本地postgres和mysql的安装路径，配置PATH环境变量
```
export PATH=/usr/local/pgsql/bin/:$PATH
export PATH=/usr/local/mysql/bin/:$PATH 
```
### 加载mysql_fdw
```
postgres=# CREATE EXTENSION mysql_fdw;
```
可采用\dx命令查看已经加载的模块
### 创建mysql_server
```
postgres=# CREATE SERVER mysql_server FOREIGN DATA WRAPPER 
           mysql_fdw OPTIONS (host'127.0.0.1', port'3306');
```
可采用/des查看已经创建的server
### 授权
```
postgres=# grant usageon FOREIGN servere mysql_server to testdb;
```
接下来的操作采用testdb用户来操作
### 创建用户映射
```
testdb=> CREATE USER MAPPING FOR postgres 
         SERVER mysql_server 
         OPTIONS (username'foo', password'bar');
```
### 创建foreign table
```
testdb=> CREATE FOREIGN TABLE warehouse(warehouse_id  int,
         warehouse_name text,warehouse_created datetime) 
         SERVER mysql_server 
         OPTIONS (dbname 'db', table_name 'warehouse');
```
