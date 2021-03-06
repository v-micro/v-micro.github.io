## kubernetes基本操作

> k8s集群解释
```
主要包括两部分：

1.一个Master节点（主节点）
Master节点主要还是负责管理和控制。
Master节点包括API Server、Scheduler、Controller manager、etcd。
API Server是整个系统的对外接口，供客户端和其它组件调用，相当于“营业厅”。
Scheduler负责对集群内部的资源进行调度，相当于“调度室”。
Controller manager负责管理控制器，相当于“大总管”。


2.一群Node节点（计算节点）
Node节点是工作负载节点，里面是具体的容器。
Node节点包括Docker、kubelet、kube-proxy、Fluentd
Docker，不用说了，创建容器的。
Kubelet，主要负责监视指派到它所在Node上的Pod，包括创建、修改、监控、删除等。
Kube-proxy，主要负责为Pod对象提供代理。
Fluentd，主要负责日志收集、存储与查询。
```

>k8s关键字说明

```
pods
在Kubernetes系统中，调度的最小颗粒不是单纯的容器，而是抽象成一个Pod，Pod是一个可以被创建、销毁、调度、管理的最小的部署单元。比如一个或一组容器。

Replication Controllers 
Replication Controller是Kubernetes系统中最有用的功能，实现复制多个Pod副本，往往一个应用需要多个Pod来支撑，并且可以保证其复制的副本数，即使副本所调度分配的主宿机出现异常，通过Replication Controller可以保证在其它主宿机启用同等数量的Pod。Replication Controller可以通过repcon模板来创建多个Pod副本，同样也可以直接复制已存在Pod，需要通过Label selector来关联。

Services 
Services是Kubernetes最外围的单元，通过虚拟一个访问IP及服务端口，可以访问我们定义好的Pod资源，目前的版本是通过iptables的nat转发来实现，转发的目标端口为Kube_proxy生成的随机端口，目前只提供GOOGLE云上的访问调度，如GCE。 

Labels 
Labels是用于区分Pod、Service、Replication Controller的key/value键值对，仅使用在Pod、Service、 Replication Controller之间的关系识别，但对这些单元本身进行操作时得使用name标签。 

Proxy
Proxy不但解决了同一主宿机相同服务端口冲突的问题，还提供了Service转发服务端口对外提供服务的能力，Proxy后端使用了随机、轮循负载均衡算法。

Deployment
Kubernetes Deployment提供了官方的用于更新Pod和Replica Set（下一代的Replication Controller）的方法Kubernetes Deployment提供了官方的用于更新Pod和Replica Set（下一代的Replication Controller）的方法，您可以在Deployment对象中只描述您所期望的理想状态（预期的运行状态），Deployment控制器为您将现在的实际状态转换成您期望的状态，例如，您想将所有的webapp:v1.0.9升级成webapp:v1.1.0，您只需创建一个Deployment，Kubernetes会按照Deployment自动进行升级。现在，您可以通过Deployment来创建新的资源（pod，rs，rc），替换已经存在的资源等。   Deployment集成了上线部署、滚动升级、创建副本、暂停上线任务，恢复上线任务，回滚到以前某一版本（成功/稳定）的Deployment等功能，在某种程度上，Deployment可以帮我们实现无人值守的上线，大大降低我们的上线过程的复杂沟通、操作风险
```

> 简介k8s发布流程

```
1.用户向Gitlab提交代码，代码中必须包含Dockerfile
2.将代码提交到远程仓库
3.用户在发布应用时需要填写git仓库地址和分支、服务类型、服务名称、资源数量、实例个数，确定后触发Jenkins自动构建
4.Jenkins的CI流水线自动编译代码并打包成docker镜像推送到Harbor镜像仓库
5.Jenkins的CI流水线中包括了自定义脚本，根据我们已准备好的kubernetes的YAML模板，将其中的变量替换成用户输入的选项
6.生成应用的kubernetes YAML配置文件
7.更新Ingress的配置，根据新部署的应用的名称，在ingress的配置文件中增加一条路由信息
8.更新PowerDNS，向其中插入一条DNS记录，IP地址是边缘节点的IP地址。关于边缘节点，请查看边缘节点配置
9.Jenkins调用kubernetes的API，部署应用
```

> 安装kubernetes服务分配

```
master：10.11.4.39
服务：apiserver, controller-manager, scheduler,etcd

node：10.11.4.62 
服务：flannel, docker, kubelet, kube-proxy
```

> master 节点配置安装

```
安装etcd
> yum install -y etcd
修改配置
> vim /etc/etcd/etcd.conf    #修改部分内容如下
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://10.11.4.39:2379"
启动etcd
> systemctl start etcd
配置etcd的flannel信息
> etcdctl -C 10.11.4.39:2379 set /atomic.io/network/config '{"Network":"172.17.0.1/16"}'

安装kubernetes-master
> yum install kubernetes-master
修改配置文件：
> vim /etc/kubernetes/apiserver
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
KUBE_ETCD_SERVERS="--etcd-servers=http://10.11.4.39:2379"
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger
其中KUBE_ADMISSION_CONTROL的原有的SecurityContextDeny和ServiceAccount是权限相关的配置需要去掉。
配置全局配置文件:
> vim /etc/kubernetes/config
KUBE_MASTER="--master=http://10.11.4.39:8080"
启动:
> systemctl start kube-apiserver kube-scheduler kube-controller-manager
测试:
访问测试master服务： http://10.11.4.39:8080/
```

> node 节点配置安装

```
安装 docker flannel kubernetes-node
> yum install -y docker flannel kubernetes-node

配置flanneld
> vim /etc/sysconfig/flanneld
FLANNEL_ETCD_ENDPOINTS="http://10.11.4.39:2379"
FLANNEL_ETCD_PREFIX="/atomic.io/network"

配置全局文件
> vim /etc/kubernetes/config
KUBE_MASTER="--master=http://10.11.4.39:8080"

配置kubelet组件
> vim /etc/kubernetes/kubelet
KUBELET_HOSTNAME="--hostname-override=node1"
KUBELET_API_SERVER="--api-servers=http://10.11.4.39:8080"

启动服务
> systemctl start kubelet kube-proxy

测试集群
在master节点运行：kubectl get nodes
显示
NAME        STATUS    AGE
127.0.0.1   Ready     21s
```

> kubectl常用基本操作

```
根据yaml 配置文件一次性创建service、rc
> kubectl create -f my-service.yaml -f my-rc.yaml

获取运行中的deployment
> kubectl get deployment -o wide

//查看详情 -o wide

查看资源对象
查看所有pod 列表
> kubectl get pod -n <namespace>
查看RC和service 列表
> kubectl get rc,svc

查看资源信息
显示Node的详细信息
> kubectl describe node <node-name>
显示Pod的详细信息
> kubectl describe pod <pod-name>

删除资源对象
基于pod.yaml 定义的名称删除pod
> kubectl delete -f pod.yaml
删除所有包含某个label的pod 和service
> kubectl delete pod,svc -l name=<label-name>
删除所有Pod
> kubectl delete pod --all
删除rc/svc
> kubectl delete rc/nginx-master

执行容器的命令
执行pod 的date 命令
> kubectl exec <pod-name> -- date
通过bash 获得pod中某个容器的TTY，相当于登陆容器
> kubectl exec -it <pod-name> -c <container-name> -- bash

查看容器的日志
> kubectl logs <pod-name>

查看未认证的节点
kubectl get csr

```

> 示例部署第一个应用

```
vim nginx-rc.yaml
内容：
piVersion: v1
kind: ReplicationController
metadata:
  name: nginx-controller
spec:
  replicas: 2
  selector:
    name: nginx
  template:
    metadata:
      labels:
        name: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80



vim nginx-service-nodeport.yaml
内容:
piVersion: v1
kind: Service
metadata:
  name: nginx-service-nodeport
spec:
  ports:
    - port: 8000
      targetPort: 80
      protocol: TCP
  type: NodePort
  selector:
    name: nginx


//查看
> kubectl create -f redis-master-controller.yaml
> kubectl get rc
> kubectl get pods

```

> 问题解决

```
问题1：registry.access.redhat.com/rhel7/pod-infrastructure:latest
解决：
> yum install rhsm
> wget http://mirror.centos.org/centos/7/os/x86_64/Packages/python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm
> rpm2cpio python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm | cpio -iv --to-stdout ./etc/rhsm/ca/redhat-uep.pem | tee /etc/rhsm/ca/redhat-uep.pem
> docker pull registry.access.redhat.com/rhel7/pod-infrastructure:latest
```

> 使用安装脚本安装

```
https://github.com/lizhenliang/ansible-install-k8s
```


> k8s yml参数注释

```
cat /opt/linux-yml/nginx.yml

kind: Deployment #对象类型，是deployment控制器，kubectl explain Deployment
apiVersion: extensions/v1beta1  #指定这个对象属于哪个api分组，它的API版本是多少，
								# kubectl explain Deployment.apiVersion
metadata:     #pod的元数据信息，kubectl explain Deployment.metadata
  labels:     #自定义pod的标签，kubectl explain Deployment.metadata.labels
    app: linux-nginx-deployment-label #标签名称为app值为linux-nginx-deployment-label， 后面会用到此标签
  name: linux-nginx-deployment        #pod的名称 
  namespace: linux-ns                 #把pod的创建到指定的namespace，默认是default
spec:            #定义deployment中容器的详细信息，kubectl explain Deployment.spec
  replicas: 1    #创建出的pod的副本数，即多少个pod，默认值为1 
  selector:      #定义标签选择器
    matchLabels: #定义匹配的标签，必须要设置
      app: linux-nginx-selector   #匹配的目标标签
  template:      #定义模板，必须定义，模板是起到描述要创建的pod的作用
    metadata:    #定义模板元数据
      labels:    #定义模板label，Deployment.spec.template.metadata.labels
        app: linux-nginx-selector   #定义标签，必须等于 matchLabels 定义的标签
    spec:          #定义pod信息
      containers:  #定义pod中容器列表，可以多个至少一个，pod不能动态增减容器(想新添加容器，需要把原有pod删除，重新创建pod)
      - name: linux-nginx-container                      #容器名称
        image: harbor.linux.com/web-images/nginx:1.14.2  #镜像地址
        #command: ["/apps/tomcat/bin/run_tomcat.sh"]     #容器启动执行的命令或脚本           
        #imagePullPolicy: IfNotPresent                   #本地有指定的镜像，就不去harbor中拉取
        imagePullPolicy: Always    #拉取镜像策略；不管本地有没有所指定的镜像，都去harbor拉取
        ports:                #定义容器端口列表
        - containerPort: 80   #定义一个端口
          protocol: TCP       #端口协议,TCP或者UDP
          name: http          #端口名称
        - containerPort: 443  #定义一个端口
          protocol: TCP       #端口协议 
          name: https         #端口名称
        env:                  #给容器传递配置的环境变量
        - name: "password"    #变量名称。必须要用引号引起来
          value: "123456"     #当前变量的值
        - name: "age"         #另一个变量名称 
          value: "18"         #另一个变量的值
        resources:            #对容器资源的请求设置和限制设置
          limits:             #硬限制；限制容器使用宿主机资源上限
            cpu: 2            #cpu的限制，单位为core数，可以写0.5或者500m(毫核)等CPU压缩值，1000毫核
            memory: 2Gi       #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
          requests:  #资源请求的设置；当构建pod时，过滤出哪些node节点的剩余资源符合下面所指定的值，则创建pod时，才有可能把pod往该node节点调度；如果不够下面所指定的值，则创建pod时不会把pod往不符合要求的node上调度
            cpu: 1        #cpu请求数，容器启动的初始可用数量,可以写0.5或者500m等CPU压缩值
            memory: 512Mi #内存请求大小，容器启动的初始可用数量，用于调度pod时候使用

---
kind: Service    #对象类型为service；pod要想被外部访问，就必须指定一个service，service会在宿主机打开一个nodeport；service不指定nodeport，则该service仅限于k8s内部调用	     
apiVersion: v1   #service API版本，service.apiVersion 
metadata:        #定义service元数据，service.metadata
  labels:        #自定义service的标签，service.metadata.labels
    app: linux-nginx     #定义service标签的内容
  name: linux-nginx-spec #定义service的名称，此名称会被DNS解析
  namespace: linux-ns    #该service隶属于的namespaces名称，即把service创建到哪个namespace里面，并且service必须与pod在同一个namespace中，否则在调用时，service找不到所调用的pod 
spec:               #定义service的详细信息，service.spec
  type: NodePort    #NodePort表示在宿主机打开一个监听端口；service的类型，定义服务的访问方式，默认为ClusterIP(不会再宿主机打开一个监听端口，只为k8s内部提供服务)，service.spec.type
  ports:            #定义访问端口， service.spec.ports
  - name: http      #定义一个端口名称
    port: 80        #service的端口 
    protocol: TCP   #协议类型;service是k8s内部的一个传输层的负载均衡 
    targetPort: 80  #目标pod的端口 
    nodePort: 30001 #node节点暴露的端口
  - name: https     #SSL 端口
    port: 443       #service 443端口 
    protocol: TCP   #端口协议 
    targetPort: 443 #目标pod端口 
    nodePort: 30043 #node节点暴露的SSL端口
  selector:   #service的标签选择器，定义把用户请求转发给指定label标签的pod；service会把符合label标签的pod添加到后端可被调度的endpoint列表中
    app: linux-nginx-selector #将流量转发到指定的pod上，须等于Deployment.spec.selector.matchLabels


编写的测试用例

命名规则 项目-服务-deploy.yaml
vim volist-test-deploy.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: volist-test
spec:
  replicas: 4
  selector:
    matchLabels:
      app: volist-test
  template:
    metadata:
      labels:
        app: volist-test
    spec:
      containers:
      - name: volist-test
        image: nginx:1.14.0  #镜像地址
        imagePullPolicy: Always
        ports:
        - containerPort: 80
          protocol: TCP
          name: http
        resources:
          limits:
            cpu: 0.5
            memory: 512Mi
          requests:
            cpu: 0.5
            memory: 512Mi

---
kind: Service   
apiVersion: v1
metadata:
  name: volist-test
spec:
  type: NodePort    #NodePort表示在宿主机打开一个监听端口；service的类型，定义服务的访问方式，默认为ClusterIP(不会再宿主机打开一个监听端口，只为k8s内部提供服务)，service.spec.type
  ports:
  - name: http      #定义一个端口名称
    port: 80        #service的端口 
    protocol: TCP   #协议类型;service是k8s内部的一个传输层的负载均衡 
    targetPort: 80  #目标pod的端口 
    nodePort: 30001 #node节点暴露的端口
  selector:
    app: volist-test

```
