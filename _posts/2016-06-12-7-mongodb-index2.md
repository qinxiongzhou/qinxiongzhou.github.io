---
layout: post
title:  "玩转mongodb（七）：索引，速度的引领（全文索引、地理空间索引）"
date:   2016-06-12 11:56:34 +0800
categories: 玩转mongodb
---
* content
{:toc}

简介				{#introduction}
=============================

<p><span style="font-size: 16px;">&nbsp;&nbsp;&nbsp;&nbsp;本篇博文主要介绍MongoDB中一些常用的特殊索引类型，主要包括：</span></p>
<ul>
<li><span style="font-size: 16px;">&nbsp;&nbsp;&nbsp;&nbsp;用于简单字符串搜索的全文本索引；</span></li>
<li><span style="font-size: 16px;">&nbsp;&nbsp;&nbsp;&nbsp;用于球体空间（2dsphere）和二维平面（2d）的地理空间索引。</span>&nbsp;</li>
</ul>

一、全文索引				{#one}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; MongoDB有一个特殊的索引用在文档中搜索文本，之前的博客都是用精确匹配来查询字符串，这些技术有一定的限制。在搜索大块文本的速度非常慢，而且无法处理自然语言礼节的问题。全文本索引使用的是&ldquo;倒排索引&rdquo;的思想来做的，和当前非常开源的lucene（全文检索，Apacle基金会下的开源项目）项目是一样的思想来做的。使用全文本索引可以非常快的进行文本搜索，MongoDB支持多种语言，可惜在免费版中，并不支持世界第一的火星文语言（汉语）。查MongoDB的官网可以看到，在企业版中是支持汉语的全文索引的。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 如果公司用的是免费版的MongoDB，而又需要用到中文的全文索引，建议使用lucene或者solr等开源项目来做。（没钱就得用技术来补，赤裸裸的现实。）</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 使用全文本检索需要专门开启这个功能才能进行使用。启动MongoDB时指定--setParameter textSearchEnabled=true选项，或者在运行时执行setParameter命令，都可以启用全文本索引。</span></p>
<div class="cnblogs_code">
<pre>db.adminCommand({"setParameter":1,"textSearchEnabled":<span style="color: #0000ff;">true</span>});</pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 准备10条数据：</span></p>
<div class="cnblogs_code" onclick="cnblogs_code_show('9c6e67fd-ceca-4cca-bd42-521346367b72')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_9c6e67fd-ceca-4cca-bd42-521346367b72" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_9c6e67fd-ceca-4cca-bd42-521346367b72" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_9c6e67fd-ceca-4cca-bd42-521346367b72" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span> db.news.insert({"title":"SEOUL","context":"SEOUL, June 10 (Reuters) - South Korean prosecutors raided the offices of Lotte Group, the country's fifth-largest conglomerate, and several affiliates on Friday, dealing a further blow to its hotel unit's planned IPO, billed as the world's biggest this year."<span style="color: #000000;">});
</span><span style="color: #008080;"> 2</span> db.news.insert({"title":"Many Chinese people","context":"Many Chinese people think that a job on a diplomatic team is too good to quit. So when 28-year-old Liu Xiaoxi left her embassy post as an attache late last year to start a career in photography, she quickly became a popular topic among the Chinese online community."<span style="color: #000000;">});
</span><span style="color: #008080;"> 3</span> db.news.insert({"title":"About","context":"About 200 investigators searched 17 locations including group headquarters in central Seoul and the homes of Chairman Shin Dong-bin and other key executives, local news agency Yonhap reported, citing the Seoul Central Prosecutor's office."<span style="color: #000000;">});
</span><span style="color: #008080;"> 4</span> db.news.insert({"title":"Three people","context":"Three people with direct knowledge of the matter told Reuters that Friday's raids were part of an investigation into a possible slush fund. They also declined to be identified."<span style="color: #000000;">});
</span><span style="color: #008080;"> 5</span> db.news.insert({"title":"A Lotte Group spokesman","context":"A Lotte Group spokesman on Friday declined to comment on the reason for the raid, when asked whether it concerned a possible slush fund. He noted, however, that the situation was difficult given the IPO plans and Lotte Chemical's Axiall bid."<span style="color: #000000;">});
</span><span style="color: #008080;"> 6</span> db.news.insert({"title":"According","context":"According to bourse rules, the deadline for Hotel Lotte to list is July 27, six months from the preliminary approval for the IPO. If it needed to refile its prospectus to warn investors about risks from Friday's probe, which appeared likely, it would probably not be able to meet that deadline, an exchange official told Reuters on Friday."<span style="color: #000000;">});
</span><span style="color: #008080;"> 7</span> db.news.insert({"title":"Friday","context":"On Friday, dozens of Chinese tourists queued as usual to access elevators to the flagship Lotte Duty Free outlet in the group's headquarters complex, as TV cameras waited for investigators to emerge from office doors around the corner."<span style="color: #000000;">});
</span><span style="color: #008080;"> 8</span> db.news.insert({"title":"Named","context":"Named after the heroine of an 18th century Goethe novel, Lotte has grown from its founding in Japan 68 years ago as a maker of chewing gum to a corporate giant with interests ranging from hotels and retail to food and chemicals. The group has annual revenue of around $60 billion in Korea."<span style="color: #000000;">});
</span><span style="color: #008080;"> 9</span> db.news.insert({"title":"Hotel Lotte's","context":"Hotel Lotte's planned flotation of around 35 percent of its shares was intended to bring transparency and improve corporate governance at a group whose ownership structure is convoluted even by the opaque standards of South Korea's conglomerates."<span style="color: #000000;">});
</span><span style="color: #008080;">10</span> db.news.insert({"title":"Shares","context":"Shares in Lotte Shopping (023530.KS) , whose units Lotte Department Store and Lotte Home Shopping were raided, fell 1.6 percent on Friday. Lotte Himart (071840.KS) , a consumer electronics retailer, dropped 2.1 percent."});</pre>
</div>
<span class="cnblogs_code_collapse">全文索引的数据准备</span></div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 一个集合上最多只能有一个全文本索引，但是全文本索引可以包含多个字段。全文索引与&ldquo;普通&rdquo;的多键索引不同，全文本索引中的字段顺序不重要：每个字段都被同等对待，可以为每个字段指定不同的权重来控制不同字段的相对重要性。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 我们来给title和context字段建立全文本索引，给title字段2的权重，context字段1的权重。（权重的范围可以是1~1,000,000,000，默认权重是1）。</span></p>
<div class="cnblogs_code">
<pre>db.news.ensureIndex({"title":"text","context":"text"},{"weights":{"title":2,"context":1}})</pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 我们利用这个全文本索引来搜索一下。搜索的内容是&ldquo;flotation&rdquo;。</span></p>
<div class="cnblogs_code">
<pre>db.news.find({$text:{$search:"flotation"}})</pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 结果如下图所示：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201606/440176-20160612000835011-427988557.png" alt="" /></span></p>
<p>&nbsp;</p>

二、2dsphere索引				{#two}
=============================

<p>&nbsp;<span style="font-size: 16px;">&nbsp; &nbsp;2dsphere索引是MongoDB最常用的地理空间索引之一，用于地球表面类型的地图。允许使用GeoJSON格式（http://www.geojson.org）指定点、线、多边形。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 点可以用形如[longitude,latitude]（[经度,纬度]）的两个元素的数组表示（<span style="color: #ff0000;">"loc"字段的名字可以是任意的，但是其中的子对象是有GeoJSON指定的，不能改变</span>）：</span></p>
<div class="cnblogs_code">
<pre><span style="color: #000000;">{
    </span>"name":"beijing"<span style="color: #000000;">,
    </span>"loc"<span style="color: #000000;">:{
        </span>"type":"<span style="color: #99cc00;">Point</span>"<span style="color: #000000;">,
        </span>"coordinates":[40,2<span style="color: #000000;">]
    }  
}</span></pre>
</div>
<p><span style="font-size: 16px; line-height: 1.5;">&nbsp; &nbsp; 线可以用一个由点组成的数组来表示：</span></p>
<div class="cnblogs_code">
<pre><span style="color: #000000;">{
    </span>"name":"changjiang"<span style="color: #000000;">,
    </span>"loc"<span style="color: #000000;">:{
        </span>"type":"<span style="color: #99cc00;">Line</span>"<span style="color: #000000;">,
        </span>"coordinates":[[1,2],[2,3],[3,4<span style="color: #000000;">]]
    }  
}</span></pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 多边形的表示方式与线一样，但是&ldquo;type&rdquo;不同：</span></p>
<div class="cnblogs_code">
<pre><span style="color: #000000;">{
    </span>"name":"shenzhen"<span style="color: #000000;">,
    </span>"loc"<span style="color: #000000;">:{
        </span>"type":"<span style="color: #99cc00;">Polygon</span>"<span style="color: #000000;">,
        </span>"coordinates":[[1,2],[2,3],[3,4<span style="color: #000000;">]]
    }  
}</span></pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 创建2dsphere索引：&nbsp;</span></p>
<div class="cnblogs_code">
<pre>db.mapinfo.ensureIndex({"loc":"2dsphere"})</pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 地理空间查询的类型有三种：<span style="color: #99cc00;">交集(intersection)</span>、<span style="color: #99cc00;">包含(within)</span>、<span style="color: #99cc00;">接近(nearness)</span>。查询时，需要将希望查找的内容指定为形如{"$geometry":geoJsonDesc}的GeoJSON对象。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 使用&ldquo;$geoIntersects&rdquo;查询位置相交的文档：</span></p>
<div class="cnblogs_code">
<pre><span style="color: #0000ff;">var</span> customMapinfo =<span style="color: #000000;"> {
    </span>"type":"Polygon"<span style="color: #000000;">,
    </span>"coordinates":[[12.2223,39,4424],[13.2223,38,4424],[13.2223,39,4424<span style="color: #000000;">]]
}

db.mapinfo.find({
    </span>"loc":{"<span style="color: #99cc00;">$geoIntersects</span>":{"$geometry"<span style="color: #000000;">:customMapinfo}} 
})</span></pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 这样就会找到所有与customMapinfo区域有交集的文档。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 使用&ldquo;$within&rdquo;查询完全包含在某个区域的文档：</span></p>
<div class="cnblogs_code">
<pre><span style="color: #000000;">db.mapinfo.find({
    </span>"loc":{"<span style="color: #99cc00;">$within</span>":{"$geometry"<span style="color: #000000;">:customMapinfo}} 
})</span></pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 使用&ldquo;$near&rdquo;查询附近的位置：</span></p>
<div class="cnblogs_code">
<pre><span style="color: #000000;">db.mapinfo.find({
    </span>"loc":{"<span style="color: #99cc00;">$within</span>":{"$geometry"<span style="color: #000000;">:customMapinfo}} 
})</span></pre>
</div>
<p>&nbsp;</p>


三、2d索引				{#three}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; 2d索引也是MongoDB最常用的地理空间索引之一，用于游戏地图。2d索引用于扁平表面，而不是球体表面。如果用在球体表面上，在极点附近会出现大量的扭曲变形。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 文档中应该使用包含两个元素的数组表示2d索引字段。</span></p>
<div class="cnblogs_code">
<pre><span style="color: #000000;">{
    </span>"name":"node1"<span style="color: #000000;">,
    </span>"tile":<span style="color: #99cc00;">[32,22</span><span style="color: #000000;"><span style="color: #99cc00;">]</span>
}</span></pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 创建索引</span></p>
<div class="cnblogs_code">
<pre>db.gameMapinfo.ensureIndex({"tile":"2d"})</pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 使用$near查询点[20,20]附近的文档：</span></p>
<div class="cnblogs_code">
<pre>db.gameMapinfo.find({"tile":{"<span style="color: #99cc00;">$near</span>":[20,20]}})</pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 使用$within查询出某个形状（矩形、圆形或者多边形）范围内的所有文档。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 矩形，可以指定$box选项（$box接受一个两元素的数组，第一个元素指定左下角的坐标，第二个元素指定右上角的坐标）：</span></p>
<div class="cnblogs_code">
<pre>db.gameMapinfo.find({"tile":{"<span style="color: #99cc00;">$within</span>":{"<span style="color: #99cc00;">$box</span>":[[10,20],[15,30]]}}})</pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 圆形，可以指定$center选项（$center接受一个两元素数组作为参数，第一个元素是一个点，用于指定圆心，第二个元素用于指定半径）：</span></p>
<div class="cnblogs_code">
<pre>db.gameMapinfo.find({"tile":{"<span style="color: #99cc00;">$within</span>":{"<span style="color: #99cc00;">$center</span>":[[12,12],5]}}})</pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 多边形，可以指定$polygon（$ploygon接受一个多元素的数组，每个元素对应多边形的点），下面以一个三角形为例：</span></p>
<div class="cnblogs_code">
<pre>db.gameMapinfo.find({"tile":{"<span style="color: #99cc00;">$within</span>":{"<span style="color: #99cc00;">$polygon</span>":[[0,20],[10,0],[-10,0]]}}})</pre>
</div>
<p>&nbsp;</p>

致谢！             {#thanks}
===========================

<p>&nbsp; &nbsp; &nbsp; &nbsp;至此，我们完成了开发编译调试到最终上线生产运行。喜欢请关注公众号--程序猿牧场，谢谢！</p>
<p>&nbsp;<img src="https://img2020.cnblogs.com/blog/440176/202003/440176-20200316224639427-719739537.png" alt="" /></p>
