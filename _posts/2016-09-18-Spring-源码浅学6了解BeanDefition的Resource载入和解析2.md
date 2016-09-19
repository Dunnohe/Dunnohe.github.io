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

## 3,bean的加载和解析

### 我们先来看看通过String path来加载bean的实现

<p>因为最终他和resource加载bean的方式一样，都是通过resource加载。只不过String加载bean多了一步，把String解析成resource。</p>

<pre>
<code>
public abstract class AbstractBeanDefinitionReader implements EnvironmentCapable, BeanDefinitionReader {
	public int loadBeanDefinitions(String location, Set&lt;Resource&gt; actualResources) throws BeanDefinitionStoreException {
		//获得resourceloader，获得不到抛异常	
		ResourceLoader resourceLoader = getResourceLoader();
		if (resourceLoader == null) {
			throw new BeanDefinitionStoreException(
					&quot;Cannot import bean definitions from location [&quot; + location + &quot;]: no ResourceLoader available&quot;);
		}

		//如果resourceloader是ResourcePatternResolver（支持把通配符配置解析成多个resource，比如&quot;/WEB-INF/*-context.xml&quot;），则获得对应解析出的多个resource
		if (resourceLoader instanceof ResourcePatternResolver) {
			// Resource pattern matching available.
			try {
				Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
				//这里面是for循环去加载和解析，实际就是调用下面的“单个解析resource的方法”。
				//同时在通过resource 来解析bean的实现中也是调用“单个解析resource的方法”。
				//所以，我们最终看这个调用单个resource方法的实现就行了。
				int loadCount = loadBeanDefinitions(resources);
				if (actualResources != null) {
					for (Resource resource : resources) {
						actualResources.add(resource);
					}
				}
				if (logger.isDebugEnabled()) {
					logger.debug(&quot;Loaded &quot; + loadCount + &quot; bean definitions from location pattern [&quot; + location + &quot;]&quot;);
				}
				return loadCount;
			}
			catch (IOException ex) {
				throw new BeanDefinitionStoreException(
						&quot;Could not resolve bean definition resource pattern [&quot; + location + &quot;]&quot;, ex);
			}
		}
		//反之获得对应解析的单个resource
		else {
			Resource resource = resourceLoader.getResource(location);
			//调用单个解析resource的方法，这个方法就是最终加载解析bean的底层公用方法
			int loadCount = loadBeanDefinitions(resource);
			if (actualResources != null) {
				actualResources.add(resource);
			}
			if (logger.isDebugEnabled()) {
				logger.debug(&quot;Loaded &quot; + loadCount + &quot; bean definitions from location [&quot; + location + &quot;]&quot;);
			}
			return loadCount;
		}
	}
}
</code>
</pre>


## 4,最终底层调用的单个解析resource的方法

<pre>
<code>
public class XmlBeanDefinitionReader extends AbstractBeanDefinitionReader {
	@Override
	public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
		//EncodedResource 多了一些对编码属性的设置
		return loadBeanDefinitions(new EncodedResource(resource));
	}
	
	//从指定的resource文件中加载bean
	public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
		Assert.notNull(encodedResource, &quot;EncodedResource must not be null&quot;);
		if (logger.isInfoEnabled()) {
			logger.info(&quot;Loading XML bean definitions from &quot; + encodedResource.getResource());
		}

		//resourcesCurrentlyBeingLoaded用来存放“正在加载中”的配置文件，通过threadlocal实现的。
		//放资源的时候，没有就创建并且放进去，有就直接放
		Set&lt;EncodedResource&gt; currentResources = this.resourcesCurrentlyBeingLoaded.get();
		if (currentResources == null) {
			currentResources = new HashSet&lt;EncodedResource&gt;(4);
			this.resourcesCurrentlyBeingLoaded.set(currentResources);
		}
		if (!currentResources.add(encodedResource)) {
			throw new BeanDefinitionStoreException(
					&quot;Detected cyclic loading of &quot; + encodedResource + &quot; - check your import definitions!&quot;);
		}
		try {
			InputStream inputStream = encodedResource.getResource().getInputStream();
			try {
				InputSource inputSource = new InputSource(inputStream);
				if (encodedResource.getEncoding() != null) {
					inputSource.setEncoding(encodedResource.getEncoding());
				}
				//具体去单个加载一个配置文件的实现（这个层级好深啊。。。。）
				return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
			}
			finally {
				inputStream.close();
			}
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					&quot;IOException parsing XML document from &quot; + encodedResource.getResource(), ex);
		}
		finally {
			//加载流程结束，移除“正在被加载”的配置文件
			currentResources.remove(encodedResource);
			//移除threadlocal里面的hashset
			if (currentResources.isEmpty()) {
				this.resourcesCurrentlyBeingLoaded.remove();
			}
		}
	}
}
</code>
</pre>

## 5,具体去单个加载一个配置文件的实现

<pre>
<code>
public class XmlBeanDefinitionReader extends AbstractBeanDefinitionReader {

	protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
			throws BeanDefinitionStoreException {
		try {
			//解析配置文件，同时验证document的语法，得到document
			Document doc = doLoadDocument(inputSource, resource);
			//注册bean
			return registerBeanDefinitions(doc, resource);
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
		catch (SAXParseException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
		}
		catch (SAXException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"XML document from " + resource + " is invalid", ex);
		}
		catch (ParserConfigurationException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Parser configuration exception parsing XML from " + resource, ex);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"IOException parsing XML document from " + resource, ex);
		}
		catch (Throwable ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Unexpected exception parsing XML document from " + resource, ex);
		}
	}
	
	//注册bean
	public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
		int countBefore = getRegistry().getBeanDefinitionCount();
		//具体注册bean的方法（按照spring的尿性肯定还有好几层，而且名字相似）
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}
	
}
</code>
</pre>


## 6,BeanDefinitionDocumentReader（负责将解析好的document注册到容器中）

<pre>
<code>
//注册接口
public interface BeanDefinitionDocumentReader {
	void registerBeanDefinitions(Document doc, XmlReaderContext readerContext)
			throws BeanDefinitionStoreException;
}

public class DefaultBeanDefinitionDocumentReader implements BeanDefinitionDocumentReader {

	@Override
	public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
		this.readerContext = readerContext;
		logger.debug("Loading bean definitions");
		Element root = doc.getDocumentElement();
		doRegisterBeanDefinitions(root);
	}
	
	protected void doRegisterBeanDefinitions(Element root) {

		BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createDelegate(getReaderContext(), root, parent);

		if (this.delegate.isDefaultNamespace(root)) {
			String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
			if (StringUtils.hasText(profileSpec)) {
				String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
						profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
				if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
					return;
				}
			}
		}

		//空方法，没有实现，也体现了人家先做好设计，再填实现的编码过程
		preProcessXml(root);
		//这里就是最底层载入和解析的入口了，接下来它就会交给BeanDefinitionDocumentReader去做具体的解析过程了。
		//下一篇学习笔记来总结具体的解析过程。
		parseBeanDefinitions(root, this.delegate);
		//空方法没有实现
		postProcessXml(root);

		this.delegate = parent;
	}
}
</code>
</pre>

































 