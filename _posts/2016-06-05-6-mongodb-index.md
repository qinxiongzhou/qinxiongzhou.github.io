---
layout: post
title:  "玩转mongodb（六）：索引，速度的引领（普通索引篇）"
date:   2016-06-05 10:56:34 +0800
categories: 玩转mongodb
---
* content
{:toc}

简介				{#introduction}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; 数据库索引与书籍的索引类似，有了索引就不需要翻整本书，数据库可以直接在索引中查找，在索引中找到条目后，就可以直接跳到目标文档的位置，这可以让查找的速度提高几个数量级。</span></p>
<p>&nbsp;</p>

一、创建索引				{#one}
=============================

<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp; 我们在person这个集合的age键上创建一个索引，比较一下创建索引前后，一个查询的语句的性能区别。</span></p>
<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp; 创建索引：db.person.ensureIndex({"age":1})。</span><span style="font-size: 16px; line-height: 24px;">这里我们使用了ensureIndex在age上建立了索引。&ldquo;1&rdquo;：表示按照age进行升序，&ldquo;-1&rdquo;：表示按照age进行降序。</span></p>
<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp; 没有索引的查询性能：</span></p>
<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201606/440176-20160604183051133-2005175824.png" alt="" /></span></p>
<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp; 有索引的查询性能：</span></p>
<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201606/440176-20160604183131742-109437967.png" alt="" />&nbsp;</span></p>
<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp; 我们主要来看这几个参数，（<span style="color: #ff9900;">参数说明，请看上一篇文章</span>）</span></p>
<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp; executionTimeMillis（这次query整体的耗时）：无索引耗时<span style="color: #ff9900;">962</span>毫秒 ；有索引耗时<span style="color: #ff9900;">143</span>毫秒。</span></p>
<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp; totalDocsExamined（文档扫描条目）：无索引是<span style="color: #ff9900;">200万</span>条；有索引是<span style="color: #ff9900;">2000</span>条。</span></p>
<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp; stage（查询的类型）：无索引是<span style="color: #ff9900;">COLLSCAN</span>（全表扫描）；有索引是<span style="color: #ff9900;">FETCH+IXSCAN</span>（索引扫描+根据索引去检索指定document）。</span></p>
<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp; executionStages.executionTimeMillisEstimate（检索document获得数据的耗时）：无索引耗时<span style="color: #ff9900;">910</span>毫秒；有索引耗时<span style="color: #ff9900;">0</span>毫秒。</span></p>
<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp; 建好索引后，这个query整体的速度<span style="color: #ff9900;">提高了1个数量级 <span style="color: #000000;">（1个数量级是10倍的意思）。<span style="color: #ff0000;">根据查询语句的不同，索引可以使速度提高几个数量级</span>。</span></span></span></p>

二、复合索引				{#two}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; 在多个键上建立的索引就是复合索</span><span style="font-size: 16px;">引，</span><span style="font-size: 16px;">有时候我们的查询不是单条件的，可能是多条件，比如查找年龄在20~30名字叫&lsquo;ryan1&rsquo;的同学，那么我们可以建立&ldquo;age&rdquo;和&ldquo;name&rdquo;</span><span style="font-size: 16px; line-height: 1.5;">的联合索引来加速查询。</span></p>
<p><span style="font-size: 16px; line-height: 1.5;">&nbsp; &nbsp; 为了演示索引的效果，我们来重新生成插入一份200万个文档的集合。</span></p>
<div class="cnblogs_code">
<pre><span style="color: #008080;"> 1</span> <span style="color: #008000;">//</span><span style="color: #008000;">删除原来的集合</span>
<span style="color: #008080;"> 2</span> <span style="color: #000000;">db.person.drop();
</span><span style="color: #008080;"> 3</span> 
<span style="color: #008080;"> 4</span> <span style="color: #008000;">//</span><span style="color: #008000;">插入200万条数据</span>
<span style="color: #008080;"> 5</span> <span style="color: #0000ff;">for</span>(<span style="color: #0000ff;">var</span> i=0;i&lt;2000000;i++<span style="color: #000000;">){
</span><span style="color: #008080;"> 6</span>      db.person.insert({"name":"ryan"+i%1000,"age":20+i%10<span style="color: #000000;">});
</span><span style="color: #008080;"> 7</span> <span style="color: #000000;">}
</span><span style="color: #008080;"> 8</span> 
<span style="color: #008080;"> 9</span> <span style="color: #008000;">//</span><span style="color: #008000;">创建三个索引</span>
<span style="color: #008080;">10</span> db.person.ensureIndex({"age":1<span style="color: #000000;">})
</span><span style="color: #008080;">11</span> db.person.ensureIndex({"name":1,"age":1<span style="color: #000000;">})
</span><span style="color: #008080;">12</span> db.person.ensureIndex({"age":1,"name":1})</pre>
</div>
<p><span style="font-size: 16px; line-height: 36px;">&nbsp; &nbsp; 我们可以用hint()方法来强制查询走哪个索引。</span></p>
<p><span style="font-size: 16px; line-height: 36px;">&nbsp; &nbsp; 我们来看一下，<span style="color: #ff9900;">当查询条件是多个的时候，复合索引相比单键索引的强大魅力</span>。</span><span style="font-size: 16px; line-height: 36px;">&nbsp; &nbsp;&nbsp;</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.find({"age":{"$gte":20,"$lte":30},"name":"ryan1"}).hint({"age":1}).explain("executionStats");</span></p>
<div class="cnblogs_code">
<pre><span style="color: #008080;"> 1</span> <span style="color: #000000;">{
</span><span style="color: #008080;"> 2</span> <span style="color: #000000;">    ...
</span><span style="color: #008080;"> 3</span>     "executionStats"<span style="color: #000000;"> : {
</span><span style="color: #008080;"> 4</span>         "executionSuccess" : <span style="color: #0000ff;">true</span><span style="color: #000000;">,
</span><span style="color: #008080;"> 5</span>         "nReturned" : 2000<span style="color: #000000;">,
</span><span style="color: #008080;"> 6</span>         "executionTimeMillis" : 2031<span style="color: #000000;">,
</span><span style="color: #008080;"> 7</span>         "totalKeysExamined" : 2000000<span style="color: #000000;">,
</span><span style="color: #008080;"> 8</span>         "totalDocsExamined" : 2000000<span style="color: #000000;">,
</span><span style="color: #008080;"> 9</span> <span style="color: #000000;">    ...    
</span><span style="color: #008080;">10</span> }</pre>
</div>
<p>&nbsp;</p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.find({"age":{"$gte":20,"$lte":30},"name":"ryan1"}).hint({"age":1,"name":1}).explain("executionStats");</span></p>
<div class="cnblogs_code">
<pre><span style="color: #008080;"> 1</span> <span style="color: #000000;">{
</span><span style="color: #008080;"> 2</span> <span style="color: #000000;">    ...
</span><span style="color: #008080;"> 3</span>     "executionStats"<span style="color: #000000;"> : {
</span><span style="color: #008080;"> 4</span>         "executionSuccess" : <span style="color: #0000ff;">true</span><span style="color: #000000;">,
</span><span style="color: #008080;"> 5</span>         "nReturned" : 2000<span style="color: #000000;">,
</span><span style="color: #008080;"> 6</span>         "executionTimeMillis" : 8<span style="color: #000000;">,
</span><span style="color: #008080;"> 7</span>         "totalKeysExamined" : 2010<span style="color: #000000;">,
</span><span style="color: #008080;"> 8</span>         "totalDocsExamined" : 2000<span style="color: #000000;">,
</span><span style="color: #008080;"> 9</span> <span style="color: #000000;">    ...
</span><span style="color: #008080;">10</span> }</pre>
</div>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 从</span><span style="font-size: 16px;">executionTimeMillis的值上，一眼就可以看出却别。单间索引耗费了2031毫秒，复合索引用了8毫秒。</span><span style="font-size: 16px;">&nbsp;由此我们可以看出，根据查询语句的不同，建立正确的索引是非常重要的，<span style="color: #ff9900;">对于查询语句中是多条件的，应多考虑复合索引的应用</span>。</span></p>
<p>&nbsp;</p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 下面，我们再说一种复合索引的重要应用情况。有对一个键排序并只要前100个结果的情景（实际项目中经常都是这种情景）。对于这种情况，索引应该这样建<span style="color: #ff9900;">{"sortKey":1,"queryCriteria":1}</span>，排序的键应该放在复合索引的第一位。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.find({"age":{"$gte":21.0,"$lte":30.0}}).sort({"name":1}).limit(100).hint({"age":1,"name":1}).explain("executionStats");</span></p>
<div class="cnblogs_code">
<pre><span style="color: #008080;"> 1</span> <span style="color: #000000;">{
</span><span style="color: #008080;"> 2</span> <span style="color: #000000;">    ...
</span><span style="color: #008080;"> 3</span> "executionStats"<span style="color: #000000;"> : {
</span><span style="color: #008080;"> 4</span>         "executionSuccess" : <span style="color: #0000ff;">true</span><span style="color: #000000;">,
</span><span style="color: #008080;"> 5</span>         "nReturned" : 100<span style="color: #000000;">,
</span><span style="color: #008080;"> 6</span>         "executionTimeMillis" : 6882<span style="color: #000000;">,
</span><span style="color: #008080;"> 7</span>         "totalKeysExamined" : 1800000<span style="color: #000000;">,
</span><span style="color: #008080;"> 8</span>         "totalDocsExamined" : 1800000<span style="color: #000000;">,
</span><span style="color: #008080;"> 9</span> <span style="color: #000000;">    ...  
</span><span style="color: #008080;">10</span> }</pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.find({"age":{"$gte":21.0,"$lte":30.0}}).sort({"name":1}).limit(100).hint({"name":1,"age":1}).explain("executionStats");</span></p>
<div class="cnblogs_code">
<pre><span style="color: #008080;"> 1</span> <span style="color: #000000;">{
</span><span style="color: #008080;"> 2</span> <span style="color: #000000;">    ...
</span><span style="color: #008080;"> 3</span>     "executionStats"<span style="color: #000000;"> : {
</span><span style="color: #008080;"> 4</span>         "executionSuccess" : <span style="color: #0000ff;">true</span><span style="color: #000000;">,
</span><span style="color: #008080;"> 5</span>         "nReturned" : 100<span style="color: #000000;">,
</span><span style="color: #008080;"> 6</span>         "executionTimeMillis" : 3<span style="color: #000000;">,
</span><span style="color: #008080;"> 7</span>         "totalKeysExamined" : 2100<span style="color: #000000;">,
</span><span style="color: #008080;"> 8</span>         "totalDocsExamined" : 2100<span style="color: #000000;">,
</span><span style="color: #008080;"> 9</span> <span style="color: #000000;">    ...
</span><span style="color: #008080;">10</span> }</pre>
</div>
<p><span style="font-size: 16px; line-height: 24px;">&nbsp; &nbsp; 从上面的结果，我们很容易看出，基于排序键的索引，效果非常好。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 分析：<span style="color: #ff6600;">第一种索引，需要找到所有复合查询条件的值（依据索引，键和文档可以快速找到），但是找到后，需要对文档在内存中进行排序，这个步骤消耗了非常多的时间。</span></span><span style="font-size: 16px; color: #ff6600;">第二种索引，效果非常好，因为不需要在内存中对大量数据进行排序。但是，MongoDB不得不扫描整个索引以便找到所有文档。因此，如果对查询结果的范围做了限制，那么MongoDB在几次匹配之后就可以不再扫描索引，在这种情况下，将排序键放在第一位是一个非常好的策略。</span>&nbsp;</p>

三、唯一索引				{#three}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; 唯一索引可以确保集合的每个文档的指定键都有唯一值。如果想保证不同文档的&ldquo;name&rdquo;键拥有不同的值，在&ldquo;name&rdquo;键上创建一个唯一索引就可以了。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.ensureIndex({"name":1},{"unique":true});</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 然后用db.person.getIndexes()命令，查看目前person集合所有的索引。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201606/440176-20160604223839805-546725729.png" alt="" /></span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 也可以创建复合的唯一索引。创建复合唯一索引时，<span style="color: #ff9900;">单个键的值可以相同，但所有键的组合值必须是唯一的</span>。&nbsp;</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.ensureIndex({"name":1,"age":1},{"unique":true});&nbsp;</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201606/440176-20160604225320774-1334049303.png" alt="" /></span></p>

四、稀疏索引				{#four}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; 唯一索引会把null看作值，所以无法将多个缺少唯一索引中的键的文档插入到集合中。然而，在有些情况下，你可能希望唯一索引只对包含相应键的文档生效。这个时候我们可以用到MongoDB中的稀疏索引。该索引与关系型数据库中的稀疏索引是完全不同的概念。<span style="color: #ff9900;">MongoDB中的稀疏索引只是不需要将每个文档都作为索引条目</span>。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 比如，如果有一个可选的mobilephone字段，但是，如果提供了这个字段，那么它的值必须是唯一的：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.ensureIndex({"mobilephone":1}{"unique":true,"sparse":true});</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #ff9900;">稀疏索引不必是唯一的</span>。只要去掉unique选项，就可以创建一个非唯一的稀疏索引。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201606/440176-20160604231022633-910012774.png" alt="" /></span></p>

五、索引管理				{#five}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; 如第一小节所述，可以使用<span style="color: #ff9900;">ensureIndex</span>方法创建新的索引，也可以使用<span style="color: #ff9900;">createIndex</span>方法。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 创建一个索引之后，可以利用<span style="color: #ff9900;">getIndexes</span>()方法来查看给定集合上的所有索引的信息。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.getIndexes();得到的结果如下所示。</span></p>
<div class="cnblogs_code" onclick="cnblogs_code_show('3bbe90c9-0c38-4cd2-b42d-20660cb1fb2c')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_3bbe90c9-0c38-4cd2-b42d-20660cb1fb2c" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_3bbe90c9-0c38-4cd2-b42d-20660cb1fb2c" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_3bbe90c9-0c38-4cd2-b42d-20660cb1fb2c" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span> <span style="color: #000000;">[
</span><span style="color: #008080;"> 2</span> <span style="color: #000000;">    {
</span><span style="color: #008080;"> 3</span>         "v" : 1<span style="color: #000000;">,
</span><span style="color: #008080;"> 4</span>         "key"<span style="color: #000000;"> : {
</span><span style="color: #008080;"> 5</span>             "_id" : 1
<span style="color: #008080;"> 6</span> <span style="color: #000000;">        },
</span><span style="color: #008080;"> 7</span>         "name" : "_id_"<span style="color: #000000;">,
</span><span style="color: #008080;"> 8</span>         "ns" : "personmap.person"
<span style="color: #008080;"> 9</span> <span style="color: #000000;">    },
</span><span style="color: #008080;">10</span> <span style="color: #000000;">    {
</span><span style="color: #008080;">11</span>         "v" : 1<span style="color: #000000;">,
</span><span style="color: #008080;">12</span>         "unique" : <span style="color: #0000ff;">true</span><span style="color: #000000;">,
</span><span style="color: #008080;">13</span>         "key"<span style="color: #000000;"> : {
</span><span style="color: #008080;">14</span>             "name" : 1.0
<span style="color: #008080;">15</span> <span style="color: #000000;">        },
</span><span style="color: #008080;">16</span>         "name" : "name_1"<span style="color: #000000;">,
</span><span style="color: #008080;">17</span>         "ns" : "personmap.person"
<span style="color: #008080;">18</span> <span style="color: #000000;">    },
</span><span style="color: #008080;">19</span> <span style="color: #000000;">    {
</span><span style="color: #008080;">20</span>         "v" : 1<span style="color: #000000;">,
</span><span style="color: #008080;">21</span>         "unique" : <span style="color: #0000ff;">true</span><span style="color: #000000;">,
</span><span style="color: #008080;">22</span>         "key"<span style="color: #000000;"> : {
</span><span style="color: #008080;">23</span>             "name" : 1.0<span style="color: #000000;">,
</span><span style="color: #008080;">24</span>             "age" : 1.0
<span style="color: #008080;">25</span> <span style="color: #000000;">        },
</span><span style="color: #008080;">26</span>         "name" : "name_1_age_1"<span style="color: #000000;">,
</span><span style="color: #008080;">27</span>         "ns" : "personmap.person"
<span style="color: #008080;">28</span> <span style="color: #000000;">    },
</span><span style="color: #008080;">29</span> <span style="color: #000000;">    {
</span><span style="color: #008080;">30</span>         "v" : 1<span style="color: #000000;">,
</span><span style="color: #008080;">31</span>         "unique" : <span style="color: #0000ff;">true</span><span style="color: #000000;">,
</span><span style="color: #008080;">32</span>         "key"<span style="color: #000000;"> : {
</span><span style="color: #008080;">33</span>             "mobilephone" : 1.0
<span style="color: #008080;">34</span> <span style="color: #000000;">        },
</span><span style="color: #008080;">35</span>         "name" : "mobilephone_1"<span style="color: #000000;">,
</span><span style="color: #008080;">36</span>         "ns" : "personmap.person"<span style="color: #000000;">,
</span><span style="color: #008080;">37</span>         "sparse" : <span style="color: #0000ff;">true</span>
<span style="color: #008080;">38</span> <span style="color: #000000;">    }
</span><span style="color: #008080;">39</span> ]</pre>
</div>
<span class="cnblogs_code_collapse">db.person.getIndexes()方法的结果</span></div>
<p>&nbsp;</p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 随着业务的不断变化，你可能会发现数据或者查询已经发生了改变，原来的索引也不那么好用了。这时可以使用dropIndex()方法删除不需要的索引：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.<span style="color: #ff9900;">dropIndex</span>("name_1");//删除索引名为name_1的索引。</span></p>
<p>&nbsp;</p>

致谢！             {#thanks}
===========================

<p>&nbsp; &nbsp; &nbsp; &nbsp;至此，我们完成了开发编译调试到最终上线生产运行。喜欢请关注公众号--程序猿牧场，谢谢！</p>
<p>&nbsp;<img src="https://img2020.cnblogs.com/blog/440176/202003/440176-20200316224639427-719739537.png" alt="" /></p>
