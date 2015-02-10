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







