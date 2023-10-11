# supervisor 安装与配置

执行以下命令安装 supervisord

```shell
yum install supervisor
```

查看版本

```shell
supervisord --version
```

如果需要配置界面访问需要修改`supervisord.conf`

路径如下

```PlainText
/etc/supervisord.conf
```

修改以下几行

```PlainText
#启用访问web控制界面，inet_http_server 修改如下
[inet_http_server]
port=*:9001

#设置账户和密码
username=user
password=passw0rd

#include part
[include]
files = supervisord.d/*.ini
```

子进程配置文件路径：`/etc/supervisord.d/`

这是官网的配置文件 👉🏻[supervisord 配置文件参考](http://www.supervisord.org/configuration.html)

在`/etc/supervisord.d/` 创建`test-server.ini`,用于启动一个`java`服务

```ini
[program:javaserver]
command=/usr/bin/java -jar app.jar ; 输入执行命令，这里表示 dotnet  demo.dll
directory=/opt/test-server ; 应用程序根目录
autostart=true ; 是否自动启动，当 supervisor 加载该配置文件的时候立即启动它
autorestart=true ; 是否自动重启，当执行 dotnet  Deploy.Linux.dll 启动失败时，会重复的自动重启
logfile_maxbytes=50MB ; 该配置文件输出单个日志文件的大小
logfile_backups=10 ; 日志备份个数
loglevel=info ; 记录日志级别
stderr_logfile=/data/logs/java-server.err.log ; 指定标准错误输出日志文件
stdout_logfile=/data/logs/java-server.access.log ; 指定标准输出日志文件
environment=ASPNETCORE_ENVIRONMENT=Production ; 可配置环境变量，该环境变量将通过执行 dotnet  Deploy.Linux.dll 命令的时候传入到 .NET Core 应用程序中
user=root ;启动服务的用户
stopsignal=INT
redirect_stderr=true
```

创建日志路径

```shell
mkdir -p /data/logs/
```

重新加载配置

```shell
systemctl daemon-reload
```

启动`supervisord`并设置为开机启动

```shell
# 启动supervisord
systemctl start supervisord

# 设置开机启动
systemctl enable supervisord

# 查看supervisord 状态
systemctl status supervisord
```
