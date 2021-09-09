# Docker 安装ElasticSearch记录

## ES

### ES镜像拉取

`docker pull elasticsearch:latest`

### 启动ES

`docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:5.6.12 `

-d  后台运行

--name 指定容器名称

-p 容器和宿主机端口映射

-e "discovery.type=single-node" 指定es为单节点模式

-e ES_JAVA_OPTS="-Xms512m -Xmx512m" 指定es运行内存

### 查看ES

localhost:9200 出现以下内容启动成功

```json
{
"name": "JQT9OAK",
"cluster_name": "elasticsearch",
"cluster_uuid": "b2Elai8xQh-IDP40S99SSQ",
"version": {
"number": "5.6.12", # 版本号
"build_hash": "cfe3d9f",
"build_date": "2018-09-10T20:12:43.732Z",
"build_snapshot": false,
"lucene_version": "6.6.1"
},
"tagline": "You Know, for Search"
}
```

## IK分词器安装

### 进入ES容器

`docker exec -it elasticsearch /bin/bash`

### 执行在线安装

注意选择es对应版本的IK分词器

`./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v5.6.12/elasticsearch-analysis-ik-5.6.12.zip`

### 退出ES容器

`exit`

### 重启ES

`docker restart elasticsearch`

## HEAD插件安装

es-head插件支持es几个版本。

Elasticsearch 5.x: docker run -p 9100:9100 mobz/elasticsearch-head:5
Elasticsearch 2.x: docker run -p 9100:9100 mobz/elasticsearch-head:2
Elasticsearch 1.x: docker run -p 9100:9100 mobz/elasticsearch-head:1

### 镜像拉取

`docker pull  mobz/elasticsearch-head:5`

### 启动

`docker run -p 9100:9100 mobz/elasticsearch-head:5`

### 修改ES跨域

`docker exec -it elasticsearch bash`

`cd config`

`vim elasticsearch.yml`

由于容器中没有vim 所以需要安装

更新 apt-get `apt-get update`

安装vim `apt-get install vim`

`vim elasticsearch.yml`

文件最后添加如下内容

`http.cors.enabled: true
http.cors.allow-origin: "*"`

退出vim

ESC - `:wq`

重启ES

`docker restart elasticsearch`

## 问题及解决方案

1. 修改ES配置文件时文件出错保存了，但是容器已经无法启动，无法通过`docker exec -it elasticsearch bash` 进入容器内部

   解决方案：将容器内的文件copy到宿主机中修改然后再copy回去

   查看日志

   `docker logs elasticsearch`

   找到原因是由于添加的内容格式不对无法解析

   copy命令 `docker cp 宿主机路径 容器名:容器文件路径(/usr/share/elasticsearch)+文件名`

   `docker cp elasticsearch:/usr/share/elasticsearch/config/elasticsearch.yml D:\elasticsearch.yml`

   本地修改完成后copy到容器内

   `docker cp D:\elasticsearch.yml elasticsearch:/usr/share/elasticsearch/config/elasticsearch.yml`

   重启es即可

2. vim编辑器临时文件问题

   swap file ".elasticsearch.yml.swp" already exists!

   原因：使用vim编辑文件实际是先copy一份临时文件并映射到内存给你编辑， 编辑的是临时文件， 当执行：w后才保存临时文件到原文件，执行：q后才删除临时文件。

   解决方式：

   1. [O]pen Read-Only, (E)dit anyway, (R)ecover, (D)elete it, (Q)uit, (A)bort: D 在提示后面直接删除

   2. 进入目录 rm .*.swp 删除临时文件如  进入ES容器内删除elasticsearch.yml.swp 

      `docker exec -it elasticsearch bash`

      `cd config`

      `rm elasticsearch.yml`

