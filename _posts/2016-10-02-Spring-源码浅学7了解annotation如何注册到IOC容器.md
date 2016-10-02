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

	private BeanNameGenerator beanNameGenerator;

	private ScopeMetadataResolver scopeMetadataResolver;

	private final Set<Class<?>> annotatedClasses = new LinkedHashSet<Class<?>>();

	private final Set<String> basePackages = new LinkedHashSet<String>();
	
	@Override
	protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) {
		AnnotatedBeanDefinitionReader reader = getAnnotatedBeanDefinitionReader(beanFactory);
		ClassPathBeanDefinitionScanner scanner = getClassPathBeanDefinitionScanner(beanFactory);

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

		if (!this.annotatedClasses.isEmpty()) {
			if (logger.isInfoEnabled()) {
				logger.info("Registering annotated classes: [" +
						StringUtils.collectionToCommaDelimitedString(this.annotatedClasses) + "]");
			}
			reader.register(this.annotatedClasses.toArray(new Class<?>[this.annotatedClasses.size()]));
		}

		if (!this.basePackages.isEmpty()) {
			if (logger.isInfoEnabled()) {
				logger.info("Scanning base packages: [" +
						StringUtils.collectionToCommaDelimitedString(this.basePackages) + "]");
			}
			scanner.scan(this.basePackages.toArray(new String[this.basePackages.size()]));
		}

		String[] configLocations = getConfigLocations();
		if (configLocations != null) {
			for (String configLocation : configLocations) {
				try {
					Class<?> clazz = getClassLoader().loadClass(configLocation);
					if (logger.isInfoEnabled()) {
						logger.info("Successfully resolved class for [" + configLocation + "]");
					}
					reader.register(clazz);
				}
				catch (ClassNotFoundException ex) {
					if (logger.isDebugEnabled()) {
						logger.debug("Could not load class for config location [" + configLocation +
								"] - trying package scan. " + ex);
					}
					int count = scanner.scan(configLocation);
					if (logger.isInfoEnabled()) {
						if (count == 0) {
							logger.info("No annotated classes found for specified class/package [" + configLocation + "]");
						}
						else {
							logger.info("Found " + count + " annotated classes in package [" + configLocation + "]");
						}
					}
				}
			}
		}
	}
}
</code>
</pre>




