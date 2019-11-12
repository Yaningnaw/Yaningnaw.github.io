---
layout: post
title: '不改变tomcat端口解决端口占用问题'
subtitle: 'java'
date: 2019-09-23
categories: 技术
tags: Spring
---

今天早上当我正常修改代码构建新项目的时候，运行之后才发现使用postman访问接口访问不到，查看springboot报错信息是这样的：
```Cmd
The Tomcat connector configured to listen on port 9000 failed to start. The port may already be in use or the connector may be misconfigured.

Action:

Verify the connector's configuration, identify and stop any process that's listening on port 9000, or configure this application to listen on another port.
```

很明显是端口占用问题，其实更改配置就可以解决这个问题。
例如：

```Propertise
server.port=7000
```

但是我并不想改变项目的端口信息，就想找到windows内部该端口杀死这个进程来解决问题，我的设想是这样的：

```Shell
netstat -ano|findstr 9000
返回该端口号
taskkill /f /t /im 该端口号
```

但是发现并不能从Windows中查找到相应的端口占用进程！！！！

我百思不得其解，最后想到一个比较暴力的解决方式：

```Shell
netsh winsock reset
```
执行后重启解决了该问题  
下面我们看看这条命令的作用：
---
netsh winsock reset
这个命令作用是重置 Winsock 目录。如果一台机器上的Winsock协议配置有问题的话将会导致网络连接等问题，就需要用netsh winsock reset命令来重置Winsock目录借以恢复网络。这个命令的好处是可以重新初始化网络环境，以解决由于软件冲突、病毒原因造成的参数错误问题。
---


我的博客：https://yaningnaw.github.io/