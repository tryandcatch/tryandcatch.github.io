---
title: Docker网络学习
tags: Docker
urlname: Docker-network
date: 2019-05-06 17:56:06
---

> 之前对Docker只是停留在听过阶段，公司的项目中使用的是Docker部署，因此对Docker有了更深入的理解与运用，但是对于Docker网络这一块，还不是很熟，遇到了些问题（eg:默认设置中容器之间不能像在本机（宿主机）那样通过ip直接访问)，很有必要学习下这块的东西，将常用的命令以及使用记录下来，便于以后查看。

## 前言

Docker在Linux和Mac上的差异：图片来源：[Docker for Mac 的网络问题](https://windard.com/blog/2018/05/01/Docker-For-Mac)

![image.png](https://i.loli.net/2019/08/25/Z9zHVYLNjM1m5vw.png)

![mac_docker_host](https://i.loli.net/2019/08/25/tFnDJNsR8pPi5wQ.png)

这个差异会导致在Mac上安装Docker后使用`ifconfig` 命令会看不到docker0这个网卡

## Docker 网络“驱动”

docker 主要分为以下几种驱动（Driver）：

- bridge：Docker网络默认使用的驱动，
- host：使用宿主机网络，没有做容器与容器/容器与宿主机间的网络隔离，也就会发生网络竞争与冲突。Docker 17.06 以后的版本支持
- overlay：不是很了解，官方的文档：可以将多个Docker守护进程（Docker daemons）连接在一起，可以使集群服务之间相互通信，可以使用`overlay`实现集群服务与独立容器间的通信，以及两个不同的Docker进程间的独立容器的通信，此策略消除了在这些容器之间系统级别路由的需要
- none：对于此容器，禁用所有联网。通常与自定义网络驱动程序一起使用。none 模式不适用于集群服务。

## Docker 网络运用

- 创建网络

  `docker network create backend_test`

  `docker network create frontend_test`

- 查看已有的网络

  `docker network ls `

- 运行docker镜像配置网络

  `docker run -itd --name docker_app_name --net backend_test hello-world`

  ...

---

参考文档：

[“深入浅出”来解读Docker网络核心原理](<https://blog.51cto.com/ganbing/2087598>)

[使用 Docker 容器网络](<https://www.ibm.com/developerworks/cn/linux/l-docker-network/index.html>)

[Networking overview](https://docs.docker.com/network/)

[docker 网络](<http://wudaijun.com/2017/11/docker-network/>)