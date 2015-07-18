---
layout: post
title: java一些常用的性能调优工具和命令
excerpt: "介绍一些java性能调休的工具和命令
modified: 2015-07-03
tags: [jvm, linux, java]
comments: true
image:
  feature: sample-image-5.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---
#java一些常用的性能调优工具和命令
<section id="table-of-contents" class="toc">
  <header>
    <h3>Overview</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

### 背景
<p>最近的工作中，和带我的到时一起做了一个项目，加载所有数据到内存，然后做分片访问，在发布几个版本之后，监控发现内存泄露了，我感觉这种问题十分难以定位，但是同事用各种监控的命令发现泄漏的地方，我感觉很有必要总结学习一下</p>

### 正文
JDK的安装包提供了很多辅助工具用来监测数据。
#### jps
>**作用：**这个命令类似于linux下的ps命令，但它只能显示java的进程
>选项：
>- [ ] -q   只输出输出进程id
>- [ ] -m  输出用于传递java主函数的参数
>- [ ] -l    输出完整的应用程序主类包名或者应用程序jar的完整路径
>- [ ] -v   输出传给JVM的参数
``` python
jps -mlv
```
#### jinfo
>**作用：**可以查看正在运行的java应用程序的拓展参数，甚至支持在运行时修改部分参数。
>- [ ] -flag < name > : 打印指定JVM的参数值  
>- [ ] -flag [+ | - ] < name > : 设置指定JVM的参数值
>- [ ] -flag < name > = < value > :  设置指定JVM的参数值
#### jmap
>**作用：**可以生成java应用程序的堆快照和对象的统计信息。
这个命令很好用，就是靠他我们发现某几个实体对象数量特别多，再检查处理那块实体逻辑才发现bug的。
``` python
//显示pid为2972的java程序的统计信息
jmap -histo 2972
```
#### jstat
>**作用：**用于观察java运行时信息的工具。非常强大
>用法：jstat [ generalOption | outputOptions vmid [interval[s|ms] [count]] ]
>选项：
> * generalOption 这项取值是一些帮助和选择就不说了，可以man jstat看一下
>  * - [ ] help --display help message
>  * - [ ] options --display help message

	