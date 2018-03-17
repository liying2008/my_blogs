---
title: Android Studio使用Vector Asset的注意事项
date: 2016-12-22
tags: [Android, Vector]
categories: Android基础
---

Vector是Android 5.0之后新增加的一项特性，目前已经可以兼容到Android 5.0之前的版本。但使用过程中依然还是可能产生一些兼容性的问题。

# 1、Android Studio创建Vector Asset

在<code>res</code>目录下的<code>drawable</code>目录上单击右键，选择New --> Vector Asset，弹出Asset Studio对话框。
目前有两种方式创建vector资源，一种是通过Material Icon生成vector资源，一种是通过SVG或PSD文件生成vector资源。具体操作过程比较简单，不再详述。

![Asset Studio](http://img.blog.csdn.net/20161222152745875)

> 注意：**最好将生成的vector drawable资源放在<code>drawable</code>目录下或<code>drawable-anydpi</code>目录下。**


# 2、兼容问题解决

如果使用的Android Studio版本为2.2以上，gradle版本也较新，生成的vector drawable资源也放在了<code>drawable</code>目录下或<code>drawable-anydpi</code>目录下，则一般就直接可以兼容Android 5.0之前操作系统。

vector drawable资源的使用方式如下，<code>ic_egg05_got.xml</code>为生成的vector 资源。
```xml
<ImageView
    android:id="@+id/ivVector"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@drawable/ic_egg05_got"/>
```

如果不满足以上的条件或由于其他原因导致Android 5.0之前的设备无法使用vector资源，也有别的解决办法。

如果设备无法使用vector资源，则会导致应用崩溃，异常信息大致如下所示：

```
Caused by: android.view.InflateException: Binary XML file line #11: Error inflating class ImageView

Caused by: android.content.res.Resources$NotFoundException: File res/drawable-nodpi-v4/egg01_got.xml from drawable resource ID #0x7f020053

Caused by: org.xmlpull.v1.XmlPullParserException: Binary XML file line #2: invalid drawable tag vector
```

## 2.1、修改build.gradle

打开该模块下的<code>build.gradle</code>文件。

 - 如果当前使用的gradle版本为2.0以上，在<code>android</code>节点下的<code>defaultConfig</code>节点下加入一行代码：

```gradle
vectorDrawables.useSupportLibrary = true
```

 - 如果使用的gradle版本为2.0以下，1.5以上，则需要在<code>android</code>节点下的<code>defaultConfig</code>节点下加入如下一行代码：

```gradle
generatedDensities = []
```
并在<code>android</code>节点下，<code>defaultConfig</code>节点后面加入

```gradle
aaptOptions {
	additionalParameters "--no-version-vectors"
}
```

推荐使用较新的gradle版本以免不必要的麻烦。

之后，在<code>dependencies</code>中引入<code>appcompat-v7</code>兼容包，版本需要<code>23.2</code>以上。

```gradle
compile 'com.android.support:appcompat-v7:25.0.1'
```

## 2.2、修改布局文件

如果在布局文件中使用了vector drawable资源的话，则需要修改其使用方式为：

```xml
app:srcCompat="@drawable/ic_egg05_got"
```

需要在根布局中引入命名空间<code>xmlns:app="http://schemas.android.com/apk/res-auto"</code>。

## 2.3、修改Activity

需要将加载vector drawable资源的Activity继承自<code>AppCompatActivity</code>。

---
做完以上工作之后，Android 5.0之前的设备就可以支持使用vector drawable资源了。

