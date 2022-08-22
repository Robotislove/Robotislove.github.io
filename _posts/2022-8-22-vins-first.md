---
layout: post
title: Vins论文阅读笔记理论篇
date: 2022-08-22
author: lau
tags: [SLAM, Blog]
comments: true
toc: true
pinned: false
---
Vins作为经典的VIO系统，这里记录它的原理和实践。

<!-- more -->

## 理论

### VINS Fusion简介

VINS Fusion在VINS Mono的基础上，添加了GPS等可以获取全局观测信息的传感器，使得VINS可以利用全局信息消除累计误差，进而减小闭环依赖。此外，全局信息可以使分多次运行的VINS Mono统一到一个坐标系，从而方便协同建图和定位。

### Fusion的思路
局部传感器(如相机，IMU，激光雷达等)被广泛应用于建图和定位算法。尽管这些传感器能够在没有GPS信息的区域，实现良好的局部定位和局部建图效果，但是这些传感器只能提供局部观测，限制了其应用场景：

1. 第一个问题是局部观测数据缺乏全局约束，当我们每次在不同的位置运行算法时，都会得到不同坐标系下的定位和建图结果，因而难以将这些测量结果结合起来，形成全局效果。
2. 第二个问题是基于局部观测的估计算法必然存在累计漂移，尽管回环检测可以一定程度上消除漂移，但是对于数据量较大的大范围场景，算法依然难以处理。
相比于局部传感器，全局传感器(如GPS，气压计和磁力计等)可以提供全局观测。这些传感器使用全局统一坐标系，并且输出的观测数据的方差不随时间累计而增加。但这些传感器也存在一些问题，导致无法直接用于精确定位和建图。以GPS为例，GPS数据通常不平滑，存在噪声，且输出速率低。

因此，一个简单而直观的想法是将局部传感器和全局传感器结合起来，以达到局部精确全局零漂的效果。这也就是VINS Fusion这篇论文的核心内容。

VINS Fusion的算法架构如图所示：

![](https://zhi-ang.github.io/2019/09/11/vins_fusion/vins_fusion.png)

下图以因子图的方式表示观测和状态之间的约束：

![](https://zhi-ang.github.io/2019/09/11/vins_fusion/constraints.PNG)

其中圆形为状态量(如位姿，速度，偏置等)，黄色正方形为局部观测的约束，即来自VO/VIO的相对位姿变换；而其他颜色的正方形为全局观测的约束，比如紫色正方形为来自GPS的约束。

## 参考文献
[1] https://blog.csdn.net/u011178262/article/details/88769414

[2] https://zhehangt.github.io/2018/04/24/SLAM/VINS/VINSVIO/

[3] https://www.codetd.com/article/4903500

[4] 沈老师PPT：https://slides.games-cn.org/pdf/GAMES201840%E6%B2%88%E5%8A%AD%E5%8A%BC.pdf

//estimate 代码及推导

[5] https://zhuanlan.zhihu.com/p/61733458

[6] https://cggos.github.io/slam/vinsmono_note_cg.html

[7] https://www.zybuluo.com/Xiaobuyi/note/866099

欧拉积分、中点积分与龙格－库塔积分：

[8] http://liuxiao.org/2018/05/%E6%AC%A7%E6%8B%89%E7%A7%AF%E5%88%86%E3%80%81%E4%B8%AD%E7%82%B9%E7%A7%AF%E5%88%86%E4%B8%8E%E9%BE%99%E6%A0%BC%EF%BC%8D%E5%BA%93%E5%A1%94%E7%A7%AF%E5%88%86/

[9] https://scm_mos.gitlab.io/2019/04/23/VINS-FUSION/

[10] https://zhi-ang.github.io/categories/SLAM/

[11] GPS/VIO https://zhi-ang.github.io/2019/09/11/vins_fusion/ 

[12] 带有代码流程图：https://www.guyuehome.com/blog/column/id/6 \ https://blog.csdn.net/qq_42700518/category_9544233.html

[13] 简洁的介绍：https://www.sohu.com/a/284064259_715754

[14] https://blog.csdn.net/qq_41839222/article/details/86633487 vins_fusion论文解读

[15] 各个线程的解读：https://blog.csdn.net/moyu123456789/category_8945611.html
