---
title: 使用webhook自动部署博客
tags: 
 - Hexo
 - webhook
urlname: use-webhook-auto-deploy-hexo-blog
date: 2019-03-11 18:26:00
---

> 现在写文章都是每次在本地写好后，都要去服务器上`git pull` 同步一下，这样比较麻烦。后来找到可以通过在`GitHub`添加`webhook`的方式实现，只要`repo`有`push`操作，那么就通知服务器部署博客。 查看了很多文章，但是没有成功，可能我的环境和别人不一样，不能照着他人那样依葫芦画瓢…记录下自己的实践过程

##  前提

服务器系统：CentOS 7

服务器已经安装好 `node`/`git`(能ssh访问)/`hexo`/`pm2`/`nginx`

并且博客目录已和`GitHub`仓库同步(也就是在博客目录下输入`git status` 会显示`git`的信息)

文件目录：

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1022lkjbsj30c308y0uj.jpg)



## 安装github-webhook-handler

root 用户(全局安装)：

`npm install -g github-webhook-handler`

:warning: 我这里是用这个安装后，到博客目录下运行js还是会报找不到`module`的错，:face_with_thermometer:解决办法：在博客目录再执行一次：`npm install github-webhook-handler` 

## 编写代码

进入到博客`deploy`目录下，新建一个`deploy.sh`脚本（用于在收到`GitHub` `push`操作后进行`pull`博客代码并部署）：

`vim deploy.sh`（不建议使用这个sh）

```shell
#!/bin/bash

WEB_PATH='/home/git/blog/public'

echo "-----start deploy-----"
cd $WEB_PATH
echo "-----pulling source code-----"
git reset --hard origin/master && git clean -f
git pull && git checkout master
echo "-----finished-----"
```

`WEB_PATH`改成博客静态页面（就是存放那些`js` `html`的目录）所在的目录，我这里是`public`下

~~:warning: 注意：上面的sh脚本会将博客仓库（我这里是`public`目录）已有的改动会丢弃掉，因此最好不要在博客仓库中改动文件，我之前就是在`public`目录下新建了`deploy.sh`和`webhook.js`结果`push`后部署将我改动的文件给删掉了... :anger:~~

---

**2019-03-14 16:28 update:**

**上面的`sh` 有些问题：本地使用`hexo d ` 将博客 `push`到 `GitHub`上后`webhook`也显示成功了，但是访问博客还是会发现是没有更新。。。**

因此将上面的sh修改下(加上了部署的耗时统计)：

```shell
#!/bin/bash
start_time=`date --date='0 days ago' "+%Y-%m-%d %H:%M:%S"`
WEB_PATH='/home/git/blog/public'
echo "-----start deploy,StartTime: $start_time-----"
cd $WEB_PATH
echo "-----pulling source code-----"
#git reset --hard origin/master && git clean -f
#git pull && git checkout master
git pull origin master
finish_time=`date --date='0 days ago' "+%Y-%m-%d %H:%M:%S"`
duration=$(($(($(date +%s -d "$finish_time")-$(date +%s -d "$start_time")))))
echo "Time consuming: $duration"
echo "-----finished.EndTime: $finish_time-----"
```

---

这里使用的是`nodejs`来实现`webhook`服务。因此在博客`deploy` 目录新建一个`webhook.js`:

`vim webhook.js`:

```javascript
var http=require('http')
var createHandler = require('github-webhook-handler')
//这里的path和secret 改成你自己想要设定的值，一会需要填写到GitHub webhooks中
var handler = createHandler({ path: '/webhooks_push', secret: 'INSERT_YOUR_SECRET' })
function run_cmd(cmd, args, callback) {
  var spawn = require('child_process').spawn;
  var child = spawn(cmd, args);
  var resp = "";
  child.stdout.on('data', function(buffer) { resp += buffer.toString(); });
  child.stdout.on('end', function() { callback (resp) });
}
handler.on('error', function (err) {
  console.error('Error:', err.message)
})
handler.on('push', function (event) {
  console.log('Received a push event for %s to %s',
    event.payload.repository.name,
    event.payload.ref);
    //这里是收到GitHub push操作后会执行的脚步就是上面写的
    run_cmd('sh', ['./deploy.sh'], function(text){ console.log(text) });
})
try {
  http.createServer(function (req, res) {
    handler(req, res, function (err) {
      res.statusCode = 404
      res.end('no such location')
    })
  }).listen(7777)//监听7777端口
}catch(err){
  console.error('Error:', err.message)
}
```

接着运行`webhook` `node`服务，我这里使用的是`pm2`的方式启动：

在博客目录下：

`pm2 start webhook.js `

不使用pm2，常规后台运行并将日志记录到`webhook.log`中:

`nohup node webhook.js > webhook.log &`

## GitHub配置webhook

打开博客GitHub的repo>`Settings`>`Webhooks`>`Add webhook`:

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1007b54g1j31q60oqjx7.jpg)



填写服务器`webhook`的访问地址和在`webhook.js`的`secret`：

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g100bbj9k4j30xz0u07ak.jpg)

## 最后

按理说照着上面一顿操作后，就可以实现自动化部署博客了，但是。。。我这个就是不行，不能访问，`GitHub` `webhook` 显示的是`Service Timeout`,后来才发现是我的服务器使用的是Nginx，没有处理转发这个请求。。。。

因此在nginx中要配置一个代理转发，然后将`GitHub` `webhook`中的链接改一下：

![nginx](https://ws3.sinaimg.cn/large/006tKfTcgy1g1033vz6ohj30g60bvq6z.jpg)



![githubwebhook-update](https://ws2.sinaimg.cn/large/006tKfTcgy1g1035j065wj312k0istaz.jpg)

<hr/>

> 参考文章:
>
> [快速搭建Hexo博客+webhook自动部署+全站HTTPS](https://www.gaoshilei.com/2017/10/30/hexo-init/)
>
> [将 Hexo 博客发布到自己的服务器上](https://blog.mutoe.com/2017/deploy-hexo-website-to-self-server/)
>
> [使用Github的webhooks进行网站自动化部署](https://aotu.io/notes/2016/01/07/auto-deploy-website-by-webhooks-of-github/index.html)



