# 导论




- docker 概述

- docker 安装

- docker 命令

- docker镜像

- 容器数据卷

- docker File

- docker网络原理

- IDEA整合docker

- docker compose

- docker swarm

- CI/CD jenkins







## Docker历史



2010年，dotcloud公司做pass云计算服务，LXC有关的容器技术，他们讲自己的技术命名为docker！

2013年，dotcloud公司开源docker后，越来越多人关注，火了。每个月更新一个版本。

2014年4月9日，docker1.0发布。

为什么这么火？docker十分轻巧！


> 和vm对比



vm，centos原生镜像（一个电脑），隔离需要开启多个虚拟机！ 几个g， 几分钟

docker隔离，镜像（最核心的环境 4m + jdk +mysql） 十分的小巧， 几个m，几秒钟


> 参考



[官网](https://www.docker.com/)

[文档](https://docs.docker.com/) 

[仓库](https://hub.docker.com/)


## Docker能干什么



> 容器化技术



容器化技术不是模拟一个完整的 操作系统







比较虚拟化技术和容器化技术



- 传统虚拟机技术，虚拟出一套硬件，一个操作系统，然后在这个系统上安装和运行软件

- 容器是没有自己的内核的，应用运行在宿主机内，也没有虚拟硬件，所以就轻便了

- 每个容器间是隔离的，每个容器都有自己的文件系统，互不影响







> DevOps 



- 更快速的交付和部署

- 更便捷的的升级和扩缩容

- 更简单的系统运维

- 更搞笑的计算资源利用



Docker是内核级的虚拟化，可以在一个物理机上运行很多的容器实例，服务器的性能可以压榨到极致！







## 安装

### 基本组成



**镜像（image）**：就好比一个模板，来创建容器服务，tomcat镜像 --->run ---> tomcat01 容器，通过这个镜像运行多个容器



**容器（container）**：Docker利用容器技术，独立运行一个或者一组通过镜像创建的应用。可以理解为一个非常简陋的Linux系统



**仓库（repository）：**存放镜像的地方。Dockerhub（默认为国外，速度较慢）



### 安装步骤

```
#1，卸载旧的
sudo yum remove docker \\
                 docker-client \\
                 docker-client-latest \\
                 docker-common \\
                 docker-latest \\
                 docker-latest-logrotate \\
                 docker-logrotate \\
                 docker-engine

#2.需要的安装包
yum install -y yum-utils

#3.设置镜像仓库
sudo yum-config-manager \\
   --add-repo \\
   http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#更新索引
yum makecache fast

#4.安装docker
yum install docker-ce docker-ce-cli containerd.io

#5.启动
systemctl start docker

#6.版本
docker version

#7.测试
docker run hello-world

#8. 查看hello-world
docker images

```







> 卸载



```bash
yum remove docker-ce docker-ce-cli containerd.io
rm -rf /var/lib/docker
rm -rf /var/lib/containerd

```







## Command line

### 帮助命令


```hash
docker version  #显示版本信息
docker info     #
docker 命令 --help
```



帮助文档的地址： https://docs.docker.com/engine/reference/commandline/cli/







### 镜像命令



```shell
docker images
docker search mysql
docker search mysql --filter=STARS=3000
docker pull --help
docker pull mysql
```



```bash
[root@192 ~]# docker pull mysql
Using default tag: latest  #如果不写tag，默认为latest
latest: Pulling from library/mysql
69692152171a: Pull complete   #分层下载， image的核心
1651b0be3df3: Pull complete
951da7386bc8: Pull complete
0f86c95aa242: Pull complete
37ba2d8bd4fe: Pull complete
6d278bb05e94: Pull complete
497efbd93a3e: Pull complete
f7fddf10c2c2: Pull complete
16415d159dfb: Pull complete
0e530ffc6b73: Pull complete
b0a4a1a77178: Pull complete
cd90f92aa9ef: Pull complete
Digest: sha256:d50098d7fcb25b1fcb24e2d3247cae3fc55815d64fec640dc395840f8fa80969
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest  #真实地址

[root@192 ~]# docker pull mysql:5.7    #指定版本下载镜像
[root@192 ~]# docker rmi c0cdc95609f1  #删除镜像
[root@192 ~]# docker rmi -f $(docker images -aq) #删除全部的镜像

```







### 容器命令



#### **新建容器并启动**



```bash
docker run [可选参数] image
#参数说明
--name=“Name” 起个名字，区分容器
-d            后台方式运行
-it           使用交互方式运行，进入容器查看内容
-p            指定容器端口
	-p ip：主机端口：容器端口
	-p 主机端口：容器端口
	-p 容器端口
-P            随机指定端口
```



```BASH
#启动并进入容器
[root@192 ~]# docker run -it centos /bin/bash
[root@83929b2c631e /]# ls
bin  etc   lib    lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
#退出容器
[root@83929b2c631e /]# exit
exit
```



#### **列出所有的容器**



```BASH
#docker ps 命令

     #列出当前正在运行的容器

-a    #列出当前正在运行的容器+ 历史运行的容器

-n=？ #列出最近创建的容器

-q    #只显示容器编号

[root@192 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@192 ~]# docker ps -a
CONTAINER ID   IMAGE         COMMAND       CREATED          STATUS                          PORTS     NAMES
83929b2c631e   centos        "/bin/bash"   3 minutes ago    Exited (0) About a minute ago             elegant\_blackburn
eace531a4318   hello-world   "/hello"      45 minutes ago   Exited (0) 45 minutes ago                 determined\_matsumoto
```



#### **退出容器**



```
exit          #退出容器并停止
CTRL + P + Q  #退出容器不停止
```



#### **删除容器**



```
docker rm 容器id                
docker rm -f $(docker ps -aq)
docker ps -a -q|xargs docker rm
```



#### **启动和停止容器**



```BASH
docker start  容器ID
docker restart 容器ID
docker stop 容器ID
docker kill  容器ID
```







### 常用重要命令



#### **后台启动容器**



```bash
[root@192 ~]# docker run -d centos
567e495b338c36ee7e06c5bce654f3a27e79dbbd0d5de1e0559d49a5b488dde0
[root@192 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
#常见的坑，docker容器使用后台运行，就必须有一个前台进程，docker发现没有前台应用，就会自动停止。

```



#### **查看日志**



```bash
#	编写一段脚本
[root@192 ~]# docker run -d centos /bin/sh -c "while true;do echo qj;sleep 2;done"  
[root@192 ~]# docker ps
#显示日志
[root@192 ~]# docker logs -ft --tail 10 f4620a87216a
```



#### **查看容器里进程信息**



```BASH
[root@192 ~]# docker top 4589010de709
UID                 PID                 PPID                C                   STIME     
root                9683                9663                0                   11:21     root                9744                9683                0                   11:22      
```



#### **查看镜像元数据**



```BASH
[root@192 ~]# docker inspect 4589010de709
```



#### **进入容器**



```bash
[root@192 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
4589010de709   centos    "/bin/sh -c 'while t…"   8 minutes ago   Up 8 minutes             festive\_gauss
[root@192 ~]# docker exec -it 4589010de709 /bin/bash
[root@4589010de709 /]# ls
bin  etc   lib    lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
[root@4589010de709 /]# ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 15:21 ?        00:00:00 /bin/sh -c while true;do echo
root        277      0  0 15:31 pts/0    00:00:00 /bin/bash
root        298      1  0 15:31 ?        00:00:00 /usr/bin/coreutils --coreutils
root        299    277  0 15:31 pts/0    00:00:00 ps -ef
```



```BASH
[root@192 ~]#docker attach 4589010de709
#docker exec  打开一个新的终端
#attach       进入正在执行的终端
```



#### 拷贝文件



```BASH
#从容器里拷贝文件到宿主机上
[root@192 ~]# docker cp 4589010de709:/11 /home

```







## 作业练习



### docker 安装Nginx



1\. 搜索镜像

2\. 下载镜像

3\. 启动镜像

4\. 进入容器内查看nginx



```bash
[root@192 home]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    f0b8a9a54136   32 hours ago   133MB
hello-world   latest    d1165f221234   2 months ago   13.3kB
centos        latest    300e315adb2f   5 months ago   209MB

# -d 后台运行
# --name 起名字
# -p 宿主机端口：容器内端口

[root@192 home]# docker run -d --name nginx02 -p 3344:80 nginx
da43321a8c51f576eac48b10c1e36de09abb9db45327a4b751e9e5ed4e4636f0
#test

[root@192 home]# curl localhost:3344
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
   body {
       width: 35em;
       margin: 0 auto;
       font-family: Tahoma, Verdana, Arial, sans-serif;
   }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>

[root@192 home]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED             STATUS             PORTS                                   NAMES
da43321a8c51   nginx     "/docker-entrypoint.…"   6 minutes ago       Up 6 minutes       0.0.0.0:3344->80/tcp, :::3344->80/tcp   nginx02
4589010de709   centos    "/bin/sh -c 'while t…"   About an hour ago   Up About an hour                                           festive\_gauss
[root@192 home]# docker exec -it da43321a8c51 /bin/bash
root@da43321a8c51:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
```



> 每次修改nginx配置文件，都需要进入容器内部，比较麻烦。如何在容器外部修改文件？







### docker安装Tomcat



```bash
docker pull tomcat
docker run -d --name tomcat01 -p 3345:8080 tomcat
docker ps
docker exec -it 3ad3fdd137ea /bin/bash
```



> 每次部署项目都要进入容器，比较麻烦。如何在容器外部部署工程？







### docker安装ES



``` BASH
1, ES比较耗内存, 增加内存限制，需要1.2g内存，-e 启动参数修改
2, ES暴露端口比较多, 指定多个-p
3，ES的数据必须放在安全目录！挂载

[root@192 home]# docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES\_JAVA\_OPTS="-Xms64m -Xmx512m" elasticsearch:7.6.2 
[root@192 home]# curl localhost:9200
{
 "name" : "8adc29b75ee9",
 "cluster\_name" : "docker-cluster",
 "cluster\_uuid" : "fpWfeOHsQUKhc3vsXVY1tQ",
 "version" : {
   "number" : "7.6.2",
   "build\_flavor" : "default",
   "build\_type" : "docker",
   "build\_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
   "build\_date" : "2020-03-26T06:34:37.794943Z",
   "build\_snapshot" : false,
   "lucene\_version" : "8.4.0",
   "minimum\_wire\_compatibility\_version" : "6.8.0",
   "minimum\_index\_compatibility\_version" : "6.0.0-beta1"
 },
 "tagline" : "You Know, for Search"
}
```



### docker安装redis



```bash
#start a redis instance
docker run --name some-redis -d redis
#start with persistent storage
docker run --name some-redis -d redis redis-server --appendonly yes
#connecting via redis-cli
docker exec -it 9d9635d22920 redis-cli

```







**Additionally, If you want to use your own redis.conf ... **



You can create your own Dockerfile that adds a redis.conf from the context into /data/, like so.



```
FROM redis
COPY redis.conf /usr/local/etc/redis/redis.conf
CMD [ "redis-server", "/usr/local/etc/redis/redis.conf" ]

```



Alternatively, you can specify something along the same lines with `docker run` options.



```

$ docker run -v /myredis/conf:/usr/local/etc/redis --name myredis redis redis-server /usr/local/etc/redis/redis.conf

```



Where `/myredis/conf/` is a local directory containing your `redis.conf` file. Using this method means that there is no need for you to have a Dockerfile for your redis container.



The mapped directory should be writable, as depending on the configuration and mode of operation, Redis may need to create additional configuration files or rewrite existing ones.











## 容器数据卷


### 什么是数据卷



需求：

1\. 数据可以持久化

2\. Mysql数据可以存储在本地

3\. 容器之间数据共享



这个就是卷技术，目录的挂载，将容器里的目录挂载到linux上



**容器的持久化和同步操作，容器间也是可以数据共享的**







### 使用数据卷


### 初识dockerfile


dockfile就是用来构建镜像的命令脚本文件，镜像是一层一层的，每个命令就是一层



### 数据卷容器


删除docker01容器后，docker02，docker03照样可以访问docker01的数据卷



## dockerfile



### 介绍



是用来构建docker镜像的，命令参数脚本



**构建步骤**：



1\. 编写一个dockerfile

2\. docker build

3\. docker run

4\. docker push（dockerhub，阿里云）







### 构建过程



**基础知识**：



1\. 保留关键字必须大写字母



2\. 从上到下执行



3\. #注释



4\. 每个指令都会创建一个镜像层，并提交















### 指令



```bash
FROM    #基础镜像
MAINTAINER    #作者
RUN    #镜像构建的时候，需要运行的命令
ADD    #步骤，添加内容
WORKDIR    #工作目录
VOLUME     #数据卷  -v
EXPOSE    #暴露端口 -p
CMD      #指定这个容器启动的时候，要运行的命令,只有最后一个会生效
ENTRYPOINT    #指定这个容器启动的时候，要运行的命令，可以追加命令
ONBUILD    #当构建一个被继承的，会出发这个指令
COPY    #类似ADD，将我们的文件拷贝到镜像中
ENV    #设置环境变量  -e
```



### 实战测试 centos



99%镜像都是FROM scratch



```BASH
# 编写dockerfile文件

[root@localhost dockerfile]# cat mydockerfile
FROM centos
MAINTAINER qj<qiaojun2002@163.com>
ENV MYPATH /usr/local
WORKDIR $MYPATH
RUN yum -y install vim
RUN yum -y install net-tools
EXPOSE 80
CMD echo $MYPATH
CMD echo "-----------------build end-----------------"
CMD /bin/bash

# 构建镜像
[root@localhost dockerfile]# docker build -f mydockerfile -t mycentos:0.1 .
Sending build context to Docker daemon  2.048kB
Step 1/10 : FROM centos
latest: Pulling from library/centos
7a0437f04f83: Pull complete
Digest: sha256:5528e8b1b1719d34604c87e11dcd1c0a20bedf46e83b5632cdeac91b8c04efc1
Status: Downloaded newer image for centos:latest
---> 300e315adb2f
Step 2/10 : MAINTAINER qj<qiaojun2002@163.com>
---> Running in 946c0366bcd4
Removing intermediate container 946c0366bcd4
---> 06cb0004f73f
Step 3/10 : ENV MYPATH /usr/local
---> Running in 9a42640297ce
Removing intermediate container 9a42640297ce
---> 2dfd922108a3
Step 4/10 : WORKDIR $MYPATH
---> Running in 321f0ebe53f8
Removing intermediate container 321f0ebe53f8
---> a99d170ce4a1
Step 5/10 : RUN yum -y install vim
---> Running in 716a53b879c7
CentOS Linux 8 - AppStream                      1.5 MB/s | 6.3 MB     00:04
CentOS Linux 8 - BaseOS                         1.0 MB/s | 2.3 MB     00:02
CentOS Linux 8 - Extras                          16 kB/s | 9.6 kB     00:00
Dependencies resolved.

================================================================================

Package             Arch        Version                   Repository      Size

================================================================================
Installing:
vim-enhanced        x86\_64      2:8.0.1763-15.el8         appstream      1.4 M
Installing dependencies:
gpm-libs            x86\_64      1.20.7-15.el8             appstream       39 k
vim-common          x86\_64      2:8.0.1763-15.el8         appstream      6.3 M
vim-filesystem      noarch      2:8.0.1763-15.el8         appstream       48 k
which               x86\_64      2.21-12.el8               baseos          49 k
Transaction Summary
================================================================================
Install  5 Packages
Total download size: 7.8 M
Installed size: 30 M
Downloading Packages:
(1/5): gpm-libs-1.20.7-15.el8.x86\_64.rpm        194 kB/s |  39 kB     00:00
(2/5): vim-filesystem-8.0.1763-15.el8.noarch.rp 310 kB/s |  48 kB     00:00
(3/5): which-2.21-12.el8.x86\_64.rpm              99 kB/s |  49 kB     00:00
(4/5): vim-enhanced-8.0.1763-15.el8.x86\_64.rpm  1.0 MB/s | 1.4 MB     00:01
(5/5): vim-common-8.0.1763-15.el8.x86\_64.rpm    1.9 MB/s | 6.3 MB     00:03

--------------------------------------------------------------------------------

Total                                           1.9 MB/s | 7.8 MB     00:04
warning: /var/cache/dnf/appstream-02e86d1c976ab532/packages/gpm-libs-1.20.7-15.el8.x86\_64.rpm: Header V3 RSA/SHA256 Signature, key ID 8483c65d: NOKEY
CentOS Linux 8 - AppStream                      1.6 MB/s | 1.6 kB     00:00
Importing GPG key 0x8483C65D:
Userid     : "CentOS (CentOS Official Signing Key) <security@centos.org>"
Fingerprint: 99DB 70FA E1D7 CE22 7FB6 4882 05B5 55B3 8483 C65D
From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
 Preparing        :                                                        1/1
 Installing       : which-2.21-12.el8.x86\_64                               1/5
 Installing       : vim-filesystem-2:8.0.1763-15.el8.noarch                2/5
 Installing       : vim-common-2:8.0.1763-15.el8.x86\_64                    3/5
 Installing       : gpm-libs-1.20.7-15.el8.x86\_64                          4/5
 Running scriptlet: gpm-libs-1.20.7-15.el8.x86\_64                          4/5
 Installing       : vim-enhanced-2:8.0.1763-15.el8.x86\_64                  5/5
 Running scriptlet: vim-enhanced-2:8.0.1763-15.el8.x86\_64                  5/5
 Running scriptlet: vim-common-2:8.0.1763-15.el8.x86\_64                    5/5
 Verifying        : gpm-libs-1.20.7-15.el8.x86\_64                          1/5
 Verifying        : vim-common-2:8.0.1763-15.el8.x86\_64                    2/5
 Verifying        : vim-enhanced-2:8.0.1763-15.el8.x86\_64                  3/5
 Verifying        : vim-filesystem-2:8.0.1763-15.el8.noarch                4/5
 Verifying        : which-2.21-12.el8.x86\_64                               5/5

Installed:
 gpm-libs-1.20.7-15.el8.x86\_64         vim-common-2:8.0.1763-15.el8.x86\_64
 vim-enhanced-2:8.0.1763-15.el8.x86\_64 vim-filesystem-2:8.0.1763-15.el8.noarch
 which-2.21-12.el8.x86\_64
Complete!

Removing intermediate container 716a53b879c7
---> 3e56881f32f3
Step 6/10 : RUN yum -y install net-tools
---> Running in 08e74effed03
Last metadata expiration check: 0:00:12 ago on Fri May 14 17:37:27 2021.
Dependencies resolved.
================================================================================
Package         Architecture Version                        Repository    Size
================================================================================
Installing:
net-tools       x86\_64       2.0-0.52.20160912git.el8       baseos       322 k
Transaction Summary
================================================================================
Install  1 Package
Total download size: 322 k
Installed size: 942 k
Downloading Packages:
net-tools-2.0-0.52.20160912git.el8.x86\_64.rpm   613 kB/s | 322 kB     00:00
--------------------------------------------------------------------------------
Total                                           338 kB/s | 322 kB     00:00

Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
 Preparing        :                                                        1/1
 Installing       : net-tools-2.0-0.52.20160912git.el8.x86\_64              1/1
 Running scriptlet: net-tools-2.0-0.52.20160912git.el8.x86\_64              1/1
 Verifying        : net-tools-2.0-0.52.20160912git.el8.x86\_64              1/1
Installed:
 net-tools-2.0-0.52.20160912git.el8.x86\_64
Complete!
Removing intermediate container 08e74effed03
---> 60ee298a7584
Step 7/10 : EXPOSE 80
---> Running in 5aa1259c277c
Removing intermediate container 5aa1259c277c
---> fdc9eb2e97df
Step 8/10 : CMD echo $MYPATH
---> Running in 83afdd200304
Removing intermediate container 83afdd200304
---> 510a6a5e710f
Step 9/10 : CMD echo "-----------------build end-----------------"
---> Running in d9362f4c5cde
Removing intermediate container d9362f4c5cde
---> 6251a37b03da
Step 10/10 : CMD /bin/bash
---> Running in 69c8acd6cce6
Removing intermediate container 69c8acd6cce6
---> b6df0e7c5686
Successfully built b6df0e7c5686
Successfully tagged mycentos:0.1

#test
[root@localhost dockerfile]# docker run -it mycentos:0.1
[root@3d6c237f79da local]# pwd
/usr/local
[root@3d6c237f79da local]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
       inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
       ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
       RX packets 8  bytes 656 (656.0 B)
       RX errors 0  dropped 0  overruns 0  frame 0
       TX packets 0  bytes 0 (0.0 B)
       TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
       inet 127.0.0.1  netmask 255.0.0.0
       loop  txqueuelen 1000  (Local Loopback)
       RX packets 0  bytes 0 (0.0 B)
       RX errors 0  dropped 0  overruns 0  frame 0
       TX packets 0  bytes 0 (0.0 B)
       TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

#镜像的构建步骤

[root@localhost dockerfile]# docker history b6df0e7c5686
IMAGE          CREATED         CREATED BY                                      S                                                                             IZE      COMMENT
b6df0e7c5686   7 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0                                                                             B
6251a37b03da   7 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0                                                                             B
510a6a5e710f   7 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0                                                                             B
fdc9eb2e97df   7 minutes ago   /bin/sh -c #(nop)  EXPOSE 80                    0                                                                             B
60ee298a7584   7 minutes ago   /bin/sh -c yum -y install net-tools             2                                                                             3.4MB
3e56881f32f3   8 minutes ago   /bin/sh -c yum -y install vim                   5                                                                             8.1MB
a99d170ce4a1   8 minutes ago   /bin/sh -c #(nop) WORKDIR /usr/local            0                                                                             B
2dfd922108a3   8 minutes ago   /bin/sh -c #(nop)  ENV MYPATH=/usr/local        0                                                                             B
06cb0004f73f   8 minutes ago   /bin/sh -c #(nop)  MAINTAINER qj<qiaojun2002…   0                                                                             B
300e315adb2f   5 months ago    /bin/sh -c #(nop)  CMD ["/bin/bash"]            0                                                                             B
<missing>      5 months ago    /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0                                                                             B
<missing>      5 months ago    /bin/sh -c #(nop) ADD file:bd7a2aed6ede423b7…   2    
```



拿到一个镜像，可以通过docker history它是怎么做的







> CMD与ENTRYPOINT对比



```BASH
#CMD实例
[root@localhost dockerfile]# cat docker-cmd-test
FROM centos
CMD ["ls","-a"]
[root@localhost dockerfile]# docker run 8146ffc523e2
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
[root@localhost dockerfile]# docker run 8146ffc523e2 -l

docker: Error response from daemon: OCI runtime create failed: container\_linux.go:367: starting container process caused: exec: "-l": executable file not found in $PATH: unknown.

ERRO[0000] error waiting for container: context canceled

```







```BASH
#ENTRYPOINT实例
[root@localhost dockerfile]# cat docker-entrypoint-test
FROM centos
ENTRYPOINT ["ls","-a"]
[root@localhost dockerfile]# docker run 73bc5d2e1d00
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
[root@localhost dockerfile]# docker run 73bc5d2e1d00 -l
total 0
drwxr-xr-x.   1 root root   6 May 14 17:59 .
drwxr-xr-x.   1 root root   6 May 14 17:59 ..
-rwxr-xr-x.   1 root root   0 May 14 17:59 .dockerenv
lrwxrwxrwx.   1 root root   7 Nov  3  2020 bin -> usr/bin
drwxr-xr-x.   5 root root 340 May 14 17:59 dev
drwxr-xr-x.   1 root root  66 May 14 17:59 etc
drwxr-xr-x.   2 root root   6 Nov  3  2020 home
lrwxrwxrwx.   1 root root   7 Nov  3  2020 lib -> usr/lib
lrwxrwxrwx.   1 root root   9 Nov  3  2020 lib64 -> usr/lib64
drwx------.   2 root root   6 Dec  4 17:37 lost+found
drwxr-xr-x.   2 root root   6 Nov  3  2020 media
drwxr-xr-x.   2 root root   6 Nov  3  2020 mnt
drwxr-xr-x.   2 root root   6 Nov  3  2020 opt
dr-xr-xr-x. 149 root root   0 May 14 17:59 proc
dr-xr-x---.   2 root root 162 Dec  4 17:37 root
drwxr-xr-x.  11 root root 163 Dec  4 17:37 run
lrwxrwxrwx.   1 root root   8 Nov  3  2020 sbin -> usr/sbin
drwxr-xr-x.   2 root root   6 Nov  3  2020 srv
dr-xr-xr-x.  13 root root   0 May 14 17:07 sys
drwxrwxrwt.   7 root root 145 Dec  4 17:37 tmp
drwxr-xr-x.  12 root root 144 Dec  4 17:37 usr
drwxr-xr-x.  20 root root 262 Dec  4 17:37 var
```







## docker网络



### 自定义网络



```BASH
[root@localhost ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
0b73fb45e41b   bridge    bridge    local
a3a375569fc0   host      host      local
a8822654faf0   none      null      local

```







**网络模式**



bridge： 桥接（默认）



none: 不配置网络



host： 和宿主机共享网络



container：容器内网络连通



**测试**



```BASH
docker run -d -P --name tomcat01
docker run -d -P --net bridge --name tomcat01
#docker0 特点： 默认，域名不能访问  --link可以打通

#自定义网络
# --driver bridge
# --subnet 192.168.0.0
# --gateway 192.168.0.1
[root@localhost ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
7ea68fbf783af33e0f2cfa5203f17a8322f2e70d143e5d670fa57f933fae3ec0

[root@localhost ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
0b73fb45e41b   bridge    bridge    local
a3a375569fc0   host      host      local
7ea68fbf783a   mynet     bridge    local
a8822654faf0   none      null      local

#查看自定义网络

[root@localhost ~]# docker network inspect 7ea68fbf783a
[
   {

       "Name": "mynet",
       "Id": "7ea68fbf783af33e0f2cfa5203f17a8322f2e70d143e5d670fa57f933fae3ec0",
       "Created": "2021-05-15T09:22:58.365432902-04:00",
       "Scope": "local",
       "Driver": "bridge",
       "EnableIPv6": false,
       "IPAM": {
           "Driver": "default",
           "Options": {},
           "Config": [
               {
                   "Subnet": "192.168.0.0/16",
                   "Gateway": "192.168.0.1"
               }
           ]
       },

       "Internal": false,
       "Attachable": false,
       "Ingress": false,
       "ConfigFrom": {
           "Network": ""
       },
       "ConfigOnly": false,
       "Containers": {},
       "Options": {},
       "Labels": {}

   }

]
#自定义网络添加容器
[root@localhost ~]# docker run -d -it -P --name centos-net-03 --net mynet centos
[root@localhost ~]# docker run -d -it -P --name centos-net-04 --net mynet centos
#测试ping命令，现在不使用--link，也可以ping名字了
[root@localhost ~]# docker exec -it centos-net-04 ping centos-net-03
PING centos-net-03 (192.168.0.2) 56(84) bytes of data.
64 bytes from centos-net-03.mynet (192.168.0.2): icmp\_seq=1 ttl=64 time=0.104 ms
64 bytes from centos-net-03.mynet (192.168.0.2): icmp\_seq=2 ttl=64 time=0.250 ms

```



好处：



自定义网络维护好了容器间对应的关系



### 网络连通


```bash
[root@localhost ~]# docker network connect --help
Usage:  docker network connect [OPTIONS] NETWORK CONTAINER
Connect a container to a network

#默认docker0添加容器
[root@localhost ~]# docker run -d -it -P --name centos-01 centos               
[root@localhost ~]# docker run -d -it -P --name centos-02 centos

#自定义网络添加容器
[root@localhost ~]# docker run -d -it -P --name centos-net-03 --net mynet centos
[root@localhost ~]# docker run -d -it -P --name centos-net-04 --net mynet centos

#docker0容器无法访问mynet网络下容器
[root@localhost ~]# docker exec -it centos-01 ping centos-net-03
ping: centos-net-03: Name or service not known

#测试打通tomcat01 - mynet
[root@localhost ~]# docker network connect mynet centos-01

#再次测试，docker0容器可以访问mynet网络下容器
[root@localhost ~]# docker exec -it centos-01 ping centos-net-03
PING centos-net-03 (192.168.0.2) 56(84) bytes of data.
64 bytes from centos-net-03.mynet (192.168.0.2): icmp\_seq=1 ttl=64 time=0.133 ms
64 bytes from centos-net-03.mynet (192.168.0.2): icmp\_seq=2 ttl=64 time=0.139 ms
64 bytes from centos-net-03.mynet (192.168.0.2): icmp\_seq=3 ttl=64 time=0.129 ms
```







## docker compose



轻松高效管理容器，定义运行多个容器



> 官方介绍



Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration， To learn more about all the features of Compose, see [the list of features](https://docs.docker.com/compose/#features).



Compose works in all environments: production, staging, development, testing, as well as CI workflows. You can learn more about each case in [Common Use Cases](https://docs.docker.com/compose/#common-use-cases).



Using Compose is basically a three-step process:



1\. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.

2\. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.

3\. Run `docker compose up` and the [Docker compose command](https://docs.docker.com/compose/cli-command/) starts and runs your entire app. You can alternatively run `docker-compose up` using the docker-compose binary.



Compose：重要概念

- 服务service： 容器，应用（redis，tomcat，etc）

- 项目project，一组容器



### 安装



```
curl -L https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```







### 快速开始



> 参考



https://docs.docker.com/compose/gettingstarted/









