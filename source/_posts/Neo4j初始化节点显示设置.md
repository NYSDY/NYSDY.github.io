---
title: Neo4j初始化节点显示设置
date: 2018-10-10 16:13:05
tags:
- Neo4j
- 数据库
english_title: Neo4j_initializes_the_node_display_Settings
categories: Neo4j
---

### 问题描述：

neo4j中有默认的初始化节点显示设置为300个节点，如果想要显示的节点多于300个，则会只显示300个，并给予以下提示语句：

`Not all return nodes are being displayed due to Initial Node Display setting. Only 300 of 300 nodes are being displayed.`

### 解决方法：

在如图所示`initial Node Display`处可以修改，在此处修改为300000.![](https://i.loli.net/2018/10/10/5bbdb388a368b.jpg)

