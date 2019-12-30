## Linux虚拟机设置静态ip

### 1、获取网关地址

该文档中使用**VMware**创建linux虚拟机，所有我们的网关地址从**VMware**中进行获取。

1. 打开VMware主界面
2. 点击`编辑`->`虚拟网络编辑器`
3. 单击`NAT 模式`后可以查看最下边一行为：子网IP和子网掩码
4. 单击`NAT 设置`，查看或修改`网关 IP`

该文档中`网关IP`使用192.168.73.2



### 2、配置静态ip

使用`ifconfig`命令查看当前虚拟机的网卡信息

```
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.73.151  netmask 255.255.255.0  broadcast 192.168.73.255
        inet6 fe80::e24c:de6e:faf9:312e  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:b8:66:21  txqueuelen 1000  (Ethernet)
        RX packets 254  bytes 24518 (23.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 248  bytes 25016 (24.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

由上可知当前虚拟机的网卡为`ens33`

使用`vim`修改`/etc/sysconfig/network-scripts/ifcfg-ens33`

```
ONBOOT="yes"
BOOTPROTO="static"
IPADDR="192.168.73.151"
NETMASK="255.255.255.0"
GATEWAY="192.168.73.2"
```



### 3、配置DNS

修改`/etc/resolv.conf`

```
nameserver 192.168.73.2
```



### 4、 使配置生效

重启主机或使用`service network restart`即可使配置生效。