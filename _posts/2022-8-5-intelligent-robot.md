---
layout: post
title: 智能服务机器人3D-SLAM优化汇总
date: 2022-08-5
author: lau
tags: [SLAM,Archive]
comments: true
toc: false
pinned: false
---
智能服务机器人3D-SLAM优化。

<!-- more -->

## 摘要

在目前开源方案里，能够同时融合gps、imu、lidar等多种传感器的激光SLAM框架还比较少。cartographer算是一种，但相对来说，hdl_graph_slam在资源消耗、代码复杂度、建图流程等方面具有较大优势，本文针对其后端优化模块阐述相关知识点、算法结构，及代码解读。

### 1.介绍

hdl-graph-slam是一个非常优秀的3d 激光SLAM框架，不仅融合了imu、gps、平面、激光等多种输入，同时也带闭环检测等后端处理，即完整的一个slam架构。其功能较为全面，包括平面检测及提取、点云预处理、激光里程计、闭环检测、GPS和IMU数据融合、后端图优化等等。

#### 1.1 框架流程

![](https://s1.328888.xyz/2022/08/10/4fcEg.jpg)

首先读入激光雷达的点云数据，然后将原始的点云数据进行预滤波，经过滤波后的数据分别给到点云匹配里程计及地面检测节点，两个节点分别计算连续两帧的相对运动和检测到的地面的参数，并将这两种消息送到hdl_graph_slam节点进行位姿图的更新及回环检测，并发布地图的点云数据。算法框架采用eigen、pcl、g2o等开源库。

**地面检测**

当所处的环境中地面为平面时，它就可以作为一个很有效的信息去约束高程的误差。在没有gps的情况下，高程误差是主要的误差项。当然在地面为非平面的时候是不能够使用的，在使用时根据环境决定是否开启这项功能。

**地图精度**

激光SLAM在构建地图的时候，精度较高，一般可达到2cm左右；VSLAM，比如常见的深度摄像机Kinect(测距范围在3-12m之间)，地图构建精度约3cm；所以激光SLAM构建的地图精度一般来说比VSLAM高，且能直接用于定位导航。

### 2. 基础知识

本章节整理了一些与3D SLAM 后优化相关的必需掌握的基础知识点，供参考。

#### 2.1 非线性优化
##### 2.1.1 高斯牛顿法
GN(Gauss-Newton)是优化算法里最简单的方法之一，思想是将$f(x)$进行一阶的泰勒展开。

$$
f(x+\Delta x) \approx f(x)+J(x) \Delta x
$$
其中$J(x)$为$f(x)$在$x$的导数，是一个雅克比矩阵。当前的目标是为了寻找下降矢量$\Delta x$，使得\|f(\boldsymbol{x}+\Delta \boldsymbol{x})\|^{2}达到最小。为了求$\Delta x$需要求解一个线性最小二乘问题：

$$
\Delta {x}^{*} =\arg \min_{\Delta \boldsymbol{x}} \frac{1}{2}\|f(\boldsymbol{x})+\boldsymbol{J}(\boldsymbol{x}) \Delta \boldsymbol{x}\|^{2}
$$

根据极值条件，将上述目标函数对求导，并令其导数为零。由于这里考虑的是的导数而不是$x$,最后将得到一个线性方程。

![](http://image.huawei.com/tiny-lts/v1/images/59c0fcbe7ec53434b985255d30f4e24e_626x268.png@900-0-90-f.png)

注，此处要求解的变量是$\Delta$，因此这是一个线性方程组,我们称为增量方程，也叫高斯牛顿方程，也叫正规方程。把左边系数定义为H，右边定义为g,则上式变为：$H\Delta=g$。这里把左侧记做 H 是有意义的，对比牛顿法。 GN用$J^TJ$作为牛顿法中的二阶Hession。从而省略H的过程。求解增量方程式整个优化问题的核心所在。

如果能顺利求解次方程，那么GN算法步骤如下：
![](http://image.huawei.com/tiny-lts/v1/images/09ed4bcf96f8c56d82524275da5c8f7b_561x167.png@900-0-90-f.png)

![](http://image.huawei.com/tiny-lts/v1/images/977b990d0d525f1748dcdd739fb3cd52_598x201.png)

##### 2.1.2 Levenberg-Marquadt

LM方法在一定程度上修正了这些问题，一般认为它比GN更为鲁棒。尽管它的收敛速度可能比GN慢，被称之为阻尼牛顿法.由于GN中采用近似的二阶泰勒展开只能在展开点附近有较好的近似效果，所以很自然的想到应该给添加一个信赖区域，不能让它太大使得近似不准确。非线性优化有一系列这类方法，也被称之为信赖区域方法。在信赖区域里面，认为近似是有效的，出了这个区域，近似可能会出问题。

那么如何确认这个信赖区域的范围？一个比较好的方法是根据近似模型跟实际函数之间的差异来确定。如果差异够小，就让范围尽可能大。如果差异大，就缩小这个近似范围。因此，可以考虑使用
![](http://image.huawei.com/tiny-lts/v1/images/768c82a7a8e6975fec3ad145723d0c7d_281x72.png@900-0-90-f.png)

来判断泰勒近似是否够好，$\rho$的分子式实际函数下降的值，分母是近似模型下降的值。如果接近于1，则近似是好的。如果太小，说明实际减小的值远少于近似减少的值，这认为近似比较差。反之，如果比较大，则说明实际下降的比预计的更大，可以放大近似范围。

##### 2.1.3 小结

实际中还存在许多其他方式来求解函数增量，例如Dog-Leg等。以上2种也是视觉SLAM 中用的最多的方式。总的来说，非线性优化问题的框架可分为Line Search 和Trust Region 两类。Line Search 先固定搜索方向，然后在该方向寻找步长，以最速下降法和GN法为代表。而Trust Region则先固定搜索区域，再考虑找该区域内的最优点。此类方法以L-M为代表。实际问题中，通常选择GN或L-M之一作为梯度下降策略。

#### 2.2 Eigen库

Eigen是一个高层次的C++开源库，有效支持线性代数，矩阵和矢量运算，数值分析及其相关的算法。从3.1.1版本开始遵从MPL2许可，除了C++标准库以外，无任何其他依赖包。使用CMake建立配置文件和单元测试，并自动安装。如果使用Eigen库，只需包括特定模块的的头文件即可。

Eigen支持包括固定大小、任意大小的所有矩阵操作，甚至是稀疏矩阵；支持所有标准的数值类型，并且可以扩展为自定义的数值类型；支持多种矩阵分解及其几何特征的求解；它为支持的模块生态系统提供了许多专门的功能，如非线性优化，矩阵功能，多项式解算器，快速傅立叶变换等。

#### 2.3 图优化
图优化是一种将非线性优化和图论结合起来的理论。
##### 2.3.1 图优化介绍

图优化里的图就是数据结构里的图，一个图由若干个顶点(vertex)，以及连接这些顶点的边（edge）组成。比如一个机器人在房屋里移动，它在某个时刻 $t$ 的位姿（pose）就是一个顶点，这个也是待优化的变量。而位姿之间的关系就构成了一个边，比如时刻 $t$ 和时刻 $t+1$ 之间的相对位姿变换矩阵就是边，边通常表示误差项。

在SLAM优化问题中，经常求解一个代价函数：

![](http://image.huawei.com/tiny-lts/v1/images/94500b62286e465e14636bd81f96a502_524x100.png@900-0-90-f.png)

可以理解，图优化就是这个代价函数的求解问题，待优化的位姿和地图点就是图的顶点，而误差项是图的边。

在SLAM里，图优化一般分解为两个任务：

1、构建图：机器人位姿作为顶点，位姿间关系作为边。

2、优化图：调整机器人的位姿（顶点）来尽量满足边的约束，使得误差最小。

##### 2.3.2 图优化优势

在SLAM后端算法领域，一般分为两种处理方法，一种是以扩展卡尔曼滤波(EKF)的方法为代表；一种是基于非线性优化的方法，以图优化为代表。现在主流是图优化的方法，因为其可以把SLAM系统的历史状态量和路标点位置考虑进去一起优化，大大提高了轨迹和地图的精度。

在以图优化框架的视觉SLAM算法里，BA(Bundle Adjustment光束平差法、捆集调整)起到了核心作用。基于视觉的SLAM方案，路标点(特征点)数据很大，意味着BA求解的计算量也很大，没法做到实时。研究人员发现，虽然视觉SLAM问题中包含大量特征点和相机位姿，但BA有着稀疏的特性，使得它能够在实时的场景中使用。这也是图优化成为主流方法的原因！

#### 2.4 g2o图优化

![](http://image.huawei.com/tiny-lts/v1/images/2fc333ab9e59c649790d55c6a7f86aad_779x411.jpg@900-0-90-f.jpg)

优化的目的是为了通过当前已知的系统理想化的模型和实际测量的数据获取最接近真实值的系统结果。滤波方法和图优化方案解决的问题都是对不可靠的测量值进行处理以获取尽可能接近真实值的结果。以卡尔曼滤波器为例，在进行操作之前需要有一个相对靠谱的预测模型用来获取先验(预测)信息，以及实时的测量数据用来矫正预测信息。

g2o图优化则是将优化问题和图论相结合，最典型的作用就是将待优化问题通过测量的数据建立最小二乘并将该最小二乘问题通过图论中的边的顶点表示出来，之后调用g2o库通过求解对应的图来实现对最小二乘问题的求解以达到优化的目的。其中图优化中的点表示待优化变量，如(ＸＹＺ)；边表示误差，边依赖于点的存在而存在，边可能和一个点、俩个点、多个点相连，每条边都表示与之相连的点之间的误差。在SLAM问题中以相机位姿和观测到的路标做点，点与点之间存在的误差（重投影误差、相机位姿估计误差）做边。图优化示例：
![](http://image.huawei.com/tiny-lts/v1/images/26a99177edd22637a7df924155413f23_576x321.png@900-0-90-f.png)

注：g2o是比较流行的图优化库，在视觉slam中的应用非常广泛。只要是能把优化问题表示成顶点和边的形式，就可以非常容易的调用g2o来进行优化。

##### 2.4.1 g2o使用步骤

使用步骤主要分成以下四部分：

1) 点和边的类型定义；

2) 构建图优化实例，配置求解器；

3) 添加点和边，构建求解图；

4) 执行优化操作。

##### 2.4.2 g2o整体结构

![](http://image.huawei.com/tiny-lts/v1/images/c944cd6a77b35b8a9628d2476e4af046_783x463.png@900-0-90-f.png)

从图中可看出整个库里较为重要的类之间的继承以及包含关系，及整个框架里面最重要的SparseOptimizer这个类（或实例）。

**Vertex和Edge**

所使用的优化器最终是一个超图（hyperGrahp），而这个超图包含了许多顶点和边。在图优化中，顶点代表了要被优化的变量，而边则是连接被优化变量的桥梁。在整个优化过程中，顶点的值会越来越趋近于最优值，优化完毕后则可以将顶点的优化值作为最优值进行使用；边则是连接顶点的类型，在SLAM问题中，一般是边连接要被优化的空间点(Point)和机器人的位姿(Pose)，当然，边还可以连接一个顶点（类似于参数估计，边的数量由量测的数量决定），也可以连接多个顶点，边在图优化中的一个很大的作用就是计算误差（视觉SLAM中计算的就是空间点的映射误差），同时计算该误差对于被优化变量的jacobian矩阵，也是比较重要的存在。

注：用g2o的时候，总会遇到自己独特的顶点类型和边类型，此时需要对顶点和边进行重写。

##### 2.4.3 g2o优化算法

涉及到优化的算法，块求解器，线性求解器等部分，在程序中，这部分通常位于g2o算法的开头配置部分，一般情况下可以随着一个例程进行配置即可。

**linearSolver线性求解器**

在求解增量方程HdeltaX=-b时，通常情况下想到线性求解deltaX=-H.inv\*b，当H的维度较小的时候，上述问题变得简单，只需矩阵的求逆就能解决问题，但是当H的维度较大时，问题变得复杂，此时就需要一些特殊的方法对矩阵进行求逆，g2o中主要有图中所示的三种方法，PCG，CSparse和Cholmod方法。

注:线性求解器只是完成了一个求解的功能，是整个优化中较靠后的计算部分。

**BlockSolver块求解器**

块求解器是包含线性求解器的存在，之所以是包含，是因为块求解器会构建好线性求解器所需要的矩阵块(即H和b)，之后给线性求解器让它进行运算。

**Algorithm优化算法**

主要有三种方法：GN，LM，PSD，不同的方法主要表现在最终的H矩阵构造不同。

##### 2.4.4 算法流程参考

![](http://image.huawei.com/tiny-lts/v1/images/808efdbdc84bc90575bebdf3de39541a_783x547.png@900-0-90-f.png)

#### 2.5 回环检测
回环检测也可以称为闭环检测，是指机器人识别曾到达场景的能力。如果检测成功，可以显著地减小累积误差。回环检测实质上是一种检测观测数据相似性的算法。对于视觉SLAM，多数系统采用目前较为成熟的词袋模型(Bag-of-Words, BoW)。词袋模型把图像中的视觉特征（SIFT, SURF等）聚类，然后建立词典，进而寻找每个图中含有哪些“单词”。也有研究者使用传统模式识别的方法，把回环检测建构成一个分类问题，训练分类器进行分类。

说明： 闭环检测是slam中的重要功能，不只在没有gps参考的情况下能减小累计误差，即便在有gps约束的情况下也能起到有效的作用。

### 3. Graph SLAM
Graph SLAM又被称为Graph-based SLAM，它的基本思想是将机器人不同时刻的位姿抽象为点(pose)，机器人在不同位置上的观测所产生的约束被抽象为点之间的边，或者叫约束。即由观测或航迹推测得到的位姿之间的约束通过节点之间的边进行编码。所谓的约束可以有多种多样的形式，比如机器人在A点和B点都看到同一个消防栓(可认为这是固定在地图上的landmark)，那么机器人在AB点观测到消防栓的相对位置，就对机器人在A点和B点的位姿产生了约束，进一步，AB两点之间也产生了约束。

Graph SLAM就是在机器人运动的过程中构建出若干点(pose)和边(constraint)组成的图，再从全图的角度进行优化。Graph SLAM 需考虑两个问题。一是如何通过传感器数据识别约束，即数据关联问题。该问题的解决方法称之为 SLAM 前端，其直接处理传感器数据。二是如何校正机器人的位姿以便在给定约束的条件下获得一致的环境地图。这部分的方法称为优化或者SLAM 后端。

#### 3.1 信息矩阵和向量
贯穿Graph SLAM中的一个非常重要的概念就是信息矩阵(information matrix)和信息向量(information vector)，通常用 Ω 和 ξ 来表示。

在SLAM中常常用高斯分布表示传感器噪声分布等，是SLAM算法重要的组成元素。通常情况下，习惯用µ（均值）和Σ（协方差）来描述一个高斯分布。而信息矩阵和信息向量，其实是另一组描述高斯分布的参数，叫做canonical parameterization(“典范参数”)，关系如下：
![](http://image.huawei.com/tiny-lts/v1/images/acad6d9e0efbf5a113dec248b9708409_161x91.png@900-0-90-f.png)

在使用典范参数表示negative log likelihood时形式更加优美：
![](http://image.huawei.com/tiny-lts/v1/images/db82484ed9bc78477488cd09b73c39a8_277x26.png@900-0-90-f.png)
,这也是Graph SLAM中引入信息矩阵和信息向量的原因。

#### 3.2   建立信息矩阵
信息矩阵反应的是某条边在优化中的权重，实际中，如果觉得优化结果不太好，可以根据实际情况去调整这些权重以取得更好的效果。每种边都有权重，可按照边的类型来构建对应的信息矩阵。

![](http://image.huawei.com/tiny-lts/v1/images/4b422f5febe7fbdbf599a292d692377e_781x349.png@900-0-90-f.png)

三张图的右侧是信息矩阵，白色格点表示该处数值为0，黑色表示正值；图中的xi表示机器人所在的位姿，例如可用(x,y,θ)来描述二维位姿；mj表示在某个位置xi所观测到的路标(landmark)，可以通过路标和xi的相对距离和角度来描述。

相邻两个xi之间的移动可以通过运动模型和里程计来产生约束，而xi和mj之间则通过观测模型来产生约束。图A->B->C清晰的描述了这一过程。这些所谓的约束其实反应了运动模型/观测模型的不确定性，简单地说，如果某个传感器的误差越大，那么信息矩阵中对应的约束值就越小；反之亦然。

随地图的扩大和landmark的增多，信息矩阵中mj项会迅速增加，由此带来巨大运算量，使得Graph SLAM不具备良好的实时性和运算效率。研究者发现，构建出来的Graph其实非常稀疏，因为每个pose xi只能观察到其附近有限的路标mj，并与它们产生约束；而其它大量的mj与当前的pose却没有发生联系。利用稀疏代数的相关技巧，可以将mj从信息矩阵中消去，进而减少了大量的运算。

下图是variable elimination算法的示意，每消除一个mj，能够观测到mj的所有{xi}之间的约束都会被增强。例如m3的消除就增强了<x2, x3>,<x3, x4>之间的约束，并在<x2, x4>之间增加了一条边(约束)。经过消除算法之后的信息矩阵就只剩下所有pose之间的约束了。

![](http://image.huawei.com/tiny-lts/v1/images/3e7f4cfda6163f491944f078cce24867_783x348.png@900-0-90-f.png)

#### 3.3 算法的基本流程
Graph SLAM算法的基本流程如下：

1). 选择节点和边的类型，确定参数化形式，加入图中；

2). 求节点pose初始的估计值µ0:t (可以用航位推算法等)，开始迭代；

3). 对信息矩阵降维(variable elimination algorithm)，求优化的梯度方向；

4). 继续迭代。如迭代结束，则返回对pose和map的优化结果

1)被称为前端，也是设计Graph SLAM算法的难点——如何根据不同的传感器和任务场景设计Graph中边（constraint）的参数化形式。2)、3)、4)被称为Graph SLAM的后端，可使用现成的工具(如g2o, ceres等)。

#### 3.4   优化-关键约束
提供了一种通用的把gps，odometry，ground plane等加入图优化的思路。其中ground plane约束的加入，大大提高了slam的轨迹精度。
![](http://image.huawei.com/tiny-lts/v1/images/7e65ab28da02ee0cf6ac0cdb413a8190_666x663.png@900-0-90-f.png)

### 4. 后端优化
后端优化是整个系统的核心，建图的核心就是综合所有信息进行优化，处理SLAM过程中噪声的问题。任何传感器都有噪声，前端给后端提供待优化的数据，以及这些数据的初始值，而后端负责整体的优化过程，它往往面对的只有数据，不必关系这些数据来自哪里，主要涉及滤波和非线性优化算法。

任何形式的里程计包括imu、编码器、激光、视觉等均存在累计误差致命缺点，会导致随着SLAM距离的增加，其与真实位置偏差逐渐变大。因此具有闭环能力的后端处理十分必要。

算法中的后端优化实现代码： hdl_graph_slam_nodelet.cpp

#### 4.1  融合约束结构
HDL后端调用g2o中图优化思想进行后端处理，关键帧作为顶点，而平面，gps，imu，闭环等信息作为边进行约束。gps可提供全局静态位置，但是误差较大，而imu可提供高速的推算功能，但存在累计误差，通过后端优化，将其进行融合约束。整个后端主要包括3个进程进行处理：顶点和边添加、定时优化、定时发布地图；
![](http://image.huawei.com/tiny-lts/v1/images/f04413fef2dc1b8fdb594f19b3b4a695_876x530.png@900-0-90-f.png)

后端优化是将所有的pose相连，以第一个pose为初始值，整体进行迭代优化，edge为二元边。

**pose graph(位姿)简图**

后端优化，即优化目标函数。对应的位姿图(pose graph)含有由激光里程计构成的运动约束，地面检测和回环检测构成的测量约束，整个pose graph可以用下面简图表示：

![](http://image.huawei.com/tiny-lts/v1/images/7244b905b08d9303887262116c0f984a_526x355.jpg@900-0-90-f.jpg)

#### 4.2  关键帧顶点
将帧间匹配中激光雷达的pose作为顶点，在hdl中引入了关键帧概念，实际上在整个SLAM过程中，选取一定距离或角度间隔的点云数据作为关键帧，其中将关键帧中的pose作为顶点。其中优化前的顶点由关键帧(激光里程计)提供的pose作为顶点初始值；每次添加顶点时，同时记录整个slam轨迹的累加值，用于后续闭环条件判断。

#### 4.3 传入位姿和点云
![](http://image.huawei.com/tiny-lts/v1/images/edd56aebbfa294f32366f8fef28d37f8_975x769.png@900-0-90-f.png)

是否关键帧的校验：（参考代码）

/hdl_graph_slam/include/hdl_graph_slam/keyframe_updater.hpp

![](http://image.huawei.com/tiny-lts/v1/images/cd7a12f4a29677406865b336441fa117_669x182.jpg@900-0-90-f.jpg)

#### 4.4   Graph约束边
##### 4.4.1 运动约束
添加由激光里程计构成的运动约束。关键帧的pose作为顶点，相邻顶点的里程计位置变换矩阵作为边。

![](http://image.huawei.com/tiny-lts/v1/images/dc479f66ef57cde586fb93aced484020_969x606.png@900-0-90-f.png)

![](http://image.huawei.com/tiny-lts/v1/images/d8d337d274f38d07e2f89d0f065d4547_969x648.png@900-0-90-f.png)

##### 4.4.2 闭环约束
参考源代码路径：include\hdl_graph_slam\loop_detector.hpp
###### 4.4.2.1 闭环检测
robot在slam中检测到曾经经历过的环境，称为闭环检测。闭环检测与前端里程计基本类似，采用仅是当前帧与历史帧中的一帧进行点云匹配，从而判断是否闭环，其准确性和可靠性可通过scan-to-map进行提升。由于所有顶点存储均为关键帧，因此闭环也就是新的关键帧是否出现在历史关键帧附近。

HDL采用较为暴力的方法，将当前帧与历史帧每一帧逐一进行匹配，确定是否满足闭环条件，然后再将满足条件的每个历史帧再一一进行点云匹配，最后找到最佳匹配点云帧即认为闭环，效率较低。可采用lego-loam中的闭环进行优化，采用KDtree进行位置搜索，找到与当前帧最近的可能闭环点，然后取出最近关键帧前后多帧点云构成submap，然后进行一次点云匹配。其中满足闭环条件的最近关键帧附近显然满足hdl中满足闭环条件的候选条件，构建局部地图后显然更加可靠和稳定，且仅需一次点云匹配即可。

**闭环检测初步条件包括：**

两次闭环需要有一定距离间隔，因为短距离内闭环无意义（即防止同一位置附近自己闭环）。闭环检测的两个keyframe需要经历一定轨迹累计间隔，防止假闭环出现（即相当于里程计功能）；两个keyframe 在优化前距离一定范围内。

在调用闭环检测功能时，实际调用的函数是detect

![](http://image.huawei.com/tiny-lts/v1/images/db1c496f0e82441a2aaeb4e773d5dee3_970x443.png@900-0-90-f.png)

**闭环检测的步骤：**

1）检测满足条件的关键帧(寻找潜在闭环帧)，以进行下一步匹配

2）对检测到的关键帧进行匹配，找出匹配精度最好的一帧，并返回匹配关系

3）把第2）步找到的匹配关系放入容器中

4）循环上面三步，直到所有新的关键帧遍历完

![](http://image.huawei.com/tiny-lts/v1/images/c1c5b6ea843ca98d993f825c71776f92_1124x715.png@900-0-90-f.png)

如满足以上条件，则可作为闭环可能匹配的候选。将所有候选keyFrame对进行一一点云匹配；如果匹配相关度高于一定值时，则检测到闭环。并获取新的关键帧与变换历史帧的相对位姿。

**潜在闭环匹配：**

上一步索引出来的关键帧还要通过这一步的检验，才能够算是真正成功实现了闭环匹配。这一步在函数matching中。

![](http://image.huawei.com/tiny-lts/v1/images/0f5eb93780760df7ea610a670b92d832_969x487.png@900-0-90-f.png)

**补充说明：**

1) 匹配点云时仍只对两帧进行匹配，如果把关键帧的前后几帧都找出来，拼成一个局部地图再去匹配应该精度会更高，毕竟两帧点云匹配太稀疏。

2) 对所有满足条件的关键帧都做一次点云匹配太费时，帧间匹配还是挺耗时间的，当地图稍大一些，有几千个关键帧时，一个闭环检测要匹配多少次，这肯定不行。或许找一个距离最近的关键帧，然后以它为中心取前后几帧构建一个局部地图去匹配就行，精度满足就建一条闭环边，不满足就不要了，等下一次。

##### 4.4.2.2添加闭环边

闭环边约束与激光里程计边约束完全一样；
![](http://image.huawei.com/tiny-lts/v1/images/5cd099b2a06f66ce3351bc58864331c9_967x253.png@900-0-90-f.png)

##### 4.4.2.3关键帧说明
在做闭环时，包含两类keyframe，已经检测过的keyframe和未检测过闭环的新添加的new_keyframe。满足条件的两个关键帧，则是将new_keyframe中的每一个和keyframe中每一个进行判断和点云匹配，较为暴力。

#### 4.4.3 平面约束
添加由地面检测对应的测量约束。平面的方程是ax + by + cz + d=0，所以一个平面(plane)可以由四个参数描述。平面在这里既用来构造顶点，又参与构造边。

##### 4.4.3.1 基本说明
主要目的是约束SLAM过程中高度上产生的误差。由于实际环境中，无论室内还是室外，采用的lidar传感器主要为水平安装，因此在高度上约束较少，在室外可采用gps提供的高度信息进行约束，如果没有gps信号，则高度误差较大。

如果提前已知robot运行的为平面环境，则可根据平面信息进行约束，即锁定高度信息，防止地图倾斜或闭环上上下重影。

注：非平面环境不可使用，否则会加大误差，因为平面约束就是认为slam所行走的轨迹均在一个平面上。

检测平面，主要涉及floor_detection_nodelet.cpp里的平面检测。在hdl整体介绍中已经说明，最后通过采用ransac 拟合计算出平面系数。通过以下收集函数进行收集对应关键帧的平面信息。

![](http://image.huawei.com/tiny-lts/v1/images/6848cf2189c30f4aa633c31905e40fa6_780x250.jpg@900-0-90-f.jpg)

##### 4.4.3.2 添加约束边

通过计算得到的平面系数来添加约束，主要对检测到的平面的顶点，添加平面约束，同时引入信息矩阵。信息矩阵是3\*3的，应该只对XYZ起到约束作用。
![](http://image.huawei.com/tiny-lts/v1/images/aa9b1f1a32e5046eccced1fe78e445b3_970x977.png@900-0-90-f.png)

![](http://image.huawei.com/tiny-lts/v1/images/aa9b1f1a32e5046eccced1fe78e445b3_970x977.png@900-0-90-f.png)

![](http://image.huawei.com/tiny-lts/v1/images/a814e3a37437b62fbf6c11f8c87f6230_1077x384.png@900-0-90-f.png)

#### 4.4.4  IMU约束

使用了imu的角度和加速度信息，角度自然就是给位姿做角度观测（四元数形式），而加速度，则是以重力作为基准方向，计算imu检测的加速度矢量和重力的不重合度，并作为误差项进行优化。

![](http://image.huawei.com/tiny-lts/v1/images/c7e47f95a66c40b18c456dbad2670359_969x276.png@900-0-90-f.png)

![](http://image.huawei.com/tiny-lts/v1/images/e27c5cd7c178abfd1220cc2d792a3fc2_969x662.png@900-0-90-f.png)

##### 4.4.4.1 添加姿态约束

![](http://image.huawei.com/tiny-lts/v1/images/a10d006fa4fcb6dc69edd31bcbb13dd7_969x145.png@900-0-90-f.png)

姿态约束是否准确? 因为时间戳并没有严格对齐，即没有进行根据时间进行插值。

##### 4.4.4.2 重力加速度约束

![](http://image.huawei.com/tiny-lts/v1/images/cafa884320452dc92d409f37ac2da644_969x151.png@900-0-90-f.png)

注：IMU消息接收函数：imu_callback

#### 4.4.5  GPS约束
GPS消息接收函数：nmea_callback，navsat_callback，gps_callback

由于直接提供的是全局绝对位置，仅需将gps经纬度转换为大地坐标系，并且将初始位置进行坐标初始处理即可。方法同样将每帧gps信息放入缓存队列中，然后根据时间戳找到对应的keyframe，即顶点进行约束。

具体来说，找到当前关键帧时间戳附近最近的gps frame，记录第一帧gps，转化到utm坐标系下，后面的数据都是按照相对第一帧来处理。keyframe和gps位姿直接相减得到先验约束加入到graph中。

![](http://image.huawei.com/tiny-lts/v1/images/d22e095d5254ca9e3b7bd549485c6c9b_969x769.png@900-0-90-f.png)

![](http://image.huawei.com/tiny-lts/v1/images/1c2124984d52ac88a65e915de39e33b7_970x597.png@900-0-90-f.png)

注：UTM（Universal Transverse Mercator Grid System，通用横墨卡托格网系统）坐标是一种平面直角坐标，这种坐标格网系统及其所依据的投影已经广泛用于地形图，作为卫星影像和自然资源数据库的参考格网以及要求精确定位的其他应用。

##### 4.4.5.1 GPS信息矩阵

GPS 坐标转换：主要是GPS地球坐标系->转换到大地坐标系（UTM)-> UTM坐标系再转换成局部坐标。只用gps的 X和Y信息，z 方向的信息不用，主要原因是不准，这就涉及设置信息矩阵，注意信息矩阵的类型：graph_slam->add_se3_prior_xy_edge((\*seek)->node, xyz.head<2>(), information_matrix);

![](http://image.huawei.com/tiny-lts/v1/images/c32fcb9b7414d194a052eae646c06738_980x298.png@900-0-90-f.png)

说明：在g2o中添加约束边时，所有的边后面都跟了添加鲁棒核的操作。如此，系统会对一些异常数据更鲁棒。

### 4.5   定时优化

之所以前面所经所有传感器信息在优化前均需要放入buffer队列中，是因为hdl采用后端优化机制是定时进行调用优化。因此在每次优化间隔间，不同传感器可独立进行收集数据缓存，在优化时根据最新的keyframe的时间戳去匹配所有其他传感器数据信息，添加对应边约束，并剔除过时的传感器数据。

![](http://image.huawei.com/tiny-lts/v1/images/cd2bea0564c4136cc2b1bfa10858756d_969x534.png@900-0-90-f.png)

![](http://image.huawei.com/tiny-lts/v1/images/b8e37e9714587ebe316dd139240d86f7_969x528.png@900-0-90-f.png)

HDL将后端优化开启一个独立线程进行，实际上优化处理较为耗时，而且如果没有新的闭环条件存在，仅存在imu和gps等其他信息进行多次优化意义并不是特别大。因此闭环检测应当进行实时判断，而当出现一定闭环时再进行一次后端优化，从而更新整个关键帧信息，可节省大量时间。

#### 4.5.1计算信息矩阵

calc_information_matrix(loop->key1->cloud, loop->key2->cloud, relpose)

relpose 表示闭环帧之间的位姿约束

![](http://image.huawei.com/tiny-lts/v1/images/0154c2439e8d7befe6c09b623df792b3_1033x636.png@900-0-90-f.png)

#### 4.5.2 关键帧发布

从KeyFrame类中取出位姿（优化之后的位姿）和点云，供地图拼接使用，这样得到的就是位姿经过优化的地图。

![](http://image.huawei.com/tiny-lts/v1/images/f4985f761b6f51acb66d8fd5912b5d08_969x608.png@900-0-90-f.png)

### 5. 地图发布
生成全局地图并定时发送，即把所有关键帧拼一起，得到全局点云地图。可在一个定时函数里完成发送到rviz上。

#### 5.1  生成地图
使用简化版关键帧进行拼接

代码：src/hdl_graph_slam/map_cloud_generator.cpp

![](http://image.huawei.com/tiny-lts/v1/images/642a16448d9b3191a425c664b94611ec_1124x621.png@900-0-90-f.png)

##### 5.1.1 以pcd形式输出

![](http://image.huawei.com/tiny-lts/v1/images/261e28900e1e5bdb13d29f817b62ff18_1124x746.png@900-0-90-f.png)

#### 5.2 定时概率图发布
由于前端里程计构建了关键帧信息，因此地图发布仅需要进行简单的点云拼接即可。如果未进行后端优化，其表明历史关键帧信息队列并没有发生变化，仅是新添加了新的关键帧。因此可采用增量式更新地图即可。当成功出现优化结果时，再进行一次全体更新。目前hdl做法较为简单，定时将所有关键帧包含的点云信息，重新进行拼接，耗时较多。

![](http://image.huawei.com/tiny-lts/v1/images/2b27b5af89ba3a7d55537b1d3b504423_969x660.png@900-0-90-f.png)

#### 5.3 地图重建-样例

![](http://image.huawei.com/tiny-lts/v1/images/582ee7753e9e30a049e7c800c9897c97_710x492.png@900-0-90-f.png)

#### 5.4 RViz-可视化
在rviz中显示顶点和边，如果运行程序，会看到rviz中把概率图可视化了。

![](http://image.huawei.com/tiny-lts/v1/images/c88d38bc32d7b4df286d29589793546d_1366x768.png@900-0-90-f.png)

**最后**

Graph SLAM将地图创建视为最优化问题，通过定义目标函数和约束条件的形式进行定义，采用数学规划方法进行求解。在采集完所有传感器数据后，在建立好所有约束条件的前提下进行离线位姿估计和地图创建,是一种基于闭环消除定位误差的地图创建方法。在通过航迹推测所估计的位姿误差较大的情况下，只能通过辅助方法建立约束条件，即在建立正确闭环的前提下，再通过优化方法校正机器人位姿和创建环境地图。

hdl_graph_slam的运行效果不错。但在在一些较大的场景中，有时会出现回环检测失败的情况，从而导致建图结果出现不一致。这种情况下，可在建图过程中多增加一些环来进行修正。另外，算法采用的地面检测算法假设了全局一致地面的存在，如果可以对不同的地面参数进行数据关联，可能效果会更好一点。另外，帧间匹配和闭环检测的精度还有待提升。

整体来说，hdl_graph_slam系统的代码模块化做的较好，思路清晰，对多信息源的融合也做的很好，即使再多增加几个传感器也易扩展实现。

参考: https://github.com/koide3/hdl_graph_slam

