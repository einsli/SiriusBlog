# mongodb docker 安装与配置

## 前期准备

依赖`docker`和 `docker-compose环境`

**注:** `docker` 环境搭建不建议用于生产环境，仅用于学习和开发环境使用

请参考

👉🏻 [docker 安装与配置](../docker/linux_docker_installed.md)

👉🏻 [docker-compose 安装](../docker/linux_docker_compose_installed.md)

## docker-compose 安装 mongodb

**1、创建专属的运维网络,用户跟其他容器交互**

**注:** 创建之前，可以执行以下命令查看`ops_network`网络是否存在，若存在就无需创建

```shell
docker network ls |grep ops_network
```

如果有如下输出就无需创建

```PlainText
514e3dd06943   ops_network   bridge    local
```

```shell
docker network create ops_network
```

**2、部署文件准备**

在`/data` 目录下，新建`mongodb`目录

```shell
mkdir /data/mongodb
```

**注:** `/data` 目录可选的，可以选择其他目录

在`/data/mongodb`下新建`docker-compose.yaml`, `run.sh` , `stop.sh`

```shell
mkdir -p /data/mongodb

cat > /data/mongodb/docker-compose.yaml<< EOF
version: "3.1"
services:
  local-mongo:
    image: mongo:7.0.0
    restart: always
    container_name: local-mongo
    hostname: mongo
    environment:
      #用户名密码
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: Passw0rd
    ports:
      - 33017:27017
    volumes:
      - ./data:/data/db
    networks:
      - ops_network

networks:
  ops_network:
    external: true
EOF
```

启动脚本

```shell
cat > /data/mongodb/run.sh<< EOF
#!/bin/bash

docker-compose -p mongodb-server up -d
EOF

chmod +x /data/mongodb/run.sh
```

停止脚本

```shell
cat > /data/mongodb/stop.sh<< EOF
#!/bin/bash

docker-compose -p mongodb-server down
EOF

chmod +x /data/mongodb/stop.sh
```

目录结构如下

```PlainText
├── data
├── docker-compose.yaml
├── run.sh
└── stop.sh
```

**3、启动与关闭**

启动

```shell
cd /data/mongodb/
./run.sh
```

关闭

```shell
cd /data/mongodb/
./stop.sh
```
