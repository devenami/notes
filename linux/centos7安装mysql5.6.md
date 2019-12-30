### 环境准备

| 环境   | 版本 |
| ------ | ---- |
| centos | 7    |
| mysql  | 5.7  |

### 必要工具

[MySQL yum源5.7](https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-server-5.7.26-1.el7.x86_64.rpm)

### 开始安装

#### 下载mysql yum源

执行如下命令下载mysql rpm 官网源：

```shell
 wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-server-5.7.26-1.el7.x86_64.rpm
#### 输出
--2019-05-08 03:25:45--  https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-server-5.7.26-1.el7.x86_64.rpm
Resolving cdn.mysql.com (cdn.mysql.com)... 23.209.176.104
Connecting to cdn.mysql.com (cdn.mysql.com)|23.209.176.104|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 173541272 (166M) [application/x-redhat-package-manager]
Saving to: ‘mysql-community-server-5.7.26-1.el7.x86_64.rpm’
100%[=================================================================>] 173,541,272 99.1MB/s   in 1.7s   
2019-05-08 03:25:46 (99.1 MB/s) - ‘mysql-community-server-5.7.26-1.el7.x86_64.rpm’ saved [173541272/173541272]
```

下载完成后会在当前目录下出现名字为：`mysql-community-server-5.7.26-1.el7.x86_64.rpm`的文件。

#### 安装yum源

通过`yum`命令安装mysql源

```shell
yum localinstall mysql-community-server-5.7.26-1.el7.x86_64.rpm
### 输出
Loaded plugins: fastestmirror
Examining mysql-community-server-5.7.26-1.el7.x86_64.rpm: mysql-community-server-5.7.26-1.el7.x86_64
Marking mysql-community-server-5.7.26-1.el7.x86_64.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package mysql-community-server.x86_64 0:5.7.26-1.el7 will be installed
--> Processing Dependency: mysql-community-common(x86-64) = 5.7.26-1.el7 for package: mysql-community-server-5.7.26-1.el7.x86_64
...
```

#### 查看当前源启用的mysql版本

```shell
yum repolist all | grep mysql
### 输出
mysql-connectors-community/x86_64 MySQL Connectors Community   enabled:      108
mysql-connectors-community-source MySQL Connectors Community - disabled
mysql-tools-community/x86_64      MySQL Tools Community        enabled:       90
mysql-tools-community-source      MySQL Tools Community - Sour disabled
mysql55-community/x86_64          MySQL 5.5 Community Server   disabled
mysql55-community-source          MySQL 5.5 Community Server - disabled
mysql56-community/x86_64          MySQL 5.6 Community Server   enabled:      463
mysql56-community-source          MySQL 5.6 Community Server - disabled
mysql57-community-dmr/x86_64      MySQL 5.7 Community Server D disabled
mysql57-community-dmr-source      MySQL 5.7 Community Server D disabled
```

倒数第二列**disable**表示直接通过**yum**进行安装时，会直接忽略对应的项目。**enable**则恰恰相反。

**通过命令启用或禁止源**

```shell
yum-config-manager --disable mysql56-community  # 禁用 mysql56-community
yum-config-manager --enable mysql56-community  # 启用 mysql56-community
```

**通过配置文件启用或禁止源**

`vim /etc/yum.repos.d/mysql-community.repo`

*enabled*为**1**时，表示启用该源。为**0**则表示禁用该源。

```properties
...
# Enable to use MySQL 5.5
[mysql55-community]
name=MySQL 5.5 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.5-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

# Enable to use MySQL 5.6
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
...
```

#### 安装mysql服务

通过`yum`安装mysql server

```shell
yum install mysql-community-server
### 输出
Loaded plugins: fastestmirror
mysql-connectors-community                                           | 2.5 kB  00:00:00
mysql-tools-community                                                | 2.5 kB  00:00:00
mysql56-community                                                    | 2.5 kB  00:00:00
...

Dependencies Resolved
=========================================================================================
 Package				Arch	Version			Repository			Size
=========================================================================================
Installing:
 mysql-community-server	x86_64	5.6.44-2.el7	mysql56-community	60 M

Transaction Summary
=======================================================================================
Install  1 Package

Total download size: 60 M
Installed size: 252 M
Is this ok [y/d/N]: y  # 这里需要确认一下
Downloading packages:
mysql-community-server-5.6.44-2.el7.x86_64.rpm                                      |  60 MB  00:00:01     
...
Installed:
  mysql-community-server.x86_64 0:5.6.44-2.el7          
Complete!
```

### 使用mysql

#### 启动和查看mysql服务状态

```shell
# 启动mysql服务
systemctl start mysqld
# 查看mysql状态
systemctl status mysqld
### 输出
● mysqld.service - MySQL Community Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2019-05-08 03:34:08 EDT; 8s ago
  Process: 2694 ExecStartPost=/usr/bin/mysql-systemd-start post (code=exited, status=0/SUCCESS)
  Process: 2634 ExecStartPre=/usr/bin/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
 Main PID: 2693 (mysqld_safe)
...
```

**Active: active (running)**表示当前服务正在运行。

#### 修改密码

**mysqladmin**可以修改root用户的密码。语法为：`mysqladmin -u用户名 -p原密码 password 新密码`。

由于mysql5.6安装完成后默认密码为空，所有我们可以使用下面的命令来将root用户的密码修改为root

```shell
mysqladmin -u root password root
```

#### 修改mysql的配置

配置文件的路径为：`/etc/my.conf`

查看数据库的编码格式

```sql
mysql> show variables like '%char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)
```

修改默认的编码格式，在配置文件中增加如下内容

```properties
[client]
default-character-set=utf8

[mysqld]
default-storage-engine=INNODB
character-set-server=utf8
collation-server=utf8_general_ci
```

修改指定表的编码格式

```sql
alter table tableName character set utf8;
```

### mysql 核心参数

#### cpu优化

| 参数                      | 参数功能                                                     | 取值范围 | 经验值                 |
| ------------------------- | ------------------------------------------------------------ | -------- | ---------------------- |
| innodb_thread_concurrency | 并发执行的线程的数量（同时干活的线程的数量），保护系统不被hang住 | 0-1000   | 一般要求是cpu核数的4倍 |

#### 内存优化

| 参数                           | 参数功能                                             | 取值范围                                                     | 经验值                                                       |
| ------------------------------ | ---------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| innodb_buffer_pool_size        | 缓存innodb表和索引数据的内存池大小                   | <=5.7.4：非动态全局；>=5.7.5：动态全局；64位：默认128M，最小5M，最大2^64-1 | 一般设置内存的50%~80%，根据实际内存大小设置，如果内存不是很大，可以考虑50~70%，如果内存很大，可以考虑到70%~80%，如果总是产生Innodb_buffer_pool_wait_free，说明buffer_pool设置过小 |
| tmpdir                         | 存放临时文件和临时表目录，可以设置多个路径用：分隔   | 全局参数，非动态，更改需要重启数据库                         | 单独挂载，对读写要求很高，放在高性能盘，独立分区             |
| innodb_buffer_pool_instances   | buffer_pool被分成多少实例                            | 非动态，全局；                                               | 8个或者16个，根据实际buffer pool大小设置，如果实例数量过小，会导致latch争用 |
| innodb_max_dirty_pages_pct     | buffer pool中最大脏页占比                            | 百分比                                                       | 75%~90%，如果io能力足够强，例如使用了闪卡，可以将这个参数调小；该参数设置越小，写入压力越大。 |
| innodb_max_dirty_pages_pct_lwm | 预刷新脏页比例，可以有效控制脏页比例达到最大脏页占比 | 动态，全局；<5.7.4：默认0，范围0~99；>=5.7.5：默认0，范围0~99.99 | 70，控制脏页比率，防止达到脏页最大占比                       |

#### io优化

| 参数                           | 参数功能                                                   | 取值范围                                                     | 经验值                                                       |
| ------------------------------ | ---------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| innodb_io_capacity             | 设定了后台任务执行的io量的上限                             | 动态，全局；64位：默认200，最小100，最大2^64-1               | 每秒后台进程处理IO数据的上限，一般为IO QPS总能力的75%        |
| innodb_io_capacity_max         | 在紧急情况下，innodbio容量上限的最大值，2000是初始默认值。 | 动态，全局；unix-64位：默认是2000，最小100，最大2^32-1       | 根据innodb_io_capacity的2倍进行设置                          |
| innodb_log_files_in_group      | redo日志的组数，即logfile的数量                            | 非动态，全局；默认2个，范围2~100                             | 一般设置5组                                                  |
| innodb_log_file_size           | 每个logfile的大小                                          | \<=5.7.10:默认50M，最小1M，最大512G/innodb_log_files_in_group；>=5.7.11：默认50M，最小4M，最大512G/innodb_log_files_in_group | 一般设置2G，每20~30分钟切换一次redo比较合适， 案例：业务人员反映晚上9:10-9:15有大量交易放弃，交易失败，让dba看看这个时间段数据库的性能情况，dba发现这个时间段有redo切换，buffer写入明显增加（写入抖动），TPS明显降低（事务提交了降低），怀疑redo文件过小，组数过少，脏数据写入速度慢。 解决方案：增加redo数量和大小，加大脏数据的写入力度，需要重启数据库 |
| innodb_flush_method            | 采用O_DIRECT方式写入的时候绕过文件系统缓存，直接写入磁盘   | 非动态，全局；默认NULL，有效值：fsync、O_DSYNC、littlesync、nosync、O_DIRECT、O_DIRECT_NO_FSYNC | =O_DIRECT                                                    |
| innodb_max_dirty_pages_pct     | buffer pool中最大脏页占比                                  |                                                              | 75%~90%，如果io能力足够强，例如使用了闪卡，可以将这个参数调小；该参数设置越小，写入压力越大。 |
| innodb_max_dirty_pages_pct_lwm | 预刷新脏页比例，可以有效控制脏页比例达到最大脏页占比       | 动态，全局；<5.7.4：默认0，范围0~99；>=5.7.5：默认0，范围0~99.99 | 70，，当脏块达到70%的时候，触发网磁盘写，控制脏页比率，防止达到脏页最大占比 |
| innodb_flush_neighbors         | 是否关闭邻接页刷新                                         | 0关闭,1不关闭                                                | 一般关闭邻接页刷新                                           |

#### 连接优化
| 参数                 | 参数功能                                                     | 取值范围                                  | 经验值                                                 |
| -------------------- | ------------------------------------------------------------ | ----------------------------------------- | ------------------------------------------------------ |
| max_connections      | 允许客户端并发连接数量                                       | 动态，全局；默认151，最小1，最大10000     | 1024，一般不要超过2000                                 |
| max_user_connections | 任何一个mysql用户允许最大并发连接数                          | 动态，全局和会话；默认0，范围0~4294967295 | 256                                                    |
| table_open_cache     | 打开表缓存，跟表数量没关系1000个连接上来，都需要访问A表，那么会打开1000个表，打开1000个表是指mysql创建1000个这个表的对象，连接直接访问表对象 | 动态，全局；默认                          | 4096 ，监控opened_tables值，如果很大，说明该值设置小了 |
| thread_cache_size    | 线程缓存                                                     |                                           | 512                                                    |
| wait_timeout         | app应用连接mysql进行操作完毕后，空闲断开的时间，单位秒       |                                           | 120                                                    |

#### 数据一致性优化

```innodb_flush_log_at_trx_commit=1``

0，不管有没有提交，每秒钟都写到binlog日志里
1，每次提交事务，都会把log buffer的内容写到磁盘里去，对日志文件做到磁盘刷新，安全最好
2，每次提交事务，都写到操作系统缓存，由OS刷新到磁盘，性能最好

```sync_binlog=1```

0，事务提交后，mysql不做fsync之类的刷盘，由文件系统来决定什么落盘
n，多少次提交，每n次提交持久化磁盘
生产设为1

