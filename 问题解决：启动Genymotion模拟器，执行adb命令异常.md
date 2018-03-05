---
title: 问题解决：启动Genymotion模拟器，执行adb命令异常
date: 2016-12-07
tags: [Genymotion]
categories: 问题解决
---
## 异常信息

```
adb server version (32) doesn't match this client (36); killing...
error: could not install *smartsocket* listener: cannot bind to 127.0.0.1:5037: 通常每个套接字地址(协议/网络地址/端口)只允许使用一次。 (10048)
could not read ok from ADB Server
* failed to start daemon *
error: cannot connect to daemon
```

## 解决办法

### 1、设置Android SDK路径

如图所示，选择“Use custom Android SDK tools”选项，然后选择一个本地的SDK路径。

![填写自己的SDK路径](http://img.blog.csdn.net/20161207223607903)

### 2、添加PATH环境变量

将上图中选择的路径添加进PATH变量里，操作流程如下图所示。

![添加环境变量](http://img.blog.csdn.net/20161207224454830)

### 3、重启Genymotion模拟器

更改完成之后，重新启动Genymotion模拟器，再次执行adb相关命令就可以正常显示结果了。