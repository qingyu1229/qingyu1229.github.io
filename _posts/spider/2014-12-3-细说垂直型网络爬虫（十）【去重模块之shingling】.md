---
layout: post
title: 细说垂直型网络爬虫（十）【去重模块之shingling】
category: 细说垂直型网络爬虫
date: 2014-12-3

---

##细说垂直型网络爬虫（十）【去重模块之shingling】

标签： 网络爬虫 文本去重

##去重的必要性
虽然爬虫在抓取的过程中对于同样的URL不会重复抓取，但是，不同的网站之间经常互相抄袭和转载，导致同样的信息可能被抓取多次。
据统计，互联网上的重复网页的占比大概在30%-45%之间。

<!-- more -->
 
##shingling算法简介
>
- 借用维基百科对于shingling算法的解释：
In natural language processing a w-shingling is a set of unique "shingles"—contiguous subsequences of tokens in a
document—that can be used to gauge the similarity of two documents. The w denotes the number of tokens in each shingle
in the set.
- 比如，一个文档：
   "a rose is a rose is a rose"
- 分词后的词汇(token，语汇单元)集合是
   (a,rose,is,a,rose,is, a, rose)
- 那么w=4的4-shingling就是集合:
   { (a,rose,is,a), (rose,is,a,rose), (is,a,rose,is), (a,rose,is,a), (rose,is,a,rose) }
- 去掉重复的子集合：
   { (a,rose,is,a), (rose,is,a,rose), (is,a,rose,is) }
- 给定shingle的大小,两个文档A和B的相似度 r 定义为:    
   r(A,B)=|S(A)∩S(B)| / |S(A)∪S(B)|

##代码实现

以后添加









