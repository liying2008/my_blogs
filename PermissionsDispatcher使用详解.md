---
title: PermissionsDispatcher使用详解
date: 2016-12-27
tags: [Android, Permissions]
categories: 开源库
---
PermissionsDispatcher是一个基于注解、帮助开发者简单处理Android 6.0系统中的运行时权限的开源库。避免了开发者编写大量繁琐的样板代码。

**开源地址：[https://github.com/hotchemi/PermissionsDispatcher](https://github.com/hotchemi/PermissionsDispatcher)**

**文档介绍：[http://hotchemi.github.io/PermissionsDispatcher/](http://hotchemi.github.io/PermissionsDispatcher/)**

下面详细介绍一下如何在Android Studio上使用该开源库。

# 1、添加依赖

首先在项目工程下的<code>build.gradle</code>文件中加入对maven仓库依赖引入的支持。

```gradle
allprojects {
    repositories {
        jcenter()
        mavenCentral()
    }
}
```

之后在module下的<code>build.gradle</code>文件中添加两项依赖：

```gradle
compile 'com.github.hotchemi:permissionsdispatcher:2.3.1'
annotationProcessor 'com.github.hotchemi:permissionsdispatcher-processor:2.3.1'
```

并将<code>targetSdkVersion</code>设为23，即：<code>targetSdkVersion 23</code>

# 2、在Activity或Fragment中使用

注解列表：

|Annotation|Required|Description|
|---|---|---|
|`@RuntimePermissions`|**✓**|注解在其内部需要使用运行时权限的Activity或Fragment上|
|`@NeedsPermission`|**✓**|注解在需要调用运行时权限的方法上，当用户给予权限时会执行该方法|
|`@OnShowRationale`||注解在用于向用户解释为什么需要调用该权限的方法上，只有当第一次请求权限被用户拒绝，下次请求权限之前会调用|
|`@OnPermissionDenied`||注解在当用户拒绝了权限请求时需要调用的方法上|
|`@OnNeverAskAgain`||注解在当用户选中了授权窗口中的***不再询问***复选框后并拒绝了权限请求时需要调用的方法，一般可以向用户解释为何申请此权限，并根据实际需求决定是否再次弹出权限请求对话框|  


> **注意：被注解的方法不能是私有方法。**

只有<code>@RuntimePermissions</code>和<code>@NeedsPermission</code>是必须的，其余注解均为可选。当使用了<code>@RuntimePermissions</code>和<code>@NeedsPermission</code>之后，需要点击菜单栏中<code>Build</code>菜单下的<code>Make Project</code>，或者按快捷键<code>Ctrl + F9</code>编译整个项目，编译器会在<code>app\build\intermediates\classes\debug</code>目录下与被注解的Activity同一个包下生成一个辅助类，名称为<code>被注解的Activity名称+PermissionsDispatcher.class</code>，如图所示：

![生成的辅助类](http://img.blog.csdn.net/20161227201859240)

接下来可以调用辅助类里面的方法完成应用的权限请求了。

在需要调用权限的位置调用辅助类里面的<code>xxxWithCheck</code>方法，<code>xxx</code>是被<code>@NeedsPermission</code>注解的方法名。如：

```java
MainActivityPermissionsDispatcher.showCameraWithCheck(this);
```

之后，还需要重写该Activity的<code>onRequestPermissionsResult()</code>方法，其方法内调用辅助类的<code>onRequestPermissionsResult()</code>方法，如下：

```java
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    // NOTE: delegate the permission handling to generated method
    MainActivityPermissionsDispatcher.onRequestPermissionsResult(this, requestCode, grantResults);
}
```

完整的<code>MainActivity</code>如下所示。

- MainActivity.java

```java
@RuntimePermissions
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        findViewById(R.id.button_camera).setOnClickListener(this);
        findViewById(R.id.button_contacts).setOnClickListener(this);
    }

    @Override
    public void onClick(@NonNull View v) {
        switch (v.getId()) {
            case R.id.button_camera:
                // NOTE: delegate the permission handling to generated method
                MainActivityPermissionsDispatcher.showCameraWithCheck(this);
                break;
            case R.id.button_contacts:
                // NOTE: delegate the permission handling to generated method
                MainActivityPermissionsDispatcher.showContactsWithCheck(this);
                break;
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        // NOTE: delegate the permission handling to generated method
        MainActivityPermissionsDispatcher.onRequestPermissionsResult(this, requestCode, grantResults);
    }

    @NeedsPermission(Manifest.permission.CAMERA)
    void showCamera() {
        // NOTE: Perform action that requires the permission. If this is run by PermissionsDispatcher, the permission will have been granted
        getSupportFragmentManager().beginTransaction()
                .replace(R.id.sample_content_fragment, CameraPreviewFragment.newInstance())
                .addToBackStack("camera")
                .commitAllowingStateLoss();
    }

    @NeedsPermission({Manifest.permission.READ_CONTACTS, Manifest.permission.WRITE_CONTACTS})
    void showContacts() {
        // NOTE: Perform action that requires the permission.
        // If this is run by PermissionsDispatcher, the permission will have been granted
        getSupportFragmentManager().beginTransaction()
                .replace(R.id.sample_content_fragment, ContactsFragment.newInstance())
                .addToBackStack("contacts")
                .commitAllowingStateLoss();
    }

    @OnShowRationale(Manifest.permission.CAMERA)
    void showRationaleForCamera(PermissionRequest request) {
        // NOTE: Show a rationale to explain why the permission is needed, e.g. with a dialog.
        // Call proceed() or cancel() on the provided PermissionRequest to continue or abort
        showRationaleDialog(R.string.permission_camera_rationale, request);
    }

    @OnShowRationale({Manifest.permission.READ_CONTACTS, Manifest.permission.WRITE_CONTACTS})
    void showRationaleForContact(PermissionRequest request) {
        // NOTE: Show a rationale to explain why the permission is needed, e.g. with a dialog.
        // Call proceed() or cancel() on the provided PermissionRequest to continue or abort
        showRationaleDialog(R.string.permission_contacts_rationale, request);
    }

    @OnPermissionDenied(Manifest.permission.CAMERA)
    void onCameraDenied() {
        // NOTE: Deal with a denied permission, e.g. by showing specific UI
        // or disabling certain functionality
        Toast.makeText(this, R.string.permission_camera_denied, Toast.LENGTH_SHORT).show();
    }

    @OnNeverAskAgain(Manifest.permission.CAMERA)
    void onCameraNeverAskAgain() {
        Toast.makeText(this, R.string.permission_camera_never_askagain, Toast.LENGTH_SHORT).show();
    }

    public void onBackClick(View view) {
        getSupportFragmentManager().popBackStack();
    }

    private void showRationaleDialog(@StringRes int messageResId, final PermissionRequest request) {
        new AlertDialog.Builder(this)
                .setPositiveButton("允许", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(@NonNull DialogInterface dialog, int which) {
                        request.proceed();
                    }
                })
                .setNegativeButton("拒绝", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(@NonNull DialogInterface dialog, int which) {
                        request.cancel();
                    }
                })
                .setCancelable(false)
                .setMessage(messageResId)
                .show();
    }
}
```

# 3、附：危险权限和权限组列表

Android官方文档：[https://developer.android.com/guide/topics/security/permissions.html#normal-dangerous](https://developer.android.com/guide/topics/security/permissions.html#normal-dangerous)

摘录如下：

|权限组|权限|
|---|---|
|**CALENDAR**|READ_CALENDAR<br>WRITE_CALENDAR|
|**CAMERA**|CAMERA|
|**CONTACTS**|READ_CONTACTS<br>WRITE_CONTACTS<br>GET_ACCOUNTS|
|**LOCATION**|ACCESS_FINE_LOCATION<br>ACCESS_COARSE_LOCATION|
|**PHONE**|READ_PHONE_STATE<br>CALL_PHONE<br>READ_CALL_LOG<br>WRITE_CALL_LOG<br>ADD_VOICEMAIL<br>USE_SIP<br>PROCESS_OUTGOING_CALLS|
|**SENSORS**|BODY_SENSORS|
|**SMS**|SEND_SMS<br>RECEIVE_SMS<br>READ_SMS<br>RECEIVE_WAP_PUSH<br>RECEIVE_MMS|
|**STORAGE**|READ_EXTERNAL_STORAGE<br>WRITE_EXTERNAL_STORAGE|

# 4、附：Demo下载

[http://download.csdn.net/detail/u012939909/9722890](http://download.csdn.net/detail/u012939909/9722890)