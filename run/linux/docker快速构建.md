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
