---
title: windows环境nodejs版本管理
date: 2017-08-12
categories: nodejs
tags: nodejs
---
由于业务需要使用nodejs，由于nodejs版本更新频繁，为方便使用通过安装nodejs版本管理器来解决这个问题，
实验了两种方案，踩过一点小坑，记录一下
## [nvmw方案](https://fengmk2.com/blog/2014/03/node-env-and-faster-npm.html)
nvmw方案是cnode社区上给出的nodejs版本管理方案，尝试按照提示步骤进行，期间遇到了几个问题
```bash
nvmw install v6.11.2
```
提示“没有文件扩展".js"的脚本引擎”
这样的错误,原因是因为JS扩展名的文件被其他软件关联了，需要取消关联。
采用的解决方案是：
在运行中输入“regedit”进入注册表，
把\[HKEY_CLASSES_ROOT\.js\]项下的那个默认值改成"JSFile"就可以正常运行JS 文件了。
解决了这个问题之后重新执行命令，又出现新的错误cscript不是内部或外部命令,未找到合适的解决方案，因此寻找新的方案。
有成功的例子详见[这篇文章](http://blog.csdn.net/duanyachao/article/details/51822015)
## [nvm-windows方案](https://github.com/coreybutler/nvm-windows)
此方案比较简单，按照说明安装软件即可，顺利达到目的
中间过程中踩了一个坑，在nvm use的命令的时候失效，原因是软件我安装在programe file下了，路径中存在空格，
重新安装软件在全英文路径下，安装成功。建议安装路径不要又空格和中文。
