+++
title = "向量时钟(Vector Clock)"
date = "2019-11-08"
author = "x1ah"
cover = ""
tags = ["Distributed"]
keywords = ["Distributed"]
description = "向量时钟(Vector Clock) 是分布式系统中检测事件因果关系的一种算法, 通常在分布式系统中用来检测多个 replication 之间是否发生数据冲突。"
showFullContent = false
+++

## 向量时钟(Vector Clock)

向量时钟是在分布式系统中检测事件因果关系的一种算法。如图：![vector clock](https://en.wikipedia.org/wiki/Vector_clock#/media/File:Vector_Clock.svg)

系统中有 ABC 三个进程，每个进程都维护自己的一个向量时钟，时钟的规则如下：
1. 初始时，所有进程的时钟都为 0
2. 进程每次处理一个内部事件，其逻辑时钟加 1
3. 每次发送消息，其逻辑时钟加 1，并且将其向量时钟一起发送
4. 每次收到消息，其逻辑时钟加 1，并更新本地时钟，逻辑时钟的值为本地时钟里值的最大值

每个进程维护的所有逻辑时钟为一个向量时钟。假设进程 A 向量时钟如下：

```python
+----+
|A:0 |
|B:3 |  ===>  这个整体称为 A 的 "向量时钟"，其中，A:0 为 A 的逻辑时钟
|C:5 |
+----+
```

### 因果关系判断规则

1. 如果时钟 V1 的每个逻辑时钟值都比时钟 V2  大，那么称 V1 比 V2 先发生。如： `V1: [A:2,B:4,C:2]` 与 `V2: [A:1,B:2,C:1]`
2. 如果不满足条件 1), 即有的值 V1 比 V2 大，有的 V2 比 V1 大，那么看做两个事件同时发生

## 应用

向量时钟通常用于检测 replication 之间的数据冲突。例如 Dynamo: [Data Versioning With DynamoDB](https://cloudacademy.com/blog/data-versioning-with-dynamodb-an-inside-look-into-nosql-part-5/)。

