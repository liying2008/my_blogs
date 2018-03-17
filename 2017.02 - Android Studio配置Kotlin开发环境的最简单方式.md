---
title: Android Studio配置Kotlin开发环境的最简单方式
date: 2017-5-11
tags: [Android, Kotlin]
categories: Kotlin
---

# 第一步：安装Kotlin插件
打开Settings面板，找到Plugins选项，点击Browse repositories(浏览仓库)，输入“Kotlin”查找，然后安装即可。安装完成之后需要重启Android Studio (切记！)。

安装完成之后如下图所示。

![安装Kotlin插件](http://upload-images.jianshu.io/upload_images/2883714-ef663d09c5a2145b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

插件当前的最新版本是<code>1.1.2-release-Studio-2.3-3</code>。

# 第二步：配置Kotlin开发环境
点击菜单栏的“Tools”选项，选择“Kotlin”，然后选择“Configure Kotlin in Project”。如下图所示。

![配置Kotlin开发环境](http://upload-images.jianshu.io/upload_images/2883714-c0e26b64474d952d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在弹出的窗口中选择**需要使用Kotlin的模块**和**Kotlin编译器和运行时的版本**，如下图所示。

![选择模块和Kotlin版本](http://upload-images.jianshu.io/upload_images/2883714-66381e59a50da4b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击“OK”之后，Kotlin插件会自动开始配置。配置完成之后，同步一下工程（Sync Project）即可。

> [可选]：在菜单栏中点击“*Code*”菜单项，选择“*Convert Java File to Kotlin File*”即可根据之前配置将已有的Java文件转换为Kotlin文件。

# 附：推荐配置
> 打开模块下的<code>build.gradle</code>文件，在<code>apply plugin: 'kotlin-android'</code>下面加入一行：<code>apply plugin: 'kotlin-android-extensions'</code>。这是一个Kotlin的扩展模块，可以让Activity自动关联xml布局中的View而不需要<code>findViewById</code>。
详情请参考：[http://kotlinlang.org/docs/tutorials/android-plugin.html](http://kotlinlang.org/docs/tutorials/android-plugin.html)

# 附：使用Kotlin编写单元测试
在Android开发中免不了要进行各种单元测试，使用Kotlin编写单元测试可以简化代码，提高效率。  
将工程切换到Project视图，展开模块下的src目录，这个目录下默认会有三个文件夹。main文件夹通常用来存放模块代码；androidTest文件夹通常用来存放Android相关的单元测试；test文件夹通常用来存放Java（Kotlin）相关的单元测试。

## Kotlin的单元测试
在测试包下新建一个Kotlin Class，例如命名为UnitTest1。在这个类中可以编写多个测试方法，不详细叙述。

```kotlin
package cc.duduhuo.kotlintest

import org.junit.Test

import org.junit.Assert.*

class UnitTest1 {
    @Test
    fun addition_isCorrect() {
        assertEquals(4, (2 + 2).toLong())
    }
}
```
## Android的单元测试
在测试包下新建一个Kotlin Class，例如命名为AndroidTest1。在这个类中可以编写多个测试方法，不详细叙述。

```kotlin
package cc.duduhuo.kotlintest

import android.support.test.InstrumentationRegistry
import android.support.test.runner.AndroidJUnit4
import org.junit.Assert.assertEquals
import org.junit.Test
import org.junit.runner.RunWith

@RunWith(AndroidJUnit4::class)
class AndroidTest1 {

    @Test
    fun useAppContext() {
        // Context of the app under test.
        val appContext = InstrumentationRegistry.getTargetContext()

        assertEquals("cc.duduhuo.kotlintest", appContext.packageName)
    }
}
```

# 附：有关Kotlin的一些有用资料
1. Kotlin官网：http://kotlinlang.org/  
1. Kotlin用户手册（英文）：http://kotlinlang.org/docs/reference/  
1. Kotin开源地址：https://github.com/JetBrains/kotlin  
1. 官方介绍如何开始使用Kotlin：http://kotlinlang.org/docs/tutorials/getting-started.html  
1. 与Kotlin相关一些库、框架和应用：http://kotlinlang.org/docs/resources.html  