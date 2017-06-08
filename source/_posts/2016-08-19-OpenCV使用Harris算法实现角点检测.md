---
title: OpenCV使用Harris算法实现角点检测
date: 2016-08-19 19:32:39
categories: OpenCV
tags: [OpenCV]
---

---
转载请说明出处！
作者：[kqw攻城狮](http://kongqw.github.io/about/index.html)
出处：[个人站](http://kongqw.github.io) | [CSDN](http://blog.csdn.net/q4878802/)

---

# 效果图

![效果图](http://img.blog.csdn.net/20160819194926871)

![原图](http://img.blog.csdn.net/20160819194937262)

# 源码

[KqwOpenCVFeaturesDemo](https://github.com/kongqw/KqwOpenCVFeaturesDemo)


**角点**是两条边缘的交点或者在局部邻域中有多个显著边缘方向的点。Harris角点检测是一种在角点检测中最常见的技术。

Harris角点检测器在图像上使用滑动窗口计算亮度的变化。



# 封装

这里用到了RxJava。主要是因为图片处理是耗时操作，会阻塞线程，为了防止界面卡顿，这里使用RxJava进行了线程切换。

``` java
/**
 * Harris角点检测
 *
 * @param bitmap 要检测的图片
 */
public void harris(Bitmap bitmap) {
    if (null != mSubscriber)
        Observable
                .just(bitmap)
                // 检测边缘
                .map(new Func1<Bitmap, Mat>() {
                    @Override
                    public Mat call(Bitmap bitmap) {
                        Mat grayMat = new Mat();
                        Mat cannyEdges = new Mat();

                        // Bitmap转为Mat
                        Mat src = new Mat(bitmap.getHeight(), bitmap.getWidth(), CvType.CV_8UC4);
                        Utils.bitmapToMat(bitmap, src);

                        // 原图置灰
                        Imgproc.cvtColor(src, grayMat, Imgproc.COLOR_BGR2GRAY);
                        // Canny边缘检测器检测图像边缘
                        Imgproc.Canny(grayMat, cannyEdges, 10, 100);

                        return cannyEdges;
                    }
                })
                // Harris对角检测
                .map(new Func1<Mat, Bitmap>() {

                    @Override
                    public Bitmap call(Mat cannyEdges) {
                        Mat corners = new Mat();
                        Mat tempDst = new Mat();

                        // 找出角点
                        Imgproc.cornerHarris(cannyEdges, tempDst, 2, 3, 0.04);

                        // 归一化Harris角点的输出
                        Mat tempDstNorm = new Mat();
                        Core.normalize(tempDst, tempDstNorm, 0, 255, Core.NORM_MINMAX);
                        Core.convertScaleAbs(tempDstNorm, corners);

                        // 在新的图像上绘制角点
                        Random r = new Random();
                        for (int i = 0; i < tempDstNorm.cols(); i++) {
                            for (int j = 0; j < tempDstNorm.rows(); j++) {
                                double[] value = tempDstNorm.get(j, i);
                                if (value[0] > 150) {
                                    Core.circle(corners, new Point(i, j), 5, new Scalar(r.nextInt(255), 2));
                                }
                            }
                        }

                        // Mat转Bitmap
                        Bitmap processedImage = Bitmap.createBitmap(corners.cols(), corners.rows(), Bitmap.Config.ARGB_8888);
                        Utils.matToBitmap(corners, processedImage);

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

// Harris角点检测
mFeaturesUtil.harris(mSelectImage);
```