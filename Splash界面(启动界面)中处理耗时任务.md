---
title: Splash界面(启动界面)中处理耗时任务
date: 2016-11-26
tags: [Android, Splash, AsyncTask]
categories: Android基础
---
## 1、SplashActivity
将SplashActivity设为Launcher Activity，只需要在<code>AndroidManifest.xml</code>文件中配置SplashActivity的<code>intent-filter</code>如下所示：
```xml
<!--启动界面-->
<activity
    android:name=".SplashActivity"
    android:theme="@style/FullScreen">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />

        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```
SplashActivity一般都是全屏显示的，所以需要为其指定一个全屏显示的theme，可以使用如下的style。
<code>FullScreen</code>
```xml
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light">
    <!-- Customize your theme here. -->
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorAccent</item>
</style>

<!--全屏-->
<style name="FullScreen" parent="AppTheme">
    <item name="windowNoTitle">true</item>
    <item name="android:windowFullscreen">true</item>
</style>
```
一般情况下，SplashActivity起到展示产品形象、显示广告、初始化数据的作用。我们需要为其设定一个展示的时间，比如800ms，800ms之后应用将自动跳转到应用主界面并销毁SplashActivity。
所以我们可以在SplashActivity的onCreate方法中使用handler的<code>postDelayed()</code>方法来实现在一定时间间隔后执行某个动作的功能。
如下面的代码所示。
```java
public class SplashActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_splash);
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                jump();// 界面跳转
            }
        }, 800);   // 停留时间800ms
    }

    /**
     * 跳转界面
     */
    private void jump() {
        // 此处可以根据版本号进行一些判断，再跳转到相应的界面
        startActivity(new Intent(this, MainActivity.class));
        finish();
    }

    @Override
    public void onBackPressed() {
        // Splash界面不允许使用back键
    }
}
```
这样就会在800ms之后执行<code>jump()</code>方法，实现Activity跳转并销毁当前的SplashActivity的效果。

## 2、SplashLoadDataTask
如果我们需要在应用启动时初始化一些数据，这些数据可以来自文件、数据库或者网络。从文件或数据库中读取大量数据或者从网络中加载数据这些任务都属于耗时任务，显然就不能放在主线程中了。因此我们新建一个继承自<code>AsyncTask</code>的任务类<code>SplashLoadDataTask</code>用于处理这些耗时任务。
```java
public class SplashLoadDataTask extends AsyncTask<Void, Void, Integer> {

    private LoadDataCallback callback;

    public SplashLoadDataTask(LoadDataCallback callback) {
        this.callback = callback;
    }

    @Override
    protected Integer doInBackground(Void... params) {
        int status = 0;
        // 在此执行耗时任务，可根据任务（数据加载）执行状态给status赋不同的值。
        // ...

        return status;
    }

    @Override
    protected void onPostExecute(Integer status) {
        super.onPostExecute(status);
        if (status == 0) {
            callback.loaded();
        } else if (status == 1) {
            callback.loadError();
        }
    }

    /**
     * 加载数据回调
     */
    public interface LoadDataCallback {
        /**
         * 数据加载完毕
         */
        void loaded();

        /**
         * 数据加载出错
         */
        void loadError();
    }

}

```
在SplashActivity中启动这个任务，并在回调方法中进行界面跳转。
```java
public class SplashActivity extends AppCompatActivity implements SplashLoadDataTask.LoadDataCallback {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_splash);
        // 启动加载应用数据任务类
        SplashLoadDataTask task = new SplashLoadDataTask(this);
        task.execute();
    }

    /**
     * 跳转界面
     */
    private void jump() {
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                // 此处可以根据版本号进行一些判断，再跳转到相应的界面
                startActivity(new Intent(SplashActivity.this, MainActivity.class));
                finish();
            }
        }, 500);   // 停留时间500ms
    }

    /**
     * 数据加载完成
     */
    @Override
    public void loaded() {
        jump();
    }

    /**
     * 数据加载出错
     */
    @Override
    public void loadError() {
        // 进行出错处理
        // ...

        jump();
    }

    @Override
    public void onBackPressed() {
        // Splash界面不允许使用back键
    }
}
```
此时要注意的是，所谓“停留时间500ms”并不是指Splash界面显示时长是500ms，而是指耗时任务完成后，再过500ms进行界面跳转，所以Splash界面的显示时间应该大于500ms，在实际的开发中应该先估算一下耗时任务执行时长，再确定<code>postDelayed()</code>的延时时长。

## 3、效果演示
![效果演示](http://img.blog.csdn.net/20161126144117814)

## 4、Demo代码下载
[Demo代码下载](http://download.csdn.net/detail/u012939909/9694299)