---
title: “深度”自由！一个开源单摄像头高精度深度图工具
description: 介绍一个由苹果开源的，单目摄像头就可以实现深度图输出的工具，号称精确到发丝级！
pubDatetime: 2024-11-12
tags:
  - 3D重建
share: true
featured: false
---
实现图像和视频的深度估计一直是CV领域重要的议题，因为它反映了计算机“空间感知”的能力。而这能力，在体感游戏、机器人导航以及自动驾驶等领域中都是非常关键的一环。
人们发展了多种深度估计方法，比如激光雷达（Lidar），深度摄像头和超声波测距等，基本思路都是采用独立于可见光的常规视觉方法来独立实现距离测量，并将深度信息融合到视觉感知系统中，显然成本是不得不考虑的问题。随着神经网络技术的发展，基于人眼双目视觉的基本视差原理，又提出了双目视觉方案，通过两个摄像头成像的差别，来生成深度图，这样可以将深度估计“原生”的集成在视觉感知系统中，降低了成本且不需要额外的传感器，但是双目摄像头方案，还是在鲁棒性和成本上存在劣势，典型的例子就是，如果一个摄像头坏了，那么不仅影响视觉判断，深度估计功能也随之丧失。

因此，打铁还需自身硬！**单摄像头图像就能提供高质量可信的深度图才是根本解决方案！**
## 项目原版
这次苹果提出的就是一个只需要一个图像输入就可以生成精确深度信息的工具。单目摄像头的深度估计从此不是问题了。开源代码地址在：[GitHub - apple/ml-depth-pro: Depth Pro: Sharp Monocular Metric Depth in Less Than a Second.](https://github.com/apple/ml-depth-pro)
但是，按照页面说明来部署的话，你得到的其实是一个CPU版本，推理速度会非常慢（项目页面中有人说是主流消费级CPU需要一个多小时才完成一次推理😂）！
因为它基于Pytorch，进而有CPU和GPU版本的区别，因此，如果你有NVIDIA的显卡，那么可以移步别人准备好的本项目的GPU版本。
## GPU加强版
**项目主页**：[lattebyte/DepthPro-Windows-GPU: GPU accelerated camera/ video depth estimation.](https://github.com/lattebyte/DepthPro-Windows-GPU)
好处在于，安装了GPU版本的Pytorch，可以在运行时调用本机的NVIDIA显卡来进行深度估计，大幅缩短了推理时间，可以近乎实时的使用（近乎实时，也就别指望太快，在我的RTX3090上其实就是每秒几帧这样）。
## GPU安装注意事项
**安装指导非常详细基本没问题！** 按照最初安装指导去做，会报错，说什么png处理有问题：["png_get_eXIf_1 ", has anyone encountered this issue? How to deal with it? · Issue #3 · lattebyte/DepthPro-Windows-GPU](https://github.com/lattebyte/DepthPro-Windows-GPU/issues/3)
所以如果你也有类似问题，在按照项目页面的README一步步实施的时候，请尝试我的方法：
1. 不用`conda install`来安装`pytorch`，安装时候会有一些组件的CUDA版本不一致，使用`pip install`来安装！即使用下面的命令：
   ```bash
   pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
	```

2. 作者忘记将`OpenCV`放入了，因此我们需要手动装一下，使用`pip install python-contrib-opencv`来进行安装
这样，应该就可以使用了，下图是我用摄像头捕捉的图像直接生成的深度图。在我RTX3090上可以跑不到10fps吧，很卡。
![屏幕截图 2024-11-12 124420.png](https://img.picui.cn/free/2024/11/12/6732dd35a44a8.png)