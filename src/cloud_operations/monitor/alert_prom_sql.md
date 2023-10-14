# prometheus 监控告警 sql

这里主要是针对日常用到的监控告警 prom_sql 的记录与描述

版本信息

> prometheus, version 2.19.2
>
> (branch: HEAD, revision: c448ada63d83002e9c1d2c9f84e09f55a61f0ff7)

### 一、主机相关

主机存活 sql(node-exporter 服务异常或者服务器宕掉就告警)

`sql
up{app="node-exporter",instance=~"192.*"} == 0
`

**注:这里对主机的 ip 进行了相关的筛选，把 instance 里面的匹配值换成自己对应的 ip 网段即可**

主机内存告警(内存使用率超过 96 告警)

```sql
((node_memory_MemTotal_bytes - (node_memory_MemFree_bytes + node_memory_Buffers_bytes + node_memory_Cached_bytes)) / node_memory_MemTotal_bytes) * 100 > 96
```

主机 CPU 监控(CPU 使用率超过 90 告警)

```sql
100 - ((avg by(instance, job, env) (irate(node_cpu_seconds_total{mode="idle"}[30s]))) * 100) > 90
```

主机网络流量监控(超过 200M 就告警)

```sql
avg by(instance, job, env) (irate(node_network_receive_bytes_total{device!="lo"}[5m]) * 8) / (1000 * 1000) > 200
```

主机磁盘监控(超过 90 就告警)

```sql
100 - (node_filesystem_avail_bytes{instance=~"172.16.*"} * 100) / node_filesystem_size_bytes{instance=~"172.16.*"} > 90
```

### 二、k8s 相关

线上 k8s 节点 CPU 监控告警(2 分钟内，CPU 使用率超过 90 就告警)

```sql
100 - avg by(instance) (rate(node_cpu_seconds_total{mode="idle",instance =~ "172.18.*"}[2m])) * 100 > 90
```

注:跟主机 CPU 监控是同一类，单独列出来放到 k8s 里面了

线上 k8s 节点状态监控

```sql
kube_node_status_condition{condition="Ready",status="true", node=~"cn-hangzhou.*"} == 0
```

线上 k8s 节点磁盘监控(磁盘使用率超过 90 就告警)

```sql
100 - (node_filesystem_avail_bytes{instance=~"172.18.*"} * 100) / node_filesystem_size_bytes{instance=~"172.18.*"} > 90
```

注:跟主机 磁盘 监控是同一类，单独列出来放到 k8s 里面了

线上 k8s 节点内存监控告警(内存使用率超过 95 就告警)

```sql
100 - node_memory_MemAvailable_bytes{instance =~ "172.18.*"} / node_memory_MemTotal_bytes{instance =~ "172.18.*"} * 100 > 95
```

### 三、pod 相关

线上 pod CPU 使用率监控 (5 分钟之内，CPU 使用率超过 80 就告警)

```sql
(sum by(name) (rate(container_cpu_usage_seconds_total{image!="",namespace=~"moushi-(prod|pre)"}[5m])) * 100) > 80
```

线上 pod 内存使用率监控(pod 内存使用率超过 80 就告警)

```sql
(sum(container_memory_working_set_bytes{id!="/",namespace=~"moushi-(prod|pre)"}) BY (instance,name,container,pod_name,namespace) / sum(container_spec_memory_limit_bytes{id!="/",namespace=~"moushi-(prod|pre)"} > 0) BY (instance,name,container,pod_name,namespace) * 100) > 98
```

线上 pod 网络流量监控(pod 流入流量 1 分钟之内超过 50M 就告警)

```sql
sum by(name) (irate(container_network_receive_bytes_total{container_name="POD",namespace=~"moushi-(pre|prod)"}[1m])) > 1024 * 1024 * 50
```

pod 启动失败监控(10 分钟之内，启动失败一次就告警)

```sql
avg_over_time(kube_pod_container_status_waiting_reason{namespace=~"moushi-(pre|prod)",reason!="PodInitializing", reason!="ContainerCreating"}[10m]) == 1
```

线上 pod 重启监控(5 分钟之内，pod 重启次数超过一次就告警)

```sql
increase(kube_pod_container_status_restarts_total{namespace=~"moushi-(prod|pre)"}[5m]) > 0
```
