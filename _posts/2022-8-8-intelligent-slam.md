---
layout: post
title: SLAM学习入门与相关知识笔记
date: 2022-08-8
author: lau
tags: [SLAM,Archive]
comments: true
toc: false
pinned: false
---
SLAM学习入门与相关知识笔记。

<!-- more -->

## SLAM算法总结

### SLAM基础知识
[多视图几何总结——基础矩阵、本质矩阵和单应矩阵的求解过程](https://blog.csdn.net/weixin_44580210/article/details/90116621)

[视觉SLAM总结——后端总结](https://blog.csdn.net/weixin_44580210/article/details/90573282)

[[V-SLAM] Bundle Adjustment 实现](https://zhuanlan.zhihu.com/p/64471565)

[Karto slam源码阅读(一) 主流程](https://zhuanlan.zhihu.com/p/493303177)

[Hector slam的后端优化](https://zhuanlan.zhihu.com/p/493322053)

[SLAM轨迹评估工具evo](https://zhuanlan.zhihu.com/p/498614224)

[VINS_MONO预积分](https://www.zhihu.com/column/slamTech)

[ORB-SLAM3源码阅读(一) 初始化](https://zhuanlan.zhihu.com/p/510748490)

[自己从零手写一个激光slam](https://github.com/softdream/Slam-Project-Of-MyOwn)

[ESKF误差卡尔玛滤波](https://zhuanlan.zhihu.com/p/441182819)

[VINS-FUSION 前端后端代码全详解](https://mp.weixin.qq.com/s/hoPDnZhT7ltkKib6mqSTcA)

[SLAM本质剖析-ceres](https://mp.weixin.qq.com/s/fKlG9LWlPI52wStUAv18iw)

[BA视觉SLAM解析](https://www.cnblogs.com/Jessica-jie/p/7739775.html)

[VIO回顾：从滤波和优化的视角](https://mp.weixin.qq.com/s/zpZERtWPKljWNAiASBLJxA)

[重投影误差](https://www.cnblogs.com/Jessica-jie/p/7242179.html)

[从零到一SLAM](https://www.guyuehome.com/blog/column/id/6)

[vins_mono源码笔记](https://blog.csdn.net/qq_42700518/category_9544233.html)


[港科大vins-fusion初探](https://blog.csdn.net/huanghaihui_123/article/details/86518880)

[VINS-Mono论文学习与代码解读——目录与参考](https://blog.csdn.net/qq_41839222/article/details/85793998)

[VINS-FUSION 前端后端代码全详解](https://www.guyuehome.com/34507)

[超级详细的slam各种框架解读](https://zhuanlan.zhihu.com/p/83775731)

[VINS技术路线与代码详解](https://blog.csdn.net/wangshuailpp/article/details/78461171)

[【SLAM】VINS-MONO解析——前端 有各种流程图](https://blog.csdn.net/iwanderu/article/details/104618993)

[cvlibs kitti 视觉slam排名](http://www.cvlibs.net/datasets/kitti/eval_odometry.php)

[vins_mono论文解读](https://blog.csdn.net/MAX_Hope/article/details/90046770?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-90046770-blog-86633487.t0_layer_searchtargeting_sa&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-90046770-blog-86633487.t0_layer_searchtargeting_sa&utm_relevant_index=4)

[移动机器人中的坐标转换](https://zhi-ang.github.io/2019/09/27/coord_trans/)
### 经典SLAM算法框架总结
首先，按照我的理解，我梳理了如下一个思维导图，如果读者发现有什么需要补充或者纠正的欢迎随时交流：
![](https://img-blog.csdnimg.cn/d35d5ae2a0d64325abbc1aad38b08ce0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBASmljaGFvX1Blbmc=,size_20,color_FFFFFF,t_70,g_se,x_16)

按照分类，我们先来讲讲视觉SLAM，视觉SLAM算法相对于激光SLAM算法的特点是信息更加丰富，由于是在二维提取特征点，因此通常可以达到更高的频率，但也正是因为信息丰富，因此更容易引入噪声，加上缺乏三维信息，导致视觉SLAM算法的鲁棒性在平均水平上要低于激光SLAM，尤其是通过传统特征进行定位和建图，在工程应用上相对受限，当前一个热门的方向是通过网络提取更加鲁棒的特征，例如SuperPixel、SuperGlue，或者直接根据网络输出定位和建图结果，这也是我之后希望进一步了解的方向：

以下是一些视觉SLAM的博客链接，感兴趣的同学可以了解下：
纯视觉方案：
[视觉SLAM总结——ORB SLAM2中关键知识点总结](https://blog.csdn.net/weixin_44580210/article/details/90760584?spm=1001.2014.3001.5501)
[视觉SLAM总结——SVO中关键知识点总结](https://blog.csdn.net/weixin_44580210/article/details/90967270?spm=1001.2014.3001.5501)
[视觉SLAM总结——LSD SLAM中关键知识点总结](https://blog.csdn.net/weixin_44580210/article/details/91174015?spm=1001.2014.3001.5501)

结合IMU方案：
[VINS-Mono关键知识点总结——前端详解](https://blog.csdn.net/weixin_44580210/article/details/94355964?spm=1001.2014.3001.5501)
[VINS-Mono关键知识点总结——边缘化marginalization理论和代码详解](https://blog.csdn.net/weixin_44580210/article/details/95748091?spm=1001.2014.3001.5501)
[VINS-Mono关键知识点总结——预积分和后端优化IMU部分](https://blog.csdn.net/weixin_44580210/article/details/93377806?spm=1001.2014.3001.5501)

[学习MSCKF笔记——前端、图像金字塔光流、Two Point Ransac](https://blog.csdn.net/weixin_44580210/article/details/107282889?spm=1001.2014.3001.5501)
[学习MSCKF笔记——四元数基础](https://blog.csdn.net/weixin_44580210/article/details/107451444?spm=1001.2014.3001.5501)
[学习MSCKF笔记——真实状态、标称状态、误差状态](https://blog.csdn.net/weixin_44580210/article/details/107602271?spm=1001.2014.3001.5501)
[学习MSCKF笔记——后端、状态预测、状态扩增、状态更新](https://blog.csdn.net/weixin_44580210/article/details/108021350?spm=1001.2014.3001.5501)

结合激光方案：
[视觉激光融合——VLOAM / LIMO算法解析](https://blog.csdn.net/weixin_44580210/article/details/119857381?spm=1001.2014.3001.5501)

我是先入门的视觉SLAM再接触的激光SLAM，因此激光SLAM我接触的时间并不是很长，但是激光SLAM和视觉SLAM的基本方法是一样的，只是在传感器输入处理上会稍有不同，正如上面提到的，激光SLAM在工程应用方面会更加成熟，以下是一些激光SLAM的博客链接：
纯激光方案：
[学习LOAM笔记——特征点提取与匹配](https://blog.csdn.net/weixin_44580210/article/details/119849673?spm=1001.2014.3001.5501)

结合IMU方案：
[激光IMU融合——LIO-Mapping / LIOM / LINS / LIO-SAM算法解析](https://blog.csdn.net/weixin_44580210/article/details/119857381?spm=1001.2014.3001.5501)

结合视觉方案：
[视觉激光融合——VLOAM / LIMO算法解析](https://blog.csdn.net/weixin_44580210/article/details/119857381?spm=1001.2014.3001.5501)
在视觉和激光结合的方向上，在2021年的ICRA上还有一片LVI-SAM，工程实现上是VINS-Mono和LIO-SAM的结合。



