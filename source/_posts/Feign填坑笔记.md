---
title: Feign填坑
tags: 
 - Spring Cloud
 - problems
keywords:
 - Spring Cloud
 - Feign
 - 微服务调用
description: 用来记录使用Feign遇到的问题以及解决办法
urlname: feign-using-problems
date: 2019-03-26 15:32:04
---

> 用来记录使用Feign遇到的问题以及解决办法，不定期更新

Feign填坑：

1.服务调用方404；

加上服务提供方的context-path

2.服务调用方：获取到是LinkedHashMap，强转会出问题

解决办法：使用泛型

[2019.07.29]更新：

3.`java.lang.IllegalStateException: RequestParam.value() was empty on parameter 0` 问题

接手同事的项目，启动项目遇到此错误：![Feign-error.jpg](https://i.loli.net/2019/07/29/5d3e6b0b99a6112507.jpg)

解决办法：在每个调用的地方指定参数的value(name)的值。。。

SOF Link：[Feign Client with Spring Boot: RequestParam.value() was empty on parameter 0](https://stackoverflow.com/questions/44313482/feign-client-with-spring-boot-requestparam-value-was-empty-on-parameter-0)



---

参考文章：

[SpringCloud实战之Feign笔记](<https://www.jianshu.com/p/5ce2182aa718>)

[SpringCloud 复杂对象接收时候对象变成LinkeHashMap](<https://blog.csdn.net/qq_20746277/article/details/80923207>)