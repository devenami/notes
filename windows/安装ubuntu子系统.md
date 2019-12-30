## 启用WSL

使用`WINDOWS + X`打开`Windows PowerShell(管理员)`，执行如下命令：

```sh
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

执行完毕后按回车键重启。

## 安装Ubuntu

1、在开始菜单中找到`Windows Store`，搜索`Ubuntu`进行安装。

2、打开开始菜单，点击刚刚安装的`Ubuntu`，系统会提示`Installing, this may take a few minutes..`

3、等待安装，命令行中会提示输入用户名和密码，按要求填写。

## 启动Ubuntu

1、通过开始菜单按钮进行启动。

2、命令行中输入`bash`或`wsl`登录子系统。

## 挂载点

`Ubuntu`子系统安装后默认挂载了`c`盘。位置在`/mnt`下面，进入该目录可以看到`C`盘的内容，但是需要通过`sudo`提权后才能查看。

## XShell 连接子系统

1、配置SSH服务，执行如下命令

```shell
sudo apt-get remove --purge openssh-server   ## 先删ssh
sudo apt-get install openssh-server          ## 在安装ssh  

sudo rm /etc/ssh/ssh_config                  ## 删配置文件，让ssh服务自己想办法链接
sudo service ssh --full-restart
```

2、直接使用XShell通过IP进行登录即可。

但是这种操作在子系统重启后就失效。设置永久生效的方法为将`sudo service ssh --full-restart`命令设置为脚本，系统每次重启后执行该脚本。

**ssh.sh**

```shell
#!/bin/sh
sudo service ssh --full-restart
```

赋予执行权限：

```shell
sudo chmod +x ssh.sh
```

使用如下命令执行脚本：

```shell
sh ssh.sh
```

## 配置国内镜像源

1、访问[清华大学开源软件镜像站](<https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/>)，选择对应的Ubuntu版本，复制镜像，如：

```shell
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

2、备份`/etc/apt/sources.list`

```shell
sudo cp sources.list sources.list.bak
```

3、编辑`/etc/apt/sources.list`，将文件内的文本全部删除，并替换为刚才复制的文本，保存并退出。

## Ubuntu apt相关命令

| 命令                                         | 解释                                   |
| -------------------------------------------- | -------------------------------------- |
| sudo apt-get update                          | 更新源                                 |
| sudo apt-get install \<package>              | 安装包                                 |
| sudo apt-get remove \<package>               | 删除包                                 |
| sudo apt-cache search \<package>             | 搜索软件包                             |
| sudo apt-cache show \<package>               | 获取包的相关信息，如说明、大小、版本等 |
| sudo apt-get install \<package> --reinstall  | 重新安装包                             |
| sudo apt-get -f install                      | 修复安装                               |
| sudo apt-get remove \<package> --purge       | 删除包，包括配置文件等                 |
| sudo apt-get build-dep \<package>            | 安装相关的编译环境                     |
| sudo apt-get upgrade                         | 更新已安装的包                         |
| sudo apt-get dist-upgrade                    | 升级系统                               |
| sudo apt-cache depends \<package>            | 了解使用该包依赖那些包                 |
| sudo apt-cache rdepends \<package>           | 查看该包被哪些包依赖                   |
| sudo apt-get source \<package>               | 下载该包的源代码                       |
| sudo apt-get clean && sudo apt-get autoclean | 清理无用的包                           |
| sudo apt-get check                           | 检查是否有损坏的依赖                   |

