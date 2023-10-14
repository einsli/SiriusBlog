# Summary

- [简介](index.md)

- [基础运维](operations.md)

  - [linux](operations/linux.md)
    - [linux 基础配置](operations/linux/linux_base_config.md)
    - [linux 内核升级(线上)](operations/linux/linux_kernel_upgrade_online.md)
    - [linux 内核升级(离线)](operations/linux/linux_kernel_upgrade_offline.md)
    - [supervisord 安装配置](operations/linux/linux_supervisor_install.md)
  - [docker](operations/docker.md)
    - [docker 安装与配置](operations/docker/linux_docker_installed.md)
    - [docker-compose 安装](operations/docker/linux_docker_compose_installed.md)
  - [redis](operations/redis.md)
    - [redis 二进制安装与配置](operations/redis/redis_binary_installed.md)

- [云原生](cloud_operations.md)
  - [监控](cloud_operations/monitor.md)
    - [夜莺监控安装配置](cloud_operations/monitor/n9e_install.md)
    - [夜莺配置 ldap 单点登录](cloud_operations/monitor/n9e_ldap_config.md)
    - [prometheus 监控告警 sql](cloud_operations/monitor/alert_prom_sql.md)
  - [kubernetes](cloud_operations/kubernetes.md)
    - [kubernetes service account](cloud_operations/kubernetes/kubernetes_sa.md)
      - [kubernetees 基于 sa 创建 kubeconfig 访问(集群版)](cloud_operations/kubernetes/kubernetes_sa/k8s_sa_kubeconfig.md)
    - [k8s 相关问题](cloud_operations/kubernetes/kubernetes_issues.md)
      - [kubernetes secret 相关问题](cloud_operations/kubernetes/kubernetes_issues/k8s_secret_issues.md)
