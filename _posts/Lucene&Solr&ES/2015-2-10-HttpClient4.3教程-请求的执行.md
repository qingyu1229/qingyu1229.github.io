---
layout: post
title: Elasticsearch简单入门
category: Lucene&Solr&ES
date: 2015-4-28
math: true
---

<!-- more -->
Elastic Search 是一个基于Lucene构建的开源，分布式，RESTful搜索引擎。
设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。
支持通过HTTP使用JSON进行数据索引。
官网：[http://www.elasticsearch.org](http://www.elasticsearch.org)

##一、下载安装Elasticsearch
从官网（http://www.elasticsearch.org）下载最新的Elasticsearch版本（1.5.1）

##二、安装插件
head插件：
使用命令行定位到bin目录下，使用命令：plugin -install mobz/elasticsearch-head安装Head插件，安装成功后启动Elasticsearc，
在浏览器中输入：http://localhost:9200/_plugin/head/ 可以看见如下界面
![Elasticsearch的Head插件](/res/img/blogimg/2015/ES/ES_head.png)

中文分词器插件：
选用的是IK分词器，在https://github.com/medcl/elasticsearch-analysis-ik上有专门为Elasticsearch做好了封装的IK分词器，但是并没有针对最新版本的，如果使用旧版本的IK插件将会报错
需要将源代码down下来，修改pom.xml中的Elasticsearch的版本信息后重新打包。
1、在ES安装目录下的plugins下面新建analysis-ik目录，将打包好的elasticsearch-analysis-ik-1.5.1.jar拷贝到这个目录下，然后再将HTTPClient和Httpcore两个jar拷贝到此。
2、将打包后的Jar文件拷贝到ES安装目录的lib目录下
3、将elasticsearch-analysis-ik项目下的config/ik文件夹复制到ES安装目录config文件夹下
在ES的配置文件config/elasticsearch.yml中增加ik的配置，如下：

{% highlight html %}
index:
  analysis:                   
    analyzer:
      ik:
          alias: [ik_analyzer]
          type: org.elasticsearch.index.analysis.IkAnalyzerProvider
      ik_max_word:
          type: ik
          use_smart: false
      ik_smart:
          type: ik
          use_smart: true

index.analysis.analyzer.default.type : "ik"

{% endhighlight %}

4、重启ES，在上一步的Head插件中建立一个索引（如果还没有建立的话），点击动作-》测试分析器

![Elasticsearch的IK插件](/res/img/blogimg/2015/ES/es_ik_1.png)

输入“今天天气不错”得到如下结果说明IK插件安装成功：

![Elasticsearch的IK插件](/res/img/blogimg/2015/ES/es_ik_2.png)
