# 使用docker搭建redis集群

redis集群提供了一种分布式的redis高可用方案，用于替换老旧的主从加哨兵的模式。

配置redis集群需要如下几步：

1. 下载redis镜像
2. 配置网络
3. 准备redis配置文件
4. 启动redis
5. 链接其中一个redis进行创建集群

## 1. 下载redis镜像

在docker下，我们使用以下命令进行下载redis镜像。

```shell
docker create redis
```

## 2. 准备网络

我们需要redis在docker中使用同一个网络，类似于现实生活中的内网环境。那么我们需要创建一个路由网络，让所有的redis实例链接到该路由。

```shell
docker network create redis-net
```

检查一下是否配置成功

```shell
[root@localhost ~]# docker network inspect redis-net
## 出现一下信息说明配置成功
[
    {
        "Name": "redis-net",
        "Id": "b2647aeae8d843f957f40e8a7a92c17ca74f9bf32f732ce0fc17c585ac7e901d",
        "Created": "2020-04-04T12:54:38.668457885+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

从上面的内从中我们可以找到网关IP为**"Gateway":"172.18.0.1"**,这个地址我们后面会临时用到。

## 3. 准备配置文件

我们的目标是生成redis集群内生成6个实例，端口从7000-7005。

下面我们使用shell加模板的方式生成这些文件。

### 3.1 增加模板

我们先准备一份redis配置模板，先填入我们需要的信息。再次之前，创建一个目录`/root/redis-cluster`用于存放所有的配置。**后续所有的操作都是该目录下**。

新建`redis.conf.tmpl`,填入一下内容,${PORT}为占位符，后面后替换为真实的端口号

```
#节点端口
port ${PORT}
#开启集群模式
protected-mode no
#cluster集群模式
cluster-enabled yes
#集群配置名
cluster-config-file nodes.conf
#超时时间
cluster-node-timeout 5000
#实际为各节点网卡分配ip  先用上网关ip代替
cluster-announce-ip 172.18.0.X
#节点映射端口
cluster-announce-port ${PORT}
#节点总线端口
cluster-announce-bus-port 1${PORT}
##持久化模式
appendonly yes
```

### 3.2 替换模板占位符并创建目录

命令行执行如下代码会在当前目录下生成**port/conf/redis.conf**和**port/data**

```shell
for port in `seq 7000 7005`; do \
mkdir -p ./${port}/conf \
&& PORT=${port} envsubst < ./redis.conf.tmpl > ./${port}/conf/redis.conf \
&& mkdir -p ./${port}/data; \
done
```

至此，准备配置文件的工作暂时告一段落。通过tree命令，我们可以看到当前文件夹内的项目结构如下：

```shell
[root@localhost redis-cluster]# tree
.
├── 7000
│   ├── conf
│   │   └── redis.conf
│   └── data
├── 7001
│   ├── conf
│   │   └── redis.conf
│   └── data
├── 7002
│   ├── conf
│   │   └── redis.conf
│   └── data
├── 7003
│   ├── conf
│   │   └── redis.conf
│   └── data
├── 7004
│   ├── conf
│   │   └── redis.conf
│   └── data
├── 7005
│   ├── conf
│   │   └── redis.conf
│   └── data
└── redis.conf.tmpl
```

## 4. 启动redis

在启动redis之前，先介绍几个docker命令：

1. `docker run ` 运行容器
2. `docker ps` 查看正在运行的容器状态
3. `docker logs` 查看指定容器的输出日志
4. `docker restart `重启容器

下面我们通过一个简单的shell脚本启动7000-7005容器

```shell
for port in `seq 7000 7005`; do \
docker run -d -it -p ${port}:${port} -p 1${port}:1${port} \
-v /root/redis-cluster/${port}/conf/redis.conf:/usr/local/etc/redis/redis.conf \
-v /root/redis-cluster/${port}/data:/data \
--restart always --name redis-${port} --net redis-net \
--sysctl net.core.somaxconn=1024 redis redis-server /usr/local/etc/redis/redis.conf; \
done
```

执行完以上代码会看到类似于一下的输出，说明容器启动成功，否则请检查上面的步骤。

```shell
96ae55d79e37ee18979a6c73fb265514baabd043dbf98074121b8be254e80a09
204adee1c52171cedc63a73e38c2de490f7b123a6cade19d511ea8460578d491
e5648b4fda0ab9923dd8e82a5e660eb58c4568f189444ecb9617813f0bb65f59
4467d2fd96b7e853f22112de9e12a96ec539689912645577fc0b6e1d18eef17b
8d83cbdbb5c10aee2ecb76b300956b79248a9ac9419de3f94b8e6f9177f2d028
187a8d112f6d9fdcb99560f5ee56f853f7392ff9e9c9fd4f03db55aeb367677c
```

通过`docker ps`命令查看容器启动状态：

```shell
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                        NAMES
187a8d112f6d        redis               "docker-entrypoint.s…"   19 minutes ago      Up 4 seconds        0.0.0.0:7005->7005/tcp, 6379/tcp, 0.0.0.0:17005->17005/tcp   redis-7005
8d83cbdbb5c1        redis               "docker-entrypoint.s…"   19 minutes ago      Up About a minute   0.0.0.0:7004->7004/tcp, 6379/tcp, 0.0.0.0:17004->17004/tcp   redis-7004
4467d2fd96b7        redis               "docker-entrypoint.s…"   19 minutes ago      Up 3 minutes        0.0.0.0:7003->7003/tcp, 6379/tcp, 0.0.0.0:17003->17003/tcp   redis-7003
e5648b4fda0a        redis               "docker-entrypoint.s…"   19 minutes ago      Up 4 minutes        0.0.0.0:7002->7002/tcp, 6379/tcp, 0.0.0.0:17002->17002/tcp   redis-7002
204adee1c521        redis               "docker-entrypoint.s…"   19 minutes ago      Up 5 minutes        0.0.0.0:7001->7001/tcp, 6379/tcp, 0.0.0.0:17001->17001/tcp   redis-7001
96ae55d79e37        redis               "docker-entrypoint.s…"   19 minutes ago      Up 7 minutes        0.0.0.0:7000->7000/tcp, 6379/tcp, 0.0.0.0:17000->17000/tcp   redis-7000
```

> 如果发现`status`一栏对应的状态不是`Up`,请使用`docker logs redis-700x`查看输出。

现在redis已正常启动，我们还差一个重要的配置没有加上，回顾一下redis.conf.tmpl文件中的ip是否存在一个x。

现在我们需要手动将docker分配给对应redis的ip重新修改到`/root/redis-cluster/port/conf/redis.conf`文件中。查看docker分配的ip：

```shell
$ docker network inspect redis-net

[
    {
        "Name": "redis-net",
        "Id": "b2647aeae8d843f957f40e8a7a92c17ca74f9bf32f732ce0fc17c585ac7e901d",
        "Created": "2020-04-04T12:54:38.668457885+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "187a8d112f6d9fdcb99560f5ee56f853f7392ff9e9c9fd4f03db55aeb367677c": {
                "Name": "redis-7005",
                "EndpointID": "4db73ed8ebf71adf6de2351583b01a3300158fa1747f860e18caf8ec84201d40",
                "MacAddress": "02:42:ac:12:00:07",
                "IPv4Address": "172.18.0.7/16",
                "IPv6Address": ""
            },
            "204adee1c52171cedc63a73e38c2de490f7b123a6cade19d511ea8460578d491": {
                "Name": "redis-7001",
                "EndpointID": "09f7add62db94945442f23286a54d3988089144807b160d0b1da40331e8b21f5",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "4467d2fd96b7e853f22112de9e12a96ec539689912645577fc0b6e1d18eef17b": {
                "Name": "redis-7003",
                "EndpointID": "5fda826ac2e8e4ffe7be2fcd0a11edcecca47204d7367ac8baf68d34f5064f75",
                "MacAddress": "02:42:ac:12:00:05",
                "IPv4Address": "172.18.0.5/16",
                "IPv6Address": ""
            },
            "8d83cbdbb5c10aee2ecb76b300956b79248a9ac9419de3f94b8e6f9177f2d028": {
                "Name": "redis-7004",
                "EndpointID": "0c7168b8e844391858e62cee99beda1a9e48f7d65ee329d5bbec2617cba51f29",
                "MacAddress": "02:42:ac:12:00:06",
                "IPv4Address": "172.18.0.6/16",
                "IPv6Address": ""
            },
            "96ae55d79e37ee18979a6c73fb265514baabd043dbf98074121b8be254e80a09": {
                "Name": "redis-7000",
                "EndpointID": "eb5e4f9d8a2fae7286ca8bbfd5203bc6220606d7981b01c3ef2bc84ff263ffb0",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "e5648b4fda0ab9923dd8e82a5e660eb58c4568f189444ecb9617813f0bb65f59": {
                "Name": "redis-7002",
                "EndpointID": "97618c2d467eba646205c4ca02890aea6f4e894b9c9eeba22ee2f9ca64fb71c5",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

从以上输出中找到Name对应的IPv4Address，分别修改对应的redis.conf文件。

全部修改完成以后，执行如下命令重启redis容器：

```shell
for port in `seq 7000 7005`; do \
docker restart redis-${port}; \
done
```

## 5. 创建集群

要将各个redis实例链接起来，我们需要链接其中一个redis实例，执行创建集群命令。

```shell
# 链接到redis-7000实例所在的容器
$ docker exec -it redis-7000 bash
# 创建集群
$ redis-cli --cluster create  172.18.0.7:7005 172.18.0.4:7001 \ 
> 172.18.0.4:7001 \
> 172.18.0.5:7003 \
> 172.18.0.6:7004 \
> 172.18.0.2:7000 \
> 172.18.0.3:7002 \
> --cluster-replicas 1 
```

创建集群命令会提示用户是否已看到完整信息，输入`yes`即可。

现在链接redis查看集群情况：

```shell
$ redis-cli -c -p 7000
127.0.0.1:7000> info replication
# Replication
role:slave
master_host:172.18.0.7
master_port:7005
master_link_status:up
master_last_io_seconds_ago:9
master_sync_in_progress:0
slave_repl_offset:84
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:8c42e4110b55a6054fa68e134350e0dca43d9654
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:84
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:15
repl_backlog_histlen:70
```

到此，redis集群搭建完成。

## 参考

[Docker Redis 5.0 集群（cluster）搭建](https://juejin.im/post/5c9ca08f5188252d5a14a31b)







