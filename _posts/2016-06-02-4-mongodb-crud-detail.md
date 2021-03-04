---
layout: post
title:  "玩转mongodb（四）：细说插入、更新、删除和查询"
date:   2016-06-02 20:19:34 +0800
categories: 玩转mongodb
---
* content
{:toc}

插入				{#insert}
=============================

<p>&nbsp;<span style="font-size: 16px;">&nbsp; &nbsp;使用insert或save方法想目标集合插入一个文档：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.insert({"name":"ryan","age":30});</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 使用batchInsert方法实现批量插入，它与insert方法非常类似，只是它接受的是一个文档数组作为参数。一次发送数十，数百乃至数千个文档会明显提高插入的速度。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.batchInsert([{"name":"ryan","age":30},{"name":"pitaya","age":2}]);</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 如果在批量插入的过程中有一个文档插入失败，那么在这个文档之前的所有文档都会成功插入到集合中，而这个文档以及之后的所有文档全部插入失败。如果希望batchInsert忽略错误并且继续执行后续插入，可以使用continueOnError选项。shell并不支持这个选项，但所有的驱动程序都支持。&nbsp;</span></p>

更新				{#update}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; 使用update方法来更新集合中的数据。update有四个参数，前两个参数是必须的。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.update({"name":"ryan"},{"$set":{"age":35}},true,true);</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 第一个参数：查询文档，用于定位需要更新的目标文档。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 第二个参数：修改器文档，用于说明要对找到的文档进行哪些修改。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 第三个参数：true表示要使用upsert，即如果没有找到符合更新条件的文档，就会以这个条件和更新文档为基础创建一个新的文档。如果找到了匹配的文档，则正常更新。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 第四个参数：true表示符合条件的所有文档，都要执行更新。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <strong>修改器：</strong></span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">$set</span>：用来指定一个字段的值。如果这个字段不存在，则创建它。对于更新而言，对符合更新条件的文档，修改执行的字段，不需要全部覆盖。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp;&nbsp;db.person.update({"name":"ryan"},{"<span style="color: #99cc00;">$set</span>":{"age":35}},true,true);</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">$inc</span>：用来增加已有键的值，或者该键不存在就创建一个。对于投票等有变化数值的场景，这个会非常方便。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.update({"name":"ryan"},{"<span style="color: #99cc00;">$inc</span>":{"age":2}},true,true);//对符合name等于ryan的文档，age字段加2。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">$push</span>：向已有数组末尾加入一个元素。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.update({"name":"ryan"},{"$set":{"language":["chinese"]}},true,true);//对符合name等于ryan的文档，添加一个language的数组</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.update({"name":"ryan"},{"<span style="color: #99cc00;">$push</span>":{"language":"english"}},true,true);//给数组的末尾添加一个值。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">$addToSet</span>：避免向数组插入重复的值。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.update({"name":"ryan"},{"<span style="color: #99cc00;">$addToSet</span>":{"language":"english"}},true,true);</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">$each</span>：与$push和$addToSet结合，一次给数组添加多个值。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp;&nbsp;db.person.update({"name":"ryan"},{"$push":{"language":{"<span style="color: #99cc00;">$each</span>":["Japanese","Portuguese"]}}},true,true);</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp;&nbsp;db.person.update({"name":"ryan"},{"$addToSet":{"language":{"<span style="color: #99cc00;">$each</span>":["Japanese","Portuguese"]}}},true,true);</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">$pop</span>：可以从数组的任何一端删除元素。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp;&nbsp;db.person.update({"name":"ryan"},{"<span style="color: #99cc00;">$pop</span>":{"language":1}},true,true);//从数组的末尾删除一个元素</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp;&nbsp;db.person.update({"name":"ryan"},{"<span style="color: #99cc00;">$pop</span>":{"language":-1}},true,true);//从数组的头部删除一个元素</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">$pull</span>：删除数组对应的值。全部删除。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.update({"name":"ryan"},{"<span style="color: #99cc00;">$pull</span>":{"language":"english"}},true,true);</span></p>

删除			{#delete}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; 使用remove方法删除集合中的数据。它可以接受一个查询文档作为可选参数。给定这个参数以后，只有符合条件的文档才能被删除。（<span style="color: #ff0000;">删除数据是永久性的，不能撤销，也不能恢复</span>）。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.remove({"name":"ryan"});//删除person集合中name字段的值等于ryan的所有文档。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.remove();//删除person集合中所有的文档。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 使用drop方法代替remove方法，可以大幅度提高删除数据的速度。但是这个方法不能指定任何限定条件。而且<span style="color: #ff0000;">整个集合都会被删除，包括索引等信息，甚用！！</span></span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.drop();</span></p>

查询			{#query}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; MongoDB中使用find方法来进行查询。查询就是返回一个集合中文档的子集，子集的范围从0个文档到整个集合。find方法接受两个参数。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 第一个参数决定了要返回哪些文档，参数的内容是查询的条件。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 第二个参数来指定想要的键（字段）。第二个参数存在的情况：键的值为1代表要显示，为0代表不显示。&ldquo;_id&rdquo;默认显示，其他默认不显示。第二个参数不存在的情况：所有字段默认显示。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.find({"name":"ryan"},{"name":1});</span></p>
<p>&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201606/440176-20160602151341664-1946235667.png" alt="" /></p>
<p>&nbsp;<span style="font-size: 16px;"> &nbsp; <strong>查询条件：</strong></span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">$lt</span>、<span style="color: #99cc00;">$lte</span>、<span style="color: #99cc00;">$gt</span>、<span style="color: #99cc00;">$gte</span>这四个，就是全部的比较操作符（<span style="color: #ff0000;">没有$eq这个操作符</span>），分别对应&lt;、&lt;=、&gt;、&gt;=。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp;&nbsp;db.person.find({"age":{"<span style="color: #99cc00;">$lt</span>":10}});</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 这里提供一段小脚本，插入10万条数据，做之后的测试用。</span></p>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> <span style="color: #0000ff;">for</span>(<span style="color: #0000ff;">var</span> i=0;i&lt;100000;i++<span style="color: #000000;">){
</span><span style="color: #008080;">2</span>     db.person.insert({"name":"ryan"+i,"age"<span style="color: #000000;">:i});
</span><span style="color: #008080;">3</span> }</pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">$in</span>、<span style="color: #99cc00;">$nin</span>，用来查询一个键的多个值。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.find({"age":{"<span style="color: #99cc00;">$in</span>":[1,3]}});//查询age等于1或3的文档。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.find({"age":{"<span style="color: #99cc00;">$nin</span>":[1,3]}});//查询age不等于1或3的文档。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">$or</span>，用来查询多个键的多个值。可以和$in等配合使用。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.find({"<span style="color: #99cc00;">$or</span>":[{"name":"ryan2"},{"age":3}]});//查询name等于ryan2 &nbsp; 或者 &nbsp; age等于3的文档。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">$exists</span>，查询的键对应是值是null的，默认会返回null和键不存在的文档。可以通过$exists来判断该键是否存在。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.find({"age":{"$in":[null],"<span style="color: #99cc00;">$exists</span>":true}});//查询age等于null，并且键是存在的文档。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">$where</span>，用它可以在查询中执行任意的javascript，这样就能在查询中做（几乎）任何事情。为了安全起见，应该严格限制或者消除"$where"语句的使用。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.find({"<span style="color: #99cc00;">$where</span>":function(){</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; &nbsp; &nbsp; ...;//这里可以是任意的javascript语句。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; }})</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">游标</span>：利用游标可以限制结果的数量，略过部分结果，根据任意键按任意顺序的组合对结果进行各种排序，或者是执行其他的一些强大的操作。</span></p>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> <span style="color: #0000ff;">var</span> cursor =<span style="color: #000000;"> db.person.find();
</span><span style="color: #008080;">2</span> <span style="color: #0000ff;">while</span><span style="color: #000000;">(cursor.hasNext()){
</span><span style="color: #008080;">3</span>     obj =<span style="color: #000000;"> cursor.next();
</span><span style="color: #008080;">4</span>     ...;<span style="color: #008000;">//</span><span style="color: #008000;">这里可以做任何事情。  </span>
<span style="color: #008080;">5</span> }</pre>
</div>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <strong>常用的shell：</strong></span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">limit</span>：只返回前面多少个结果。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.find().<span style="color: #99cc00;">limit</span>(2);//查询符合条件的文档，显示前两个文档。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">skip</span>：跳过多少个结果后显示剩余的。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.find().<span style="color: #99cc00;">skip</span>(2);//查询符合条件的文档，显示跳过2个文档后剩余的所有文档。&nbsp;</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <span style="color: #99cc00;">sort</span>：用于排序。接受一个对象（一组键值对）作为参数，键对应文档的键名，值代表排序的方向。排序的方向可以是1（升序）或者-1（降序）。如果指定了多个键，则按照这些键被指定的顺序逐个排序。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; db.person.find().<span style="color: #99cc00;">sort</span>({"name":1,"age":-1});//查询的结果，按照name升序，age降序来排序显示。</span></p>
<p>&nbsp;</p>



致谢！             {#thanks}
===========================

<p>&nbsp; &nbsp; &nbsp; &nbsp;至此，我们完成了开发编译调试到最终上线生产运行。喜欢请关注公众号--程序猿牧场，谢谢！</p>
<p>&nbsp;<img src="https://img2020.cnblogs.com/blog/440176/202003/440176-20200316224639427-719739537.png" alt="" /></p>
