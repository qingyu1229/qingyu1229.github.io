---
layout: post
title: Elasticsearch之建立索引
category: Lucene&Solr&ES
date: 2015-5-8
math: true
---

<!-- more -->
Elastic Search 是一个基于Lucene构建的开源，分布式，RESTful搜索引擎。
设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。
支持通过HTTP使用JSON进行数据索引。
官网：[http://www.elasticsearch.org](http://www.elasticsearch.org)

##设置Mapping
使用ES创建索引时，ES能够自动识别被索引对象的属性类型，但是通常还是需要制定其他分词方式等。
下面介绍使用配置文件设置Mapping：
在ES目录下的config目录下新建mappings目录，新建一个以索引类型为名字的Json格式的文件，放置在/mappings/索引名称/下。
示例：news.json
{% highlight html %}
{
"news":{
"properties": {
"content": {
"type": "string"
},
"webSectionName": {
"type": "string"
},
"title": {
"type": "string"
},
"p_url": {
"type": "string",
"index": "not_analyzed"
},
"tagName": {
"type": "string",
"index_analyzer": "whitespace"
},
"keyWords": {
"type": "string",
"index_analyzer": "whitespace"
},
"isTask": {
"type": "long"
},
"newsDate": {
"type": "long"
},
"crawlDate": {
"type": "long"
},
"webName": {
"type": "string"
},
"crawl_id": {
"type": "long"
},
"url": {
"type": "string",
"index": "not_analyzed"
},
"website_id": {
"type": "string",
"index_analyzer": "whitespace"
}
}
}
}

{% endhighlight %}

##建立索引

{% highlight java %}

public class News {

	/**
	 * 新闻ID
	 */
	private int crawl_id;
	/**
	 * 新闻URL
	 */
	private String url;
	/**
	 * 新闻标题
	 */
	private String title;
	/**
	 * 新闻内容
	 */
	private String content;
	/**
	 * 新闻时间
	 */
	private Long newsDate;
	/**
	 * 新闻抓取时间
	 */
	private Long crawlDate;
	/**
	 * 新闻来源网站名称
	 */
	private String webName;
	/**
	 * 新闻来源板块名称
	 */
	private String webSectionName;
	/**
	 * 新闻来源对应的Website_id
	 */
	private String website_id;

	/**
	 * 新闻来源（信息源）的URL
	 */
	private String p_url;
	/**
	 * 新闻所属标签名称
	 */
	private String tagName;
	/**
	 * 新闻关键词
	 */
	private String keyWords;

	//getter setter

		public String toJsonString(){
    		ObjectMapper mapper = new ObjectMapper();
    		try {
    			String json = mapper.writeValueAsString(this);
    			return json;
    		} catch (JsonProcessingException e) {
    			e.printStackTrace();
    		}
    		return null;
    	}
}
{% endhighlight %}

{% highlight java %}
	/**
	 * 将News对象建立索引
	 * @param news News对象
	 * @param indexName Index名称
	 * @return
	 */
	public boolean index(News news,String indexName){
		/**
		 * 使用文档ID作为索引中的ID
		 */
		IndexRequestBuilder indexRequestBuilder =IndexESClientFactory.getClient().prepareIndex(indexName, "news", String.valueOf(news.getCrawl_id()));
		indexRequestBuilder.setSource(news.toJsonString());
		logger.info("开始建立索引,crawl_id:"+news.getCrawl_id());
		try {
			 indexRequestBuilder.execute().actionGet();
		} catch (ElasticsearchException e) {
			logger.error(e.getDetailedMessage());
			return false;
		}
		return true;
	}

{% endhighlight %}