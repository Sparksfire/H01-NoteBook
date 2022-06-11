## MySQL主从复制介绍

MySQL的主从复制是一个异步的复制过程，数据从一个MySQL数据库（Master）复制到另一个MySQL数据库（Slave），在Master与Slave之间实现整个主从辅助的过程是由三个线程参与完成的。其中有2个线程（SQL线程和I/O线程）在Slave，另一个线程（I/O线程）在Master端。

要实现MySQL的主从复制，首先需要打开Master端的binlog记录功能，否则无法实现。因为整个复制过程实际上就是Slave与Master端获取binlog日志，然后Slave上以相同的顺序执行获取的binlog日志中所记录的各种SQL操作。

要打开MySQL的binlog功能，需要通过在MySQL的配置文件my.cnf中的mysqld模块（[mysqld]标识符后的参数部分）增加"log-bin"参数选项来实现，具体信息如下。

```cnf
[mysqld]
log-bin = /data/mysql-bin
```

## MySQL主从复制原理过程

下面简单描述MySQL Replication的复制原理过程。

1. 在Slave服务器上执行start slave命令开启主从复制开关，开始进行主从复制。
2. 此时，Slave服务器的I/O线程会通过在Master上已经授权的复制用户权限请求连接Master服务器，并请求从指定binlog日志文件的指定位置（日志文件名和位置就是在配置主从复制服务时执行change master命令指定的）之后开始发送binlog日志内容。
3. Master服务器接收到来自Slave服务器的I/O线程的请求后，其上负责复制的I/O线程会根据Slave服务器的I/O线程请求的信息分批读取指定binlog日志文件指定位置之后的binlog日志信息，然后返回给Slave端的I/O线程。返回的信息中除了binlog日志内容外，还有在Master服务器端记录的新的binlog文件名称，以及在新的binlog中的下一个指定更新位置。
4. 当Slave服务器的I/O线程获取到Master服务器上I/O线程发送的日志内容、日志文件及位置点后，会将binlog日志内容依次写到Slave端自身的Relay Log（即中继日志）文件（MySQL-relay-bin.xxxxxx）的最末端，并将新的binlog文件名和位置记录到master-info文件中，以便下一次读取Master端新binlog日志时能够告诉Master服务器从新binlog日志的指定文件及位置开始请求新的binlog日志内容。
5. 当Slave服务器的I/O线程获取到Master服务器上I/O线程发送的日志内容、日志文件及位置点后，会将binlog日志内容依次写到Slave端自身的Relay Log（即中继日志）文件（MySQL-relay-bin.xxxxxx）的最末端，并将新的binlog文件名和位置记录到master-info文件中，以便下一次读取Master端新binlog日志时能够告诉Master服务器从新binlog日志的指定文件及位置开始请求新的binlog日志内容。

经过了上面的过程，就可以确保在Master端和Slave端执行了同样的SQL语句。当复制状态正常时，Master端和Slave端的数据是完全一样的。当然，MySQL的复制机制也有一些特殊情况，具体请参考官方的说明，大多数情况下，大家不用担心。

![](https://images.icodedream.com/images/2022/05/16/mysql-v1.jpg)


## 主从复制实践

### 在主库Master上执行操作配置
1. 设置server-id并开启binlog

MySQL主从复制原理我们知道，要实现主从复制，关键是要开启binlog日志功能，所以，首先来打开主库的binlog日志参数。

修改主库my.cnf配置文件，在[mysqld]模块下新增参数：

```cnf
server-id=190
log-bin=/data/mysqlbinlog/mysqlbin
```
>[!Tip]server-id的值使用服务器IP地址最后一个小数点后面数字，如19，目的是避免不同机器或实例ID重复（不适合多实例）

2. 重启主库MySQL服务后，登录数据库检查参数

```mysql
mysql> show variables like 'server_id';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 190   |
+---------------+-------+
1 row in set (0.02 sec)

mysql> 
```

#### 创建用于主从复制的账号

根据主从复制的原理，从库要想和主库同步，必须有一个可以连接主库的账号，并且这个账号是主库上创建的，权限是允许主库的从库连接并同步数据。

1. 登录主库MySQL数据库，创建用于主从复制的账号

```mysql
mysql> grant replication slave on *.* to 'rep'@'192.168.35.%' identified by 'password';
Query OK, 0 rows affected, 1 warning (0.04 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> 
```
>[!Tip]192.168.35.%  通配符，表示0-255的IP都可访问主服务器，正式环境请配置指定从服务器IP,若将 192.168.17.% 改为 %，则任何ip均可作为其从数据库来访问主服务器

2. 查看主库rep账号命令及结果：

```mysql
mysql> select user, host from mysql.user;
+---------------+--------------+
| user          | host         |
+---------------+--------------+
| root          | %            |
| rep           | 192.168.35.% |
| mysql.session | localhost    |
| mysql.sys     | localhost    |
+---------------+--------------+
4 rows in set (0.00 sec)

mysql> show grants for rep@'192.168.35.%';
+--------------------------------------------------------+
| Grants for rep@192.168.35.%                            |
+--------------------------------------------------------+
| GRANT REPLICATION SLAVE ON *.* TO 'rep'@'192.168.35.%' |
+--------------------------------------------------------+
1 row in set (0.00 sec)

mysql> show master status;
+-----------------+----------+--------------+------------------+-------------------+
| File            | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-----------------+----------+--------------+------------------+-------------------+
| mysqlbin.000001 |     1761 |              |                  |                   |
+-----------------+----------+--------------+------------------+-------------------+
1 row in set (0.44 sec)

mysql> 
```

### 在从库Slave上执行的操作过程

1. 设置server-id值并关闭binlog功能参数

数据库的server-id一般在一套主从复制体系内是唯一的，这里从库的server-id要和主库及其他从库的不同，并且要注释掉从库的binlog参数配置，如果从库不做级联复制，并且不作为备份用，就不要开启binlog，开启了反而会增加从库磁盘I/O等的压力。

但是，有以下两种情况需要打开从库的binlog记录功能，记录数据库更新的SQL语句：

- 级联同步A→B→C中间的B时，就要开启binlog记录功能。
- 在从库做数据库备份，数据库备份必须要有全备和binlog日志，才是完整的备份。

修改my.cnf配置文件,配置从库的相关参数

```cnf
[mysqld]
server-id = 191
```

2. 重启MySQL服务，并进入数据库检查参数

```mysql
mysql> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+
1 row in set (0.00 sec)

mysql> show variables like 'server_id';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 191   |
+---------------+-------+
1 row in set (0.00 sec)

mysql> 
```

3. MySQL从库连接主库

```mysql
mysql> change master to master_host='192.168.35.190', master_user='rep', master_password='password', master_port=3306, master_log_file='mysqlbin.000001', master_log_pos=154;
Query OK, 0 rows affected, 2 warnings (0.02 sec)

mysql> 
```

上述的操作原理就是把用户密码等上述信息写入从库新的master.info文件中。

```bash
[root@localhost ~]# cat /usr/local/mysql/data/master.info 
25
mysqlbin.000001
1761
192.168.35.190
rep
password
3306
60
0





0
30.000

0
eb390291-d4fa-11ec-925d-000c2930872e
86400


0


[root@localhost ~]# 
```

4. 启动从库主从复制

```mysql
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

mysql> 
```

5. 查看从库复制状态

```mysql
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.35.190
                  Master_User: rep
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysqlbin.000001
          Read_Master_Log_Pos: 591
               Relay_Log_File: localhost-relay-bin.000002
                Relay_Log_Pos: 756
        Relay_Master_Log_File: mysqlbin.000001
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
          Exec_Master_Log_Pos: 591
              Relay_Log_Space: 967
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
             Master_Server_Id: 190
                  Master_UUID: eb390291-d4fa-11ec-925d-000c2930872e
             Master_Info_File: /usr/local/mysql/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)

ERROR: 
No query specified

mysql> 
```

6. 主从复制是否成功，最关键的为下面的3项状态参数：

```bash
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Seconds_Behind_Master: 0
```

- Slave_IO_Running：Yes，这个是I/O线程状态，I/O线程负责从从库到主库读取binlog日志，并写入从库的中继日志，状态为Yes表示I/O线程工作正常。
- Slave_SQL_Running：Yes，这个是SQL线程状态，SQL线程负责读取中继日志（relay-log）中的数据并转换为SQL语句应用到从数据库中，状态为Yes表示I/O线程工作正常。
- Seconds_Behind_Master：0，这个是复制过程中从库比主库延迟的秒数，这个参数很重要，但企业里更准确地判断主从延迟的方法为：在主库写时间戳，然后从库读取时间戳，和当前数据库时间进行比较，从而认定是否延迟。

7. 测试主从复制

我们在主库创建名为test的数据库，并创建一张表。我们可以从下图中看出主从库是同步的。

![](https://images.icodedream.com/images/2022/05/16/mysql-v2.png)