## MySQL配置文件的优化

MySQL自身的优化主要是对其配置文件`my.cnf`文件`[mysqld]`段落里面的内容进行的。

`[mysqld]`段落内包括了mysqld服务启动时的参数，涉及很多方面，包含MySQL的目录和文件、通信、网络、信息安全、内存管理、优化、查询缓存、还有MySQL日志设置等。

```
# mysqld服务运行时的端口号
port = 3306

# 用户在Linux环境下进行客户端连接时可以不通过TCP/IP网络而直接使用socket连接MySQL
socket = /tmp/mysql.sock

# 避免MySQL的外部锁定，减少出错几率，增强稳定性
skip-external-locking

# 禁止MySQL对外部连接进行DNS解析，使用这一选项可以消除MySQL进行DNS解析的时间。需要注意的是，如果开启该选项，所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求
skip-name-resolve

# 禁止MySQL对外部连接进行DNS解析，使用这一选项可以消除MySQL进行DNS解析的时间。需要注意的是，如果开启该选项，所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求
back_log = 384

# key_buffer_size指定用于索引的缓冲区大小，增加它可得到更好的索引处理性能。对于内存在4GB左右的服务器来说，该参数可设置为256MB或384MB。不建议将该参数值设置得过大，这样反而会使服务器的整体效率降低。
key_buffer_size = 384M

# 在网络传输中，一次消息传输量的最大值，系统默认值为1MB，最大值是1GB，必须设定为1024的倍数，单位为字节
max_allowed_packet = 4M

# 设置MySQL每个线程的堆栈大小，默认值足够大，可满足普通操作。可设置范围为128KB至4GB，默认值为192KB
thread_stack = 256k

#table_cache表示表高速缓存的大小。当MySQL访问一个表时，如果在MySQL表缓冲区中还有空间，那么这个表就被打开并放入表缓冲区，这样做的好处是可以更快速地访问表中的内容。一般来说，可以通过查看数据库运行峰值时间的状态值Open_tables和Opened_tables，用以判断是否需要增加table_cache值，即open_tables接近table_cache的时候，并且Opened_tables这个值在逐步增加，那就要考虑增加这个值的大小了
table_cache = 614k

# 查询排序时所能使用的缓冲区大小，系统默认大小为2MB。从5.1.23版本开始，在除了Windows之外的64位平台上可以超出4GB的限制
sort_buffer_size = 6M

# 注意该参数对应的分配内存是每个连接独占的，如果有100个连接，那么实际分配的总排序缓冲区大小为100× 6MB=600MB。所以，对于内存在4GB左右的服务器来说，推荐将其设置为6MB～8MB。
```

---

```bash
# 读查询操作所能使用的缓冲区大小。和sort_buffer_size一样，该参数对应的分配内存也是每个连接独享
read_buffer_size = 4M

# 联合查询操作所能使用的缓冲区大小，和sort_buffer_size一样，该参数对应的分配内存也是每个连接独享。
# 设置在REPAIR TABLE或用CREATE INDEX创建索引或ALTER TABLE的过程中排序索引所分配的缓冲区大小，可设置范围4M至4GB，默认为8MB
join_buffer_size = 64M

# 设置Thread Cache池中可以缓存的连接线程最大数量，可设置为0至16384，默认为0。这个值表示可以重新利用保存在缓存中线程的数量，当断开连接时，如果缓存中还有空间，那么客户端的线程将被放到缓存中。如果线程重新被请求，那么请求将从缓存中读取；如果缓存中是空的或者是新的请求，那么这个线程将被重新创建；如果有很多新的线程，增加这个值可以改善系统性能。通过比较Connections和Threads_created状态的变量，可以看到这个变量的作用。我们可以根据物理内存设置规则如下：1G内存配置为8,2G内存配置为16,3G内存配置为32,4G或4G以上配置为64或更大的数值。
myisam_sort_buffer_size = 64M

# 指定MySQL查询缓冲区的大小。可以通过在MySQL控制台观察，如果Qcache_lowmem_prunes的值非常大，则表明经常出现缓冲不够的情况；如果Qcache_hits的值非常大，则表明查询缓冲使用得非常频繁。另外，如果该值较小反而会影响效率，那么可以考虑不用查询缓冲。对于Qcache_free_blocks，如果该值非常大，则表明缓冲区中碎片很多。
query_cache_size = 64M

# 设置内存临时表最大值。如果超过该值，则会将临时表写入到磁盘，其范围为1KB到4GB。
tmp_table_size = 256M

# 指定MySQL允许的最大连接进程数。如果在访问论坛时经常出现Too Many Connections的错误提示，则需要增大该参数值。
max_connections = 768

# 设置每个主机的连接请求异常中断的最大次数，当超过该次数，MySQL服务器将禁止host的连接请求，直到MySQL服务器重启或通过flush hosts命令清空此host的相关信息，此值可设置为1至4G，默认为10
max_connect_errors = 1000 

# 指定一个请求的最大连接时间，对于4GB左右内存的服务器来说，可以将其设置为5～10。
wait_timeout = 10

# 该参数取值为服务器逻辑CPU数量乘以2，在本例中，服务器有2个物理CPU，而每个物理CPU又支持H.T超线程，所以实际取值为4× 2=8，这也是目前双四核主流服务器的配置。
thread_concurrency = 8

# 开启该选项可以彻底关闭MySQL的TCP/IP连接方式，如果Web服务器是以远程连接的方式访问MySQL数据库服务器，不要开启该选项，否则将无法正常连接。
skip-networking

# 物理内存越大，设置就越大。默认为2402，调整到512～1024之间最佳。
table_cache = 1024

# 默认为2MB。
innodb_additional_mem_pool_size = 4M

# 设置为0就是等到innodb_log_buffer_size列队满后再统一储存，默认为1。
innodb_flush_log_at_trx_commit = 1 

# 默认为1MB。
innodb_log_buffer_size = 2M

# 服务器有几个CPU就设置为几，建议用默认设置，一般为8。
innodb_thread_concurrency = 8

# 默认设置内存临时表最大值。如果超过该值，则会将临时表写入到磁盘，设置范围为1KB至4GB。
tmp_table_size = 64M

# read_rnd_buffer_size是设置进行随机读的时候所使用的缓冲区。此参数和read_buffer_size设置的Buffer相反，一个是顺序读的时候使用，另一个是随机读的时候使用。但两者都是针对于线程的设置，每个线程都可以产生两种Buffer中的任何一个。read_rnd_buffer_size的默认值为256KB，最大值为4GB。值得注意的是，如果key_reads太大，则应该把my.cnf中的key_buffer_size变大，保持key_reads/key_read_requests至少在1/100以上，越小越好；如果qcache_lowmem_prunes很大，就应增加query_cache_size的值。
read_rnd_buffer_size = 16M

```

很多时候需要具体情况具体分析，其他参数的变更可以等MySQL稳定上线一段时间后根据status值进行调整。


## 根据status状态进行适当优化

```mysql
SHOW GLOBAL STATUS;
```

### 慢查询

定位系统中效率比较低下的query语句，需要打开慢查看日志，也就是slow query log，查询慢查询日志的相关命令如下：

```mysql
mysql> show variables like '%slow%';
+---------------------------+---------------------------------+
| Variable_name             | Value                           |
+---------------------------+---------------------------------+
| log_slow_admin_statements | OFF                             |
| log_slow_slave_statements | OFF                             |
| slow_launch_time          | 2                               |
| slow_query_log            | OFF                             |
| slow_query_log_file       | /var/lib/mysql/mysql57-slow.log |
+---------------------------+---------------------------------+
5 rows in set (0.02 sec)

mysql> show global status like '%slow%';
+---------------------+-------+
| Variable_name       | Value |
+---------------------+-------+
| Slow_launch_threads | 0     |
| Slow_queries        | 0     |
+---------------------+-------+
2 rows in set (0.02 sec)

mysql>
```

打开慢查询日志可能会对系统性能有一点影响，如果是MySQL主从接口，可以考虑打开其中一台从服务器慢查询日志，这样既能可以监控慢查询还会对系统性能影响降低到最小。另外，还可以用MySQL自带命令mysqldumpslow进行查询，如下面的命令就可以查询访问次数最多的20条sql语句；

```mysql
mysqldumpslow -s c -t 20 host-slow.log
```

### 连接数

如果经常遇见“MySQL: ERROR 1040: Too manyconnections”的问题，一种情况是访问量确实很高，MySQL服务器抗不住，这个时候需要考虑增加从服务器分散读压力；另外一种情况是MySQL配置文件中max_connections的值过小。

```mysql
mysql> show variables like 'max_connections';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 151   |
+-----------------+-------+
1 row in set (0.02 sec)

mysql>
```

可以从上面的查询中看出，当前MySQL服务器最大的连接数是151，接来下查询一下该服务器的最大连接数。

```mysql
mysql> show variables like 'Max_used_connections';
Empty set (0.02 sec)

mysql>
```

窝草，这破服务器居然没有查询到！在这里说明下，该值最理想的设置是：

```mysql
Max_used_connections / max_connections * 100% ≈ 85%
```

最大连接数占上限连接数的85%左右，如果发现比例在10%以下，则说明MySQL服务器连接数的上限设置过高。

### Key_buffer_size

```mysql
mysql> show variables like 'key_buffer_size';
+-----------------+---------+
| Variable_name   | Value   |
+-----------------+---------+
| key_buffer_size | 8388608 |
+-----------------+---------+
1 row in set (0.03 sec)

mysql>
```

从上面的查询可以看出，系统分配了8M内存给key_buffer_size，接下在查询一下key_buffer_size的使用情况：

```mysql
mysql> show global status like 'key_read%';
+-------------------+---------+
| Variable_name     | Value   |
+-------------------+---------+
| Key_read_requests | 9587016 |
| Key_reads         | 395     |
+-------------------+---------+
2 rows in set (0.03 sec)

mysql>
```

可以看出，一共有9587016个索引读取请求，有395个请求在内存中没有找到，直接从硬盘读取索引，计算索引未命中缓存的概率：key_cache_miss_rate = Key_reads / Key_read_requests * 100%

MySQL服务器还提供了key_blocks_*参数:

```mysql
mysql> show global status like 'key_blocks_u%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Key_blocks_unused | 6580  |
| Key_blocks_used   | 133   |
+-------------------+-------+
2 rows in set (0.02 sec)

mysql>
```

Key_blocks_unused表示未使用的缓存簇（blocks）数，Key_blocks_used表示曾经用到的最大的blocks数，比如这台服务器，所有的缓存都用到了，要么增加key_buffer_size，要么就是过渡索引，把缓存占满了。比较理想的设置是：Key_blocks_used/（Key_blocks_unused +Key_blocks_used)*100%≈80%

### 临时表

当执行语句时，关于已经被创造的隐含临时表的数量，我们可以用如下命令查知其具体情况：

```mysql
mysql> show global status like 'created_tmp%';
+-------------------------+--------+
| Variable_name           | Value  |
+-------------------------+--------+
| Created_tmp_disk_tables | 50971  |
| Created_tmp_files       | 8      |
| Created_tmp_tables      | 150652 |
+-------------------------+--------+
3 rows in set (0.03 sec)

mysql>
```

每次创建临时表时，Created_tmp_tables都会增加，如果是在磁盘上创建临时表，Created_tmp_disk_tables也会增加。Created_tmp_files表示MySQL服务创建的临时文件数，比较理想的配置是：Created_tmp_disk_tables/Created_tmp_tables*100%<=25%。

比如上面的服务器Created_tmp_disk_tables/Created_tmp_tables*100%=33%，有点超了啊，窝草！我们再看一下MySQL服务器对临时表的配置：

```mysql
mysql> show variables where Variable_name in('tmp_table_size','max_heap_table_size');
+---------------------+----------+
| Variable_name       | Value    |
+---------------------+----------+
| max_heap_table_size | 16777216 |
| tmp_table_size      | 16777216 |
+---------------------+----------+
2 rows in set (0.03 sec)

mysql>
```

从上面的数据中可以看出，有16M的临时表才能放在内存中，超过的就会放到硬盘临时表！好像上面已经满了啊。。。

### Open Table的情况

Open_tables表示打开表的数量，Opened_tables表示打开过的表数量，可以用如下命令查看其具体情况：

```mysql
mysql> show global status like 'open%tables%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Open_tables   | 2000  |
| Opened_tables | 96924 |
+---------------+-------+
2 rows in set (0.02 sec)

mysql>
```
如果Opened_tables数量过大，说明配置中table_cache（MySQL5.1.3之后这个值叫做table_open_cache）的值可能太小，我们查询一下服务器table_cache值：

```mysql
mysql> show global status like 'table_cache';
Empty set (0.02 sec)

mysql>
```

啊，没设置！ 那么设置多少比较合适呢。

```bash
Open_tables / Opened_tables * 100% >= 85%
Open_tables / table_cache * 100% <= 95%
```


### 进程使用情况

如果我们在MySQL服务器的配置文件中设置了thread_cache_size，当客户端断开之时，服务器处理此客户请求的线程将会缓存起来以响应下一个客户而不是销毁（前提是缓存数未达上限）。Threads_created表示创建过的线程数，可以用如下命令查看：

```mysql
mysql> show global status like 'Thread%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Threads_cached    | 8     |
| Threads_connected | 1     |
| Threads_created   | 98    |
| Threads_running   | 1     |
+-------------------+-------+
4 rows in set (0.02 sec)

mysql>
```

如果发现Threads_created的值过大，表明MySQL服务器一直在创建线程，这也是比较耗费资源的，可以适当增大配置文件中thread_cache_size的值，查询服务器thread_cache_size配置

```mysql
mysql> show global status like 'thread_cache_size';
Empty set (0.03 sec)

mysql>
```

哎呀，还是没设置！窝草。。。

###  查询缓存（query cache）

qrery_cache_size和query_cache_type。其中query_cache_size设置MySQL的Query Cache大小，query_cache_type设置使用查询缓存的类型，可以用如下命令查看其具体情况：

```mysql
mysql> show global status like 'qcache%';
+-------------------------+---------+
| Variable_name           | Value   |
+-------------------------+---------+
| Qcache_free_blocks      | 1       |
| Qcache_free_memory      | 1031832 |
| Qcache_hits             | 0       |
| Qcache_inserts          | 0       |
| Qcache_lowmem_prunes    | 0       |
| Qcache_not_cached       | 1731555 |
| Qcache_queries_in_cache | 0       |
| Qcache_total_blocks     | 1       |
+-------------------------+---------+
8 rows in set (0.03 sec)

mysql>
```

MySQL查询缓存变量的相关解释如下所示：

- Qcache_free_blocks：缓存中相邻内存块的个数，数目大说明可能有碎片。FLUSH QUERYCACHE会对缓存中的碎片进行整理，从而得到一个空闲块。
- Qcache_free_memory：缓存中的空闲内存。
- Qcache_hits：表示有多少次命中。通过这个参数可以查看到Query Cache的基本效果。
- Qcache_inserts：每插入一个查询时就会增大。命中次数除以插入次数就是不中比率。
- Qcache_lowmem_prunes：表示有多少条Query因为内存不足而被清除出Query Cache。通过“Qcache_lowmem_prunes”和“Qcache_free_memory”相互结合，能够更清楚地了解系统中Query Cache的内存大小是否真的足够，是否频繁出现因为内存不足而有Query被换出的情况。
- Qcache_not_cached：不适合进行缓存的查询数量，通常是由于这些查询不是SELECT语句或用了now()之类的函数。
- Qcache_queries_in_cache：当前缓存的查询（和响应）数量。
- Qcache_total_blocks：缓存中块的数量。

我们再查询一下服务器上关于query_cache的配置：

```mysql

mysql> show variables like 'query_cache%';
+------------------------------+---------+
| Variable_name                | Value   |
+------------------------------+---------+
| query_cache_limit            | 1048576 |
| query_cache_min_res_unit     | 4096    |
| query_cache_size             | 1048576 |
| query_cache_type             | OFF     |
| query_cache_wlock_invalidate | OFF     |
+------------------------------+---------+
5 rows in set (0.02 sec)

mysql>
```

各字段的解释如下所示：
- query_cache_limit：超过此大小的查询将不缓存。
- query_cache_min_res_unit：缓存块的最小值。
- query_cache_size：查询缓存大小。
- query_cache_type：缓存类型，决定缓存什么样的查询，示例中表示不缓存selectsql_no_cache查询。
- query_cache_wlock_invalidate：表示当有其他客户端正在对MyISAM表进行写操作时，读请求是要等WRITE LOCK释放资源后再查询还是允许直接从Query Cache中读取结果，默认为FALSE（可以直接从Query Cache中取得结果）。

query_cache_min_res_unit的配置是一柄“双刃剑”，默认是4KB，设置值大对大数据查询有好处，但如果都是小数据查询，就容易造成内存碎片和浪费。

查询缓存碎片率=Qcache_free_blocks / Qcache_total_blocks * 100%

如果查询缓存碎片率超过20%，可以用FLUSH QUERY CACHE整理缓存碎片，或者如果你的查询都是小数据量，尝试减小query_cache_min_res_unit。

查询缓存利用率=（query_cache_size-Qcache_free_memory）/query_cache_size*100%

查询缓存利用率在25%以下说明query_cache_size设置过大，可适当减小；查询缓存利用率在80%以上且Qcache_lowmem_prunes＞50则说明query_cache_size可能过小，或是碎片太多。

查询缓存命中率=（Qcache_hits-Qcache_inserts）/Qcache_hits*100%

示例服务器中的查询缓存碎片率等于20.46%，查询缓存利用率等于62.26%，查询缓存命中率等于1.94%，说明命中率很差，可能写操作比较频繁，而且可能存在碎片。

### 排序使用情况

表示系统中对数据进行排序时使用的Buffer，我们可以用如下命令查看：

```mysql
mysql> show global status like 'sort%';
+-------------------+---------+
| Variable_name     | Value   |
+-------------------+---------+
| Sort_merge_passes | 1       |
| Sort_range        | 164197  |
| Sort_rows         | 4778842 |
| Sort_scan         | 99132   |
+-------------------+---------+
4 rows in set (0.02 sec)

mysql>
```

Sort_merge_passes包括如下步骤：MySQL首先会尝试在内存中排序，使用的内存大小由系统变量sort_buffer_size决定，如果它不够大，则把所有的记录都读到内存中，而MySQL则会把每次在内存中排序的结果存到临时文件中，等MySQL找到所有记录之后，再把临时文件中的记录做一次排序，这次再排序就会增加sort_merge_passes。实际上，MySQL会用另一个临时文件来存储再次排序的结果，所以我们通常会看到sort_merge_passes增加的数值是创建临时文件数的两倍。因为用到了临时文件，所以速度可能会较慢，增大sort_buffer_size会减少sort_merge_passes和创建临时文件的次数，但盲目地增大sort_buffer_size并不一定能提高速度。

### 文件打开数（open_files）

我们在处理MySQL故障时发现，当open_files大于open_files_limit值时，MySQL数据库就会发生卡住的现象，导致Apache服务器也打不开相应页面，大家在工作中要注意这个问题，我们可以用如下命令查看其具体情况：

```mysql
mysql> show global status like 'open_files%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Open_files    | 158   |
+---------------+-------+
1 row in set (0.02 sec)

mysql> show variables like 'open_files_limit%';
+------------------+---------+
| Variable_name    | Value   |
+------------------+---------+
| open_files_limit | 1048576 |
+------------------+---------+
1 row in set (0.02 sec)

mysql>
```

比较合适的设置是：Open_files/open_files_limit*100%<=75%

### Innodb_buffer_pool_size的合理设置

InnoDB存储引擎的缓存机制和MyISAM的最大区别在于，InnoDB不仅仅缓存索引，同时还会缓存实际的数据。此参数用于设置InnoDB最主要的buffer（InnoDB buffer pool）大小，也就是用户表及索引数据的最主要缓存空间，对InnoDB整体性能的影响也最大。

MySQL官方手册以及网络上许多人分享的InnoDB优化建议，都是建议简单地将此值设置为整个系统物理内存的50%～80%之间。待业务上线后，在根据实际的运行场景来正确设置此项参数(配置Innodb_buffer_pool_size = 2MB)。通过以下命令观察：

```mysql
mysql> show status like 'Innoad_buffer_pool_%';
Empty set (0.02 sec)

mysql>
```

额，这破服务器居然没有查询到，那就简单说下把，InnoDB buffer pool的read命中率：
(Innodb_buffer_pool_read_requests - Innodb_buffer_pool_reads) / Innodb_buffer_pool_read_requests


write命中率： Innodb_buffer_pool_pages_data / Innodb_buffer_pool_page_total