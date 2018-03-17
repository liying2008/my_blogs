---
title: 问题解决：Fragment not attached to Activity
date: 2016-11-26
tags: [Android, Fragment]
categories: 问题解决
---
## 1、问题引入
在Fragment中执行一段耗时任务，在任务未结束的时候，重建Activity就会导致<code>getActivity()</code>为<code>null</code>，所有用到<code>getActivity()</code>的地方都会引起空指针异常，如果使用了<code>getResources()</code>方法，就会导致<code>Fragment not attached to Activity</code>。

为了重现这一异常，我们编写如下代码：

- FirstFragment.java

```java
public class FirstFragment extends Fragment implements View.OnClickListener {
    private TextView tvMsg;
    private Button btnStartTask, btnRecreate;
    private static final String TAG = "FirstFragment";

    public FirstFragment() {
        // Required empty public constructor
    }


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_first, container, false);
        tvMsg = (TextView) view.findViewById(R.id.tvMsg);
        btnStartTask = (Button) view.findViewById(R.id.btnStartTask);
        btnRecreate = (Button) view.findViewById(R.id.btnRecreate);
        btnStartTask.setOnClickListener(this);
        btnRecreate.setOnClickListener(this);
        return view;
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.btnStartTask:
                // 模拟一个耗时任务
                new AsyncTask<Void, Void, Void>() {

                    @Override
                    protected Void doInBackground(Void... params) {
                        try {
                            Thread.sleep(2000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        return null;
                    }

                    @Override
                    protected void onPostExecute(Void aVoid) {
                        super.onPostExecute(aVoid);
                        Log.d(TAG, "getActivity = " + getActivity());
                        tvMsg.setText(getResources().getString(R.string.app_name));
                    }
                }.execute();
                break;
            case R.id.btnRecreate:
                // 重新创建MainActivity
                getActivity().recreate();
                break;
        }
    }
}
```

- SecondFragment.java

```java
public class SecondFragment extends Fragment {
    
    public SecondFragment() {
        // Required empty public constructor
    }
    
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_second, container, false);
    }
}

```

- fragment_first.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="cc.duduhuo.fragmentattachdemo.fragment.FirstFragment">

    <TextView
        android:id="@+id/tvMsg"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="The First Fragment" />

    <Button
        android:id="@+id/btnStartTask"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="耗时任务" />

    <Button
        android:id="@+id/btnRecreate"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="重建Activity" />
</LinearLayout>
```

- fragment_second.xml

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="cc.duduhuo.fragmentattachdemo.fragment.SecondFragment">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="The Second Fragment" />

</FrameLayout>
```

- MainActivity.java

```java
public class MainActivity extends FragmentActivity {
    private ViewPager mViewPager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mViewPager = (ViewPager) this.findViewById(R.id.pager);
        initial();
    }

    private void initial() {
        List<Fragment> fragmentList = new ArrayList<>();
        List<String> titleList = new ArrayList<>();
        fragmentList.add(new FirstFragment());
        fragmentList.add(new SecondFragment());
        titleList.add("First");
        titleList.add("Second");

        MyFragmentPageAdapter adapter = new MyFragmentPageAdapter(getSupportFragmentManager(), fragmentList, titleList);
        mViewPager.setAdapter(adapter);
    }

    private class MyFragmentPageAdapter extends FragmentPagerAdapter {
        private List<Fragment> fragmentList;
        private List<String> titleList;

        public MyFragmentPageAdapter(FragmentManager fm, List<Fragment> fragmentList, List<String> titleList) {
            super(fm);
            this.fragmentList = fragmentList;
            this.titleList = titleList;
        }

        @Override
        public Fragment getItem(int position) {
            return (fragmentList == null || fragmentList.size() == 0) ? null : fragmentList.get(position);
        }

        @Override
        public CharSequence getPageTitle(int position) {
            return (titleList.size() > position) ? titleList.get(position) : "";
        }

        @Override
        public int getCount() {
            return fragmentList == null ? 0 : fragmentList.size();
        }
    }
}
```

- activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="cc.duduhuo.fragmentattachdemo.MainActivity">

    <android.support.v4.view.ViewPager
        android:id="@+id/pager"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.v4.view.PagerTabStrip
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="top" />
    </android.support.v4.view.ViewPager>
</LinearLayout>
```
当点击<code>FirstFragment</code>里面的“耗时任务”按钮时，会执行一个2000ms的任务（上面的代码是用休眠2000ms代替一个耗时任务）。如果点过之后静静等待2000ms，上面的<code>TextView</code>的文本就会变成FragmentAttachDemo，并不会报出任何异常。但是当我们点击“耗时任务”按钮之后，在它还未执行完毕时，点击下面的“重建ACTIVITY”按钮，很快程序就会崩溃。

控制台打印出来的信息如下图所示：
![错误信息](http://img.blog.csdn.net/20161126221331962)

除了点击“重建ACTIVITY”按钮之外，点击“耗时任务”按钮之后立即旋转手机屏幕也会导致此异常，因为默认情况下屏幕旋转也会重建Activity。

## 2、问题解决
将<code>FirstFragment</code>中<code>onPostExecute()</code>方法中的
```java
tvMsg.setText(getResources().getString(R.string.app_name));
```
改为
```java
if (isAdded()) {
    tvMsg.setText(getResources().getString(R.string.app_name));
}
```

<code>isAdded()</code>方法可以判断当前的<code>Fragment</code>是否已经添加到<code>Activity</code>中，只有当<code>Fragment</code>已经添加到<code>Activity</code>中时才执行<code>getResources()</code>等方法。

另请参考：[http://stackoverflow.com/questions/10919240/fragment-myfragment-not-attached-to-activity](http://stackoverflow.com/questions/10919240/fragment-myfragment-not-attached-to-activity)

当然，以上只是引起该异常的一个例子，并不能解决所有“Fragment not attached to Activity”的问题。
## 3、代码下载
[Demo代码下载](http://download.csdn.net/detail/u012939909/9694610)