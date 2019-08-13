---
title: linux下查看IP
date: 2019-08-12 11:26:42
categories:
- 收藏
- js技术
tags: 
- linux
---
在微信公众号的开发过程中需要将服务器外网IP地址加入微信白名单才能调用其提供的API。
这里记录下如何查看linux服务器外网IP的命令。

```shell
ifconfig -a
```

这个是linux最常见的查看ip命令。但是有时候会出现下面的情况，查不到结果。

![查看IP失败](http://res.troubledot.cn/checkip.png)

原因不深究了，这种时候我们可以试试看访问一些网站查IP地址。

```shell
curl ifconfig.me
curl www.trackip.net/ip
curl ipecho.net/plain
```
