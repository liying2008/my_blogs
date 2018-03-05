---
title: Android获取本机IPv4地址
date: 2016-12-19
tags: [Android, IP, IPv4]
categories: Android基础
---

> 获取本机IPv4地址可分两种情况，一种是**WiFi已开启**，一种是**蜂窝移动数据已开启**。

# 1、WiFi已开启

WiFi开启的情况下，通过WiFi获取本机IP地址，如果仅仅打开WiFi，但并未接入网络，则IP地址可认为是<code>0.0.0.0</code>。
如果已经接入网络，则可以通过如下方式获取IPv4地址。

```java
// 获取WiFi服务
WifiManager wifiManager = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);
// 判断WiFi是否开启
WifiInfo wifiInfo = wifiManager.getConnectionInfo();
int ipAddress = wifiInfo.getIpAddress();
String ip = (ipAddress & 0xFF) + "." +
                ((ipAddress >> 8) & 0xFF) + "." +
                ((ipAddress >> 16) & 0xFF) + "." +
                (ipAddress >> 24 & 0xFF);
```

# 2、蜂窝移动数据已开启

蜂窝移动数据开启的情况下可以通过如下方式获取IPv4地址。

```java
try {
	NetworkInterface networkInterface;
	InetAddress inetAddress;
	for (Enumeration<NetworkInterface> en = NetworkInterface.getNetworkInterfaces(); en.hasMoreElements(); ) {
		networkInterface = en.nextElement();
		for (Enumeration<InetAddress> enumIpAddr = networkInterface.getInetAddresses(); enumIpAddr.hasMoreElements(); ) {
			inetAddress = enumIpAddr.nextElement();
			if (!inetAddress.isLoopbackAddress() && !inetAddress.isLinkLocalAddress()) {
				String ip = inetAddress.getHostAddress();
			}
		}
	}
} catch (SocketException ex) {
	ex.printStackTrace();
}
```

# 3、封装为工具类

可以将以上两种方式封装为一个工具类<code>IPUtils</code>，由工具类自动判断网络类型并调用相应的方法，最终返回一个IPv4地址。
完整代码如下所示。

 - IPUtils.java

```java
import android.content.Context;
import android.net.wifi.WifiInfo;
import android.net.wifi.WifiManager;

import java.net.InetAddress;
import java.net.NetworkInterface;
import java.net.SocketException;
import java.util.Enumeration;

/**
 * =======================================================
 * 版权：Copyright LiYing 2015-2016. All rights reserved.
 * 作者：liying - liruoer2008@yeah.net
 * 日期：2016/12/19 19:43
 * 版本：1.0
 * 描述：IP地址工具类
 * 备注：
 * =======================================================
 */
public class IPUtils {

    /**
     * 获取本机IPv4地址
     *
     * @param context
     * @return 本机IPv4地址；null：无网络连接
     */
    public static String getIpAddress(Context context) {
        // 获取WiFi服务
        WifiManager wifiManager = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);
        // 判断WiFi是否开启
        if (wifiManager.isWifiEnabled()) {
            // 已经开启了WiFi
            WifiInfo wifiInfo = wifiManager.getConnectionInfo();
            int ipAddress = wifiInfo.getIpAddress();
            String ip = intToIp(ipAddress);
            return ip;
        } else {
            // 未开启WiFi
            return getIpAddress();
        }
    }

    private static String intToIp(int ipAddress) {
        return (ipAddress & 0xFF) + "." +
                ((ipAddress >> 8) & 0xFF) + "." +
                ((ipAddress >> 16) & 0xFF) + "." +
                (ipAddress >> 24 & 0xFF);
    }

    /**
     * 获取本机IPv4地址
     *
     * @return 本机IPv4地址；null：无网络连接
     */
    private static String getIpAddress() {
        try {
            NetworkInterface networkInterface;
            InetAddress inetAddress;
            for (Enumeration<NetworkInterface> en = NetworkInterface.getNetworkInterfaces(); en.hasMoreElements(); ) {
                networkInterface = en.nextElement();
                for (Enumeration<InetAddress> enumIpAddr = networkInterface.getInetAddresses(); enumIpAddr.hasMoreElements(); ) {
                    inetAddress = enumIpAddr.nextElement();
                    if (!inetAddress.isLoopbackAddress() && !inetAddress.isLinkLocalAddress()) {
                        return inetAddress.getHostAddress();
                    }
                }
            }
            return null;
        } catch (SocketException ex) {
            ex.printStackTrace();
            return null;
        }
    }
}
```

之后，只需要在需要得到IPv4地址的位置调用<code>IPUtils.getIpAddress(Context context)</code>即可。

最后，别忘了在<code>AndroidManifest.xml</code>文件中声明需要的权限，如下。

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
```
