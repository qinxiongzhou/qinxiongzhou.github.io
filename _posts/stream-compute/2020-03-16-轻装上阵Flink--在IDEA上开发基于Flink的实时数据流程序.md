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

<center><img src="/source/stream-compute/01.jpg" alt="还清清单"></center>

轻装上阵				{#qianzhuangshangzhen}
==============================

1、IP设置               {#IPset}
------------------------------
<p>&nbsp; &nbsp; &nbsp; Centos的设置静态IP为192.168.2.20，关闭防火墙</p>

```shell
vi /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes #开机启动eth0网卡
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.2.20
GATEWAY=192.168.2.1
NETMASK=255.255.255.0
```

如果此时ping www.baidu.com等不通，需要我们添加dns服务器。

```shell
[root@localhost network-scripts]# vi /etc/resolv.conf
nameserver 192.168.2.1
```

重新启动网络服务

```shell
[root@localhost network-scripts]# service network restart                   
正在关闭接口 eth0：[确定]
关闭环回接口：[确定]
弹出环回接口：[确定]
弹出界面 eth0：Determining if ip address 192.168.2.20 is already in use for device eth0...
                                                           [确定]
```

关闭防火墙

```shell
[root@localhost network-scripts]# service iptables stop
```

2、创建项目              {#createProject}
------------------------------

在win7的命令行下，用mvn命令创建开发模板

```shell
mvn archetype:generate -DarchetypeGroupId=org.apache.flink -DarchetypeArtifactId=flink-quickstart-java -DarchetypeVersion=1.10.0
```

这种方式允许你为新项目命名。它将以交互式的方式询问你项目的 groupId、artifactId 和 package 名称。<br />用tree命令看下，如下结构。项目是一个 Maven project，它包含了两个类：StreamingJob 和 BatchJob <br />分别是 DataStream and DataSet 程序的基础骨架程序。main 方法是程序的入口，既可用于IDE测试/执行，也可用于部署。<br />

```shell
│  pom.xml
└─src
    └─main
        ├─java
        │  └─com
        │      └─ryan
        │              BatchJob.java
        │              StreamingJob.java
        └─resources
                log4j.properties
```

3、写一个自己的DataStream的程序              {#myDataStream}
------------------------------

<p>&nbsp;功能介绍：WindowWordCount.java，5s为一个时间窗口，摄取数据源的数据，计算单词出现的次数。</p>
<p>&nbsp;实时数据流计算简易架构图：</p>
<center><img src="/source/stream-compute/02.png" alt="实时数据流计算简易架构图"></center>
<p>为了演示方便，这里我们只演示消息队列和Flink Job两个模块，利用nc工具来替代消息队列作为Flink Job摄取的数据源。</p>
<p>代码：</p>

```java
package com.ryan;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;
public class WindowWordCount {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Tuple2<String, Integer>> dataStream = env
                .socketTextStream("192.168.2.20", 9999)
                .flatMap(new Splitter())
                .keyBy(0)
                .timeWindow(Time.seconds(5))
                .sum(1);
        dataStream.print();
        env.execute("Window WordCount");
    }
    public static class Splitter implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String sentence, Collector<Tuple2<String, Integer>> out) throws Exception {
            for (String word: sentence.split(" ")) {
                out.collect(new Tuple2<String, Integer>(word, 1));
            }
        }
    }
}
```

在centos机器上，命令行启动nc

```shell
nc -lk 9999
```

IDEA上直接run main方法，然后在centos机器上，不断输入单词。

```shell
[ryan@localhost ~]$ nc -lk 9999
java
java
shen
深圳 深圳
```

IDEA控制台上输出如下：
<center><img src="/source/stream-compute/03.jpg" alt=""></center>

<p><span style="color: #ff0000;">注意</span>：第一次在IDEA上运行这个程序，可能会报如下异常</p>

```java
java.lang.NoClassDefFoundError: org/apache/flink/streaming/api/datastream/DataStream
```

原因是IDEA没有导入flink 的lib下的jar包。导入即可。

<center><img src="/source/stream-compute/04.jpg" alt=""></center>

4、打包发布到centos平台上的Flink集群             {#deployFlink}
------------------------------

修改pom.xml文件的mainclass的值为com.ryan.WindowWordCount

```xml
<mainClass>com.ryan.WindowWordCount</mainClass>
```

执行mvn clean install，得到flink-demo-1.0-SNAPSHOT.jar，并上传到centos机器上。

```shell
mvn clean install
```

打开两个centos的控制台，一个用于打开nc，一个用于运行我们打包好的Flink jar包。

```shell
[ryan@localhost ~]$ nc -lk 9999
java
shen
深圳 深圳 深圳
```

```shell
[root@localhost flink-1.10.0]# bin/flink run flink-demo/flink-demo-1.0-SNAPSHOT.jar 
Job has been submitted with JobID 9931a9dfc2eddeb2d0b5ed15578bd488
```


回到win7上，用浏览器打开http://192.168.2.20:8081/，在Running Jobs上，可以看到一条记录。
<center><img src="/source/stream-compute/05.jpg" alt=""></center> <br />

在Task Managers上，Stdout模块看到程序输出的结果。
<center><img src="/source/stream-compute/06.jpg" alt=""></center><br />

所有代码都上传到github上，有需要的朋友可以下载

```shell
https://github.com/qinxiongzhou/flink-demo
```

致谢！             {#thanks}
===========================

<p>&nbsp; &nbsp; &nbsp; &nbsp;至此，我们完成了开发编译调试到最终上线生产运行。喜欢请关注公众号--程序猿牧场，谢谢！</p>
<p>&nbsp;<img src="https://img2020.cnblogs.com/blog/440176/202003/440176-20200316224639427-719739537.png" alt="" /></p> 