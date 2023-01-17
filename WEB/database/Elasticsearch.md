# Elasticsearch
[官网](https://www.elastic.co/cn/)

一个开源的高扩展的分布式全文搜索引擎



The Elastic Stack, 包括 Elasticsearch、Kibana、Beats 和 Logstash（也称为 ELK Stack）。能够安全可靠地获取任何来源、任何格式的数据，然后实时地对数据进行搜索、分析和可视化。Elaticsearch是整个Elastic Stack技术栈的核心。它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理PB级别的数据。



官网文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html

官网中文：https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html

社区中文：http://doc.codingdict.com/elasticsearch/



腾讯云使用Kafka或Logstash作为日志采集，存储到es集群，使用可视化界面快速定位日志异常

ELK快速定位日志
	1）使用LogStash作为日志收集的汇聚节点
	2）使用ES集群作为存储
	3）使用Kibana作为可视化



## Elasticsearch And Solr

都是基于Lucene搭建的，可以独立部署启动的搜索引擎服务软件

![image-20211031124211229](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211031124211229.png)



* 与Solr相比，Elasticsearch易于安装且非常轻巧。此外，你可以在几分钟内安装并运行Elasticsearch。但是，如果Elasticsearch管理不当，这种易于部署和使用可能会成为一个问题。基于JSON的配置很简单，但如果要为文件中的每个配置指定注释，那么它不适合您。总的来说，如果你的应用使用的是JSON，那么Elasticsearch是一个更好的选择。否则，请使用Solr，因为它的schema.xml和solrconfig.xml都有很好的文档记录。

*  Solr拥有更大，更成熟的用户，开发者和贡献者社区。ES虽拥有的规模较小但活跃的用户社区以及不断增长的贡献者社区。

* Solr贡献者和提交者来自许多不同的组织，而Elasticsearch提交者来自单个公司。

*  Solr更成熟，但ES增长迅速，更稳定。

* Solr是一个非常有据可查的产品，具有清晰的示例和API用例场景。 Elasticsearch的文档组织良好，但它缺乏好的示例和清晰的配置说明。

那么，到底是Solr还是Elasticsearch

*  由于易于使用，Elasticsearch在新开发者中更受欢迎。一个下载和一个命令就可以启动一切。

* 如果除了搜索文本之外还需要它来处理分析查询，Elasticsearch是更好的选择

* 如果需要分布式索引，则需要选择Elasticsearch。对于需要良好可伸缩性和以及性能分布式环境，Elasticsearch是更好的选择。

*  Elasticsearch在开源日志管理用例中占据主导地位，许多组织在Elasticsearch中索引它们的日志以使其可搜索。

* 如果你喜欢监控和指标，那么请使用Elasticsearch，因为相对于Solr，Elasticsearch暴露了更多的关键指标



## 安装

https://www.elastic.co/cn/downloads/past-releases#elasticsearch



windows版 elasticsearch.bat启动es服务

9300端口为Elasticsearch集群间组件的通信端口，9200端口为浏览器访问的http协议RESTful端口

http://localhost:9200测试结果

如果系统配置JAVA_HOME，es使用系统默认JDK，否则使用自带的JDK，一般建议使用系统配置

空间不足修改config/jvm.options 堆内存



docker

```shell
mkdir -p /mydata/elasticsearch/config # 用来存放配置文件
mkdir -p /mydata/elasticsearch/data  # 数据
echo "http.host: 0.0.0.0" >/mydata/elasticsearch/config/elasticsearch.yml # 允许任何机器访问
chmod -R 777 /mydata/elasticsearch/ ## 设置elasticsearch文件可读写权限

docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e  "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v  /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.4.2 


docker update elasticsearch --restart=always #开机自启

#Kibana 
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://192.168.56.10:9200 -p 5601:5601 -d kibana:7.4.2

#http://192.168.56.10:9200 改成自己Elasticsearch上的地址
```



## 入门

Elasticsearch是面向文档型数据库，一条数据在这里就是一个文档

![image-20211031130342892](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211031130342892.png)

Elasticsearch 7.X中, Type的概念已经被删除



==倒排索引 (将关键字作为索引)==



### http操作

#### _cat

```
GET /_cat/nodes：查看所有节点
GET /_cat/health：查看 es 健康状况
GET /_cat/master：查看主节点
GET /_cat/incices：查看所有索引 show databases;
```



#### 索引操作

```shell
#创建索引
#向ES服务器发PUT请求 http://127.0.0.1:9200/shopping
{
    "acknowledged"【响应结果】: true, # true操作成功
    "shards_acknowledged"【分片结果】: true, # 分片操作成功
    "index"【索引名称】: "shopping"
}
#注意：创建索引库的分片数默认1片，在7.0.0之前的Elasticsearch版本中，默认5片

#查看索引
#向ES服务器发GET请求 ：http://127.0.0.1:9200/_cat/indices?v
#_cat表示查看的意思，indices表示索引  查看当前ES服务器中的所有索引
#返回结果如下
```

| 表头           | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
| health         | 当前服务器健康状态：  **green**(集群完整) **yellow**(单点正常、集群不完整)  red(单点不正常) |
| status         | 索引打开、关闭状态                                           |
| index          | 索引名                                                       |
| uuid           | 索引统一编号                                                 |
| pri            | 主分片数量                                                   |
| rep            | 副本数量                                                     |
| docs.count     | 可用文档数量                                                 |
| docs.deleted   | 文档删除状态（逻辑删除）                                     |
| store.size     | 主分片和副分片整体占空间大小                                 |
| pri.store.size | 主分片占空间大小                                             |

```shell
#查看单个索引
#向ES服务器发GET请求 ：http://127.0.0.1:9200/shopping
{
    "shopping"【索引名】: {        
        "aliases"【别名】: {},
        "mappings"【映射】: {},
        "settings"【设置】: {
            "index"【设置 - 索引】: {
                "creation_date"【设置 - 索引 - 创建时间】: "1614265373911",
                "number_of_shards"【设置 - 索引 - 主分片数量】: "1",
                "number_of_replicas"【设置 - 索引 - 副分片数量】: "1",
                "uuid"【设置 - 索引 - 唯一标识】: "eI5wemRERTumxGCc1bAk2A",
                "version"【设置 - 索引 - 版本】: {
                    "created": "7080099"
                },
                "provided_name"【设置 - 索引 - 名称】: "shopping"
            }
        }
    }
}

#删除索引
#向ES服务器发DELETE请求 ：http://127.0.0.1:9200/shopping
```

#### 文档操作

##### 创建文档

```shell
向ES服务器发POST请求 ：http://127.0.0.1:9200/shopping/_doc
{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":3999.00
}

#返回
{
    "_index"【索引】: "shopping",
    "_type"【类型-文档】: "_doc",
    "_id"【唯一标识】: "Xhsa2ncBlvF_7lxyCE9G", #可以类比为MySQL中的主键，随机生成
    "_version"【版本】: 1,
    "result"【结果】: "created", #这里的create表示创建成功
    "_shards"【分片】: {
        "total"【分片 - 总数】: 2,
        "successful"【分片 - 成功】: 1,
        "failed"【分片 - 失败】: 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
#上面的数据创建后，由于没有指定数据唯一性标识（ID），默认情况下，ES服务器会随机生成一个。
#如果想要自定义唯一性标识，需要在创建时指定：http://127.0.0.1:9200/shopping/_doc/1
#如果增加数据时明确数据主键，那么请求方式也可以为PUT
```

##### 查看文档

```shell
#GET请求 ：http://127.0.0.1:9200/shopping/_doc/1
#返回
{
    "_index"【索引】: "shopping",
    "_type"【文档类型】: "_doc",
    "_id": "1",
    "_version": 2,
    "_seq_no": 2,#并发控制字段，每次更新就会+1，用来做乐观锁
    "_primary_term": 2,  #同上，主分片重新分配，如重启，就会变化
    "found"【查询结果】: true, # true表示查找到，false表示未查找到
    "_source"【文档源信息】: {
        "title": "华为手机",
        "category": "华为",
        "images": "http://www.gulixueyuan.com/hw.jpg",
        "price": 4999.00
    }
}

#GET请求 ：http://127.0.0.1:9200/shopping/_search  查询全部
#更新携带：?if_seq_no=4&if_primary_term=1  乐观锁
```

##### 修改文档

```shell
#POST请求 ：http://127.0.0.1:9200/shopping/_doc/1  会覆盖
{
    "title":"华为手机",
    "category":"华为",
    "images":"http://www.gulixueyuan.com/hw.jpg",
    "price":4999.00
}

#返回
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version"【版本】: 2,
    "result"【结果】: "updated", # updated表示数据被更新
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 2,
    "_primary_term": 2
}
-------------------------------------
POST customer/external/1/_update
{
	"doc":{
		"name":"John Doew"
	}
}
或者
POST customer/external/1
{
	"name":"John Doe2"
}
或者
PUT customer/external/1
{
	"name":"jack"
}

#不同：Post  _update操作会对比源文档数据，数据如果一样就不进行任何操作
#	Post/PUT操作总会将数据重新保存并增加 version 版本

#对于大并发更新，不带update
#对于大并发查询并偶尔更新，带update 对比更新，重新计算分配规则
```

##### 修改字段

```shell
#POST请求 ：http://127.0.0.1:9200/shopping/_update/1
{ 
  "doc": {
    "price":3000.00
  } 
}
```

##### 删除文档

```shell
#DELETE请求 ：http://127.0.0.1:9200/shopping/_doc/1 逻辑删除

#返回
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version"【版本】: 4, #对数据的操作，都会更新版本
    "result"【结果】: "deleted", # deleted表示数据被标记为删除
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 4,
    "_primary_term": 2
}
```

##### 条件删除文档

```shell
#POST请求 ：http://127.0.0.1:9200/shopping/_delete_by_query
{
  "query":{
    "match":{
      "price":4000.00
    }
  }
}
#返回
{
    "took"【耗时】: 175,
    "timed_out"【是否超时】: false,
    "total"【总数】: 2,
    "deleted"【删除数量】: 2,
    "batches": 1,
    "version_conflicts": 0,
    "noops": 0,
    "retries": {
        "bulk": 0,
        "search": 0
    },
    "throttled_millis": 0,
    "requests_per_second": -1.0,
    "throttled_until_millis": 0,
    "failures": []
}
```

#### 批量bulk 

```shell
#语法
{action:{metadata}}\n
{requeestBody}\n
{action:{metadata}}\n
{requesetbod }\n

POST customer/external/_bulk
{"index":{"_id":"1"}}
{"name":"John Doe"}
{"index":{"_id":"2"}}
{"name":"John Doe"}

#复杂实例
POST /_bulk
{"delete":{"_index":"website","_type":"blog","_id":"123"}}
{"create":{"_index":"website","_type":"blog","_id":"123"}}
{"title":"my first blog post"}
{"index":{"_index":"website","_type":"blog"}}
{"title":"my second blog post"}
{"update":{"_index":"website","_type":"blog","_id":"123"}}
{"doc":{"title":"my updated blog post"}}
```

bulk 按顺序执行所有的action (动作)。如果一一个单个的动作因任何原因而失败，它将继续处理它后面剩余的动作。当bulkAPI 返回时，它将提供每个动作的状态(与发送的顺序相同)



样本测试数据

https://github.com/elastic/elasticsearch/edit/master/docs/src/test/resources/accounts.json

#### [映射操作](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/mapping-types.html)

映射，类似于数据库(database)中的表结构(table)。创建数据库表需要设置字段名称，类型，长度，约束等；索引库也一样，需要知道这个类型下有哪些字段，每个字段有哪些约束信息，这就叫做映射(mapping)



ES 7.x 

URL 中的 type 参数 可选，比如索引一个文档不再要求提供文档类型

ES 8.X

不在支持 URL 中的 type 参数

```shell
#创建映射 / 添加新映射
#PUT请求 ：http://127.0.0.1:9200/student/_mapping
{
  "properties": {
    "name":{
      "type": "text",
      "index": true
    },
    "sex":{
      "type": "text",
      "index": false
    },
    "age":{
      "type": "long",
      "index": false
    }
  }
}
#映射数据说明
字段名：任意填写，下面指定许多属性，例如：title、subtitle、images、price
type：类型，Elasticsearch中支持的数据类型非常丰富，说几个关键的：
	String类型，又分两种：
		text：可分词
		keyword：不可分词，数据会作为完整字段进行匹配
	Numerical：数值类型，分两类
		基本数据类型：long、integer、short、byte、double、float、half_float
		浮点数的高精度类型：scaled_float
	Date：日期类型
	Array：数组类型
	Object：对象
index：是否索引，默认为true，也就是说你不进行任何配置，所有字段都会被索引。
	true：字段会被索引，则可以用来进行搜索
	false：字段不会被索引，不能用来搜索
store：是否将数据进行独立存储，默认为false
	原始的文本会存储在_source里面，默认情况下其他提取出来的字段都不是独立存储的，是从_source里面提取出来的。当然	 你也可以独立的存储某个字段，只要设置"store": true即可，获取独立存储的字段要比从_source中解析快得多，但是也	会占用更多的空间，所以要根据实际业务需求来设置。
analyzer：分词器，这里的ik_max_word即使用ik分词器,后面会有专门的章节学习


#查看映射
#GET请求 ：http://127.0.0.1:9200/student/_mapping

#索引映射关联
#PUT请求 ：http://127.0.0.1:9200/student1
{
  "settings": {},
  "mappings": {
	  "properties": {
		"name":{
		  "type": "text",
		  "index": true
		  
		},
		"sex":{
		  "type": "text",
		  "index": false
		},
		"age":{
		  "type": "long",
		  "index": false
		}
	  }
  }
}

#对于已经存在的映射字段，我们不能更新，更新必须创建新的索引进行数据迁移


#数据迁移
#先创 new_twitter 的正确映射，然乎使用如下方式进行数据迁移
POST _reindex [固定写法]
{
  "source":{
    "index":"twitter"
  },
  "dest":{
    "index":"new_twitter"
  }
}
## 将旧索引的 type 下的数据进行迁移
POST _reindex
{
  "source": {
    "index":"twitter",
    "type":"tweet"
  },
  "dest":{
    "index":"twweets"
  }
}
```

参考官网：https://www.elastic.co/guide/en/elasticsearch/reference/7.10/mapping-types.html

参数映射规则：https://www.elastic.co/guide/en/elasticsearch/reference/7.10/mapping-params.html#mapping-params

#### 高级查询

ES 支持两种基本方式检索:

- 一个是通过使用 REST request URL,发送搜索参数，(uri + 检索参数)
- 另一个是通过使用 REST request bod 来发送他们，(uri + 请求体)



DSL （domain-specifig langurage 领域特定语言）， Query DSL 





```json
GET /bank/_search
{
    "query": { #query 定义如何查询
    	"match_all": {}  # match_all 查询类型【代表查询所有的所有】
    	"match": {
    		"address": "mill lane"  #字符串分词匹配   非字符串精确匹配   查询结果评分排序
    		"address.keyword": "mill lane" #keyword精确匹配
		} ,
		"match_phrase": { "address": "mill lane" }#不分词匹配
		"multi_match": {
      		"query": "mill",
      		"fields": ["address","city"]  #两个字段任意一个包含 多字段匹配 会分词
    	}
	},  
	"sort": [
    	{ "account_number": "asc" }
	],
	"from": 10,
	"size": 10
	"_source": ["firstname","lastname"] #返回部分字段
}
```





##### 查询所有文档

```shell
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "match_all": {}
  }
}
# "query"：这里的query代表一个查询对象，里面可以有不同的查询属性
# "match_all"：查询类型，例如：match_all(代表查询所有)， match，term ， range 等等
# {查询条件}：查询条件会根据类型的不同，写法也有差异

#返回
{
  "took【查询花费时间，单位毫秒】" : 1116,
  "timed_out【是否超时】" : false,
  "_shards【分片信息】" : {
    "total【总数】" : 1,
    "successful【成功】" : 1,
    "skipped【忽略】" : 0,
    "failed【失败】" : 0
  },
  "hits【搜索命中结果】" : {
     "total"【搜索条件匹配的文档总数】: {
         "value"【总命中计数的值】: 3,
         "relation"【计数规则】: "eq" # eq 表示计数准确， gte表示计数不准确
     },
    "max_score【匹配度分值】" : 1.0,
    "hits【命中结果集合】" : [
       。。。
      }
    ]
  }
}
```

##### 匹配查询

```shell
#GET请求 ：http://127.0.0.1:9200/student/_search   
#match匹配类型查询，会把查询条件进行分词，然后进行查询，多个词条之间是or的关系
{
  "query": {
    "match": {
        "name":"zhangsan"
    }
  }
}

```

##### 字段匹配查询

```shell
#GET请求 ：http://127.0.0.1:9200/student/_search
#multi_match与match类似，不同的是它可以在多个字段中查询
{
  "query": {
    "multi_match": {
        "query": "zhangsan",
        "fields": ["name","nickname"]
    }
  }
}
```

##### term

和 match 一样，匹配某个属性的值，全文检索字段用 match，其他非text字段匹配用 term

```shell
#term查询，精确的关键词匹配查询，不对查询条件进行分词
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "term": {
      "name": {
        "value": "zhangsan"
      }
    }
  }
}
```

##### 多关键字精确查询

```shell
#terms 查询和 term 查询一样，但它允许你指定多值进行匹配
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "terms": {
      "name": ["zhangsan","lisi"]
    }
  }
}
```

##### 指定查询字段

```shell
#GET请求 ：http://127.0.0.1:9200/student/_search
#_source过滤字段
{
  "_source": ["name","nickname"],  
  "query": {
    "terms": {
      "nickname": ["zhangsan"]
    }
  }
}
```

#####  过滤字段

```shell
#includes：来指定想要显示的字段
#excludes：来指定不想要显示的字段
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "_source": {
    "includes": ["name","nickname"]
    "excludes": ["name","nickname"]
  },  
  "query": {
    "terms": {
      "nickname": ["zhangsan"]
    }
  }
}
```

##### 组合查询-bool

复合语句可以合并 任何 其他嵌套语句，包括复合语句





should: 达到会增加相关文档的评分，并不会改变查询的结果，如果 query 中只有 should 且只有一种匹配规则，那么 should的条件就会被作为默认匹配条件而区别查询结果

使用`minimum_should_match`，至少匹配一项`should`子句

```shell
#`bool`把各种其它查询通过`must`（必须 ）、`must_not`（必须不）、`should`（应该）的方式进行组合
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "zhangsan"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "age": "40"
          }
        }
      ],
      "should": [
        {
          "match": {
            "sex": "男"
          }
        }
      ]
      "filter": {  #不会产生相关得分
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```

#####  范围查询

```shell
#range 查询找出那些落在指定区间内的数字或者时间  gt> lt< gte>= lte<=
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 30,
        "lte": 35
      }
    }
  }
}
```

##### 模糊查询

```shell
#编辑距离是将一个术语转换为另一个术语所需的一个字符更改的次数。这些更改可以包括：
#更改字符（box → fox）
#删除字符（black → lack）
#插入字符（sic → sick）
#转置两个相邻字符（act → cat）
#为了找到相似的术语，fuzzy查询会在指定的编辑距离内创建一组搜索词的所有可能的变体或扩展。然后查询返回每个扩展的完全匹配。
#通过fuzziness修改编辑距离。一般使用默认值AUTO，根据术语的长度生成编辑距离。

#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "fuzzy": {
      "title": {
        "value": "zhangsan"
      }
    }
  }
}
#或
{
  "query": {
    "fuzzy": {
      "title": {
        "value": "zhangsan",
		"fuzziness": 2
      }
    }
  }
}
```

##### 单字段排序

```shell
#sort 可以让我们按照不同的字段进行排序，并且通过order指定排序的方式。desc降序，asc升序
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "match": {
        "name":"zhangsan"
    }
  },
  "sort": [{
    "age": {
        "order":"desc"
    }
  }]
}

```

##### 多字段排序

```shell
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    },
    {
      "_score":{
        "order": "desc"
      }
    }
  ]
}

```



##### 高亮查询

```shell
#使用match查询的同时，加上一个highlight属性
#pre_tags：前置标签
#post_tags：后置标签
#fields：需要高亮的字段
#title：这里声明title字段需要高亮，后面可以为这个字段设置特有配置，也可以空

#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "match": {
      "name": "zhangsan"
    }
  },
  "highlight": {
    "pre_tags": "<font color='red'>",
    "post_tags": "</font>",
    "fields": {
      "name": {}
    }
  }
}
```

##### 分页查询

```shell
#from：当前页的起始索引，默认从0开始。 from = (pageNum - 1) * size
#size：每页显示多少条 
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ],
  "from": 0,
  "size": 2
}
```

##### 聚合查询 -aggregations

聚合允许使用者对es文档进行统计分析，类似与关系型数据库中的group by，当然还有很多其他的聚合，例如取最大值、平均值等等

聚合提供了从数据分组和提取数据的能力，最简单的聚合方法大致等于 SQL GROUP BY 和 SQL 聚合函数，在 ES 中，你有执行搜索返回 hits （命中结果） 并且同时返回聚合结果，把一个响应中的所有 hits（命中结果）分隔开的能力，这是非常强大有效的，你可以执行查询和多个聚合，并且在一个使用中得到各自的（任何一个的）返回结果，使用一次简洁简化的 API 来避免网络往返



```shell
#GET请求 ：http://127.0.0.1:9200/student/_search 
#对某个字段取最大值max
{
    "aggs":{
      "max_age":{
        "max":{"field":"age"}
        "min":{"field":"age"}
        "sum":{"field":"age"}
      }
    },
    "size":0
}
#min
{
    "aggs":{
      "min_age":{
        "min":{"field":"age"}
      }
    },
    "size":0
}
#sum
{
    "aggs":{
      "sum_age":{
        "sum":{"field":"age"}
      }
    },
    "size":0
}
#avg
{
    "aggs":{
      "avg_age":{
        "avg":{"field":"age"}
      }
    },
    "size":0
}
#去重
{
    "aggs":{
      "distinct_age":{
        "cardinality":{"field":"age"}
      }
    },
    "size":0
}
#State聚合  一次性返回count，max，min，avg和sum五个指标
{
    "aggs":{
      "stats_age":{
        "stats":{"field":"age"}
      }
    },
    "size":0
}
```

**搜索address中包含mill的所有人的年龄分布以及平均年龄**

```http
GET bank/_search
{
  "query": {
    "match": {
      "address": "mill"
    }
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 10
      }
    },
    "ageAvg":{
      "avg": {
        "field": "age"
      }
    },
    "balanceAvg":{
      "avg": {
        "field": "balance"
      }
    }
    
  },
  "size":0
}
```

**按照年龄聚合，并且请求这些年龄段的这些人的平均薪资**

```java
GET bank/_search
{
  "query":{
    "match_all": {}
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 10
      },
      "aggs": {
        "ageAvg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

**查出所有年龄分布，并且这些年龄段中M的平均薪资和 F 的平均薪资以及这个年龄段的总体平均薪资**

```java
GET /bank/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 100
      },
      "aggs": {
        "genderAgg": {
          "terms": {
            "field": "gender.keyword",
            "size": 10
          },
          "aggs": {
            "balanceAvg": {
              "avg": {
                "field": "balance"
                }
            }
          }
        }
      }
    }
  }
}
```

##### 桶聚合查询

```shell
#桶聚和相当于sql中的group by语句
#terms聚合，分组统计

#GET请求 ：http://127.0.0.1:9200/student/_search
{
    "aggs":{
      "age_groupby":{
        "terms":{"field":"age"}
      }
    },
    "size":0
} 

#terms分组下再进行聚合
{
    "aggs":{
      "age_groupby":{
        "terms":{"field":"age"}
        "aggs":{
          "sum_age":{
            "sum":{"filed":"age"}
          }
        }
      }
    },
    "size":0
}
```

### java api操作

```xml
<dependencies>
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>7.8.0</version>
    </dependency>
    <!-- elasticsearch的客户端 -->
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>7.8.0</version>
    </dependency>
    <!-- elasticsearch依赖2.x的log4j -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.8.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.8.2</version>
</dependency>
	<dependency>
		<groupId>com.fasterxml.jackson.core</groupId>
		<artifactId>jackson-databind</artifactId>
		<version>2.9.9</version>
	</dependency>
    <!-- junit单元测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>
```

#### 客户端对象

```
// 创建客户端对象
RestHighLevelClient client = new RestHighLevelClient(
		RestClient.builder(new HttpHost("localhost", 9200, "http"))
);

...

// 关闭客户端连接
client.close();
```

#### 索引操作

##### 创建索引

```java
// 创建索引 - 请求对象
CreateIndexRequest request = new CreateIndexRequest("user");
// 发送请求，获取响应
CreateIndexResponse response = client.indices().create(request, RequestOptions.DEFAULT);
boolean acknowledged = response.isAcknowledged();
// 响应状态
System.out.println("操作状态 = " + acknowledged);
```

##### 查看索引

```java
// 查询索引 - 请求对象
GetIndexRequest request = new GetIndexRequest("user");
// 发送请求，获取响应
GetIndexResponse response = client.indices().get(request, RequestOptions.DEFAULT);
System.out.println("aliases:"+response.getAliases());
System.out.println("mappings:"+response.getMappings());
System.out.println("settings:"+response.getSettings());
```

##### 删除索引

```java
// 删除索引 - 请求对象
DeleteIndexRequest request = new DeleteIndexRequest("user");
// 发送请求，获取响应
AcknowledgedResponse response = client.indices().delete(request, RequestOptions.DEFAULT);
// 操作结果
System.out.println("操作结果 ： " + response.isAcknowledged());
```

#### 文档操作

##### 增

```java
@Data
class User {                         
    private String name;             
    private Integer age;             
    private String sex; 
}

// 新增文档 - 请求对象
IndexRequest request = new IndexRequest();
// 设置索引及唯一性标识
request.index("user").id("1001");
// 创建数据对象
User user = new User();
user.setName("zhangsan");
user.setAge(30);
user.setSex("男");
ObjectMapper objectMapper = new ObjectMapper();
String productJson = objectMapper.writeValueAsString(user);
// 添加文档数据，数据格式为JSON格式
request.source(productJson,XContentType.JSON);
// 客户端发送请求，获取响应对象
IndexResponse response = client.index(request, RequestOptions.DEFAULT);
////3.打印结果信息
System.out.println("_index:" + response.getIndex());
System.out.println("_id:" + response.getId());
System.out.println("_result:" + response.getResult());  

```

##### 删

```java
//创建请求对象
DeleteRequest request = new DeleteRequest().index("user").id("1");
//客户端发送请求，获取响应对象
DeleteResponse response = client.delete(request, RequestOptions.DEFAULT);
//打印信息
System.out.println(response.toString());
```

##### 改

```java
// 修改文档 - 请求对象
UpdateRequest request = new UpdateRequest();
// 配置修改参数
request.index("user").id("1001");
// 设置请求体，对数据进行修改
request.doc(XContentType.JSON, "sex", "女");
// 客户端发送请求，获取响应对象
UpdateResponse response = client.update(request, RequestOptions.DEFAULT);
System.out.println("_index:" + response.getIndex());
System.out.println("_id:" + response.getId());
System.out.println("_result:" + response.getResult());
```

##### 查

```java
//1.创建请求对象
GetRequest request = new GetRequest().index("user").id("1001");
//2.客户端发送请求，获取响应对象
GetResponse response = client.get(request, RequestOptions.DEFAULT);
////3.打印结果信息
System.out.println("_index:" + response.getIndex());
System.out.println("_type:" + response.getType());
System.out.println("_id:" + response.getId());
System.out.println("source:" + response.getSourceAsString());
```

##### 批量

批量新增

```java
//创建批量新增请求对象
BulkRequest request = new BulkRequest();
request.add(new IndexRequest().index("user").id("1001").source(XContentType.JSON, "name", "zhangsan"));
request.add(new IndexRequest().index("user").id("1002").source(XContentType.JSON, "name", "lisi"));
request.add(new IndexRequest().index("user").id("1003").source(XContentType.JSON, "name", "wangwu"));
//客户端发送请求，获取响应对象
BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
//打印结果信息
System.out.println("took:" + responses.getTook());
System.out.println("items:" + responses.getItems());
```

批量删除

```java
//创建批量删除请求对象
BulkRequest request = new BulkRequest();
request.add(new DeleteRequest().index("user").id("1001"));
request.add(new DeleteRequest().index("user").id("1002"));
request.add(new DeleteRequest().index("user").id("1003"));
//客户端发送请求，获取响应对象
BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
//打印结果信息
System.out.println("took:" + responses.getTook());
System.out.println("items:" + responses.getItems());
```

#### 高级查询

##### 查询所有索引数据

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
// 查询所有数据
sourceBuilder.query(QueryBuilders.matchAllQuery());
request.source(sourceBuilder);

SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### term查询，查询条件为关键字

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.termQuery("age", "30"));
request.source(sourceBuilder);

SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 分页

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.matchAllQuery());

// 分页查询
// 当前页其实索引(第一条数据的顺序号)，from
sourceBuilder.from(0);
// 每页显示多少条size
sourceBuilder.size(2);

request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");

```

##### 排序

```java
// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.matchAllQuery());

// 排序
sourceBuilder.sort("age", SortOrder.ASC);

request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");

```

##### 过滤字段

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.matchAllQuery());

//查询字段过滤
String[] excludes = {};
String[] includes = {"name", "age"};
sourceBuilder.fetchSource(includes, excludes);

request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 组合查询

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
// 必须包含
boolQueryBuilder.must(QueryBuilders.matchQuery("age", "30"));
// 一定不含
boolQueryBuilder.mustNot(QueryBuilders.matchQuery("name", "zhangsan"));
// 可能包含
boolQueryBuilder.should(QueryBuilders.matchQuery("sex", "男"));

sourceBuilder.query(boolQueryBuilder);
request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 范围查询

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("age");
// 大于等于
rangeQuery.gte("30");
// 小于等于
rangeQuery.lte("40");

sourceBuilder.query(rangeQuery);
request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");

```

##### 模糊查询

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

sourceBuilder.query(QueryBuilders.fuzzyQuery("name","zhangsan").fuzziness(Fuzziness.ONE));
request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 高亮查询

```java
// 高亮查询
SearchRequest request = new SearchRequest().indices("student");
//2.创建查询请求体构建器
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
//构建查询方式：高亮查询
TermsQueryBuilder termsQueryBuilder = QueryBuilders.termsQuery("name","zhangsan");
//设置查询方式
sourceBuilder.query(termsQueryBuilder);
//构建高亮字段
HighlightBuilder highlightBuilder = new HighlightBuilder();
highlightBuilder.preTags("<font color='red'>");//设置标签前缀
highlightBuilder.postTags("</font>");//设置标签后缀
highlightBuilder.field("name");//设置高亮字段
//设置高亮构建对象
sourceBuilder.highlighter(highlightBuilder);
//设置请求体
request.source(sourceBuilder);
//3.客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);

//4.打印响应结果
SearchHits hits = response.getHits();
System.out.println("took::"+response.getTook());
System.out.println("time_out::"+response.isTimedOut());
System.out.println("total::"+hits.getTotalHits());
System.out.println("max_score::"+hits.getMaxScore());
System.out.println("hits::::>>");
for (SearchHit hit : hits) {
	String sourceAsString = hit.getSourceAsString();
	System.out.println(sourceAsString);
	//打印高亮结果
	Map<String, HighlightField> highlightFields = hit.getHighlightFields();
	System.out.println(highlightFields);
}
System.out.println("<<::::");

```

##### 聚合查询

```java
SearchRequest request = new SearchRequest().indices("student");
//max age
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.aggregation(AggregationBuilders.max("maxAge").field("age"));
//设置请求体
request.source(sourceBuilder);
//3.客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);

//4.打印响应结果
SearchHits hits = response.getHits();
System.out.println(response);

```

```java
SearchRequest request = new SearchRequest().indices("student");
//分组
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.aggregation(AggregationBuilders.terms("age_groupby").field("age"));

//设置请求体
request.source(sourceBuilder);
//3.客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);

//4.打印响应结果
SearchHits hits = response.getHits();
System.out.println(response);

```

### kibana操作

#### 索引

```shell
#创建索引
# PUT 索引名称(小写)
PUT test_ index

# PUT索引
# 增加配置↓JSON格式的主体内容
PUT test_ index 1
"aliases": {
	"test1": {}
}

# HEAD索引HTTP状态码: 200， 404
head test_ index1

#查询索引
# GET索引名称
GET test_ index
#查询所有索引
GET _cat/indices

#删除索引
# delete 索引名称
DELETE test_ index_ 1
 #kibana 不许修改索引
```

#### 文档

```shell
# 创建文档(索引数据)。增加唯一性标识(手动， 自动)
PUT test_ doc
PUT test_ doc/_ doc/1001
{
	"id" : 1001,
	"name" : "zhangsan",
	"age" : 30
} 
POST test. _doc/_ doc
{
"id" : 1002,
"name" : "lisi",
"age" : 40
}

#查询文档
GET test_doc/_doc/1001
#查询索引中所有的文档数据
GET test doc/_ search
#修改数据
PUT/POST test_doc/_doc/1001
{
"id" : 10011,
"name" : "zhangsan1"，
"age" : 300,
"tel": 123121 }

#删除数据
DELETE test_ doc/_ doc/1002|

```





## 环境

### 集群

大于等于2个节点就可以看做集群。一般出于高性能及高可用方面来考虑集群中节点数量都是3个以上

一个Elasticsearch集群有一个唯一的名字标识，这个名字默认就是”elasticsearch”。这个名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入这个集群。

#### Windows集群

1. 创建elasticsearch-cluster文件夹，在内部复制三个elasticsearch服务

2. 修改集群文件目录中每个节点的 config/elasticsearch.yml配置文件

   node1

   ```yaml
   #节点1的配置信息：
   #集群名称，节点之间要保持一致
   cluster.name: my-elasticsearch
   #节点名称，集群内要唯一
   node.name: node-1001
   node.master: true
   node.data: true
    
   #ip地址
   network.host: localhost
   #http端口
   http.port: 1001
   #tcp监听端口
   transport.tcp.port: 9301
   
   #discovery.seed_hosts: ["localhost:9301", "localhost:9302","localhost:9303"]
   #discovery.zen.fd.ping_timeout: 1m
   #discovery.zen.fd.ping_retries: 5
    
   #集群内的可以被选为主节点的节点列表
   #cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
    
   #跨域配置
   #action.destructive_requires_name: true
   http.cors.enabled: true
   http.cors.allow-origin: "*"
   
   ```

   node2

   ```yaml
   #节点2的配置信息：
   #集群名称，节点之间要保持一致
   cluster.name: my-elasticsearch
   #节点名称，集群内要唯一
   node.name: node-1002
   node.master: true
   node.data: true
    
   #ip地址
   network.host: localhost
   #http端口
   http.port: 1002
   #tcp监听端口
   transport.tcp.port: 9302
   
   discovery.seed_hosts: ["localhost:9301"]
   discovery.zen.fd.ping_timeout: 1m
   discovery.zen.fd.ping_retries: 5
    
   #集群内的可以被选为主节点的节点列表
   #cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
    
   #跨域配置
   #action.destructive_requires_name: true
   http.cors.enabled: true
   http.cors.allow-origin: "*"
   
   ```

   node3

   ```yaml
   #节点3的配置信息：
   #集群名称，节点之间要保持一致
   cluster.name: my-elasticsearch
   #节点名称，集群内要唯一
   node.name: node-1003
   node.master: true
   node.data: true
    
   #ip地址
   network.host: localhost
   #http端口
   http.port: 1003
   #tcp监听端口
   transport.tcp.port: 9303
   #候选主节点的地址，在开启服务后可以被选为主节点 
   discovery.seed_hosts: ["localhost:9301", "localhost:9302"]
   discovery.zen.fd.ping_timeout: 1m
   discovery.zen.fd.ping_retries: 5
    
   #集群内的可以被选为主节点的节点列表
   #cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
    
   #跨域配置
   #action.destructive_requires_name: true
   http.cors.enabled: true
   http.cors.allow-origin: "*"
   
   ```

3. 启动前先删除每个节点中的data目录中所有内容（如果存在）

   双击执行 bin/elasticsearch.bat    启动后，会自动加入指定名称的集群

4. 测试  

   GET 请求 ：http://127.0.0.1:1001/_cluster/health

   GET 请求 ：http://127.0.0.1:1002/_cluster/health

   GET 请求 ：http://127.0.0.1:1003/_cluster/health

   status  

   ​	green 主分片副分片都正常

   ​	yellow 主分片正常 副分片有不正常的

   ​	red 主分片不正常

#### linux 集群

```shell
# 解压缩
tar -zxvf elasticsearch-7.8.0-linux-x86_64.tar.gz -C /opt/module
# 改名
mv elasticsearch-7.8.0 es-cluster

#因为安全问题，Elasticsearch不允许root用户直接运行，所以要在每个节点中创建新用户，在root用户中创建新用户 
useradd es #新增es用户
passwd es #为es用户设置密码

userdel -r es #如果错了，可以删除再加
chown -R es:es /opt/module/es-cluster #文件夹所有者

```

修改/opt/module/es/config/elasticsearch.yml文件，分发文件

```yaml
# 加入如下配置
#集群名称
cluster.name: cluster-es
#节点名称，每个节点的名称不能重复
node.name: node-1
#ip地址，每个节点的地址不能重复
network.host: linux1
#是不是有资格主节点
node.master: true
node.data: true
http.port: 9200
# head 插件需要这打开这两个配置
http.cors.allow-origin: "*"
http.cors.enabled: true
http.max_content_length: 200mb
#es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举master
cluster.initial_master_nodes: ["node-1"]
#es7.x 之后新增的配置，节点发现
discovery.seed_hosts: ["linux1:9300","linux2:9300","linux3:9300"]
gateway.recover_after_nodes: 2
network.tcp.keep_alive: true
network.tcp.no_delay: true
transport.tcp.compress: true
#集群内同时启动的数据任务个数，默认是2个
cluster.routing.allocation.cluster_concurrent_rebalance: 16
#添加或删除节点及负载均衡时并发恢复的线程个数，默认4个
cluster.routing.allocation.node_concurrent_recoveries: 16
#初始化数据恢复时，并发恢复线程的个数，默认4个
cluster.routing.allocation.node_initial_primaries_recoveries: 16
```

修改/etc/security/limits.conf ，分发文件

```yaml
# 在文件末尾中增加下面内容
es soft nofile 65536
es hard nofile 65536
```

修改/etc/security/limits.d/20-nproc.conf，分发文件

```yaml
\# 在文件末尾中增加下面内容
es soft nofile 65536
es hard nofile 65536
\* hard nproc 4096
\# 注：* 带表Linux所有用户名称
```

修改/etc/sysctl.conf

```yaml
\# 在文件中增加下面内容
vm.max_map_count=655360
```

重新加载

```shell
sysctl -p 
```

启动

```shell
cd /opt/module/es-cluster
#启动
bin/elasticsearch
#后台启动
bin/elasticsearch -d
```

### linux 单机

https://www.elastic.co/cn/downloads/past-releases/elasticsearch-7-8-0

```shell
# 解压缩
tar -zxvf elasticsearch-7.8.0-linux-x86_64.tar.gz -C /opt/module
# 改名
mv elasticsearch-7.8.0 es

#因为安全问题，Elasticsearch不允许root用户直接运行，所以要创建新用户，在root用户中创建新用户 
useradd es #新增es用户
passwd es #为es用户设置密码

userdel -r es #如果错了，可以删除再加
chown -R es:es /opt/module/es #文件夹所有者

#修改/opt/module/es/config/elasticsearch.yml文件
# 加入如下配置
cluster.name: elasticsearch
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["node-1"]

#修改/etc/security/limits.conf 
# 在文件末尾中增加下面内容
# 每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536

#修改/etc/security/limits.d/20-nproc.conf
# 在文件末尾中增加下面内容
# 每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536
# 操作系统级别对每个用户创建的进程数的限制
* hard nproc 4096
# 注：* 带表Linux所有用户名称

#修改/etc/sysctl.conf
# 在文件中增加下面内容
# 一个进程可以拥有的VMA(虚拟内存区域)的数量,默认值为65536
vm.max_map_count=655360

#重新加载
sysctl -p

cd /opt/module/es/
#启动
bin/elasticsearch
#后台启动
bin/elasticsearch -d

#暂时关闭防火墙
systemctl stop firewalld

#永久关闭防火墙
systemctl enable firewalld.service #打开防火墙永久性生效，重启后不会复原
systemctl disable firewalld.service #关闭防火墙，永久性生效，重启后不会复原
```

### kibana

https://artifacts.elastic.co/downloads/kibana/kibana-7.8.0-windows-x86_64.zip

修改config/kibana.yml文件

```yaml
# 默认端口
server.port: 5601
# ES服务器的地址
elasticsearch.hosts: ["http://localhost:9200"]
# 索引名
kibana.index: ".kibana"
# 支持中文
i18n.locale: "zh-CN"
```

Windows环境下执行bin/kibana.bat文件

## 进阶

### 核心概念

* 索引 (名字小写字母)

  **Elasticsearch索引的精髓：一切设计都是为了提高搜索的性能**

* 类型

  一个索引中，你可以定义一种或多种类型

  7.X 默认不再支持自定义索引类型（默认类型为：_doc）

* 文档

  一个文档是一个可被索引的基础信息单元，也就是一条数据

* 字段（Field）

  相当于是数据表的字段，对文档数据根据不同属性进行的分类标识

* 映射

  mapping是处理数据的方式和规则方面做一些限制，如：某个字段的数据类型、默认值、分析器、是否被索引等等。这些都是映射里面可以设置的，其它就是处理ES里面数据的一些使用规则设置也叫做映射，按着最优规则处理数据对性能提高很大，因此才需要建立映射，并且需要思考如何建立映射才能对性能更好

* 分片（Shards）

  一个索引可以存储超出单个节点硬件限制的大量数据。比如，一个具有10亿文档数据的索引占据1TB的磁盘空间，而任一节点都可能没有这样大的磁盘空间。或者单个节点处理搜索请求，响应太慢。为了解决这个问题，Elasticsearch提供了将索引划分成多份的能力，每一份就称之为分片。当你创建一个索引的时候，你可以指定你想要的分片的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上。

  分片很重要，主要有两方面的原因：

  1. 允许你水平分割 / 扩展你的内容容量。

  2. 允许你在分片之上进行分布式的、并行的操作，进而提高性能/吞吐量。

  至于一个分片怎样分布，它的文档怎样聚合和搜索请求，是完全由Elasticsearch管理的，对于作为用户的你来说，这些都是透明的，无需过分关心。

  **被混淆的概念是，一个 Lucene 索引 我们在 Elasticsearch 称作 分片 。 一个 Elasticsearch 索引 是分片的集合。 当 Elasticsearch 在索引中搜索的时候， 他发送查询到每一个属于索引的分片(Lucene 索引)，然后合并每个分片的结果到一个全局的结果集**

* 副本（Replicas）

  Elasticsearch允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片(副本)。

  * 在分片/节点失败的情况下，提供了高可用性。因为这个原因，注意到复制分片从不与原/主要（original/primary）分片置于同一节点上是非常重要的。

  * 扩展你的搜索量/吞吐量，因为搜索可以在所有的副本上并行运行。

  总之，每个索引可以被分成多个分片。一个索引也可以被复制0次（意思是没有复制）或多次。一旦复制了，每个索引就有了主分片（作为复制源的原来的分片）和复制分片（主分片的拷贝）之别。分片和复制的数量可以在索引创建的时候指定。在索引创建之后，你可以在任何时候动态地改变复制的数量，但是你事后不能改变分片的数量。默认情况下，Elasticsearch中的每个索引被分片1个主分片和1个复制，这意味着，如果你的集群中至少有两个节点，你的索引将会有1个主分片和另外1个复制分片（1个完全拷贝），这样的话每个索引总共就有2个分片，我们需要根据索引需要确定分片个数。

* 分配（Allocation）

  将分片分配给某个节点的过程，包括分配主分片或者副本。如果是副本，还包含从主分片复制数据的过程。这个过程是由master节点完成的

### 架构

一个运行中的 Elasticsearch 实例称为一个节点，而集群是由一个或者多个拥有相同 cluster.name 配置的节点组成， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

当一个节点被选举成为主节点时， 它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。 而主节点并不需要涉及到文档级别的变更和搜索等操作，所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。 任何节点都可以成为主节点。我们的示例集群就只有一个节点，所以它同时也成为了主节点。

作为用户，我们可以将请求发送到集群中的任何节点 ，包括主节点。 每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。 无论我们将请求发送到哪个节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回給客户端。 Elasticsearch 对这一切的管理都是透明的。



### 分布式集群

#### 单节点集群

分配3个主分片和一份副本（每个主分片拥有一个副本分片）

```json
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
```

通过elasticsearch-head插件查看集群情况

集群健康值:yellow( 3 of 6 ) : 表示当前集群的全部主分片都正常运行，但是副本分片没有全部处在正常状态

3个主分片正常

3个副本分片都是 Unassigned —— 它们都没有被分配到任何节点。 在同一个节点上既保存原始数据又保存副本是没有意义的，因为一旦失去了那个节点，我们也将丢失该节点上的所有副本数据。

当前我们的集群是正常运行的，但是在硬件故障时有丢失数据的风险

#### 故障转移

增加一个节点  主分片和副本分片都被分配

3个副本分片被分配到新节点

#### 水平扩容

启动第三个节点，我们的集群将会拥有三个节点的集群 : 为了分散负载而对分片进行重新分配

扩容超过6个节点时

在运行中的集群上是可以动态调整副本分片数目的，按需伸缩集群。把副本数从默认的 1 增加到 2

####  应对故障

选举一个新的主节点

新的主节点立即将这些分片在 Node 2 和 Node 3 上对应的副本分片提升为主分片

此时集群的状态将会为 yellow



重新启动 Node 1 ，集群可以将缺失的副本分片再次进行分配，那么集群的状态也将恢复成之前的状态。 如果 Node 1 依然拥有着之前的分片，它将尝试去重用它们，同时仅从主分片复制发生了修改的数据文件。和之前的集群相比，只是Master节点切换了

### 路由计算

创建文档时，应当被存储在哪个分片

![image-20211101165540472](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211101165540472.png)

routing 是一个可变值，默认是文档的 _id ，也可以设置成一个自定义的值。 routing 通过 hash 函数生成一个数字，然后这个数字再除以 number_of_primary_shards （主分片的数量）后得到余数 。这个分布在 0 到 number_of_primary_shards-1 之间的余数，就是我们所寻求的文档所在分片的位置



这就解释了为什么我们要在创建索引的时候就确定好主分片的数量 并且永远不会改变这个数量：因为如果数量变化了，那么所有之前路由的值都会无效，文档也再也找不到了。

所有的文档 API（ get 、 index 、 delete 、 bulk 、 update 以及 mget ）都接受一个叫做 routing 的路由参数 ，通过这个参数我们可以自定义文档到分片的映射。一个自定义的路由参数可以用来确保所有相关的文档——例如所有属于同一个用户的文档——都被存储到同一个分片中。

### 分片控制

我们可以发送请求到集群中的任一节点。 每个节点都有能力处理任意请求。 每个节点都知道集群中任一文档位置，所以可以直接将请求转发到需要的节点上。 在下面的例子中，将所有的请求发送到 Node 1，我们将其称为 **协调节点**(coordinating node) 。

**当发送请求的时候，** **为了扩展负载，更好的做法是轮询集群中所有的节点。**

#### 写流程

新建、索引和删除 请求都是 写 操作， 必须在主分片上面完成之后才能被复制到相关的副本分片



* 客户端向 Node 1 发送请求
* 通过id找到主分片
* 主分片执行, 副本同步 , 完成返回客户端



有一些可选的请求参数允许您影响这个过程，可能以数据安全为代价提升性能。这些选项很少使用，因为Elasticsearch已经很快，但是为了完整起见，请参考下面表格：

| 参数        | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| consistency | consistency，即一致性。在默认设置下，即使仅仅是在试图执行一个_写_操作之前，主分片都会要求 必须要有 规定数量(quorum)（或者换种说法，也即必须要有大多数）的分片副本处于活跃可用状态，才会去执行_写_操作(其中分片副本可以是主分片或者副本分片)。这是为了避免在发生网络分区故障（network partition）的时候进行_写_操作，进而导致数据不一致。_规定数量_即：  **int( (primary  + number_of_replicas) / 2 ) + 1**  consistency 参数的值可以设为 one （只要主分片状态 ok 就允许执行_写_操作）,all（必须要主分片和所有副本分片的状态没问题才允许执行_写_操作）, 或 quorum 。默认值为  quorum , 即大多数的分片副本状态没问题就允许执行_写_操作。  注意，规定数量 的计算公式中 number_of_replicas 指的是在索引设置中的设定副本分片数，而不是指当前处理活动状态的副本分片数。如果你的索引设置中指定了当前索引拥有三个副本分片，那规定数量的计算结果即：  **int( (primary  + 3 replicas) / 2 ) + 1 = 3**  如果此时你只启动两个节点，那么处于活跃状态的分片副本数量就达不到规定数量，也因此您将无法索引和删除任何文档。 |
| timeout     | 如果没有足够的副本分片会发生什么？ Elasticsearch会等待，希望更多的分片出现。默认情况下，它最多等待1分钟。 如果你需要，你可以使用 timeout 参数 使它更早终止： 100 100毫秒，30s 是30秒。 |

新索引默认有 1 个副本分片，这意味着为满足规定数量应该需要两个活动的分片副本。 但是，这些默认的设置会阻止我们在单一节点上做任何事情。为了避免这个问题，要求只有当 number_of_replicas 大于1的时候，规定数量才会执行

#### 读流程

在处理读取请求时，协调结点在每次请求的时候都会通过轮询所有的副本分片来达到负载均衡。在文档被检索时，已经被索引的文档可能已经存在于主分片上但是还没有复制到副本分片。 在这种情况下，副本分片可能会报告文档不存在，但是主分片可能成功返回文档。 一旦索引请求成功返回给用户，文档在主分片和副本分片都是可用的



#### 更新流程



* 从主分片检索文档，修改 _source 字段中的 JSON ，并且尝试重新索引主分片的文档。 如果文档已经被另一个进程修改，它会重试本步骤，超过 retry_on_conflict 次后放弃。
* 更新成功,同步副本,返回

当主分片把更改转发到副本分片时， 它不会转发更新请求。 相反，它转发完整文档的新版本。请记住，这些更改将会异步转发到副本分片，并且不能保证它们以发送它们相同的顺序到达。 如果Elasticsearch仅转发更改请求，则可能以错误的顺序应用更改，导致得到损坏的文档。

#### 多文档操作流程

mget 和 bulk API 的模式类似于单文档模式。区别在于协调节点知道每个文档存在于哪个分片中。它将整个多文档请求分解成 每个分片 的多文档请求，并且将这些请求并行转发到每个参与节点。

协调节点一旦收到来自每个节点的应答，就将每个节点的响应收集整理成单个响应，返回给客户端

**用单个 mget** **请求取回多个文档所需的步骤顺序**:

* 客户端向 Node 1 发送 mget 请求。

* Node 1 为每个分片构建多文档获取请求，然后并行转发这些请求到托管在每个所需的主分片或者副本分片的节点上。一旦收到所有答复， Node 1 构建响应并将其返回给客户端。

可以对 docs 数组中每个文档设置 routing 参数



**bulk API**， 允许在单个批量请求中执行多个创建、索引、删除和更新请求

bulk API 按如下步骤顺序执行：

*  客户端向 Node 1 发送 bulk 请求。

* Node 1 为每个节点创建一个批量请求，并将这些请求并行转发到每个包含主分片的节点主机。

* 主分片一个接一个按顺序执行每个操作。当每个操作成功时，主分片并行转发新文档（或删除）到副本分片，然后执行下一个操作。 一旦所有的副本分片报告所有操作成功，该节点将向协调节点报告成功，协调节点将这些响应收集整理并返回给客户端。

### 分片原理

#### 倒排索引

正向索引，就是搜索引擎会将待搜索的文件都对应一个文件ID，搜索时将这个ID和搜索关键字进行对应，形成K-V对，然后对关键字进行统计计数

倒排索引, 把文件ID对应到关键词的映射转换为关键词到文件ID的映射，每个关键词都对应着一系列的文件，这些文件中都出现这个关键词。



分词和标准化的过程称为**分析**

这非常重要。你只能搜索在索引中出现的词条，所以索引文本和查询字符串必须标准化为相同的格式。

#### 文档搜索

早期的全文检索会为整个文档集合建立一个很大的倒排索引并将其写入到磁盘。 一旦新的索引就绪，旧的就会被其替换，这样最近的变化便可以被检索到。

倒排索引被写入磁盘后是 不可改变 的:它永远不会修改。

不变性有重要的价值：

* 不需要锁。如果你从来不更新索引，你就不需要担心多进程同时修改数据的问题。

* 一旦索引被读入内核的文件系统缓存，便会留在哪里，由于其不变性。只要文件系统缓存中还有足够的空间，那么大部分读请求会直接请求内存，而不会命中磁盘。这提供了很大的性能提升。

* 其它缓存(像filter缓存)，在索引的生命周期内始终有效。它们不需要在每次数据改变时被重建，因为数据不会变化。

* 写入单个大的倒排索引允许数据被压缩，减少磁盘 I/O 和 需要被缓存到内存的索引的使用量。

当然，一个不变的索引也有不好的地方。主要事实是它是不可变的! 你不能修改它。如果你需要让一个新的文档 可被搜索，你需要重建整个索引。这要么对一个索引所能包含的数据量造成了很大的限制，要么对索引可被更新的频率造成了很大的限制。



#### 动态更新索引

保留不变性的前提下实现倒排索引的更新

答案是: 用更多的索引。通过增加新的补充索引来反映新近的修改，而不是直接重写整个倒排索引。每一个倒排索引都会被轮流查询到，从最早的开始查询完后再对结果进行合并。

Elasticsearch 基于 Lucene, 这个java库引入了**按段搜索**的概念。 每一 段 本身都是一个倒排索引， 但索引在 Lucene 中除表示所有段的集合外， 还增加了提交点的概念 — 一个列出了所有已知段的文件



这种方式可以用相对较低的成本将新文档添加到索引。

段是不可改变的，所以既不能从把文档从旧的段中移除，也不能修改旧的段来进行反映文档的更新。 取而代之的是，每个提交点会包含一个 .del 文件，文件中会列出这些被删除文档的段信息。

当一个文档被 “删除” 时，它实际上只是在 .del 文件中被 标记 删除。一个被标记删除的文档仍然可以被查询匹配到， 但它会在最终结果被返回前从结果集中移除。

文档更新也是类似的操作方式：当一个文档被更新时，旧版本文档被标记删除，文档的新版本被索引到一个新的段中。 可能两个版本的文档都会被一个查询匹配到，但被删除的那个旧版本文档在结果集返回前就已经被移除。



#### 近实时搜索

随着按段（per-segment）搜索的发展，一个新的文档从索引到可被搜索的延迟显著降低了。新文档在几分钟之内即可被检索，但这样还是不够快。磁盘在这里成为了瓶颈。提交（Commiting）一个新的段到磁盘需要一个 fsync 来确保段被物理性地写入磁盘，这样在断电的时候就不会丢失数据。 但是 fsync 操作代价很大; 如果每次索引一个文档都去执行一次的话会造成很大的性能问题。

我们需要的是一个更轻量的方式来使一个文档可被搜索，这意味着 fsync 要从整个过程中被移除。在Elasticsearch和磁盘之间是文件系统缓存。 像之前描述的一样， 在内存索引缓冲区中的文档会被写入到一个新的段中。 但是这里新段会被先写入到文件系统缓存—这一步代价会比较低，稍后再被刷新到磁盘—这一步代价比较高。不过只要文件已经在缓存中， 就可以像其它文件一样被打开和读取了。



在 Elasticsearch 中，写入和打开一个新段的轻量的过程叫做 refresh 。 默认情况下每个分片会每秒自动刷新一次。这就是为什么我们说 Elasticsearch 是 近 实时搜索: 文档的变化并不是立即对搜索可见，但会在一秒之内变为可见。

这些行为可能会对新用户造成困惑: 他们索引了一个文档然后尝试搜索它，但却没有搜到。这个问题的解决办法是用 refresh API 执行一次手动刷新: /users/_refresh

尽管刷新是比提交轻量很多的操作，它还是会有性能开销。当写测试的时候， 手动刷新很有用，但是不要在生产环境下每次索引一个文档都去手动刷新。 相反，你的应用需要意识到 Elasticsearch 的近实时的性质，并接受它的不足。

并不是所有的情况都需要每秒刷新。可能你正在使用 Elasticsearch 索引大量的日志文件， 你可能想优化索引速度而不是近实时搜索， 可以通过设置 refresh_interval ， 降低每个索引的刷新频率

```
{
  "settings": {
    "refresh_interval": "30s" 
  }
}
```

refresh_interval 可以在既存索引上进行动态更新。 在生产环境中，当你正在建立一个大的新索引时，可以先关闭自动刷新，待开始使用该索引时，再把它们调回来

```
# 关闭自动刷新
PUT /users/_settings
{ "refresh_interval": -1 } 

# 每一秒刷新
PUT /users/_settings
{ "refresh_interval": "1s" } 
```

#### 持久化变更

如果没有用 fsync 把数据从文件系统缓存刷（flush）到硬盘，我们不能保证数据在断电甚至是程序正常退出之后依然存在。为了保证 Elasticsearch 的可靠性，需要确保数据变化被持久化到磁盘。在 动态更新索引，我们说一次完整的提交会将段刷到磁盘，并写入一个包含所有段列表的提交点。Elasticsearch 在启动或重新打开一个索引的过程中使用这个提交点来判断哪些段隶属于当前分片。

即使通过每秒刷新（refresh）实现了近实时搜索，我们仍然需要经常进行完整提交来确保能从失败中恢复。但在两次提交之间发生变化的文档怎么办？我们也不希望丢失掉这些数据。Elasticsearch 增加了一个 translog ，或者叫事务日志，在每一次对 Elasticsearch 进行操作时均进行了日志记录



整个流程如下：

1. 一个文档被索引之后，就会被添加到内存缓冲区，并且追加到了 translog

2. 刷新（refresh）使分片每秒被刷新（refresh）一次：

   * 这些在内存缓冲区的文档被写入到一个新的段中，且没有进行 fsync 操作。

   * 这个段被打开，使其可被搜索

   * 内存缓冲区被清空

3.  这个进程继续工作，更多的文档被添加到内存缓冲区和追加到事务日志

4. 每隔一段时间—例如 translog 变得越来越大—索引被刷新（flush）；一个新的 translog 被创建，并且一个全量提交被执行

   * 所有在内存缓冲区的文档都被写入一个新的段。

   * 缓冲区被清空。

   * 一个提交点被写入硬盘。

   * 文件系统缓存通过 fsync 被刷新（flush）。

   * 老的 translog 被删除



translog 提供所有还没有被刷到磁盘的操作的一个持久化纪录。当 Elasticsearch 启动的时候， 它会从磁盘中使用最后一个提交点去恢复已知的段，并且会重放 translog 中所有在最后一次提交后发生的变更操作。

translog 也被用来提供实时 CRUD 。当你试着通过ID查询、更新、删除一个文档，它会在尝试从相应的段中检索之前， 首先检查 translog 任何最近的变更。这意味着它总是能够实时地获取到文档的最新版本

执行一个提交并且截断 translog 的行为在 Elasticsearch 被称作一次 flush

分片每30分钟被自动刷新（flush），或者在 translog 太大的时候也会刷新

你很少需要自己手动执行 flush 操作；通常情况下，自动刷新就足够了。这就是说，在重启节点或关闭索引之前执行 flush 有益于你的索引。当 Elasticsearch 尝试恢复或重新打开一个索引， 它需要重放 translog 中所有的操作，所以如果日志越短，恢复越快。

translog 的目的是保证操作不会丢失，在文件被 fsync 到磁盘前，被写入的文件在重启之后就会丢失。默认 translog 是每 5 秒被 fsync 刷新到硬盘， 或者在每次写请求完成之后执行(e.g. index, delete, update, bulk)。这个过程在主分片和复制分片都会发生。最终， 基本上，这意味着在整个请求被 fsync 到主分片和复制分片的translog之前，你的客户端不会得到一个 200 OK 响应。

在每次请求后都执行一个 fsync 会带来一些性能损失，尽管实践表明这种损失相对较小（特别是bulk导入，它在一次请求中平摊了大量文档的开销）。

但是对于一些大容量的偶尔丢失几秒数据问题也并不严重的集群，使用异步的 fsync 还是比较有益的。比如，写入的数据被缓存到内存中，再每5秒执行一次 fsync 。如果你决定使用异步 translog 的话，你需要 保证 在发生crash时，丢失掉 sync_interval 时间段的数据也无所谓。请在决定前知晓这个特性。如果你不确定这个行为的后果，最好是使用默认的参数（ "index.translog.durability": "request" ）来避免数据丢失

#### 段合并

由于自动刷新流程每秒会创建一个新的段 ，这样会导致短时间内的段数量暴增。而段数目太多会带来较大的麻烦。 每一个段都会消耗文件句柄、内存和cpu运行周期。更重要的是，每个搜索请求都必须轮流检查每个段；所以段越多，搜索也就越慢。



Elasticsearch通过在后台进行段合并来解决这个问题。小的段被合并到大的段，然后这些大的段再被合并到更大的段

被删除的文档（或被更新文档的旧版本）不会被拷贝到新的大段中



合并大的段需要消耗大量的I/O和CPU资源，如果任其发展会影响搜索性能。Elasticsearch在默认情况下会对合并流程进行资源限制，所以搜索仍然 有足够的资源很好地执行



### 文档分析

分析 包含下面的过程：

* 将一块文本分成适合于倒排索引的独立的 词条

*  将这些词条统一化为标准格式以提高它们的“可搜索性”，或者 recall

分析器执行上面的工作。分析器实际上是将三个功能封装到了一个包里：

* 字符过滤器

  首先，字符串按顺序通过每个 字符过滤器 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 & 转化成 and

* 分词器

  其次，字符串被 分词器 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条

* Token过滤器

  最后，词条按顺序通过每个 token 过滤器 。这个过程可能会改变词条（例如，小写化 Quick ），删除词条（例如， 像 a， and， the 等无用词），或者增加词条（例如，像 jump 和 leap 这种同义词）

#### 内置分析器

Elasticsearch还附带了可以直接使用的预包装的分析器



"Set the shape to semi-transparent by calling set_trans(5)"

* 标准分析器

  标准分析器是Elasticsearch默认使用的分析器。它是分析各种语言文本最常用的选择。它根据 Unicode 联盟 定义的 单词边界 划分文本。删除绝大部分标点。最后，将词条小写。它会产生：

  ​	set, the, shape, to, semi, transparent, by, calling, set_trans, 5

* 简单分析器

  简单分析器在任何不是字母的地方分隔文本，将词条小写。它会产生：

  ​	set, the, shape, to, semi, transparent, by, calling, set, trans

* 空格分析器

  空格分析器在空格的地方划分文本。它会产生：

  ​	Set, the, shape, to, semi-transparent, by, calling, set_trans(5)

* 语言分析器

  特定语言分析器可用于 很多语言。它们可以考虑指定语言的特点。例如， 英语 分析器附带了一组英语无用词（常用单词，例如 and 或者 the ，它们对相关性没有多少影响），它们会被删除。 由于理解英语语法的规则，这个分词器可以提取英语单词的 词干 。

  英语 分词器会产生下面的词条：

  ​	set, shape, semi, transpar, call, set_tran, 5

  ​	注意看 transparent、 calling 和 set_trans 已经变为词根格式



#### 分析器使用场景

当我们 索引 一个文档，它的全文域被分析成词条以用来创建倒排索引。 但是，当我们在全文域 搜索 的时候，我们需要将查询字符串通过 相同的分析过程 ，以保证我们搜索的词条格式与索引中的词条格式一致。

全文查询，理解每个域是如何定义的，因此它们可以做正确的事：

*  当你查询一个 全文 域时， 会对查询字符串应用相同的分析器，以产生正确的搜索词条列表。

* 当你查询一个 精确值 域时，不会分析查询字符串，而是搜索你指定的精确值

#### 测试分析器

可以使用 analyze API 来看文本是如何被分析的。在消息体里，指定分析器和要分析的文本

```shell
GET http://localhost:9200/_analyze
{
  "analyzer": "standard",
  "text": "Text to analyze"
}

#结果中每个元素代表一个单独的词条：
#token 是实际存储到索引中的词条。 position 指明词条在原始文本中出现的位置。 start_offset 和 end_offset 指明字符在原始字符串中的位置
{
   "tokens": [
      {
         "token":        "text",
         "start_offset": 0,
         "end_offset":   4,
         "type":         "<ALPHANUM>",
         "position":     1
      },
      {
         "token":        "to",
         "start_offset": 5,
         "end_offset":   7,
         "type":         "<ALPHANUM>",
         "position":     2
      },
      {
         "token":        "analyze",
         "start_offset": 8,
         "end_offset":   15,
         "type":         "<ALPHANUM>",
         "position":     3
      }
   ]
}

```

#### IK分词器

IK中文分词器，下载地址为: [https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v7.8.0](https://github.com/medcl/elasticsearch-analysis-ik)

下载与 es对应的版本

安装后拷贝到 plugins 目录下

```shell
# GET http://localhost:9200/_analyze
{
	"text":"测试单词",
	"analyzer":"ik_max_word"
}

#ik_max_word：会将文本做最细粒度的拆分
#ik_smart：会将文本做最粗粒度的拆分
```

**扩展词汇**

首先进入ES根目录中的plugins文件夹下的ik文件夹，进入config目录，创建custom.dic文件，写入弗雷尔卓德。同时打开IKAnalyzer.cfg.xml文件，将新建的custom.dic配置其中，重启ES服务器

**自定义词库**

修改 /usr/share/elasticsearch/plugins/ik/config/中的 IKAnalyzer.cfg.xml

/usr/share/elasticsearch/plugins/ik/config

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 -->
        <entry key="ext_dict"></entry>
         <!--用户可以在这里配置自己的扩展停止词字典-->
        <entry key="ext_stopwords"></entry>
        <!--用户可以在这里配置远程扩展字典 使用nginx-->
         <entry key="remote_ext_dict">http://192.168.56.10/es/fenci.txt</entry>
        <!--用户可以在这里配置远程扩展停止词字典-->
        <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>

```



#### 自定义分析器

Elasticsearch真正的强大之处在于，你可以通过在一个适合你的特定数据的设置之中组合字符过滤器、分词器、词汇单元过滤器来创建自定义的分析器



**字符过滤器**

字符过滤器 用来 整理 一个尚未被分词的字符串。例如，如果我们的文本是HTML格式的，它会包含像 <p> 或者 <div> 这样的HTML标签，这些标签是我们不想索引的。我们可以使用 html清除 字符过滤器 来移除掉所有的HTML标签，并且像把 &Aacute; 转换为相对应的Unicode字符 Á 这样，转换HTML实体。一个分析器可能有0个或者多个字符过滤器。



 **分词器**

一个分析器 必须 有一个唯一的分词器。 分词器把字符串分解成单个词条或者词汇单元。 标准 分析器里使用的 标准 分词器 把一个字符串根据单词边界分解成单个词条，并且移除掉大部分的标点符号，然而还有其他不同行为的分词器存在。

例如， 关键词 分词器 完整地输出 接收到的同样的字符串，并不做任何分词。 空格 分词器 只根据空格分割文本 。 正则 分词器 根据匹配正则表达式来分割文本 。



**词单元过滤器**

经过分词，作为结果的 词单元流 会按照指定的顺序通过指定的词单元过滤器 。

词单元过滤器可以修改、添加或者移除词单元。我们已经提到过 lowercase 和 stop 词过滤器 ，但是在 Elasticsearch 里面还有很多可供选择的词单元过滤器。 词干过滤器 把单词 遏制 为 词干。 ascii_folding 过滤器移除变音符，把一个像 "très" 这样的词转换为 "tres" 。 ngram 和 edge_ngram 词单元过滤器 可以产生 适合用于部分匹配或者自动补全的词单元



创建自定义的分析器

```shell
# PUT http://localhost:9200/my_index
{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type":       "mapping",
                    "mappings": [ "&=> and "]
            }},
            "filter": {
                "my_stopwords": {
                    "type":       "stop",
                    "stopwords": [ "the", "a" ]
            }},
            "analyzer": {
                "my_analyzer": {
                    "type":         "custom",
                    "char_filter":  [ "html_strip", "&_to_and" ],
                    "tokenizer":    "standard",
                    "filter":       [ "lowercase", "my_stopwords" ]
            }}
}}}
#测试
# GET http://127.0.0.1:9200/my_index/_analyze
{
    "text":"The quick & brown fox",
    "analyzer": "my_analyzer"
}
#返回
{
    "tokens": [
        {
            "token": "quick",
            "start_offset": 4,
            "end_offset": 9,
            "type": "<ALPHANUM>",
            "position": 1
        },
        {
            "token": "and",
            "start_offset": 10,
            "end_offset": 11,
            "type": "<ALPHANUM>",
            "position": 2
        },
        {
            "token": "brown",
            "start_offset": 12,
            "end_offset": 17,
            "type": "<ALPHANUM>",
            "position": 3
        },
        {
            "token": "fox",
            "start_offset": 18,
            "end_offset": 21,
            "type": "<ALPHANUM>",
            "position": 4
        }
    ]
}

```

### 文档处理

#### 文档冲突

当我们使用 index API 更新文档 ，可以一次性读取原始文档，做我们的修改，然后重新索引 整个文档 。 最近的索引请求将获胜：无论最后哪一个文档被索引，都将被唯一存储在 Elasticsearch 中。如果其他人同时更改这个文档，他们的更改将丢失。

很多时候这是没有问题的。也许我们的主数据存储是一个关系型数据库，我们只是将数据复制到 Elasticsearch 中并使其可被搜索。 也许两个人同时更改相同的文档的几率很小。或者对于我们的业务来说偶尔丢失更改并不是很严重的问题。

但有时丢失了一个变更就是 非常严重的 。试想我们使用 Elasticsearch 存储我们网上商城商品库存的数量， 每次我们卖一个商品的时候，我们在 Elasticsearch 中将库存数量减少。有一天，管理层决定做一次促销。突然地，我们一秒要卖好几个商品。 假设有两个 web 程序并行运行，每一个都同时处理所有商品的销售



在数据库领域中，有两种方法通常被用来确保并发更新时变更不会丢失：

**悲观并发控制**

这种方法被关系型数据库广泛使用，它假定有变更冲突可能发生，因此阻塞访问资源以防止冲突。 一个典型的例子是读取一行数据之前先将其锁住，确保只有放置锁的线程能够对这行数据进行修改。

**乐观并发控制**

Elasticsearch 中使用的这种方法假定冲突是不可能发生的，并且不会阻塞正在尝试的操作。 然而，如果源数据在读写当中被修改，更新将会失败。应用程序接下来将决定该如何解决冲突。 例如，可以重试更新、使用新的数据、或者将相关情况报告给用户

#### 乐观并发控制

Elasticsearch 是分布式的。当文档创建、更新或删除时， 新版本的文档必须复制到集群中的其他节点。Elasticsearch 也是异步和并发的，这意味着这些复制请求被并行发送，并且到达目的地时也许 顺序是乱的 。 Elasticsearch 需要一种方法确保文档的旧版本不会覆盖新的版本。

当我们之前讨论 index ， GET 和 delete 请求时，我们指出每个文档都有一个 _version （版本）号，当文档被修改时版本号递增。 Elasticsearch 使用这个 version 号来确保变更以正确顺序得到执行。如果旧版本的文档在新版本之后到达，它可以被简单的忽略。

我们可以利用 version 号来确保 应用中相互冲突的变更不会导致数据丢失。我们通过指定想要修改文档的 version 号来达到这个目的。 如果该版本不是当前版本号，我们的请求将会失败。

老的版本es使用version，但是新版本不支持了，会报下面的错误，提示我们用if_seq_no和if_primary_term

#### 外部系统版本控制

一个常见的设置是使用其它数据库作为主要的数据存储，使用 Elasticsearch 做数据检索， 这意味着主数据库的所有更改发生时都需要被复制到 Elasticsearch ，如果多个进程负责这一数据同步，你可能遇到类似于之前描述的并发问题。

如果你的主数据库已经有了版本号 — 或一个能作为版本号的字段值比如 timestamp — 那么你就可以在 Elasticsearch 中通过增加 version_type=external 到查询字符串的方式重用这些相同的版本号， 版本号必须是大于零的整数， 且小于 9.2E+18 — 一个 Java 中 long 类型的正值。

外部版本号的处理方式和我们之前讨论的内部版本号的处理方式有些不同， Elasticsearch 不是检查当前 _version 和请求中指定的版本号是否相同， 而是检查当前 _version 是否 小于 指定的版本号。 如果请求成功，外部的版本号作为文档的新 _version 进行存储。

外部版本号不仅在索引和删除请求是可以指定，而且在 创建 新文档时也可以指定。

## 集成



### Elasticsearch - Rest - client

1. 9300：TCP

   Spring-data-elasticsearch:transport-api.jar

   SpringBoot版本不同，`transport-api.jar` 不同，不能适配 es 版本

   7.x 已经不在适合使用，8 以后就要废弃

2. 9200：HTTP

   JestClient 非官方，更新慢

   RestTemplate:默认发送 HTTP 请求，ES很多操作都需要自己封装、麻烦

   HttpClient：同上

   **Elasticsearch - Rest - Client：官方RestClient，封装了 ES 操作，API层次分明**

最终选择 Elasticsearch - Rest - Client （elasticsearch - rest - high - level - client）

### SpringBoot 整合

```xml
<!-- 导入es的 rest-high-level-client -->
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.4.2</version>
</dependency>
```

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-getting-started-maven.html

```java
 * 1、导入配置
 * 2、编写配置，给容器注入一个RestHighLevelClient
 * 3、参照API 官网进行开发
 */
@Configuration
public class GulimallElasticsearchConfig {


    public static final RequestOptions COMMON_OPTIONS;
    static {
        RequestOptions.Builder builder = RequestOptions.DEFAULT.toBuilder();
//        builder.addHeader("Authorization", "Bearer " + TOKEN);
//        builder.setHttpAsyncResponseConsumerFactory(
//                new HttpAsyncResponseConsumerFactory
//                        .HeapBufferedResponseConsumerFactory(30 * 1024 * 1024 * 1024));
        COMMON_OPTIONS = builder.build();
    }



    @Bean
    public RestHighLevelClient esRestClient() {
        RestClientBuilder builder = null;
        builder = RestClient.builder(new HttpHost("192.168.56.10", 9200, "http"));

        RestHighLevelClient client = new RestHighLevelClient(builder);
//        RestHighLevelClient client = new RestHighLevelClient(
//                RestClient.builder(
//                        new HttpHost("localhost", 9200, "http"),
//                        new HttpHost("localhost", 9201, "http")));
        return client;
    }

}
```

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-getting-started-initialization.html

```java
@Autowired
private RestHighLevelClient client;
/**
 * 添加或者更新
 * @throws IOException
 */
@Test
public void indexData() throws IOException {
    IndexRequest indexRequest = new IndexRequest("users");
    User user = new User();
    user.setAge(19);
    user.setGender("男");
    user.setUserName("张三");
    String jsonString = JSON.toJSONString(user);
    indexRequest.source(jsonString,XContentType.JSON);

    // 执行操作
    IndexResponse index = client.index(indexRequest,GulimallElasticsearchConfig.COMMON_OPTIONS);

    // 提取有用的响应数据
    System.out.println(index);
}

@Test
public void searchTest() throws IOException {
    // 1、创建检索请求
    SearchRequest searchRequest = new SearchRequest();
    // 指定索引
    searchRequest.indices("bank");
    // 指定 DSL，检索条件
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

    sourceBuilder.query(QueryBuilders.matchQuery("address", "mill"));

    //1、2 按照年龄值分布进行聚合
    TermsAggregationBuilder aggAvg = AggregationBuilders.terms("ageAgg").field("age").size(10);
    sourceBuilder.aggregation(aggAvg);

    //1、3 计算平均薪资
    AvgAggregationBuilder balanceAvg = AggregationBuilders.avg("balanceAvg").field("balance");
    sourceBuilder.aggregation(balanceAvg);


    System.out.println("检索条件" + sourceBuilder.toString());

    searchRequest.source(sourceBuilder);

    // 2、执行检索
    SearchResponse searchResponse = client.search(searchRequest, GulimallElasticsearchConfig.COMMON_OPTIONS);

    // 3、分析结果
    System.out.println(searchResponse.toString());

    // 4、拿到命中得结果
    SearchHits hits = searchResponse.getHits();
    // 5、搜索请求的匹配
    SearchHit[] searchHits = hits.getHits();
    // 6、进行遍历
    for (SearchHit hit : searchHits) {
        // 7、拿到完整结果字符串
        String sourceAsString = hit.getSourceAsString();
        // 8、转换成实体类
        Accout accout = JSON.parseObject(sourceAsString, Accout.class);
        System.out.println("account:" + accout );
    }

    // 9、拿到聚合
    Aggregations aggregations = searchResponse.getAggregations();
    //        for (Aggregation aggregation : aggregations) {
    //
    //        }
    // 10、通过先前名字拿到对应聚合
    Terms ageAgg1 = aggregations.get("ageAgg");
    for (Terms.Bucket bucket : ageAgg1.getBuckets()) {
        // 11、拿到结果
        String keyAsString = bucket.getKeyAsString();
        System.out.println("年龄:" + keyAsString);
        long docCount = bucket.getDocCount();
        System.out.println("个数：" + docCount);
    }
    Avg balanceAvg1 = aggregations.get("balanceAvg");
    System.out.println("平均薪资：" + balanceAvg1.getValue());
    System.out.println(searchResponse.toString());
}
```

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-search.html#java-rest-high-search-request-optional





### Spring Data

一个用于简化数据库、非关系型数据库、索引库访问，并支持云服务的开源框架

https://spring.io/projects/spring-data

 Spring Data Elasticsearch

https://spring.io/projects/spring-data-elasticsearch

目前最新springboot对应Elasticsearch7.6.2，Spring boot2.3.x一般可以兼容Elasticsearch7.x



#### demo

##### 搭建

pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.6.RELEASE</version>
        <relativePath/>
    </parent>


    <groupId>com.atguigu.es</groupId>
    <artifactId>springdata-elasticsearch</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-test</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
        </dependency>
    </dependencies>
</project>
```

application.properties

```properties
# es服务地址
elasticsearch.host=127.0.0.1
# es服务端口
elasticsearch.port=9200
# 配置日志级别,开启debug日志
logging.level.com.atguigu.es=debug
```

数据实体类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class Product {
    private Long id;//商品唯一标识
    private String title;//商品名称
    private String category;//分类名称
    private Double price;//商品价格
    private String images;//图片地址

}
```

配置类

* ElasticsearchRestTemplate是spring-data-elasticsearch项目中的一个类，和其他spring项目中的template类似。

* 在新版的spring-data-elasticsearch中，ElasticsearchRestTemplate代替了原来的ElasticsearchTemplate。

* 原因是ElasticsearchTemplate基于TransportClient，TransportClient即将在8.x以后的版本中移除。所以，我们推荐使用ElasticsearchRestTemplate。

* ElasticsearchRestTemplate基于RestHighLevelClient客户端的。需要自定义配置类，继承AbstractElasticsearchConfiguration，并实现elasticsearchClient()抽象方法，创建RestHighLevelClient对象。

```java
@ConfigurationProperties(prefix = "elasticsearch")
@Configuration
@Data
public class ElasticsearchConfig extends AbstractElasticsearchConfiguration {
    private String host ;
    private Integer port ;

    //重写父类方法
    @Override
    public RestHighLevelClient elasticsearchClient() {
        RestClientBuilder builder = RestClient.builder(new HttpHost(host, port));
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);
        return restHighLevelClient;
    }
}
```

dao

```java
@Repository
public interface ProductDao extends ElasticsearchRepository<Product,Long> {

}
```

 实体类映射操作

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Document(indexName = "shopping", shards = 3, replicas = 1)
public class Product {
    //必须有id,这里的id是全局唯一的标识，等同于es中的"_id"
    @Id
    private Long id;//商品唯一标识
    /**
     * type : 字段数据类型
     * analyzer : 分词器类型
     * index : 是否索引(默认:true)
     * Keyword : 短语,不进行分词
     */
    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String title;//商品名称
    @Field(type = FieldType.Keyword)
    private String category;//分类名称
    @Field(type = FieldType.Double)
    private Double price;//商品价格
    @Field(type = FieldType.Keyword, index = false)
    private String images;//图片地址
}
```

##### 测试

索引操作

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringDataESIndexTest {
    //注入ElasticsearchRestTemplate
    @Autowired
    private ElasticsearchRestTemplate elasticsearchRestTemplate;

    //创建索引并增加映射配置
    @Test
    public void createIndex(){
        //创建索引，系统初始化会自动创建索引
        System.out.println("创建索引");
    }

    @Test
    public void deleteIndex(){
        //创建索引，系统初始化会自动创建索引
        boolean flg = elasticsearchRestTemplate.deleteIndex(Product.class);
        System.out.println("删除索引 = " + flg);
    }
}
```

文档操作

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringDataESProductDaoTest {
    @Autowired
    private ProductDao productDao;

    /**
     * 新增
     */
    @Test
    public void save(){
        Product product = new Product();
        product.setId(2L);
        product.setTitle("华为手机");
        product.setCategory("手机");
        product.setPrice(2999.0);
        product.setImages("http://www.atguigu/hw.jpg");
        productDao.save(product);
    }

    //修改
    @Test
    public void update(){
        Product product = new Product();
        product.setId(1L);
        product.setTitle("小米2手机");
        product.setCategory("手机");
        product.setPrice(9999.0);
        product.setImages("http://www.atguigu/xm.jpg");
        productDao.save(product);
    }

    //根据id查询
    @Test
    public void findById(){
        Product product = productDao.findById(1L).get();
        System.out.println(product);
    }

    //查询所有
    @Test
    public void findAll(){
        Iterable<Product> products = productDao.findAll();
        for (Product product : products) {
            System.out.println(product);
        }
    }

    //删除
    @Test
    public void delete(){
        Product product = new Product();
        product.setId(1L);
        productDao.delete(product);
    }

    //批量新增
    @Test
    public void saveAll(){
        List<Product> productList = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            Product product = new Product();
            product.setId(Long.valueOf(i));
            product.setTitle("["+i+"]小米手机");
            product.setCategory("手机");
            product.setPrice(1999.0+i);
            product.setImages("http://www.atguigu/xm.jpg");
            productList.add(product);
        }
        productDao.saveAll(productList);
    }

    //分页查询
    @Test
    public void findByPageable(){
        //设置排序(排序方式，正序还是倒序，排序的id)
        Sort sort = Sort.by(Sort.Direction.DESC,"id");
        int currentPage=0;//当前页，第一页从0开始，1表示第二页
        int pageSize = 5;//每页显示多少条
        //设置查询分页
        PageRequest pageRequest = PageRequest.of(currentPage, pageSize,sort);
        //分页查询
        Page<Product> productPage = productDao.findAll(pageRequest);
        for (Product Product : productPage.getContent()) {
            System.out.println(Product);
        }
    }
}

```

文档搜索

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringDataESSearchTest {
    @Autowired
    private ProductDao productDao;

    /**
     * term查询
     * search(termQueryBuilder) 调用搜索方法，参数查询构建器对象
     */
    @Test
    public void termQuery(){
        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("title", "小米");
        Iterable<Product> products = productDao.search(termQueryBuilder);
        for (Product product : products) {
            System.out.println(product);
        }
    }

    /**
     * term查询加分页
     */
    @Test
    public void termQueryByPage(){
        int currentPage= 0 ;
        int pageSize = 5;
        //设置查询分页
        PageRequest pageRequest = PageRequest.of(currentPage, pageSize);
        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("title", "小米");
        Iterable<Product> products = productDao.search(termQueryBuilder,pageRequest);
        for (Product product : products) {
            System.out.println(product);
        }
    }
}
```

### Spark Streaming集成

Spark Streaming是Spark core API的扩展，支持实时数据流的处理，并且具有可扩展，高吞吐量，容错的特点。 数据可以从许多来源获取，如Kafka，Flume，Kinesis或TCP sockets，并且可以使用复杂的算法进行处理，这些算法使用诸如map，reduce，join和window等高级函数表示。 最后，处理后的数据可以推送到文件系统，数据库等。 实际上，您可以将Spark的机器学习和图形处理算法应用于数据流



#### demo

创建Maven项目 pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.es</groupId>
    <artifactId>sparkstreaming-elasticsearch</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>3.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming_2.12</artifactId>
            <version>3.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>7.8.0</version>
        </dependency>
        <!-- elasticsearch的客户端 -->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>7.8.0</version>
        </dependency>
        <!-- elasticsearch依赖2.x的log4j -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>2.8.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.8.2</version>
        </dependency>
<!--        <dependency>-->
<!--            <groupId>com.fasterxml.jackson.core</groupId>-->
<!--            <artifactId>jackson-databind</artifactId>-->
<!--            <version>2.11.1</version>-->
<!--        </dependency>-->
<!--        &lt;!&ndash; junit单元测试 &ndash;&gt;-->
<!--        <dependency>-->
<!--            <groupId>junit</groupId>-->
<!--            <artifactId>junit</artifactId>-->
<!--            <version>4.12</version>-->
<!--        </dependency>-->
    </dependencies>
</project>

```

```java
object SparkStreamingESTest {

    def main(args: Array[String]): Unit = {
        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("ESTest")
        val ssc = new StreamingContext(sparkConf, Seconds(3))

        val ds: ReceiverInputDStream[String] = ssc.socketTextStream("localhost", 9999)

        ds.foreachRDD(
            rdd => {
                println("*************** " + new Date())
                rdd.foreach(
                    data => {
                        val client = new RestHighLevelClient(
                            RestClient.builder(new HttpHost("localhost", 9200, "http"))
                        );
                        // 新增文档 - 请求对象
                        val request = new IndexRequest();
                        // 设置索引及唯一性标识
                        val ss = data.split(" ")
                        println("ss = " + ss.mkString(","))
                        request.index("sparkstreaming").id(ss(0));
                        val productJson =
                            s"""
                              | { "data":"${ss(1)}" }
                              |""".stripMargin;
                        // 添加文档数据，数据格式为JSON格式
                        request.source(productJson,XContentType.JSON);
                        // 客户端发送请求，获取响应对象
                        val response = client.index(request, RequestOptions.DEFAULT);
                        System.out.println("_index:" + response.getIndex());
                        System.out.println("_id:" + response.getId());
                        System.out.println("_result:" + response.getResult());

                        client.close()
                    }
                )
            }
        )
        ssc.start()
        ssc.awaitTermination()
    }
}
```

### Flink集成

Apache Spark是一种基于内存的快速、通用、可扩展的大数据分析计算引擎。

Apache Spark掀开了内存计算的先河，以内存作为赌注，赢得了内存计算的飞速发展。但是在其火热的同时，开发人员发现，在Spark中，计算框架普遍存在的缺点和不足依然没有完全解决，而这些问题随着5G时代的来临以及决策者对实时数据分析结果的迫切需要而凸显的更加明显

* 数据精准一次性处理（Exactly-Once）

*  乱序数据，迟到数据

* 低延迟，高吞吐，准确性

*  容错性

Apache Flink是一个框架和分布式处理引擎，用于对无界和有界数据流进行有状态计算。在Spark火热的同时，也默默地发展自己，并尝试着解决其他计算框架的问题。

慢慢地，随着这些问题的解决，Flink慢慢被绝大数程序员所熟知并进行大力推广，阿里公司在2015年改进Flink，并创建了内部分支Blink，目前服务于阿里集团内部搜索、推荐、广告和蚂蚁等大量核心实时业务。

#### demo

Maven项目pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.es</groupId>
    <artifactId>flink-elasticsearch</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-scala_2.12</artifactId>
            <version>1.12.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-scala_2.12</artifactId>
            <version>1.12.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_2.12</artifactId>
            <version>1.12.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-elasticsearch7_2.11</artifactId>
            <version>1.12.0</version>
        </dependency>

        <!-- jackson -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.11.1</version>
        </dependency>
    </dependencies>
</project>

```

```java
public class FlinkElasticsearchSinkTest {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<String> source = env.socketTextStream("localhost", 9999);

        List<HttpHost> httpHosts = new ArrayList<>();
        httpHosts.add(new HttpHost("127.0.0.1", 9200, "http"));
        //httpHosts.add(new HttpHost("10.2.3.1", 9200, "http"));

// use a ElasticsearchSink.Builder to create an ElasticsearchSink
        ElasticsearchSink.Builder<String> esSinkBuilder = new ElasticsearchSink.Builder<>(
                httpHosts,
                new ElasticsearchSinkFunction<String>() {
                    public IndexRequest createIndexRequest(String element) {
                        Map<String, String> json = new HashMap<>();
                        json.put("data", element);

                        return Requests.indexRequest()
                                .index("my-index")
                                //.type("my-type")
                                .source(json);
                    }

                    @Override
                    public void process(String element, RuntimeContext ctx, RequestIndexer indexer) {
                        indexer.add(createIndexRequest(element));
                    }
                }
        );

// configuration for the bulk requests; this instructs the sink to emit after every element, otherwise they would be buffered
        esSinkBuilder.setBulkFlushMaxActions(1);

// provide a RestClientFactory for custom configuration on the internally created REST client
//        esSinkBuilder.setRestClientFactory(
//                restClientBuilder -> {
//                    restClientBuilder.setDefaultHeaders(...)
//                    restClientBuilder.setMaxRetryTimeoutMillis(...)
//                    restClientBuilder.setPathPrefix(...)
//                    restClientBuilder.setHttpClientConfigCallback(...)
//                }
//        );

        source.addSink(esSinkBuilder.build());

        env.execute("flink-es");
    }
}

```

## 优化

### 硬件选择

Elasticsearch的基础是 Lucene，所有的索引和文档数据是存储在本地的磁盘中，具体的路径可在 ES 的配置文件../config/elasticsearch.yml中配置path.data



1. 使用 SSD
2. 使用 RAID 0。条带化 RAID 会提高磁盘 I/O，代价显然就是当一块硬盘故障时整个就故障了。不要使用镜像或者奇偶校验 RAID 因为副本已经提供了这个功能。
3. 另外，使用多块硬盘，并允许 Elasticsearch 通过多个 path.data 目录配置把数据条带化分配到它们上面。
4. 不要使用远程挂载的存储，比如 NFS 或者 SMB/CIFS。这个引入的延迟对性能来说完全是背道而驰的。

### 分片策略

#### 合理设置分片数

分片和副本的设计为 ES 提供了支持分布式和故障转移的特性，但并不意味着分片和副本是可以无限分配的。而且索引的分片完成分配后由于索引的路由机制，我们是不能重新修改分片数的。

一个分片并不是没有代价的

* 一个分片的底层即为一个 Lucene 索引，会消耗一定文件句柄、内存、以及 CPU 运转。

* 每一个搜索请求都需要命中索引中的每一个分片，如果每一个分片都处于不同的节点还好， 但如果多个分片都需要在同一个节点上竞争使用相同的资源就有些糟糕了。 

*  用于计算相关度的词项统计信息是基于分片的。如果有许多分片，每一个都只有很少的数据会导致很低的相关度。



一个业务索引具体需要分配多少分片可能需要架构师和技术人员对业务的增长有个预先的判断，横向扩展应当分阶段进行。为下一阶段准备好足够的资源。 只有当你进入到下一个阶段，你才有时间思考需要作出哪些改变来达到这个阶段。一般来说，我们遵循一些原则：

* 控制每个分片占用的硬盘容量不超过ES的最大JVM的堆空间设置（一般设置不超过32G，参考下文的JVM设置原则），因此，如果索引的总容量在500G左右，那分片大小在16个左右即可；当然，最好同时考虑原则2。

* 考虑一下node数量，一般一个节点有时候就是一台物理机，如果分片数过多，大大超过了节点数，很可能会导致一个节点上存在多个分片，一旦该节点故障，即使保持了1个以上的副本，同样有可能会导致数据丢失，集群无法恢复。所以， 一般都设置分片数不超过节点数的3倍。

*  主分片，副本和节点最大数之间数量，我们分配的时候可以参考以下关系：节点数<=主分片数*（副本数+1）

#### 推迟分片分配

对于节点瞬时中断的问题，默认情况，集群会等待一分钟来查看节点是否会重新加入，如果这个节点在此期间重新加入，重新加入的节点会保持其现有的分片数据，不会触发新的分片分配。这样就可以减少 ES 在自动再平衡可用分片时所带来的极大开销。

通过修改参数 delayed_timeout ，可以延长再均衡的时间，可以全局设置也可以在索引级别进行修改:

```shell
PUT /_all/_settings 
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "5m" 
  }
}
```

### 路由选择

当我们查询文档的时候，Elasticsearch 如何知道一个文档应该存放到哪个分片中呢？它其实是通过下面这个公式来计算出来:

shard = hash(routing) % number_of_primary_shards

routing 默认值是文档的 id，也可以采用自定义值，比如用户 id。

**不带 routing** **查询**

在查询的时候因为不知道要查询的数据具体在哪个分片上，所以整个过程分为 2 个步骤

* 分发：请求到达协调节点后，协调节点将查询请求分发到每个分片上。

*  聚合: 协调节点搜集到每个分片上查询结果，在将查询的结果进行排序，之后给用户返回结果。

**带 routing** **查询**

查询的时候，可以直接根据 routing 信息定位到某个分配查询，不需要查询所有的分配，经过协调节点排序。

向上面自定义的用户查询，如果 routing 设置为 userid 的话，就可以直接查询出数据来，效率提升很多。

### 写入速度优化

ES的默认配置，是综合了数据可靠性、写入速度、搜索实时性等因素。实际使用时，我们需要根据公司要求，进行偏向性的优化。

针对于搜索性能要求不高，但是对写入要求较高的场景，我们需要尽可能的选择恰当写优化策略。综合来说，可以考虑以下几个方面来提升写索引的性能：

* 加大 Translog Flush ，目的是降低 Iops、Writeblock。

* 增加 Index Refresh 间隔，目的是减少 Segment Merge 的次数。
* 调整 Bulk 线程池和队列。
* 优化节点间的任务分布。
* 优化 Lucene 层的索引建立，目的是降低 CPU 及 IO。

#### 批量数据提交

ES 提供了 Bulk API 支持批量操作，当我们有大量的写任务时，可以使用 Bulk 来进行批量写入。

通用的策略如下：Bulk 默认设置批量提交的数据量不能超过 100M。数据条数一般是根据文档的大小和服务器性能而定的，但是单次批处理的数据大小应从 5MB～15MB 逐渐增加，当性能没有提升时，把这个数据量作为最大值

#### 优化存储设备

ES 是一种密集使用磁盘的应用，在段合并的时候会频繁操作磁盘，所以对磁盘要求较高，当磁盘速度提升之后，集群的整体性能会大幅度提高

#### 合理使用合并

Lucene 以段的形式存储数据。当有新的数据写入索引时，Lucene 就会自动创建一个新的段。

随着数据量的变化，段的数量会越来越多，消耗的多文件句柄数及 CPU 就越多，查询效率就会下降。

由于 Lucene 段合并的计算量庞大，会消耗大量的 I/O，所以 ES 默认采用较保守的策略，让后台定期进行段合并

#### 减少Refresh的次数

Lucene 在新增数据时，采用了延迟写入的策略，默认情况下索引的 refresh_interval 为 1 秒。

Lucene 将待写入的数据先写到内存中，超过 1 秒（默认）时就会触发一次 Refresh，然后 Refresh 会把内存中的的数据刷新到操作系统的文件缓存系统中。

如果我们对搜索的实效性要求不高，可以将 Refresh 周期延长，例如 30 秒。

这样还可以有效地减少段刷新次数，但这同时意味着需要消耗更多的Heap内存

#### 加大Flush设置

Flush 的主要目的是把文件缓存系统中的段持久化到硬盘，当 Translog 的数据量达到 512MB 或者 30 分钟时，会触发一次 Flush。

index.translog.flush_threshold_size 参数的默认值是 512MB，我们进行修改。

增加参数值意味着文件缓存系统中可能需要存储更多的数据，所以我们需要为操作系统的文件缓存系统留下足够的空间

#### 减少副本的数量

ES 为了保证集群的可用性，提供了 Replicas（副本）支持，然而每个副本也会执行分析、索引及可能的合并过程，所以 Replicas 的数量会严重影响写索引的效率。

当写索引时，需要把写入的数据都同步到副本节点，副本节点越多，写索引的效率就越慢。

如果我们需要大批量进行写入操作，可以先禁止 Replica 复制，设置 index.number_of_replicas: 0 关闭副本。在写入完成后，Replica 修改回正常的状态

### 内存设置

ES 默认安装后设置的内存是 1GB，对于任何一个现实业务来说，这个设置都太小了。

如果是通过解压安装的 ES，则在 ES 安装文件中包含一个 jvm.option 文件，添加如下命令来设置 ES 的堆大小，Xms 表示堆的初始大小，Xmx 表示可分配的最大内存，都是 1GB。

确保 Xmx 和 Xms 的大小是相同的，其目的是为了能够在 Java 垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小而浪费资源，可以减轻伸缩堆大小带来的压力。



ES 堆内存的分配需要满足以下两个原则：

* 不要超过物理内存的 50%：Lucene 的设计目的是把底层 OS 里的数据缓存到内存中。

  Lucene 的段是分别存储到单个文件中的，这些文件都是不会变化的，所以很利于缓存，同时操作系   统也会把这些段文件缓存起来，以便更快的访问。

  如果我们设置的堆内存过大，Lucene 可用的内存将会减少，就会严重影响降低 Lucene 的全文本查  询性能。

* 堆内存的大小最好不要超过 32GB：在 Java 中，所有对象都分配在堆上，然后有一个 Klass Pointer 指针指向它的类元数据。

  这个指针在 64 位的操作系统上为 64 位，64 位的操作系统可以使用更多的内存（2^64）。在 32 位  的系统上为 32 位，32 位的操作系统的最大寻址空间为 4GB（2^32）。

  但是 64 位的指针意味着更大的浪费，因为你的指针本身大了。浪费内存不算，更糟糕的是，更大的   指针在主内存和缓存器（例如 LLC, L1等）之间移动数据的时候，会占用更多的带宽。

  

最终我们都会采用 31 G 设置

  -Xms 31g

  -Xmx 31g

假设你有个机器有 128 GB 的内存，你可以创建两个节点，每个节点内存分配不超过 32 GB。 也就是说不超过 64 GB 内存给 ES 的堆内存，剩下的超过 64 GB 的内存给 Lucene

### 重要配置

| 参数名                             | 参数值        | 说明                                                         |
| ---------------------------------- | ------------- | ------------------------------------------------------------ |
| cluster.name                       | elasticsearch | 配置 ES 的集群名称，默认值是ES，建议改成与所存数据相关的名称，ES 会自动发现在同一网段下的集群名称相同的节点 |
| node.name                          | node-1        | 集群中的节点名，在同一个集群中不能重复。节点的名称一旦设置，就不能再改变了。当然，也可以设置成服务器的主机名称，例如 node.name:${HOSTNAME}。 |
| node.master                        | true          | 指定该节点是否有资格被选举成为 Master 节点，默认是  True，如果被设置为 True，则只是有资格成为 Master 节点，具体能否成为  Master 节点，需要通过选举产生。 |
| node.data                          | true          | 指定该节点是否存储索引数据，默认为 True。数据的增、删、改、查都是在 Data 节点完成的。 |
| index.number_of_shards             | 1             | 设置都索引分片个数，默认是 1 片。也可以在创建索引时设置该值，具体设置为多大都值要根据数据量的大小来定。如果数据量不大，则设置成 1 时效率最高 |
| index.number_of_replicas           | 1             | 设置默认的索引副本个数，默认为 1 个。副本数越多，集群的可用性越好，但是写索引时需要同步的数据越多。 |
| transport.tcp.compress             | true          | 设置在节点间传输数据时是否压缩，默认为 False，不压缩         |
| discovery.zen.minimum_master_nodes | 1             | 设置在选举 Master 节点时需要参与的最少的候选主节点数，默认为 1。如果使用默认值，则当网络不稳定时有可能会出现脑裂。  合理的数值为(master_eligible_nodes/2)+1，其中 master_eligible_nodes 表示集群中的候选主节点数 |
| discovery.zen.ping.timeout         | 3s            | 设置在集群中自动发现其他节点时 Ping 连接的超时时间，默认为  3 秒。  在较差的网络环境下需要设置得大一点，防止因误判该节点的存活状态而导致分片的转移 |

## 面试题

### 为什么要使用Elasticsearch?

使用ES做一个全文索引，提高查询速度。

### Elasticsearch的master选举流程？

* Elasticsearch的选主是ZenDiscovery模块负责的，主要包含Ping（节点之间通过这个RPC来发现彼此）和Unicast（单播模块包含一个主机列表以控制哪些节点需要ping通）这两部分
* 对所有可以成为master的节点（node.master: true）根据nodeId字典排序，每次选举每个节点都把自己所知道节点排一次序，然后选出第一个（第0位）节点，暂且认为它是master节点。
* 如果对某个节点的投票数达到一定的值（可以成为master节点数n/2+1）并且该节点自己也选举自己，那这个节点就是master。否则重新选举一直到满足上述条件。
* master节点的职责主要包括集群、节点和索引的管理，不负责文档级别的管理；data节点可以关闭http功能。

### Elasticsearch集群脑裂问题？

“脑裂”问题可能的成因:

* 网络问题
* 节点负载：主节点的角色既为master又为data，访问量较大时可能会导致ES停止响应造成大面积延迟，此时其他节点得不到主节点的响应认为主节点挂掉了，会重新选取主节点。
* 内存回收：data节点上的ES进程占用的内存较大，引发JVM的大规模内存回收，造成ES进程失去响应。

脑裂问题解决方案：
* 减少误判：discovery.zen.ping_timeout节点状态的响应时间，默认为3s，可以适当调大
* 选举触发: discovery.zen.minimum_master_nodes:1
    该参数是用于控制选举行为发生的最小集群主节点数量。当备选主节点的个数大于等于该参数的值，	且备选主节点中有该参数个节点认为主节点挂了，进行选举。官方建议为（n/2）+1，n为主节点个数	（即有资格成为主节点的节点个数）
* 角色分离：即master节点与data节点分离，限制角色
    主节点配置为：node.master: true node.data: false
    从节点配置为：node.master: false node.data: true

### Elasticsearch索引文档的流程？

* 协调节点默认使用文档ID参与计算（也支持通过routing），以便为路由提供合适的分片：
  shard = hash(document_id) % (num_of_primary_shards)
* 当分片所在的节点接收到来自协调节点的请求后，会将请求写入到Memory Buffer，然后定时（默认是每隔1秒）写入到Filesystem Cache，这个从Memory Buffer到Filesystem Cache的过程就叫做refresh；
* 当然在某些情况下，存在Momery Buffer和Filesystem Cache的数据可能会丢失，ES是通过translog的机制来保证数据的可靠性的。其实现机制是接收到请求后，同时也会写入到translog中，当Filesystem cache中的数据写入到磁盘中时，才会清除掉，这个过程叫做flush；
* 在flush过程中，内存中的缓冲将被清除，内容被写入一个新段，段的fsync将创建一个新的提交点，并将内容刷新到磁盘，旧的translog将被删除并开始一个新的translog。
* flush触发的时机是定时触发（默认30分钟）或者translog变得太大（默认为512M）时；  



### Elasticsearch更新和删除文档的流程？

* 删除和更新也都是写操作，但是Elasticsearch中的文档是不可变的，因此不能被删除或者改动以展示其变更；
* 磁盘上的每个段都有一个相应的.del文件。当删除请求发送后，文档并没有真的被删除，而是在.del文件中被标记为删除。该文档依然能匹配查询，但是会在结果中被过滤掉。当段合并时，在.del文件中被标记为删除的文档将不会被写入新段。
* 在新的文档被创建时，Elasticsearch会为该文档指定一个版本号，当执行更新时，旧版本的文档在.del文件中被标记为删除，新版本的文档被索引到一个新段。旧版本的文档依然能匹配查询，但是会在结果中被过滤掉。 

### Elasticsearch搜索的流程？

* 搜索被执行成一个两阶段过程，我们称之为 Query Then Fetch；
* 在初始查询阶段时，查询会广播到索引中每一个分片拷贝（主分片或者副本分片）。 每个分片在本地执行搜索并构建一个匹配文档的大小为 from + size 的优先队列。PS：在搜索的时候是会查询Filesystem Cache的，但是有部分数据还在Memory Buffer，所以搜索是近实时的。
* 每个分片返回各自优先队列中 所有文档的 ID 和排序值 给协调节点，它合并这些值到自己的优先队列中来产生一个全局排序后的结果列表。
* 接下来就是取回阶段，协调节点辨别出哪些文档需要被取回并向相关的分片提交多个 GET 请求。每个分片加载并丰富文档，如果有需要的话，接着返回文档给协调节点。一旦所有的文档都被取回了，协调节点返回结果给客户端。
* Query Then Fetch的搜索类型在文档相关性打分的时候参考的是本分片的数据，这样在文档数量较少的时候可能不够准确，DFS Query Then Fetch增加了一个预查询的处理，询问Term和Document frequency，这个评分更准确，但是性能会变差。 

### Elasticsearch 在部署时，对 Linux 的设置有哪些优化方法？

* 64 GB 内存/32 GB /16 GB    少于8 GB 不行。
* 更快的 CPUs 和更多的核心之间选择，选择更多的核心更好。多个内核提供的额外并发远胜过稍微快一点点的时钟频率。
* SSD
* 避免集群跨越多个数据中心。绝对要避免集群跨越大的地理距离。
* 请确保运行你应用程序的 JVM 和服务器的 JVM 是完全一样的。 在 Elasticsearch 的几个地方，使用 Java 的本地序列化。
* 通过设置gateway.recover_after_nodes、gateway.expected_nodes、gateway.recover_after_time可以在集群重启的时候避免过多的分片交换，这可能会让数据恢复从数个小时缩短为几秒钟。
* Elasticsearch 默认被配置为使用单播发现，以防止节点无意中加入集群。只有在同一台机器上运行的节点才会自动组成集群。最好使用单播代替组播。
* 不要随意修改垃圾回收器（CMS）和各个线程池的大小。
* 把你的内存的（少于）一半给 Lucene（但不要超过 32 GB！），通过ES_HEAP_SIZE 环境变量设置。
* 内存交换到磁盘对服务器性能来说是致命的。如果内存交换到磁盘上，一个 100 微秒的操作可能变成 10 毫秒。 再想想那么多 10 微秒的操作时延累加起来。 不难看出 swapping 对于性能是多么可怕。
* Lucene 使用了大量的文件。同时，Elasticsearch 在节点和 HTTP 客户端之间进行通信也使用了大量的套接字。 所有这一切都需要足够的文件描述符。你应该增加你的文件描述符，设置一个很大的值，如 64,000。



补充：索引阶段性能提升方法

* 使用批量请求并调整其大小：每次批量数据 5–15 MB 大是个不错的起始点。
* 使用 SSD

* 段和合并：Elasticsearch 默认值是 20 MB/s，对机械磁盘应该是个不错的设置。如果你用的是 SSD，可以考虑提高到 100–200 MB/s。如果你在做批量导入，完全不在意搜索，你可以彻底关掉合并限流。另外还可以增加 index.translog.flush_threshold_size 设置，从默认的 512 MB 到更大一些的值，比如 1 GB，这可以在一次清空触发的时候在事务日志里积累出更大的段。

* 如果你的搜索结果不需要近实时的准确度，考虑把每个索引的index.refresh_interval 改到30s。
* 如果你在做大批量导入，考虑通过设置index.number_of_replicas: 0 关闭副本。

### GC方面，在使用Elasticsearch时要注意什么？

* 倒排词典的索引需要常驻内存，无法GC，需要监控data node上segment memory增长趋势。
* 各类缓存，field cache, filter cache, indexing cache, bulk queue等等，要设置合理的大小，并且要应该根据最坏的情况来看heap是否够用，也就是各类缓存全部占满的时候，还有heap空间可以分配给其他任务吗？避免采用clear cache等“自欺欺人”的方式来释放内存。
* 避免返回大量结果集的搜索与聚合。确实需要大量拉取数据的场景，可以采用scan & scroll api来实现。
* cluster stats驻留内存并无法水平扩展，超大规模集群可以考虑分拆成多个集群通过tribe node连接。
* 想知道heap够不够，必须结合实际应用场景，并对集群的heap使用情况做持续的监控。

### Elasticsearch对于大数据量（上亿量级）的聚合如何实现？

  Elasticsearch 提供的首个近似聚合是cardinality 度量。它提供一个字段的基数，即该字段的distinct或者unique值的数目。它是基于HLL算法的。HLL 会先对我们的输入作哈希运算，然后根据哈希运算的结果中的 bits 做概率估算从而得到基数。其特点是：可配置的精度，用来控制内存的使用（更精确 ＝ 更多内存）；小的数据集精度是非常高的；我们可以通过配置参数，来设置去重需要的固定内存使用量。无论数千还是数十亿的唯一值，内存使用量只与你配置的精确度相关 



### 在并发情况下，Elasticsearch如果保证读写一致？

* 通过版本号使用乐观并发控制
* 另外对于写操作，一致性级别支持quorum/one/all，默认为quorum，即只有当大多数分片可用时才允许写操作。但即使大多数可用，也可能存在因为网络等原因导致写入副本失败，这样该副本被认为故障，分片将会在一个不同的节点上重建。
* 对于读操作，可以设置replication为sync(默认)，这使得操作在主分片和副本分片都完成后才会返回；如果设置replication为async时，也可以通过设置搜索请求参数_preference为primary来查询主分片，确保文档是最新版本。 

### 如何监控 Elasticsearch 集群状态？

elasticsearch-head插件
通过 Kibana 监控 Elasticsearch。你可以实时查看你的集群健康状态和性能，也可以分析过去的集群、索引和节点指标

### 是否了解字典树？

常用字典数据结构如下所示:

![image-20211102162051333](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211102162051333.png)

==字典树又称单词查找树，Trie树，是一种树形结构，是一种哈希树的变种。典型应用是用于统计，排序和保存大量的字符串（但不仅限于字符串），所以经常被搜索引擎系统用于文本词频统计。它的优点是：利用字符串的公共前缀来减少查询时间，最大限度地减少无谓的字符串比较，查询效率比哈希树高。==

* Trie的核心思想是空间换时间，利用字符串的公共前缀来降低查询时间的开销以达到提高效率的目的。它有3个基本性质:
  * 根节点不包含字符，除根节点外每一个节点都只包含一个字符。
  * 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串。
  * 每个节点的所有子节点包含的字符都不相同。
* 对于中文的字典树，每个节点的子节点用一个哈希表存储，这样就不用浪费太大的空间，而且查询速度上可以保留哈希的复杂度O(1)。

### Elasticsearch中的集群、节点、索引、文档、类型是什么？

* 集群是一个或多个节点（服务器）的集合，它们共同保存您的整个数据，并提供跨所有节点的联合索引和搜索功能。群集由唯一名称标识，默认情况下为“elasticsearch”。此名称很重要，因为如果节点设置为按名称加入群集，则该节点只能是群集的一部分。
* 节点是属于集群一部分的单个服务器。它存储数据并参与群集索引和搜索功能。
* 索引就像关系数据库中的“数据库”。它有一个定义多种类型的映射。索引是逻辑名称空间，映射到一个或多个主分片，并且可以有零个或多个副本分片。 MySQL =>数据库 Elasticsearch =>索引
* 文档类似于关系数据库中的一行。不同之处在于索引中的每个文档可以具有不同的结构（字段），但是对于通用字段应该具有相同的数据类型。 MySQL => Databases => Tables => Columns / Rows Elasticsearch => Indices => Types =>具有属性的文档
* 类型是索引的逻辑类别/分区，其语义完全取决于用户。

### Elasticsearch中的倒排索引是什么

ES中的倒排索引其实就是lucene的倒排索引，区别于传统的正向索引，倒排索引会再存储数据时将关键词和数据进行关联，保存到倒排表中，然后查询时，将查询内容进行分词后在倒排表中进行查询，最后匹配数据即可。



## Elastic 8.0版

jdk17

### 安装

![image-20220727174353950](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727174353950.png)

![image-20220727174506498](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727174506498.png)

![image-20220727174551671](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727174551671.png)

![image-20220727174745713](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727174745713.png)

![image-20220727174824367](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727174824367.png)

![image-20220727180418576](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727180418576.png)

![image-20220727180759983](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727180759983.png)

![image-20220727180906870](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727180906870.png)