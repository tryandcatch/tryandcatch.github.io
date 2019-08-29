---
title: Spring数据源配置相关
tags: 
 - Spring
 - 数据库
 - JDBC
urlname: spring-datasource
date: 2019-03-16 15:01:24
---

> 最近在看`Spring`(主要是`Spring Boot`)配置数据源相关的东西，这篇文章主要记录下所了解到的知识，怕以后有些东西又忘记:sweat_smile:，方便以后查阅，多数据源配置有待补充......

## 主角

`Spring boot` 中涉及到数据库这一块的几个重要的类/接口：

- 数据源相关：
  - DataSource(DataSourceAutoConfiguration)
- 事务相关：
  - DataSourceTransactionManager(DataSourceTransactionManagerAutoConfiguration)
- 操作相关：
  - JdbcTemplate(JdbcTemplateAutoConfiguration)

## 常用的配置

```properties
#简单的配置：
spring.datasource.url=jdbc:mysql://ip/db
#项目中用到的：
#spring.datasource.url=jdbc:mysql://ip:port/db?characterEncoding=utf8&useSSL=true&serverTimezone=Asia/Shanghai
spring.datasource.username=dbuser
spring.datasource.password=dbpassword
#这个是可选的，spring 会根据url是使用哪种数据库从而使用默认的数据库驱动。之前一直以为这个是必须配置的
#另外com.mysql.jdbc.Driver 是 mysql-connector-java 5中的，com.mysql.cj.jdbc.Driver 是 mysql-connector-java 6中的
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

```

简单的数据库配置上面的其实已经够了，但是我们在实际应用中还会有这样的场景：

需要初始化数据库（建表和添加初始数据）

这里我们不采用`ORM`(`hibernate`)方式，而是通过在`resource`文件夹下添加`schema.sql`和`data.sql`这两个文件（参见：[Initialize a Database](<https://docs.spring.io/spring-boot/docs/current/reference/html/howto-database-initialization.html#howto-initialize-a-database-using-spring-jdbc>)），使用非内嵌/内存数据库需要添加配置：`spring.datasource.initialization-mode=always` ，才会自动初始化

:warning: 注意：配置了这个将会在每次项目启动的时候初始化数据一遍

## 自动初始化数据库实现的原理

简单描述：在`DataSourceAutoConfiguration` 中使用[Import](<https://segmentfault.com/a/1190000011068471>)注解，将`DataSourceInitializationConfiguration`“导入”，`DataSourceInitializationConfiguration`又将`DataSourceInitializerInvoker`“导入”,而`DataSourceInitializerInvoker`中会调用`DataSourceInitializer`的`createSchema`和`createSchema`方法，分别去建表和初始化数据。

![image-20190316175431149](https://ws3.sinaimg.cn/large/006tKfTcgy1g14sb7rd6gj31pm0u0npd.jpg)

---

