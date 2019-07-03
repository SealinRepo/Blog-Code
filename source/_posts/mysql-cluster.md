---
title: MySQL搭建主从同步数据库(一主多从)
tags:
  - 数据库
  - MySQL
originContent: ''
categories:
  - 数据库
  - 技术
toc: true
date: 2019-07-02 23:15:20
---

例如,我们此时有3台服务器,分别为
~~~
192.168.1.100 --- Master
192.168.1.101 --- Slave
192.168.1.102 --- Slave
~~~
1.**Mysql默认是不允许远程连接的,首先打开每个服务器的远程访问权限,每个Mysql数据库都需要打开此项**
```
--打开远程访问用户
grant all on *.* to 'user'@'192.168.1.%' identified by 'password' with grant option;
--刷新权限配置
flush privileges;
```
2.修改Master(主库)配置文件,默认安装的配置文件一般在:
```
/etc/my.cnf
```
在[mysqld]节点加上如下配置:
```
server-id=1
log-bin=master-bin
log-bin-index=master-bin.index
#只同步test数据库(可选配置)
binlog-do-db=test
```
3.重启Mysql服务
```
systemctl restart mariadb
```
4.查看并记录master的信息
```
show master status;
```
![MysqlMaster.png](https://upload-images.jianshu.io/upload_images/8936944-34a339adea4011dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
#File
master-bin.000001
#################
#Position:
2077
```
这时Master库已经配置完成了,接下来配置Slaver库:
1.同样的,先配置/etc/my.cnf,与Master不同的只有server-id一项
```
server-id=10
log-bin=master-bin
log-bin-index=master-bin.index
```
2.重启数据库服务
```
systemctl restart mariadb
```
3.连接Mysql配置Master信息并开启Slave
```
--设置Master信息
change master to 
 master_host='192.168.1.100'
,master_user='user'
,master_password='password'
,master_log_file='master-bin.000001'
,master_log_pos=2077;

--开启slave
start slave;
```
4.查看slave状态:
```
show slave status \G;
```
结果如下:
~~~
MariaDB [(none)]> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.1.100
                  Master_User: user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master-bin.000001
          Read_Master_Log_Pos: 2077
               Relay_Log_File: mariadb-relay-bin.000002
                Relay_Log_Pos: 623
        Relay_Master_Log_File: master-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 2077
              Relay_Log_Space: 919
              Until_Condition: None
                        ..........
~~~

需要注意的是
-
```
 Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

两项要同时为Yes,同步服务才是正常状态.
-
配置192.168.1.102跟192.168.1.101是完全一样的,只要在配置my.cnf时,将server-id少做修改即可,比如
-
~~~
server-id=11
~~~