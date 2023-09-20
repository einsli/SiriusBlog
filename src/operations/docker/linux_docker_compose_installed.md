# linux docker-compose 安装

## 前期准备

- 安装之前，请先确保 docker 已经安装，如果没有安装，请参考[linux docker 安装与配置](linux_docker_installed.md)

- docker-compose 各版本 [下载地址](https://github.com/docker/compose/releases)

## 二进制安装 docker-compose

1、下载对应 linux 版本的 docker-compose 二进制文件

```shell
yum -y install wget
wget https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-linux-x86_64
```

2、赋予执行权限

```shell
chmod +x docker-compose-linux-x86_64
```

3、移动到`/usr/bin/` 路径下，并重命名为`docker-compose`

```shell
mv docker-compose-linux-x86_64 /usr/bin/docker-compose
```

4、验证并查看版本

```shell
docker-compose version
```

如下图所示，则安装成功

![avatar](../../images/operations/docker/docker-compose-version.png)
