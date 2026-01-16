## 环境准备
### ES安装 
下载压缩包解压缩，并执行bin目录下的elasticsearch.bat<br>
[下载](https://www.elastic.co/cn/downloads/past-releases)

安装完成后，可以使用下列CAT命令验证ES

**CAT API** 

```
/_cat/allocation  #查看单节点的shard分配整体情况
/_cat/shards #查看各shard的详细情况
/_cat/shards/{index} #查看指定分片的详细情况
/_cat/master #查看master节点信息
/ cat/nodes #查看所有节点信息
/_cat/indices #查看集群中所有index的详细信息
/_cat/indices/{index} #查看集群中指定index的详细信息
/_cat/count #查看当前集群的doc数量
/ cat/count/{index} #查看指定索引的doc数量
/_cat/recovery #查看集群内每个shard的recovery过程.调整replica
/_cat/recovery/{index}#査看指定索引shard的recovery过程
/_cat/health #查看集群当前状态:红、黄、绿
/_cat/pending tasks #查看当前集群的pending task
/_cat/aliases/{alias} #查看指定索引的alias信息
/_cat/thread poo1 #集群各节点内部不同类型的threadpoo1的统计信息，
/_cat/plugins #查看集群各个节点上的plugin信息
/ cat/fielddata #查看当前集群各个节点的fielddata内存使用情况
```

运行本地ES后，执行[示例](http://localhost:9200/_cat/health?v)

### 客户端Kibana
下载压缩包解压缩，并执行bin目录下的kibana.bat<br>
[下载](https://www.elastic.co/downloads/past-releases/kibana-7-17-4)
> **需要保证kibana 版本与ES版本一致**


<br>
<br>


## 核心概念

### 索引 Index
一个索引就是一个拥有几分相似特征的文档的集合。比如说，你可以有一个商品数据的索引，一个订单数据的索引，还有一个用户数据的索引。一个索引由一个名字来标识(必须全部是小写字母的)，并且当我们要对这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字。


**创建**
```

PUT /user

GET /user

PUT /student index
{
	"settings":{
		"number of shards": 1,
		"number of replicas": 1
	},
	"mappings":{
		"properties"{
			"name":{
				"type" :"text"
			},
			"age":{
				type":"integer
			},
			"enrolled date":{
				"type": "date"
			}
		}
	}
}


GET /student index
```

**查询**
```
GET /employee/_doc/1

GET /employee/_mget{
	"ids": ["1","2","3"]
}

```

更多示例参考官网教程， [官网](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/index.html)

### 映射 Mapping
映射是定义一个文档和它所包含的字段如何被存储和索引的过程。在默认配置下，ES可以根据插入的数据 自动地创建mapping，也可以手动创建mapping。mapping中主要包括字段名字段类型等

### 文档
文档是索引中存储的一条条数据。轻量级的JSON格式数据来表示。

**创建**
```

PUT /employee/_doc/1
{
	"name":"张三”,
	"sex": 1,
	"age": 25,
	"address":"广州天河公园",
	"remark":"java developer"
}

POST /employee/_doc
{
	"name":"李四”,
	"sex": 1,
	"age": 25,
	"address":"广州天河公园",
	"remark":"java developer"
}


```
