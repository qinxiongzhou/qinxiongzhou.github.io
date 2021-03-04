---
layout: post
title:  "轻装上阵Flink--在IDEA上开发基于Flink的实时数据流程序"
date:   2020-03-16 18:13:38 +0800
categories: 流计算
---


* content
{:toc}

前言				{#qianyan}
=============================
<p>&nbsp; &nbsp; &nbsp; 本文介绍如何在IDEA上快速开发基于Flink框架的DataStream程序。先直接上手！</p>

环境清单				{#qianyan}
=============================
<p>&nbsp; &nbsp; &nbsp; 案例是在win7运行。安装VirtualBox，在VirtualBox上安装Centos操作系统。所有资源都在百度云上，有需要请直接下载。安装教程基本都是傻瓜式，文章不做讲述，有需要直接网上搜索。</p>
<table>
<tbody>
<tr>
<td valign="top" width="268">资源</td>
<td valign="top" width="268">版本</td>
</tr>
<tr>
<td valign="top" width="268">VirtualBox</td>
<td valign="top" width="268">5.2.16</td>
</tr>
<tr>
<td valign="top" width="268">Centos</td>
<td valign="top" width="268">6.5</td>
</tr>
<tr>
<td valign="top" width="268">Maven</td>
<td valign="top" width="268">3.6.3</td>
</tr>
<tr>
<td valign="top" width="268">JDK</td>
<td valign="top" width="268">8u241</td>
</tr>
<tr>
<td valign="top" width="268">IDEA</td>
<td valign="top" width="268">2019.3.2</td>
</tr>
<tr>
<td rowspan="1" colspan="1" valign="top">Flink</td>
<td rowspan="1" colspan="1" valign="top">1.10.0</td>
</tr>
</tbody>
</table>
<p>链接：https://pan.baidu.com/s/12rXlY_z_Fck8-NRXdZ5row</p>
<p>提取码：qt2p</p>
<p><img src="https://img2020.cnblogs.com/i-beta/440176/202003/440176-20200316223057831-2081089221.png" alt="" /></p>

轻装上阵				{#qianzhuangshangzhen}
==============================

1、IP设置               {#IPset}
------------------------------
<p>&nbsp; &nbsp; &nbsp; Centos的设置静态IP为192.168.2.20，关闭防火墙</p>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> vi /etc/sysconfig/network-scripts/ifcfg-<span style="color: #000000;">eth0
</span><span style="color: #008080;">2</span> DEVICE=<span style="color: #000000;">eth0
</span><span style="color: #008080;">3</span> TYPE=<span style="color: #000000;">Ethernet
</span><span style="color: #008080;">4</span> ONBOOT=<span style="color: #000000;">yes #开机启动eth0网卡
</span><span style="color: #008080;">5</span> NM_CONTROLLED=<span style="color: #000000;">yes
</span><span style="color: #008080;">6</span> BOOTPROTO=<span style="color: #0000ff;">static</span>
<span style="color: #008080;">7</span> IPADDR=192.168.2.20
<span style="color: #008080;">8</span> GATEWAY=192.168.2.1
<span style="color: #008080;">9</span> NETMASK=255.255.255.0</pre>
</div>
<pre class="code-snippet__js" data-lang="nginx"><span style="font-family: 'PingFang SC', 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px;">     如果此时ping www.baidu.com等不通，需要我们添加dns服务器。<br /></span></pre>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> [root@localhost network-scripts]# vi /etc/<span style="color: #000000;">resolv.conf
</span><span style="color: #008080;">2</span> nameserver 192.168.2.1</pre>
</div>
<pre class="code-snippet__js" data-lang="nginx"><span style="font-family: 'PingFang SC', 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px;">　&nbsp;</span><span style="font-family: 'PingFang SC', 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px;">重新启动网络服务</span></pre>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> [root@localhost network-<span style="color: #000000;">scripts]# service network restart                   
</span><span style="color: #008080;">2</span> <span style="color: #000000;">正在关闭接口 eth0：[确定]
</span><span style="color: #008080;">3</span> <span style="color: #000000;">关闭环回接口：[确定]
</span><span style="color: #008080;">4</span> <span style="color: #000000;">弹出环回接口：[确定]
</span><span style="color: #008080;">5</span> 弹出界面 eth0：Determining <span style="color: #0000ff;">if</span> ip address 192.168.2.20 is already in use <span style="color: #0000ff;">for</span><span style="color: #000000;"> device eth0...
</span><span style="color: #008080;">6</span>                                                            [确定]</pre>
</div>
<pre class="code-snippet__js" data-lang="css"><span style="font-family: 'PingFang SC', 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px;">&nbsp; &nbsp; &nbsp; 关闭防火墙</span></pre>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> [root@localhost network-scripts]# service iptables stop</pre>
</div>
2、创建项目              {#createProject}
------------------------------

<p>&nbsp; &nbsp;在win7的命令行下，用mvn命令创建开发模板</p>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> mvn archetype:generate -DarchetypeGroupId=org.apache.flink -DarchetypeArtifactId=flink-quickstart-java -DarchetypeVersion=1.10.0</pre>
</div>
<pre class="code-snippet__js" data-lang="nginx"><span style="font-family: 'PingFang SC', 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px;">这种方式允许你为新项目命名。它将以交互式的方式询问你项目的 groupId、artifactId 和 package 名称。<br />用tree命令看下，如下结构。项目是一个 Maven project，它包含了两个类：StreamingJob 和 BatchJob <br />分别是 DataStream and DataSet 程序的基础骨架程序。main 方法是程序的入口，既可用于IDE测试/执行，也可用于部署。<br /></span></pre>
<div class="cnblogs_code">
<pre><span style="color: #008080;"> 1</span> <span style="color: #000000;">│  pom.xml
</span><span style="color: #008080;"> 2</span> <span style="color: #000000;">└─src
</span><span style="color: #008080;"> 3</span> <span style="color: #000000;">    └─main
</span><span style="color: #008080;"> 4</span> <span style="color: #000000;">        ├─java
</span><span style="color: #008080;"> 5</span> <span style="color: #000000;">        │  └─com
</span><span style="color: #008080;"> 6</span> <span style="color: #000000;">        │      └─ryan
</span><span style="color: #008080;"> 7</span> <span style="color: #000000;">        │              BatchJob.java
</span><span style="color: #008080;"> 8</span> <span style="color: #000000;">        │              StreamingJob.java
</span><span style="color: #008080;"> 9</span> <span style="color: #000000;">        └─resources
</span><span style="color: #008080;">10</span>                 log4j.properties</pre>
</div>

3、写一个自己的DataStream的程序              {#myDataStream}
------------------------------

<p>&nbsp;功能介绍：WindowWordCount.java，5s为一个时间窗口，摄取数据源的数据，计算单词出现的次数。</p>
<p>&nbsp;实时数据流计算简易架构图：</p>
<p><img src="https://img2020.cnblogs.com/i-beta/440176/202003/440176-20200316223645010-920292942.png" alt="" /></p>
<p>为了演示方便，这里我们只演示消息队列和Flink Job两个模块，利用nc工具来替代消息队列作为Flink Job摄取的数据源。</p>
<p>代码：</p>
<div class="cnblogs_code">
<pre><span style="color: #008080;"> 1</span> <span style="color: #0000ff;">package</span><span style="color: #000000;"> com.ryan;
</span><span style="color: #008080;"> 2</span> <span style="color: #0000ff;">import</span><span style="color: #000000;"> org.apache.flink.api.common.functions.FlatMapFunction;
</span><span style="color: #008080;"> 3</span> <span style="color: #0000ff;">import</span><span style="color: #000000;"> org.apache.flink.api.java.tuple.Tuple2;
</span><span style="color: #008080;"> 4</span> <span style="color: #0000ff;">import</span><span style="color: #000000;"> org.apache.flink.streaming.api.datastream.DataStream;
</span><span style="color: #008080;"> 5</span> <span style="color: #0000ff;">import</span><span style="color: #000000;"> org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
</span><span style="color: #008080;"> 6</span> <span style="color: #0000ff;">import</span><span style="color: #000000;"> org.apache.flink.streaming.api.windowing.time.Time;
</span><span style="color: #008080;"> 7</span> <span style="color: #0000ff;">import</span><span style="color: #000000;"> org.apache.flink.util.Collector;
</span><span style="color: #008080;"> 8</span> <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span><span style="color: #000000;"> WindowWordCount {
</span><span style="color: #008080;"> 9</span>     <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">static</span> <span style="color: #0000ff;">void</span> main(String[] args) <span style="color: #0000ff;">throws</span><span style="color: #000000;"> Exception {
</span><span style="color: #008080;">10</span>         StreamExecutionEnvironment env =<span style="color: #000000;"> StreamExecutionEnvironment.getExecutionEnvironment();
</span><span style="color: #008080;">11</span>         DataStream&lt;Tuple2&lt;String, Integer&gt;&gt; dataStream =<span style="color: #000000;"> env
</span><span style="color: #008080;">12</span>                 .socketTextStream("192.168.2.20", 9999<span style="color: #000000;">)
</span><span style="color: #008080;">13</span>                 .flatMap(<span style="color: #0000ff;">new</span><span style="color: #000000;"> Splitter())
</span><span style="color: #008080;">14</span>                 .keyBy(0<span style="color: #000000;">)
</span><span style="color: #008080;">15</span>                 .timeWindow(Time.seconds(5<span style="color: #000000;">))
</span><span style="color: #008080;">16</span>                 .sum(1<span style="color: #000000;">);
</span><span style="color: #008080;">17</span> <span style="color: #000000;">        dataStream.print();
</span><span style="color: #008080;">18</span>         env.execute("Window WordCount"<span style="color: #000000;">);
</span><span style="color: #008080;">19</span> <span style="color: #000000;">    }
</span><span style="color: #008080;">20</span>     <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">static</span> <span style="color: #0000ff;">class</span> Splitter <span style="color: #0000ff;">implements</span> FlatMapFunction&lt;String, Tuple2&lt;String, Integer&gt;&gt;<span style="color: #000000;"> {
</span><span style="color: #008080;">21</span> <span style="color: #000000;">        @Override
</span><span style="color: #008080;">22</span>         <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">void</span> flatMap(String sentence, Collector&lt;Tuple2&lt;String, Integer&gt;&gt; out) <span style="color: #0000ff;">throws</span><span style="color: #000000;"> Exception {
</span><span style="color: #008080;">23</span>             <span style="color: #0000ff;">for</span> (String word: sentence.split(" "<span style="color: #000000;">)) {
</span><span style="color: #008080;">24</span>                 out.collect(<span style="color: #0000ff;">new</span> Tuple2&lt;String, Integer&gt;(word, 1<span style="color: #000000;">));
</span><span style="color: #008080;">25</span> <span style="color: #000000;">            }
</span><span style="color: #008080;">26</span> <span style="color: #000000;">        }
</span><span style="color: #008080;">27</span> <span style="color: #000000;">    }
</span><span style="color: #008080;">28</span> }</pre>
</div>
<p>在centos机器上，命令行启动nc</p>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> nc -lk 9999</pre>
</div>
<p>IDEA上直接run main方法，然后在centos机器上，不断输入单词。</p>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> [ryan@localhost ~]$ nc -lk 9999
<span style="color: #008080;">2</span> <span style="color: #000000;">java
</span><span style="color: #008080;">3</span> <span style="color: #000000;">java
</span><span style="color: #008080;">4</span> <span style="color: #000000;">shen
</span><span style="color: #008080;">5</span> 深圳 深圳</pre>
</div>
<pre class="code-snippet__js" data-lang="http"><span style="font-family: 'PingFang SC', 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px;">IDEA控制台上输出如下：</span></pre>
<p><img src="https://img2020.cnblogs.com/i-beta/440176/202003/440176-20200316223808468-1469708168.png" alt="" /></p>
<p><span style="color: #ff0000;">注意</span>：第一次在IDEA上运行这个程序，可能会报如下异常</p>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> java.lang.NoClassDefFoundError: org/apache/flink/streaming/api/datastream/DataStream</pre>
</div>
<p>原因是IDEA没有导入flink 的lib下的jar包。导入即可。</p>
<p><img src="https://img2020.cnblogs.com/i-beta/440176/202003/440176-20200316223856965-1675046420.png" alt="" /></p>

4、打包发布到centos平台上的Flink集群             {#deployFlink}
------------------------------

<p>&nbsp; &nbsp; &nbsp; 修改pom.xml文件的mainclass的值为com.ryan.WindowWordCount</p>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">mainClass</span><span style="color: #0000ff;">&gt;</span>com.ryan.WindowWordCount<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">mainClass</span><span style="color: #0000ff;">&gt;</span></pre>
</div>
<p>&nbsp; &nbsp; &nbsp; 执行mvn clean install，得到flink-demo-1.0-SNAPSHOT.jar，并上传到centos机器上。</p>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> mvn clean install</pre>
</div>
<p>&nbsp; &nbsp; &nbsp; 打开两个centos的控制台，一个用于打开nc，一个用于运行我们打包好的Flink jar包。</p>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> [ryan@localhost ~]$ nc -lk 9999
<span style="color: #008080;">2</span> <span style="color: #000000;">java
</span><span style="color: #008080;">3</span> <span style="color: #000000;">shen
</span><span style="color: #008080;">4</span> 深圳 深圳 深圳</pre>
</div>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> [root@localhost flink-1.10.0]# bin/flink run flink-demo/flink-demo-1.0-<span style="color: #000000;">SNAPSHOT.jar 
</span><span style="color: #008080;">2</span> Job has been submitted with JobID 9931a9dfc2eddeb2d0b5ed15578bd488</pre>
</div>
<pre class="code-snippet__js" data-lang="ruby"><span style="font-family: 'PingFang SC', 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px;">&nbsp; 回到win7上，用浏览器打开http://192.168.2.20:8081/，在Running Jobs上，可以看到一条记录。</span></pre>
<p><img src="https://img2020.cnblogs.com/i-beta/440176/202003/440176-20200316224051536-1109600237.png" alt="" /></p>
<p>&nbsp;</p>
<p>&nbsp;&nbsp; &nbsp; &nbsp; 在Task Managers上，Stdout模块看到程序输出的结果。</p>
<p><img src="https://img2020.cnblogs.com/i-beta/440176/202003/440176-20200316224106552-1826418068.png" alt="" /></p>
<p>&nbsp;&nbsp; &nbsp; &nbsp; 所有代码都上传到github上，有需要的朋友可以下载</p>
<div class="cnblogs_code">
<pre><span style="color: #008080;">1</span> https:<span style="color: #008000;">//</span><span style="color: #008000;">github.com/qinxiongzhou/flink-demo</span></pre>
</div>

致谢！             {#thanks}
===========================

<p>&nbsp; &nbsp; &nbsp; &nbsp;至此，我们完成了开发编译调试到最终上线生产运行。喜欢请关注公众号--程序猿牧场，谢谢！</p>
<p>&nbsp;<img src="https://img2020.cnblogs.com/blog/440176/202003/440176-20200316224639427-719739537.png" alt="" /></p>