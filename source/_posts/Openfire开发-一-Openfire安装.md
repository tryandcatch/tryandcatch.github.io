---
title: Openfire开发(一)-Openfire搭建
tags: 
 - IM
 - Openfire
keywords:
 - Openfire
 - IM
 - 即时通讯
description: 在Mac上搭建Openfire
urlname: build-openfire-in-Mac
date: 2019-05-09 20:33:05
---

> 公司要把原有的聊天从使用FireBase那一套转为使用Openfire相关的，我负责后端的一些模块。由于是第一次接触IM相关的业务，在前期的技术调研的过程中花了比较多的时间，前期真是一脸懵逼，看了很多文章才明白了一些知识，而且很多文章都比较老旧了，都是使用Eclipse，在项目中引入jar包这种方式做的。因此打算写个基于IDEA和maven这一套来开发Openfire插件系列文章（~~希望能克服懒惰把这个系列写完~~ :laughing: ,公司已经不做这个了。。。。🤦‍♂️）

## Openfire 简介

官网链接：[Openfire](https://www.igniterealtime.org/projects/openfire/)

`Openfire`是免费的、开源的、基于可拓展通讯和表示协议([XMPP]([https://xmpp.org](https://xmpp.org/)))、采用`Java`开发的实时协作服务器，可用于`IM`服务器。支持Windows/Linux/Mac OS

## 搭建 Openfire

### 下载安装

[下载链接](https://www.igniterealtime.org/downloads/index.jsp#openfire) 选择适合自己操作系统的版本下载，目前（2019年5月9日）最新的版本是4.3.2。

下载好了后点击dmg 一步一步安装即可（如果出现安装不了记得在 设置》安全性与隐私 看一下，选择允许），需要注意的是1⃣电脑要配置好`Java`运行环境，2⃣`$JAVA_HOME` 要有值，否则`Openfire`启动不了，我一开始就遇到这个问题，`java -version`是没问题的，但是`echo $JAVA_HOME` 却没有值，解决办法：[Mac配置JAVA_HOME](https://www.jianshu.com/p/27e494e45f78) 。默认安装的路径在`/usr/local/openfire`

### 配置数据库

这里使用MySQL，创建数据库：

`create database openfire default character set utf8 default collate utf8_general_ci;`

导入Openfire安装目录下的数据库文件：`/usr/local/openfire/resources/database/openfire_mysql.sql`

### 启动并配置Openfire

方式一：在设置面板中找到 Openfire 并打开，`Start Openfire`,

方式二：进入`Openfire`的`bin`目录，执行启动脚本：

```shell
cd /usr/local/openfire/bin
./openfire.sh
```

推荐是第二种方式，因为在启动的时候会有详细的信息

启动成功后打开localhost:9090，就能配置Openfire了:

![image.png](https://i.loli.net/2019/08/25/KqaEZbsDHOd5Jln.png)

如果出现上图中的错误信息:`没有找到安装目录。定义openfire安装目录或新建或增加openfire_init.xml文件到类路径`

以及启动的时候出现无法创建log的相关错误信息：`Could not create directory /usr/local/openfire/logs`

解决办法：
为Openfire的安装目录（`/usr/local/openfire/bin`）添加读和写的权限，点击文件夹》显示简介，添加当前用户赋予读与写的权限（当然也可以用Linux权限的相关命令，这个我不熟。。🤦‍♂️）

Tips: 查看当前用户：`whoami`

![image.png](https://i.loli.net/2019/08/25/ezywq9rFacj8l6J.png)

解决完上面的问题，就能正常启动并访问配置页面了

![image.png](https://i.loli.net/2019/08/25/LXP4CxMyfdjmokn.png)

![image.png](https://i.loli.net/2019/08/25/mbsvVtpneYJr7Dq.png)

然后后续按照下一步继续配置就可以了。需要注意的是数据库配置，数据库连接URL配置成这样，配置utf-8否则数据库存储中文消息的时候会成???，而且&符号必须被转义为`&amp;`，[参考文章](<https://djkin.iteye.com/blog/1900523>)

`jdbc:mysql://localhost:3306/openfire?rewriteBatchedStatements=true&amp;useUnicode=true&amp;characterEncoding=utf8`

![image.png](https://i.loli.net/2019/08/25/QLHKE5ZvxAm7Ijb.png)

附：

[Spark](<https://www.igniterealtime.org/projects/spark/index.jsp>)

---

参考文章：

[一步一步教你XMPP环境搭建](https://www.jianshu.com/p/f51e007be982)



