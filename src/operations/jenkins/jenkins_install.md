# jenkins 安装与配置

> [jenkins 下载地址](https://mirrors.jenkins.io/)

## 前期准备

安装 openjdk 11

```shell
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# 安装openjdk 11
yum install fontconfig java-17-openjdk
```

## 安装

命令行安装

```shell
yum install jenkins
```

## 问题排查

问题: `Failed to start Jenkins Continuous Integration Server`

参考 👉🏻[Jenkins：Failed to start Jenkins Continuous Integration Server 问题解决](https://blog.csdn.net/weixin_45428910/article/details/130315891)

问题: `AWT is not properly configured on this server. Perhaps you need to run your container with "-Djava.awt.headless=true"? See also: https://www.jenkins.io/redirect/troubleshooting/java.awt.headless`

参考 👉🏻[Jenkins war update caused issue when Ruby runtime plugin is installed](https://community.jenkins.io/t/jenkins-war-update-caused-issue-when-ruby-runtime-plugin-is-installed/3282)

问题: `plugin install Failed`

参考 👉🏻[jekins 安装插件报错](https://blog.51cto.com/ckl893/2839767)

**参考文档**

- [Jenkins Redhat Packages](https://pkg.jenkins.io/redhat-stable/)
