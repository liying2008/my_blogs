---
title: Android时间轴(Timeline)效果的实现
date: 2016-11-26
tags: [Android, View, Timeline]
categories: View
---

## 1、时间轴效果
我们要实现的时间轴效果如下图所示，这是锤子手机的查看物流信息界面的截图。
![物流信息的时间线](http://img.blog.csdn.net/20161126155452052)
## 2、主要布局的实现
在<code>TraceActivity</code>关联的布局文件<code>activity_trace.xml</code>中放置一个<code>ListView</code>，代码如下。由于这个列表只是用于展示信息，并不需要用户去点击，所以将其<code>clickable</code>属性置为<code>false</code>；为了消除<code>ListView</code>点击产生的波纹效果，我们设置其<code>listSelector</code>属性的值为透明；我们不需要列表项之间的分割线，所以设置其<code>divider</code>属性的值为<code>null</code>。

 - activity_trace.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@android:color/white"
    tools:context="cc.duduhuo.timelinedemo.TraceActivity">

    <ListView
        android:id="@+id/lvTrace"
        android:layout_width="match_parent"
        android:divider="@null"
        android:clickable="false"
        android:listSelector="@android:color/transparent"
        android:dividerHeight="0dp"
        android:layout_height="wrap_content" />
</LinearLayout>
```
之后再设计每个列表项的布局<code>item_trace.xml</code>，代码如下。由于时间轴的点和线都位于item布局中，为了使线是连续的，所以设置上面<code>ListView</code>的<code>dividerHeight</code>属性值为<code>0dp</code>，即垂直方向每个列表项都是紧挨着的。在item的布局中，我们先使用<code>LinearLayout</code>将布局分成左右两个部分，左边就是时间轴的布局，右边是物流信息的布局。

先说物流信息的布局，物流信息是一个<code>RelativeLayout</code>，其内放置两个<code>TextView</code>分别用于承载物流信息的时间的描述的显示，为了不使两个列表项的文本靠得太近，我们在<code>RelativeLayout</code>中设置其<code>paddingBottom</code>和<code>paddingTop</code>属性。

再说时间轴的布局，时间轴的布局也是一个<code>RelativeLayout</code>，但是不设置其<code>padding</code>属性，宽度为32dp。为了使时间轴的圆点和显示时间的文本对齐，我们需要在圆点之上再放置一条竖线，所以整体的布局就是 <code>线 - 点 - 线</code>。为了让线可以正好对准圆点的中心，我们让线和点都水平居中，即<code>android:layout_centerHorizontal="true"</code>。

 - item_trace.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="horizontal">

    <RelativeLayout
        android:id="@+id/rlTimeline"
        android:layout_width="32dp"
        android:layout_height="match_parent">

        <TextView
            android:id="@+id/tvTopLine"
            android:layout_width="0.5dp"
            android:layout_height="12dp"
            android:layout_centerHorizontal="true"
            android:background="#999" />

        <TextView
            android:id="@+id/tvDot"
            android:layout_width="5dp"
            android:layout_height="5dp"
            android:layout_below="@id/tvTopLine"
            android:layout_centerHorizontal="true"
            android:background="@drawable/timelline_dot_normal" />

        <TextView
            android:layout_width="0.5dp"
            android:layout_height="match_parent"
            android:layout_below="@id/tvDot"
            android:layout_centerHorizontal="true"
            android:background="#999" />
    </RelativeLayout>

    <RelativeLayout
        android:id="@+id/rlCenter"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingBottom="6dp"
        android:paddingRight="10dp"
        android:paddingTop="6dp">

        <TextView
            android:id="@+id/tvAcceptTime"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="2014/06/24 20:55:28"
            android:textColor="#999"
            android:textSize="12sp" />

        <TextView
            android:id="@+id/tvAcceptStation"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_below="@id/tvAcceptTime"
            android:layout_marginTop="5dp"
            android:text="快件在 深圳 ,准备送往下一站 深圳集散中心 [深圳市]"
            android:textColor="#999"
            android:textSize="12sp" />
    </RelativeLayout>
</LinearLayout>
```
圆点的背景资源如下代码：

 - timelline_dot_first.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">

    <size
        android:width="10dp"
        android:height="10dp" />
    <solid android:color="#555555" />
</shape>
```
 - timelline_dot_normal.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">

    <size
        android:width="10dp"
        android:height="10dp" />
    <solid android:color="#999999" />
</shape>
```

## 3、Activity

为了可以看到布局的效果，我们在<code>TraceActivity</code>中模拟一些假数据。需要定义一个实体类<code>Trace</code>，它有两个属性，<code>acceptTime</code>和<code>acceptStation</code>，代码如下：

- Trace.java

```java
public class Trace {
    /** 时间 */
    private String acceptTime;
    /** 描述 */
    private String acceptStation;

    public Trace() {
    }

    public Trace(String acceptTime, String acceptStation) {
        this.acceptTime = acceptTime;
        this.acceptStation = acceptStation;
    }

    public String getAcceptTime() {
        return acceptTime;
    }

    public void setAcceptTime(String acceptTime) {
        this.acceptTime = acceptTime;
    }

    public String getAcceptStation() {
        return acceptStation;
    }

    public void setAcceptStation(String acceptStation) {
        this.acceptStation = acceptStation;
    }
}
```

 - TraceActivity.java
```java
public class TraceActivity extends AppCompatActivity {

    private ListView lvTrace;
    private List<Trace> traceList = new ArrayList<>(10);
    private TraceListAdapter adapter;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_trace);
        findView();
        initData();
    }

    private void findView() {
        lvTrace = (ListView) findViewById(R.id.lvTrace);
    }

    private void initData() {
        // 模拟一些假的数据
        traceList.add(new Trace("2016-05-25 17:48:00", "[沈阳市] [沈阳和平五部]的派件已签收 感谢使用中通快递,期待再次为您服务!"));
        traceList.add(new Trace("2016-05-25 14:13:00", "[沈阳市] [沈阳和平五部]的东北大学代理点正在派件 电话:18040xxxxxx 请保持电话畅通、耐心等待"));
        traceList.add(new Trace("2016-05-25 13:01:04", "[沈阳市] 快件到达 [沈阳和平五部]"));
        traceList.add(new Trace("2016-05-25 12:19:47", "[沈阳市] 快件离开 [沈阳中转]已发往[沈阳和平五部]"));
        traceList.add(new Trace("2016-05-25 11:12:44", "[沈阳市] 快件到达 [沈阳中转]"));
        traceList.add(new Trace("2016-05-24 03:12:12", "[嘉兴市] 快件离开 [杭州中转部]已发往[沈阳中转]"));
        traceList.add(new Trace("2016-05-23 21:06:46", "[杭州市] 快件到达 [杭州汽运部]"));
        traceList.add(new Trace("2016-05-23 18:59:41", "[杭州市] 快件离开 [杭州乔司区]已发往[沈阳]"));
        traceList.add(new Trace("2016-05-23 18:35:32", "[杭州市] [杭州乔司区]的市场部已收件 电话:18358xxxxxx"));
        adapter = new TraceListAdapter(this, traceList);
        lvTrace.setAdapter(adapter);
    }
}
```

## 4、Adapter
需要为<code>ListView</code>设置一个Adapter。定义一个<code>TraceListAdapter</code>，代码如下。由于第一行的物流信息的显示形式和其他的不一样，所以把item分成两个类型，<code>TYPE_TOP </code>指第一行的item，<code>TYPE_NORMAL</code>指其他的item。区别在于，第一行的item的时间轴布局中最上面的线不显示，而且文字颜色和圆点的颜色较深。其余item文字颜色和圆点颜色稍浅一些，而且<code>线 - 点 - 线</code>也全都显示。

- TraceListAdapter.java
```java
public class TraceListAdapter extends BaseAdapter {
    private Context context;
    private List<Trace> traceList = new ArrayList<>(1);
    private static final int TYPE_TOP = 0x0000;
    private static final int TYPE_NORMAL= 0x0001;

    public TraceListAdapter(Context context, List<Trace> traceList) {
        this.context = context;
        this.traceList = traceList;
    }

    @Override
    public int getCount() {
        return traceList.size();
    }

    @Override
    public Trace getItem(int position) {
        return traceList.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder;
        final Trace trace = getItem(position);
        if (convertView != null) {
            holder = (ViewHolder) convertView.getTag();
        } else {
            holder = new ViewHolder();
            convertView = LayoutInflater.from(context).inflate(R.layout.item_trace, parent, false);
            holder.tvAcceptTime = (TextView) convertView.findViewById(R.id.tvAcceptTime);
            holder.tvAcceptStation = (TextView) convertView.findViewById(R.id.tvAcceptStation);
            holder.tvTopLine = (TextView) convertView.findViewById(R.id.tvTopLine);
            holder.tvDot = (TextView) convertView.findViewById(R.id.tvDot);
            convertView.setTag(holder);
        }

        if (getItemViewType(position) == TYPE_TOP) {
            // 第一行头的竖线不显示
            holder.tvTopLine.setVisibility(View.INVISIBLE);
            // 字体颜色加深
            holder.tvAcceptTime.setTextColor(0xff555555);
            holder.tvAcceptStation.setTextColor(0xff555555);
            holder.tvDot.setBackgroundResource(R.drawable.timelline_dot_first);
        } else if (getItemViewType(position) == TYPE_NORMAL) {
            holder.tvTopLine.setVisibility(View.VISIBLE);
            holder.tvAcceptTime.setTextColor(0xff999999);
            holder.tvAcceptStation.setTextColor(0xff999999);
            holder.tvDot.setBackgroundResource(R.drawable.timelline_dot_normal);
        }

        holder.tvAcceptTime.setText(trace.getAcceptTime());
        holder.tvAcceptStation.setText(trace.getAcceptStation());
        return convertView;
    }

    @Override
    public int getItemViewType(int id) {
        if (id == 0) {
            return TYPE_TOP;
        }
        return TYPE_NORMAL;
    }

    static class ViewHolder {
        public TextView tvAcceptTime, tvAcceptStation;
        public TextView tvTopLine, tvDot;
    }
}
```

## 5、效果展示
![Trace_Demo](http://img.blog.csdn.net/20161126165230733)

## 6、代码下载
[Demo代码下载](http://download.csdn.net/detail/u012939909/9694387)

