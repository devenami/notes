# Centos安装docker

## 安装前准备工作

* 一台正常联网的CentOS主机[虚拟机]
* yum可以正常访问**互联网**

## 卸载docker

在安装docker之前，我们需要通过一下命令将本地以前安装的docker进行完整的删除。

```shell
yum remove docker \
                   docker-client \
                   docker-client-latest \
                   docker-common \
                   docker-latest \
                   docker-latest-logrotate \
                   docker-logrotate \
                   docker-engine
```

## 安装yum工具

docker需要使用yum提供的工具来下载文件，所以我们需提前安装好所有的工具。

```shell
yum install -y yum-utils \
   device-mapper-persistent-data \
   lvm2
```

## 添加docker文件仓库

docker安装文件默认没有存放在yum官方仓库中，需要额外添加仓库路径。

```shell
yum-config-manager \
     --add-repo \
     https://download.docker.com/linux/centos/docker-ce.repo
```

## 开始安装

```shell
yum install docker-ce docker-ce-cli containerd.io
```

整个安装过程耗时可能比较长，这取决于yum的下载速度。

## 启动docker

```shell
systemctl start docker
```

## 运行demo

```shell
docker run hello-world
```

如果你遇到一下提示，请为docker设置代理，或使用第三方仓库（推荐）

```shell
Unable to find image 'hello-world:latest' locally
docker: Error response from daemon: Get https://registry-1.docker.io/v2/library/hello-world/manifests/latest: Get https://auth.docker.io/token?scope=repository%3Alibrary%2Fhello-world%3Apull&service=registry.docker.io: net/http: request canceled (Client.Timeout exceeded while awaiting headers).
See 'docker run --help'.
```

