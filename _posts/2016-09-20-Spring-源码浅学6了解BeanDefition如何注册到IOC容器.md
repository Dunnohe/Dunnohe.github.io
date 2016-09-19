---
layout:     post
title:      "spring源码了解"
subtitle:   " \"6，了解BeanDefition如何注册到IOC容器\""
date:       2016-09-05 23:43:00
author:     "Dunno"
header-img: "img/post-bg-2015.jpg"
tags:
    - Spring
---

# 目录

- <a href="#js">背景介绍</a>
- <a href="#dmlj">BeanDefition的Resource定位</a>

# <a name="js">背景介绍</a>
<p>上一篇博客了解了applicationcontext是如何载入并解析beandefidition的，这里我们一起了解BeanDefition如何注册具体到IOC容器里面，建立好相应的数据结构</p>

# <a name="dmlj">了解BeanDefition的Resource载入和解析</a>

<p>我们继续从refreshBeanFactory开始看起，它的作用是通知子类去刷新自己的bean容器。</p>

## 1,refreshBeanFactory方法（刷新beanFactory）
这里可以把refresh理解成xxapplicationcontext的重启方法，refreshBeanFactory理解成bean容器的重启方法。

<pre>
<code>
//这里只贴出部分核心的代码
public abstract class AbstractRefreshableApplicationContext extends AbstractApplicationContext {

	@Override
	protected final void refreshBeanFactory() throws BeansException {
		//如果已经创建的bean容器
		if (hasBeanFactory()) {
			//清除已经创建的beans
			destroyBeans();
			//关闭beanfactory容器
			closeBeanFactory();
		}
		try {
			//创建beanfactory，可见默认使用的beanfactory容器是这个啊。
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			beanFactory.setSerializationId(getId());
			//这里面设置beanfactory是否支持“bean覆盖”和“循环引用”两种选项
			customizeBeanFactory(beanFactory);
			//加载bean resource
			loadBeanDefinitions(beanFactory);
			//设置beanfactory属性
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException(&quot;I/O error parsing bean definition source for &quot; + getDisplayName(), ex);
		}
	}
	
}
</code>
</pre>

































 