## docker私有镜像库搭建

> 下载镜像包

```
//下载镜像包
docker pull registry:latest

//运行镜像并配置宿主机镜像位置
//镜像存储目录/opt/data/registry
docker run -d --name=MyRegistry -p 5001:5000 -v /opt/data/registry:/tmp/registry registry:latest
```

> 问题集合

```
The push refers to a repository [10.11.4.39:5001/nginx]
Get https://10.11.4.39:5001/v1/_ping: http: server gave HTTP response to HTTPS client
解释：Docker从1.3.x之后，与docker registry交互默认使用的是https，但是此处搭建私有仓库却只提供http服务，所以当和私有仓库交互时报上述错误。因此需要在启动docker server时增加启动参数为默认使用http访问

解决方案：
【推送方】
vim  /etc/docker/daemon.json
{"insecure-registries":["10.11.4.39:5001"]}
systemctl daemon-reload
systemctl restart docker

删除镜像报错
{"errors":[{"code":"UNSUPPORTED","message":"The operation is unsupported."}]}
拷贝配置文件
docker cp bb4b04962fef:/etc/docker/registry/config.yml ./
设置允许删除
vim config.yml
storage:
  delete:
    enabled: true
//配置文件放进去
docker cp ./config.yml bb4b04962fef:/etc/docker/registry/config.yml 
//重启
docker restart MyRegistry
```

> 镜像仓库操作

```
通过docker 删除镜像
docker exec <容器id> rm -rf /var/lib/registry/docker/registry/v2/repositories/<删除镜像>
```

> 远程仓库命令
```
//查看远程仓库镜像
curl -X GET http://10.11.4.39:5001/v2/_catalog

//查看远程仓库镜像标签
//image_name 镜像名称
curl -X GET http://10.11.4.39:5001/v2/<image_name>/tags/list
例子：curl -X GET http://10.11.4.39:5001/v2/nginx/tags/list

//获取镜像详细信息
//image_name 镜像名称
//image_tag 镜像标签
curl -X GET http://10.11.4.39:5001/v2/<image_name>/manifests/<image_tag>
例子：curl -X GET http://10.11.4.39:5001/v2/nginx/manifests/latest
获取digest_hash例子：curl http://10.11.4.39:5001/v2/nginx/manifests/latest -H "Accept: application/vnd.docker.distribution.manifest.v2+json"


//删除镜像
//image_name 镜像名称
curl -delete  http://10.11.4.39:5001/v2/<image_name>/manifests/<digest_hash>
例子：curl -I -X DELETE http://10.11.4.39:5001/v2/nginx/manifests/sha256:f6d0b4767a6c466c178bf718f99bea0d3742b26679081e52dbf8e0c7c4c42d74

curl -X DELETE http://10.11.4.39:5001/v2/nginx/manifests/sha256:0b159cd1ee1203dad901967ac55eee18c24da84ba3be384690304be93538bea8


//推送镜像
git push https://10.11.4.39:5001/nginx

```

>如果需要密码则需要这样启动

```
//利用docker创建密码
docker run --entrypoint htpasswd registry:latest -Bbn caihui 123456 >/home/htpasswd

//查看账号密码
cat htpasswd
zxg:$2y$05$qCY7iWVJIoOrnIp17WQOf.fcXUTo5xm4DwP3a/8ggzZlEZ3bsnonm

//启动带参数的镜像库
docker run -d -p 5000:5000 --restart=always --name registryauth -v /opt/registry:/var/lib/registry -v /auth:/auth -e "REGISTRY_AUTH=htpasswd" -e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd registry:latest 

//设置密码后需要先登录在提交镜像
docker login 10.11.4.39:5001:5000
```

