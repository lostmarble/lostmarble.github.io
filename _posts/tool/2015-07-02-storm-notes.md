---
layout: post
title: Storm学习笔记
category: 技术 
keywords: storm,0.9.5
---

## 安装

运行storm-0.9.5版本的example\storm-starter时
```
mvn compile exec:java -Dstorm.topology=storm.starter.WordCountTopology
```
出错

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further detail s.
```

应该是版本的总题在网上辗转找到一个解决方法,在POM.XML添加新版本的依赖：
```
    <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.5</version>
    </dependency>

    <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.6.4</version>
    </dependency>

    <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.5</version>
    </dependency>
```

## 参考
http://stackoverflow.com/questions/7421612/slf4j-failed-to-load-class-org-slf4j-impl-staticloggerbinder



