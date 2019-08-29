---
title: '[译]Jib介绍:使用Jib来快速构建你的Java Docker 镜像'
tags: 
 - Docker
 - Java
 - 翻译
urlname: using-Jib-to-build-Java-Docker-images
date: 2019-03-24 11:13:59
---

> 在一个群里看到有群友讨论`Jib`这个东西，大致了解了`Jib`是`Google`出品的一个便于`Java`开发者构建`Dokcer`镜像的工具。由于现在开发的项目也用到了`Docker`，开发使用下来的感受就是：每构建一个项目都得写一个`Dockerfile`（按理说这个应该的是`DevOps`的工作→_→），write a project，provide a Dockerfile.于是今天就看了下官方的关于`Jib`介绍的博客，打算试着翻译下（主要是文章不长，哈哈哈哈）,翻译的不好，还请移步阅读原文，哈哈

原文博客链接：[Introducing Jib — build Java Docker images better](https://cloudplatform.googleblog.com/2018/07/introducing-jib-build-java-docker-images-better.html)

---

容器使得`Java`开发者更加接近“write once,run anyway”（一次编写，到处运行）的工作流程，但是容器化一个`Java`应用程序并不是一件简单的事，通常的流程是，需要你编写为应用程序编写一个`Dockerfile`，以`root`运行`Docker`的守护进程，等待构建的完成，最后将构建好的镜像`push`到远程注册中心。并不是所有的`Java`开发者都是容器专家，仅仅是构建一个`JAR`(这句真不知如何翻译，“what happened to just building a JAR?”)

为了应对这种挑战（需要`Java`开发人构建容器），我们很高心向大家介绍`Jib`—谷歌一个开源的`Java`容器化工具，能够让`Java`开发者使用他们所熟悉的工具来构建容器。`Jib`是一个简单快读的容器构建工具，不需要你编写`Dockerfile`和安装`Docker`就能够处理将`Java`应用程序打包成一个容器镜像的所有步骤，通过集成`Maven` 或 `Gradle`，添加`Maven`或`Gradle`插件，就能快速的将你的`Java`应用程序打包成容器.

`Docker`构建常规流程：

![docker-build-flow](<https://ws4.sinaimg.cn/large/006tKfTcgy1g1drh13ahej30gi031dg1.jpg>)

使用`Jib`后构建容器的流程：

![jib-build-flow](<https://ws4.sinaimg.cn/large/006tKfTcgy1g1dri02ix3j30ey016web.jpg>)

## Jib是如何使得开发更便利的

`Jib`通过借鉴`Docker` `layering`（分层）的优点，并与构建系统集成，通过以下方式优化`Java`容器镜像构建过程:

- 简单—`Jib`是基于`Java`实现的，并作为`Maven` 或`Gradle`构建中的一部分运行，你不需要维护`Dockerfile`，启动`Docker`的守护进程，不用担心会创建一个添加了所有依赖包的大`Jar`包.由于`Jib` 与`Java`构建过程的紧密结合，能够访问你所需要打包的程序，因此在容器构建后将自动检测到你的程序构建的变化。
- 快速—`Jib`利用镜像`layering`和注册缓存来实现快速，增量的构建，`Jib`会读取你的构建配置，将你的程序组织成不同的`layer`(项目依赖，资源文件和`class`文件)，，并且只会重新构建和`push`已更改的`layer`.在项目快速迭代的过程当中，每一次构建`Jib`只会`push`已更改的`layer`到注册中心而不是整个项目从而节省很多时间。
- 可复用—`Jib`支持以声明的方式从`Maven`和`Gradle`的愿数据构建容器镜像，因此只要你输入保持不变，就可以通过更改配置来实现复用性

## 如何使用Jib来容器化你的应用

`Jib`可以作为`Maven`或`Gradle`的一个插件来使用，只需很少的配置。将`Jib`插件添加到你的构建定义当中并配置最终的镜像，如果你是注册中的中心是私服，确保[在私服中为Jib配置了凭证](https://cloudtoolsforjava.page.link/bCK1) .最简单方法是使用`credential helper` 之类的工具,例如[docker-credential-gcr](https://cloudtoolsforjava.page.link/sZZL) . `Jib`同样提供构建镜像到`Docker`中.

在`Maven`中使用`Jib`:

```xml
<plugin>
  <groupId>com.google.cloud.tools</groupId>
  <artifactId>jib-maven-plugin</artifactId>
  <version>0.9.0</version>
  <configuration>
    <to>
      <image>gcr.io/my-project/image-built-with-jib</image>
    </to>
  </configuration>
</plugin>
```

命令：

```sh
# Builds to a container image registry.
$ mvn compile jib:build
# Builds to a Docker daemon.
$ mvn compile jib:dockerBuild
```

在`Gradle`中使用`Jib`：

```json
plugins {
  id 'com.google.cloud.tools.jib' version '0.9.0'
}
jib.to.image = 'gcr.io/my-project/image-built-with-jib'
```

命令：

```sh
# Builds to a container image registry.
$ gradle jib
# Builds to a Docker daemon.
$ gradle jibDockerBuild
```



我们希望大家使用`Jib`更便捷的开发`Java`，`Jib`适用于大多数`cloud providers`，尝试使用`Jib`并一起改进它，GitHub地址：[github.com/GoogleContainerTools/jib](http://github.com/GoogleContainerTools/jib)

