# node 安装与配置

> [node 下载地址](https://nodejs.org/en/about/previous-releases)

命令行安装

```shell
yum -y install wget && wget https://nodejs.org/download/release/v16.20.2/node-v16.20.2-linux-x64.tar.gz

tar -zxvf node-v16.20.2-linux-x64.tar.gz -C /opt/

ln -s /opt/node-v16.20.2-linux-x64 /opt/node
```

配置环境变量

配置到 `~/.bashrc`

```PlainText
export NODE_HOME=/opt/node/bin
export PATH=$PATH:$NODE_HOME
```

```shell
source ~/.bashrc
```

验证安装

```shell
node -v
```

**centos 7 安装 node18 失败**

参考:

- [centos7 安装 node-v18 版本真是难呢](https://www.cnblogs.com/grey-wolf/p/17788353.html)
- [node: /lib64/libm.so.6: version `GLIBC_2.27' not found](https://www.cnblogs.com/dingshaohua/p/17103654.html)
