---
layout: post
title: HttpClient4.3教程-请求的执行
category: HttpClient4.3教程
date: 2015-2-10
math: true
---

<!-- more -->

本文翻译自：[http://hc.apache.org/httpcomponents-client-4.3.x/tutorial/html/index.html](http://hc.apache.org/httpcomponents-client-4.3.x/tutorial/html/index.html)

HttpClient最基本的功能就是执行Http方法。一个Http方法的执行涉及到一个或者多个Http请求/Http响应的交互，通常这个过程都会自动被HttpClient处理，对用户透明。用户只需要提供Http请求对象，
HttpClient就会将http请求发送给目标服务器，并且接收服务器的响应，如果http请求执行不成功，httpclient就会抛出异常。
下面是个很简单的http请求执行的例子：
{% highlight java %}
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    <...>
} finally {
    response.close();
}
{% endhighlight %}

##1.1.1.HTTP request
所有HTTP请求都包含有一个方法名、请求URI和HTTP协议版本。HttpClient支持的所有HTTP / 1.1规范中定义的方法： GET, HEAD, POST, PUT, DELETE, TRACE 和 OPTIONS，与之相对应的类：
HttpGet, HttpHead, HttpPost, HttpPut, HttpDelete, HttpTrace, 以及 HttpOptions。
请求的URI包含的信息有：网络协议、主机名称、端口、资源路径,请求参数以及其他非必须的参数。
{% highlight java %}
HttpGet httpget = new HttpGet(
     "http://www.google.com/search?hl=en&q=httpclient&btnG=Google+Search&aq=f&oq=");
{% endhighlight %}
HttpClient中的URIBuilder类提供了创建和修改URI的功能。
{% highlight java %}
URI uri = new URIBuilder()
        .setScheme("http")
        .setHost("www.google.com")
        .setPath("/search")
        .setParameter("q", "httpclient")
        .setParameter("btnG", "Google Search")
        .setParameter("aq", "f")
        .setParameter("oq", "")
        .build();
HttpGet httpget = new HttpGet(uri);
System.out.println(httpget.getURI());
//输出 ：http://www.google.com/search?q=httpclient&btnG=Google+Search&aq=f&oq=
{% endhighlight %}

##1.1.2. HTTP response
HTTP response是服务器端在接收到HTTP请求后返回给客户端的消息。HTTP response的第一行信息包含HTTP协议的版本、返回在状态码以及对应的文本短语。
{% highlight java %}
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1,
HttpStatus.SC_OK, "OK");

System.out.println(response.getProtocolVersion());
System.out.println(response.getStatusLine().getStatusCode());
System.out.println(response.getStatusLine().getReasonPhrase());
System.out.println(response.getStatusLine().toString());
//输出 ：HTTP/1.1
      200
      OK
      HTTP/1.1 200 OK
{% endhighlight %}

##1.1.3. 消息头（Headers）
一个HTTP消息可以包含多个用来描述消息内容长度和类型的信息。HttpClient 提供了对Headers进行搜索、添加、移除以及枚举的方法。
{% highlight java %}
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1,
    HttpStatus.SC_OK, "OK");
response.addHeader("Set-Cookie",
    "c1=a; path=/; domain=localhost");
response.addHeader("Set-Cookie",
    "c2=b; path=\"/\", c3=c; domain=\"localhost\"");
Header h1 = response.getFirstHeader("Set-Cookie");
System.out.println(h1);
Header h2 = response.getLastHeader("Set-Cookie");
System.out.println(h2);
Header[] hs = response.getHeaders("Set-Cookie");
System.out.println(hs.length);
//输出：
//Set-Cookie: c1=a; path=/; domain=localhost
//Set-Cookie: c2=b; path="/", c3=c; domain="localhost"
//2
{% endhighlight %}
最有效的方式来获得给定类型的所有头文件是使用HeaderIterator接口
{% highlight java %}
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1,
    HttpStatus.SC_OK, "OK");
response.addHeader("Set-Cookie",
    "c1=a; path=/; domain=localhost");
response.addHeader("Set-Cookie",
    "c2=b; path=\"/\", c3=c; domain=\"localhost\"");

HeaderIterator it = response.headerIterator("Set-Cookie");

while (it.hasNext()) {
    System.out.println(it.next());
}
//输出：
//Set-Cookie: c1=a; path=/; domain=localhost
//Set-Cookie: c2=b; path="/", c3=c; domain="localhost"
{% endhighlight %}

同时HTTPClient还提供了简单的方法来解析HTTP消息为单独的头元素。
{% highlight java %}
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1,
    HttpStatus.SC_OK, "OK");
response.addHeader("Set-Cookie",
    "c1=a; path=/; domain=localhost");
response.addHeader("Set-Cookie",
    "c2=b; path=\"/\", c3=c; domain=\"localhost\"");

HeaderElementIterator it = new BasicHeaderElementIterator(
    response.headerIterator("Set-Cookie"));

while (it.hasNext()) {
    HeaderElement elem = it.nextElement();
    System.out.println(elem.getName() + " = " + elem.getValue());
    NameValuePair[] params = elem.getParameters();
    for (int i = 0; i < params.length; i++) {
        System.out.println(" " + params[i]);
    }
}
//输出
//c1 = a
// path=/
// domain=localhost
//  c2 = b
//  path=/
//  c3 = c
//  domain=localhost
{% endhighlight %}

##1.1.4. HTTP实体
Http消息可以携带http实体，这个http实体既可以是附加在HTTP请求上的，也可以是http响应后返回的。Http实体，可以在某些http请求或者响应中发现，但不是必须的。Http规范中定义了两种包含请求的方法：POST和PUT。HTTP响应一般会包含一个内容实体。当然这条规则也有异常情况，如Head方法的响应，204没有内容，304没有修改或者205内容资源重置。
HttpClient根据来源的不同，划分了三种不同的Http实体内容。
•	streamed: Http内容是通过流来接受或者generated on the fly。特别是，streamed这一类包含从http响应中获取的实体内容。一般说来，streamed实体是不可重复的。
•	self-contained: 存在于内存中或者与其他的实体相关。self-contained类型的实体内容通常是可重复的。这种类型的实体通常用于关闭http请求。
•	wrapping: 这种类型的内容是从另外的http实体中获取的。
当从Http响应中读取内容时，上面的三种区分对于连接管理器来说是非常重要的。请求类的实体通常由应用程序创建，由HttpClient发送给服务器，在请求类的实体中，streamed和self-contained两种类型的区别就不重要了。在这种情况下，一般认为不可重复的实体是streamed类型，可重复的实体时self-contained。

##1.1.4.1. 可重复的实体
一个实体是可重复的，也就是说它的包含的内容可以被多次读取。这种多次读取只有self contained（自包含）的实体能做到（比如ByteArrayEntity或者StringEntity)。






