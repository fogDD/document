## mysql高可用

### mysql主从复制

搭建两台MySQL服务器，一台作为主服务器，一台作为从服务器，实现主从复制

环境：

  主数据库： 192.168.12.110 master

  从数据库： 192.168.12.111 slave

架构：![](C:\Users\h1172\Desktop\document\mysql主从架构图.png)



#### 1.数据库设置授权远程连接

```
mysql> use mysql;
Database changed
mysql> grant all privileges  on *.* to 'root'@'%' identified by "你的数据库授权用户密码";
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> select host,user,password from user;
+--------------+------+-------------------------------------------+
| host         | user | password                                  |
+--------------+------+-------------------------------------------+
| localhost    | root | *A731AEBFB621E354CD41BAF207D884A609E81F5E |
| 192.168.1.1 | root | *A731AEBFB621E354CD41BAF207D884A609E81F5E |
| %            | root | *A731AEBFB621E354CD41BAF207D884A609E81F5E |
+--------------+------+-------------------------------------------+
3 rows in set (0.00 sec)
```



##### 1.2数据库设置改表远程连接

```
use mysql;

update user set host = '%' where user = 'root';

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```



#### 2.修改mysql配置文件

```
 vim /etc/my.cnf
 
 #主数据库： 192.168.12.110
 server-id=1
 log-bin=mysql-bin
 log-slave-updates
 slave-skip-errors=all
 
  #主数据库： 192.168.12.111
 server-id=2
 log-bin=mysql-bin
 log-slave-updates
 slave-skip-errors=all
```



#### 3.所有节点重启mysql

```
systemctl restart mysqld
systemctl status mysqld
```



#### 4.登陆mysql检查配置文件是否生效

```
#master节点mysql> show variables like 'server_id';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 1     |
+---------------+-------+1
row in set (0.00 sec)

#slave节点mysql> show variables like 'server_id';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 2     |
+---------------+-------+
1 row in set (0.00 sec)
```



#### 5.登陆master节点执行如下命令

```
mysql> show master status; +------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      120|              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```



#### 6.登陆slave节点执行如下命令

```
mysql> change master to
-> master_host='192.168.12.110(主数据库ip)',
-> master_user='root',
-> master_password='123456',
-> master_log_file='mysql-bin.000001',
-> master_log_pos=154;
Query OK, 0 rows affected, 2 warnings (0.00 sec)
```

开启从节点

```
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```



#### 7.查看同步状态

```
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.12.110
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 1206
               Relay_Log_File: mysqld-relay-bin.000003
                Relay_Log_Pos: 1369
        Relay_Master_Log_File: mysql-bin.000002
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
          Exec_Master_Log_Pos: 1206
              Relay_Log_Space: 1706
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: 1a66d353-e9f9-11eb-a338-000c298024f0
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)

```



#### 8.在matser数据库创建以一个test数据库测试主从服务

```
#####master数据库######
mysql> create database test;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+


#####slave数据库######
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
```

