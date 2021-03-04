---
layout: post
title:  "玩转mongodb（五）：mongodb 3.0+ 查询性能分析"
date:   2016-06-04 15:19:34 +0800
categories: 玩转mongodb
---
* content
{:toc}

mongodb性能分析方法：explain()				{#explain}
=============================

<p><span style="font-size: 16px; line-height: 1.5;">&nbsp; &nbsp; 为了演示的效果，我们先来创建一个有200万个文档的记录。（我自己的电脑耗了15分钟左右插入完成。如果你想插更多的文档也没问题，只要有耐心等就可以了。）</span></p>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> <span style="color: #0000ff;">for</span>(<span style="color: #0000ff;">var</span> i=0;i&lt;2000000;i++<span style="color: #000000;">){
</span><span style="color: #008080;">2</span>     db.person.insert({"name":"ryan"+i,"age"<span style="color: #000000;">:i});
</span><span style="color: #008080;">3</span> }</pre>
</div>
<p>&nbsp;<img src="http://images2015.cnblogs.com/blog/440176/201606/440176-20160603223717696-61563768.png" alt="" /></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; MongoDB 3.0之后，explain的返回与使用方法与之前版本有了很大的变化，介于3.0之后的优秀特色和我们目前所使用给的是3.0.7版本，本文仅针对MongoDB 3.0+的explain进行讨论。3.0+的explain有三种模式，分别是：queryPlanner、executionStats、allPlansExecution。现实开发中，常用的是executionStats模式，主要分析这种模式。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 给这个person集合创建age键的索引：db.person.createIndex({"age":1})</span></p>
<div class="cnblogs_code" onclick="cnblogs_code_show('b5272c77-f41e-494f-bbdc-1f254e0cf02f')"><img src="http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif" alt="" id="code_img_closed_b5272c77-f41e-494f-bbdc-1f254e0cf02f" class="code_img_closed" /><img src="http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif" alt="" id="code_img_opened_b5272c77-f41e-494f-bbdc-1f254e0cf02f" class="code_img_opened" style="display: none;" />
<div id="cnblogs_code_open_b5272c77-f41e-494f-bbdc-1f254e0cf02f" class="cnblogs_code_hide">
<pre><span style="color: #008080;"> 1</span> <span style="color: #000000;">{
</span><span style="color: #008080;"> 2</span>     "queryPlanner"<span style="color: #000000;"> : {
</span><span style="color: #008080;"> 3</span>         "plannerVersion" : 1<span style="color: #000000;">,
</span><span style="color: #008080;"> 4</span>         "namespace" : "personmap.person"<span style="color: #000000;">,
</span><span style="color: #008080;"> 5</span>         "indexFilterSet" : <span style="color: #0000ff;">false</span><span style="color: #000000;">,
</span><span style="color: #008080;"> 6</span>         "parsedQuery"<span style="color: #000000;"> : {
</span><span style="color: #008080;"> 7</span>             "age"<span style="color: #000000;"> : {
</span><span style="color: #008080;"> 8</span>                 "$lte" : 2000.0
<span style="color: #008080;"> 9</span> <span style="color: #000000;">            }
</span><span style="color: #008080;">10</span> <span style="color: #000000;">        },
</span><span style="color: #008080;">11</span>         "winningPlan"<span style="color: #000000;"> : {
</span><span style="color: #008080;">12</span>             "stage" : "FETCH"<span style="color: #000000;">,
</span><span style="color: #008080;">13</span>             "inputStage"<span style="color: #000000;"> : {
</span><span style="color: #008080;">14</span>                 "stage" : "IXSCAN"<span style="color: #000000;">,
</span><span style="color: #008080;">15</span>                 "keyPattern"<span style="color: #000000;"> : {
</span><span style="color: #008080;">16</span>                     "age" : 1.0
<span style="color: #008080;">17</span> <span style="color: #000000;">                },
</span><span style="color: #008080;">18</span>                 "indexName" : "age_1"<span style="color: #000000;">,
</span><span style="color: #008080;">19</span>                 "isMultiKey" : <span style="color: #0000ff;">false</span><span style="color: #000000;">,
</span><span style="color: #008080;">20</span>                 "direction" : "forward"<span style="color: #000000;">,
</span><span style="color: #008080;">21</span>                 "indexBounds"<span style="color: #000000;"> : {
</span><span style="color: #008080;">22</span>                     "age"<span style="color: #000000;"> : [ 
</span><span style="color: #008080;">23</span>                         "[-1.#INF, 2000.0]"
<span style="color: #008080;">24</span> <span style="color: #000000;">                    ]
</span><span style="color: #008080;">25</span> <span style="color: #000000;">                }
</span><span style="color: #008080;">26</span> <span style="color: #000000;">            }
</span><span style="color: #008080;">27</span> <span style="color: #000000;">        },
</span><span style="color: #008080;">28</span>         "rejectedPlans"<span style="color: #000000;"> : []
</span><span style="color: #008080;">29</span> <span style="color: #000000;">    },
</span><span style="color: #008080;">30</span>     "executionStats"<span style="color: #000000;"> : {
</span><span style="color: #008080;">31</span>         "executionSuccess" : <span style="color: #0000ff;">true</span><span style="color: #000000;">,
</span><span style="color: #008080;">32</span>         "nReturned" : 2001<span style="color: #000000;">,
</span><span style="color: #008080;">33</span>         "executionTimeMillis" : 143<span style="color: #000000;">,
</span><span style="color: #008080;">34</span>         "totalKeysExamined" : 2001<span style="color: #000000;">,
</span><span style="color: #008080;">35</span>         "totalDocsExamined" : 2001<span style="color: #000000;">,
</span><span style="color: #008080;">36</span>         "executionStages"<span style="color: #000000;"> : {
</span><span style="color: #008080;">37</span>             "stage" : "FETCH"<span style="color: #000000;">,
</span><span style="color: #008080;">38</span>             "nReturned" : 2001<span style="color: #000000;">,
</span><span style="color: #008080;">39</span>             "executionTimeMillisEstimate" : 0<span style="color: #000000;">,
</span><span style="color: #008080;">40</span>             "works" : 2002<span style="color: #000000;">,
</span><span style="color: #008080;">41</span>             "advanced" : 2001<span style="color: #000000;">,
</span><span style="color: #008080;">42</span>             "needTime" : 0<span style="color: #000000;">,
</span><span style="color: #008080;">43</span>             "needFetch" : 0<span style="color: #000000;">,
</span><span style="color: #008080;">44</span>             "saveState" : 16<span style="color: #000000;">,
</span><span style="color: #008080;">45</span>             "restoreState" : 16<span style="color: #000000;">,
</span><span style="color: #008080;">46</span>             "isEOF" : 1<span style="color: #000000;">,
</span><span style="color: #008080;">47</span>             "invalidates" : 0<span style="color: #000000;">,
</span><span style="color: #008080;">48</span>             "docsExamined" : 2001<span style="color: #000000;">,
</span><span style="color: #008080;">49</span>             "alreadyHasObj" : 0<span style="color: #000000;">,
</span><span style="color: #008080;">50</span>             "inputStage"<span style="color: #000000;"> : {
</span><span style="color: #008080;">51</span>                 "stage" : "IXSCAN"<span style="color: #000000;">,
</span><span style="color: #008080;">52</span>                 "nReturned" : 2001<span style="color: #000000;">,
</span><span style="color: #008080;">53</span>                 "executionTimeMillisEstimate" : 0<span style="color: #000000;">,
</span><span style="color: #008080;">54</span>                 "works" : 2002<span style="color: #000000;">,
</span><span style="color: #008080;">55</span>                 "advanced" : 2001<span style="color: #000000;">,
</span><span style="color: #008080;">56</span>                 "needTime" : 0<span style="color: #000000;">,
</span><span style="color: #008080;">57</span>                 "needFetch" : 0<span style="color: #000000;">,
</span><span style="color: #008080;">58</span>                 "saveState" : 16<span style="color: #000000;">,
</span><span style="color: #008080;">59</span>                 "restoreState" : 16<span style="color: #000000;">,
</span><span style="color: #008080;">60</span>                 "isEOF" : 1<span style="color: #000000;">,
</span><span style="color: #008080;">61</span>                 "invalidates" : 0<span style="color: #000000;">,
</span><span style="color: #008080;">62</span>                 "keyPattern"<span style="color: #000000;"> : {
</span><span style="color: #008080;">63</span>                     "age" : 1.0
<span style="color: #008080;">64</span> <span style="color: #000000;">                },
</span><span style="color: #008080;">65</span>                 "indexName" : "age_1"<span style="color: #000000;">,
</span><span style="color: #008080;">66</span>                 "isMultiKey" : <span style="color: #0000ff;">false</span><span style="color: #000000;">,
</span><span style="color: #008080;">67</span>                 "direction" : "forward"<span style="color: #000000;">,
</span><span style="color: #008080;">68</span>                 "indexBounds"<span style="color: #000000;"> : {
</span><span style="color: #008080;">69</span>                     "age"<span style="color: #000000;"> : [ 
</span><span style="color: #008080;">70</span>                         "[-1.#INF, 2000.0]"
<span style="color: #008080;">71</span> <span style="color: #000000;">                    ]
</span><span style="color: #008080;">72</span> <span style="color: #000000;">                },
</span><span style="color: #008080;">73</span>                 "keysExamined" : 2001<span style="color: #000000;">,
</span><span style="color: #008080;">74</span>                 "dupsTested" : 0<span style="color: #000000;">,
</span><span style="color: #008080;">75</span>                 "dupsDropped" : 0<span style="color: #000000;">,
</span><span style="color: #008080;">76</span>                 "seenInvalidated" : 0<span style="color: #000000;">,
</span><span style="color: #008080;">77</span>                 "matchTested" : 0
<span style="color: #008080;">78</span> <span style="color: #000000;">            }
</span><span style="color: #008080;">79</span> <span style="color: #000000;">        }
</span><span style="color: #008080;">80</span> <span style="color: #000000;">    },
</span><span style="color: #008080;">81</span>     "serverInfo"<span style="color: #000000;"> : {
</span><span style="color: #008080;">82</span>         "host" : "qinxiongzhou"<span style="color: #000000;">,
</span><span style="color: #008080;">83</span>         "port" : 27017<span style="color: #000000;">,
</span><span style="color: #008080;">84</span>         "version" : "3.0.7"<span style="color: #000000;">,
</span><span style="color: #008080;">85</span>         "gitVersion" : "6ce7cbe8c6b899552dadd907604559806aa2e9bd"
<span style="color: #008080;">86</span> <span style="color: #000000;">    },
</span><span style="color: #008080;">87</span>     "ok" : 1.0
<span style="color: #008080;">88</span> }</pre>
</div>
<span class="cnblogs_code_collapse">db.getCollection('person').find({"age":{"$lte":2000}}).explain("executionStats")</span></div>

对queryPlanner分析				{#queryplanner}
=============================

<p><span style="font-size: 16px;">&nbsp; &nbsp; queryPlanner:&nbsp;queryPlanner的返回</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; queryPlanner.namespace:该值返回的是该query所查询的表</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; queryPlanner.indexFilterSet:针对该query是否有indexfilter</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; queryPlanner.winningPlan:查询优化器针对该query所返回的最优执行计划的详细内容。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; queryPlanner.winningPlan.stage:最优执行计划的stage，这里返回是FETCH，可以理解为通过返回的index位置去检索具体的文档（stage有数个模式，将在后文中进行详解）。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; queryPlanner.winningPlan.inputStage:用来描述子stage，并且为其父stage提供文档和索引关键字。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; queryPlanner.winningPlan.stage的child&nbsp;stage，此处是IXSCAN，表示进行的是index&nbsp;scanning。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; queryPlanner.winningPlan.keyPattern:所扫描的index内容，此处是did:1,status:1,modify_time:&nbsp;-1与scid&nbsp;:&nbsp;1</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; queryPlanner.winningPlan.indexName：winning&nbsp;plan所选用的index。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; queryPlanner.winningPlan.isMultiKey是否是Multikey，此处返回是false，如果索引建立在array上，此处将是true。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; queryPlanner.winningPlan.direction：此query的查询顺序，此处是forward，如果用了.sort({modify_time:-1})将显示backward。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; queryPlanner.winningPlan.indexBounds:winningplan所扫描的索引范围,如果没有制定范围就是[MaxKey,&nbsp;MinKey]，这主要是直接定位到mongodb的chunck中去查找数据，加快数据读取。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; queryPlanner.rejectedPlans：其他执行计划（非最优而被查询优化器reject的）的详细返回，其中具体信息与winningPlan的返回中意义相同，故不在此赘述。</span></p>

对executionStats返回逐层分析				{#executionStats}
=============================

<p><span style="font-size: 18pt;"><strong><span style="line-height: 1.5;"></span></strong></span></p>

<p><strong><span style="font-size: 16px; line-height: 1.5;">&nbsp; &nbsp; 第一层，executionTimeMillis</span></strong></p>
<p><span style="font-size: 16px; line-height: 1.5;">&nbsp; &nbsp; 最为直观explain返回值是executionTimeMillis值，指的是我们这条语句的执行时间，这个值当然是希望越少越好。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 其中有3个executionTimeMillis，分别是：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; executionStats.executionTimeMillis</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 该query的整体查询时间。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; executionStats.executionStages.</span><span style="font-size: 16px;">executionTimeMillisEstimate</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 该查询根据index去检索document获得2001条数据的时间。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; executionStats.executionStages.inputStage.executionTimeMillisEstimate</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 该查询扫描2001行index所用时间。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <strong>第二层，index与document扫描数与查询返回条目数</strong></span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 这个主要讨论3个返回项，nReturned、totalKeysExamined、totalDocsExamined，分别代表该条查询返回的条目、索引扫描条目、文档扫描条目。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 这些都是直观地影响到executionTimeMillis，我们需要扫描的越少速度越快。</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 对于一个查询，我们最理想的状态是：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; nReturned=totalKeysExamined=totalDocsExamined</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; <strong>第三层，stage状态分析</strong></span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 那么又是什么影响到了totalKeysExamined和totalDocsExamined？是stage的类型。类型列举如下：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; COLLSCAN：全表扫描</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; IXSCAN：索引扫描</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; FETCH：根据索引去检索指定document</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; SHARD_MERGE：将各个分片返回数据进行merge</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; SORT：表明在内存中进行了排序</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; LIMIT：使用limit限制返回数</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; SKIP：使用skip进行跳过<br /></span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; IDHACK：针对_id进行查询</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; SHARDING_FILTER：通过mongos对分片数据进行查询</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; COUNT：利用db.coll.explain().count()之类进行count运算</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; COUNTSCAN：count不使用Index进行count时的stage返回</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; COUNT_SCAN：count使用了Index进行count时的stage返回</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; SUBPLA：未使用到索引的$or查询的stage返回</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; TEXT：使用全文索引进行查询时候的stage返回</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; PROJECTION：限定返回字段时候stage的返回</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 对于普通查询，我希望看到stage的组合(<span style="color: #ff0000;">查询的时候尽可能用上索引</span>)：</span></p>
<p><span style="font-size: 16px; color: #ff0000;">&nbsp; &nbsp; Fetch+IDHACK</span></p>
<p><span style="font-size: 16px; color: #ff0000;">&nbsp; &nbsp; Fetch+ixscan</span></p>
<p><span style="font-size: 16px; color: #ff0000;">&nbsp; &nbsp; Limit+（Fetch+ixscan）</span></p>
<p><span style="font-size: 16px; color: #ff0000;">&nbsp; &nbsp; PROJECTION+ixscan</span></p>
<p><span style="font-size: 16px; color: #ff0000;">&nbsp; &nbsp; SHARDING_FITER+ixscan</span></p>
<p><span style="font-size: 16px; color: #ff0000;">&nbsp; &nbsp; COUNT_SCAN</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; 不希望看到包含如下的stage：</span></p>
<p><span style="font-size: 16px;">&nbsp; &nbsp; COLLSCAN(全表扫描),SORT(使用sort但是无index),不合理的SKIP,SUBPLA(未用到index的$or),COUNTSCAN(不使用index进行count)</span></p>



致谢！             {#thanks}
===========================

<p>&nbsp; &nbsp; &nbsp; &nbsp;至此，我们完成了开发编译调试到最终上线生产运行。喜欢请关注公众号--程序猿牧场，谢谢！</p>
<p>&nbsp;<img src="https://img2020.cnblogs.com/blog/440176/202003/440176-20200316224639427-719739537.png" alt="" /></p>
