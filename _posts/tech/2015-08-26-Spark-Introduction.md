---
lgyout: post
title: 2015-08-26-Spark-Introduction.md
category: 技术 
tags: flume
keywords: flume
description: 
---

RDD(Resilent Distributed Dataset)是Spark中最基本的数据抽象，代表一个元素中不可变的、分块的元素的集合，这个集合中的元素可以被平行操作。这个类包含了在所有RDD上最基本的操作，比如map,filter和persist。在内部rdd有5大特征：
1.partitions的列表
2.计算每个分片的函数
3.依赖其它rdd的列表
4.一个Partitioner for key-value RDD (可选)
5.一个计算每个分片的位置(如，HDFS文件的块地址)(可选)
Spark中所有的规划与执行都是基于这些方法，允许每个rdd完成自己的实现方式。用户可以覆盖这些函数来实现个性化rdd.

rdd主要分为两种，Tansformations和Actions.
Transformations包含:map,filter,flatMap等函数。
Actions包含：count，collect，reduce等函数。

transformations都是lazy execution的，需要具体的action去触发，每个action操作都是一个单独的job。




