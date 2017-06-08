---
title: Android使用OpenCV实现人脸识别
date: 2016-07-06 17:48:57
categories: OpenCV
tags: [OpenCV]
---

---
转载请说明出处！
作者：[kqw攻城狮](http://kongqw.github.io/about/index.html)
出处：[个人站](http://kongqw.github.io) | [CSDN](http://blog.csdn.net/q4878802/)

---

# 效果图

先上效果图，GIF不好弄

![人脸检测](http://img.blog.csdn.net/20160706191730959)

![人脸特征保存](http://img.blog.csdn.net/20160706191746100)

![人脸特征保存](http://img.blog.csdn.net/20160706191806413)

![人脸对比](http://img.blog.csdn.net/20160706191823102)


在网上找了在Android平台上使用OpenCV相关的教程，很少，大部分也都主要是介绍下人脸检测，很少有讲人脸识别，还有的人连人脸检测和人脸识别的概念都没有搞清，人脸识别只是识别到有人脸，能获取到一个人脸的大概位置，有几个人脸，而人脸识别是要获取到人脸特征做对比，识别这个人脸。有好多文章都写自己在讲人脸识别，实际上他只是在做人脸检测。

[OpenCV官网](http://opencv.org/)

官方给的Demo是在Eclipse工程下的，如果你现在已经是在Android Studio下开发，因为Eclipse工程有makefile文件，迁移到Android Studio好像还是有点麻烦，我是干脆就在Eclipse下跑的Demo。

先甩过来官方给的一些文档：

[OpenCV4Android SDK](http://docs.opencv.org/2.4/doc/tutorials/introduction/android_binary_package/O4A_SDK.html)


[Android Development with OpenCV](http://docs.opencv.org/2.4/doc/tutorials/introduction/android_binary_package/dev_with_OCV_on_Android.html#dev-with-ocv-on-android)


# 实现方式

按照官方的文档，我们在Eclipse里导入Demo进去以后，是不能直接运行的，需要安装Manager的一个APK,然后在Demo工程里通过AIDL的方式，调用OpenCV的核心方法，不过Demo给实现的功能也只是一个人脸检测。

# SDK

[SDK下载](http://opencv.org/downloads.html)

下面来看一下SDK

![下载SDK](http://img.blog.csdn.net/20160706192039369)

目录：

* apk：Manager的apk
* doc：一些文档
* samples：示例工程和一些编译好的apk
* sdk：一些库文件

>  当然, 如果你的C/C++足够好，你肯定可以自己编译一个库，直接导入到工程，就不用安装Manager了，可惜了我自己还不行，哈哈……无奈安装Manager把……

如何将Demo导入到Eclipse并运行，上面官方的文档已经说的比较清楚了，至于会有什么问题就自行Google吧。

# 人脸检测

其实人脸检测并不是重点，Demo里已经实现了人脸检测的功能。

主要的实现方式：OpenCV有一个自己的`org.opencv.android.JavaCameraView`自定义控件，它循环的从摄像头抓取数据，在回调方法中，我们能获取到Mat数据，然后通过调用OpenCV的Native方法，检测当前是否有人脸，我们会获取到一个`Rect`数组，里面会有人脸数据，最后将人脸画在屏幕上，到此为止，Demo的人脸检测功能，就结束了。


# 人脸识别

人脸识别我这里用到了JavaCV

![JavaCV](http://img.blog.csdn.net/20160706192304870)

人脸识别逻辑：人脸识别的主要方式就是获取到人脸的特征值，然后将两个特征值做比对，取到一个相似度去做人脸识别，OpenCV这里的特征值，其实就是一张图片。
我们的从回调的Mat数据检测到有人脸以后，提取特征值（也就是保存人脸的一张图片到某个路径），然后比较特征值

为了提高识别的准确度，需要在检测到人脸以后，把人脸的部分截取出来，然后置灰（置灰的目的是为了方式色泽和明暗度对识别有影响）。


## 保存人脸特征值

``` java
/**
 * 特征保存
 *
 * @param image    Mat
 * @param rect     人脸信息
 * @param fileName 文件名字
 * @return 保存是否成功
 */
public boolean saveImage(Mat image, Rect rect, String fileName) {
    try {
        String PATH = Environment.getExternalStorageDirectory() + "/FaceDetect/" + fileName + ".jpg";
        // 把检测到的人脸重新定义大小后保存成文件
        Mat sub = image.submat(rect);
        Mat mat = new Mat();
        Size size = new Size(100, 100);
        Imgproc.resize(sub, mat, size);
        Highgui.imwrite(PATH, mat);
        return true;
    } catch (Exception e) {
        e.printStackTrace();
        return false;
    }
}
```

## 提取特征值

之前已经说了嘛，人脸特征其实就是一张人脸图片,所以提取人脸特征其实就是获取一张人脸图片


``` java
/**
 * 提取特征
 *
 * @param fileName 文件名
 * @return 特征图片
 */
public Bitmap getImage(String fileName) {
    try {
        return BitmapFactory.decodeFile(Environment.getExternalStorageDirectory() + "/FaceDetect/" + fileName + ".jpg");
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }
}
```

## 人脸识别

这里主要使用了JavaCV的方法

``` java
/**
 * 特征对比
 *
 * @param file1 人脸特征
 * @param file2 人脸特征
 * @return 相似度
 */
public double CmpPic(String file1, String file2) {
    int l_bins = 20;
    int hist_size[] = {l_bins};

    float v_ranges[] = {0, 100};
    float ranges[][] = {v_ranges};

    opencv_core.IplImage Image1 = cvLoadImage(Environment.getExternalStorageDirectory() + "/FaceDetect/" + file1 + ".jpg", CV_LOAD_IMAGE_GRAYSCALE);
    opencv_core.IplImage Image2 = cvLoadImage(Environment.getExternalStorageDirectory() + "/FaceDetect/" + file2 + ".jpg", CV_LOAD_IMAGE_GRAYSCALE);

    opencv_core.IplImage imageArr1[] = {Image1};
    opencv_core.IplImage imageArr2[] = {Image2};

    opencv_imgproc.CvHistogram Histogram1 = opencv_imgproc.CvHistogram.create(1, hist_size, CV_HIST_ARRAY, ranges, 1);
    opencv_imgproc.CvHistogram Histogram2 = opencv_imgproc.CvHistogram.create(1, hist_size, CV_HIST_ARRAY, ranges, 1);

    cvCalcHist(imageArr1, Histogram1, 0, null);
    cvCalcHist(imageArr2, Histogram2, 0, null);

    cvNormalizeHist(Histogram1, 100.0);
    cvNormalizeHist(Histogram2, 100.0);

    return cvCompareHist(Histogram1, Histogram2, CV_COMP_CORREL);
}
```

# 最后

看我轻描淡写了一篇博客写完了，看上去好像挺容易的，但是对于第一次做人脸识别的人或者对JNI还不是很熟的人来说，可能并没有想象的那么简单，你会遇到各种库找不到，编译不通过等等的问题，总之吧，一个大概的实现思路呈现出来，**仅供参考**，就是这样！有时间的话会再深入学习一下。
