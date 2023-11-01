# 这里主要记录 docker 使用过程中遇到的问题以及解决办法

## 1、pip install --upgrade pip 报错 RuntimeError: can't start new thread

**问题描述**

python 项目打包，docker 基础镜像用的是`python:3.8`,然后构建报错如下

**RuntimeError: can't start new thread**

✅ **解决方法**

> 基础镜像问题，将镜像由`python:3.8` 换成`python3.6`即可，前提是对`python`要求版本不高，如果对版本要求过高，需要自己单独做镜像
