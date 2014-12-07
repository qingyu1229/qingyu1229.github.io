---
layout: post
title: 细说垂直型网络爬虫（七）【解析模块之HTMLCleaner】
category: 细说垂直型网络爬虫
date: 2014-11-28

---

##细说垂直型网络爬虫（七）【解析模块之HTMLCleaner】

标签： 网络爬虫 抓取模块 解析 Parser HTMLCleaner

##HTMLcleaner是什么
HtmlCleaner是一个开源的Java语言的Html文档解析器，用户可以提供自定义tag和规则组来进行过滤和匹配。与Jsoup使用Css Query来选
择元素不同，HtmlCleaner通过Xpath来选择元素。

<!-- more -->

maven:
{% highlight xml %}
<dependency>
     <groupId>net.sourceforge.htmlcleaner</groupId>
     <artifactId>htmlcleaner</artifactId>
     <version>2.10</version>
</dependency>
{% endhighlight %}











