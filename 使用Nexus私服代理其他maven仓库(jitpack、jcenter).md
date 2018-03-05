---
title: 使用Nexus私服代理其他maven仓库(jitpack、jcenter)
date: 2017-03-24
tags: [Android, Nexus, Maven, jitpack, jcenter]
categories: 相关技能
---
# Nexus下载及运行

Nexus3下载地址：https://support.sonatype.com/hc/en-us/articles/218637467-Download-Nexus-Repository-Manager-3  
Nexus3使用文档：http://books.sonatype.com/nexus-book/reference3/  

> 从Nexus Repository Manager 3.1.0-04开始，Nexus不再提供各平台的二进制安装文件。

**下面以Windows平台为例：**

下载Nexus压缩包解压之后，进入bin目录，在命令提示符中运行<code>nexus.exe /run</code>即可启动nexus服务。按<code>Ctrl + C</code>可关闭Nexus。

### （可选）安装为Windows服务：
以管理员身份打开命令提示符，同样进入bin目录下，运行：<code>nexus.exe /install</code>，即可将nexus安装为系统服务，然后在Windows服务管理窗口设置为开机启动即可。  
如果想删除nexus服务，同样在bin目录下，以管理员身份打开命令提示符并运行<code>nexus.exe /uninstall</code>。

### （可选）添加环境变量：
将Nexus的bin目录加入系统PATH变量中，以后就可以在任意目录中执行<code>nexus.exe /run</code>命令了。

# 使用Nexus代理其他仓库

打开Nexus管理面板。如果Nexus安装在本机，则默认地址是<code>http://127.0.0.1:8081/</code>。然后使用管理员帐户登录系统，管理员帐户的默认用户名和密码是：<code>admin</code>和<code>admin123</code>，进入系统之后可以创建其他用户，不再赘述。

首先需要按如下步骤创建一个仓库：

![创建仓库](http://img.blog.csdn.net/20170324151310072)

选择仓库类型为：maven2 (proxy)，进入仓库配置。

**jitpack仓库配置如下图所示：**

![jitpack仓库配置](http://img.blog.csdn.net/20170324151729828)

**jcenter仓库配置如下图所示：**

![jcenter仓库配置](http://img.blog.csdn.net/20170324151935221)

> jitpack仓库URL：https://jitpack.io  
> jcenter仓库URL：http://jcenter.bintray.com/  

# 让Android Studio使用本地Nexus私服

首先在<code>maven-public</code>仓库中将上一步添加的代理仓库加入，如下图所示：

![maven-public配置](http://img.blog.csdn.net/20170324152721658)

之后打开Android Studio项目，在Project的<code>build.gradle</code>中将原本的<code>allprojects</code>闭包改为：
```gradle
allprojects {
    repositories {
        maven { url "http://localhost:8081/repository/maven-public/" }
    }
}
```

之后Sync Project或Make Project即可。

### 查看仓库内容

如图所示，在Components下可以浏览所有仓库中的内容。

![查看仓库内容](http://img.blog.csdn.net/20170324153606322)

下图是我jcenter仓库中的内容：

![我的jcenter仓库中的内容](http://img.blog.csdn.net/20170324154030789)