---
title: 使用Android Studio进行多渠道打包
date: 2016-12-17
tags: [Android, 打包, 多渠道]
categories: 相关技能
---
以**友盟统计**为例。

在<code>AndroidManifest.xml</code>文件中声明channel信息，如下。

```xml
<meta-data
    android:name="UMENG_APPKEY"
    android:value="产品的APPKEY" />
<meta-data
    android:name="UMENG_CHANNEL"
    android:value="${UMENG_CHANNEL_VALUE}" />
```

在主module下的<code>build.gradle</code>文件中<code>android</code>节点下配置<code>productFlavors</code>，如下。

```gradle
productFlavors {
    fir {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "fir"]
    }
    _360 {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "360"]
    }
    baidu {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "baidu"]
    }
    _91 {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "91"]
    }
    hiapk {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "hiapk"]
    }
    tqq {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "tqq"]
    }
    wandoujia {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia"]
    }
    xiaomi {
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi"]
    }
}
```

 >  **上面的fir、_360、baidu， _91等是Flavor名称，双引号里面的fir、360、baidu、91等是渠道名称。** 

最后可以 在<code>android</code>节点下的<code>buildTypes</code>节点下指定输出APK的文件名。
比如要输出文件名格式为“ipgw_4.2.5_fir.apk”，其中，ipgw为产品名称，4.2.5为版本名称，fir为渠道名称，各部分之间用下划线分割。
代码片段如下：

```gradle
applicationVariants.all {
    variant ->
        variant.outputs.each {
            output ->
                def outFile = output.outputFile
                if (outFile != null && outFile.name.endsWith(".apk")) {
                    def fileName = "ipgw_4.2.5_${variant.productFlavors[0].name.replaceAll("_", "")}.apk"
                    output.outputFile = new File(outFile.parent, fileName)
                }
        }
}
```

到此，配置就算完成了。

点击Android Studio菜单栏中的“Build”，选择“Generate Signed APK...”，选择keystore，输入密码等，点击“Next”，出现渠道选择界面，如下图。

![选择渠道](http://img.blog.csdn.net/20161217161351553)

此处可以选择需要打包的渠道，当然也可以按<code>Ctrl + A</code>全选。
点击Finish开始打包。
打包完成之后，在工程的主模块目录下就可以看到已经经过 **打包、签名、命名** 好的APK文件了，如下图所示。

![打包完成](http://img.blog.csdn.net/20161217162538709)
