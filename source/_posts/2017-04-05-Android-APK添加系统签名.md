---
title: Android APK添加系统签名
date: 2017-04-05 11:52:01
categories:
tags:
---

---
转载请说明出处！
作者：[kqw攻城狮](http://kongqw.github.io/about/index.html)
出处：[个人站](http://kongqw.github.io) | [CSDN](http://blog.csdn.net/q4878802/)

---

将应用设置为系统级应用。可以调用系统级别API。

[下载](http://download.csdn.net/detail/q4878802/9902460) 签名文件

在`AndroidManifest.xml`中添加`sharedUserId` 为 `android.uid.system`，设置应用为系统级。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="……"
    android:sharedUserId="android.uid.system">
    ……
</manifest>
```

生成APK

> 此时生成的APK是无法安装并运行的，因为在上一步已经设置了应用为系统级应用，但是还并没有添加系统签名。

解压下载好的签名文件并添加系统签名

``` shell
java -jar signapk.jar platform.x509.pem platform.pk8 上一步生成的未添加系统签名的APK文件.apk 要生成的签名文件.apk
```



