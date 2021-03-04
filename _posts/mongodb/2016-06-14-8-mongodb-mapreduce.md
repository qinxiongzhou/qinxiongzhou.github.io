---
layout: post
title:  "玩转mongodb（八）：分布式计算--MapReduce"
date:   2016-06-14 15:56:34 +0800
categories: 玩转mongodb
---
* content
{:toc}

简介				{#introduction}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; MongoDB提供了MapReduce的聚合工具来实现任意复杂的逻辑，它非常强大，非常灵活。MapReduce使用JavaScript作为&ldquo;查询语言&rdquo;，能够在多台服务器之间并行执行。它会将一个大问题拆分为多个小问题，将各个小问题发送到不同的机器上，每台机器只负责完成一部分工作。所有机器都完成时，再将这些零碎的解决方案合并为一个完整的解决方案。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 在一般情况下，MapReduce操作有2个阶段：第一个阶段是映射（map）阶段，处理每一个<span style="color: #99cc00;">符合要求的文档</span>（即每个符合要求的文档都执行一次map的方法），然后利用emit函数产生一些键和这些键对应的多个值（最后组成一个列表）。第二个阶段是化简（reduce）阶段，把列表中的值化简成一个单值。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; MapReduce使用自定义JavaScript函数执行map和reduce操作，具有极大的灵活性，但这种强大是有代价的，MapReduce非常慢，不应该用在实时的数据分析中。</span></p>

例子				{#example}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; 下面来看一个例子：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 主要的功能：计算出每个用户的状态为A的订单的总额。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp;&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201606/440176-20160612143257777-1582323552.png" alt="" /></span></p>
<p>&nbsp;</p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 首先查找所有订单（如果mongodb有进行分片，则每个分片的订单都会找出来）状态为&ldquo;A&rdquo;的订单。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 然后每个订单都会执行map的方法，map方法主要是输出以cust_id为key，amount为value的一个键值对。紧跟着的一个步骤，是把所有相同的key的所有value，组成一个数组，传给后面的reduce。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 最后的reduce步骤，是把由map传回来的key/value的value进行求和，得到最终以每个用户（cust_id）为key，所有金额求和的值为value的结果。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; reduce步骤产生的结果，放在&ldquo;order_totals&rdquo;这个集合中。可以用db.order_totals.find()来查看这整个MapReduce的结果</span></p>
<p>&nbsp;</p>


致谢！             {#thanks}
===========================

<p>&nbsp; &nbsp; &nbsp; &nbsp;至此，我们完成了开发编译调试到最终上线生产运行。喜欢请关注公众号--程序猿牧场，谢谢！</p>
<p>&nbsp;<img src="https://img2020.cnblogs.com/blog/440176/202003/440176-20200316224639427-719739537.png" alt="" /></p> 