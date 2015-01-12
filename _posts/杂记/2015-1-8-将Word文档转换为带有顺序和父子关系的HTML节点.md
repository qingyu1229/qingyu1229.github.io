---
layout : post
title: 将Word文档转换为带有顺序和父子关系的HTML节点
category : 杂记
duoshuo: true
math: true
date : 2015-1-8
---

<!-- more -->

##需求来源
前不久遇到一个项目，需要将Word文档结构化为一个可以在富文本编辑器中进行编辑的HTML文档，而且要求按照Word文档目录中的先后顺序以及父子节点关系将整个文档结构化若干个HTML节点。
这个需求是整个项目中的一个难点，查阅了很多开源项目、开源工具、收费的工具或服务之后发现并没有一个能够直接满足我们需求的。最后决定自己开发这个功能。
下面简单记录一下这个需求的实现过程。
（Word为office2007版本，此方法不适用2003版本）

##技术选型
Jacob+Jsoup:
Jacob:将word文档转换为HTML；
Jsoup:从HTML文档中对信息进行结构化。

##主要的实现思路
1、使用Jacob从Wrod文档中生成目录（如果原来存在目录，删除原来的目录重新抽取目录），并返回目录结构wordToc。
2、使用Jacob将Word文档另存为HTML文档，此时在HTML文档中的目录为一系列的A标签htmlToc，通过比较目录的具体标题就可以将wordToc和htmlToc一一对应上。
3、在HTML文档中点击目录中的A标签就可以跳转到页面中的对应位置，于是wordToc就和具体的HTML文档片段对应上了，每两个目录A标签的对应位置之间的HTML片段就是我们需要的内容。
例如下下图中，要获取节点“海口出促楼市发展政策：限价房申购门槛由35岁降到25岁”对应的HTML片段：
![word目录](/res/img/blogimg/2015/01/20150112-3.png)
转换为HTML后，目录中节点“海口出促楼市发展政策：限价房申购门槛由35岁降到25岁”的href属性为一个以#_Toc开头的字符串，“href="#_Toc404261462"”：
![HTML目录](/res/img/blogimg/2015/01/20150112-1.png)
在下面的HTML文档中，节点“海口出促楼市发展政策：限价房申购门槛由35岁降到25岁”上有一个对应的A标签：<a name="_Toc404261462"></a>，下一个节点“广州市房协喊话：南沙增城等楼市尽快放开限购”也对应了一个这样的A标签，
这两个连续的A标签之间的HTML片段就是节点“海口出促楼市发展政策：限价房申购门槛由35岁降到25岁”对应的HTML片段：
![HTML片段](/res/img/blogimg/2015/01/20150112-2.png)
4、使用Jsoup依据以上的处理逻辑，按照目录结构解析出HTML文档中需要的HTML片段，然后转换为Java对象的集合。

##主要代码实现
**打开Word文档：**
{% highlight java %}
     // word文档
      private Dispatch doc;
     // word运行程序对象
      private ActiveXComponent word;
     // 所有word文档集合
     private Dispatch documents;
     // 选定的范围或插入点
     private Dispatch selection;
     private boolean saveOnExit = true;

   public void openDocument(String docPath) {
                closeDocument();
                doc = Dispatch.call(documents, "Open", docPath).toDispatch();
                selection = Dispatch.get(word, "Selection").toDispatch();
        }
{% endhighlight %}

**生成目录：**
{% highlight java %}

    private void extractToc() {
        Dispatch tocs = Dispatch.get(doc, "TablesOfContents").toDispatch();
        int tocCount = Dispatch.get(tocs,"Count").getInt();
        //System.out.println("toc count is "+ tocCount);
        Dispatch toc = Dispatch.call(tocs, "Item", new Variant(tocCount)).toDispatch();
        Dispatch.call(toc, "Delete");
        toc = Dispatch.call(tocs,"Add",
        		Dispatch.call(doc,"Range",new Variant(0),new Variant(0)).toDispatch(),   //目录出现的区域。如果该区域未折叠，目录将替换该区域。
        		new Variant(true),   //如果为 True，则使用内置标题样式创建目录。默认值为 True。
        		new Variant(1),      //目录的起始标题级别。对应于使用 \o 开关的目录 (TOC) 域的起始值。默认值为 1。
        		new Variant(3),      //目录的结束标题级别。对应于使用 \o 开关的目录 (TOC) 域的结束值。默认值为 9。
        		new Variant(true),   //如果目录项 (TC) 域用于创建目录，则为 True。可用 MarkEntry 方法标记要包括在目录中的各项。默认值为 False。
        		new Variant("A"),    //用于从 TC 域创建目录的单字母标识符。对应于目录 (TOC) 域中的 \f 开关。例如，“T”表示使用表格标识符 T 从 TC 域建立目录。如果省略此参数，则不使用 TC 域。
        		new Variant(true),   //如果目录中的页码与右边距对齐，则为 True。默认值为 True。
        		new Variant(true),   //如果为 True，则在目录中包含页码。默认值为 True。
        		new Variant(""),     //用来编译目录的其他样式的字符串名称（标题 1 – 标题 9 样式之外的样式）。可用 HeadingStyles 对象的 Add 方法创建新的标题样式。
        		new Variant(true),   //如果在文档被发布到网站上时，目录中的各项应转为超链接格式，则为 True。默认值为 True。
        		new Variant(true),   //如果在文档被发布到网站上时，目录中的页码应隐藏，则为 True。默认值为 True。
        		new Variant(true)    //如果为 True，则使用大纲级别创建目录。默认值为 False。
         ).toDispatch();
        System.out.println(Dispatch.get(toc,"UseHyperlinks"));
        Dispatch.put(toc,"UseHyperlinks",new Variant(true));
        System.out.println(Dispatch.get(toc,"UseHyperlinks"));
       // Dispatch.put(toc,"UseOutlineLevels",new Variant(true));
        Dispatch.call(toc, "Update");
        Dispatch range = Dispatch.get(toc,"Range").toDispatch();
        Dispatch links = Dispatch.get(range,"Hyperlinks").toDispatch();
        int count = Dispatch.get(links,"Count").getInt();
        TocNode root = new TocNode("");
        TocNode last = root;
        for(int i = 1; i <= count;++i) {
            Dispatch link = Dispatch.call(links,"Item", i).toDispatch();
            Dispatch linkRange = Dispatch.get(link,"Range").toDispatch();
            String name = Dispatch.get(linkRange,"Text").toString();
            name = extractTitleFromTocItem(name);
            //System.out.println(name);
            Dispatch paragraphFormat = Dispatch.get(linkRange,"ParagraphFormat").toDispatch();
            double indent = caculateIndent(paragraphFormat);
            //System.out.println("indent is "+ indent);
            TocNode father = last.findFather(indent);
            TocNode current = new TocNode(name,indent);
            father.addChild(current);
            last = current;
        }

        printTocTree(root, 0);
    }


    private double caculateIndent(Dispatch paragraph) {
        Variant firstIndent = Dispatch.get(paragraph, "FirstLineIndent");
        String indentStr= firstIndent.toString();
        double indent1 = Double.parseDouble(indentStr);

        Variant leftIndent = Dispatch.get(paragraph, "LeftIndent");
        indentStr= leftIndent.toString();
        double indent = Double.parseDouble(indentStr);
        return indent1+indent;
    }
{% endhighlight %}

TocNode的Bean结构：
{% highlight java %}
public class TocNode {

    private String name;

    private ArrayList<TocNode> children = new ArrayList<TocNode>();

    private TocNode father = null;

    private double indent = -1;

    private String hrefStr;

    private String text;
    //……
    //geter  seter
}
{% endhighlight %}

**使用Jsoup抽取内容：**
{% highlight java %}

public class TestToc {

	int order=0;
	int index=0;
	int p_id=-1;

	List<CatalogNode> list=new ArrayList<CatalogNode>();
	List<String> nodetextList=new ArrayList<String>();
	Map<String,TocNode> tocNodeMap=new HashMap<String,TocNode>();

	@Test
	public void test() {
		  	DocMain app = new DocMain(false);
	        app.setSaveOnExit(false);
	        app.openDocument("E:/givelqy/目录和内容不统一/1416469292074.doc");
	        TocNode root= app.extractToc();
	        app.saveAs("E:/上传问题文档/tmp-5.html");

	        Document  document=getDocument("E:/上传问题文档/tmp-5.html","gb2312");
	       // printTocTree(root,0);
	        fentch(root,document);
	        app.close();
	        //System.out.println(list);
	        System.out.println("ok");

	}

	public void fentch(TocNode root,Document doc){
		readNode(root,0);
		Elements tocElements= doc.body().getElementsByAttributeValueStarting("class",
				"MsoToc");
		Map<String,String> tempMap=new HashMap<String,String>();
		//获取目录中 每个标题需要跳转到的href
		for(int i=0;i<tocElements.size();i++){
			Elements a_elements = tocElements.get(i).getElementsByTag("a");

			for(Element e:a_elements){

				String tocText=cleanoutNodeValue(e.text());
				String href=e.attr("href");

				//目录处的连接都以#_Toc开头
				if(tocNodeMap.containsKey(tocText)&&href.startsWith("#_Toc")){
					//保存跳转href
					String c_href= href.replace("#", "");
				//	System.out.println("=="+c_href);
					tocNodeMap.get(tocText).setHrefStr(c_href);
					tempMap.put(c_href, tocText);
					break;
				}
			}

		}
		System.out.println(tempMap);
		Elements tempElements=new Elements();
		String lastNodeKey="";
		boolean isCatalogNode=false;
		int counter=0;//目录节点个数记录器

		//遍历整个文档，每个节点对应的内容
		Elements AllContentElements=doc.select("body > div > p,body > div > div,body > div > table,body > div > h1,body > div > h2,body > div > h3,body > div > b,body > div > span");
		for(int i=0;i<AllContentElements.size();i++){
			Element contentElement=AllContentElements.get(i);
			Elements aElements=contentElement.getElementsByTag("a");

			for(Element ae:aElements){
				String aName=ae.attr("name");
				if(aName==null){
					continue;
				}
				//此标签是一个目录节点
				if(tempMap.containsKey(aName)){

					System.out.println("aName:"+aName);
					//当找到第一个节点的时候，将第一个节点之前的所有Elements内容删除
					if(counter==0){
						tempElements.empty();
						tempElements.clear();
					}
					counter++;
					if(i!=0){
						if(aName.equals("_Toc389577860")){
							System.out.println("Str:"+tempMap.get(aName)+"==="+aName+lastNodeKey);
						}

						tocNodeMap.get(lastNodeKey).setText(tempElements.toString());
						//System.out.println("nodeText:"+lastNodeKey+"\n "+"text:"+tempElements.toString()+"\n ====================");
					}
					lastNodeKey=tempMap.get(aName);

					tempElements.empty();
					tempElements.clear();
					isCatalogNode=true;
				}
			}
			if(isCatalogNode){
				isCatalogNode=false;
				continue;
			}
			tempElements.add(contentElement);

			if(i==AllContentElements.size()-1){
				tocNodeMap.get(lastNodeKey).setText(tempElements.toString());
			}
		}

		//System.out.println(tocNodeMap);

		Iterator<TocNode> iterator= tocNodeMap.values().iterator();
		while(iterator.hasNext()){
			TocNode tn=iterator.next();

			System.out.println(tn+"\n ==============================");
		}

	}

	public void readNode(TocNode node,int level){
		String tmplate = "name:%s\t";
    	String nameStr = String.format(tmplate, node.getName());
    	boolean hasChild=true;
    	if(node.getChildren().size()==0){
    		hasChild=false;
    	}
    	tocNodeMap.put(node.getName().trim(), node);
    	System.out.println("level:"+level+"\t"+"hasChild:"+hasChild+"\t"+"indent:"+node.getIndent()+"\t"+nameStr);
    	for(TocNode child : node.getChildren()) {
    		readNode(child,level+1);
    	}
	}



    public void printTocTree(TocNode node,int level) {
    	String tmplate = "name:%s\t";
    	String nameStr = String.format(tmplate, node.getName());
    	boolean hasChild=true;
    	if(node.getChildren().size()==0){
    		hasChild=false;
    	}
    	nodetextList.add(node.getName());
    	CatalogNode cn=new CatalogNode();
    	cn.setId(index++);
    	cn.setP_id(p_id);
    	cn.setContent(node.getName());
    	list.add(cn);
    	if(hasChild){
    		p_id=index;
    	}
    	System.out.println("level:"+level+"\t"+"hasChild:"+hasChild+"\t"+"order:"+order+"\t"+"indent:"+node.getIndent()+"\t"+nameStr);
    	for(TocNode child : node.getChildren()) {
    		printTocTree(child,level+1);
    	}
    }

    /**
	 * 依据文件路径和编码将文件封装成DOCUMENT
	 * @param filePath
	 * @param encoding
	 * @return
	 */
	public  Document getDocument(String filePath,String encoding){
		String fileString="";
		try {
			fileString = FileUtils.readFileToString(new File(
					filePath), encoding);
		} catch (IOException e) {
			return null;
		}
		Document doc = Jsoup.parse(fileString);
		modifyImageSrc(doc);
		doc.getElementsByAttributeValueContaining("style", "display:none").remove();
		return doc;
	}
	/**
	 * 修改图片的路径，将本地路径改为网络路径
	 * @param document
	 */
	protected void modifyImageSrc(Document document){
		if(document==null){
			return;
		}
		Elements elements=document.getElementsByTag("img");
		if(elements==null){
			return;
		}
		for(Element element:elements){
			String srcString=element.attr("src");
			boolean bl= srcString.startsWith("file");
			if(bl){
				String[] ps=srcString.split("\\\\");
				String path= ps[ps.length-2]+"/"+ps[ps.length-1];
				element.attr("src", "upload/"+path);
			}else{

				element.attr("src", "upload/"+srcString);
			}
		}
	}

	public String cleanoutNodeValue(String v){
		if(v.contains("...")){
			return v.split("\\...")[0];
		}
		return v;
	}

}
{% endhighlight %}
