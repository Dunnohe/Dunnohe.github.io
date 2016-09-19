---
layout:     post
title:      "spring源码了解"
subtitle:   " \"6，了解BeanDefition的Resource载入和解析2\""
date:       2016-09-18 23:02:00
author:     "Dunno"
header-img: "img/post-bg-2015.jpg"
tags:
    - Spring
---

# 目录

- <a href="#js">背景介绍</a>
- <a href="#dmlj">具体解析bean的过程</a>

# <a name="js">背景介绍</a>
<p>上一篇学习笔记跟了大半天，才跟到加载bean解析的入口，这篇学习笔记则学习一下具体的解析部分的代码。</p>

# <a name="dmlj">具体解析bean的过程</a>

<p>我们继续从refreshBeanFactory开始看起，它的作用是通知子类去刷新自己的bean容器。</p>

## 1,doRegisterBeanDefinitions方法（具体解析bean的入口）
这里可以把refresh理解成xxapplicationcontext的重启方法，refreshBeanFactory理解成bean容器的重启方法。

<pre>
<code>
//这里只贴出部分核心的代码
public class DefaultBeanDefinitionDocumentReader implements BeanDefinitionDocumentReader {
	protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		//如果使用默认的命名空间
		if (delegate.isDefaultNamespace(root)) {
			NodeList nl = root.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (node instanceof Element) {
					Element ele = (Element) node;
					//如果元素使用的是默认命名空间
					if (delegate.isDefaultNamespace(ele)) {
						//解析默认元素
						parseDefaultElement(ele, delegate);
					}
					else {
						//解析定制元素
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
			//解析定制元素
			delegate.parseCustomElement(root);
		}
	}
	
	//解析默认元素
	private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
	
		//1，如果是import node
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
			//解析import 资源
			importBeanDefinitionResource(ele);
		}
		
		//2，如果是alias node
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
			//处理alias 资源
			processAliasRegistration(ele);
		}
		
		//3，如果是bean node
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			//处理bean node
			processBeanDefinition(ele, delegate);
		}
		
		//4，如果是 beans node
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
			// 递归执行doRegisterBeanDefinitions，又回到了解析的入口函数了
			doRegisterBeanDefinitions(ele);
		}
	}
	
	//我们来重点看看3和4的实现，先看3的
	protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
		
		//把element解析成BeanDefinitionHolder
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
		if (bdHolder != null) {
			bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
			try {
				// 注册bean到容器.
				BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error("Failed to register bean definition with name '" +
						bdHolder.getBeanName() + "'", ele, ex);
			}
			// 发送注册事件，这个很重要，我们可以通过实现对应的监听器来获得这个bean注册的信息了。
			// 既然这里有事件，那我们很容易联想到1，2是不是也有对应的事件？点开验证一下
			// 果然有对应的事件，分别是fireImportProcessed，fireAliasRegistered
			getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
		}
	}
}
	
</code>
</pre>

## 2,registerBeanDefinition（具体把bean注册到容器的入口）

<pre>
<code>
public class BeanDefinitionReaderUtils {
	public static void registerBeanDefinition(
			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
			throws BeanDefinitionStoreException {

		// Register bean definition under primary name.
		String beanName = definitionHolder.getBeanName();
		registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

		// Register aliases for bean name, if any.
		String[] aliases = definitionHolder.getAliases();
		if (aliases != null) {
			for (String alias : aliases) {
				registry.registerAlias(beanName, alias);
			}
		}
	}
}
	
</code>
</pre>

