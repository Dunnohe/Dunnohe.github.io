---
layout:     post
title:      "Java-mybatis generator返回主键的用法"
subtitle:   " \"了解mybatis generator返回主键的用法\""
date:       2016-07-22 20:09:00
author:     "Dunno"
header-img: "img/post-bg-2015.jpg"
tags:
    - Java
---

# 目录

- <a href="#js">用法介绍</a>
- <a href="#ckzl">附：我的generatorConfig全部配置</a>
- <a href="#xkd">附：mybatis主键的用法</a>


# <a name="js">用法介绍</a>
使用mybatis plugin,配置如下：
<pre>
<code>
&lt;plugin&gt;
    &lt;groupId&gt;org.mybatis.generator&lt;/groupId&gt;
    &lt;artifactId&gt;mybatis-generator-maven-plugin&lt;/artifactId&gt;
    &lt;version&gt;1.3.2&lt;/version&gt;
    &lt;executions&gt;
        &lt;execution&gt;
            &lt;phase&gt;install&lt;/phase&gt;
            &lt;id&gt;Generate MyBatis Artifacts&lt;/id&gt;
            &lt;goals&gt;
                &lt;goal&gt;generate&lt;/goal&gt;
            &lt;/goals&gt;
        &lt;/execution&gt;
    &lt;/executions&gt;
    &lt;configuration&gt;
        &lt;configurationFile&gt;src/main/resources/generatorConfig.xml&lt;/configurationFile&gt;
        &lt;overwrite&gt;true&lt;/overwrite&gt;
    &lt;/configuration&gt;
&lt;/plugin&gt;
</code>
</pre>

查看官方对于生成主键的配置的解释

> http://www.mybatis.org/generator/configreference/generatedKey.html 

generatorConfig.xml 配置
<pre>
<code>
&lt;table schema=&quot;hotel_pda&quot; tableName=&quot;vg_upload_file&quot;&gt;
    &lt;property name=&quot;useActualColumnNames&quot; value=&quot;false&quot;/&gt;
    &lt;generatedKey column=&quot;id&quot; sqlStatement=&quot;MySql&quot; identity=&quot;true&quot; /&gt;
&lt;/table&gt;
</code>
</pre>

# <a name="ckzl">附：我的generatorConfig全部配置</a>

<pre>
<code class="XML">
&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; ?&gt;
&lt;!DOCTYPE generatorConfiguration PUBLIC &quot;-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN&quot;
        &quot;http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd&quot; &gt;

&lt;generatorConfiguration&gt;
    &lt;classPathEntry location=&quot;/Users/lianghe/.m2/repository/mysql/mysql-connector-java/5.1.36/mysql-connector-java-5.1.36.jar&quot;/&gt;

    &lt;context id=&quot;mysql&quot; targetRuntime=&quot;MyBatis3&quot;&gt;
        &lt;plugin type=&quot;org.mybatis.generator.plugins.EqualsHashCodePlugin&quot; /&gt;
        &lt;plugin type=&quot;org.mybatis.generator.plugins.SerializablePlugin&quot; /&gt;
        &lt;plugin type=&quot;org.mybatis.generator.plugins.CaseInsensitiveLikePlugin&quot; /&gt;
        &lt;plugin type=&quot;org.mybatis.generator.plugins.ToStringPlugin&quot; /&gt;
        &lt;plugin type=&quot;org.mybatis.generator.plugins.RowBoundsPlugin&quot;/&gt;
        &lt;plugin type=&quot;org.mybatis.generator.plugins.CaseInsensitiveLikePlugin&quot;/&gt;

        &lt;!-- &#27880;&#37322; --&gt;
        &lt;commentGenerator &gt;
            &lt;property name=&quot;suppressAllComments&quot; value=&quot;true&quot;/&gt;&lt;!-- &#26159;&#21542;&#21462;&#28040;&#27880;&#37322; --&gt;
            &lt;property name=&quot;suppressDate&quot; value=&quot;true&quot; /&gt; &lt;!-- &#26159;&#21542;&#29983;&#25104;&#27880;&#37322;&#20195;&#26102;&#38388;&#25139;--&gt;
        &lt;/commentGenerator&gt;

        &lt;!-- jdbc --&gt;
        &lt;jdbcConnection driverClass=&quot;com.mysql.jdbc.Driver&quot;
                        connectionURL=&quot;jdbc:mysql://xx.xx.xx.xx/hotel_pda&quot;
                        userId=&quot;xxx&quot;
                        password=&quot;xxx&quot; /&gt;
        &lt;!-- &#29983;&#25104;model --&gt;
        &lt;javaModelGenerator targetPackage=&quot;com.xxx.domain&quot;
                            targetProject=&quot;src/main/java&quot;&gt;
            &lt;property name=&quot;enableSubPackages&quot; value=&quot;false&quot;/&gt;
            &lt;property name=&quot;trimStrings&quot; value=&quot;false&quot;/&gt;
        &lt;/javaModelGenerator&gt;

        &lt;!-- &#29983;&#25104;sql --&gt;
        &lt;sqlMapGenerator targetPackage=&quot;mapper/auto&quot;
                         targetProject=&quot;src/main/resources&quot; &gt;
            &lt;property name=&quot;enableSubPackages&quot; value=&quot;false&quot;/&gt;
        &lt;/sqlMapGenerator&gt;

        &lt;!-- &#29983;&#25104;sql client--&gt;
        &lt;javaClientGenerator targetPackage=&quot;com.xxx.dao.auto&quot;
                             targetProject=&quot;src/main/java&quot;
                             type=&quot;XMLMAPPER&quot; &gt;
            &lt;property name=&quot;enableSubPackages&quot; value=&quot;false&quot;/&gt;
        &lt;/javaClientGenerator&gt;

        &lt;!-- Tables --&gt;
        &lt;table schema=&quot;hotel_pda&quot; tableName=&quot;vg_upload_file&quot;&gt;
            &lt;property name=&quot;useActualColumnNames&quot; value=&quot;false&quot;/&gt;
            &lt;generatedKey column=&quot;id&quot; sqlStatement=&quot;MySql&quot; identity=&quot;true&quot; /&gt;
        &lt;/table&gt;
    &lt;/context&gt;
&lt;/generatorConfiguration&gt;

</code>
</pre>

# <a name="xkd">附：mybatis返回主键的用法</a>
<pre>
<code class="Java">
public int insert(Domain domain) {
	//插入完成后 domain的主键id字段会被赋值，直接返回即可
	domainMapper.insertSelective(domain);
	return domain.getId();
}
</code>
</pre>











 