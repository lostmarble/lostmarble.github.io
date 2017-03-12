---
layout: post
title: 2016-02-23-Use-Flume-Kafka-ElasticSearch-To-Build-Log-Analysis-System
category: 技术 
tags: Flume, Kafka, ElasticSearch
keywords: 日志分析系统
description: 使用Flume，Kafka,ElasticSearch搭建日志分析系统
---

##下载及运行
1. ElasticSearch2.1.0
https://download.elasticsearch.org/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.1.0/elasticsearch-2.1.0.tar.gz

2. Kibana5.0.0
http://download.elastic.co/kibana/kibana-snapshot/kibana-5.0.0-snapshot-linux-x64.tar.gz
- 配置config/kibana.yml

```
server.port: 5601
elasticsearch.url: "http://xxx:9200"
```

- 运行

```
bin/kibana
```

3. Flume1.6.0
http://www.apache.org/dyn/closer.lua/flume/1.6.0/apache-flume-1.6.0-bin.tar.gz
- 修改254行:
JAVA_OPTS="-Xmx200m"
- 切换到flume目录，运行命令

```
./bin/flume-ng agent -n a1 -c conf/ -f conf/flume-caras.conf -Dflume.root.logger=DEBUG,console&
```

##需求定制
根据业务需要，我们需要定制
1. flume-spooldir-source
由于正常的应用中日志根据时间进行分割，如果source由于某种原因挂掉后，其文件名会被改变，而原始source记录读取的位置包括文件名、文件相对偏移量，当进程重启后会根据这个位置继续向下读，所以我们进程重启时需要将文件名变成最老的日志上。
2. flume-es-sink
- flume-es-sink批量提交日志到es时，使用了事务，如果有一个失败，整个事务回滚，日志放回channel会被再次提交，这样已经提交成功的日志会不断地再次写入es. 我们捕获了提交失败的exception，自动忽略出错的日志。
- ElasticSearch默认的日期格式是YYYY/MM/dd HH:mm:ss,由于我们使用dynamic mapping需要将原始日志的日期格式转换成es的默认格式。











