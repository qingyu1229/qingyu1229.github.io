---
layout: post
title: Elasticsearch之Client
category: Lucene&Solr&ES
date: 2015-5-5
math: true
---

<!-- more -->
Elastic Search 是一个基于Lucene构建的开源，分布式，RESTful搜索引擎。
设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。
支持通过HTTP使用JSON进行数据索引。
官网：[http://www.elasticsearch.org](http://www.elasticsearch.org)

##一、方法一,NodeClient

{% highlight java %}
import static org.elasticsearch.node.NodeBuilder.*;

Node node = nodeBuilder().node();
//Node node = nodeBuilder().clusterName("yourclustername").node();
Client client = node.client();

/*
*处理数据
*/
// 关闭
node.close();

{% endhighlight %}

##二、方法二，TransportClient

{% highlight java %}
Settings settings = ImmutableSettings.settingsBuilder().put("cluster.name", "myClusterName").build();
Client client = new TransportClient(settings)
        .addTransportAddress(new InetSocketTransportAddress("host1", 9300))
        .addTransportAddress(new InetSocketTransportAddress("host2", 9300));

/*
*处理数据
*/

// 关闭
client.close();
{% endhighlight %}