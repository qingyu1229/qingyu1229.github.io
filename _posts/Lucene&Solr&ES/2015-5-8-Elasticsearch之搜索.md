---
layout: post
title: Elasticsearch之搜索
category: Lucene&Solr&ES
date: 2015-5-8
math: true
---

<!-- more -->
Elastic Search 是一个基于Lucene构建的开源，分布式，RESTful搜索引擎。
设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。
支持通过HTTP使用JSON进行数据索引。
官网：[http://www.elasticsearch.org](http://www.elasticsearch.org)

##一个普通的查询

{% highlight java %}

// 依据查询索引库名称创建查询索引
SearchRequestBuilder searchRequestBuilder = SearchESClientFactory.getClient().prepareSearch(
				SysParameters.INDEXNAME);
//设置查询类型
searchRequestBuilder.setSearchType(SearchType.DFS_QUERY_THEN_FETCH);
//设置分页信息
searchRequestBuilder.setFrom(0).setSize(10);
//创建查询条件
BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
boolQueryBuilder.must(QueryBuilders.termQuery("title", "银行"));
//设置查询条件
searchRequestBuilder.setQuery(boolQueryBuilder);
// 设置是否按查询匹配度排序
searchRequestBuilder.setExplain(true);
// 按照时间降序
searchRequestBuilder.addSort("crawlDate", SortOrder.DESC);
//执行查询
SearchResponse response = searchRequestBuilder.execute().actionGet();
SearchHits searchHits = response.getHits();
System.out.println("总数："+searchHits.getTotalHits());
SearchHit[] hits = searchHits.getHits();
for (SearchHit hit : hits) {
		String json = hit.getSourceAsString();
		ObjectMapper mapper = new ObjectMapper();
		try {
			News news = mapper.readValue(json, News.class);
			System.out.println(news.getTitle()+"	"+hit.getScore()+"  "+news.getCrawlDate());
		} catch (JsonParseException e) {
			e.printStackTrace();
		} catch (JsonMappingException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}
{% endhighlight %}

这是一个去查询新闻标题中含有“银行”的搜索，按照抓取时间倒序去取前十条。

简单介绍下各个搜索设置：

首先需要创建SearchRequestBuilder，由它的对象来执行搜索。
searchRequestBuilder.setSearchType，设置搜索类型，主要的搜索类型有：
  QUERY_THEN_FETCH:查询是针对所有的块执行的，但返回的是足够的信息，而不是文档内容（Document）。结果会被排序和分级，基于此，只有相关的块的文档对象会被返回。由于被取到的仅仅是这些，故而返回的hit的大小正好等于指定的size。这对于有许多块的index来说是很便利的（返回结果不会有重复的，因为块被分组了）
  QUERY_AND_FETCH:最原始（也可能是最快的）实现就是简单的在所有相关的shard上执行检索并返回结果。每个shard返回一定尺寸的结果。由于每个shard已经返回了一定尺寸的hit，这种类型实际上是返回多个shard的一定尺寸的结果给调用者。
  DFS_QUERY_THEN_FETCH：与QUERY_THEN_FETCH相同，预期一个初始的散射相伴用来为更准确的score计算分配了的term频率。
  DFS_QUERY_AND_FETCH:与QUERY_AND_FETCH相同，预期一个初始的散射相伴用来为更准确的score计算分配了的term频率。
  SCAN：在执行了没有进行任何排序的检索时执行浏览。此时将会自动的开始滚动结果集。
  COUNT：只计算结果的数量，也会执行facet。

searchRequestBuilder.setFrom(0).setSize(10)，用来分页
BoolQueryBuilder是一种布尔查询条件，除了布尔查询，ES还有很多其他查询
searchRequestBuilder.setQuery，设置查询
searchRequestBuilder.addSort，设置排序

##依据一个具体需求做出搜索设置

网络爬虫从网络上抓取到了很多新闻，需要建立索引，News类具体属性如下：

{% highlight java %}
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

	private String extraTag;
{% endhighlight %}

依据新闻的这些属性，用户会进行各种不确定属性个数的情况下进行搜索，也就是用户可能按照任何一种属性或者多个属性进行搜索，每个属性有Must、MustNot和Should三种可能情况：

建立一个搜索条件类NewsSearchCondition，具体属性如下：

{% highlight java %}
	/**
	 * 必须被包含在正文中的词语
	 */
	private List<String> mustContentWords;

	/**
	 * 不能被包含在正文中的词语
	 */
	private List<String> mustNotContentWords;

	/**
	 * 可以出现在正文中的词语
	 */
	private List<String> shouldContentWords;

	/**
	 * 标题中必须出现的词语
	 */
	private List<String> mustTitleWords;

	/**
	 * 标题中不能出现的词语
	 */
	private List<String> mustNotTitleWords;

	/**
	 * 标题中可以出现的词语
	 */
	private List<String> shouldTitleWords;

	/**
	 * 必须属于的标签
	 */
	private List<String> mustTagNames;

	/**
	 * 不能被包含的标签
	 */
	private List<String> mustNotTagNames;

	/**
	 * 可以出现的标签
	 */
	private List<String> shouldTagNames;

	/**
	 * 必须的关键词
	 */
	private List<String> mustKeywords;

	/**
	 * 不能出现的关键词
	 */
	private List<String> mustNotKeywords;

	/**
	 * 可以出现的关键词
	 */
	private List<String> shouldKeywords;

	/**
	 * 搜索必须从这些网站来的新闻
	 */
	private List<String> mustWebNames;

	/**
	 * 搜索不能从这些网站来的新闻
	 */
	private List<String> mustNotWebNames;

	/**
	 * 搜索可以从这些网站来的新闻
	 */
	private List<String> shouldWebNames;

	/**
	 * 搜索必须从这些网站来的新闻
	 */
	private List<String> mustSectionNames;

	/**
	 * 搜索不能从这些板块来的新闻
	 */
	private List<String> mustNotSectionNames;

	/**
	 * 搜索可以从这些板块来的新闻
	 */
	private List<String> shouldSectionNames;

	/**
	 * 必须属于的扩展标签
	 */
	private List<String> mustExtraTags;

	/**
	 * 不能属于的扩展标签
	 */
	private List<String> mustNotExtraTags;

	/**
	 * 可以属于的扩展标签
	 */
	private List<String> shouldExtraTags;

	/**
	 * 来源网址
	 */
	private List<Integer> websiteIDs;

{% endhighlight %}

NewsSearchCondition类中对于各个搜索条件的封装方法：

{% highlight java %}

	public BoolQueryBuilder getQueryBuilder(){
		BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
		if(mustTagNames!=null||mustNotTagNames!=null||shouldTagNames!=null){
			BoolQueryBuilder tagNameQueryBuilder = composeQueryBuilder(mustTagNames,mustNotTagNames,shouldTagNames,"tagName");
			boolQueryBuilder.must(tagNameQueryBuilder);
		}
		if(mustContentWords!=null||mustNotContentWords!=null||shouldContentWords!=null){
			BoolQueryBuilder contentQueryBuilder =composeStrQueryBuilder(mustContentWords,mustNotContentWords,shouldContentWords,"content");
			boolQueryBuilder.must(contentQueryBuilder);
		}
		if(mustTitleWords!=null||mustNotTitleWords!=null||shouldTitleWords!=null){
			BoolQueryBuilder titleQueryBuilder =composeStrQueryBuilder(mustTitleWords,mustNotTitleWords,shouldTitleWords,"title");
			boolQueryBuilder.must(titleQueryBuilder);
		}
		if(mustKeywords!=null||mustNotKeywords!=null||shouldKeywords!=null){
			BoolQueryBuilder keywordQueryBuilder = composeQueryBuilder(mustKeywords,mustNotKeywords,shouldKeywords,"keyWords");
			boolQueryBuilder.must(keywordQueryBuilder);
		}
		if(mustWebNames!=null||mustNotWebNames!=null||shouldWebNames!=null){
			BoolQueryBuilder webnameQueryBuilder = composeQueryBuilder(mustWebNames,mustNotWebNames,shouldWebNames,"webName");
			boolQueryBuilder.must(webnameQueryBuilder);
		}
		if(mustSectionNames!=null||mustNotSectionNames!=null||shouldSectionNames!=null){
			BoolQueryBuilder sectionNameQueryBuilder = composeQueryBuilder(mustSectionNames,mustNotSectionNames,shouldSectionNames,"webSectionName");
			boolQueryBuilder.must(sectionNameQueryBuilder);
		}
		if(mustExtraTags!=null||mustNotExtraTags!=null||shouldExtraTags!=null){
			BoolQueryBuilder extraTagQueryBuilder = composeQueryBuilder(mustExtraTags,mustNotExtraTags,shouldExtraTags,"extraTag");
			boolQueryBuilder.must(extraTagQueryBuilder);
		}
		if (websiteIDs!= null) {
			BoolQueryBuilder websiteIDQueryBuilder = composeIntQueryBuilder(null, null, websiteIDs,
					"website_id");
			boolQueryBuilder.must(websiteIDQueryBuilder);
		}
		return boolQueryBuilder;
	}

{% endhighlight %}


这样，只需要设置好NewsSearchCondition类里面的各个搜索属性就可以完成一次搜索条件的拼接。

NewsSearchCondition类中其他方法：

{% highlight java %}

	private BoolQueryBuilder composeQueryBuilder(List<String> mustCondition, List<String> mustNotCondition,
			List<String> shouldCondition, String fieldName) {

		BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
		if (mustCondition != null) {
			BoolQueryBuilder mustTagsBuilder = QueryBuilders.boolQuery();
			for (String c : mustCondition) {
				mustTagsBuilder.must(QueryBuilders.termQuery(fieldName, c));
			}
			boolQueryBuilder.must(mustTagsBuilder);
		}

		if (shouldCondition != null) {
			BoolQueryBuilder shouldTagsBuilder = QueryBuilders.boolQuery();
			for (String c : shouldCondition) {
				shouldTagsBuilder.should(QueryBuilders.termQuery(fieldName, c));
			}
			boolQueryBuilder.must(shouldTagsBuilder);
		}

		if (mustNotCondition != null) {
			BoolQueryBuilder mustNotTagsBuilder = QueryBuilders.boolQuery();
			for (String c : mustNotCondition) {
				mustNotTagsBuilder.mustNot(QueryBuilders.termQuery(fieldName, c));
			}
			boolQueryBuilder.must(mustNotTagsBuilder);
		}

		return boolQueryBuilder;
	}

	private BoolQueryBuilder composeStrQueryBuilder(List<String> mustCondition, List<String> mustNotCondition,
			List<String> shouldCondition, String fieldName) {

		BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
		if (mustCondition != null) {
			BoolQueryBuilder mustTagsBuilder = QueryBuilders.boolQuery();
			for (String c : mustCondition) {
				mustTagsBuilder.must(QueryBuilders.matchPhraseQuery(fieldName, c));
			}
			boolQueryBuilder.must(mustTagsBuilder);
		}

		if (shouldCondition != null) {
			BoolQueryBuilder shouldTagsBuilder = QueryBuilders.boolQuery();
			for (String c : shouldCondition) {
				shouldTagsBuilder.should(QueryBuilders.matchPhraseQuery(fieldName, c));
			}
			boolQueryBuilder.must(shouldTagsBuilder);
		}

		if (mustNotCondition != null) {
			BoolQueryBuilder mustNotTagsBuilder = QueryBuilders.boolQuery();
			for (String c : mustNotCondition) {
				mustNotTagsBuilder.mustNot(QueryBuilders.matchPhraseQuery(fieldName, c));
			}
			boolQueryBuilder.must(mustNotTagsBuilder);
		}

		return boolQueryBuilder;
	}


	private BoolQueryBuilder composeIntQueryBuilder(List<Integer> mustCondition, List<Integer> mustNotCondition,
			List<Integer> shouldCondition, String fieldName) {

		BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
		if (mustCondition != null) {
			BoolQueryBuilder mustTagsBuilder = QueryBuilders.boolQuery();
			for (Integer c : mustCondition) {
				mustTagsBuilder.must(QueryBuilders.termQuery(fieldName, c));
			}
			boolQueryBuilder.must(mustTagsBuilder);
		}

		if (shouldCondition != null) {
			BoolQueryBuilder shouldTagsBuilder = QueryBuilders.boolQuery();
			for (Integer c : shouldCondition) {
				shouldTagsBuilder.should(QueryBuilders.termQuery(fieldName, c));
			}
			boolQueryBuilder.must(shouldTagsBuilder);
		}

		if (mustNotCondition != null) {
			BoolQueryBuilder mustNotTagsBuilder = QueryBuilders.boolQuery();
			for (Integer c : mustNotCondition) {
				mustNotTagsBuilder.mustNot(QueryBuilders.termQuery(fieldName, c));
			}
			boolQueryBuilder.must(mustNotTagsBuilder);
		}
		return boolQueryBuilder;
	}


{% endhighlight %}