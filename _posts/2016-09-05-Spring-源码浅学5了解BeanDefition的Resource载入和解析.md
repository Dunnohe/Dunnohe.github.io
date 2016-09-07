---
layout:     post
title:      "spring源码了解"
subtitle:   " \"5，了解BeanDefition的Resource载入和解析\""
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
<p>这里我们一起了解一下IOC容器是如何载入和解析bean的</p>

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
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
	
}
</code>
</pre>

## 2,loadBeanDefinitions（加载以及解析配置文件）

<pre>
<code>
//这里只贴出部分核心的代码
public abstract class AbstractXmlApplicationContext extends AbstractRefreshableConfigApplicationContext {

	@Override
	protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
		// 由这里可以看到默认读取Xml的reader是这个
		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

		// Configure the bean definition reader with this context's
		// resource loading environment.
		beanDefinitionReader.setEnvironment(this.getEnvironment());
		beanDefinitionReader.setResourceLoader(this);
		beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

		initBeanDefinitionReader(beanDefinitionReader);
		//具体加载beanDefinition的方法
		loadBeanDefinitions(beanDefinitionReader);
	}
	
	protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
		
		//通过resource方法加载bean	
		Resource[] configResources = getConfigResources();
		if (configResources != null) {
			reader.loadBeanDefinitions(configResources);
		}
		
		//通过string的形式加载bean
		String[] configLocations = getConfigLocations();
		if (configLocations != null) {
			reader.loadBeanDefinitions(configLocations);
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
	public int loadBeanDefinitions(String location, Set<Resource> actualResources) throws BeanDefinitionStoreException {
		//获得resourceloader，获得不到抛异常	
		ResourceLoader resourceLoader = getResourceLoader();
		if (resourceLoader == null) {
			throw new BeanDefinitionStoreException(
					"Cannot import bean definitions from location [" + location + "]: no ResourceLoader available");
		}

		//如果resourceloader是ResourcePatternResolver（支持把通配符配置解析成多个resource，比如"/WEB-INF/*-context.xml"），则获得对应解析出的多个resource
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
					logger.debug("Loaded " + loadCount + " bean definitions from location pattern [" + location + "]");
				}
				return loadCount;
			}
			catch (IOException ex) {
				throw new BeanDefinitionStoreException(
						"Could not resolve bean definition resource pattern [" + location + "]", ex);
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
				logger.debug("Loaded " + loadCount + " bean definitions from location [" + location + "]");
			}
			return loadCount;
		}
	}
}
</code>
</pre>

## 3,最终底层调用的单个解析resource的方法

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
		Assert.notNull(encodedResource, "EncodedResource must not be null");
		if (logger.isInfoEnabled()) {
			logger.info("Loading XML bean definitions from " + encodedResource.getResource());
		}

		//resourcesCurrentlyBeingLoaded用来存放“正在加载中”的配置文件，通过threadlocal实现的。
		//放资源的时候，没有就创建并且放进去，有就直接放
		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
		if (currentResources == null) {
			currentResources = new HashSet<EncodedResource>(4);
			this.resourcesCurrentlyBeingLoaded.set(currentResources);
		}
		if (!currentResources.add(encodedResource)) {
			throw new BeanDefinitionStoreException(
					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
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
					"IOException parsing XML document from " + encodedResource.getResource(), ex);
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

## 4,具体去单个加载一个配置文件的实现

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


## 5,BeanDefinitionDocumentReader（负责将解析好的document注册到容器中）

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
				//
				String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
						profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
				if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
					return;
				}
			}
		}

		preProcessXml(root);
		//这里面就是具体的载入配置的逻辑代码了，感兴趣的可以去研究具体怎么载入解析的
		parseBeanDefinitions(root, this.delegate);
		postProcessXml(root);

		this.delegate = parent;
	}
}
</code>
</pre>

































 