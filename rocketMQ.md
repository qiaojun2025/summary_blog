## MQ简介
[学习视频](https://www.bilibili.com/video/BV1jSKAzME6E?spm_id_from=333.788.videopod.episodes&vd_source=46bda19e04eb9188c7f6a377526437bd&p=3)<br>
MQ: Message Queue， 消息队列。
MQ的作用主要包括以下三个方面：
- 异步
- 解耦
- 削峰

## RocketMQ 简介
Alibaba 开源的一个消息中间件

## RocketMQ 特点
目前市场比较流行的MQ，影响力较大的有Kafka， RabbitMQ，RocketMQ，适用不同的业务场景。

|  | 优点 | 缺点 | 使用场景 |
| --- | --   | --  | --       |
| Kafka  | 吞吐量非常大，性能非常好，集群高可用 | 数据可能丢失，功能单一 | 日志分析，大数据采集 |
| RabbitMQ  | 消息可靠性高，功能全面 | erlang语言不好定制，吞吐量低 | 企业内部小规模服务调用 |
| RocketMQ  | 高吞吐，高性能，高可用，功能全面，使用java，方便定制  | 服务加载比较慢 | 几乎全场景，特别适合金融 |


## 安装
依赖JAVA 环境
[下载](https://rocketmq.apache.org/download/)

```
java -version

vi bin/runserver.sh
#修改choose_gc_options方法，设置-Xmx1g -Xms1g -Xmn 512m
 
vi bin/runbroker.sh
 设置-Xmx1g -Xms1g
 
 nohup sh bin/mqnamesrv.sh &
  
  #check nohub.log 
  #The name server boot success
  
  vi ~/.bash_profile
  
  export NAMESRV_ADDR='localhost:9876'
  
  source ~/.bash_profile
  
  nohub bin/mqbroker.sh &
  
  #check nohub.log
  #The broker  boot success
  
  jps
  9122 NamesrvStartup
  9123 BrokerStartup
  
  #send message
  bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
  
  #consume message
  bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
  
```



## 搭建Dashboard
[下载](https://rocketmq.apache.org/download/)，拉到下载页面最下方，下载RocketMQ Dashboard

需要我们自己编译，使用maven构建
```
mvn clean package -Dmaven.test.skip=true
```

在源码的target目录下生成了可运行的jar包rocketmq-dashboard-1.0.1-SNAPSHOT.jar
接下来我们在jar包所在的目录下创建一个application.yml配置文件
```yaml
rocketmq:
  config:
    namesrvAddrs:
	  - 127.0.0.1:9876
```
主要时指定nameserver的地址
接下来通过java指令运行这个jar

```shell
java -jar rocketmq-dashboard-1.0.1-SNAPSHOT.jar
```

## 升级分布式集群

