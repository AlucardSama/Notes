## 2016/8/31 20:16:15 

#### XML解析
* SAX
* PULL
* DOM

今天解析的xml示例（channels.xml）如下：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<channel>
<item id="0" url="http://www.baidu.com">百度</item>
<item id="1" url="http://www.qq.com">腾讯</item>
<item id="2" url="http://www.sina.com.cn">新浪</item>
<item id="3" url="http://www.taobao.com">淘宝</item>
</channel>

```
 

 一、使用sax方式解析

 基础知识：

这种方式解析是一种基于事件驱动的api，有两个部分，解析器和事件处理器，解析器就是XMLReader接口，负责读取XML文档，和向事件处理器发送事件(也是事件源)，事件处理ContentHandler接口，负责对发送的事件响应和进行XML文档处理。

下面是ContentHandler接口的常用方法
``` java
public abstract void characters (char[] ch, int start, int length)
```
这个方法来接收字符块通知，解析器通过这个方法来报告字符数据块，解析器为了提高解析效率把读到的所有字符串放到一个字符数组（ch）中，作为参数传递给character的方法中，如果想获取本次事件中读取到的字符数据，需要使用start和length属性。
``` java
//接收文档开始的通知
public abstract void startDocument () 
//接收文档结束的通知
public abstract void endDocument () 
//接收文档开始的标签
public abstract void startElement (String uri, String localName, String qName, Attributes atts) 
//接收文档结束的标签
public abstract void endElement (String uri, String localName, String qName) 

```

在一般使用中为了简化开发，在org.xml.sax.helpers提供了一个DefaultHandler类，它实现了ContentHandler的方法，我们只想继承DefaultHandler方法即可。

另外SAX解析器提供了一个工厂类：**SAXParserFactory**，SAX的解析类为**SAXParser**可以调用它的**parser**方法进行解析。

看了些基础以后开始上代码吧（核心代码，下载代码在附件）

``` java
 1 public class SAXPraserHelper extends DefaultHandler {
 2 
 3     final int ITEM = 0x0005;
 4 
 5     List<channel> list;
 6     channel chann;
 7     int currentState = 0;
 8 
 9     public List<channel> getList() {
10         return list;
11     }
12 
13     /*
14      * 接口字符块通知
15 */
16     @Override
17     public void characters(char[] ch, int start, int length)
18             throws SAXException {
19         // TODO Auto-generated method stub
20         // super.characters(ch, start, length);
21         String theString = String.valueOf(ch, start, length);
22         if (currentState != 0) {
23             chann.setName(theString);
24             currentState = 0;
25         }
26         return;
27     }
28 
29     /*
30      * 接收文档结束通知
31 */
32     @Override
33     public void endDocument() throws SAXException {
34         // TODO Auto-generated method stub
35         super.endDocument();
36     }
37 
38     /*
39      * 接收标签结束通知
40 */
41     @Override
42     public void endElement(String uri, String localName, String qName)
43             throws SAXException {
44         // TODO Auto-generated method stub
45         if (localName.equals("item"))
46             list.add(chann);
47     }
48 
49     /*
50      * 文档开始通知
51 */
52     @Override
53     public void startDocument() throws SAXException {
54         // TODO Auto-generated method stub
55         list = new ArrayList<channel>();
56     }
57 
58     /*
59      * 标签开始通知
60 */
61     @Override
62     public void startElement(String uri, String localName, String qName,
63             Attributes attributes) throws SAXException {
64         // TODO Auto-generated method stub
65         chann = new channel();
66         if (localName.equals("item")) {
67             for (int i = 0; i < attributes.getLength(); i++) {
68                 if (attributes.getLocalName(i).equals("id")) {
69                     chann.setId(attributes.getValue(i));
70                 } else if (attributes.getLocalName(i).equals("url")) {
71                     chann.setUrl(attributes.getValue(i));
72                 }
73             }
74             currentState = ITEM;
75             return;
76         }
77         currentState = 0;
78         return;
79     }
80 }
```
开始解析XML数据
``` java
 1 private List<channel> getChannelList() throws ParserConfigurationException, SAXException, IOException
 2     {
 3         //实例化一个SAXParserFactory对象
 4         SAXParserFactory factory=SAXParserFactory.newInstance();
 5         SAXParser parser;
 6         //实例化SAXParser对象，创建XMLReader对象，解析器
 7         parser=factory.newSAXParser();
 8         XMLReader xmlReader=parser.getXMLReader();
 9         //实例化handler，事件处理器
10         SAXPraserHelper helperHandler=new SAXPraserHelper();
11         //解析器注册事件
12         xmlReader.setContentHandler(helperHandler);
13         //读取文件流
14         InputStream stream=getResources().openRawResource(R.raw.channels);
15         InputSource is=new InputSource(stream);
16         //解析文件
17         xmlReader.parse(is);
18         return helperHandler.getList();
19     }
```

从第二部分代码，可以看出使用SAX解析XML的步骤：

1、实例化一个工厂SAXParserFactory

2、实例化SAXPraser对象，创建XMLReader 解析器

3、实例化handler，处理器

4、解析器注册一个事件

4、读取文件流

5、解析文件

二、使用pull方式解析

基础知识：

在android系统中，很多资源文件中,很多都是xml格式，在android系统中解析这些xml的方式，是使用pul解析器进行解析的，它和sax解析一样（个人感觉要比sax简单点），也是采用事件驱动进行解析的，当pull解析器，开始解析之后，我们可以调用它的next（）方法，来获取下一个解析事件（就是开始文档，结束文档，开始标签，结束标签），当处于某个元素时可以调用XmlPullParser的getAttributte()方法来获取属性的值，也可调用它的nextText()获取本节点的值。

其实以上描述，就是对整个解析步骤的一个描述，看看代码吧

``` java
 1 private List<Map<String, String>> getData() {
 2         List<Map<String, String>> list = new ArrayList<Map<String, String>>();
 3         XmlResourceParser xrp = getResources().getXml(R.xml.channels);
 4 
 5         try {
 6             // 直到文档的结尾处
 7             while (xrp.getEventType() != XmlResourceParser.END_DOCUMENT) {
 8                 // 如果遇到了开始标签
 9                 if (xrp.getEventType() == XmlResourceParser.START_TAG) {
10                     String tagName = xrp.getName();// 获取标签的名字
11                     if (tagName.equals("item")) {
12                         Map<String, String> map = new HashMap<String, String>();
13                         String id = xrp.getAttributeValue(null, "id");// 通过属性名来获取属性值
14                         map.put("id", id);
15                         String url = xrp.getAttributeValue(1);// 通过属性索引来获取属性值
16                         map.put("url", url);
17                         map.put("name", xrp.nextText());
18                         list.add(map);
19                     }
20                 }
21                 xrp.next();// 获取解析下一个事件
22             }
23         } catch (XmlPullParserException e) {
24             // TODO Auto-generated catch block
25             e.printStackTrace();
26         } catch (IOException e) {
27             // TODO Auto-generated catch block
28             e.printStackTrace();
29         }
30 
31         return list;
32     }
```
 

三、使用Dom方式解析

基础知识：

最后来看看Dom解析方式，这种方式解析自己之前也没有用过（在j2ee开发中比较常见，没有做过这方面的东西），在Dom解析的过程中，是先把dom全部文件读入到内存中，然后使用dom的api遍历所有数据，检索想要的数据，这种方式显然是一种比较消耗内存的方式，对于像手机这样的移动设备来讲，内存是非常有限的，所以对于比较大的XML文件,不推荐使用这种方式，但是Dom也有它的优点，它比较直观，在一些方面比SAX方式比较简单。在xml文档比较小的情况下也可以考虑使用dom方式。

Dom方式解析的核心代码如下：

``` java
 1 public static List<channel> getChannelList(InputStream stream)
 2     {
 3         List<channel> list=new ArrayList<channel>();
 4         
 5         //得到 DocumentBuilderFactory 对象, 由该对象可以得到 DocumentBuilder 对象
 6         DocumentBuilderFactory factory=DocumentBuilderFactory.newInstance();
 7         
 8         try {
 9             //得到DocumentBuilder对象
10             DocumentBuilder builder=factory.newDocumentBuilder();
11             //得到代表整个xml的Document对象
12             Document document=builder.parse(stream);
13             //得到 "根节点" 
14             Element root=document.getDocumentElement();
15             //获取根节点的所有items的节点
16             NodeList items=root.getElementsByTagName("item");  
17             //遍历所有节点
18             for(int i=0;i<items.getLength();i++)
19             {
20                 channel chann=new channel();
21                 Element item=(Element)items.item(i);
22                 chann.setId(item.getAttribute("id"));
23                 chann.setUrl(item.getAttribute("url"));
24                 chann.setName(item.getFirstChild().getNodeValue());
25                 list.add(chann);
26             }
27             
28         } catch (ParserConfigurationException e) {
29             // TODO Auto-generated catch block
30             e.printStackTrace();
31         } catch (SAXException e) {
32             // TODO Auto-generated catch block
33             e.printStackTrace();
34         } catch (IOException e) {
35             // TODO Auto-generated catch block
36             e.printStackTrace();
37         }
38         
39         return list;
40     }
```

总结一下Dom解析的步骤（和sax类似）

1、调用 DocumentBuilderFactory.newInstance() 方法得到 DOM 解析器工厂类实例。

2、调用解析器工厂实例类的 newDocumentBuilder() 方法得到 DOM 解析器对象

3、调用 DOM 解析器对象的 parse() 方法解析 XML 文档得到代表整个文档的 Document 对象。

