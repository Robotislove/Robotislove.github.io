---
layout: post
title: 多源融合SLAM的现状与挑战综述学习笔记
date: 2022-05-23
author: lau
tags: [SLAM, Archive]
comments: true
toc: false
pinned: false
---
同时定位与地图构建（Simultaneous Localization and Mapping, SLAM）现状与挑战。

<!-- more -->

同时定位与地图构建（Simultaneous Localization and Mapping, SLAM）技术自Smith（1986）提出以来，经过三十五年的发展，已取得众多优秀的研究成果。它所关注的问题是载有传感器的机器人如何在未知环境中定位并构建出环境地图，是机器人感知自身状态和外部环境的关键技术，在现实生活中实现了大规模的应用，如虚拟现实、增强现实、混合现实、自动驾驶、服务机器人等，且在向工业界稳定过渡。

SLAM的发展经历了三个阶段：第一阶段为早年的“经典”时期，主要完成了SLAM的概率解释，如基于扩展卡尔曼滤波器、粒子滤波器和极大似然估计的方法；第二阶段为“算法分析”时期，主要研究SLAM的基本性质，如收敛性、一致性、可观测性和稀疏性；第三阶段为当前的“鲁棒感知”时期，主要解决复杂环境下的适应性问题。面对日趋复杂的应用场景，设计SLAM算法应统筹兼顾，有针对性地对算力、精度、鲁棒性等进行取舍。

![image](https://user-images.githubusercontent.com/89834571/169831938-93114ec4-0c2a-4ec8-ad61-b32a2942d2d2.png)

![image](https://user-images.githubusercontent.com/89834571/169832022-c01e95b0-0005-4eb7-a40f-7fcb2a00a052.png)

## 一．多传感器融合

用于估计机器人状态的多传感器融合系统分为感知自身运动信息的本体传感器和感知外部环境的外感受型传感器。本体传感器包括编码器、磁力计、轮速计、惯性测量单元等，其中惯性测量单元用得最为广泛。外感受型传感器有相机、激光雷达、毫米波雷达等，直接对周边环境进行观测。
针对用单传感器构建SLAM系统的局限性，研究者们首先提出将多种传感器组合，利用不同传感器的优势克服其他传感器的缺陷来提高定位建图算法在不同场景中的适用性和对位姿估计的准确性，涌现出视觉惯导、激光惯导、激光视觉惯导等多传感器融合系统。

![image](https://user-images.githubusercontent.com/89834571/169832113-d89e7e7b-4d5c-47c8-b97a-e549c562b152.png)

![image](https://user-images.githubusercontent.com/89834571/169832169-a6811a16-70d8-441b-baa6-9709244965ea.png)

![image](https://user-images.githubusercontent.com/89834571/169832187-690dc4e9-8a4e-410f-89b9-8ddd425ea0e0.png)

## 二．多特征基元融合

针对复杂环境，基于单特征基元的SLAM在精度和鲁棒性方面都有所欠缺，容易受到光照、运动、纹理等因素的影响，不确定性较高。而对于相机图像，不仅可以从中提取特征点，**还可以获得线段特征、平面特征、像素灰度等信息特征**；对于激光雷达点云，可以将其分为**线特征点、面特征点和有正态分布特性的体素特征**。在视觉SLAM系统中，**特征点法通过提取和匹配相邻图像（关键）帧的特征点**估计对应的帧间相机运动，包括特征检测、匹配、运动估计和优化等步骤；直接法则直接使用像素强度信息，通过最小化光度误差来实现运动估计。利用特征法和直接法的优点，可以构建出两种方法结合的SLAM系统。

![image](https://user-images.githubusercontent.com/89834571/169832263-a18ca670-cda1-4127-97c0-c8ee882da737.png)

低维度的点特征在视觉SLAM和激光SLAM中是被研究者们普遍使用的，然而在走廊、礼堂和地下车库等环境中，由于无法有效提取足够的特征点，基于点特征的方法就不再适用，但是上述环境中的直线、曲线、平面、曲面、立方体等高维几何特征却十分丰富，这为研究者们解决问题提供了思路。

![image](https://user-images.githubusercontent.com/89834571/169832338-c5a4de85-85e6-47e1-be38-b0908d2a645e.png)

## 三．多维度信息融合

传统SLAM系统发展几十年至今，理论方面逐渐趋于成熟。其中的闭环检测通常依赖于从传感器原始数据中提取的几何基元特征，如特征点、线、面等，通过对几何特征编码的场景特征向量进行匹配实现闭环检测。但在恶劣环境下，几何特征提取极其不稳定，难以保证准确的闭环检测。而语义信息，是一种长期稳定的特征，不易受环境因素的影响，但只用语义信息无法实现精确定位。因此，可以尝试将语义信息融合到传统SLAM系统中，构建长期稳定的定位系统。近年来，基于数据驱动的深度学习的方法逐渐兴起，通过对大量数据的学习可以得到比手工涉及更加精确的模型，故而传统方法与学习方法有效融合可以提升定位系统的精度和鲁棒性。

![image](https://user-images.githubusercontent.com/89834571/169832427-aa21fcb7-2364-4f4a-b104-f8acd1442d7a.png)

里程计用两帧或多帧传感器数据来估计载体的相对位姿变化，以初始状态为基础推算出全局姿态，其核心问题就是如何从各种传感器测量中准确地估计出平移和旋转变换。当前的深度学习方法在视觉里程计、惯性里程计、视觉惯性里程计、激光里程计等应用中已经实现端到端的方案。基于监督学习的方法大多使用卷积神经网络和递归神经网络的组合方式实现视觉里程计的端到端的学习,卷积神经网络完成成对图像的视觉特征提取，递归神经网络则用来传递特征并对其时间相关性建模。

![image](https://user-images.githubusercontent.com/89834571/169832486-dfbd9308-f10a-439b-a8d6-96e129f1bbf5.png)

SLAM算法一般部署在特定环境中的特定机器人平台上，环境和机器人平台自身的物理信息可以为二者的状态估计提供有效约束。利用物理信息辅助SLAM任务中位姿估计主要有两类方法：一类方法直接使用传感器测量相应的物理量，如水压、气压、物理接触等；另一类方法在没有直接的传感器测量情况下，从背景知识出发间接制定约束条件，如推进力和地形等。

## 四．问题与挑战

尽管多源融合在过去几十年中取得了重大进展，但仍有许多挑战需要应对。

（1）合理融合多传感器的测量往往能提升系统性能，但在融合之前需要标定不同传感器之间的相对位姿变换与时间戳，一般会选择离线标定，然而在使用过程中，传感器间的外参由于机械变形及外界物理环境的变化，很有可能发生改变。虽然有在估计器中进行在线外参标定的解决方案，但其容易受到退化运动的影响而失效。故而需要开发鲁棒适应性强的实时在线多传感器外参标定、时间同步方法。

（2）在具有挑战性的动态复杂环境条件下（如变化的光照，剧烈运动，开阔场地，缺乏纹理的走廊等），需要长期稳健安全的算法来支撑，应该具备有效表示、实时检测跟踪运动物体的能力，将低维几何特征与高维几何特征相结合，在此过程中，通用鲁棒的几何特征提取、参数化、数据关联、耦合方法显得尤为重要。

（3）在融合几何信息和语义信息、传统方法与学习方法、外界自身物理信息等多维度信息时，大量的数据处理将非常耗时，而且对计算性能有一定要求，如何构建轻量级、紧凑部署、快速响应的SLAM系统仍是一个挑战。此外，能否构建出统一的、适应性极强的SLAM框架来应对不断出现的新传感器、新表示方法、新学习方法，是一个值得探究的问题。

# 另一篇 

SLAM包含两个主要任务，定位和建图。这是移动机器人自主完成作业任务需要解决的基本问题，特别是在未知环境的情况下，移动机器人既要确定自身在环境中的位姿，又要根据确定的位姿来创建所处环境的地图，这是一个相辅相成、不断迭代的过程。因此，SLAM问题是一个复杂的耦合问题，也可以被看作是先有鸡还是鸡蛋的问题[1]。

SLAM问题的概念最初是在1986年在加利福尼亚州旧金山举行的IEEE机器人和自动化会议上提出来的[2-3]。1987年，Prter Chesseman首次提出使用EKF(扩展卡尔曼滤波器)来估计机器人的位置和姿态[4]。1995年，Hugh Durrant-Whyte 等人在国际机器人研讨会上对 SLAM 问题的理论框架进行阐述，验证了 SLAM 问题的收敛性[5]。1986-2004年，由于概率方法的广泛应用，SLAM 问题的研究得到了快速的发展，这一时期也被称作SLAM问题的“经典时期”(classic age)，主要研究方法包括扩展卡尔曼滤波(Extended Kalman Filters, EKF)、粒子滤波(Rao-Blackwellized)和最大似然估计等[6]。这些方法遇到的最大瓶颈是计算的复杂度，受限于当时的计算水平，难以满足构建大规模地图的要求。2004-2015年，SLAM 问题的研究进入“算法分析时期”(algorithm-analysis age)，Dissanayake等人从状态的可观测性、状态估计的收敛性、一致性和算法计算效率等角度对这一时期的部分工作做了综述[7]，很好的描述了SLAM 基本特性研究的进展。同时期，Cesar等人对SLAM 的研究做了系统性的综述，包括算法的鲁棒性、应用的可扩展性、地图表示形式（度量地图和语义地图）等问题，并且分别针对这些方面的研究提出了一些有待解决的问题[8]。2015年-至今，是“鲁棒性-预测性时代”(Robust-Predictive age)，主要研究鲁棒性、高级别的场景理解，计算资源优化，任务驱动的环境感知。

本文的组织结构如下:第一节将阐述激光SLAM，包括激光传感器、开源激光SLAM系统以及激光SLAM未来的挑战。第二节重点介绍了视觉SLAM，包括摄像机传感器、开源视觉SLAM系统、视觉SLAM未来的发展以及深度学习在视觉SLAM中的研究进展。第三节将展示激光和视觉融合的SLAM。最后，本文讨论了SLAM未来的研究方向，以及对SLAM的发展作出展望。

**1 激光SLAM**

激光SLAM 所需要的传感器一般有激光雷达（Lidar）、惯性测量单元（IMU）、里程计（Odometry）。通常室内采用二维激光雷达，室外采用三维激光雷达，里程计采用轮式里程计。由于IMU 具有较高的角速度测量精度，里程计具有较高的局部位置测量精度，一般用IMU 计算角度信息，里程计计算位置信息，再配合激光雷达进行同时定位与建图。

**1.1 激光雷达传感器**

激光雷达传感器可分为二维激光雷达和三维激光雷达，它们由激光雷达光束的数量来定义。就生产工艺而言，激光雷达也可分为机械激光雷达、微机电等混合固态激光雷达和固态激光雷达。固态激光雷达主要应用相控阵和Flash技术。在自动驾驶快速发展的趋势下，小型化和轻型化的固态激光雷达将逐步占据市场，并满足大多数应用。

目前生产激光雷达知名公司有Velodyne，其主要产品是机械激光雷达，有VLP-16, HDL-32E和HDL-64E三种类型的产品。SLAMTEC主要生产低成本的激光雷达，其产品有RPLIDAR A1、A2和R3三种类型。Ouster主要生产机械激光雷达，有16到128个频道不同的类型激光雷达。Quanergy发布了世界上第一个的固态激光雷达—S3。S3是一种微型固态激光雷达。

**1.2 激光SLAM系统**

当前，激光SLAM 框架一般分为前端扫描匹配、后端优化、闭环检测、地图构建四个关键模块。前端扫描匹配是激光SLAM 的核心步骤，工作内容是已知前一帧位姿并利用相邻帧之间的关系估计当前帧的位姿；前端扫描匹配能给出短时间内的位姿和地图，但由于不可避免的误差累积，后端优化正是当长时间增量式扫描匹配后优化里程计及地图信息；闭环检测负责通过检测闭环而减少全局地图的漂移现象，以便生成全局一致性地图；地图构建模块负责生成和维护全局地图。

基于激光的SLAM开源方法有Gmapping、HectorSlam、LagoSLAM、Cartographer、Loam、Lego-Loam等。Gmapping是基于粒子滤波的方法，增加了扫描匹配方法来估计机器人的位置[9]。HectorSlam利用扫描匹配技术和惯性传感器(Inertial Measurement Unit，IMU)进行同时定位与建图[10]。LagoSLAM是基于图优化的SLAM，利用非线性使得非凸成本的最小化[11]。Cartographer是谷歌发布的一个SLAM算法，它采用子映射和回环检测来获得更好的性能[12]。Loam是一种使用三维激光雷达进行同时定位建图的方法[13]。Lego-Loam是将VLP-16激光雷达的点云数据和IMU数据作为输入,实时输出姿态估计，并进行全局优化和回环检测[14]。

**1.3挑战与未来**

**1.3.1成本与适应性**

激光雷达的优点是它能提供三维信息，不受光照变化的影响。此外，视角相对较大，可以达到360度。但是激光雷达的技术门槛很高，导致开发周期长，成本高。未来，小型化、成本合理、固态化以及实现高可靠性和适应性是激光雷达的发展趋势。

**1.3.2 低纹理和动态环境**

大部分SLAM系统只能在固定的环境中工作，但实际环境是在不断变化的。此外，走廊、管道等低纹理环境也会给激光SLAM带来很大困难。利用惯性测量单元可以解决上述问题[15]。此外，将时间维度结合到绘图过程中，以使机器人在动态环境中能够具有精确的地图[16]。如何使激光SLAM对低纹理和动态环境更加鲁棒，以及如何保持地图的精确更新是未来需要解决的问题。

**2 视觉SLAM**

随着中央处理器和图形处理器的发展，图形处理能力越来越强。相机传感器越来越便宜，越来越轻，同时也越来越通用。在过去的十年里，视觉SLAM发展迅速。相比于激光SLAM，使用相机的视觉SLAM系统具有更便宜、更轻巧等特性。现在，视觉SLAM系统可以在微型计算机和嵌入式设备上运行[17]，甚至在移动设备上也能使用，比如智能手机[18]。

**2.1 视觉传感器**

视觉SLAM最常用的传感器是相机。具体来说，摄像机可以分为单目相机、立体相机、RGB-D相机、事件相机等。单目相机无法获得真正的深度，真实的轨迹与地图有一定的比例，这称为尺度模糊[19]。基于单目相机的SLAM必须初始化，并存在漂移问题。立体相机是两个单目相机的组合，两个单目摄像机之间的基线距离是已知的，所以深度可以基于校准、校正、匹配和计算得到。深度相机通过立体、结构光和飞行时间等技术可以直接输出像素深度。结构光是红外激光器向物体表面发射一些具有结构特征的图案。然后红外照相机将收集由于表面深度不同而引起的图案变化。飞行时间将测量激光飞行的时间来计算距离。事件相机不是以固定的速率捕捉图像，而是异步测量每个像素的亮度变化[20]。事件摄像机具有很高的动态性范围、高时间分辨率、低功耗等特性，并且不会受到运动模糊的影响。因此，事件摄像机在高速和高动态范围内的性能优于传统摄像机[21]。

**2.2 视觉SLAM系统**

自2002 年起，视觉SLAM 算法的研究开始引起关注，但重大的进展是从2007 年Klein 等人[22]提出的并行跟踪与建图( parallel tracking and mapping，PTAM) 算法开始的。他们提出一个可以将地图创建和位姿估计过程放在两个并发线程中运行的非滤波器算法框架，并取得了实时运行效果。其后，本领域的大部分研究成果都延续了PTAM 框架中的思想。ORB-SLAM使用三个线程，包括跟踪、基于对极约束的局部优化和基于姿态图的全局优化，并且支持单目相机、立体相机和RGB-D相机[23]。Pro-SLAM是一个轻量级视觉SLAM系统，易于理解。它是一种基于特征跟踪的方法，可以有效地匹配一个或多个视频序列之间的特征点对应关系[24]。LSD-SLAM提出了一种新的基于李代数和直接法的同时定位与建图方法，该方法支持立体相机[25]。RGBD-SLAM是基于深度相机的，可以在没有其他传感器的帮助下重建三维场景地图[26]。EVO是一种基于事件的视觉里程计算法，该算法不受运动的影响，在强烈光照变化的动态场景中，可以良好运行[27]。

**2.3 挑战与未来**

**2.3.1 鲁棒性和可移植性**

视觉SLAM仍然面临着光照变化、动态环境、快速运动、剧烈旋转和低纹理环境等重要问题。解决思路如下：首先，事件相机每秒能够产生多达一百万个事件，足以在高速和高动态范围内进行非常快速的运动，可以精确的对相机进行姿态估计。其次，使用边缘、平面、表面等特征，可以减少特征的相关性，进行有效地特征跟踪。

视觉SLAM在未来的应用，一个是基于智能手机和无人机等嵌入式平台的SLAM，另一个是三维场景重建和场景理解。在动态的、非结构化的大规模环境中需要平衡实时性和准确性是一个至关重要的问题[28]。

**2.3.2 语义SLAM**

随着深度学习在计算机视觉领域的快速发展，研究者对深度学习在视觉SLAM中的应用有很大的兴趣。视觉SLAM中的特征匹配、闭环检测等模块都可以通过深度学习来获得更优的结果。在视觉SLAM中应用深度学习可以实现目标识别和分割，有助于SLAM系统更好地感知周围环境。语义SLAM也有助于全局优化、回环检测和重定位[29]。传统的同时定位和建图方法依赖于点、线等几何特征来推断环境结构的平面。语义SLAM可以实现大规模场景中高精度同时定位与建图。

**3 激光和视觉融合的SLAM**

激光和视觉传感器在SLAM应用中都有其局限性，基于激光和视觉传感器融合的SLAM方法能够有效的利用各个传感器的优势，弥补传感器在某些特殊环境下的劣势，成为当前研究的热点之一。

**3.1 激光和视觉传感器数据的外部标定**

激光和视觉传感器数据融合的前提条件是不同传感器对同一目标在同一时刻的描述。因此，不同传感器数据之间的自动标定以及不同传感器的数据融合是激光和视觉融合SLAM需要解决的关键问题。

传感器的标定方法可以分为两种，第一种是按照给定的相对变换关系安装传感器；第二种则是根据不同传感器数据之间的约束关系来计算两个传感器之间的相对变换关系。当传感器发生故障后，第一种方法需要重新校正，而且移动机器人运动过程中的振动会使得相对误差逐渐增大，因此更多的采用第二种标定方法，这种标定方法类似于单目相机的内参标定，通过给定的标定板，利用几何约束关系构建坐标转换系数矩阵方程，从而确定相机坐标系和激光雷达坐标系之间的转换关系[30]。而且，针对未标定过或者标定误差较大的单目相机，该方法还可以采用全局优化的方法同步优化相机内部标定和外部标定。Lidar-Camera提出了一种新的管道和实验装置，以找到精确的刚体变换，用于利用3D-3D点对应关系对激光雷达和相机进行外部校准[31]。

**3.2 激光和视觉传感器的融合**

传感器数据融合层次一般分为三种：数据层融合、特征层融合和决策层融合。激光和视觉传感器是异质的，因此数据无法在数据层进行融合，而决策层融合预处理代价高，而且融合的结果相对而言最不准确，因此激光和视觉传感器数据融合主要是特征层的数据融合。

一种较为简单、直观的融合方法是基于估计理论数据融合方法主要包括卡尔曼滤波方法、协方差融合方法、最小二乘法等。这种方法为不同的传感器数据建立状态空间模型，然后对其进行状态估计，从而实现数据融合[32]。另一种是基于推理数据的融合方法，主要包括贝叶斯估计法和Dempster-Shafer(D-S)证据推理法等。贝叶斯估计法属于静态环境信息融合方法，信息描述为概率分布，适应于具有可加高斯噪声的不确定性信息处理。多贝叶斯估计把每个传感器作为贝叶斯估计，将环境中各个物体的关联概率分布结合成联合的后验概率分布函数，通过使联合分布函数的似然函数为最大，提供最终融合值，D-S证据推理法是贝叶斯估计法的扩展方法[33]。V-Loam提出了一个将视觉里程计和激光雷达结合的通用框架，以在线的方法从视觉里程计和基于匹配的激光雷达里程计两个方面入手，同时改进了运动估计和点云配准[34]。

**3.3 挑战与未来**

**3.3.1 数据关联**

SLAM的未来必须集成多个传感器。但不同的传感器有不同的数据类型、时间戳和坐标系表达式，需要统一处理。此外，还应考虑多传感器之间的物理模型建立、状态估计和优化。

**3.3.2 硬件集成**

目前，还没有合适的芯片和硬件来集成SLAM技术，使其成为产品。另一方面，如果传感器的精度由于故障或老化而降低，传感器测量的噪声与模型不匹配。前端传感器应具备处理数据的能力，并能从硬件层向算法层传递高质量的数据。

**3.3.3 抵御风险和故障恢复的能力**

SLAM系统应该是具有抵御风险和故障恢复的能力，这里不是重新定位或回环检测的问题，而是SLAM系统必须有能力应对风险或故障。同时，SLAM系统应该能够在不同的平台上运行，不管平台的计算约束如何。如何平衡准确性、鲁棒性和有限的资源是一个具有挑战性的问题。

**4 发展趋势**

**4.1 大规模复杂环境的适应性和鲁棒性的提高**

大规模复杂环境的适应性和鲁棒性是SLAM 系统实用化的必然要求，其趋势在于从当前的室内或简单室外环境转向大范围的、动态的、复杂的室外环境研究。适应动态环境、抗光照变化、一些低纹理环境下的SLAM 技术、无须初始化的SLAM 技术也正在发展。此外，模式识别和机器学习理论的发展也正在使得闭环检测效果更加精确[35]。

**4.2 提高计算效率并优化精度**

在提高计算效率方面，一些基于嵌入式、低功耗设备上SLAM 技术的研究也逐渐受到技术企业的关注，例如VR 和AR 头盔乃至手机设备都能提供较好的使用体验，同时摆脱对笨重计算设备的依赖[36]。

**4.3 多机器人和多传感器协同SLAM**

在多机器人和多传感器系统SLAM 方面，多机器人协同建图、基于多传感器融合的SLAM 系统、visual-inertial SLAM等研究也逐渐兴起。KIT 和CMU 等大学将自有地面无人车辆采集的数据打包发布供研究者测试和对比各自算法在真实环境中的效果，这些数据不仅包括单目相机的数据，还有如惯性导航、GPS、双目视觉、激光雷达数据等数据[37]。因此，未来基于多传感器融合的SLAM 算法将是一个热门的研究方向。

**5 总结**

本文分析了三种类型的同时定位与地图创建方法的各个基本组件，并对比了近年来重要算法的设计思路。综合近年来的重要成果不难发现，同时定位与地图创建算法的发展正在朝着越来越注重准确性、实时运行和具备较强的故障恢复能力等方向发展，这一趋势也为此类技术在各种环境下的实用性奠定了基础。SLAM 技术会从实验室引入生活和工业应用场景中，使得生活环境越来越智能化。

**参考文献**

[1] John J, Hugh F. Simultaneous map buildingand localization for an autonomous mobile robot[J]. IEEE/RSJ International Workshop on Intelligent Robots ,1991, 7(6): 1442–1447.

[2] Durrant H, Bailey T. Simultaneous localization and mapping(slam):Part i[J]. IEEE Robotics & Automation Magazine, 2006, 13(2): 99-110.

[3] Bailey T, Durran H. Simultaneous localization and mapping(slam):Part ii[J]. IEEE Robotics & Automation Magazine, 2006, 13(3): 108-117.

[4] Randall S, Matthew S. Estimating uncertain spatial relationships in robotics[J]. Autonomous robot vehicles, 1987, 13(3): 167–193.

[5] Durrant H, Rye D. Localization of autonomous guided vehicles[J]. Robotics Research.1996, 12(1): 613-625.

[6] Aulinas J, Petillot Y, Salvi Joaquim, et al. The slam problem: A survey [J]. Artificial Intelligence Research and Developmen, 2008, 11(2): 363-371.

[7] Dissanayake G, Huang S, Wang Z, et al. Areview of recent developments in simultaneous localization and mapping[J]. IEEE International Conference on Industrial and Information Systems. 2011,5(2): 477-482.

[8] Candena C, Carlone L, Carrillo H, et al. Past, present, and future of simultaneous localization and mapping：Towards the Robust-Perception Age[J]. IEEE Transactions on Robotics, 2016, 32(6): 1309-1332.

[9] Giorgio G, Cyrill S, Wolfram B, et al. Improved techniques for grid mapping with rao-black wellized particle filters[J]. IEEE transactions on Robotics, 2007,23(1): 34-50.

[10] Michael M, Sebastian Thrun, Daphne K, Ben W, et al. Fast-slam 2.0: An improved particle filtering algorithm for simultaneous localization and mapping that provably converges[J]. 2003, 32(6): 1151–1156.

[11] Luca C, Rosario A. A linear approximation for graph-based simultaneous localization and mapping[J]. Robotics: Science and Systems VII, 2012, 12(5): 41–48.

[12] Wolfgang H, Damon K, Holger R, and Daniel A. Realtime loop closure in 2d lidar slam[J]. IEEE International Conference on Robotics and Automation, 2016, 15(5): 1271–1278.

[13] Ji Z, Sanjiv S. Loam: Lidar odometry and mapping in real-time[J]. Science and Systems, 2014, 15(5): 9-25.

[14] Tixiao S, Brendan E. Lego-loam: Lightweight and ground optimized lidar odometry and mapping on variable terrain[J]. International Conference on Intelligent Robots and Systems, 2018,14(5): 4758–4765.

[15] Zhong W, Yan C, Yue M. IMU-Assisted 2d slam method for low-texture and dynamic environments[J]. Applied Sciences, 2018, 8(12): 25-34.

[16] Aisha W, Michael K, Hordur J. Dynamic pose graph slam: Long-term mapping in low dynamic environments[J]. International Conference on Intelligent Robots and Systems, 2012, 5(5): 1871–1878.

[17] Raul M, Jose M. Orb-slam: a versatile and accurate monocular slam system[J]. IEEE transactions on robotics, 2015, 31(5):1147–1163.[18] Tong Q, Peiliang L, and Shaojie S. Vins-mono: A robust and versatile monocular visual-inertial state estimator[J]. IEEE Transactions on Robotics, 2018, 34(4):1004–1020.

[19] Simon L, Torsten S, Michael B. Get out of my lab: Large-scale, real time visual-inertial localization[J]. In Robotics: Science and Systems, 2015, 18(5): 10-25.

[20] Guillermo G, Tobi D, Garrick O, et al. Event-based vision: A survey[J]. IEEE Transactions on Robotics, 2019, 34(4):1004–1020.

[21] Patrick L, Christoph P, and Tobi D. A 128x128120db 15us latency asynchronous temporal contrast vision sensor[J]. IEEE journal of solid-state circuits, 2008, 43(2):566–576.

[22] Klein G，Murray D． Parallel tracking and mapping for small AR workspaces[J]. Proc of IEEE and ACM International Symposium on Mixed and Augmented Reality. 2007, 10(5): 1-10．

[23] Ethan R, Vincent R, Kurt K, and Gary R. Orb: An efficient alternative to sift or surf[J]. ICCV, 2011,10(5): 2-15.

[24] Guofeng Z, Haomin L, et al. Efficient non-consecutive feature tracking for robust structure-from-motion[J]. IEEE Transactions on Image Processing, 2016, 25(12):5957–5970.

[25] Jakob E, Thomas S, and Daniel C. Lsd-slam: Largescale direct monocular slam[J]. European conference on computer ision, 2014, 12(5): 834–849.

[26] Felix E, Hess J, Daniel C. 3-d mapping with an rgb-d camera[J]. IEEE transactions on robotics, 2014, 30(1):177–187.[27] Henri R, Timo H, Guillermo G. Evo: A geometric approach to event-based 6-dof parallel tracking and mapping in real time[J]. IEEE Robotics and Automation Letters, 2016, 2(2):593–600.

[28] Muhammad S, Gon W. Simultaneous localization and mapping in the epoch of semantics: A survey[J]. International Journal of Control, Automation and Systems, 2019, 17(3): 729–742.

[29] Nikolay A, Sean L, Kostas D. A unifying view of geometry, semantics, and data association in slam[J]. IJCAI, 2018, 12(4): 5204–5208.

[30] Jesse L, Sebastian T. Automatic online calibration of cameras and lasers[J]. Robotics: Science and Systems, 2013,10(5): 2-16.

[31] Dhall A, Chelani, V, Radhakrishnan M. Camera calibration using 3D-3D point correspondences[J]. ArXiv eprints, 2017, 11(4): 24-36.[32] Wen S, Othman K, Rad A, et al. Indoor slam using laser and camera with closed-loop controller for nao hunmanoid robot[J]. Abstract and Applied Analysis,2014, 12(5): 1-8.

[33]Chang H, Lee C, Lu Y. P-SLAM: Simultaneous localization and mapping with environmental-structure prediction[J]. IEEE Transactions on Robotics, 2007, 23(2): 281-293.

[34] Ji Z and Sanjiv S. Visual-lidar odometry and mapping: Low-drift, robust, and fast[J]. IEEE International Conference on Robotics and Automation, 2015, 10(5): 2174–2181.

[35] Stefan M, Georg A, Christian Witt. Visual slam for automated driving: exploring the applications of deep learning[J]. IEEE Conference on Computer Vision and Pattern Recognition , 2018, 10(5): 360–370.

[36] Oscar G, Grasa1 J. EKF Monocular slam 3D modeling, measuring and augmented reality from endoscope image sequences[J]. IEEE International Conference on Robotics and Automation , 2009, 10(1): 174–185.

[37] Urtasun R，Lenz P，Geiger A． Are we ready for autonomous driving? The KITTI vision benchmark suite[C]. IEEE Conference on Computer Vision and Pattern Recognition． Washington DC: IEEE Computer Society，2012: 3354-3361．
