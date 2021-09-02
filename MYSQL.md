# 源码编译安装
```shell
yum install lrzsz bison biso-devel zlib-devel libcurl-devel libarchive-devel boost-devel gcc gcc-c++ cmake ncurses-devel gnutls-devel libxml2-devel openssl-devel libevent-devel libaio-devel  -y 

groupadd mysql
useradd -r -g mysql -s /sbin/nologin mysql
 
rm -f CMakeLists.txt
创建数据存储目录
mkdir -p /data/mysql /apps/mysql

预编译参数
cmake . -DCMAKE_INSTALL_PREFIX=/apps/mysql \
-DMYSQL_DATADIR=/data/mysql \
-DSYSCONFDIR=/apps/mysql \
-DEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DENABLED_LOCAL_INFILE=1 

echo PATH=/apps/mysql/bin:$PATH > /etc/profile.d/mysql.sh  
source /etc/profile.d/mysql.sh  

mysql_install_db --datadir=/data/mysql --user=mysql #初始化数据库  
mysqld --initialize --user=mysql --basedir=/apps/mysql --datadir=/data/mysql 2>&1 | tee /data/mysql_init.log
```

# mysql修改用户密码

```shell
mysqladmin -u USERNAME -h HSOTNAME password 'new_passswd' -p
mysql>set password for 'root'@'localhost'=PASSWORD('123456');
mysql>UPDATE mysql.user set PASSWORD=PASSWORD('123456') WHERE user='root';
mysql>flush privileges;
```



# 二进制安装mysql

```shell
#安装开始
groupadd mysql && useradd -r -g mysql -s /sbin/nologin mysql
cd /
[root@mysql ~]# tar xf mysql-5.7.35-el7-x86_64.tar.gz -C / 
[root@mysql ~]# mv /mysql-5.7.35-el7-x86_64/ /mysql
[root@mysql /]# mkdir -pv /data/ && mkdir -pv /mysql/{logs,binlog}
[root@mysql /]# chown -R mysql:mysql /data
[root@mysql /]# chown -R mysql:mysql /mysql
[root@mysql /]# echo "PATH=/mysql/bin:$PATH" > /etc/profile.d/mysql.sh
[root@mysql /]# source /etc/profile.d/mysql.sh
[root@srv01 mysql]# cat /etc/my.cnf 
[client]
port=3306
socket=/mysql/mysql.sock

[mysqld]
server-id = 1
port=3306
user = mysql
basedir = /mysql
datadir = /data
pid-file= /mysql/mysql.pid
socket= /mysql/mysql.sock
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci
init_connect='SET NAMES utf8mb4'
explicit_defaults_for_timestamp = ON
log_timestamps = SYSTEM
skip-name-resolve
log-error = /mysql/logs/error.log
log-bin= /mysql/binlog/mysql-bin.log

[root@mysql /]# mysqld --initialize  --user=mysql --basedir=/mysql --datadir=/data
[root@mysql /]# mysqld --defaults-file=/etc/my.cnf --initialize 
[root@mysql /]# cp /mysql/support-files/mysql.server /etc/init.d/mysql.server  
[root@mysql /]# /etc/init.d/mysql.server  start


  

忘记密码修改
skip-grant-tables#在/etc/my.cnf中添加
update mysql.user set authentication_string=password('123456') where user='root'; #修改密码后重启mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456'
flush privileges;
```

# systemd启动mysql

```shell
添加一个系统服务单元配置文件，其中包含有关MySQL服务的详细信息。该文件被命名 mysqld.service并放置在中 /usr/lib/systemd/system。
shell> cd /usr/lib/systemd/system
shell> touch mysqld.service
shell> chmod 644 mysqld.service
将此配置信息添加到 mysqld.service文件中：
[Unit]
Description=MySQL Server
Documentation=man:mysqld(7)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql

Type=forking

PIDFile=/data/mysql/mysql.pid

# Disable service start and stop timeout logic of systemd for mysqld service.
TimeoutSec=0

# Start main service
ExecStart=/data/mysql/bin/mysqld --defaults-file=/etc/my.cnf --daemonize  --pid-file=/data/mysql/mysql.pid   

# Use this to switch malloc implementation
EnvironmentFile=-/etc/sysconfig/mysql

# Sets open_files_limit
LimitNOFILE = 5000

Restart=on-failure

RestartPreventExitStatus=1

PrivateTmp=false


[root@Centos7 system]# systemctl enable mysqld
Created symlink from /etc/systemd/system/multi-user.target.wants/mysqld.service to /usr/lib/systemd/system/mysqld.service.
[root@Centos7 system]# systemctl start mysqld
[root@Centos7 system]# ps -ef | grep mysql
mysql      1672      1  4 11:15 ?        00:00:00 /data/mysql/bin/mysqld --defaults-file=/etc/my.cnf --daemonize --pid-file=/data/mysql/mysql.pid
root       1701   1256  0 11:15 pts/0    00:00:00 grep --color=auto mysql
[root@Centos7 system]# 
[root@Centos7 system]# systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2020-09-04 11:15:43 CST; 22s ago
     Docs: man:mysqld(7)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 1670 ExecStart=/data/mysql/bin/mysqld --defaults-file=/etc/my.cnf --daemonize --pid-file=/data/mysql/mysql.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
 Main PID: 1672 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─1672 /data/mysql/bin/mysqld --defaults-file=/etc/my.cnf --daemonize --pid-file=/data/mysql/mysql.pid

Sep 04 11:15:43 Centos7 systemd[1]: Starting MySQL Server...
Sep 04 11:15:43 Centos7 systemd[1]: Started MySQL Server.
[root@Centos7 system]# systemctl stop mysqld
[root@Centos7 system]# systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:mysqld(7)
           http://dev.mysql.com/doc/refman/en/using-systemd.html

Sep 04 11:15:43 Centos7 systemd[1]: Starting MySQL Server...
Sep 04 11:15:43 Centos7 systemd[1]: Started MySQL Server.
Sep 04 11:16:13 Centos7 systemd[1]: Stopping MySQL Server...
Sep 04 11:16:15 Centos7 systemd[1]: Stopped MySQL Server.
```

# 配置主从复制

master：CentOS7 10.10.20.105

slave：CentOS7 10.10.20.104

Server version:         5.7.23-log MySQL Community Server (GPL)

## 1.修改master端配置文件/etc/my.cnf

```shell
#server-id给数据库服务的唯一标识
server-id=1

#log-bin设置此参数表示启用binlog功能，并指定路径名称
log-bin=/var/lib/mysql/mysql-bin
sync_binlog=0
#设置日志的过期天数
expire_logs_days=7
#binlog_cache_size此参数表示binlog使用的内存大小
binlog_cache_size=1M
```

## 2.在master端上建立SLAVE账户并授权从库

```shell
#master端操作
mysql> GRANT REPLICATION SLAVE ON *.* TO 'slave'@'10.10.20.104' IDENTIFIED BY '111111';
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> 
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```

## 3.master端重启mysql

```shell
 systemctl restart mysqld
```

## 4.修改slave端配置文件/etc/my.cnf

```shell
#server-id给数据库服务的唯一标识
server-id=2
#read_only设置数据库为只读，防止从库数据修改后，主从数据不一致，但是有Super权限的账号还是有写的权限，所以要某个账号只读的话，可以去掉账号的Super权限
read_only=1
#binlog_cache_size此参数表示binlog使用的内存大小
binlog_cache_size=1M
relay-log=/data/mysql/relaylog/mysql-relay-bin.log
```

## 5.slave端重启mysql

```shell
 systemctl restart mysqld
```

## 6.查看master端binlog位置点

```shell
mysql>  show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000037 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql>
```



## 7.在slave端配置连接master端

```shell
mysql> CHANGE MASTER TO MASTER_HOST='10.10.20.105', MASTER_USER='slave',MASTER_PASSWORD='111111',MASTER_LOG_FILE='mysql-bin.000002',MASTER_LOG_POS=154;
Query OK, 0 rows affected, 2 warnings (0.01 sec)
mysql> 
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
```



## 8.在slave端启动主从复制

```shell
mysql> 
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.10.20.105
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000037
          Read_Master_Log_Pos: 154
               Relay_Log_File: db02-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000037
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:  
```

## MYSQL服务器命令

```shell
mysql> help create index
[root@mysql mysql]# mysqladmin create hellodb
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| hellodb            |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql>Bye
[root@mysql mysql]# mysqladmin ping 
mysqld is alive
[root@mysql mysql]# mysqladmin processlist
+----+------+-----------+----+---------+------+----------+------------------+
| Id | User | Host      | db | Command | Time | State    | Info             |
+----+------+-----------+----+---------+------+----------+------------------+
| 10 | root | localhost |    | Query   | 0    | starting | show processlist |
+----+------+-----------+----+---------+------+----------+------------------+
[root@mysql mysql]# mysqladmin status
Uptime: 1728  Threads: 1  Questions: 106  Slow queries: 0  Opens: 115  Flush tables: 1  Open tables: 108  Queries per second avg: 0.061
[root@mysql mysql]# mysqladmin extended-status  #状态变量
[root@mysql mysql]# mysqladmin variables #服务器变量
[root@mysql mysql]# mysqladmin flush-privileges | mysqladmin reload
[root@mysql mysql]# mysqladmin flush-status #重置服务器变量
[root@mysql mysql]# mysqladmin flush-logs #日志滚动（二进制和中继日志）
[root@mysql mysql]# mysqladmin refresh #mysqladmin flush-hosts && mysqladmin flush-logs
[root@mysql mysql]# mysqladmin version
[root@mysql mysql]# mysqladmin start-slave | mysqladmin stop-slave #启动从服务器复制线程
[root@mysql mysql]# mysql -A -e "show databases;"
[root@mysql mysql]# mysql -A -e "show global variables like 'sql%'"
[root@mysql mysql]# mysql -A -e "show session variables;"
mysql> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)

mysql>use mysql
mysql> show table status like 'user'\G
*************************** 1. row ***************************
           Name: user
         Engine: MyISAM
        Version: 10
     Row_format: Dynamic
           Rows: 3
 Avg_row_length: 128
    Data_length: 384
Max_data_length: 281474976710655
   Index_length: 4096
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2021-09-02 21:10:30
    Update_time: 2021-09-02 21:11:43
     Check_time: NULL
      Collation: utf8_bin
       Checksum: NULL
 Create_options: 
        Comment: Users and global privileges
1 row in set (0.00 sec)
```

- MyISAM:每张表三个文件。
  - .frm：表结构 
  - .MYD：表数据 
  - .MYI：表索引
- InnoDB：所有表共享一个表空间；
  - .frm：表结构 
  - .ibd:表空间（表数据和表索引

```shell
/etc/my.cnf
/etc/mysql/my.cnf
$MYSQL_HOME/my.cnf
--default-extra-file=/path/to/somefile
~/.my.cnf
```

  











