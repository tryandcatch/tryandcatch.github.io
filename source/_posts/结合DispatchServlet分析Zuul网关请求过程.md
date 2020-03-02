---
title: 结合DispatchServlet分析Zuul网关请求过程
urlname: analysis-zuul-with-dispatcherservlet
date: 2020-02-24 17:16:43
tags:
 - SpringCloud
 - Zuul
 - Java
keywords:
 - SpringCloud
 - Zuul
 - SpringMVC
description:
 - 结合SpringMVC中的核心DispatchServlet来分析Zuul 1.x网关处理请求的过程。
---
> 基于Zuul1.3.1RELEASE进行分析，代码：https://github.com/tryandcatch/zuul

当一个请求到达`Zuul`的时候首先会经过一系列的 `Filter`（这里的`Filter`是`Servlet`下的）然后进入到`DispatchServlet`的`doDispatch`方法:
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    //...
    // Determine handler for the current request.
	mappedHandler = getHandler(processedRequest);
    //...
}
//....
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    for (HandlerMapping hm : this.handlerMappings) {
        if (logger.isTraceEnabled()) {
            logger.trace(
                    "Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
        }
        HandlerExecutionChain handler = hm.getHandler(request);
        if (handler != null) {
            return handler;
        }
    }
    return null;
}
//....
```
`getHandler()`方法主要是根据请求遍历`handlerMappings`，调用`HandlerMapping`的`getHandler()`方法(`HandlerMapping`是个接口，在这里实际是调用的`AbstractHandlerMapping`的`getHandler()`方法),找出能够处理该请求的`handler`（HandlerExecutionChain）。

`AbstractHandlerMapping`的`getHandler()`方法：
```java
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    //getHandlerInternal 是个抽象的方法，该处会调用其子类AbstractUrlHandlerMapping、AbstractHandlerMethodMapping中的方法
    Object handler = getHandlerInternal(request);
    if (handler == null) {
        handler = getDefaultHandler();
    }
    if (handler == null) {
        return null;
    }
    // Bean name or resolved handler?
    if (handler instanceof String) {
        String handlerName = (String) handler;
        handler = getApplicationContext().getBean(handlerName);
    }

    HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);
    if (CorsUtils.isCorsRequest(request)) {
        CorsConfiguration globalConfig = this.globalCorsConfigSource.getCorsConfiguration(request);
        CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
        CorsConfiguration config = (globalConfig != null ? globalConfig.combine(handlerConfig) : handlerConfig);
        executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
    }
    return executionChain;
}
```
`HandlerMapping`、`AbstractHandlerMapping`、`SimpleUrlHandlerMapping`、`ZuulHandlerMapping`之间的关系：
![ZuulHandlerMapping.png](https://i.loli.net/2020/02/25/CAh3U1yBtPng5KV.png)
![SimpleUrlHandlerMapping.png](https://i.loli.net/2020/02/25/snjYBT5HOgo8IuG.png)

`AbstractUrlHandlerMapping`的`getHandlerInternal`方法：
```java
protected Object getHandlerInternal(HttpServletRequest request) throws Exception {
    String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
    Object handler = lookupHandler(lookupPath, request);
    if (handler == null) {
        // We need to care for the default handler directly, since we need to
        // expose the PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE for it as well.
        Object rawHandler = null;
        if ("/".equals(lookupPath)) {
            rawHandler = getRootHandler();
        }
        if (rawHandler == null) {
            rawHandler = getDefaultHandler();
        }
        if (rawHandler != null) {
            // Bean name or resolved handler?
            if (rawHandler instanceof String) {
                String handlerName = (String) rawHandler;
                rawHandler = getApplicationContext().getBean(handlerName);
            }
            validateHandler(rawHandler, request);
            handler = buildPathExposingHandler(rawHandler, lookupPath, lookupPath, null);
        }
    }
    if (handler != null && logger.isDebugEnabled()) {
        logger.debug("Mapping [" + lookupPath + "] to " + handler);
    }
    else if (handler == null && logger.isTraceEnabled()) {
        logger.trace("No handler mapping found for [" + lookupPath + "]");
    }
    return handler;
}
```

`Zuul`中请求流程大致如下图：

![ZuulRequestFlow.png](https://i.loli.net/2020/03/02/O8tPFwkV4vjGdCY.png)