## 快速构建

> 快速构建redis

```bin/bash
//下载docker镜像
docker pull redis
//启动redis 并设置密码为123456
docker run -d --name myredis -p 6379:6379 redis --requirepass "123456"
```

> 快速构建mysql

```bin/bash
//下载mysql 5.7
docker pull  mysql:5.7 
//运行mysql镜像
docker run -p 3332:3306 --name mysql04 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

> 快速构建rabbitmq

```bin/bash
//下载rabbitmq  management带访问界面
docker pull rabbitmq:management
//运行镜像
docker run -d --name my_rabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 rabbitmq:management
```

> 快速构建nginx

```bin/bash
//下载镜像
docker pull nginx
//运行镜像
docker run -d -p 81:80 --name some-nginx81 some-content-nginx
//拷贝静态资源文件到镜像
docker cp index.html a4274a5e3589:/usr/share/nginx/html/index.html
```

> 快速构建registry私有库

```
//下载镜像包
docker pull registry:latest

//运行镜像并配置宿主机镜像位置
//镜像存储目录/opt/data/registry
docker run -d --name=MyRegistry -p 5001:5000 -v /opt/data/registry:/tmp/registry registry:latest
```

> 快速构建elasticsearch

```
//下载镜像
docker pull elasticsearch:7.10.1
//运行镜像
docker run -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" --name MyElasticsearch elasticsearch:7.10.1
//端口解释
# 9200 主要用于外部 http通讯
# 9300 作为tcp协议 es集群就是通过9300通讯
//报错解决
-e "ES_JAVA_OPTS=-Xms64m -Xmx251m"
```


> 快速构建kibana

```
//下载镜像
docker pull kibana:7.10.1
//运行镜像
docker run -d -p 5601:5601 --net=host --name MyKibana kibana:7.10.1
//运行kibana 并连接es
docker run -d -p 5601:5601 --net=host --name MyKibana -e ELASTICSEARCH_URL=http://127.0.0.1:9200 kibana:7.10.1
//如法加载则
修改镜像配置然后重启
//配置
vim  /usr/share/kibana/config/kibana.yml
elasticsearch.hosts: [ "http://127.0.0.1:9200" ]     es 端口
i18n.locale: "zh-CN"    //设置中文

```
