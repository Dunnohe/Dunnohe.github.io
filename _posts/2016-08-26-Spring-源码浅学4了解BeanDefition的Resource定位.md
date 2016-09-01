---
layout:     post
title:      "spring源码了解"
subtitle:   " \"4，了解BeanDefition的Resource定位\""
date:       2016-08-26 19:51:00
author:     "Dunno"
header-img: "img/post-bg-2015.jpg"
tags:
    - Spring
---

# 目录

- <a href="#js">背景介绍</a>
- <a href="#ckzl">参考资料</a>
- <a href="#dmlj">BeanDefition的Resource定位</a>

# <a name="js">背景介绍</a>
<p>之前我们了解了IOC初略的一个结构，这里我们开始学习一下spring是如何定位资源的。（桶子到哪里装水）</p>

# <a name="ckzl">参考资料</a>

> http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/ 

> spring 技术内幕

# <a name="dmlj">BeanDefition的Resource定位</a>

我们以FileSystemXmlApplicationContext为例，看一下它的类图（图片如果不清楚，可以右键保存下来看），我们主要观察蓝色线条的部分：
<br/>

![IOC容器结构](http://dunnohe.github.io/img/spring/3/applicationcontext.png)

<p>这个关系是这样的：FileSystemXmlApplicationContext --》 AbstractXmlApplicationContext --》 AbstractRefreshableConfigApplicationContext --》 AbstractApplicationContext --》 AbstractApplicationContext --》 DefaultResourceLoader</p>

<p>下面我们从FileSystemXmlApplicationContext代码开始看起：</p>

## FileSystemXmlApplicationContext

<pre>
<code>
//这里只贴出部分核心的代码
public class FileSystemXmlApplicationContext extends AbstractXmlApplicationContext {
	
	//其他构造方法最终的都是调用这个方法
	public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent) throws BeansException {
		
		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}

	@Override
	protected Resource getResourceByPath(String path) {
		if (path != null && path.startsWith("/")) {
			path = path.substring(1);
		}
		return new FileSystemResource(path);
	}
}
</code>
</pre>

### BeanFactory

<pre>
<code>
//封装了一些基本的访问bean的方法
public interface BeanFactory {
	//&amp;xxxbean对象拿到的是xxxbean对应的factorybean
	String FACTORY_BEAN_PREFIX = &quot;&amp;&quot;;

	Object getBean(String name) throws BeansException;

	&lt;T&gt; T getBean(String name, Class&lt;T&gt; requiredType) throws BeansException;

	&lt;T&gt; T getBean(Class&lt;T&gt; requiredType) throws BeansException;

	Object getBean(String name, Object... args) throws BeansException;

	&lt;T&gt; T getBean(Class&lt;T&gt; requiredType, Object... args) throws BeansException;

	boolean containsBean(String name);

	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, Class&lt;?&gt; typeToMatch) throws NoSuchBeanDefinitionException;

	Class&lt;?&gt; getType(String name) throws NoSuchBeanDefinitionException;

	String[] getAliases(String name);

}
</code>
</pre>

### HierarchicalBeanFactory

<pre>
<code>
//在BeanFactory基础上加了获得父BeanFactory的方法
public interface HierarchicalBeanFactory extends BeanFactory {

	BeanFactory getParentBeanFactory();

	boolean containsLocalBean(String name);

}
</code>
</pre>

### ConfigurableBeanFactory
在HierarchicalBeanFactory的基础上增加了一些对BeanFactory的配置功能，比如setParentBeanFactory()设置双亲IOC容器。

### ListableBeanFactory
在BeanFactory基础上增加了对容器bean的各种属性查询方法。

## <a name="res">ResourceLoader</a>

ResourceLoader,顾名思义：资源加载者。前面讲到了把beanFactory理解成装实体的桶子，那么我们到怎么装呢？到哪里去装呢？这就是ResourceLoader负责的了。
<pre>
<code>
public interface ResourceLoader {
	//这个值是classpath:  方便classpathresource时使用
	String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;
	//获得资源，也就是告诉你装东西的地址
	Resource getResource(String location);
	//
	ClassLoader getClassLoader();

}
</code>
</pre>

## <a name="app">xxxApplicationContext</a>

它是一个更高级的容器，容器只是它的一个子功能，同时它还支持其他特性：

- 支持国际化 多语言版本的支持(MessageSource)
- 访问资源 容器加载bean是要有一个加载源的，这个resource就是加载源。
- 支持应用事件 在整个bean的声明周期中，需要引入应用事件，便于更好的管理。

applicationContext怎么理解呢，你就把它理解成一个工具箱，里面放了桶子（xxBeanFactory），还放了给桶子装东西的方法(resourcesLoader),还有中英文说明书（MessageSource）,还有遥控器可以遥控桶子（ApplicationEventPublisher）。有了这些大工具箱，你能想到的启动容器的条件是不是就都具备了！






























 