---
title: OpenCV+JavaCV实现人脸识别
date: 2016-09-09 16:29:53
categories: 开源
tags: [OpenCV]
---

---
转载请说明出处！
作者：[kqw攻城狮](http://kongqw.github.io/about/index.html)
出处：[个人站](http://kongqw.com/2016/09/09/2016-09-09-OpenCV-JavaCV%E5%AE%9E%E7%8E%B0%E4%BA%BA%E8%84%B8%E8%AF%86%E5%88%AB/) | [CSDN](http://blog.csdn.net/q4878802/article/details/52488447)

---

OpenCV主要实现人脸检测功能

JavaCV主要实现人脸对比功能

具体的就不啰嗦了，本来最近很忙，主要是因为好多人私信我要 [Android使用OpenCV实现「人脸检测」和「人脸识别」](http://kongqw.com/2016/07/06/2016-07-06-Android%E4%BD%BF%E7%94%A8OpenCV%E5%AE%9E%E7%8E%B0%E4%BA%BA%E8%84%B8%E8%AF%86%E5%88%AB/) 的Demo，今天特意抽出时间写了一下。

# 效果图

![效果图](http://img.blog.csdn.net/20160909162919759)

![效果图](http://img.blog.csdn.net/20160909162931572)

# 源码

[KqwFaceDetectionDemo](https://github.com/kongqw/KqwFaceDetectionDemo)

感觉有用的话，就给个`star`吧，谢谢！！


# 注意

最后啰嗦一点，如果你的程序是跑在手机、pad等设备上，一般没有什么问题。
但是如果你是在自己的开发板上跑，可能会有一些小插曲。

比如我司的机器人是定制的Android板子，对系统做了裁剪，很多摄像头的方法可能就用不了

例如这样一个错误

``` java
AndroidRuntime: java.lang.RuntimeException: setParameters failed
```

当打开程序的时候，OpenCV会提示，没有找到可用摄像头或者摄像头被锁住（大概这个意思，我就不截图了），一种可能是设备真的没有接摄像头，也有可能是摄像头定制过，导致某些方法用不了，比如上面的错误就是我遇到的其中一个。