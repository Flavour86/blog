---
layout: title
title: Weinre 使用入门
date: 2018-08-10 14:44:04
tags:
- Weinre
- 调试
categories:
- 工具使用
---

# Weinre 使用入门

### 介绍

Weinre(Web Inspector Remote)是一款基于Web Inspector(Webkit)的远程调试工具， 它使用JS编写， 可以让我们在电脑上直接调试运行在手机上的远程页面。 与传统的Web Inspector的使用场景不同， Weinre的使用场景如下图， 调试的页面在手机上， 调试工具在PC的chrome， 二者通过网络连接通信。

### 安装

```
npm -g install weinre
```

### 运行

命令行中输入

```
weinre
```
若运行成功，输出如下：

![image](https://camo.githubusercontent.com/b08079949243875601a2ceacd7a6ff53c3a7138c/687474703a2f2f68616974616f2e6e6f732e6e6574656173652e636f6d2f30356136653635666139353334623963386636356131663531656439383963612e6a7067)

### 参数
Weinre还提供了下面的启动参数:

1. help : 显示Weinre的Help 
2. httpPort   [portNumber] : 设置Weinre使用的端口号， 默认是8080
3. boundHost  [hostname | ip address | -all-] : 默认是'localhost'， 这个参数是为了限制可以访问Weinre Server的设备， 设置为-all-或者指定ip， 那么任何设备都可以访问Weinre Server。
4. verbose   [true | false] : 如果想看到更多的关于Weinre运行情况的输出， 那么可以设置这个选项为true， 默认为false；
5. debug   [true | false] : 这个选项与--verbose类似， 会输出更多的信息。默认为false。
6. readTimeout   [seconds] : Server发送信息到Target/Client的超时时间， 默认为5s。
7. deathTimeout   [seconds] : 默认为3倍的readTimeout， 如果页面超过这个时间都没有任何响应， 那么就会断开连接。

### 使用
在页面中，有两部分是我们要使用的，第一部分是Access Points，如下图：

![image](https://camo.githubusercontent.com/05a033afa28fe82757d996d22d3d20821feb1cd2/687474703a2f2f68616974616f2e6e6f732e6e6574656173652e636f6d2f31326333643831613733333934663065623731613631643037613632626231622e6a7067)

第二个部分是Target Script，如下图，这个地址是系统根据我们启动Weinre服务时的参数设置生成的target-script.js文件的链接地址。我们需要将这个js文件嵌入到待测试的页面中。要注意的是不要使用localhost:8080打开Weinre服务，否则生成的TargetScript链接也以localhost开头，这样直接复制到手机，就无法获取到文件了。

![image](https://camo.githubusercontent.com/6aefd3336c34fa1944a9df0594f710e711748f8c/687474703a2f2f68616974616f2e6e6f732e6e6574656173652e636f6d2f37616362303665373336396334313962393530623936323466653464666262652e6a7067)

打开后的DebugClient如下图:

![image](https://camo.githubusercontent.com/59e75aaaf65edc200181124eecca6ef219faef6e/687474703a2f2f68616974616f2e6e6f732e6e6574656173652e636f6d2f37656563663434343564633134656465396666313238353032386365646432382e6a7067)