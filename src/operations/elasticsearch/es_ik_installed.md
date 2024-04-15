# elasticsearch ik 分词器安装(7.10.0 版本)

👉🏻 [elasticsearch ik 分词器 github](https://github.com/medcl/elasticsearch-analysis-ik)

👉🏻 [elasticsearch ik 分词器 7.10.0 下载地址](https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.10.0/elasticsearch-analysis-ik-7.10.0.zip)

## 前期准备

`elasticserach` 已经安装部署完成

安装部署请参考 👉🏻 [elasticsearch docker-compose 安装与配置(7.10.0 版本)](es_docker_installed.md)

**1、停止服务**

```shell
cd /data/elasticserach/
.stop.sh
```

**2、准备 plugins 文件**

```shell
mkdir /data/elasticsearch/plugins

chown -R 1000:1000 /data/elasticsearch/plugins
```

**3、更新 docker-compose.yaml**文件

需要将`/data/elasticsearch/docker-compose.yaml`文件做如下更改

```yaml
volumes:
  - /etc/localtime:/etc/localtime
  - ./es_config:/usr/share/elasticsearch/config
  - ./es_data:/usr/share/elasticsearch/data
+  - ./plugins:/usr/share/elasticsearch/plugins
```

完整`docker-compose.yaml`文件如下

```yaml
version: '3.1'

services:
  elasticsearch:
    image: 'docker.elastic.co/elasticsearch/elasticsearch:${ES_VERSION}'
    container_name: elasticsearch
    environment:
      TZ: Asia/Shanghai
      LANG: en_US.UTF-8
      discovery.type: single-node
      ES_JAVA_OPTS: '-Xmx512m -Xms512m'
      ELASTIC_PASSWORD: 'Passw0rd'
    volumes:
      - /etc/localtime:/etc/localtime
      - ./es_config:/usr/share/elasticsearch/config
      - ./es_data:/usr/share/elasticsearch/data
      - ./plugins:/usr/share/elasticsearch/plugins # 新增
    ports:
      - '12200:9200'
      - '12300:9300'
    user: '1000:1000'
    networks:
      - ops_network

  kibana:
    depends_on:
      - elasticsearch
    image: 'docker.elastic.co/kibana/kibana:${ES_VERSION}'
    container_name: kibana
    environment:
      - ELASTICSEARCH_URL=http://elasticsearch:9200
    volumes:
      - /etc/localtime:/etc/localtime
      - ./kibana_config:/usr/share/kibana/config
    ports:
      - '8601:5601'
    links:
      - elasticsearch
    networks:
      - ops_network

networks:
  ops_network:
    external: true
```

**3、准备 ik 文件并配置**

```shell
# 新建zips 文件，用于备份存放zip文件
mkdir -p /data/elasticsearch/zips

# 新建 analysis-ik 插件名称
mkdir -p /data/elasticsearch/plugins/analysis-ik

# 下载ik 分词器插件
cd /data/elasticsearch/plugins/analysis-ik
yum -y install wget && wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.10.0/elasticsearch-analysis-ik-7.10.0.zip
yum -y install unzip && unzip elasticsearch-analysis-ik-7.10.0.zip


# zip 文件移动到创建好的zips 目录下
mv /data/elasticsearch/plugins/analysis-ik/elasticsearch-analysis-ik-7.10.0.zip /data/elasticsearch/zips

# 再次重新赋予下权限
chown -R 1000:1000 /data/elasticsearch/plugins/
```

**注:**如果下载失败，请通过其他渠道下载后上传到`/data/elasticsearch/plugins`目录下

目录结构如下

```PlainText
├── .env
├── docker-compose.yaml
├── es_config
├── es_data
├── kibana_config
├── plugins
├── run.sh
├── stop.sh
└── zips
```

**4、启动服务**

启动

```shell
cd /data/elsaticsearch/
./run.sh
```

**4、问题排查**

如果遇到了下面的问题

```PlainText
"Caused by: java.nio.file.FileSystemException: /usr/share/elasticsearch/plugins/httpclient-4.5.2.jar/plugin-descriptor.properties: Not a directory",
```

说明你把 ik 分词器的文件放到了`plugins`目录下，导致的报错

请参考 👉🏻 [记一次 docker 安装 elasticsearch 遇到的坑](https://blog.csdn.net/dxtljly/article/details/127102211)

**5、测试 ik 分词器**

在`kibana`中请求如下内容

```elasticsearch
GET _analyze
{
  "analyzer": "ik_max_word",
  "text":"我是中国人"
}
```

响应内容

```json
{
  "tokens": [
    {
      "token": "我",
      "start_offset": 0,
      "end_offset": 1,
      "type": "CN_CHAR",
      "position": 0
    },
    {
      "token": "是",
      "start_offset": 1,
      "end_offset": 2,
      "type": "CN_CHAR",
      "position": 1
    },
    {
      "token": "中国人",
      "start_offset": 2,
      "end_offset": 5,
      "type": "CN_WORD",
      "position": 2
    },
    {
      "token": "中国",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "国人",
      "start_offset": 3,
      "end_offset": 5,
      "type": "CN_WORD",
      "position": 4
    }
  ]
}
```
