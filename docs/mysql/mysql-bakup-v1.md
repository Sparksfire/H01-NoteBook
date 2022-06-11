## 数据库备份的分类

### 从物理与逻辑的角度，备份可分为

1. 物理备份：对数据库操作系统的物理文件（如数据文件、日志文件等）的备份

物理备份又可以分为脱机备份（冷备份）和联机备份（热备份）
 - 冷备份：是在关闭数据库的时候进行的
 - 热备份：数据库处于运行状态，这种备份方法依赖于数据库的日志文件
  
热备份：无需关机，无需关闭业务，就可直接识别

2. 逻辑备份：对数据库逻辑组件（如表等数据库对象）的备份

逻辑备份是对某个表、数据库备份

逻辑上的一张表映射出物理层面的三个文件：
 - 表的结构文件frm
 - 表的数据文件MYD
 - 表的索引文件MYI

### 从数据库的备份策略角度，备份可分为：

#### 完全备份

每次对数据进行完整的备份，会把服务器内的所有数据全部备份，每次都这么执行。优点：安全，缺点：数据备份冗余，占用磁盘空间

#### 差异备份

备份那些自从上次完全备份之后被修改过的文件，前提是必须要备份一次完全备份，接下来每次只备份基于完全备份的基础上被修改过的文件

#### 增量备份

只有那些在上次完全备份或者增量备份后被修改的文件才会被备份

差异备份和增量备份的相同点就是，基础都是完全备份，不同点在于差异备份只参考基础的完全备份，增量备份是参考上一次的数据备份与当前状态进行对比，备份那些被修改的文件，增量备份效率高，空间利用率很高，但是安全性能不高。

## MySQL完全备份

### 备份介绍

- 完全备份是对整个数据库的备份、数据库结构和文件结构的备份
- 完全备份保存的是备份完成时刻的数据库
- 完全备份是增量备份的基础
  
### 备份的优缺点

- 优点
  - 备份与恢复操作简单方便
- 缺点
  - 数据存在大量的重复
  - 占用大量的备份空间
  - 备份与恢复空间长

### 使用mysqldump

- mysql数据库的备份可以采用多种方式
  - 直接打包数据库文件夹，如/usr/local/mysql/data————这种是物理层面的备份
  - 使用工具 mysqldump————这种是逻辑层面的备份
  - 使用工具 xtrabackup

我们在之前已经创建好了数据库test，我们就以这个为例。

```bash
[root@localhost ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.31-log MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
5 rows in set (0.02 sec)

mysql> 
```

#### 使用tar进行物理上的备份

```bash
[root@localhost ~]# tar -Jcvf /opt/mysql-$(date +%F).tar.gz /usr/local/mysql/data/
tar: Removing leading `/' from member names
/usr/local/mysql/data/
/usr/local/mysql/data/ibdata1
/usr/local/mysql/data/ib_logfile1
/usr/local/mysql/data/ib_logfile0
/usr/local/mysql/data/auto.cnf
......
/usr/local/mysql/data/test/
/usr/local/mysql/data/test/db.opt
/usr/local/mysql/data/test/sms_staff.frm
/usr/local/mysql/data/test/sms_staff.ibd
/usr/local/mysql/data/ib_buffer_pool
/usr/local/mysql/data/ibtmp1
/usr/local/mysql/data/localhost.localdomain.pid
[root@localhost ~]# ls -al /opt/
total 772
drwxr-xr-x.  2 root root     37 May 17 01:26 .
dr-xr-xr-x. 18 root root    236 May 16 17:42 ..
-rw-r--r--   1 root root 789496 May 17 01:26 mysql-2022-05-17.tar.gz
[root@localhost ~]# 
```

#### mysqldump命令对库备份

- mysql自带的备份工具，相当方便对mysql进行备份
- 通过该命令工具可以将指定的库、表或全部的库导出为sql脚本，在需要恢复时可进行数据恢复

1. mysqldump命令对单个库进行完全备份

mysqdump -u 用户名 -p [密码] [选项] [数据库名] > /备份路径/备份文件名

单库备份举例：

`mysqldump -u root -p test > /backup/test.sql`

```bash
[root@localhost bin]# ./mysqldump -uroot -p test > /opt/test.sql
Enter password: 
[root@localhost bin]# ls -al /opt/
total 776
drwxr-xr-x.  2 root root     53 May 17 01:33 .
dr-xr-xr-x. 18 root root    236 May 16 17:42 ..
-rw-r--r--   1 root root 789496 May 17 01:26 mysql-2022-05-17.tar.gz
-rw-r--r--   1 root root   2291 May 17 01:33 test.sql
[root@localhost bin]# 
```

2. mysqldump命令对多个库进行完全备份

mysqdump -u 用户名 -p [密码] [选项] --databases 库名1 [库名2]… > /备份路径/备份文件名

多库备份举例：

`mysqldump -uroot -p  --databases test test2 > /opt/databases-test-test2.sql`

```bash
[root@localhost bin]# ./mysqldump -uroot -p  --databases test test2 > /opt/databases-test-test2.sql
Enter password: 
[root@localhost bin]# ls -al /opt/
total 780
drwxr-xr-x.  2 root root     85 May 17 01:37 .
dr-xr-xr-x. 18 root root    236 May 16 17:42 ..
-rw-r--r--   1 root root   3427 May 17 01:37 databases-test-test2.sql
-rw-r--r--   1 root root 789496 May 17 01:26 mysql-2022-05-17.tar.gz
-rw-r--r--   1 root root   2291 May 17 01:35 test.sql
[root@localhost bin]# 

```

3. 对所有库进行完全备份

mysqdump -u 用户名 -p [密码] [选项] --all-databases > /备份路径/备份文件名

所有库备份举例：

`mysqldump -u root -p --opt --all-databases > /backup/all-data.sql`

```bash
[root@localhost bin]# ./mysqldump -uroot -p --opt --all-databases > /opt/all-data.sql
Enter password: 
[root@localhost bin]# ls -al /opt/
total 1624
drwxr-xr-x.  2 root root    105 May 17 01:40 .
dr-xr-xr-x. 18 root root    236 May 16 17:42 ..
-rw-r--r--   1 root root 863617 May 17 01:40 all-data.sql
-rw-r--r--   1 root root   3427 May 17 01:37 databases-test-test2.sql
-rw-r--r--   1 root root 789496 May 17 01:26 mysql-2022-05-17.tar.gz
-rw-r--r--   1 root root   2291 May 17 01:35 test.sql
[root@localhost bin]# 
```

#### mysqdump命令备份表

1. 使用mysqldump备份表的操作
mysqdump -u 用户名 -p [密码] [选项] 数据库名 表名 > /备份路径/备份文件名

备份表的举例：

`mysqldump -u root -p mysql user > /backup/mysql-user.sql`

2. 表和数据备份

```bash
[root@localhost bin]# ./mysqldump -uroot -p test sms_staff > /opt/sms_staff.sql
Enter password: 
[root@localhost bin]# ls -al /opt/
total 1628
drwxr-xr-x.  2 root root    126 May 17 01:54 .
dr-xr-xr-x. 18 root root    236 May 16 17:42 ..
-rw-r--r--   1 root root 863617 May 17 01:40 all-data.sql
-rw-r--r--   1 root root   3427 May 17 01:37 databases-test-test2.sql
-rw-r--r--   1 root root 789496 May 17 01:26 mysql-2022-05-17.tar.gz
-rw-r--r--   1 root root   2291 May 17 01:54 sms_staff.sql
-rw-r--r--   1 root root   2291 May 17 01:35 test.sql
[root@localhost bin]# 
```

3. 单个表的结构

```bash
[root@localhost bin]# ls -al /opt/
total 1632
drwxr-xr-x.  2 root root    148 May 17 01:56 .
dr-xr-xr-x. 18 root root    236 May 16 17:42 ..
-rw-r--r--   1 root root 863617 May 17 01:40 all-data.sql
-rw-r--r--   1 root root   3427 May 17 01:37 databases-test-test2.sql
-rw-r--r--   1 root root 789496 May 17 01:26 mysql-2022-05-17.tar.gz
-rw-r--r--   1 root root   2100 May 17 01:56 sms_staffd.sql
-rw-r--r--   1 root root   2291 May 17 01:56 sms_staff.sql
-rw-r--r--   1 root root   2291 May 17 01:35 test.sql
[root@localhost bin]# cat /opt/sms_staffd.sql 
-- MySQL dump 10.13  Distrib 5.7.31, for linux-glibc2.12 (x86_64)
--
-- Host: localhost    Database: test
-- ------------------------------------------------------
-- Server version	5.7.31-log

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `sms_staff`
--

DROP TABLE IF EXISTS `sms_staff`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `sms_staff` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL,
  `status` int(1) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `gender` int(1) DEFAULT NULL,
  `skd_flag` int(1) DEFAULT NULL,
  `title` varchar(64) DEFAULT NULL,
  `name` varchar(64) DEFAULT NULL,
  `dept_id` bigint(20) DEFAULT NULL,
  `role_id` bigint(20) DEFAULT NULL,
  `registration_rank_id` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='员工表';
/*!40101 SET character_set_client = @saved_cs_client */;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2022-05-17  1:56:40
[root@localhost bin]# 
```

3. 完整的数据库表结构

```bash
[root@localhost bin]# ./mysqldump -uroot -p -q -d test > /opt/testd.sql
Enter password: 
[root@localhost bin]# ls -al /opt/
total 1632
drwxr-xr-x.  2 root root    165 May 17 02:03 .
dr-xr-xr-x. 18 root root    236 May 16 17:42 ..
-rw-r--r--   1 root root 863617 May 17 01:40 all-data.sql
-rw-r--r--   1 root root   3427 May 17 01:37 databases-test-test2.sql
-rw-r--r--   1 root root 789496 May 17 01:26 mysql-2022-05-17.tar.gz
-rw-r--r--   1 root root      0 May 17 02:01 sms_staffd.sql
-rw-r--r--   1 root root   2291 May 17 01:56 sms_staff.sql
-rw-r--r--   1 root root   2686 May 17 02:04 testd.sql
-rw-r--r--   1 root root   2291 May 17 01:35 test.sql
[root@localhost bin]# cat /opt/testd.sql 
-- MySQL dump 10.13  Distrib 5.7.31, for linux-glibc2.12 (x86_64)
--
-- Host: localhost    Database: test
-- ------------------------------------------------------
-- Server version	5.7.31-log

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `sms_role_permission_relation`
--

DROP TABLE IF EXISTS `sms_role_permission_relation`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `sms_role_permission_relation` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `role_id` bigint(20) DEFAULT NULL,
  `permission_id` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='员工角色与权限关系表';
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Table structure for table `sms_staff`
--

DROP TABLE IF EXISTS `sms_staff`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `sms_staff` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL,
  `status` int(1) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `gender` int(1) DEFAULT NULL,
  `skd_flag` int(1) DEFAULT NULL,
  `title` varchar(64) DEFAULT NULL,
  `name` varchar(64) DEFAULT NULL,
  `dept_id` bigint(20) DEFAULT NULL,
  `role_id` bigint(20) DEFAULT NULL,
  `registration_rank_id` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='员工表';
/*!40101 SET character_set_client = @saved_cs_client */;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2022-05-17  2:04:06
[root@localhost bin]# 
```

4. 完整数据库的数据不含表结构

```bash
[root@localhost bin]# ./mysqldump -uroot -p -q -t test > /opt/test-data.sql
Enter password: 
[root@localhost bin]# ls -al /opt/
total 1636
drwxr-xr-x.  2 root root    186 May 17 02:08 .
dr-xr-xr-x. 18 root root    236 May 16 17:42 ..
-rw-r--r--   1 root root 863617 May 17 01:40 all-data.sql
-rw-r--r--   1 root root   3427 May 17 01:37 databases-test-test2.sql
-rw-r--r--   1 root root 789496 May 17 01:26 mysql-2022-05-17.tar.gz
-rw-r--r--   1 root root      0 May 17 02:01 sms_staffd.sql
-rw-r--r--   1 root root   2291 May 17 01:56 sms_staff.sql
-rw-r--r--   1 root root   1720 May 17 02:09 test-data.sql
-rw-r--r--   1 root root   2686 May 17 02:04 testd.sql
-rw-r--r--   1 root root   2291 May 17 01:35 test.sql
[root@localhost bin]# cat /opt/test-data.sql 
-- MySQL dump 10.13  Distrib 5.7.31, for linux-glibc2.12 (x86_64)
--
-- Host: localhost    Database: test
-- ------------------------------------------------------
-- Server version	5.7.31-log

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Dumping data for table `sms_role_permission_relation`
--

LOCK TABLES `sms_role_permission_relation` WRITE;
/*!40000 ALTER TABLE `sms_role_permission_relation` DISABLE KEYS */;
/*!40000 ALTER TABLE `sms_role_permission_relation` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Dumping data for table `sms_staff`
--

LOCK TABLES `sms_staff` WRITE;
/*!40000 ALTER TABLE `sms_staff` DISABLE KEYS */;
/*!40000 ALTER TABLE `sms_staff` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2022-05-17  2:09:00
[root@localhost bin]# 
```

> - --add-locks:导出过程中锁定表，完成后回解锁。-q：不缓冲查询，直接导出至标准输出
> - -d：只导出表结构，不含数据
> - -t：只导出数据，不含表结构


## MySQL增量备份

在做MySQL增量备份之前，我们需要查看一下MySQL的log_bin是否开启，因为要做增量备份首先要开启 log_bin。

```mysql
mysql> show variables like '%log_bin%';
+---------------------------------+----------------------------------+
| Variable_name                   | Value                            |
+---------------------------------+----------------------------------+
| log_bin                         | ON                               |
| log_bin_basename                | /data/mysqlbinlog/mysqlbin       |
| log_bin_index                   | /data/mysqlbinlog/mysqlbin.index |
| log_bin_trust_function_creators | OFF                              |
| log_bin_use_v1_row_events       | OFF                              |
| sql_log_bin                     | ON                               |
+---------------------------------+----------------------------------+
6 rows in set (0.08 sec)

mysql> 
```

查看当前使用的 mysql_bin.000*** 日志文件

```mysql
mysql> show master status;
+-----------------+----------+--------------+------------------+-------------------+
| File            | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-----------------+----------+--------------+------------------+-------------------+
| mysqlbin.000011 |      154 |              |                  |                   |
+-----------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> 
```

可以看出，前正在记录日志的文件名为 mysqlbin.000008，当前数据库中有如下数据：

```mysql
mysql> use test;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select * from sms_staff;
+------+----------+----------------------------------+--------+---------------------+--------+----------+-------+---------+---------+---------+----------------------+
| id   | username | password                         | status | create_time         | gender | skd_flag | title | name    | dept_id | role_id | registration_rank_id |
+------+----------+----------------------------------+--------+---------------------+--------+----------+-------+---------+---------+---------+----------------------+
| 1001 | captain  | $BzoxAjoavjdgCqlKsg/exBBcx8eYxN. |      1 | 2022-05-11 01:04:53 |   NULL |     NULL | test  | captain |       1 |       1 |                    2 |
| 1002 | test     | $BzoxAjoavjdgCqlKsg/exBBcx8eYxN. |      2 | 2022-05-13 01:04:53 |   NULL |     NULL | test  | test    |       1 |       1 |                    2 |
+------+----------+----------------------------------+--------+---------------------+--------+----------+-------+---------+---------+---------+----------------------+
2 rows in set (0.00 sec)

mysql> 
```

接下来，我们向sms_staff表中插入一条数据：

```mysql
mysql> INSERT INTO `sms_staff` (`id`, `username`, `password`, `status`, `create_time`, `gender`, `skd_flag`, `title`, `name`, `dept_id`, `role_id`, `registration_rank_id`) VALUES (1003, 'test2', '$BzoxAjoavjdgCqlKsg/exBBcx8eYxN.', 2, '2022-05-14 01:04:53', NULL, NULL, 'test', 'test2', 1, 1, 2);
Query OK, 1 row affected (0.00 sec)

mysql> select * from sms_staff;
+------+----------+----------------------------------+--------+---------------------+--------+----------+-------+---------+---------+---------+----------------------+
| id   | username | password                         | status | create_time         | gender | skd_flag | title | name    | dept_id | role_id | registration_rank_id |
+------+----------+----------------------------------+--------+---------------------+--------+----------+-------+---------+---------+---------+----------------------+
| 1001 | captain  | $BzoxAjoavjdgCqlKsg/exBBcx8eYxN. |      1 | 2022-05-11 01:04:53 |   NULL |     NULL | test  | captain |       1 |       1 |                    2 |
| 1002 | test     | $BzoxAjoavjdgCqlKsg/exBBcx8eYxN. |      2 | 2022-05-13 01:04:53 |   NULL |     NULL | test  | test    |       1 |       1 |                    2 |
| 1003 | test2    | $BzoxAjoavjdgCqlKsg/exBBcx8eYxN. |      2 | 2022-05-14 01:04:53 |   NULL |     NULL | test  | test2   |       1 |       1 |                    2 |
+------+----------+----------------------------------+--------+---------------------+--------+----------+-------+---------+---------+---------+----------------------+
3 rows in set (0.00 sec)

mysql> 
```

接下来我们执行以下命令，使用新的日志文件。

```bash
[root@localhost bin]# ./mysqladmin -uroot -p flush-logs
Enter password: 
[root@localhost bin]# 
```
---

```mysql
mysql> show master status;
+-----------------+----------+--------------+------------------+-------------------+
| File            | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-----------------+----------+--------------+------------------+-------------------+
| mysqlbin.000012 |      154 |              |                  |                   |
+-----------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> 
```

从上面可以看出，MySQL日志文件从mysqlbin.000011变成了mysqlbin.000012，而mysqlbin.000011则记录着刚刚insert命令的日志。我们可以使用`show binlog events in 'mysqlbin.000011';`来看一下：

```bash
mysql> show binlog events in 'mysqlbin.000011';
+-----------------+-----+----------------+-----------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Log_name        | Pos | Event_type     | Server_id | End_log_pos | Info                                                                                                                                                                                                                                                                                              |
+-----------------+-----+----------------+-----------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| mysqlbin.000011 |   4 | Format_desc    |       190 |         123 | Server ver: 5.7.31-log, Binlog ver: 4                                                                                                                                                                                                                                                             |
| mysqlbin.000011 | 123 | Previous_gtids |       190 |         154 |                                                                                                                                                                                                                                                                                                   |
| mysqlbin.000011 | 154 | Anonymous_Gtid |       190 |         219 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                                                                                                                                                                              |
| mysqlbin.000011 | 219 | Query          |       190 |         291 | BEGIN                                                                                                                                                                                                                                                                                             |
| mysqlbin.000011 | 291 | Rows_query     |       190 |         602 | # INSERT INTO `sms_staff` (`id`, `username`, `password`, `status`, `create_time`, `gender`, `skd_flag`, `title`, `name`, `dept_id`, `role_id`, `registration_rank_id`) VALUES (1003, 'test2', '$BzoxAjoavjdgCqlKsg/exBBcx8eYxN.', 2, '2022-05-14 01:04:53', NULL, NULL, 'test', 'test2', 1, 1, 2) |
| mysqlbin.000011 | 602 | Table_map      |       190 |         675 | table_id: 109 (test.sms_staff)                                                                                                                                                                                                                                                                    |
| mysqlbin.000011 | 675 | Write_rows     |       190 |         804 | table_id: 109 flags: STMT_END_F                                                                                                                                                                                                                                                                   |
| mysqlbin.000011 | 804 | Xid            |       190 |         835 | COMMIT /* xid=11 */                                                                                                                                                                                                                                                                               |
+-----------------+-----+----------------+-----------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
8 rows in set (0.00 sec)

mysql> 
```

还可以使用mysqlbinlog工具查看：

```bash
[root@localhost bin]# ./mysqlbinlog --no-defaults /data/mysqlbinlog/mysqlbin.000011 --database EpointFrame --base64-output=decode-rows -vv --skip-gtids=true
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#220518  1:43:12 server id 190  end_log_pos 123 CRC32 0xc9d3e6a3 	Start: binlog v 4, server v 5.7.31-log created 220518  1:43:12 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
/*!50616 SET @@SESSION.GTID_NEXT='AUTOMATIC'*//*!*/;
# at 123
# at 154
# at 219
#220518  1:44:43 server id 190  end_log_pos 291 CRC32 0xeff09852 	Query	thread_id=2	exec_time=0	error_code=0
SET TIMESTAMP=1652809483/*!*/;
SET @@session.pseudo_thread_id=2/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1436549152/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8mb4 *//*!*/;
SET @@session.character_set_client=45,@@session.collation_connection=45,@@session.collation_server=8/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
BEGIN
/*!*/;
# at 291
#220518  1:44:43 server id 190  end_log_pos 602 CRC32 0xd1fec19d 	Rows_query
# INSERT INTO `sms_staff` (`id`, `username`, `password`, `status`, `create_time`, `gender`, `skd_flag`, `title`, `name`, `dept_id`, `role_id`, `registration_rank_id`) VALUES (1003, 'test2', '$BzoxAjoavjdgCqlKsg/exBBcx8eYxN.', 2, '2022-05-14 01:04:53', NULL, NULL, 'test', 'test2', 1, 1, 2)
# at 602
# at 675
# at 804
#220518  1:44:43 server id 190  end_log_pos 835 CRC32 0xe2dc7d5f 	Xid = 11
COMMIT/*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
[root@localhost bin]# 
```

那么到现在为止，其实已经完成了增量备份了。

### 恢复增量备份

我们删除刚才插入表中的数据：

```mysql
mysql> delete from sms_staff where id = 1003;
Query OK, 1 row affected (0.00 sec)

mysql> select * from sms_staff;
+------+----------+----------------------------------+--------+---------------------+--------+----------+-------+---------+---------+---------+----------------------+
| id   | username | password                         | status | create_time         | gender | skd_flag | title | name    | dept_id | role_id | registration_rank_id |
+------+----------+----------------------------------+--------+---------------------+--------+----------+-------+---------+---------+---------+----------------------+
| 1001 | captain  | $BzoxAjoavjdgCqlKsg/exBBcx8eYxN. |      1 | 2022-05-11 01:04:53 |   NULL |     NULL | test  | captain |       1 |       1 |                    2 |
| 1002 | test     | $BzoxAjoavjdgCqlKsg/exBBcx8eYxN. |      2 | 2022-05-13 01:04:53 |   NULL |     NULL | test  | test    |       1 |       1 |                    2 |
+------+----------+----------------------------------+--------+---------------------+--------+----------+-------+---------+---------+---------+----------------------+
2 rows in set (0.00 sec)

mysql> 
```

接下来，我们从mysqlbin.000011日志中恢复数据：

```bash
[root@localhost bin]# ./mysqlbinlog --no-defaults /data/mysqlbinlog/mysqlbin.000011 | mysql -uroot -proot  test;
mysql: [Warning] Using a password on the command line interface can be insecure.
[root@localhost bin]# 
```

上面的语句指定了需要恢复的mysqlbin文件，指定了用户名：root 、密码：root，数据库名：test，效果如下：

```mysql
mysql> select * from sms_staff;
+------+----------+----------------------------------+--------+---------------------+--------+----------+-------+---------+---------+---------+----------------------+
| id   | username | password                         | status | create_time         | gender | skd_flag | title | name    | dept_id | role_id | registration_rank_id |
+------+----------+----------------------------------+--------+---------------------+--------+----------+-------+---------+---------+---------+----------------------+
| 1001 | captain  | $BzoxAjoavjdgCqlKsg/exBBcx8eYxN. |      1 | 2022-05-11 01:04:53 |   NULL |     NULL | test  | captain |       1 |       1 |                    2 |
| 1002 | test     | $BzoxAjoavjdgCqlKsg/exBBcx8eYxN. |      2 | 2022-05-13 01:04:53 |   NULL |     NULL | test  | test    |       1 |       1 |                    2 |
| 1003 | test2    | $BzoxAjoavjdgCqlKsg/exBBcx8eYxN. |      2 | 2022-05-14 01:04:53 |   NULL |     NULL | test  | test2   |       1 |       1 |                    2 |
+------+----------+----------------------------------+--------+---------------------+--------+----------+-------+---------+---------+---------+----------------------+
3 rows in set (0.00 sec)

mysql> 
```

OK，到此差不多了！ 后期在更新关于MySQL binlog日志解析的内容。



