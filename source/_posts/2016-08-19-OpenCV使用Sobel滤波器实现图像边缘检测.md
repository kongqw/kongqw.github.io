---
title: OpenCV使用Sobel滤波器实现图像边缘检测
date: 2016-08-19 17:35:55
categories: OpenCV
tags: [OpenCV]
---

---
转载请说明出处！
作者：[kqw攻城狮](http://kongqw.github.io/about/index.html)
出处：[个人站](http://kongqw.github.io) | [CSDN](http://blog.csdn.net/q4878802/)

---

# 效果图

![效果图](http://img.blog.csdn.net/20160819175140742)

![原图](http://img.blog.csdn.net/20160819175154258)

# 源码

[KqwOpenCVFeaturesDemo](https://github.com/kongqw/KqwOpenCVFeaturesDemo)

**Sobel滤波器**也叫**Sobel算子**，与Canny边缘检测一样，需要计算像素的灰度梯度，只不过是换用另一种方式。

# 使用Sobel算子计算边缘的步骤

1. 将图像转为灰度图像

	``` java
	// 原图置灰
	Imgproc.cvtColor(src, grayMat, Imgproc.COLOR_BGR2GRAY);
	```

2. 计算水平方向灰度梯度的绝对值

	``` java
	Imgproc.Sobel(grayMat, grad_x, CvType.CV_16S, 1, 0, 3, 1, 0);
	Core.convertScaleAbs(grad_x, abs_grad_x);
	```

3. 计算垂直方法灰度梯度的绝对值

	``` java
	Imgproc.Sobel(grayMat, grad_y, CvType.CV_16S, 0, 1, 3, 1, 0);
	Core.convertScaleAbs(grad_y, abs_grad_y);
	```

4. 计算最终梯度

	``` java
    // 计算结果梯度
    Core.addWeighted(abs_grad_x, 0.5, abs_grad_y, 0.5, 1, sobel);
	```

最终的梯度实质上就是边缘。

这里用到了两个`3 * 3`的核对图像做卷积来近似地计算水平和垂直方向的灰度梯度

![核](http://img.blog.csdn.net/20160819183329090)

# 封装

这里用到了RxJava。主要是因为图片处理是耗时操作，会阻塞线程，为了防止界面卡顿，这里使用RxJava进行了线程切换。

``` java
/**
 * Sobel滤波器
 *
 * @param bitmap 要检测的图片
 */
public void sobel(Bitmap bitmap) {
    if (null != mSubscriber)
        Observable
                .just(bitmap)
                .map(new Func1<Bitmap, Bitmap>() {

                    @Override
                    public Bitmap call(Bitmap bitmap) {

                        Mat grayMat = new Mat();
                        Mat sobel = new Mat();
                        Mat grad_x = new Mat();
                        Mat grad_y = new Mat();
                        Mat abs_grad_x = new Mat();
                        Mat abs_grad_y = new Mat();

                        // Bitmap转为Mat
                        Mat src = new Mat(bitmap.getHeight(), bitmap.getWidth(), CvType.CV_8UC4);
                        Utils.bitmapToMat(bitmap, src);
                        // 原图置灰
                        Imgproc.cvtColor(src, grayMat, Imgproc.COLOR_BGR2GRAY);

                        // 计算水平方向梯度
                        Imgproc.Sobel(grayMat, grad_x, CvType.CV_16S, 1, 0, 3, 1, 0);
                        // 计算垂直方向梯度
                        Imgproc.Sobel(grayMat, grad_y, CvType.CV_16S, 0, 1, 3, 1, 0);
                        // 计算两个方向上的梯度的绝对值
                        Core.convertScaleAbs(grad_x, abs_grad_x);
                        Core.convertScaleAbs(grad_y, abs_grad_y);
                        // 计算结果梯度
                        Core.addWeighted(abs_grad_x, 0.5, abs_grad_y, 0.5, 1, sobel);

                        // Mat转Bitmap
                        Bitmap processedImage = Bitmap.createBitmap(sobel.cols(), sobel.rows(), Bitmap.Config.ARGB_8888);
                        Utils.matToBitmap(sobel, processedImage);

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

// Sobel滤波器检测图像边缘
mFeaturesUtil.sobel(mSelectImage);
```
