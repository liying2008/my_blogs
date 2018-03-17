---
title: Simpler - 轻量级的微博客户端(开源)
date: 2017-06-18
tags: [Android, 微博]
categories: 我的开源
---
# 项目地址
**[https://github.com/liying2008/Simpler](https://github.com/liying2008/Simpler)** 
 欢迎大家Star/Fork。

> **此项目仅供Android开发学习交流使用，不得用于其他用途。**

[下载安装包 (v1.0.2)](https://github.com/liying2008/Simpler/releases/download/v1.0.2/simpler_1.0.2.apk)

# 编译环境
- Android Studio 2.3.3
- Gradle 3.3

## 说明
1. 应用中使用的图片资源大多来自[锤子科技论坛](http://www.smartisan.com/apps/bbs)，应用界面设计也较大程度参考了锤子科技论坛，特此声明。
1. 应用采用OAuth2认证授权，应用本身**不会**保存你的任何帐号和密码信息。
1. **此项目仅供Android开发学习交流使用，不得用于其他用途，也不允许将此项目打包后在应用商店中发布。**

# 主要功能
Simpler是一款轻量级的第三方微博应用，具有微博的基础功能，兼有外观优雅，运行流畅，内存占用低，省电省流量等特点。

# 屏幕截图
|截图|截图|
|:---:|:---:|
| ![截图1](http://upload-images.jianshu.io/upload_images/2883714-045ee52d63d95f5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) |![截图2](http://upload-images.jianshu.io/upload_images/2883714-06342d83f8883395.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)|
|![截图3](http://upload-images.jianshu.io/upload_images/2883714-9bc4e8f5eb05098d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)|![截图4](http://upload-images.jianshu.io/upload_images/2883714-0b51c2a480301493.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)|
|![截图5](http://upload-images.jianshu.io/upload_images/2883714-59feaa22aed5cac6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)|![截图6](http://upload-images.jianshu.io/upload_images/2883714-0de9a182a67c944c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)|
|![截图7](http://upload-images.jianshu.io/upload_images/2883714-e5cf7627acbfaae0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)|![截图8](http://upload-images.jianshu.io/upload_images/2883714-3ab602814a565fe6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)|
|![截图9](http://upload-images.jianshu.io/upload_images/2883714-85a931918be142b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)|![截图10](http://upload-images.jianshu.io/upload_images/2883714-25e22a4384eb694f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)|
|![截图11](http://upload-images.jianshu.io/upload_images/2883714-4ae7f172fbc355a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)|![截图12](http://upload-images.jianshu.io/upload_images/2883714-dc0bf2bd0a75f610.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)|
|![截图13](http://upload-images.jianshu.io/upload_images/2883714-6a5bc4d5a260de99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)|![截图14](http://upload-images.jianshu.io/upload_images/2883714-ea9cf6bc5da4597f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)|

 ![截图15](http://upload-images.jianshu.io/upload_images/2883714-ccc2e5d5f6f5d9e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 实现功能
1. 微博：浏览微博 / 发布微博 / 转发微博 / 复制微博内容 / 热门微博 / 搜索微博 / 周边动态
2. 评论：浏览评论 / 发表评论 / 转发评论 / 回复评论 / 复制评论
3. 用户：查看用户基本信息 / 粉丝  / 关注  / 用户微博 / 相册
4. 关系：关注用户 / 取消关注 / 移出分组 / 更改分组
5. 消息：消息通知 / 私信通知 / 发出的评论  / 收到的评论 / @我的微博 / @我的评论
6. 话题：热门话题 / 查看话题 / 搜索话题微博
7. 收藏：添加收藏 / 取消收藏 / 查看收藏
8. 帐号：添加帐号 / 退出登录 / 多用户切换 / 更改头像
9. 草稿箱：将未发布的微博加入草稿箱 / 查看草稿箱 / 删除草稿
10. 分组：查看分组微博 / 添加分组 / 删除分组 / 编辑分组
11. 微博详情：查看图片 / 下载图片 / 分享图片 / 观看秒拍视频 / 下载秒拍视频
12. 赞：点赞 / 取消赞 / 原微博点赞和取消赞
13. 网页版微博
14. 设置：微博字体大小 / 每次刷新微博数 / 微博图片显示质量 / 图片上传质量
15. 消息通知：关闭 / 开启 / 通知时间间隔
16. 其他：清理图片缓存 / 检查更新 / 用户反馈

## 其他特性
> 
1. 支持绑定多个用户  
1. 支持分页加载和下拉刷新  
1. 支持发送带有多张图片的微博  
1. 支持微博缓存（节省流量）  
1. 微博列表快速滑动时不加载图片（节省流量）  
1. 视频播放界面，允许锁定横屏或竖屏（非常适用于躺在床上看视频）  
1. 支持将关注的好友添加进多个分组  
1. 支持为好友添加备注  
1. 如果发布的微博是分组微博、私密微博或朋友圈微博，微博列表会标注该条微博的可见分组。  

# 已知BUG
1. 加载GIF图时无法获取加载进度。  

# Thanks
- [weibo_android_sdk](https://github.com/sinaweibosdk/weibo_android_sdk)
- [PhotoView](https://github.com/chrisbanes/PhotoView)
- [glide](https://github.com/bumptech/glide)
- [okhttp](https://github.com/square/okhttp)
- [okio](https://github.com/square/okio)
- [butterknife](https://github.com/JakeWharton/butterknife)
- [CircleImageView](https://github.com/hdodenhof/CircleImageView)
- [jsoup](https://github.com/jhy/jsoup)
- [LoadingProgress](https://github.com/peng8350/LoadingProgress)
- [guava](https://github.com/google/guava)
- [FileDownloader](https://github.com/lingochamp/FileDownloader)