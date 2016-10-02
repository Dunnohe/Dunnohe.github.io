---
layout:     post
title:      "spring源码了解"
subtitle:   " \"7，了解annotation如何注册到IOC容器\""
date:       2016-09-05 23:43:00
author:     "Dunno"
header-img: "img/post-bg-2015.jpg"
tags:
    - Spring
---

# 目录

- <a href="#js">背景介绍</a>
- <a href="#dmlj">加载的入口</a>

# <a name="js">背景介绍</a>
<p>之前已经简单的总结了spring读取xml配置到注册容器的整个过程。但是实际工作中我们会大量的使用到annotation，annotation注册到容器的过程是如何的呢。</p>

# <a name="dmlj">加载的入口</a>

<p>之前我们就了解到，加载的入口在AbstractRefreshableApplicationContext的loadBeanDefinitions方法，我们看一下这个方法的实现，发现有annotation的实现版本。</p>

![IOC容器结构](http://dunnohe.github.io/img/spring/7/access.png)

## 1,doRegisterBeanDefinitions方法（具体解析bean的入口）
这里可以把refresh理解成xxapplicationcontext的重启方法，refreshBeanFactory理解成bean容器的重启方法。

<pre>
<code>
	
</code>
</pre>




