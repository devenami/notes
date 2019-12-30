### 前言

linux下扩大系统swap分区有两种方法，一种是重新建立swap分区，一种是增加swap分区。

**注：**必须拥有`root`权限，并且不小心会破坏硬盘数据（做好备份），建议增加swap分区。



#### 方法一：新建swap分区

1、以root身份进入系统，输入下面命令停止交换分区

```sh
# swapoff -a
```

2、用 `fdisk` 命令加swap分区的盘符，（例如：`# fdisk /dev/sdb`）剔除swap分区，输入 `d` 删除swap分区，然后再 `n` 添加分区（添加时硬盘必须要有可用空间），然后再用 `t` 将新添的分区`id` 改为 `82`（linux swap类型），最后用 `w` 将操作实际写入硬盘。

3、格式化swap分区，`sdb2`盘符要使用 `p` 命令显示的实际分区设备名

```sh
# mkswap /dev/sdb2
```

4、启动新的swap分区

```sh
swapon /dev/sdb2
```

5、配置开机启动swap分区，在 `/etc/fstab` 文件内最后添加一行

```
/dev/sdb2 swap swap defaults 0 0
```



#### 方法二：增加swap分区

1、创建交换分区的文件，增加1G大小的交换分区，命令如下，其中count等于要增加的块大小

```sh
# dd if=/dev/zero of /home/swapfile bs=1M count=1024
```

2、设置交换分区文件，建立swap的文件系统

```sh
# mkswap /home/swapfile
```

3、立即启用交换分区文件

```sh
# swapon /home/swapfile
```

4、配置开机启动swap分区，在 `/etc/fstab` 文件内最后添加一行

```
/home/swapfile swap swap defaults 0 0
```

