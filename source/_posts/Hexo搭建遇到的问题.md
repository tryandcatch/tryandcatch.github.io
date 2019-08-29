---
title: Hexo搭建遇到的问题
tags: 
- problems
- Hexo
urlname: Build-hexo-problems
date: 2019-03-08 20:57:50
---
## 部署需要输入git用户名密码的问题

本地已经配置了ssh的方式访问git，并且使用ssh -T git@github.com 命令返回的也是正常的，但是在使用hexo d（或hexo deploy）进行部署的时候出现了需要输入用户名密码的问题：

![image-20190307212646367](https://ws3.sinaimg.cn/large/006tKfTcgy1g0v4qdqjcsj30hd04fq3g.jpg)

解决办法:修改`_confi.yml`，将`deploy`节点下的`repo`改为ssh的git，例：`git@github.com:USERNAME/REPO.git`

> [Why is Github asking for username/password when following the instructions on screen and pushing a new repo?](https://stackoverflow.com/questions/10909221/why-is-github-asking-for-username-password-when-following-the-instructions-on-sc)

## 写作遇到的“小问题”

> 准确来说不算是问题，算是自己的野路子。

直接将以后的md文件复制到`hexo` 博客根目录下的`source/_posts`文件夹下，然后使用`hexo clean` `hexo g` `hexo d`部署后访问的博客是没有title的，而且文章标题没有链接，点击不会跳转.以后写作还是老老实实按照流程来：`hexo new post "title"` :)
