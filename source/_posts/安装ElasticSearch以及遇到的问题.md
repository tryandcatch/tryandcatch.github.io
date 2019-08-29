---
title: 安装运行ElasticSearch以及遇到的问题
tags: 
 - ElastiSearch
 - problems
urlname: install-elastic-search
date: 2019-08-24 18:08:49
keywords:
 - ElasticSearch
description:
 - 记录在Mac上安装ElasticSearch 7.3.0，运行遇到的问题以及解决办法
---

最近在极客时间购买了[《Elasticsearch核心技术与实战》](<https://time.geekbang.org/course/intro/197?code=oZh6Y5%2F8sex7eCaBwcQ2AEJVC%2Fy4E%2FDF6JxlSUPcJ-k%3D&utm_term=SPoster>) (链接中已带code)，依照视频学着一步一步来操作。

## 安装并运行ElasticSearch

安装比较简单，从[官网下载](<https://www.elastic.co/cn/downloads/elasticsearch>)对应操作系统的安装包/压缩包，我这下载的是目前的最新版 7.3.0（然鹅在我这个文章的时候我发现，最新版已经是7.3.1了...）然后解压，解压后在安装目录下执行`bin/elasticsearch`就能单节点运行ElasticSearch了，能正常访问 `http://localhost:9200`就证明安装完成。

单节点运行没问题，但是在实验多节点运行的时候，达不到预期的效果，访问`_cat/nodes`并没有返回多个节点的信息：

![image.png](https://i.loli.net/2019/08/25/zW9DVgk57XJtcRE.png)

多节点启动ElasticSearch：（`-d`表示后台启动，后台启动方式，结束ElasticSearch：ps | grep elasticsearch，kill pid）

```shell
bin/elasticsearch -E node.name=node0 -E cluster.name=hxt -E path.data=node0_data -d
bin/elasticsearch -E node.name=node1 -E cluster.name=hxt -E path.data=node1_data -d
bin/elasticsearch -E node.name=node2 -E cluster.name=hxt -E path.data=node2_data -d
bin/elasticsearch -E node.name=node3 -E cluster.name=hxt -E path.data=node3_data -d
```



后来查看安装目录发现多了这些目录：

![filepath.jpg](https://i.loli.net/2019/08/24/45Wom6J9z3Nnij2.jpg)

因此应该是启动的时候配置参数`path.data`配置的不对，因此将参数修改为`path.data=data/xxx`,修改后命令如下:

```shell
bin/elasticsearch -Enode.name=node0 -Ecluster.name=hxt -Epath.data=data/node0_data -d
bin/elasticsearch -Enode.name=node1 -Ecluster.name=hxt -Epath.data=data/node1_data -d
bin/elasticsearch -Enode.name=node2 -Ecluster.name=hxt -Epath.data=data/node2_data -d
bin/elasticsearch -Enode.name=node3 -Ecluster.name=hxt -Epath.data=data/node3_data -d
```

这样访问`_cat/nodes`就能看到多个节点了。

---

参考文章：

[path.data and path.logs](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/path-settings.html)

[Node data path settings](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/modules-node.html#_node_data_path_settings)