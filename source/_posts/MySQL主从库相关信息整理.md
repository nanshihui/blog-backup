---
title: MySQL主从库相关信息整理
date: 2016-04-18 21:41:21
categories:
- 运维
tag:
- MySQL
---
<h2 id="intro">前言</h2>仅记录在主从库遇到的坑
### 相关的配置方式
- **服务创建:** 基本配置；
- **注意事项:** 一些细节注意的地方；
- **主从原理:** 基本的原理解析；

<!-- more -->
## 服务创建
  需要主库以及从库
### 相关步骤
* 创建用户

``` bash
        GRANT REPLICATION SLAVE ON *.* TO 'slave'@'localhost' IDENTIFIED BY '123456'  
```

* 修改MySQL配置文件

a)master
    
``` bash
# vi /etc/my.cnf   
    =============   
    [mysqld]   
    server-id = 1   
    log_bin = mysql-bin   
    binlog-do-db = test # 多个写多行   
    binlog-ignore-db = mysql #多个写多行   
    max_binlog_size = 500M
    binlog_cache_size = 128K
    log-slave-updates
    expire_logs_day=2
    binlog_format="MIXED"
    =============   

```

b)slave，配置每个slave

``` bash
    # vi /etc/my.cnf   
    [mysqld]   
    server-id = 2   
    log_bin = mysql-bin   
    replicate-do-db = test   
    replicate-ignore-db = mysql   
    replicate-ignore-db = information_schema   
    slave-skip-errors=1007,1008,1053,1062,1213,1158,1159
    master-info-file = /home/mysql/logs/master.info
    relay-log = /home/mysql/logs/relay-bin
    relay-log-index = /home/mysql/logs/relay-bin.index
    relay-log-info-file = /home/mysql/logs/relay-log.info
    =============   
         
```

* 启动slave后执行(各台slave操作类似)

``` bash
    CHANGE MASTER TO MASTER_HOST = ’127.0.0.1', MASTER_USER = 'slave', MASTER_PASSWORD = '123456', MASTER_LOG_FILE = 'mysql-bin.000001', MASTER_LOG_POS = 107; 
    //（通过在master上SHOW MASTER STATUS 来得到MASTER_LOG_FILE和MASTER_LOG_POS的值）   
    SLAVE START;   
    SHOW SLAVE STATUS ; //查看是否同步上   
```

* 登录master，增删改数据后看下各slave是否正常同步到

## 注意事项
　　在同步时候，有时候会发现经常1062的错误，可以在配置的时候，添加配置项skip-error项。如果修改了连接主库相关信息，重启之前一定要删除master.info文件，否则重启之后由于连接信息改变从库而不会自动连接主库，造成同步失败。此文件是保存连接主库信息的。

## 主从原理
### Replication 线程
　　Mysql的 Replication 是一个异步的复制过程（mysql5.1.7以上版本分为异步复制和半同步两种模式），从一个 Mysql instace(我们称之为 Master)复制到另一个Mysql instance(我们称之 Slave)。在 Master 与 Slave 之间的实现整个复制过程主要由三个线程来完成，其中两个线程(Sql线程和IO线程)在 Slave端，另外一个线程(IO线程)在 Master 端。要实现 MySQL 的 Replication ，首先必须打开 Master 端的Binary Log(mysql-bin.xxxxxx功能，否则无法实现。因为整个复制过程实际上就是Slave从Master端获取该日志然后再在自己身上完全 顺序的执行日志中所记录的各种操作。打开 MySQL 的 Binary Log 可以通过在启动 MySQL Server 的过程中使用 “—log-bin” 参数选项，或者在 my.cnf 配置文件中的 mysqld 参数组([mysqld]标识后的参数部分)增加 “log-bin” 参数项。

### MySQL 复制的基本过程如下
　　1. Slave 上面的IO线程连接上 Master，并请求从指定日志文件的指定位置(或者从最开始的日志)之后的日志内容；
　　2. Master 接收到来自 Slave 的 IO 线程的请求后，通过负责复制的 IO 线程根据请求信息读取指定日志指定位置之后的日志信息，返回给 Slave 端的 IO 线程。返回信息中除了日志所包含的信息之外，还包括本次返回的信息在 Master 端的 Binary Log 文件的名称以及在 Binary Log 中的位置；
　　3. Slave 的 IO 线程接收到信息后，将接收到的日志内容依次写入到 Slave 端的Relay Log文件(mysql-relay-bin.xxxxxx)的最末端，并将读取到的Master端的bin-log的文件名和位置记录到master- info文件中，以便在下一次读取的时候能够清楚的高速Master“我需要从某个bin-log的哪个位置开始往后的日志内容，请发给我”
　　4. Slave 的 SQL 线程检测到 Relay Log 中新增加了内容后，会马上解析该 Log 文件中的内容成为在 Master 端真实执行时候的那些可执行的 Query 语句，并在自身执行这些 Query。这样，实际上就是在 Master 端和 Slave 端执行了同样的 Query，所以两端的数据是完全一样的

## 附录
### MySQL 的主从配置文件
相关内容仅供参考:
主库配置文件
``` bash
# Example MySQL config file for medium systems.
#
# This is for a system with little memory (32M - 64M) where MySQL plays
# an important part, or systems up to 128M where MySQL is used together with
# other programs (such as a web server)
#
# You can copy this file to
# /etc/my.cnf to set global options,
# mysql-data-dir/my.cnf to set server-specific options (in this
# installation this directory is C:\mysql\data) or
# ~/.my.cnf to set user-specific options.
#
# In this file, you can use all long options that a program supports.
# If you want to know which options a program supports, run the program
# with the "--help" option.

# The following options will be passed to all MySQL clients
[client]
port        = 3306

socket = "D:\mysql_main\mysql-5.7.13-winx64"



[mysql]

no-auto-rehash
# Remove the next comment character if you are not familiar with SQL
#safe-updates


[mysqld]
server-id=5
port=3306
log-bin=mysql-bin
binlog-do-db=t_qcpj
binlog_format="MIXED"
socket = "D:/mysql_main/mysql-5.7.13-winx64/mysql.sock"
explicit_defaults_for_timestamp = TRUE
character-set-server=utf8
basedir=D:\mysql_main\mysql-5.7.13-winx64
datadir=D:\mysql_main\mysql-5.7.13-winx64\data

log-output=FILE
general-log=0
general_log_file="WIN-KGGGQSE2NIE.log"
slow-query-log=1
slow_query_log_file="WIN-KGGGQSE2NIE-slow.log"
long_query_time=10
log-error="mysqlerr.log"
ft_min_word_len=1
pid_file  = "mysql.pid"
default-storage-engine=INNODB
max_binlog_size=500M
binlog_cache_size=128K
binlog_ignore_db=mysql
log-slave-updates
max_connections=151
query_cache_size=0
tmp_table_size=310M
table_open_cache=2000
thread_cache_size=10
myisam_max_sort_file_size=100G
sql-mode="STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
myisam_sort_buffer_size=608M
key_buffer_size=8M
read_buffer_size=64K
read_rnd_buffer_size=256K
sort_buffer_size=256K

innodb_flush_log_at_trx_commit=1
innodb_log_buffer_size=1M
innodb_buffer_pool_size=8M
innodb_log_file_size=48M
innodb_thread_concurrency=17
innodb_autoextend_increment=64
innodb_buffer_pool_instances=8

innodb_concurrency_tickets=5000
innodb_old_blocks_time=1000
innodb_open_files=300
innodb_stats_on_metadata=0
innodb_file_per_table=1
innodb_checksum_algorithm=0
back_log=80
flush_time=0
join_buffer_size=256K
max_allowed_packet=4M
max_connect_errors=100
open_files_limit=4161
query_cache_type=0
table_definition_cache=1400
binlog_row_event_max_size=8K
sync_master_info=10000

# If the value of this variable is greater than 0, the MySQL server synchronizes its relay log to disk.
# (using fdatasync()) after every sync_relay_log writes to the relay log.
sync_relay_log=10000

# If the value of this variable is greater than 0, a replication slave synchronizes its relay-log.info file to disk.
# (using fdatasync()) after every sync_relay_log_info transactions.
sync_relay_log_info=10000

# Load mysql plugins at start."plugin_x ; plugin_y".
# plugin_load

# MySQL server's plugin configuration.
# loose_mysqlx_port=33060

[mysqldump]
quick
max_allowed_packet = 16M

[isamchk]
key_buffer       = 20M
sort_buffer_size = 20M
read_buffer      = 2M
write_buffer     = 2M

[myisamchk]
key_buffer       = 20M
sort_buffer_size = 20M
read_buffer      = 2M
write_buffer     = 2M

[mysqlhotcopy]
interactive-timeout

  
```

从库配置文件

``` bash
[client]
port        = 3307
socket = "D:\mysql_sub\mysql-5.7.13-winx64"

[client]
user = root 
password =China1000
[mysql]
no-auto-rehash

[mysqld]
server-id =3
log-bin=mysql-bin
replicate-do-db=t_qcpj
replicate-ignore-db=mysql
slave-skip-errors=1007,1008,1053,1062,1213,1158,1159
binlog_format=mixed
port=3307
socket = "D:/mysql_sub/mysql-5.7.13-winx64/mysql.sock"
explicit_defaults_for_timestamp = TRUE
basedir=D:/mysql_sub/mysql-5.7.13-winx64
log-error="mysqlerr.log"
datadir=D:/mysql_sub/mysql-5.7.13-winx64/data
pid_file  = "mysql.pid"
default-storage-engine=INNODB
max_binlog_size=500M
binlog_cache_size=128K
binlog_ignore_db=mysql
log-slave-updates
ft_min_word_len=1
max_connections=151
query_cache_size=0
tmp_table_size=310M
table_open_cache=2000
thread_cache_size=10
myisam_max_sort_file_size=100G
sql-mode="STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
myisam_sort_buffer_size=608M
key_buffer_size=8M
read_buffer_size=64K
read_rnd_buffer_size=256K
sort_buffer_size=256K

innodb_flush_log_at_trx_commit=1
innodb_log_buffer_size=1M
innodb_buffer_pool_size=8M
innodb_log_file_size=48M
innodb_thread_concurrency=17
innodb_autoextend_increment=64
innodb_buffer_pool_instances=8

innodb_concurrency_tickets=5000
innodb_old_blocks_time=1000
innodb_open_files=300
innodb_stats_on_metadata=0
innodb_file_per_table=1
innodb_checksum_algorithm=0
back_log=80
flush_time=0
join_buffer_size=256K
max_allowed_packet=4M
max_connect_errors=100
open_files_limit=4161
query_cache_type=0
table_definition_cache=1400
binlog_row_event_max_size=8K
sync_master_info=10000

# If the value of this variable is greater than 0, the MySQL server synchronizes its relay log to disk.
# (using fdatasync()) after every sync_relay_log writes to the relay log.
sync_relay_log=10000

# If the value of this variable is greater than 0, a replication slave synchronizes its relay-log.info file to disk.
# (using fdatasync()) after every sync_relay_log_info transactions.
sync_relay_log_info=10000

# Load mysql plugins at start."plugin_x ; plugin_y".
# plugin_load

# MySQL server's plugin configuration.
# loose_mysqlx_port=33060
[mysqldump]
quick
max_allowed_packet = 16M

[isamchk]
key_buffer       = 20M
sort_buffer_size = 20M
read_buffer      = 2M
write_buffer     = 2M

[myisamchk]
key_buffer       = 20M
sort_buffer_size = 20M
read_buffer      = 2M
write_buffer     = 2M

[mysqlhotcopy]
interactive-timeout
```