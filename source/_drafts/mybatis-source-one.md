---
title: Mybatis源码学习：配置文件解析
category:
  - MyBatis
tags:
  - MyBatis
abbrlink: e781838b
date: 2019-06-20 15:59:53
layout: false
---

<!--![](https://ws1.sinaimg.cn/large/64202e18gy1g47obrjquaj20w40fqmzp.jpg)-->

# 介绍
MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。
# 使用
每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先定制的 Configuration 的实例构建出 SqlSessionFactory 的实例。
直接看从XML中构建SqlSessionFactory的代码实现。
```java
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```
首先，通过MyBatis的Resources工具类加载配置文件得到一个输入流，其次，使用SqlSessionFactoryBuilder创建一个SqlSessionFactory对象。下面就是看SqlSessionFactoryBuilder是怎么创建SqlSessionFactory对象的。
```java
// SqlSessionFactoryBuilder.java #64
public SqlSessionFactory build(InputStream inputStream) {
    // 调用重载方法
    return build(inputStream, null, null);
}
// 这里才是SqlSessionFactory的具体创建过程
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
        // 对XML解析，底层是调用Java的Xpath工具类解析节点的。
        // 首先，创建XMLConfigBuilder
        XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
        // 创建SqlSessionFactory
        return build(parser.parse());
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
        ErrorContext.instance().reset();
        try {
        inputStream.close();
        } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
        }
    }
}
```
XMLConfigBuilder会通过构造函数设置一些参数，并且会创建一个Document对象。创建好XMLConfigBuilder之后，接着调用了 `XMLConfigBuilder#parse()`方法.
```java
public Configuration parse() {
    if (parsed) {
      // 是否解析过该配置文件，不能重复解析
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true; // 标记位，判断是否已经解析过了。
    // parser.evalNode("/configuration") 是得到一个XNode， configuration标签是MyBatis配置文件的配置入口。
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;

}
private void parseConfiguration(XNode root) {
    try {
      //issue #117 read properties first
      // 解析properties配置
      propertiesElement(root.evalNode("properties"));
      // 解析settings配置并转为Properties对象
      Properties settings = settingsAsProperties(root.evalNode("settings"));
      loadCustomVfs(settings);
      // 解析typeAliases标签
      typeAliasesElement(root.evalNode("typeAliases"));
      // 解析plugins配置
      pluginElement(root.evalNode("plugins"));
      // 解析objectFactory配置
      objectFactoryElement(root.evalNode("objectFactory"));
      // 解析objectWrapperFactory配置
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      // 解析reflectorFactory配置
      reflectorFactoryElement(root.evalNode("reflectorFactory"));
      settingsElement(settings);
      // read it after objectFactory and objectWrapperFactory issue #631
      environmentsElement(root.evalNode("environments"));
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      // 解析 mappers 标签
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
```
上面需要解析配置的标签比较多，下面只挑选 mappers 标签的解析过程说说。
```java
private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
      // 循环遍历 parent 的子节点， parent的子节点就是我们配置的 具体mapper文件类似下面这样
        /*
        * <mappers>
        *     <mapper resource="mapper/UserMapper.xml"/>
        *     <mapper resource="mapper/ClassesMapper.xml"/>
        * </mappers>
        */
      for (XNode child : parent.getChildren()) {
        if ("package".equals(child.getName())) {
            // 获取 <package> 节点中的 name 属性
          String mapperPackage = child.getStringAttribute("name");
          // 从指定包中查找 mapper 接口，并根据 mapper 接口解析映射配置
          configuration.addMappers(mapperPackage);
        } else {
          // 我们是配置文件，所以会走这里。获取 resource、url、mapperClass属性
          String resource = child.getStringAttribute("resource");
          String url = child.getStringAttribute("url");
          String mapperClass = child.getStringAttribute("class");
          if (resource != null && url == null && mapperClass == null) {
            ErrorContext.instance().resource(resource);
            InputStream inputStream = Resources.getResourceAsStream(resource);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url != null && mapperClass == null) {
            ErrorContext.instance().resource(url);
            InputStream inputStream = Resources.getUrlAsStream(url);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url == null && mapperClass != null) {
            Class<?> mapperInterface = Resources.classForName(mapperClass);
            configuration.addMapper(mapperInterface);
          } else {
            throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
          }
        }
      }
    }
}
```


