### 软件

```
centos-release-7-5.1804.1.el7.centos.x86_64
Redis server v=4.0.11
ruby 2.5.0p0 (2017-12-25 revision 61468) [x86_64-linux]
gem 2.7.3
```

### 系统准备

系统必要更新

```shell
# yum -y update 
# yum -y upgrade
```

安装 gcc 、gcc-c++、openssl-devel

```shell
# yum install -y gcc gcc-c++ openssl-devel
```

关闭防火墙

```shell
# systemctl stop firewalld.service
```

### 安装 Ruby 及 RubyGems

查看当前安装的 Ruby 及 Gems

```shell
# ruby -v
# gem -v
```

如果出现版本号，则说明当前环境已安装过，可以跳过安装

若版本太低，删除原有 Ruby

```shell
# yum remove ruby ruby-devel
```

下载需要安装的 Ruby 版本：http://cache.ruby-lang.org/pub/ruby/ 

下载完成后利用 FTP 传入 Centos 机器，也可以使用如下命令下载

```shell
# wget http://cache.ruby-lang.org/pub/ruby/ruby-2.5.0.tar.gz
```

开始安装

```shell
# tar zxvf ruby-2.5.0.tar.gz
# cd ruby-2.5.0
# ./configure
# make && make install
```

#### 配置 zlib 

```shell
# cd ruby-2.5.0/ext/zlib
# ruby ./extconf.rb
# vim Makefile 
```

更改 Makefile 中的配置

```shell
 zlib.o: $(top_srcdir)/include/ruby.h  
 改成   
 zlib.o: ../../include/ruby.h
 // 退出 vim 并保存
 # make && make install
```



#### 配置 openssl

```shell
# cd ruby-2.5.0/ext/openssl
# ruby ./extconf.rb
# vim Makefile 
```

更改 Makefile 中的配置

```shell
将所有的  $(top_srcdir)
替换为： ../..
vim 快捷命令  :1,$s/$(top_srcdir)/..\/..
 // 退出 vim 并保存
# make && make install
```

安装 zlib-devel (当时忘记了这一步是否执行完全了)
`yum install zlib-devel && rvm reinstall 2.5.1`

这里提示-bash: rvm: command not found，
然后执行 `curl -L get.rvm.io | bash -s stable`，报错为gpg: Can't check signature: No public key，
然后执行 `gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3`，再执行`curl -L get.rvm.io | bash -s stable`，到这里rvm安装成功！

```shell
# gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
# yum install zlib-devel && rvm reinstall 2.5.1
```

更新 RubyGems 和 Bundler (可选)

```shell
# gem update --system
# gem install bundler
```

更新老版本安装的 RubyGems

```shell
# gem update
```

安装 redis 插件

```shell
# gem install redis
```

### 安装 Redis

#### 安装 Redis

下载，解压，编译安装

```shell
cd /opt
# wget http://download.redis.io/releases/redis-4.0.1.tar.gz
# tar xzf redis-4.0.1.tar.gz
# cd redis-4.0.1
# make
```

如果因为上次编译失败，有残留的文件

```shell
# make distclean
```

#### 创建节点

1.首先在 192.168.252.101机器上 /opt/redis-4.0.1目录下创建 `redis-cluster` 目录

```shell
# mkdir /opt/redis-4.0.1/redis-cluster
```

2.在 `redis-cluster` 目录下，创建名为`7000、7001、7002`的目录

```shell
# cd /opt/redis-4.0.1/redis-cluster
# mkdir 7000 7001 7002
```

3.分别修改这三个配置文件，把如下`redis.conf 配置`内容粘贴进去

```shell
# vi 7000/redis.conf
# vi 7001/redis.conf
# vi 7002/redis.conf
```

redis.conf 配置

```properties
port 7000
bind 192.168.252.101
daemonize yes
pidfile /var/run/redis_7000.pid
cluster-enabled yes
cluster-config-file nodes_7000.conf
cluster-node-timeout 10100
appendonly yes
```

redis.conf 配置说明

```properties
#端口7000,7001,7002
port 7000

#默认ip为127.0.0.1，需要改为其他节点机器可访问的ip，否则创建集群时无法访问对应的端口，无法创建集群
bind 192.168.252.101

#redis后台运行
daemonize yes

#pidfile文件对应7000，7001，7002
pidfile /var/run/redis_7000.pid

#开启集群，把注释#去掉
cluster-enabled yes

#集群的配置，配置文件首次启动自动生成 7000，7001，7002          
cluster-config-file nodes_7000.conf

#请求超时，默认15秒，可自行设置 
cluster-node-timeout 10100    
        
#aof日志开启，有需要就开启，它会每次写操作都记录一条日志
appendonly yes
```

接着在另外两台机器上(192.168.252.102，192.168.252.103)重复以上三步，只是把目录改为7003、7004、7005、7006、7007、7008对应的配置文件也按照这个规则修改即可

启动集群

```shell
#第一台机器上执行 3个节点
# for((i=0;i<=2;i++)); do /opt/redis-4.0.1/src/redis-server /opt/redis-4.0.1/redis-cluster/700$i/redis.conf; done

#第二台机器上执行 3个节点
# for((i=3;i<=5;i++)); do /opt/redis-4.0.1/src/redis-server /opt/redis-4.0.1/redis-cluster/700$i/redis.conf; done

#第三台机器上执行 3个节点 
# for((i=6;i<=8;i++)); do /opt/redis-4.0.1/src/redis-server /opt/redis-4.0.1/redis-cluster/700$i/redis.conf; done
```

#### 检查服务

检查各 Redis 各个节点启动情况

```shell
# ps -ef | grep redis           //redis是否启动成功
# netstat -tnlp | grep redis    //监听redis端口
```

#### 创建集群

**注意：在任意一台上运行** 不要在每台机器上都运行，一台就够了

Redis 官方提供了 `redis-trib.rb` 这个工具，就在解压目录的 src 目录中

```shell
# /opt/redis-4.0.1/src/redis-trib.rb create --replicas 1 192.168.252.101:7000192.168.252.101:7001 192.168.252.101:7002 192.168.252.102:7003 192.168.252.102:7004192.168.252.102:7005 192.168.252.103:7006 192.168.252.103:7007 192.168.252.103:7008
```

出现以下内容

```
[root@localhost redis-cluster]# /opt/redis-4.0.1/src/redis-trib.rb create --replicas 1 192.168.252.101:7000 192.168.252.101:7001 192.168.252.101:7002 192.168.252.102:7003 192.168.252.102:7004 192.168.252.102:7005 192.168.252.103:7006 192.168.252.103:7007 192.168.252.103:7008
>>> Creating cluster
>>> Performing hash slots allocation on 9 nodes...
Using 4 masters:
192.168.252.101:7000
192.168.252.102:7003
192.168.252.103:7006
192.168.252.101:7001
Adding replica 192.168.252.102:7004 to 192.168.252.101:7000
Adding replica 192.168.252.103:7007 to 192.168.252.102:7003
Adding replica 192.168.252.101:7002 to 192.168.252.103:7006
Adding replica 192.168.252.102:7005 to 192.168.252.101:7001
Adding replica 192.168.252.103:7008 to 192.168.252.101:7000
M: 7c622ac191edd40dd61d9b79b27f6f69d02a5bbf 192.168.252.101:7000
   slots:0-4095 (4096 slots) master
M: 44c81c15b01d992cb9ede4ad35477ec853d70723 192.168.252.101:7001
   slots:12288-16383 (4096 slots) master
S: 38f03c27af39723e1828eb62d1775c4b6e2c3638 192.168.252.101:7002
   replicates f1abb62a8c9b448ea14db421bdfe3f1d8075189c
M: 987965baf505a9aa43e50e46c76189c51a8f17ec 192.168.252.102:7003
   slots:4096-8191 (4096 slots) master
S: 6555292fed9c5d52fcf5b983c441aff6f96923d5 192.168.252.102:7004
   replicates 7c622ac191edd40dd61d9b79b27f6f69d02a5bbf
S: 2b5ba254a0405d4efde4c459867b15176f79244a 192.168.252.102:7005
   replicates 44c81c15b01d992cb9ede4ad35477ec853d70723
M: f1abb62a8c9b448ea14db421bdfe3f1d8075189c 192.168.252.103:7006
   slots:8192-12287 (4096 slots) master
S: eb4067373d36d8a8df07951f92794e67a6aac022 192.168.252.103:7007
   replicates 987965baf505a9aa43e50e46c76189c51a8f17ec
S: 2919e041dd3d1daf176d6800dcd262f4e727f366 192.168.252.103:7008
   replicates 7c622ac191edd40dd61d9b79b27f6f69d02a5bbf
Can I set the above configuration? (type 'yes' to accept): yes
```

**输入 yes**

```
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join.........
>>> Performing Cluster Check (using node 192.168.252.101:7000)
M: 7c622ac191edd40dd61d9b79b27f6f69d02a5bbf 192.168.252.101:7000
   slots:0-4095 (4096 slots) master
   2 additional replica(s)
S: 6555292fed9c5d52fcf5b983c441aff6f96923d5 192.168.252.102:7004
   slots: (0 slots) slave
   replicates 7c622ac191edd40dd61d9b79b27f6f69d02a5bbf
M: 44c81c15b01d992cb9ede4ad35477ec853d70723 192.168.252.101:7001
   slots:12288-16383 (4096 slots) master
   1 additional replica(s)
S: 2919e041dd3d1daf176d6800dcd262f4e727f366 192.168.252.103:7008
   slots: (0 slots) slave
   replicates 7c622ac191edd40dd61d9b79b27f6f69d02a5bbf
M: f1abb62a8c9b448ea14db421bdfe3f1d8075189c 192.168.252.103:7006
   slots:8192-12287 (4096 slots) master
   1 additional replica(s)
S: eb4067373d36d8a8df07951f92794e67a6aac022 192.168.252.103:7007
   slots: (0 slots) slave
   replicates 987965baf505a9aa43e50e46c76189c51a8f17ec
S: 38f03c27af39723e1828eb62d1775c4b6e2c3638 192.168.252.101:7002
   slots: (0 slots) slave
   replicates f1abb62a8c9b448ea14db421bdfe3f1d8075189c
S: 2b5ba254a0405d4efde4c459867b15176f79244a 192.168.252.102:7005
   slots: (0 slots) slave
   replicates 44c81c15b01d992cb9ede4ad35477ec853d70723
M: 987965baf505a9aa43e50e46c76189c51a8f17ec 192.168.252.102:7003
   slots:4096-8191 (4096 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

#### 关闭集群

这样也可以，推荐

```shell
# pkill redis
```

循环节点逐个关闭

```shell
# for((i=0;i<=2;i++)); do /opt/redis-4.0.1/src/redis-cli -c -h 192.168.252.101 -p 700$i shutdown; done

# for((i=3;i<=5;i++)); do /opt/redis-4.0.1/src/redis-cli -c -h 192.168.252.102 -p 700$i shutdown; done

# for((i=6;i<=8;i++)); do /opt/redis-4.0.1/src/redis-cli -c -h 192.168.252.103 -p 700$i shutdown; done
```

#### 集群验证

参数 -C 可连接到集群，因为 redis.conf 将 bind 改为了ip地址，所以 -h 参数不可以省略，-p 参数为端口号

- **我们在192.168.252.101机器redis 7000 的节点set 一个key**

```
$ /opt/redis-4.0.1/src/redis-cli -h 192.168.252.101 -c -p 7000
192.168.252.101:7000> set name www.ymq.io
-> Redirected to slot [5798] located at 192.168.252.102:7003
OK
192.168.252.102:7003> get name
"www.ymq.io"
192.168.252.102:7003>
```

发现redis set name 之后重定向到192.168.252.102机器 redis 7003 这个节点

- **我们在192.168.252.103机器redis 7008 的节点get一个key**

```
[root@localhost redis-cluster]# /opt/redis-4.0.1/src/redis-cli -h 192.168.252.103 -c -p 7008
192.168.252.103:7008> get name
-> Redirected to slot [5798] located at 192.168.252.102:7003
"www.ymq.io"
192.168.252.102:7003> 
```

发现redis get name 重定向到192.168.252.102机器 redis 7003 这个节点

> 如果您看到这样的现象，说明集群已经是可用的了

#### 检查集群状态

```
$ /opt/redis-4.0.1/src/redis-trib.rb check 192.168.252.101:7000
```

```
>>> Performing Cluster Check (using node 192.168.252.101:7000)
M: 7c622ac191edd40dd61d9b79b27f6f69d02a5bbf 192.168.252.101:7000
   slots:0-4095 (4096 slots) master
   2 additional replica(s)
S: 6555292fed9c5d52fcf5b983c441aff6f96923d5 192.168.252.102:7004
   slots: (0 slots) slave
   replicates 7c622ac191edd40dd61d9b79b27f6f69d02a5bbf
M: 44c81c15b01d992cb9ede4ad35477ec853d70723 192.168.252.101:7001
   slots:12288-16383 (4096 slots) master
   1 additional replica(s)
S: 2919e041dd3d1daf176d6800dcd262f4e727f366 192.168.252.103:7008
   slots: (0 slots) slave
   replicates 7c622ac191edd40dd61d9b79b27f6f69d02a5bbf
M: f1abb62a8c9b448ea14db421bdfe3f1d8075189c 192.168.252.103:7006
   slots:8192-12287 (4096 slots) master
   1 additional replica(s)
S: eb4067373d36d8a8df07951f92794e67a6aac022 192.168.252.103:7007
   slots: (0 slots) slave
   replicates 987965baf505a9aa43e50e46c76189c51a8f17ec
S: 38f03c27af39723e1828eb62d1775c4b6e2c3638 192.168.252.101:7002
   slots: (0 slots) slave
   replicates f1abb62a8c9b448ea14db421bdfe3f1d8075189c
S: 2b5ba254a0405d4efde4c459867b15176f79244a 192.168.252.102:7005
   slots: (0 slots) slave
   replicates 44c81c15b01d992cb9ede4ad35477ec853d70723
M: 987965baf505a9aa43e50e46c76189c51a8f17ec 192.168.252.102:7003
   slots:4096-8191 (4096 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

#### 列出集群节点

列出集群当前已知的所有节点（node），以及这些节点的相关信息

```shell
# /opt/redis-4.0.1/src/redis-cli -h 192.168.252.101 -c -p 7000

192.168.252.101:7000> cluster nodes
```

```
6555292fed9c5d52fcf5b983c441aff6f96923d5 192.168.252.102:7004@17004 slave 7c622ac191edd40dd61d9b79b27f6f69d02a5bbf 0 1502815268317 5 connected
44c81c15b01d992cb9ede4ad35477ec853d70723 192.168.252.101:7001@17001 master - 0 1502815268000 2 connected 12288-16383
2919e041dd3d1daf176d6800dcd262f4e727f366 192.168.252.103:7008@17008 slave 7c622ac191edd40dd61d9b79b27f6f69d02a5bbf 0 1502815269000 9 connected
7c622ac191edd40dd61d9b79b27f6f69d02a5bbf 192.168.252.101:7000@17000 myself,master - 0 1502815269000 1 connected 0-4095
f1abb62a8c9b448ea14db421bdfe3f1d8075189c 192.168.252.103:7006@17006 master - 0 1502815269000 7 connected 8192-12287
eb4067373d36d8a8df07951f92794e67a6aac022 192.168.252.103:7007@17007 slave 987965baf505a9aa43e50e46c76189c51a8f17ec 0 1502815267000 8 connected
38f03c27af39723e1828eb62d1775c4b6e2c3638 192.168.252.101:7002@17002 slave f1abb62a8c9b448ea14db421bdfe3f1d8075189c 0 1502815269327 7 connected
2b5ba254a0405d4efde4c459867b15176f79244a 192.168.252.102:7005@17005 slave 44c81c15b01d992cb9ede4ad35477ec853d70723 0 1502815270336 6 connected
987965baf505a9aa43e50e46c76189c51a8f17ec 192.168.252.102:7003@17003 master - 0 1502815271345 4 connected 4096-8191
192.168.252.101:7000> 
```

打印集群信息

```
# 192.168.252.101:7000> cluster info
```

```
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:9
cluster_size:4
cluster_current_epoch:9
cluster_my_epoch:1
cluster_stats_messages_ping_sent:485
cluster_stats_messages_pong_sent:485
cluster_stats_messages_sent:970
cluster_stats_messages_ping_received:477
cluster_stats_messages_pong_received:485
cluster_stats_messages_meet_received:8
cluster_stats_messages_received:970
192.168.252.101:7000> 
```

### 集群命令

#### 语法格式

```
redis-cli -c -p port
```

#### 集群

```
cluster info ：打印集群的信息
cluster nodes ：列出集群当前已知的所有节点（ node），以及这些节点的相关信息。
```

#### 节点

```
cluster meet <ip> <port> ：将 ip 和 port 所指定的节点添加到集群当中，让它成为集群的一份子。
cluster forget <node_id> ：从集群中移除 node_id 指定的节点。
cluster replicate <node_id> ：将当前节点设置为 node_id 指定的节点的从节点。
cluster saveconfig ：将节点的配置文件保存到硬盘里面。
```

#### 槽(slot)

```
cluster addslots <slot> [slot ...] ：将一个或多个槽（ slot）指派（ assign）给当前节点。
cluster delslots <slot> [slot ...] ：移除一个或多个槽对当前节点的指派。
cluster flushslots ：移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点。
cluster setslot <slot> node <node_id> ：将槽 slot 指派给 node_id 指定的节点，如果槽已经指派给另一个节点，那么先让另一个节点删除该槽>，然后再进行指派。
cluster setslot <slot> migrating <node_id> ：将本节点的槽 slot 迁移到 node_id 指定的节点中。
cluster setslot <slot> importing <node_id> ：从 node_id 指定的节点中导入槽 slot 到本节点。
cluster setslot <slot> stable ：取消对槽 slot 的导入（ import）或者迁移（ migrate）。
```

#### 键

```
cluster keyslot <key> ：计算键 key 应该被放置在哪个槽上。
cluster countkeysinslot <slot> ：返回槽 slot 目前包含的键值对数量。
cluster getkeysinslot <slot> <count> ：返回 count 个 slot 槽中的键 。
```