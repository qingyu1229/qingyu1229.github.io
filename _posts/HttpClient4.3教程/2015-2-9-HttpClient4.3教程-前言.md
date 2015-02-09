---
layout: post
title: HttpClient4.3教程-前言
category: HttpClient4.3教程
date: 2015-2-9
math: true
---

<!-- more -->

本文翻译自：[http://hc.apache.org/httpcomponents-client-4.3.x/tutorial/html/index.html](http://hc.apache.org/httpcomponents-client-4.3.x/tutorial/html/index.html)

##前言
HTTP协议应该是当今互联网上最重要的协议（没有之一），越来越多的Web服务、可联网的智能家具以及其他日益增长的网络计算技术推动着HTTP协议朝着浏览器之外的其他网络技术方向发展。

虽然java.net提供了通过Http协议访问网络资源的基本功能，但是，它满足不了大部分的灵活多变的应用程序。HttpClient试图通过提供一个基于HTTP协议的实时、高效、功能丰富的客户端来填补这一空白。

##HttpClient的定义
1、基于HttpCore的客户端Http传输类库
2、基于传统的（阻塞）IO
3、内容无关

##HttpClient的局限
HttpClient并不是一个浏览器，而只是一个HTTP客户端，用来传输和接收HTTP消息，HttpClient不会去执行Html页面中的JS以及猜测页面类型，如果没有明确的设置，HttpClient也不会去
格式化URL以及跟踪重定向的URL，也不会去处理其他功能无关的HTTP传输。












