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
- <a href="#dmlj">具体解析bean的过程</a>

# <a name="js">背景介绍</a>
<p>上一篇学习笔记跟了大半天，才跟到加载bean解析的入口，这篇学习笔记则学习一下具体的解析部分以及注册到容器的代码。</p>

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

		// beanName会从id属性拿，否则就自动构建defaultName，调用buildDefaultName（）
		String beanName = definitionHolder.getBeanName();
		// 注册
		registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

		//注册设置了别名的bean
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

## DefaultListableBeanFactory （注册的具体过程）
<pre>
<code>
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
		implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {
	@Override
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		Assert.hasText(beanName, "Bean name must not be empty");
		Assert.notNull(beanDefinition, "BeanDefinition must not be null");

		if (beanDefinition instanceof AbstractBeanDefinition) {
			try {
				((AbstractBeanDefinition) beanDefinition).validate();
			}
			catch (BeanDefinitionValidationException ex) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Validation of bean definition failed", ex);
			}
		}

		BeanDefinition oldBeanDefinition;
		//beanDefinitionMap 这个就是“容器”的本质了，它的结构是这样的，beanName-beanDefinition
		oldBeanDefinition = this.beanDefinitionMap.get(beanName);
		
		//如果容器里面已经有这个beanDefinition，会根据相应的判断，给出相应的日志提示，建议启动关注这块日志，实际规范的项目不应该让他打出这个日志，打出来了建议找到对应的地方改掉。可能会造成故障。
		if (oldBeanDefinition != null) {
			//是否允许覆盖，不允许就抛异常
			if (!isAllowBeanDefinitionOverriding()) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Cannot register bean definition [" + beanDefinition + "] for bean '" + beanName +
						"': There is already [" + oldBeanDefinition + "] bound.");
			}
			//
			else if (oldBeanDefinition.getRole() < beanDefinition.getRole()) {
				// e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
				if (this.logger.isWarnEnabled()) {
					this.logger.warn("Overriding user-defined bean definition for bean '" + beanName +
							"' with a framework-generated bean definition: replacing [" +
							oldBeanDefinition + "] with [" + beanDefinition + "]");
				}
			}
			else if (!beanDefinition.equals(oldBeanDefinition)) {
				if (this.logger.isInfoEnabled()) {
					this.logger.info("Overriding bean definition for bean '" + beanName +
							"' with a different definition: replacing [" + oldBeanDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			else {
				if (this.logger.isDebugEnabled()) {
					this.logger.debug("Overriding bean definition for bean '" + beanName +
							"' with an equivalent definition: replacing [" + oldBeanDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			//新的beanDefinition放到容器里面
			this.beanDefinitionMap.put(beanName, beanDefinition);
		}
		else {
			//如果容器是第一次注册这个definition的话
			if (hasBeanCreationStarted()) {
				// Cannot modify startup-time collection elements anymore (for stable iteration)
				synchronized (this.beanDefinitionMap) {
					this.beanDefinitionMap.put(beanName, beanDefinition);
					List<String> updatedDefinitions = new ArrayList<String>(this.beanDefinitionNames.size() + 1);
					updatedDefinitions.addAll(this.beanDefinitionNames);
					updatedDefinitions.add(beanName);
					this.beanDefinitionNames = updatedDefinitions;
					if (this.manualSingletonNames.contains(beanName)) {
						Set<String> updatedSingletons = new LinkedHashSet<String>(this.manualSingletonNames);
						updatedSingletons.remove(beanName);
						this.manualSingletonNames = updatedSingletons;
					}
				}
			}
			else {
				// Still in startup registration phase
				this.beanDefinitionMap.put(beanName, beanDefinition);
				this.beanDefinitionNames.add(beanName);
				this.manualSingletonNames.remove(beanName);
			}
			this.frozenBeanDefinitionNames = null;
		}

		if (oldBeanDefinition != null || containsSingleton(beanName)) {
			resetBeanDefinition(beanName);
		}
	}
}
	
</code>
</pre>

