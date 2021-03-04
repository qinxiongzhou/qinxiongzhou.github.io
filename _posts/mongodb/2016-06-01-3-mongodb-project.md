---
layout: post
title:  "玩转mongodb（三）：mongodb项目实战（初战）"
date:   2016-06-01 23:13:38 +0800
categories: 玩转mongodb
---
* content
{:toc}

简介				{#introduction}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; 主要功能：对mongodb的集合做增删改查。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 项目的运行环境：tomcat6、jdk8。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 所用技术：jsp/servlet、前端bootstrap。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; mongodb：personmap。</span></p>

mongodb工具类：				{#toolUtil}
=============================

<p><span style="line-height: 24px;">&nbsp; &nbsp; <span style="font-size: 16px;">定义一个MongoDBUtil的枚举类，枚举类中定义一个instance实例。</span></span></p>
<p><span style="line-height: 24px; font-size: 16px;">&nbsp;&nbsp;&nbsp; MongoDB工具类 Mongo实例代表了一个数据库连接池，即使在多线程的环境中，一个Mongo实例对我们来说已经足够。</span><span style="line-height: 24px; font-size: 16px;"><br />&nbsp;&nbsp;&nbsp;&nbsp;注意Mongo已经实现了连接池，并且是线程安全的。<br />&nbsp;&nbsp;&nbsp; 设计为单例模式， 因 MongoDB的Java驱动是线程安全的，对于一般的应用，只要一个Mongo实例即可。<br />&nbsp;&nbsp;&nbsp; Mongo有个内置的连接池（默认为10个） 对于有大量写和读的环境中，为了确保在一个Session中使用同一个DB时，DB和DBCollection是绝对线程安全的</span></p>
<div class="cnblogs_code" onclick="cnblogs_code_show('e5f1feaa-6321-4490-b255-65dcc963b31e')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_e5f1feaa-6321-4490-b255-65dcc963b31e" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_e5f1feaa-6321-4490-b255-65dcc963b31e" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_e5f1feaa-6321-4490-b255-65dcc963b31e" class="cnblogs_code_hide">
<pre><span style="color: #008080;">1</span> <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">enum</span><span style="color: #000000;"> MongoDBUtil {
</span><span style="color: #008080;">2</span>     <span style="color: #008000;">/**</span>
<span style="color: #008080;">3</span> <span style="color: #008000;">     * 定义一个枚举的元素，它代表此类的一个实例
</span><span style="color: #008080;">4</span>      <span style="color: #008000;">*/</span>
<span style="color: #008080;">5</span> <span style="color: #000000;">    instance;
</span><span style="color: #008080;">6</span> }</pre>
</div>
<span class="cnblogs_code_collapse">View Code</span></div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 在MongoDBUtil类中，定义一个MongoClient对象，并根据IP和端口获得该对象。</span></p>
<div class="cnblogs_code" onclick="cnblogs_code_show('c26346bf-1852-4761-b5b3-fc2fe3f1978b')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_c26346bf-1852-4761-b5b3-fc2fe3f1978b" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_c26346bf-1852-4761-b5b3-fc2fe3f1978b" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_c26346bf-1852-4761-b5b3-fc2fe3f1978b" class="cnblogs_code_hide">
<pre><span style="color: #008080;">1</span> <span style="color: #0000ff;">private</span><span style="color: #000000;"> MongoClient mongoClient;
</span><span style="color: #008080;">2</span>     <span style="color: #0000ff;">static</span><span style="color: #000000;"> {
</span><span style="color: #008080;">3</span>         System.out.println("===============MongoDBUtil初始化========================"<span style="color: #000000;">);
</span><span style="color: #008080;">4</span>         <span style="color: #008000;">//</span><span style="color: #008000;"> 从配置文件中获取属性值</span>
<span style="color: #008080;">5</span>         String ip = "localhost"<span style="color: #000000;">;
</span><span style="color: #008080;">6</span>         <span style="color: #0000ff;">int</span> port = 27017<span style="color: #000000;">;
</span><span style="color: #008080;">7</span>         instance.mongoClient = <span style="color: #0000ff;">new</span><span style="color: #000000;"> MongoClient(ip, port);
</span><span style="color: #008080;">8</span>     }</pre>
</div>
<span class="cnblogs_code_collapse">View Code</span></div>
<p>&nbsp; &nbsp; &nbsp;<span style="font-size: 16px;">根据MongoClient对象，得到MongoDataBase对象和MongoConnection&lt;Document&gt;对象。</span></p>
<div class="cnblogs_code" onclick="cnblogs_code_show('74a5bc98-e573-4e8e-a3f0-430497113493')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_74a5bc98-e573-4e8e-a3f0-430497113493" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_74a5bc98-e573-4e8e-a3f0-430497113493" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_74a5bc98-e573-4e8e-a3f0-430497113493" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span>  <span style="color: #008000;">/**</span>
<span style="color: #008080;"> 2</span> <span style="color: #008000;">     * 获取DB实例 - 指定DB
</span><span style="color: #008080;"> 3</span> <span style="color: #008000;">     * 
</span><span style="color: #008080;"> 4</span> <span style="color: #008000;">     * </span><span style="color: #808080;">@param</span><span style="color: #008000;"> dbName
</span><span style="color: #008080;"> 5</span> <span style="color: #008000;">     * </span><span style="color: #808080;">@return</span>
<span style="color: #008080;"> 6</span>      <span style="color: #008000;">*/</span>
<span style="color: #008080;"> 7</span>     <span style="color: #0000ff;">public</span><span style="color: #000000;"> MongoDatabase getDB(String dbName) {
</span><span style="color: #008080;"> 8</span>         <span style="color: #0000ff;">if</span> (dbName != <span style="color: #0000ff;">null</span> &amp;&amp; !""<span style="color: #000000;">.equals(dbName)) {
</span><span style="color: #008080;"> 9</span>             MongoDatabase database =<span style="color: #000000;"> mongoClient.getDatabase(dbName);
</span><span style="color: #008080;">10</span>             <span style="color: #0000ff;">return</span><span style="color: #000000;"> database;
</span><span style="color: #008080;">11</span> <span style="color: #000000;">        }
</span><span style="color: #008080;">12</span>         <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">null</span><span style="color: #000000;">;
</span><span style="color: #008080;">13</span> <span style="color: #000000;">    }
</span><span style="color: #008080;">14</span>     <span style="color: #008000;">/**</span>
<span style="color: #008080;">15</span> <span style="color: #008000;">     * 获取collection对象 - 指定Collection
</span><span style="color: #008080;">16</span> <span style="color: #008000;">     * 
</span><span style="color: #008080;">17</span> <span style="color: #008000;">     * </span><span style="color: #808080;">@param</span><span style="color: #008000;"> collName
</span><span style="color: #008080;">18</span> <span style="color: #008000;">     * </span><span style="color: #808080;">@return</span>
<span style="color: #008080;">19</span>      <span style="color: #008000;">*/</span>
<span style="color: #008080;">20</span>     <span style="color: #0000ff;">public</span> MongoCollection&lt;Document&gt;<span style="color: #000000;"> getCollection(String dbName, String collName) {
</span><span style="color: #008080;">21</span>         <span style="color: #0000ff;">if</span> (<span style="color: #0000ff;">null</span> == collName || ""<span style="color: #000000;">.equals(collName)) {
</span><span style="color: #008080;">22</span>             <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">null</span><span style="color: #000000;">;
</span><span style="color: #008080;">23</span> <span style="color: #000000;">        }
</span><span style="color: #008080;">24</span>         <span style="color: #0000ff;">if</span> (<span style="color: #0000ff;">null</span> == dbName || ""<span style="color: #000000;">.equals(dbName)) {
</span><span style="color: #008080;">25</span>             <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">null</span><span style="color: #000000;">;
</span><span style="color: #008080;">26</span> <span style="color: #000000;">        }
</span><span style="color: #008080;">27</span>         MongoCollection&lt;Document&gt; collection =<span style="color: #000000;"> mongoClient.getDatabase(dbName).getCollection(collName);
</span><span style="color: #008080;">28</span>         <span style="color: #0000ff;">return</span><span style="color: #000000;"> collection;
</span><span style="color: #008080;">29</span>     }</pre>
</div>
<span class="cnblogs_code_collapse">View Code</span></div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; &nbsp;工具类的查询、插入、更新、删除方法。</span></p>
<div class="cnblogs_code" onclick="cnblogs_code_show('8b000d72-5968-46eb-bc7b-403a3459167a')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_8b000d72-5968-46eb-bc7b-403a3459167a" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_8b000d72-5968-46eb-bc7b-403a3459167a" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_8b000d72-5968-46eb-bc7b-403a3459167a" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span>     <span style="color: #008000;">/**</span><span style="color: #008000;">条件查询</span><span style="color: #008000;">*/</span>
<span style="color: #008080;"> 2</span>     <span style="color: #0000ff;">public</span> MongoCursor&lt;Document&gt; find(MongoCollection&lt;Document&gt;<span style="color: #000000;"> coll, Bson filter) {
</span><span style="color: #008080;"> 3</span>         <span style="color: #0000ff;">if</span>(<span style="color: #0000ff;">null</span>!=<span style="color: #000000;">filter){
</span><span style="color: #008080;"> 4</span>             <span style="color: #0000ff;">return</span><span style="color: #000000;"> coll.find(filter).iterator();
</span><span style="color: #008080;"> 5</span>         }<span style="color: #0000ff;">else</span><span style="color: #000000;">{
</span><span style="color: #008080;"> 6</span>             <span style="color: #0000ff;">return</span><span style="color: #000000;"> coll.find().iterator();
</span><span style="color: #008080;"> 7</span> <span style="color: #000000;">        }
</span><span style="color: #008080;"> 8</span> <span style="color: #000000;">    }
</span><span style="color: #008080;"> 9</span>     <span style="color: #008000;">/**</span><span style="color: #008000;">插入一条数据</span><span style="color: #008000;">*/</span>
<span style="color: #008080;">10</span>     <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">void</span> insert(MongoCollection&lt;Document&gt;<span style="color: #000000;"> coll,Document doc){
</span><span style="color: #008080;">11</span> <span style="color: #000000;">        coll.insertOne(doc);
</span><span style="color: #008080;">12</span> <span style="color: #000000;">    }
</span><span style="color: #008080;">13</span>     
<span style="color: #008080;">14</span>     <span style="color: #008000;">/**</span><span style="color: #008000;">更新一条数据</span><span style="color: #008000;">*/</span>
<span style="color: #008080;">15</span>     <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">void</span> update(MongoCollection&lt;Document&gt;<span style="color: #000000;"> coll,Document querydoc,Document updatedoc){
</span><span style="color: #008080;">16</span> <span style="color: #000000;">        coll.updateMany(querydoc, updatedoc);
</span><span style="color: #008080;">17</span> <span style="color: #000000;">    }
</span><span style="color: #008080;">18</span> 
<span style="color: #008080;">19</span>     <span style="color: #008000;">/**</span><span style="color: #008000;">删除一条数据</span><span style="color: #008000;">*/</span>
<span style="color: #008080;">20</span>     <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">void</span> delete(MongoCollection&lt;Document&gt;<span style="color: #000000;"> coll,Document doc){
</span><span style="color: #008080;">21</span> <span style="color: #000000;">        coll.deleteMany(doc);
</span><span style="color: #008080;">22</span>     }</pre>
</div>
<span class="cnblogs_code_collapse">View Code</span></div>
<p>&nbsp;</p>

项目中的增删改查				{#IRUD}
=============================

<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp; 插入：对应MongoDB中脚本的db.getCollection('person').insert({"name":"ryan1","age":21})</span></p>
<div class="cnblogs_code" onclick="cnblogs_code_show('5e946c41-1b27-4af2-8e79-db93a01de0d6')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_5e946c41-1b27-4af2-8e79-db93a01de0d6" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_5e946c41-1b27-4af2-8e79-db93a01de0d6" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_5e946c41-1b27-4af2-8e79-db93a01de0d6" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span>     String name = request.getParameter("name"<span style="color: #000000;">);
</span><span style="color: #008080;"> 2</span>     Double age = Double.valueOf(request.getParameter("age"<span style="color: #000000;">));
</span><span style="color: #008080;"> 3</span>     
<span style="color: #008080;"> 4</span>     String dbName = "personmap"<span style="color: #000000;">;
</span><span style="color: #008080;"> 5</span>         String collName = "person"<span style="color: #000000;">;
</span><span style="color: #008080;"> 6</span>         MongoCollection&lt;Document&gt; coll =<span style="color: #000000;"> MongoDBUtil.instance.getCollection(dbName, collName);
</span><span style="color: #008080;"> 7</span>         
<span style="color: #008080;"> 8</span>         Document doc = <span style="color: #0000ff;">new</span><span style="color: #000000;"> Document();
</span><span style="color: #008080;"> 9</span>         doc.put("name"<span style="color: #000000;">, name);
</span><span style="color: #008080;">10</span>         doc.put("age"<span style="color: #000000;">, age);
</span><span style="color: #008080;">11</span> <span style="color: #000000;">        MongoDBUtil.instance.insert(coll, doc);
</span><span style="color: #008080;">12</span>         
<span style="color: #008080;">13</span>         PrintWriter out =<span style="color: #000000;"> response.getWriter();
</span><span style="color: #008080;">14</span>         out.write("insert success!");</pre>
</div>
<span class="cnblogs_code_collapse">插入功能的servlet</span></div>
<div class="cnblogs_code" onclick="cnblogs_code_show('8283885e-b8a4-45d7-b25e-43c45fe3f41d')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_8283885e-b8a4-45d7-b25e-43c45fe3f41d" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_8283885e-b8a4-45d7-b25e-43c45fe3f41d" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_8283885e-b8a4-45d7-b25e-43c45fe3f41d" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span> <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">div </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="panel    panel-warning"</span><span style="color: #ff0000;"> style</span><span style="color: #0000ff;">="width:20%"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 2</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">div </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="panel-heading"</span><span style="color: #0000ff;">&gt;</span>删除文档<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">div</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 3</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">div </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="panel-body"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 4</span>         <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">form </span><span style="color: #ff0000;">action</span><span style="color: #0000ff;">="InsertPerson"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 5</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">input </span><span style="color: #ff0000;">type</span><span style="color: #0000ff;">="text"</span><span style="color: #ff0000;"> name</span><span style="color: #0000ff;">="name"</span><span style="color: #ff0000;"> class</span><span style="color: #0000ff;">="form-control"</span><span style="color: #ff0000;"> placeholder</span><span style="color: #0000ff;">="name"</span><span style="color: #ff0000;"> aria-describedby</span><span style="color: #0000ff;">="basic-addon1"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 6</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">br</span><span style="color: #0000ff;">/&gt;</span>
<span style="color: #008080;"> 7</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">input </span><span style="color: #ff0000;">type</span><span style="color: #0000ff;">="text"</span><span style="color: #ff0000;"> name</span><span style="color: #0000ff;">="age"</span><span style="color: #ff0000;"> class</span><span style="color: #0000ff;">="form-control"</span><span style="color: #ff0000;"> placeholder</span><span style="color: #0000ff;">="age"</span><span style="color: #ff0000;"> aria-describedby</span><span style="color: #0000ff;">="basic-addon1"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 8</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">br</span><span style="color: #0000ff;">/&gt;</span>
<span style="color: #008080;"> 9</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">button </span><span style="color: #ff0000;">type</span><span style="color: #0000ff;">="submit"</span><span style="color: #ff0000;"> class</span><span style="color: #0000ff;">="btn btn-default"</span><span style="color: #0000ff;">&gt;</span>插入<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">button</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">10</span>         <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">form</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">11</span>     <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">div</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">12</span> <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">div</span><span style="color: #0000ff;">&gt;</span></pre>
</div>
<span class="cnblogs_code_collapse">插入功能的jsp</span></div>
<p>&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201606/440176-20160601101047289-1570037732.png" alt="" /></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 更新：对应MongoDB中脚本的db.getCollection('person').update({"name":"ryan1"}{"$set":{"age":22}})</span></p>
<div class="cnblogs_code" onclick="cnblogs_code_show('54e53ff2-89f0-44c5-aa4e-a03e68a5b397')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_54e53ff2-89f0-44c5-aa4e-a03e68a5b397" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_54e53ff2-89f0-44c5-aa4e-a03e68a5b397" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_54e53ff2-89f0-44c5-aa4e-a03e68a5b397" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span>     String queryname = request.getParameter("queryname"<span style="color: #000000;">);
</span><span style="color: #008080;"> 2</span>     Double updateage = Double.valueOf(request.getParameter("updateage"<span style="color: #000000;">));
</span><span style="color: #008080;"> 3</span>     
<span style="color: #008080;"> 4</span>     String dbName = "personmap"<span style="color: #000000;">;
</span><span style="color: #008080;"> 5</span>     String collName = "person"<span style="color: #000000;">;
</span><span style="color: #008080;"> 6</span>     MongoCollection&lt;Document&gt; coll =<span style="color: #000000;"> MongoDBUtil.instance.getCollection(dbName, collName);
</span><span style="color: #008080;"> 7</span>         
<span style="color: #008080;"> 8</span>     Document querydoc = <span style="color: #0000ff;">new</span><span style="color: #000000;"> Document();
</span><span style="color: #008080;"> 9</span>     querydoc.put("name"<span style="color: #000000;">, queryname);
</span><span style="color: #008080;">10</span> 
<span style="color: #008080;">11</span>     Document updatedoc = <span style="color: #0000ff;">new</span><span style="color: #000000;"> Document();
</span><span style="color: #008080;">12</span>     updatedoc.put("name"<span style="color: #000000;">, queryname);
</span><span style="color: #008080;">13</span>     updatedoc.put("age"<span style="color: #000000;">, updateage);
</span><span style="color: #008080;">14</span>                 
<span style="color: #008080;">15</span>     MongoDBUtil.instance.update(coll, querydoc , <span style="color: #0000ff;">new</span> Document("$set"<span style="color: #000000;">,updatedoc));
</span><span style="color: #008080;">16</span>         
<span style="color: #008080;">17</span>         PrintWriter out =<span style="color: #000000;"> response.getWriter();
</span><span style="color: #008080;">18</span>         out.write("update success!");</pre>
</div>
<span class="cnblogs_code_collapse">更新功能的servlet</span></div>
<div class="cnblogs_code" onclick="cnblogs_code_show('80489b6e-f419-446b-88e4-7f4fa90ff206')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_80489b6e-f419-446b-88e4-7f4fa90ff206" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_80489b6e-f419-446b-88e4-7f4fa90ff206" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_80489b6e-f419-446b-88e4-7f4fa90ff206" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span> <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">div </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="panel    panel-warning"</span><span style="color: #ff0000;"> style</span><span style="color: #0000ff;">="width:20%"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 2</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">div </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="panel-heading"</span><span style="color: #0000ff;">&gt;</span>根据name更新age<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">div</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 3</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">div </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="panel-body"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 4</span>         <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">form </span><span style="color: #ff0000;">action</span><span style="color: #0000ff;">="UpdatePerson"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 5</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">input </span><span style="color: #ff0000;">type</span><span style="color: #0000ff;">="text"</span><span style="color: #ff0000;"> name</span><span style="color: #0000ff;">="queryname"</span><span style="color: #ff0000;"> class</span><span style="color: #0000ff;">="form-control"</span><span style="color: #ff0000;"> placeholder</span><span style="color: #0000ff;">="queryname"</span><span style="color: #ff0000;"> aria-describedby</span><span style="color: #0000ff;">="basic-addon1"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 6</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">br</span><span style="color: #0000ff;">/&gt;</span>
<span style="color: #008080;"> 7</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">input </span><span style="color: #ff0000;">type</span><span style="color: #0000ff;">="text"</span><span style="color: #ff0000;"> name</span><span style="color: #0000ff;">="updateage"</span><span style="color: #ff0000;"> class</span><span style="color: #0000ff;">="form-control"</span><span style="color: #ff0000;"> placeholder</span><span style="color: #0000ff;">="updateage"</span><span style="color: #ff0000;"> aria-describedby</span><span style="color: #0000ff;">="basic-addon1"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 8</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">br</span><span style="color: #0000ff;">/&gt;</span>
<span style="color: #008080;"> 9</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">button </span><span style="color: #ff0000;">type</span><span style="color: #0000ff;">="submit"</span><span style="color: #ff0000;"> class</span><span style="color: #0000ff;">="btn btn-default"</span><span style="color: #0000ff;">&gt;</span>更新<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">button</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">10</span>     <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">form</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">11</span>     <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">div</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">12</span> <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">div</span><span style="color: #0000ff;">&gt;</span></pre>
</div>
<span class="cnblogs_code_collapse">更新功能的jsp</span></div>
<p>&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201606/440176-20160601101528492-1052110041.png" alt="" /></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 删除：对应MongoDB中脚本的db.getCollection('person').remove({"name":"ryan1"})</span></p>
<div class="cnblogs_code" onclick="cnblogs_code_show('dab3ce1c-cd2f-450b-a41c-2dffa752e21f')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_dab3ce1c-cd2f-450b-a41c-2dffa752e21f" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_dab3ce1c-cd2f-450b-a41c-2dffa752e21f" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_dab3ce1c-cd2f-450b-a41c-2dffa752e21f" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span>     String name = request.getParameter("name"<span style="color: #000000;">);
</span><span style="color: #008080;"> 2</span>     
<span style="color: #008080;"> 3</span>     String dbName = "personmap"<span style="color: #000000;">;
</span><span style="color: #008080;"> 4</span>     String collName = "person"<span style="color: #000000;">;
</span><span style="color: #008080;"> 5</span>     MongoCollection&lt;Document&gt; coll =<span style="color: #000000;"> MongoDBUtil.instance.getCollection(dbName, collName);
</span><span style="color: #008080;"> 6</span>         
<span style="color: #008080;"> 7</span>     Document doc = <span style="color: #0000ff;">new</span><span style="color: #000000;"> Document();
</span><span style="color: #008080;"> 8</span>     doc.put("name"<span style="color: #000000;">, name);
</span><span style="color: #008080;"> 9</span> 
<span style="color: #008080;">10</span> <span style="color: #000000;">    MongoDBUtil.instance.delete(coll, doc);
</span><span style="color: #008080;">11</span>         
<span style="color: #008080;">12</span>     PrintWriter out =<span style="color: #000000;"> response.getWriter();
</span><span style="color: #008080;">13</span>     out.write("delete success!");</pre>
</div>
<span class="cnblogs_code_collapse">删除功能的servlet</span></div>
<div class="cnblogs_code" onclick="cnblogs_code_show('df166f53-e046-4b1d-a843-b267468722ba')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_df166f53-e046-4b1d-a843-b267468722ba" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_df166f53-e046-4b1d-a843-b267468722ba" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_df166f53-e046-4b1d-a843-b267468722ba" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span> <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">div </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="panel    panel-warning"</span><span style="color: #ff0000;"> style</span><span style="color: #0000ff;">="width:20%"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 2</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">div </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="panel-heading"</span><span style="color: #0000ff;">&gt;</span>删除文档<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">div</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 3</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">div </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="panel-body"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 4</span>         <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">form </span><span style="color: #ff0000;">action</span><span style="color: #0000ff;">="DeletePerson"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 5</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">input </span><span style="color: #ff0000;">type</span><span style="color: #0000ff;">="text"</span><span style="color: #ff0000;"> name</span><span style="color: #0000ff;">="name"</span><span style="color: #ff0000;"> class</span><span style="color: #0000ff;">="form-control"</span><span style="color: #ff0000;"> placeholder</span><span style="color: #0000ff;">="name"</span><span style="color: #ff0000;"> aria-describedby</span><span style="color: #0000ff;">="basic-addon1"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 6</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">br</span><span style="color: #0000ff;">/&gt;</span>
<span style="color: #008080;"> 7</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">button </span><span style="color: #ff0000;">type</span><span style="color: #0000ff;">="submit"</span><span style="color: #ff0000;"> class</span><span style="color: #0000ff;">="btn btn-default"</span><span style="color: #0000ff;">&gt;</span>删除<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">button</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 8</span>         <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">form</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 9</span>     <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">div</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">10</span> <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">div</span><span style="color: #0000ff;">&gt;</span></pre>
</div>
<span class="cnblogs_code_collapse">删除功能的jsp</span></div>
<p>&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201606/440176-20160601101852867-434459322.png" alt="" /></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 查找：对应MongoDB中脚本的db.getCollection('person').find({})</span></p>
<div class="cnblogs_code" onclick="cnblogs_code_show('72bdcda2-1c13-4aa8-8775-bd9b7191ee5a')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_72bdcda2-1c13-4aa8-8775-bd9b7191ee5a" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_72bdcda2-1c13-4aa8-8775-bd9b7191ee5a" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_72bdcda2-1c13-4aa8-8775-bd9b7191ee5a" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span>     String dbName = "personmap"<span style="color: #000000;">;
</span><span style="color: #008080;"> 2</span>     String collName = "person"<span style="color: #000000;">;
</span><span style="color: #008080;"> 3</span>     MongoCollection&lt;Document&gt; coll =<span style="color: #000000;"> MongoDBUtil.instance.getCollection(dbName, collName);
</span><span style="color: #008080;"> 4</span>         
<span style="color: #008080;"> 5</span>     List&lt;Person&gt; personList = <span style="color: #0000ff;">new</span> ArrayList&lt;Person&gt;<span style="color: #000000;">();
</span><span style="color: #008080;"> 6</span>     <span style="color: #008000;">//</span><span style="color: #008000;"> 查询所有
</span><span style="color: #008080;"> 7</span>     <span style="color: #008000;">//</span><span style="color: #008000;">Bson filter = Filters.eq("name", "ryan1");</span>
<span style="color: #008080;"> 8</span>     Bson filter = <span style="color: #0000ff;">null</span><span style="color: #000000;">;
</span><span style="color: #008080;"> 9</span>     MongoCursor&lt;Document&gt; cursor =<span style="color: #000000;"> MongoDBUtil.instance.find(coll, filter);
</span><span style="color: #008080;">10</span>     <span style="color: #0000ff;">while</span><span style="color: #000000;">(cursor.hasNext()){
</span><span style="color: #008080;">11</span>         Document tempdoc =<span style="color: #000000;"> cursor.next();
</span><span style="color: #008080;">12</span>         Person person = <span style="color: #0000ff;">new</span><span style="color: #000000;"> Person();
</span><span style="color: #008080;">13</span>         person.set_id(tempdoc.get("_id"<span style="color: #000000;">).toString());
</span><span style="color: #008080;">14</span>         person.setName(tempdoc.get("name"<span style="color: #000000;">).toString());
</span><span style="color: #008080;">15</span> person.setAge(Double.valueOf(tempdoc.get("age"<span style="color: #000000;">).toString()));
</span><span style="color: #008080;">16</span> <span style="color: #000000;">            personList.add(person);
</span><span style="color: #008080;">17</span> <span style="color: #000000;">    }
</span><span style="color: #008080;">18</span>         
<span style="color: #008080;">19</span>     Gson gson = <span style="color: #0000ff;">new</span><span style="color: #000000;"> Gson();
</span><span style="color: #008080;">20</span>         
<span style="color: #008080;">21</span>     PrintWriter out =<span style="color: #000000;"> response.getWriter();
</span><span style="color: #008080;">22</span>     out.write(gson.toJson(personList));</pre>
</div>
<span class="cnblogs_code_collapse">查找功能的servlet</span></div>
<div class="cnblogs_code" onclick="cnblogs_code_show('72cd5e26-4c6a-496e-9c49-873042f9ca5c')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_72cd5e26-4c6a-496e-9c49-873042f9ca5c" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_72cd5e26-4c6a-496e-9c49-873042f9ca5c" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_72cd5e26-4c6a-496e-9c49-873042f9ca5c" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span> <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">div </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="panel panel-warning"</span><span style="color: #ff0000;"> style</span><span style="color: #0000ff;">="width:50%"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 2</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">div </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="panel-heading"</span><span style="color: #0000ff;">&gt;</span>查找全部数据<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">div</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 3</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">div </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="panel-body"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 4</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">table </span><span style="color: #ff0000;">data-toggle</span><span style="color: #0000ff;">="table"</span> 
<span style="color: #008080;"> 5</span> <span style="color: #ff0000;">               data-url</span><span style="color: #0000ff;">="FindPerson"</span>
<span style="color: #008080;"> 6</span> <span style="color: #ff0000;">               data-classes</span><span style="color: #0000ff;">="table table-hover table-condensed"</span>
<span style="color: #008080;"> 7</span> <span style="color: #ff0000;">               data-striped</span><span style="color: #0000ff;">="true"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 8</span>         <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">thead</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 9</span>             <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">tr</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">10</span>                 <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">th </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="col-xs-2"</span><span style="color: #ff0000;"> data-field</span><span style="color: #0000ff;">="_id"</span><span style="color: #0000ff;">&gt;</span>_id<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">th</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">11</span>                 <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">th </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="col-xs-2"</span><span style="color: #ff0000;"> data-field</span><span style="color: #0000ff;">="name"</span><span style="color: #0000ff;">&gt;</span>name<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">th</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">12</span>                 <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">th </span><span style="color: #ff0000;">class</span><span style="color: #0000ff;">="col-xs-2"</span><span style="color: #ff0000;"> data-field</span><span style="color: #0000ff;">="age"</span><span style="color: #0000ff;">&gt;</span>age<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">th</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">13</span>             <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">tr</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">14</span>         <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">thead</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">15</span>     <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">table</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">16</span>     <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">div</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">17</span> <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">div</span><span style="color: #0000ff;">&gt;</span></pre>
</div>
<span class="cnblogs_code_collapse">查找功能的jsp</span></div>
<p>&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201606/440176-20160601102412992-1575692544.png" alt="" /></p>


源代码下载				{#download-source}
=============================

<p><span style="font-size: 16px; color: #00ff00;">&nbsp; &nbsp; <a title="mongodb项目实战（初战）源代码" href="http://download.csdn.net/detail/zhouqinxiong/9537463" target="_blank">源代码下载</a></span></p>

致谢！             {#thanks}
===========================

<p>&nbsp; &nbsp; &nbsp; &nbsp;至此，我们完成了开发编译调试到最终上线生产运行。喜欢请关注公众号--程序猿牧场，谢谢！</p>
<p>&nbsp;<img src="https://img2020.cnblogs.com/blog/440176/202003/440176-20200316224639427-719739537.png" alt="" /></p>