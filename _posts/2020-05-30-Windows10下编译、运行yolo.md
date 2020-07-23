---
title: Windows10下编译、运行yolo
tags: 图像
---

1. 安装Nvidia CUDA（CUDA自带Nvidia显卡驱动），最新版10.2就可以
https://developer.nvidia.com/cuda-downloads，接着安装cuDNN（注意要与CUDA版本对应）需要注册Nvidia开发者账户
https://developer.nvidia.com/cudnn

2. 安装OpenCV，最新版4.30就可以
https://opencv.org/releases/

3. 安装Visual Studio Community 2019
https://visualstudio.microsoft.com/zh-hans/

4. 下载（或者git clone）darknet框架，和网络权值文件
https://github.com/AlexeyAB/darknet.git
https://pjreddie.com/media/files/yolov3.weights

5. 进入darknet文件夹，使用Powershell运行build.ps1

编译成功后，参照yolo官网的运行步骤，先将yolov3.weights移动到darknet目录下，然后在Powershell下运行命令即可

![](https://upload-images.jianshu.io/upload_images/20868148-938d939b35ca3278.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

运行时间约为50ms，结束后会通过OpenCV展示图像，也可以在darknet目录下看到识别后的图像predictions.jpg

![predictions.jpg](https://upload-images.jianshu.io/upload_images/20868148-2a02b8cb30a75649.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 参考：

- [YOLO: Real-Time Object Detection](https://pjreddie.com/darknet/yolo/)

- [AlexeyAB/darknet](https://github.com/AlexeyAB/darknet#how-to-compile-on-windows-using-cmake)
