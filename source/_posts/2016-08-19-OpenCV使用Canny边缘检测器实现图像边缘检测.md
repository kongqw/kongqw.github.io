---
title: OpenCV使用Canny边缘检测器实现图像边缘检测
date: 2016-08-19 16:31:02
categories: OpenCV
tags: [OpenCV]
---

---
转载请说明出处！
作者：[kqw攻城狮](http://kongqw.github.io/about/index.html)
出处：[个人站](http://kongqw.github.io) | [CSDN](http://blog.csdn.net/q4878802/)

---

# 效果图

![边缘检测结果](http://img.blog.csdn.net/20160819171248584)

![原图](http://img.blog.csdn.net/20160819171257725)

# 源码

[KqwOpenCVFeaturesDemo](https://github.com/kongqw/KqwOpenCVFeaturesDemo)

**Canny边缘检测器**是一种被广泛使用的算法，并被认为是边缘检测最优的算法，该方法使用了比高斯差分算法更复杂的技巧，如多向灰度梯度和滞后阈值化。

# Canny边缘检测器算法基本步骤

1. **平滑图像**：通过使用合适的模糊半径执行高斯模糊来减少图像内的噪声。
2. **计算图像的梯度**：这里计算图像的梯度，并将梯度分类为垂直、水平和斜对角。这一步的输出用于在下一步中计算真正的边缘。
3. **非最大值抑制**：利用上一步计算出来的梯度方向，检测某一像素在梯度的正方向和负方向上是否是局部最大值，如果是，则抑制该像素（像素不属于边缘）。这是一种边缘细化技术，用最急剧的变换选出边缘点。
4. **用滞后阈值化选择边缘**：最后一步，检查某一条边缘是否明显到足以作为最终输出，最后去除所有不明显的边缘。

算法比较复杂，但是使用很简单，首先将图像灰度化

``` java
// 原图置灰
Imgproc.cvtColor(src, grayMat, Imgproc.COLOR_BGR2GRAY);
```

然后调用`Imgproc.Canny()`方法即可

``` java
// Canny边缘检测器检测图像边缘
Imgproc.Canny(grayMat, cannyEdges, 10, 100);
```

* 第一个参数表示图像输入
* 第二个参数表述图像输出
* 第三个参数表示**低阈值**
* 第四个参数表示**高阈值**

在Canny边缘检测算法中，将图像中的点归为三类：

* 被抑制点

	灰度梯度值 < 低阈值

* 弱边缘点

	低阈值 <= 灰度梯度值 <= 高阈值

* 强边缘点

	高阈值 < 灰度梯度值


# 封装

这里用到了RxJava。主要是因为图片处理是耗时操作，会阻塞线程，为了防止界面卡顿，这里使用RxJava进行了线程切换。

``` java
/**
 * Canny边缘检测算法
 *
 * @param bitmap 要检测的图片
 */
public void canny(Bitmap bitmap) {
    if (null != mSubscriber)
        Observable
                .just(bitmap)
                .map(new Func1<Bitmap, Bitmap>() {

                    @Override
                    public Bitmap call(Bitmap bitmap) {

                        Mat grayMat = new Mat();
                        Mat cannyEdges = new Mat();

                        // Bitmap转为Mat
                        Mat src = new Mat(bitmap.getHeight(), bitmap.getWidth(), CvType.CV_8UC4);
                        Utils.bitmapToMat(bitmap, src);

                        // 原图置灰
                        Imgproc.cvtColor(src, grayMat, Imgproc.COLOR_BGR2GRAY);
                        // Canny边缘检测器检测图像边缘
                        Imgproc.Canny(grayMat, cannyEdges, 10, 100);

                        // Mat转Bitmap
                        Bitmap processedImage = Bitmap.createBitmap(cannyEdges.cols(), cannyEdges.rows(), Bitmap.Config.ARGB_8888);
                        Utils.matToBitmap(cannyEdges, processedImage);

                        return processedImage;
                    }
                })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(mSubscriber);
}
```


# 使用

``` java
// 图片特征提取的工具类
mFeaturesUtil = new FeaturesUtil(new Subscriber<Bitmap>() {
    @Override
    public void onCompleted() {
        // 图片处理完成
        dismissProgressDialog();
    }

    @Override
    public void onError(Throwable e) {
        // 图片处理异常
        dismissProgressDialog();
    }

    @Override
    public void onNext(Bitmap bitmap) {
        // 获取到处理后的图片
        mImageView.setImageBitmap(bitmap);
    }
});

// Canny边缘检测器检测图像边缘
mFeaturesUtil.canny(mSelectImage);
```