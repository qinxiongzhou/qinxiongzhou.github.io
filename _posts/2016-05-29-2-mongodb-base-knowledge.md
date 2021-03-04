---
layout: post
title:  "玩转mongodb（二）：mongodb基础知识"
date:   2016-05-29 16:13:38 +0800
categories: 玩转mongodb
---
* content
{:toc}

常用基本数据类型				{#base}
=============================

<ul>
<li><span style="font-size: 16px;">null</span></li>
</ul>
<p><span style="font-size: 16px;">&nbsp; &nbsp; null用于表示空值或者不存在的字段：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; {"data":null}</span></p>
<ul>
<li><span style="font-size: 16px;">布尔型</span></li>
</ul>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 布尔类型只有两个值，true和false：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; {"data":true}、{"data":false}</span></p>
<ul>
<li><span style="font-size: 16px;">字符串</span></li>
</ul>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 字符串类型的数据是由UTF-8字符组成：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; {"data":"pingan"}</span></p>
<ul>
<li><span style="font-size: 16px;">正则表达式</span></li>
</ul>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 查询时，使用正则表达式作为限定条件，语法和javascript的正则表达式一样：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; {"data":/pingan/i}</span></p>
<ul>
<li><span style="font-size: 16px;">对象id</span></li>
</ul>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 对象id是一个12字节（24字符）的ID，是文档的唯一标识。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; {"data":ObjectId()}</span></p>
<ul>
<li><span style="font-size: 16px;">数值</span></li>
</ul>
<p><span style="font-size: 16px;">&nbsp; &nbsp; shell默认使用64位的浮点型数值，即Double类型。对于整型值，可以使用NumberInt类（4字节带符号整数）或NumberLong类（8字节带符号整数）。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; {"data":3.33}，表示Double类型</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; {"data":NumberInt("3")}，表示Int类型</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; {"data":NumberLong("3")}，表示Long类型</span></p>
<ul>
<li><span style="font-size: 16px;">数组</span></li>
</ul>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 数据列表或者数据集都可以表示为数组。数组的元素可以是数值、字符串等等其他基本数据类型，元素之间用英文逗号分隔开。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; {"data":[1,2,3]}、{"data":["a","b","c"]}</span></p>
<ul>
<li><span style="font-size: 16px;">日期</span></li>
</ul>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 日期被存储为自新纪元以来经过的毫秒数，不存储时区：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; {"data":new Date()}</span></p>
<ul>
<li><span style="font-size: 16px;">内嵌文档</span></li>
</ul>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 文档可以嵌套其他文档，被嵌套的文档作为父文档的值：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; {"data":{"company":"pingan"}}</span></p>
<ul>
<li><span style="font-size: 16px;">二进制数据</span></li>
</ul>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 二进制数据是一个任意字节的字符串，要将非UTF-8字符保存到数据库中，二进制数据是唯一的方式。比如保存图片的数据。但是不能直接在shell中使用。</span></p>
<p>&nbsp;</p>
<div class="cnblogs_code" onclick="cnblogs_code_show('02ccec8e-6a9d-48ce-ab31-8592039c8d60')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_02ccec8e-6a9d-48ce-ab31-8592039c8d60" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_02ccec8e-6a9d-48ce-ab31-8592039c8d60" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_02ccec8e-6a9d-48ce-ab31-8592039c8d60" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span> <span style="color: #008000;">//</span><span style="color: #008000;">把图片存到mongodb中</span>
<span style="color: #008080;"> 2</span> <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">void</span> SaveImgMG(<span style="color: #0000ff;">byte</span><span style="color: #000000;">[] byteImg)
</span><span style="color: #008080;"> 3</span> <span style="color: #000000;">{
</span><span style="color: #008080;"> 4</span>     Document doc = <span style="color: #0000ff;">new</span><span style="color: #000000;"> Document();
</span><span style="color: #008080;"> 5</span>     doc["ID"] = 1<span style="color: #000000;">;
</span><span style="color: #008080;"> 6</span>     doc["Img"] =<span style="color: #000000;"> byteImg;
</span><span style="color: #008080;"> 7</span> <span style="color: #000000;">    mongoCollection.Save(doc);
</span><span style="color: #008080;"> 8</span> <span style="color: #000000;">}
</span><span style="color: #008080;"> 9</span> <span style="color: #008000;">//</span><span style="color: #008000;">获取mongodb存储的图片字节数据</span>
<span style="color: #008080;">10</span> <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">byte</span><span style="color: #000000;">[] GetImgMG()
</span><span style="color: #008080;">11</span> <span style="color: #000000;">{
</span><span style="color: #008080;">12</span>   Document doc=  mongoCollection.FindOne(<span style="color: #0000ff;">new</span> Document { { "ID", 1<span style="color: #000000;"> } });
</span><span style="color: #008080;">13</span>   <span style="color: #0000ff;">return</span> doc["Img"<span style="color: #000000;">] as Binary;
</span><span style="color: #008080;">14</span> }</pre>
</div>
<span class="cnblogs_code_collapse">View Code</span></div>
<p>&nbsp;</p>

文档				{#document}
=============================

<p>&nbsp; &nbsp; &nbsp;<span style="font-size: 16px;">文档就是键值对的一个有序集，是MongoDB中数据的基本单元，非常类似于关系型数据库管理系统中的行，但更具表现力。</span></p>
<div class="cnblogs_code">
<pre>1 var mydoc = {
2                _id: ObjectId("5099803df3f4948bd2f98391"),
3                name: { first: "Alan", last: "Turing" },
4                birth: new Date('Jun 23, 1912'),
5                death: new Date('Jun 07, 1954'),
6                contribs: [ "Turing machine", "Turing test", "Turingery" ],
7                views : NumberLong(1250000)
8             }</pre>
</div>

集合				{#set}
=============================

<p>&nbsp; &nbsp; <span style="font-size: 16px;">集合就是一组文档，如果将MongoDB中的一个文档比喻为关系型数据库中的一行，那么一个集合就相当于一张表的概念。</span></p>

数据库				{#database}
=============================

<p><span style="font-size: 18pt;"><strong></strong></span></p>
<p>&nbsp; &nbsp; <span style="font-size: 16px;">在MongoDB中，多个文档组成集合，而多个集合可以组成数据库，一个MongoDB实例，可以承载多个数据库，每个数据库拥有0个或者多个集合。MongoDB3.0这个版本中，有三个数据库名是保留的。分别是：admin、local、config。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; admin：从身份验证的角度来讲，这是&ldquo;root&rdquo;数据库。如果将一个新建的一个用户添加到admin数据库，这个用户就自动获得所有数据库的权限。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; local：这个数据库永远都不可以复制，且一台服务器上的所有本地集合都可以存储在这数据库中。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; config：MongoDB用于分片设置时，分片信息会存储在config数据库中。&nbsp;</span></p>

shell中的基本操作				{#shell}
=============================

<p>&nbsp; &nbsp;<span style="font-size: 16px;"> shell会用到4个基本的操作：创建、读取、更新和删除（即CRUD操作）。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 创建：</span></p>
<div class="cnblogs_code">
<pre>1 db.person.insert({"name":"ryan","age":26});
2 db.person.find({"name":"ryan"});</pre>
</div>
<p>&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201605/440176-20160529145701913-92984636.png" alt="" /></p>
<p>&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201605/440176-20160529145719553-976042858.png" alt="" /></p>
<p>&nbsp; &nbsp; <span style="font-size: 16px;">更新：使用update修改人员信息。update接受（至少）两个参数，第一个是限定条件（用于匹配待更新的文档），第二个是新的文档。</span></p>
<div class="cnblogs_code">
<pre>1 db.person.update({"name":"ryan"},{"name":"ryan","age":27});
2 db.person.find({"name":"ryan"});</pre>
</div>
<p>&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201605/440176-20160529150009038-75063722.png" alt="" /></p>
<p>&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201605/440176-20160529150028631-15185797.png" alt="" /></p>
<p>&nbsp; &nbsp; <span style="font-size: 18px;">删除：使用remove方法可将文档从数据库中永久删除。如果没有使用任何参数，它会将集合内的所有文档全部删除（甚用！！）。它可以接受一个作为限定条件的文档作为参数。</span></p>
<div class="cnblogs_code">
<pre>1 db.person.remove({"name":"ryan"});
2 db.person.find({"name":"ryan"});</pre>
</div>
<p>&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201605/440176-20160529150414959-1575269647.png" alt="" /></p>
<p>&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201605/440176-20160529150430944-1020948266.png" alt="" /></p>
<p>&nbsp;</p>

致谢！             {#thanks}
===========================

<p>&nbsp; &nbsp; &nbsp; &nbsp;至此，我们完成了开发编译调试到最终上线生产运行。喜欢请关注公众号--程序猿牧场，谢谢！</p>
<p>&nbsp;<img src="https://img2020.cnblogs.com/blog/440176/202003/440176-20200316224639427-719739537.png" alt="" /></p>