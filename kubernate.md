## 官网交互体验

```bash
kubectl version

kubectl get nodes

kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

kubectl get deployments

echo -e "\n\n\n\e[92mStarting Proxy. After starting it will not output a response. Please click the first Terminal Tab\n";  kubectl proxy

curl http://localhost:8001/version
```



## 资源对象

### Master

集群控制节点。在master上运行一下进程：

- kube-apiserver，资源的增删改查入口
- kube-controller-manager，资源对象大总管
- kube-scheduler，资源调度控制
- etcd，保存资源对象的数据

### Node

工作负载节点。在node上运行以下进程：

- kubelet: 负责pod对应容器创建，启停等任务，同时与master密切协作 
- kube-proxy: 实现service的通信和负载均衡
- docker: docker 引擎

```bash
#kubectl get nodes
#kubectl describe node <node name>
```



### Pod

如图所示，每个pod都有一个特殊的被称为根容器的Pause容器。Parse容器对应的镜像是Kubernates平台的一部分。每个pod包含一个或者多个用户业务容器。

设计Pod的目的：

- 在一组容器作为一个单元的情况下，简单的对整体进行判断和行动，引入与业务无关的Pause容器，以它的装台代表整个容器组的状态
- 多个业务容器共享Pause容器的IP，Volume，解决通信和文件共享问题

Pod的类型

- 普通的Pod
- 静态的Pod

普通的Pod一旦创建，就会放入etcd存储，随后会被Master调度到某个具体的Node上进行绑定，随后该Pod被对应Node上的kubelet进行实例化一组组相关的docker容器并启动，如图所示


所有的资源对象都可以采用yaml和json文件进行描述。如图所示

- kind:Pod 表明这是一个pod的定义
- metadata子元素name，pod的名称
- metadata子元素labels，name=myweb的标签
- spec， 包含的容器组
- pod的IP 加上容器端口，这里的containerPort，组成一个新的概念--Endpoint



### Replication Controller

RC 是Kubernetes系统中核心概念之一，简单来说，它其实定义了一个期望的场景，即声明某种Pod的副本数量在任意时刻都符合某个预期值，所以RC的定义包含以下几个部分：

- Pod期待的副本数量
- 用于筛选目标Pod的 Label Selector
- 当Pod的副本数量小于预期数量时，用于创建新的Pod的模板



### Deployment

可以看做RC的一次升级，两者的相似程度超过90%



### StatefulSet

- 每个节点都有固定的身份ID
- 集群的规模比较固定
- 集群的每个节点都是有状态 的，通常会持久化数据到永久存储中
- 如果某个节点无法工作，集群功能受损

StatefulSet就是为了解决这些问题推出的资源对象



### Service

Kubernetes的Service就是微服务架构中的一个微服务。如图显示了Pod，RC和Service之间的关系。



Service定义了一个服务的访问入口地址，前段的应用Pod通过这个入口地址访问背后的一组由Pod副本组成的集群实例，Service与其后端Pod副本集群之间则通过Label Selector来实现无缝对接的。RC的作用实际上是保证Service的服务能力和服务质量始终服务预期标准。

Service yaml定义文件



上述内容定义了一个名为tomcat-service的Service，服务端口8080，拥有【tier=frontend】标签的Pod实例都属于它，运行下面的命令创建

```bash
#创建Service
#kubectl create -f xxx.yaml

#查看Endpoint
#kubectl get endpoints

#查看Service的Cluster IP
#kubectl get svc XXX -o yaml
```



> 服务发现机制

首先 每个Service都有唯一的Cluster IP及唯一的名称，名称也是由开发者自己定义，部署时没有必要改变，所以可以被固定在配置中。

Kubernetes通过Add-On增值包引入了DNS系统，把服务名称作为DNS域名，这样程序就可以直接使用服务名来简历通信连接了。目前，Kubernetes上的大部分应用都采用DNS这种服务发现机制。



> 外部系统访问Service

Kubernetes的3种IP

- Node IP，集群种每个节点的物理网卡的IP地址
- Pod IP，每个Pod的IP地址，它是Docker0网桥的IP地址段进行分配的
- Cluster IP，它是一个虚拟的IP，更像一个伪造的IP网络
  - 仅仅作用于Service对象
  - 无法被ping
  - 只能结合Service Port组成一个具体的通信端口
  - Node IP，Pod IP与Cluster IP之间的通信，采用的是Kubernetes自己的设计的特殊路由规则

采用NodePort是解决Service被外部访问的最直接，最有效的常见做法。如图所示





