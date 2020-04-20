---
title: 解决Linux docker启动不了的问题
date: 2020-04-20 21:20:18
tags: 
 - Docker
 - problems
urlname: fixed-docker-start-failed-on-linux
keywords:
 - docker start failed
 - docker启动不了
description: 记录解决在Linux环境下启动Docker失败的处理办法
---

## 错误信息

使用的是阿里云的云主机，四台机器，里面都已经安装好了`Docker`，用于搭建K8S集群，其中三台的`docker`没能正常启动。执行`docker info` ,报错：

![image-20200419213927696](http://img.huangxiutao.cn/image-20200419213927696.png)

这表明`docker`没有启动成功。

## 排查过程
通过`journalctl -u docker.service` 查看`docker`的日志（这是一个需要注意的地方，）

![image-20200420213748031](http://img.huangxiutao.cn/image-20200420213748031.png)

通过上面的日志可以看出，由于``chmod /var/lib/docker: read-only file system`  ,`/var/lib/docker` 为只读权限，每次都启动失败，启动失败太多次，`docker`就停止启动了（`start request repeated too quickly for docker.service`）于是执行（`root`用户）：`mount -o remount rw /`。

再次启动：`systemctl start docker.service`，嗯 再一次报错了。。。。。

![image-20200420214432422](http://img.huangxiutao.cn/image-20200420214432422.png)

这次是由于`failed to start daemon: failed to dial "/run/containerd/containerd.sock": unknown service containerd.services.namespaces.v1.Namespaces: not implemented`，启动失败次数太多导致停止启动，这个地方试了[很多办法](https://github.com/docker/for-linux/issues/162)都没能搞定。。。终于在[Docker daemon and Containerd dockerd out of sync in 18.09 #421](https://github.com/docker/for-linux/issues/421#issuecomment-416742531) 这个`issue`中看到了这个方法，先将`containerd`停掉，再启动（发现直接`restart`不管用，）：

```shell
systemctl stop containerd
systemctl start containerd

#启动docker
systemctl start docker.service
```



## 总结

`Linux`中排查`docker`启动过程,可以通过如下思路：

```shell
#查看docker的状态
sudo docker info 
#启动正常输出client 和 server的信息

#通用查看docker状态的方式
systemctl status docker.service


#启动失败 查看docker的日志，
journalctl -u docker.service

#根据启动失败日志分析原因，如果是文件权限相关，则修改权限

```