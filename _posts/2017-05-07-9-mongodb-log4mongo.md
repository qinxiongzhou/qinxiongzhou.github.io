---
layout: post
title:  "玩转mongodb（九）：通过log4jmongo来实现分布式系统的日志统一管理"
date:   2017-05-07 11:56:34 +0800
categories: 玩转mongodb
---
* content
{:toc}

背景				{#background}
=============================

<p>　　在分布式系统中，我们有多个web app，这些web app可能分别部署在不同的物理服务器上，并且有各自的日志输出。当生产问题来临时，很多时候都需要去各个日志文件中查找可能的异常，相当耗费人力。日志存储多以文本文件形式存在，当有需求需要对日志进行分析挖掘时，这个处理起来也是诸多不便，而且效率低下。</p>
<p>　　为了方便对这些日志进行统一管理和分析，我们可以将日志统一输出到指定的数据库系统中，再由日志分析系统去管理。由于这里是mongodb的篇章，所以主观上以mongodb来做日志数据存储；客观上，一是因为它轻便、简单，与log4j整合方便，对<span style="color: #ff0000;">系统的侵入性低</span>。二是因为它与大型的关系型数据库相比有很多优势，比如<span style="color: #ff0000;">查询快速</span>、<span style="color: #ff0000;">bson存储结构利于扩展</span>、<span style="color: #ff0000;">免费</span>等。</p>

解决方案				{#solution}
=============================

<p>整合mongodb和log4j</p>
<p>1、安装mongodb数据库，并在本地启动，默认端口是27017，详细请参考：<span style="font-size: 14px;"><a id="cb_post_title_url" class="postTitle2" href="http://www.cnblogs.com/zhouqinxiong/p/5536143.html">玩转mongodb（一）：初识mongodb</a></span></p>
<p><span style="font-size: 14px;">2、新建一个maven（maven版本要求3.0以上）工程，选择maven-archetype-quickstart，工程名：log4j2mongo</span></p>
<p><span style="font-size: 14px;">3、在pom.xml文件中，添加log4j、log4mongo-java、mongo-java-driver三个依赖。具体代码如下：</span></p>
<div class="cnblogs_code" onclick="cnblogs_code_show('557a32a8-fe52-42ce-9cf0-7ffd8f4c0c82')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_557a32a8-fe52-42ce-9cf0-7ffd8f4c0c82" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_557a32a8-fe52-42ce-9cf0-7ffd8f4c0c82" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_557a32a8-fe52-42ce-9cf0-7ffd8f4c0c82" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span> <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">project </span><span style="color: #ff0000;">xmlns</span><span style="color: #0000ff;">="http://maven.apache.org/POM/4.0.0"</span><span style="color: #ff0000;"> xmlns:xsi</span><span style="color: #0000ff;">="http://www.w3.org/2001/XMLSchema-instance"</span>
<span style="color: #008080;"> 2</span> <span style="color: #ff0000;">  xsi:schemaLocation</span><span style="color: #0000ff;">="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 3</span>   <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">modelVersion</span><span style="color: #0000ff;">&gt;</span>4.0.0<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">modelVersion</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 4</span> 
<span style="color: #008080;"> 5</span>   <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">groupId</span><span style="color: #0000ff;">&gt;</span>com.manyjar<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">groupId</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 6</span>   <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">artifactId</span><span style="color: #0000ff;">&gt;</span>log4j2mongo<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">artifactId</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 7</span>   <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">version</span><span style="color: #0000ff;">&gt;</span>0.0.1-SNAPSHOT<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">version</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 8</span>   <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">packaging</span><span style="color: #0000ff;">&gt;</span>jar<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">packaging</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;"> 9</span> 
<span style="color: #008080;">10</span>   <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">name</span><span style="color: #0000ff;">&gt;</span>log4j2mongo<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">name</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">11</span>   <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">url</span><span style="color: #0000ff;">&gt;</span>http://maven.apache.org<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">url</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">12</span>   
<span style="color: #008080;">13</span>   <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">properties</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">14</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">project.build.sourceEncoding</span><span style="color: #0000ff;">&gt;</span>UTF-8<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">project.build.sourceEncoding</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">15</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">junit.version</span><span style="color: #0000ff;">&gt;</span>3.8.1<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">junit.version</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">16</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">log4j.version</span><span style="color: #0000ff;">&gt;</span>1.2.17<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">log4j.version</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">17</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">log4mongo.version</span><span style="color: #0000ff;">&gt;</span>0.7.4<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">log4mongo.version</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">18</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">mongo-java-driver.version</span><span style="color: #0000ff;">&gt;</span>2.8.0<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">mongo-java-driver.version</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">19</span>   <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">properties</span><span style="color: #0000ff;">&gt;</span> 
<span style="color: #008080;">20</span> 
<span style="color: #008080;">21</span>   <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">dependencies</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">22</span>     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">dependency</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">23</span>       <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">groupId</span><span style="color: #0000ff;">&gt;</span>junit<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">groupId</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">24</span>       <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">artifactId</span><span style="color: #0000ff;">&gt;</span>junit<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">artifactId</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">25</span>       <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">version</span><span style="color: #0000ff;">&gt;</span>${junit.version}<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">version</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">26</span>       <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">scope</span><span style="color: #0000ff;">&gt;</span>test<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">scope</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">27</span>     <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">dependency</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">28</span>   
<span style="color: #008080;">29</span>       <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">dependency</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">30</span>           <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">groupId</span><span style="color: #0000ff;">&gt;</span>log4j<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">groupId</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">31</span>           <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">artifactId</span><span style="color: #0000ff;">&gt;</span>log4j<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">artifactId</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">32</span>           <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">version</span><span style="color: #0000ff;">&gt;</span>${log4j.version}<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">version</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">33</span>       <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">dependency</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">34</span>   
<span style="color: #008080;">35</span>       <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">dependency</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">36</span>           <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">groupId</span><span style="color: #0000ff;">&gt;</span>org.log4mongo<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">groupId</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">37</span>           <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">artifactId</span><span style="color: #0000ff;">&gt;</span>log4mongo-java<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">artifactId</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">38</span>           <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">version</span><span style="color: #0000ff;">&gt;</span>${log4mongo.version}<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">version</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">39</span>       <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">dependency</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">40</span>   
<span style="color: #008080;">41</span>       <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">dependency</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">42</span>           <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">groupId</span><span style="color: #0000ff;">&gt;</span>org.mongodb<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">groupId</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">43</span>           <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">artifactId</span><span style="color: #0000ff;">&gt;</span>mongo-java-driver<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">artifactId</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">44</span>           <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">version</span><span style="color: #0000ff;">&gt;</span>${mongo-java-driver.version}<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">version</span><span style="color: #0000ff;">&gt;</span>  
<span style="color: #008080;">45</span>       <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">dependency</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">46</span>   <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">dependencies</span><span style="color: #0000ff;">&gt;</span>
<span style="color: #008080;">47</span> <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">project</span><span style="color: #0000ff;">&gt;</span></pre>
</div>
<span class="cnblogs_code_collapse">pom.xml</span></div>
<p>4、在resources文件夹中，添加log4j.properties文件。文件中主要添加log4j对mongodb的适配器org.log4mongo.MongoDbAppender。这里的适配器是log4mongo-java这个jar包提供。mongodb数据库的ip：127.0.0.1，port：27017，库名：logs，集合名：log。具体配置如下：</p>
<div class="cnblogs_code" onclick="cnblogs_code_show('27cd9f07-554f-45e0-a249-e27688f48cb5')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_27cd9f07-554f-45e0-a249-e27688f48cb5" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_27cd9f07-554f-45e0-a249-e27688f48cb5" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_27cd9f07-554f-45e0-a249-e27688f48cb5" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span> log4j.rootLogger=<span style="color: #0000ff;">DEBUG</span>,<span style="color: #000000;">MongoDB
</span><span style="color: #008080;"> 2</span> log4j.appender.MongoDB=org.log4mongo.<span style="color: #000000;">MongoDbAppender
</span><span style="color: #008080;"> 3</span> log4j.appender.MongoDB.databaseName=<span style="color: #000000;">logs
</span><span style="color: #008080;"> 4</span> log4j.appender.MongoDB.collectionName=<span style="color: #000000;">log
</span><span style="color: #008080;"> 5</span> log4j.appender.MongoDB.hostname=127.0.0.1
<span style="color: #008080;"> 6</span> log4j.appender.MongoDB.port=27017
<span style="color: #008080;"> 7</span>    
<span style="color: #008080;"> 8</span> log4j.appender.stdout=org.apache.log4j.<span style="color: #000000;">ConsoleAppender
</span><span style="color: #008080;"> 9</span> log4j.appender.stdout.layout=org.apache.log4j.<span style="color: #000000;">PatternLayout
</span><span style="color: #008080;">10</span> log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH<span style="color: #800000;">:mm:ss</span>,SSS} <span style="color: #800080;">%5</span>p %c{1}:%L - %m%n</pre>
</div>
<span class="cnblogs_code_collapse">log4j.properties</span></div>
<p>5、在java文件夹中，com.manyjar.log4j2mongo这个包中，添加Main.java文件，观察文件中的代码可以发现，和我们平时用log4j来写日志一模一样，把日志流到mongodb这件事情，对业务开发的程序员完全透明。具体代码如下：</p>
<div class="cnblogs_code" onclick="cnblogs_code_show('1005cb36-3dd9-4c2d-b2e8-21a756f38182')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_1005cb36-3dd9-4c2d-b2e8-21a756f38182" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_1005cb36-3dd9-4c2d-b2e8-21a756f38182" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_1005cb36-3dd9-4c2d-b2e8-21a756f38182" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span> <span style="color: #0000ff;">package</span><span style="color: #000000;"> com.manyjar.log4j2mongo;
</span><span style="color: #008080;"> 2</span> 
<span style="color: #008080;"> 3</span> <span style="color: #0000ff;">import</span><span style="color: #000000;"> org.apache.log4j.Logger;
</span><span style="color: #008080;"> 4</span> 
<span style="color: #008080;"> 5</span> <span style="color: #0000ff;">import</span><span style="color: #000000;"> com.mongodb.BasicDBObject;
</span><span style="color: #008080;"> 6</span> <span style="color: #0000ff;">import</span><span style="color: #000000;"> com.mongodb.DBObject;
</span><span style="color: #008080;"> 7</span> 
<span style="color: #008080;"> 8</span> <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span><span style="color: #000000;"> Main {
</span><span style="color: #008080;"> 9</span>     
<span style="color: #008080;">10</span>     <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">static</span> <span style="color: #0000ff;">void</span><span style="color: #000000;"> main(String[] args) {
</span><span style="color: #008080;">11</span>         Logger logger = Logger.getLogger(Main.<span style="color: #0000ff;">class</span><span style="color: #000000;">);
</span><span style="color: #008080;">12</span>             
<span style="color: #008080;">13</span>             <span style="color: #0000ff;">for</span> (<span style="color: #0000ff;">int</span> i = 0; i &lt; 10000000; i++<span style="color: #000000;">) {
</span><span style="color: #008080;">14</span>                 DBObject bson = <span style="color: #0000ff;">new</span><span style="color: #000000;"> BasicDBObject();
</span><span style="color: #008080;">15</span>                 bson.put("name", "ryan"+<span style="color: #000000;">i);
</span><span style="color: #008080;">16</span> <span style="color: #000000;">                logger.debug(bson);
</span><span style="color: #008080;">17</span> <span style="color: #000000;">            }
</span><span style="color: #008080;">18</span> <span style="color: #000000;">    }
</span><span style="color: #008080;">19</span> }</pre>
</div>
<span class="cnblogs_code_collapse">Main.java</span></div>
<p>6、步骤5中，我们执行了1000万次的日志插入，数据结构如下：</p>
<p><img src="http://images2015.cnblogs.com/blog/440176/201705/440176-20170507214522148-1344626629.png" alt="" /></p>
<p>默认的数据量大小有10G：</p>
<p><img src="http://images2015.cnblogs.com/blog/440176/201705/440176-20170507214318929-333411405.png" alt="" /></p>
<p>这里，我们可以看到，日志的数据存储量相对不小。如果需要修改日志数据的存储结构，可以用log4mongo的源代码进行二次开发。</p>
<p>如果数据量过大，我们可以用TTL索引（过期自动删除）或固定集合大小两种方式来解决：</p>
<p><code class="sql plain">TTL索引:<span style="font-size: 14px;">db.log_events.createIndex({</span></code><span style="font-size: 14px;"><code class="sql string">"timestamp"</code></span><code class="sql plain"><span style="font-size: 14px;">: 1},{expireAfterSeconds: 60*60*24*30})</span> #1个月后过期后删除</code></p>
<p><code class="sql plain"></code><span class="pun"><span class="pln">将log集合修改成固定大小集合：<span style="font-size: 14px; font-family: 宋体;">db</span><span class="pun"><span style="font-size: 14px; font-family: 宋体;">.</span><span class="pln"><span style="font-size: 14px; font-family: 宋体;">runCommand</span><span class="pun"><span style="font-size: 14px; font-family: 宋体;">({</span><span class="str"><span style="font-size: 14px; font-family: 宋体;">"convertToCapped"</span><span class="pun"><span style="font-size: 14px; font-family: 宋体;">:</span><span class="str"><span style="font-size: 14px; font-family: 宋体;">"log"</span><span class="pun"><span style="font-size: 14px; font-family: 宋体;">,</span><span class="pln"><span style="font-size: 14px; font-family: 宋体;">size</span><span class="pun"><span style="font-size: 14px; font-family: 宋体;">:</span><span class="lit"><span style="font-size: 14px; font-family: 宋体;">10000</span><span class="pun"><span style="font-size: 14px; font-family: 宋体;">})</span>。</span></span></span></span></span></span></span></span></span></span></span></span></span></p>
<p>&nbsp;</p>


致谢！             {#thanks}
===========================

<p>&nbsp; &nbsp; &nbsp; &nbsp;至此，我们完成了开发编译调试到最终上线生产运行。喜欢请关注公众号--程序猿牧场，谢谢！</p>
<p>&nbsp;<img src="https://img2020.cnblogs.com/blog/440176/202003/440176-20200316224639427-719739537.png" alt="" /></p> 
