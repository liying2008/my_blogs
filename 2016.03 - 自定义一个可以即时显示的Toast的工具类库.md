---
title: 自定义一个可以即时显示的Toast的工具类库
date: 2016-11-25
tags: [Android, Toast, Library]
categories: 我的开源
---
## 1 AppToast介绍

### 1.1 实现方式  

全局只有一个Toast实例，每次调用<code>show()</code>方法显示Toast前都要先取消上次的Toast显示，然后显示本次的消息。  

首先创建一个名为AppToast的类，在里面定义一个全局静态Toast对象和一个全局Application对象的弱引用。
```java
private static Toast toast = null;  // Global Toast
private static WeakReference<Application> app;
```
定义一个<code>init()</code>方法，用于得到用户传入的Application实例。
```java
public static void init(Application application) {
    app = new WeakReference<Application>(application);
}
```
封装<code>showToast()</code>方法，方便调用。
```java
/**
 * Display Toast
 *
 * @param resId The resource id of the string resource to use.  Can be formatted text.
 */
public static void showToast(@StringRes int resId) {
    if (toast != null) {
        toast.cancel();
        toast = null;
    }
    toast = Toast.makeText(app.get(), resId, LENGTH_SHORT);
    toast.show();
}
```
也可以封装一个<code>getToast()</code>方法用于得到Toast实例，允许我们设置其属性，便于自定义Toast显示的效果。
```java
/**
 * Get a Toast object <br>
 * Need to call show() method to be displayed
 *
 * @return Toast object.
 */
public static Toast getToast() {
    if (toast != null) {
        toast.cancel();
        toast = null;
    }
    toast = Toast.makeText(app.get(), "", Toast.LENGTH_SHORT);
    return toast;
}
```
### 1.2 使用方法

首先创建一个类继承自<code>Application</code>，在其<code>onCreate()</code>方法中调用我们之前写的<code>init()</code>方法进行AppToast类的初始化。
```java
public class MyApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        // 初始化AppToast库
        AppToast.init(this);
    }
}
```

> 注意：不要忘记在AndroidManifest.xml文件中的application节点下配置android:name属性。

```xml
<application
    ...
    android:name=".MyApplication" >
    <activity android:name=".MainActivity" >
        ...
    </activity>
</application>
```
之后就可以在代码中进行使用了，比如：
```java
AppToast.showToast(R.string.toast2);
```
和
```java
Toast toast = AppToast.getToast();
toast.setGravity(Gravity.TOP | Gravity.CENTER, 0, 0);
toast.setText("自定义Toast");
toast.setDuration(Toast.LENGTH_SHORT);
toast.show();
```
显示效果如下图：
![显示效果](http://img.blog.csdn.net/20161125215420046)

开源库、样例工程、详细文档下载地址：  
[liying2008/ApplicationToast](https://github.com/liying2008/ApplicationToast)

该库已上传至jcenter仓库，使用Android Studio可以通过在线依赖引用的方式引入该库。
```gradle  
dependencies {
  compile 'cc.duduhuo.applicationtoast:applicationtoast:0.3'
}
```

## 2 CusToast介绍

### 2.1 功能介绍

CusToast是一个具有即时显示并且内置了10种样式的Toast工具库，现在简单介绍其实现原理。
在CusToast类中定义了一个枚举类型Style，即Toast显示的样式。
```java
public enum Style {
    DEFAULT,
    LIGHT_BLUE,
    BLUE,
    LIGHT_RED,
    RED,
    LIGHT_GREEN,
    GREEN,
    LIGHT_YELLOW,
    YELLOW,
    GRAY_1
}
```
为了方便对Toast对象进行操作，我们创建一个自定义的Toast类，其继承自Toast，方便我们扩展Toast的功能，比如显示带图片的Toast和显示带副标题的Toast。
通过向<code>DToast</code>类的<code>setView()</code>方法传入样式名，得到不同样式的<code>DToast</code>。
```java
/**
 * Add a view to CusToast.
 *
 * @param application this application.
 * @param style       the style of CusToast.
 * @return current instance.
 */
public DToast setView(Application application, CusToast.Style style) {
    dView = View.inflate(application, R.layout.ddh_cus_toast, null);
    dText = (TextView) dView.findViewById(R.id.dText);
    setStyle(style);
    super.setView(dView);
    return this;
}
```
其余方法和布局文件请参考文末链接。

<code>CusToast</code>类中的<code>showToast()</code>方法如下所示。
```java
/**
 * Display Toast.
 *
 * @param text The resource id of the string resource to use.  Can be formatted text.
 */
public static void showToast(@StringRes int text) {
    clearToast();
    toast = new DToast(app.get());
    toast.setView(app.get(), defStyle);
    toast.setText(text);
    toast.setDuration(Toast.LENGTH_SHORT);
    toast.show();
}
```
<code>clearToast()</code>方法如下，目的就是立即取消正在显示的“旧”Toast。
```java
/**
 * Clear an existing CusToast.
 */
private static void clearToast() {
    if (toast != null) {
        toast.cancel();
        toast = null;
    }
}
```
### 2.2 使用方法

首先，和AppToast一样，在自己项目的<code>Application</code>类中初始化<code>CusToast</code>库，方法也和AppToast类似。
```java  
public class MyApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        // 初始化CusToast库（两种方式选其一）
        // 方式1：初始化同时指定CusToast的默认显示样式
        CusToast.init(this, CusToast.Style.RED);
        // 方式2：初始化，使用默认显示样式
        // CusToast.init(this);
    }
}
```
之后就可以在代码中进行使用了，比如：
```java  
CusToast.showToast("Toast 1");
```
```java  
CusToast.showToast("Toast 3", Toast.LENGTH_LONG, CusToast.Style.LIGHT_RED);
```
```java  
DToast toast = CusToast.getToast("自定义Toast");
toast.setCusToastGravity(Gravity.CENTER, 0, 0)
        .setTextSize(16)
        .setStyle(CusToast.Style.GRAY_1)
        .setTextColor(Color.WHITE)
        // .setBackground(R.mipmap.ic_launcher)
        // .setBackgroundColor(0xffff3444)
        .setCusToastDuration(Toast.LENGTH_SHORT)
        .show();
```
在此列举一下CusToast的几种内置样式。

| Style | 预览
| :--: | :--: 
| DEFAULT | ![DEFAULT](https://github.com/liying2008/CusToast/raw/master/pic/style_default.png)
| LIGHT_BLUE | ![LIGHT_BLUE](https://github.com/liying2008/CusToast/raw/master/pic/style_lightblue.png)
| BLUE | ![BLUE](https://github.com/liying2008/CusToast/raw/master/pic/style_blue.png)
| LIGHT_RED | ![LIGHT_RED](https://github.com/liying2008/CusToast/raw/master/pic/style_lightred.png)
| RED | ![RED](https://github.com/liying2008/CusToast/raw/master/pic/style_red.png)
| LIGHT_GREEN | ![LIGHT_GREEN](https://github.com/liying2008/CusToast/raw/master/pic/style_lightgreen.png)
| GREEN | ![GREEN](https://github.com/liying2008/CusToast/raw/master/pic/style_green.png)
| LIGHT_YELLOW | ![LIGHT_YELLOW](https://github.com/liying2008/CusToast/raw/master/pic/style_lightyellow.png)
| YELLOW | ![YELLOW](https://github.com/liying2008/CusToast/raw/master/pic/style_yellow.png)
| GRAY_1 | ![GRAY_1](https://github.com/liying2008/CusToast/raw/master/pic/style_gray1.png)

**其他样式**

| 样式 | 预览
| :--: | :--: 
| CusToastWithSub | ![CusToastWithSub](https://github.com/liying2008/CusToast/raw/master/pic/custoast_with_sub.png)
| CusToastWithIcon | ![CusToastWithIcon](https://github.com/liying2008/CusToast/raw/master/pic/custoast_with_icon.png)

开源库、样例工程、详细文档下载地址：  
[liying2008/CusToast](https://github.com/liying2008/CusToast)

该库已上传至jcenter仓库，使用Android Studio可以通过在线依赖引用的方式引入该库。
```gradle  
dependencies {
  compile 'cc.duduhuo.custoast:custoast:0.2'
}
```
