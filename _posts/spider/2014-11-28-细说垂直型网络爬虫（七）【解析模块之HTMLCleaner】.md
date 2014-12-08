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

##常用的方法
HtmlCleaner和Jsoup很相似，也有很多通过元素特征，例如ID、Class、Attribute等属性获取相应的元素。与Jsoup的select（String cssQuery）
方法不同的是evaluateXPath(String xpath)是通过Xpath去获取元素的，这里主要介绍Xpath。

|路径表达式|表达意义 |
|:-----|:----|
|/|	从根节点选取。|
|//	|从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置。|
|.|	选取当前节点。|
|..	|选取当前节点的父节点。|
|@	|选取属性。|
|body	|选取 body 元素的所有子节点。|
|/body	|选取根元素 body。注释：假如路径起始于正斜杠( / )，则此路径始终代表到某元素的绝对路径！|
|body/div	|选取属于 body 的子元素的所有 div 元素。|
|//div	|选取所有 div 子元素，而不管它们在文档中的位置。|
|body//div|	选择属于 body 元素的后代的所有 div 元素，而不管它们位于 body 之下的什么位置。|
|//@lang|	选取名为 lang 的所有属性。|
|/bookstore/book[1]	|选取属于 bookstore 子元素的第一个 book 元素。|
|/bookstore/book[last()]|	选取属于 bookstore 子元素的最后一个 book 元素。|
|/bookstore/book[last()-1]|	选取属于 bookstore 子元素的倒数第二个 book 元素。|
|/bookstore/book[position()<3]|	选取最前面的两个属于 bookstore 元素的子元素的 book 元素|
|//title[@lang]	|选取所有拥有名为 lang 的属性的 title 元素。|
|//title[@lang='eng']|	选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。|
|/bookstore/book[price>35.00]|	选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。|
|/bookstore/book[price>35.00]/title	|选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。|
|//book/title\|//book/price|	选取 book 元素的所有 title 和 price 元素。|






