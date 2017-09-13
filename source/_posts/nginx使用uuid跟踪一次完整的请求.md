---
title: nginx使用uuid跟踪一次完整的请求
date: 2017-09-13 10:55:49
categories: nginx
tags: nginx uuid
---
nginx的配置中有一个字段$uuid，这个变量并不是nginx的常量，那为什么可以直接使用它呢，他的作用又是什么呢

### uuid的作用
Nginx功能很强大，但它还缺少一些常见的功能。 例如，访问日志中经常需要每个请求有唯一ID（UUID），以便可以通过UUID跟踪单个请求在多个服务中的处理流程。 如果发现某个case,可以从nginx中找出该条日志，进而根据UUID可以定位后续一系列服务模块的日志，从而快速定位问题。

### uuid如何生成
* 使用Lua，可自行搜索这个东西，文后也提供了两个链接
* 自行编写nginx module生成uuid，估计需要深入了解nginx才可得心应手

对如何生成uuid不仔细介绍，请自行搜索或者查看文后的参考文档

### uuid的使用
生成uuid之后如何使用它，首先再nginx中要先打印出这个uuid来标记这次唯一的请求，后边引用层可以通过不同的方式引用
* 如果你使用的是proxy模式你可以使用proxy_set_headers
* 如果你通过fastcgi协议传递给后端的PHP，你可以使用fastcgi_param
* 也可以使用nginx模块将uuid种在cookie中，后续服务从cookie中读取，在依次传递出去

我们使用的是最后一种方式，注意点是种cookie的时间点不要太长，或者后续服务取到cookie之后就清掉它。

关于排查问题还有一点建议，nginx是反向代理服务器，一台nginx可能打到后端多台服务中的某一台，最好在前端的页面上隐藏的打印出连接关系，方便快速定位日志；这种问题排查方法和app中留一些特殊的后门都是相同的；

### 参考
* [Using Lua in Nginx for unique request IDs and millisecond times in logs](https://blog.ryandlane.com/2014/12/11/using-lua-in-nginx-for-unique-request-ids-and-millisecond-times-in-logs/)
* [Nginx Unique Tracing ID](http://www.jianshu.com/p/5e103e1eb017)
* [使用nginx + lua 自定义access.log](http://bikong0411.github.io/2015/11/05/ngx-lua-custom-access-log-format.html)