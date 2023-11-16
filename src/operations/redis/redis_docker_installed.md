# redis docker 安装与配置

## 前期准备

依赖`docker`和 `docker-compose环境`

**注:** docker 环境搭建不建议用于生产环境，仅用于学习和开发环境使用

请参考

👉🏻 [docker 安装与配置](../docker/linux_docker_installed.md)

👉🏻 [docker-compose 安装](../docker/linux_docker_compose_installed.md)

## docker-compose 安装

创建专属的运维网络,用户跟其他容器交互

```shell
docker network create ops_network
```

在`/data` 目录下，新建`redis`目录

```
mkdir /data/redis
```

**注:** `/data` 目录可选的，可以选择其他目录

在`/data/redis`下新建`docker-compose.yaml`, `run.sh` , `stop.sh` 三个文件和`conf`文件夹

`redis` 文件夹下存放 redis.conf 文件

```shell
mkdir -p /data/redis/conf
cat > /data/redis/docker-compose.yaml<< EOF
version: '3.1'

services:
  redis:
    image: redis:latest
    container_name: local-redis
    command: redis-server /usr/local/etc/redis/redis.conf
    restart: always
    ports:
      - 9379:6379
    volumes:
      - ./data:/data
      - ./conf/redis.conf:/usr/local/etc/redis/redis.conf
    networks:
      - ops_network

networks:
  ops_network:
    external: true
EOF
```

启动脚本

```shell
cat > /data/redis/run.sh<< EOF
#!/bin/bash

docker-compose -p redis-server up -d
EOF

chmod +x /data/redis/run.sh
```

停止脚本

```shell
cat > /data/redis/stop.sh<< EOF
#!/bin/bash

docker-compose -p redis-server down
EOF

chmod +x /data/redis/stop.sh
```

准备`redis.conf`配置文件

`redis.conf`

```conf
# 绑定地址
bind 0.0.0.0
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize no
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile ""
databases 16
always-show-logo yes
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir ./
replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
replica-priority 100
# 开启密码验证
requirepass Passw0rd
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes
```

**注:**这是从官方过滤出来的主要配置，具体配置还要以应用场景为准
