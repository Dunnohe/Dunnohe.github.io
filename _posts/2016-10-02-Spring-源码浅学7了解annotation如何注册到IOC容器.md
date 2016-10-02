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

## 1,Annotation版本对于loadBeanDefinitions方法的实现

<pre>
<code>
public class AnnotationConfigWebApplicationContext extends AbstractRefreshableWebApplicationContext
		implements AnnotationConfigRegistry {

	//生成beanName的类，它会读id标签的值，没有的话就生成默认的
	private BeanNameGenerator beanNameGenerator;

	//解析Scope元数据的，封装了一些对scope属性的判断的方法
	private ScopeMetadataResolver scopeMetadataResolver;

	
	private final Set&lt;Class&lt;?&gt;&gt; annotatedClasses = new LinkedHashSet&lt;Class&lt;?&gt;&gt;();

	//这里存放所有的扫描的包路径
	private final Set&lt;String&gt; basePackages = new LinkedHashSet&lt;String&gt;();
	
	@Override
	protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) {
		//构造annotationReader，值得一提，这个annotationReader并没有继承BeanDefinitionReader的接口，它是自己单独实现了一套逻辑
		AnnotatedBeanDefinitionReader reader = getAnnotatedBeanDefinitionReader(beanFactory);
		//构造classpath扫描器
		ClassPathBeanDefinitionScanner scanner = getClassPathBeanDefinitionScanner(beanFactory);

		//构造beanName生成器，用来设置bean的Name的
		BeanNameGenerator beanNameGenerator = getBeanNameGenerator();
		if (beanNameGenerator != null) {
			reader.setBeanNameGenerator(beanNameGenerator);
			scanner.setBeanNameGenerator(beanNameGenerator);
			beanFactory.registerSingleton(AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR, beanNameGenerator);
		}

		ScopeMetadataResolver scopeMetadataResolver = getScopeMetadataResolver();
		if (scopeMetadataResolver != null) {
			reader.setScopeMetadataResolver(scopeMetadataResolver);
			scanner.setScopeMetadataResolver(scopeMetadataResolver);
		}

		//如果this.annotatedClasses.isEmpty不为空，就让reader把这个annotationclass注册到容器里面，当然如果通过web起来的话，这里是为空的。
		if (!this.annotatedClasses.isEmpty()) {
			if (logger.isInfoEnabled()) {
				logger.info(&quot;Registering annotated classes: [&quot; +
						StringUtils.collectionToCommaDelimitedString(this.annotatedClasses) + &quot;]&quot;);
			}
			reader.register(this.annotatedClasses.toArray(new Class&lt;?&gt;[this.annotatedClasses.size()]));
		}

		//如果this.basePackages.isEmpty不为空，也需要把这个信息设置到Scanner，当然这里肯定是为空的。当然如果通过web起来的话，这里是为空的。
		if (!this.basePackages.isEmpty()) {
			if (logger.isInfoEnabled()) {
				logger.info(&quot;Scanning base packages: [&quot; +
						StringUtils.collectionToCommaDelimitedString(this.basePackages) + &quot;]&quot;);
			}
			scanner.scan(this.basePackages.toArray(new String[this.basePackages.size()]));
		}

		//获得config，交给reader去注册到容器中，于是我们来看看reader注册到容器中的过程
		String[] configLocations = getConfigLocations();
		if (configLocations != null) {
			for (String configLocation : configLocations) {
				try {
					Class&lt;?&gt; clazz = getClassLoader().loadClass(configLocation);
					if (logger.isInfoEnabled()) {
						logger.info(&quot;Successfully resolved class for [&quot; + configLocation + &quot;]&quot;);
					}
					//使用reader注册到容器中
					reader.register(clazz);
				}
				catch (ClassNotFoundException ex) {
					if (logger.isDebugEnabled()) {
						logger.debug(&quot;Could not load class for config location [&quot; + configLocation +
								&quot;] - trying package scan. &quot; + ex);
					}
					int count = scanner.scan(configLocation);
					if (logger.isInfoEnabled()) {
						if (count == 0) {
							logger.info(&quot;No annotated classes found for specified class/package [&quot; + configLocation + &quot;]&quot;);
						}
						else {
							logger.info(&quot;Found &quot; + count + &quot; annotated classes in package [&quot; + configLocation + &quot;]&quot;);
						}
					}
				}
			}
		}
	}
}
</code>
</pre>

## 1,AnnotatedBeanDefinitionReader注册的具体过程

<pre>
<code>
public class AnnotatedBeanDefinitionReader {

	private final BeanDefinitionRegistry registry;

	private BeanNameGenerator beanNameGenerator = new AnnotationBeanNameGenerator();

	private ScopeMetadataResolver scopeMetadataResolver = new AnnotationScopeMetadataResolver();

	private ConditionEvaluator conditionEvaluator;
	
	//最终调用的是registerBean(Class&lt;?&gt; annotatedClass, String name, Class&lt;? extends Annotation&gt;... qualifiers)的方法
	public void register(Class&lt;?&gt;... annotatedClasses) {
		for (Class&lt;?&gt; annotatedClass : annotatedClasses) {
			registerBean(annotatedClass);
		}
	}

	public void registerBean(Class&lt;?&gt; annotatedClass) {
		registerBean(annotatedClass, null, (Class&lt;? extends Annotation&gt;[]) null);
	}

	@SuppressWarnings(&quot;unchecked&quot;)
	public void registerBean(Class&lt;?&gt; annotatedClass, Class&lt;? extends Annotation&gt;... qualifiers) {
		registerBean(annotatedClass, null, qualifiers);
	}

	@SuppressWarnings(&quot;unchecked&quot;)
	public void registerBean(Class&lt;?&gt; annotatedClass, String name, Class&lt;? extends Annotation&gt;... qualifiers) {
		AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(annotatedClass);
		if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
			return;
		}

		ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
		abd.setScope(scopeMetadata.getScopeName());
		String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));
		AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
		if (qualifiers != null) {
			for (Class&lt;? extends Annotation&gt; qualifier : qualifiers) {
				if (Primary.class == qualifier) {
					abd.setPrimary(true);
				}
				else if (Lazy.class == qualifier) {
					abd.setLazyInit(true);
				}
				else {
					abd.addQualifier(new AutowireCandidateQualifier(qualifier));
				}
			}
		}

		BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
		definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
		BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
	}
}
</code>
</pre>



