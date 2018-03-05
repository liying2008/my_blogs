---
title: Android Studio2.2中JNI的简单使用
date: 2016-11-27
tags: [Android, JNI]
categories: NDK
---

## 方式1：直接创建包含C++支持的项目
如下图所示，创建项目时，勾选“Include C++ Support”选项。

![Jni include](http://img.blog.csdn.net/20161127134708974)

## 方式2：手动引入C++支持
在没有勾选“Include C++ Support”选项的情况下，创建出来的项目大致是这样子的（Project视图）：

![不含JNI的目录结构](http://img.blog.csdn.net/20161127135338837)

***引入C++支持的步骤：***

###1、配置NDK路径
在项目节点上点右键，选择“Open Module Settings”。
配置Android NDK Location，如下图所示：
![NDK Location](http://img.blog.csdn.net/20161127142125347)
> 需要下载NDK、CMake等工具。

### 2、编写CMakeLists.txt文件

在app目录下新建一个文件(File)，名称为 "CMakeLists.txt" 。
在其中添加如下内容：  

```
cmake_minimum_required(VERSION 3.4.1) # CMake版本

add_library(
             test-lib # 动态库的名称

             SHARED # 作为共享库（动态引入）

             src/main/cpp/test-lib.cpp # cpp源文件路径
            )

find_library(
                log-lib
                log
            )

# 被链接的目标库的名称
target_link_libraries(
                       test-lib
                       ${log-lib}
                     )

```

### 3、在Java文件中创建native方法、加载动态库并使用

- activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="cc.duduhuo.jnidemo.MainActivity">

    <TextView
        android:id="@+id/tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="默认字符串" />
    
    <Button
        android:id="@+id/btn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```

- MainActivity.java

```java
public class MainActivity extends AppCompatActivity {
    private TextView tv;
    private Button btn;

    // 加载动态库
    static {
        System.loadLibrary("test-lib");
    }

    // 创建native方法
    public native String getTextFromCpp();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        tv = (TextView) findViewById(R.id.tv);
        btn = (Button) findViewById(R.id.btn);
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 使用native方法
                tv.setText(getTextFromCpp());
            }
        });
    }
}
```

### 4、编写C++代码

首先我们需要在app/src/main目录下创建一个目录(Directory)，名称为“cpp”。
之后在cpp目录下创建一个cpp文件(C/C++ Source File)，名称为“test-lib.cpp”。
在其中编写如下代码：

- test-lib.cpp

```cpp
#include <jni.h>
#include <string>

extern "C"
jstring
Java_cc_duduhuo_jnidemo_MainActivity_getTextFromCpp(
        JNIEnv* env,
        jobject /* this */) {
    std::string text = "Text From C++";
    return env->NewStringUTF(text.c_str());
}
```
其中<code>jstring</code>是C++函数的返回值类型，<code>Java_cc_duduhuo_jnidemo_MainActivity_getTextFromCpp</code>是函数名称，由Java+包名+类名+native方法名构成。

### 5、配置app目录下的<code>build.gradle</code>文件

在<code>defaultConfig</code>节点下添加如下内容：

```gradle
externalNativeBuild {
    cmake {
        cppFlags ""
    }
}
```
在<code>android</code>节点下，添加如下内容：
```gradle
externalNativeBuild {
    cmake {
        path "CMakeLists.txt"
    }
}
```
完整代码如下：

- build.gradle

```gradle
apply plugin: 'com.android.application'

android {
    compileSdkVersion 24
    buildToolsVersion "25.0.0"
    defaultConfig {
        applicationId "cc.duduhuo.jnidemo"
        minSdkVersion 11
        targetSdkVersion 24
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        externalNativeBuild {
            cmake {
                cppFlags ""
            }
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:24.2.1'
    testCompile 'junit:junit:4.12'
}
```
### 6、构建项目并运行

点击“Make Project”，编译完成之后，再点击“Run 'app'”，运行项目。
效果如下图所示：
![Jni_Demo](http://img.blog.csdn.net/20161127145009324)

### 7、一些说明

以debug模式运行项目之后，会在<code>app/build/intermediates/cmake/debug/obj</code>目录下生成各个平台下的so库，名称为<code>libtest-lib.so</code>。
Android Studio会将这些平台下的so库全部打包进apk中。

### 8、代码下载

[Demo代码下载](http://download.csdn.net/detail/u012939909/9694888)