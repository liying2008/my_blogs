---
title: Android中接口(Interface)的简单使用
date: 2016-08-20
tags: [Android, 接口, Java]
categories: Android基础
---

> Java中的接口可以被看作是`只包含常量和抽象方法的抽象类`  。

 可以使用如下方式定义一个接口：

``` java
public interface InterfaceDemo {
    int i = 10;
    void method1();
    int method2();
}
```
## 使用1： 解决“多重继承”的问题
Java语言本身是不支持类的多重继承(多重继承是指一个类从多个类继承而来，即一个类拥有多个超类)的，但一个类却可以实现多个接口。这样，我们可以将一些抽象方法定义在接口中，间接地达到多重继承的目的。
例如：

 - <code>MyInterface1.java</code>

```  java
public interface MyInterface1 {
    void fly();
}
```
 - <code>MyInterface2.java</code>

``` java
public interface MyInterface2 {
    void walk();
}
```
 - <code>Bird.java</code>

``` java
public class Bird implements MyInterface1, MyInterface2 {
    private static final String TAG = "Bird";
    @Override
    public void fly() {
        Log.i(TAG, "I can fly");
    }

    @Override
    public void walk() {
        Log.i(TAG, "I can walk");
    }
}
```

## 使用2： 定义一个规范（协议）
同一个接口可以有多个不同的实现类，但每一个实现类都必须重写接口中所有的抽象方法。即接口不考虑这些实现类各自采用什么方式实现这些功能，但它要求所有的实现类都必须有这些功能。
例如：
首先定义一个计算器的接口：<code>Calculator.java</code>,所有实现该接口的类，都必须具有计算两个数相加、相减、相乘、相除的功能。

``` java
public interface Calculator {
    /** 计算器可以计算两个数的和 */
    int add(int a, int b);
    /** 计算器可以计算两个数的差 */
    int sub(int a, int b);
    /** 计算器可以计算两个数的积 */
    long mul(int a, int b);
    /** 计算器可以计算两个数的商 */
    float div(int a, int b);
}
```
再定义一个实现该接口的类<code>ACalculator.java</code>

``` java
public class ACalculator implements Calculator {
    @Override
    public int add(int a, int b) {
        return a + b;
    }

    @Override
    public int sub(int a, int b) {
        return a - b;
    }

    @Override
    public long mul(int a, int b) {
        return a * b;
    }

    @Override
    public float div(int a, int b) {
        return (float) a / (float) b;
    }
}
```
在其他类中使用ACalculator进行两数之和的计算。其他类并不需要了解ACalculator是通过什么方式计算的，只需要了解Calculator接口中的相关方法定义即可。  

<code>Test.java</code>

``` java
public class Test {
    public static void main(String[] args) {
        Calculator calculator = new ACalculator();
        int sum = calculator.add(12, 14);
        System.out.println("sum = " + sum);
    }
}
```
## 使用3： 用于回调
我们知道，一般情况下主线程是不执行耗时任务的，如果遇到一些耗时任务（如网络请求，文件读写，数据库读写等等），我们会将其放入子线程中去执行，当执行完毕后，子线程再将执行结果返回给主线程。这个过程就是回调。

看一个例子。
首先定义一个回调接口。

<code>OnInfoFetchCallback.java</code>

``` java
public interface OnInfoFetchCallback {
    /** 获取信息成功 */
    void onSuccess(String info);
    /** 获取信息失败 */
    void failure();
}
```
再定义一个用于获取信息的任务类，在这个类中我们执行一些耗时操作。  

<code>InfoService.java</code>

``` java
public class InfoService {
    private OnInfoFetchCallback callback;
    public InfoService(OnInfoFetchCallback callback) {
        this.callback = callback;
    }

    public void getInfo() {
        // 模拟一个耗时操作
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000);
                    // 信息获取成功，传递结果
                    callback.onSuccess("结果是：" + new Date());
                } catch (InterruptedException e) {
                    // 信息获取失败
                    callback.failure();
                }
            }
        });
        thread.start();
    }
}
```
在<code>MainActivity</code>中调用<code>InfoService</code>中的<code>getInfo()</code>方法执行耗时操作。

``` java
public class MainActivity extends AppCompatActivity implements OnInfoFetchCallback {
    private static final String TAG = "MainActivity";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    /**
     * 获取信息
     * @param view
     */
    public void fetchInfo(View view) {
        InfoService service = new InfoService(this);
        service.getInfo();
    }

    @Override
    public void onSuccess(String info) {
        Log.i(TAG, info);
    }

    @Override
    public void failure() {
        Log.i(TAG, "获取信息失败");
    }
}
```
由于<code>MainActivity</code>已经实现了<code>OnInfoFetchCallback</code> 接口，所以在实例化<code>InfoService</code>对象时，直接将<code>this</code>传入即可。当任务执行结束后，调用<code>MainActivity</code>中的<code>onSuccess(String info)</code>或<code>failure()</code>方法将结果返回。


<code>MainActivity</code>也可以不用实现<code>OnInfoFetchCallback</code> 接口，此时可以采用匿名内部类的写法。  
如下所示：

``` java
    /**
     * 获取信息
     * @param view
     */
    public void fetchInfo(View view) {
        InfoService service = new InfoService(new OnInfoFetchCallback() {
            @Override
            public void onSuccess(String info) {
                Log.i(TAG, info);
            }

            @Override
            public void failure() {
                Log.i(TAG, "获取信息失败");
            }
        });
        service.getInfo();
    }
```

布局文件非常简单，只有一个按钮。

<code>activity_main.xml</code>

``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="cc.duduhuo.interfacedemo.MainActivity">

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="获取信息"
        android:onClick="fetchInfo"/>
</RelativeLayout>
```

Android中接口的三种基本使用方式已经介绍完了，该博文相关代码可通过下方地址下载：


[相关源码点击此处下载](http://download.csdn.net/detail/u012939909/9608555)

