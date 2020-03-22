---
title: Zuul网关请求流重复读取
date: 2020-03-22 11:27:50
tags:
 - SpringCloud
 - Zuul
 - problems
urlname: zuul-requestio-repeatedly-read
keywords:
 - SpringCloud
 - 微服务
 - Zuul
description:
 - 最近一直负责公司基础组件API网关相关的工作，最近遇到一个很怪异的问题，导致网关转发post请求失败，此文记录一下解决问题的过程，同时也提供一个定位分析问题的思路。
---

大致的场景就是公司要对微服务进行分片，数据库分片之前听过（也只是停留在听过的层面），但微服务分片还真是头一次接触，后来了解到公司微服务分片的目的是为了减少服务发生故障影响的范围，对服务进行分片后，当请求过来根据请求获取分片号，然后将请求转发到分片的服务上，这样一来，如果某个分片的服务异常也只会影响到部分用户。公司目前的实现逻辑是一个服务多个分片在注册中心注册为不同的服务，例如`serviceA`，搞了3个分片，在注册中心会有`serviceA001`（001分片）、`serviceA002`（002分片）、`serviceA003`（003分片）、`serviceA`（默认分片）。大致如下图：

![servicesharding.png](https://i.loli.net/2020/03/11/VEaUnH7XJbOBrdK.png)

获取分片号这个是单独做成了SDK的方式，只需要在网关中引入相关类调用接口即可，网关根据返回的分片号通过修改请求路径从而实现路由转发。网关中修改路径需要在请求进入`DispatcherServlet`之前，为什么要在DispatcherServlet之前可以参考之前写的[结合DispatchServlet分析Zuul网关请求过程](http://www.huangxiutao.cn/2020/02/24/analysis-zuul-with-dispatcherservlet.html)。因此这里采取的是实现一个`Filter`，注意这里的`Filter`是`javax.servlet`包下的，并非`Zuul`中的`ZuulFilter`。

之前都是使用GET方法在测试验证功能，今天正式联调同事采用POST方法验证的时候出现了，而且报错信息有点迷惑分析分方向。报错的信息提示的是`I/O Exception readtimout` 导致`hystrix`超时之类的错误，因为公司的代码审查较严格，因此截图就没能放出来了。基于上篇文章的代码[Zuul](https://github.com/tryandcatch/zuul)本地大致试着复现一下。通过网关`POST`访问下游服务，但本地复现的时候报的并不是之前的 `IO readtimeout`，而是如下的这样错误：

网关中：

![Vpd60k](http://img.huangxiutao.cn/Vpd60k.png)

下游服务：

![7CPNa9](http://img.huangxiutao.cn/7CPNa9.png)

解决的思路：

最开始是看到网关中的这个`warn`：`Content-length different from byte array length! cl=23,array=21`然后在这方面找原因，但是发现这个方面没有什么进展，于是就`debug`下游的服务，看请求是否有到达相应的`controller`，发现请求并没有到`controller`中。于是只能到网关中找原因，一步一步`debug`，最终发现问题就出在调用获取分片号接口上，这个接口会获取`Request`中的IO流然后获取其中的请求参数，进一步根据请求参数获取此次请求的分片号，而`Request`中的IO流只能被读取一次，因此解决办法就是：将IO流“拷贝”一份，然后拿着这份拷贝的流往下游服务下发。参考方法：

```java
public class ZuulHttpServletRequestWrapper extends HttpServletRequestWrapper {
    private final byte[] body;
    public ZuulHttpServletRequestWrapper(HttpServletRequest request) throws IOException {
        super(request);
        body = StreamUtils.copyToByteArray(request.getInputStream());
    }
    @Override
    public BufferedReader getReader() throws IOException {
        return new BufferedReader(new InputStreamReader(getInputStream()));
    }
    @Override
    public ServletInputStream getInputStream() throws IOException {
        final ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(body);
        return new ServletInputStream() {
            @Override
            public boolean isFinished() {
                return false;
            }
            @Override
            public boolean isReady() {
                return false;
            }
            @Override
            public void setReadListener(ReadListener readListener) {
            }
            @Override
            public int read() throws IOException {
                return byteArrayInputStream.read();
            }
        };
    }
}
```
