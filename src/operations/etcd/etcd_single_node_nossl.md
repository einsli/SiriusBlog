# etcd docker 安装与配置

## 前期准备

依赖`docker`和 `docker-compose环境`

**注:** `docker` 环境搭建不建议用于生产环境，仅用于学习和开发环境使用

请参考

👉🏻 [docker 安装与配置](../docker/linux_docker_installed.md)

👉🏻 [docker-compose 安装](../docker/linux_docker_compose_installed.md)

## docker-compose 安装 etcd

**1、创建专属的运维网络,用户跟其他容器交互**

**注:** 创建之前，可以执行以下命令查看`ops_network`网络是否存在，若存在就无需创建

```shell
docker network ls |grep ops_network
```

如果有如下输出就无需创建

```PlainText
514e3dd06943   ops_network   bridge    local
```

如果没有`ops_network`,执行以下命令创建即可

```shell
docker network create ops_network
```

**2、部署文件准备**

在`/data` 目录下，新建`etcd`目录

```shell
mkdir /data/etcd
```

**注:** `/data` 目录可选的，可以选择其他目录

在`/data/redis`下新建`docker-compose.yaml`, `run.sh` , `stop.sh` 三个文件和`etcd_data`文件夹

```shell
mkdir -p /data/etcd/etcd_data

chown -R 1001:1001 /data/etcd/etcd_data # etcd 启动不是root用户，所以要改变下权限

cat > /data/redis/docker-compose.yaml<< EOF
version: '3.1'

services:
  etcd-server:
    image: 'bitnami/etcd:latest'
    container_name: "etcd-server"
    environment:
      - "ETCD_NAME=node1"
      - "ETCD_ROOT_PASSWORD=Passw0rd"
      - "ETCD_CLIENT_CERT_AUTH=true"
      - "ETCD_PEER_CLIENT_CERT_AUTH=true"
      - "ETCD_ADVERTISE_CLIENT_URLS=http://192.168.1.102:42379"
      - "ETCD_INITIAL_ADVERTISE_PEER_URLS=http://192.168.1.102:42380"
      - "ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379"
      - "ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380"
      - "ETCD_INITIAL_CLUSTER_TOKEN=etcd_cluster"
      - "ETCD_INITIAL_CLUSTER=node1=http://192.168.1.102:42380"
      - "ETCD_INITIAL_CLUSTER_STATE=new"
    volumes:
      - ./etcd_data:/bitnami/etcd/data
      - ./etcd_conf:/opt/bitnami/etcd/conf
    ports:
      - 42379:2379
      - 42380:2380
    networks:
      - ops_network

  etcdkeeper:
    image: "deltaprojects/etcdkeeper:202311192256"
    container_name: "etcdkeeper"
    ports:
      - 11080:8080
    depends_on:
      - etcd-server
    networks:
      - ops_network

networks:
  ops_network:
    external: true
EOF
```

**注:** `etcdkeeper` 镜像`deltaprojects/etcdkeeper:202311192256` 为自己构建的镜像，官方提供的镜像无法开启验证登录

启动脚本

```shell
cat > /data/etcd/run.sh<< EOF
#!/bin/bash

docker-compose -p etcd-server up -d
EOF

chmod +x /data/etcd/run.sh
```

停止脚本

```shell
cat > /data/etcd/stop.sh<< EOF
#!/bin/bash

docker-compose -p etcd-server down
EOF

chmod +x /data/etcd/stop.sh
```

目录结构如下

```PlainText
.
├── etcd_conf
├── etcd_data
├── docker-compose.yaml
├── run.sh
└── stop.sh
```

**3、启动与关闭**

启动

```shell
cd /data/redis/
./run.sh
```

关闭

```shell
cd /data/redis/
./stop.sh
```

**附：** etcdkeeper 镜像制作

下载二进制包

```shell
wget https://github.com/evildecay/etcdkeeper/releases/download/v0.7.6/etcdkeeper-v0.7.6-linux_x86_64.zip
```

在 `/data/`下面新建`dockerfiles`文件夹，在`其下面新建etcdkeeper`文件

```shell
mkdir -p /data/dockerfiles/etcdkeeper
```

解压`etcdkeeper` 并移动到 `/data/dockerfiles/etcdkeeper`目录下

```shell
unzip etcdkeeper-v0.7.6-linux_x86_64.zip
mv etcdkeeper /data/dockerfiles/etcdkeeper
```

准备`Dockerfile`

基于官方文件改造的

```shell
cat > /data/dockerfiles/etcdkeeper/Dockerfile<< EOF
FROM deltaprojects/etcdkeeper:latest

RUN rm -rf /etcdkeeper

COPY etcdkeeper /etcdkeeper

WORKDIR /etcdkeeper

ENTRYPOINT ./etcdkeeper -h 0.0.0.0 -p 8080 -auth true # 这么做是开启了认证
EOF
```

构建镜像

```shell
cd /data/dockerfiles/etcdkeeper/

docker build --no-cache -t deltaprojects/etcdkeeper:202311192256
```
