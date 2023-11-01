# git http 方式免密克隆代码

免密拉取配置
执行以下命令，在`home`目录下，创建`.git-credentials`文件 写入如下内容

```shell
cat > $HOME/.git-credentials << EOF
https://username:password@git.gitxx.com
EOF
```

其中 `username`替换为自己的用户名，`password`替换为登录的密码

比如 <https://einsli:passw0rd@git.einsli.com>

执行以下命令更新配置

```shell
git config --global credential.helper store
```

执行以下命令验证

```shell
git config --list
```

输出如下

```PlainText
credential.helper=store
```

或者查看一下`$HOME/.gitconfig` 文件

```shell
cat $HOME/.gitconfig
```

输出如下

```PlainText
[credential]
	helper = store
```

这时候再用 git 拉取仓库就不需要输入用户名和密码了。
