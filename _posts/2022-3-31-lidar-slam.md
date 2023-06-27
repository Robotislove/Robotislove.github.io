---
layout: post
title: Lidar SLAM学习笔记
date: 2022-03-31
author: lau
tags: [SLAM, Archive]
comments: true
toc: false
pinned: false
---

目前开源的激光雷达SLAM学习。

<!-- more -->

## 概述

## 激光雷达SLAM

SLAM问题被认为是真正意义上实现机器人自主移动的关键。其问题可以理解为如下的生活问题。

当你来到一个陌生的环境时，为了迅速熟悉环境并完成自己的任务（比如找饭馆，找旅馆），你应当做以下事情：

a. 用眼睛观察周围地标如建筑、大树、花坛等，并记住他们的特征（特征提取）

b. 在自己的脑海中，根据双目获得的信息，把特征地标在三维地图中重建出来（三维重建）

c. 当自己在行走时，不断获取新的特征地标，并且校正自己头脑中的地图模型（bundle adjustment or EKF）

d. 根据自己前一段时间行走获得的特征地标，确定自己的位置（trajectory）

e. 当无意中走了很长一段路的时候，和脑海中的以往地标进行匹配，看一看是否走回了原路（loop-closure detection）。实际这一步可有可无。

以上五步是同时进行的，因此是simultaneous localization and mapping。

SLAM算法在实现的时候主要要考虑以下4个方面吧：

- 地图表示问题，比如dense和sparse都是它的不同表达方式，这个需要根据实际场景需求去抉择

- 信息感知问题，需要考虑如何全面的感知这个环境，RGBD摄像头FOV通常比较小，但激光雷达比较大

- 数据关联问题，不同的sensor的数据类型、时间戳、坐标系表达方式各有不同，需要统一处理

- 定位与构图问题，就是指怎么实现位姿估计和建模，这里面涉及到很多数学问题，物理模型建立，状态估计和优化

其他的还有回环检测问题，探索问题（exploration），以及绑架问题（kidnapping）。

本文主要学习激光雷达SLAM，激光雷达传感器利用光原理进行工作，激光雷达代表光探测和测距。它们可以探测到300米以内的障碍物，并准确估计它们的位置。在自动驾驶汽车中，这是用于位置估计的最精确的传感器。

激光雷达传感器由两部分组成：**激光发射（顶部）**和**激光接收（底部）**。发射系统的工作原理是利用多层激光束，层数越多，激光雷达就越精确。层数越多，传感器就越大。激光被发射到障碍物并反射，当这些激光击中障碍物时，它们会产生一组点云，传感器与飞行时间（TOF）进行工作，从本质上说，它测量的是每束激光反射回来所需的时间。

当激光雷达的质量和价格非常高时，激光雷达是可以创建丰富的三维环境，并且每秒最多可以发射200万个点。点云表示三维世界激光雷达传感器获得每个撞击点的精确（X，Y，Z）位置。 

激光雷达传感器可以是固态的，也可以是旋转的，固态激光雷达将把检测的重点放在一个位置上，并提供一个覆盖范围（比如FOV为90° ）。在后一种情况下，它将围绕自身旋转，并提供360度旋转。在这种情况下，一般把它放在设备顶上，以提高能见度。

激光雷达很少用作独立传感器。它们通常与相机或雷达结合在一起，这一过程称为传感器融合。融合过程可分为早期融合和后期融合。早期融合是指点云与图像像素融合，后期融合是指单个检测物的融合。

激光雷达进行障碍物的步骤通常分为4个步骤：

- 点云处理

- 点云分割

- 障碍聚类

- 边界框拟合

为了处理点云，我们可以使用最流行的**库PCL**（point cloud library）。它在Python中可用，但是在C++中使用它更为合理，因为语言更适合机器人学。它也符合ROS（机器人操作系统）。PCL库可以完成探测障碍物所需的大部分计算，从加载点到执行算法。这个库相当于OpenCV的计算机视觉。因为激光雷达的输出很容易达到每秒100000个点，所以我们需要使用一种称为体素网格的方法来对点云进行下采样。体素网格是一个三维立方体，通过每个立方体只留下一个点来过滤点云。立方体越大，点云的最终分辨率越低。最终，我们可以将点云的采样从几万点减少到几千点。

滤波完成后我们可以进行的第二个操作是ROI（感兴趣区域）的提取，我们只需删除不属于特定区域的每一些点云数据，例如左右距离10米以上的点云，前后超过100米的点云都通过滤波器滤除。现在我们有了**降采样并滤波后的点云**了，此时可以继续进行**点云的分割、聚类和边界框实现**。

点云分割任务是将场景与其中的障碍物分离开来，其实就是**地面的分割**。一种非常流行的分割方法称为RANSAC（RANdom Sample consenses）。该算法的目标是识别一组点中的异常值。点云的输出通常表示一些形状。有些形状表示障碍物，有些只是表示地面上的反射。RANSAC的目标是识别这些点，并通过拟合平面或直线将它们与其他点分开。

可以考虑线性回归。但是有这么多的异常值，线性回归会试图平均结果，而得出错误的拟合结果，与线性回归相反，这里的ransac算法将识别这些异常值，且不会拟合它们。 

![](https://img-blog.csdnimg.cn/img_convert/5ad8c07d3796a2fae7fe6d25adecb922.gif)

如上图所示我们可以将这条线视为场景的目标路径（即道路），而孤立点则是障碍物。它是如何工作的？

过程如下：

1. 随机选取2个点

2. 将线性模型拟合到这些点计算每隔一点到拟合线的距离。如果距离在定义的阈值距离公差范围内，则将该点添加到内联线列表中。

因此需要算法一个参数：距离阈值。

3. 最后选择内点最多的迭代作为模型；其余的都是离群值。这样，我们就可以把每一个内点视为道路的一部分，把每一个外点视为障碍的一部分。RANSAC应用在3D点云中。在这种情况下，3个点之间的构成的平面是算法的基础。然后计算点到平面的距离。

RANSAC是一个非常强大和简单的点云分割算法。**它试图找到属于同一形状的点云和不属于同一形状的点云**，然后将其分开。 

### 激光雷达从2D到3D（Cartographer）

Cartographer是由谷歌于2016年开源的一个支持ROS的室内SLAM库，并在截至目前为止，仍然处于不断的更新维护之中。 
特点：代码极为工程，多态、继承、层层封装的十分完善。提供了方便的接口，便于接入IMU、（单/多线）雷达、里程计、甚至为二维码辅助等视觉识别方式也预留了接口（Landmark）。 

- 2D-SLAM：基于2D栅格地图，可以直接用于导航。

- 3D-SLAM：基于hybridGrid，译为混合概率地图，可以理解为3D栅格地图。 **明确：**RViz 仅显示3D混合概率网格的2D 投影(以灰度形式)。该地图难以直接使用。

在Cartographer前端中，不断维护的是scan和submap之间的位姿。整个的map是由一个个submap组成的。

#### Cartographer的匹配方法

Cartographer采用的是scan-map的匹配方法。 一般情况的SLAM有如下三种：

- scan-scan： 这个意味着利用两帧激光数据（每帧激光束的数目相同），计算二者之间的变换。典型方法：ICP。
- scan-map： 利用一帧激光数据和地图数据，找到激光数据在地图中的位置。 
- map-map： 利用一个子地图数据，在一个更大的地图中找到它合适的位置。

不管是2D还是3D，首先要有一个初始的位姿，在此基础上进行优化：

- 有IMU，则采纳其角速度积分作为初始姿态。不信任IMU任何加速度信息。     
- 有里程计，则采纳里程计的线速度积分作为初始平移。   
-  二者都没有，根据之前的运动做一个匀速的假设。  

注意，cartographer的多传感器融合是一个松耦合，主要依赖激光来定位。IMU和里程计数据并没有被构建到真正优化的目标函数中。

在得到了初始位姿以后，初始位姿要经过第一阶段解算：CSM（Correlation Scan Match 相关扫描匹配）——构建似然场。即对原先的地图map进行一个高斯模糊，让它膨胀一些，然后把激光scan在一个搜索窗口内暴力匹配，计算得分。注意，这里有两个问题：

1.得分怎么算？

- 如果scan的点落在障碍物模糊区域内，落的越多，得分越高。

2.地图不是无限大的吗，你怎么保证在搜索窗口里就能找到位姿呢？

- 因为有初始位姿。误差肯定在一个范围内而不会马上发散到很远，所以可以在一个位姿的窗口内，对位姿进行暴力匹配搜索。（初始位姿估计中，里程计数据不会突然激增；imu的加速度信息会漂移，但是算法对于imu加速度数据选择直接丢弃不看；而根据之前位姿匀速假设也不会飘走）

这时候我们就要考虑，什么是位姿？位置+姿态。对于2Dslam而言，有三个变量，$x$，$y$，$yaw$角。 对于3Dslam而言，有$x$，$y$，$z$，$roll$，$pitch$，$yaw$六个变量。

- 2dslam中，采用三层循环，（最外层为θ，减小sin和cos的频繁计算），对$x$，$y$，$\theta$在给定大小的搜索窗口内进行穷举，计算最高得分的$x$，$y$，$\theta$作为一阶段解算的输出位姿。 
- 3dslam中，采用六层循环，对$x$，$y$，$z$，$roll$，$pitch$，$yaw$六个变量在搜索窗口内穷举，计算得分最高的作为一阶段解算输出位姿。 很显然，3d-slam的这种方式对于计算资源依赖较大，复杂度达到$O(n^6)$级别。因此3d-slam的CSM方法，作为一个配置选项，默认是不开启的。当然如果用户机器比较牛逼，也可以选择开启。

第一阶段CSM解算中，位姿在其中是一个离散的变量，通过暴力枚举获得输出结果；但是暴力枚举也是存在分辨率的，例如：如果角度步长设为$1$度，但如果刚好真正的角度是$5.5$度，那么CSM只能搜索到$5$或$6$度，而无法进一步细化，逐步累积将会造成误差。 因此，引入第二阶段位姿解算：非线性优化。

$$
E(T)=\arg \min _{T} \sum\left[1-M\left(S_{i}(T)\right)\right]^{2}
$$

$S_i(T)$表示把激光数据$S$用位姿$T$进行转换，$M(x)$表示得到坐标$x$的地图占用概率。思路：$S$代表了激光击中障碍物，将激光点在机器人坐标系下的位置，经过$T$转换到世界坐标系下以后，应该尽可能的落在已有地图的障碍物上。

第二阶段的位姿求解，显然位姿在其中是一个连续的变量，通过梯度下降的方法求解目标函数。由于地图是离散的，因此需要对地图进行插值处理，使地图也变成一个可以求导的连续变量，这样才能优化前述目标函数。

线性插值：已知数据 $(x_0, y_0)$ 与$ (x_1, y_1)$，要计算 $[x_0, x_1]$ 区间内某一位置 $x$ 在直线上的$y$值； 

双线性插值本质上就是在两个方向上做线性插值。 双三次插值：更加复杂的插值方式，它能创造出比双线性插值更平滑的图像边缘。使用最近$16$个点插值。

#### Cartographer的建图方法

Cartographer的地图（map）以子地图（submap）的形式组成。分为前端和后端。 前端：根据帧间匹配算法（scan-match），实时根据激光(scan)来推测累积的scan相对于submap的位姿。 后端：检测回环（发现在已到达的位置附近），修正各个submap之间的位姿。根据代码可以判断，2D和3D基于的是同一套思路，但是在实现上有一定区别。 接下来结合2D和3D部分，对比介绍实现定位和建图的方法。

#### Cartographer的后端

Cartographer在后端主要寻找回环，并根据建立的约束对所有的sumap进行统一优化。 回环检测目的是：检测当前位置是否曾经来过，即采用当前scan在历史中搜索，确认是否匹配。

为什么要有回环检测呢？原因有二：

1. 已有地图时位姿初始化，不知道当前帧初始位姿，也就不清楚在地图中哪个位置，无法做定位。 
2. 有累积误差，仅靠前端递推，不进行修正的话，地图很容易变形。

因此接下来我们探讨两个问题：1. 如何检测回环。2. 检测回环后该怎么做。

#### 如何检测回环

检测回环和前端的思路也比较相似，先通过穷举暴力匹配，再通过优化精细修正。但是，前端的暴力穷举，是在有个初始位姿的基础上在一个小窗口内穷举。 后端重定位，没有初始位姿了，暴力匹配的范围变成了整个地图。 因此必须采用算法加速处理：多分辨率地图+分枝定界操作。

假设有一帧激光：

![](https://img-blog.csdnimg.cn/20210426170657810.png)

蓝色代表障碍物：

![](https://img-blog.csdnimg.cn/20210426170906439.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3prazk1Mjc=,size_16,color_FFFFFF,t_70)

 在高分辨率的地图上，四个点命中3个；在低分辨率的地图上，四个点全部命中。激光在低分辨率的地图上匹配情况： 代表得分的上界 （再往精细展开，匹配得分只能更低，不能更高） 在高分辨率的地图上匹配情况： 代表得分的下界（ 再往粗略缩放，匹配得分只能更高，不会更低）

分支定界：

1. 先把整个地图中的一个区域展开到底（最高分辨率），得到一个匹配分数（得分下界）；
2. 然后把其他区域不展开，算匹配分数。（得分上界）
3. 如果低分辨率区域的得分上界，还没有已展开到底的高分辨率区域的下界高，这个低分辨率区域就不再展开了，统统丢掉不要。

左图四个命中$3$个，得分$75$； 右图四个命中$2$个，得分$50$；  那么激光打在子区域A的可能性就要大于B，因此B就无需继续展开成更精细的地图了。

1. 2D-slam的思路比较简单 前端：小范围内穷举+非线性优化方法修正位姿。 回环检测：大范围内穷举（利用分支定界加速） +非线性优化修正位姿。
2. 3D-slam有所不同。 前端的暴力匹配方法，是直接6层循环暴力枚举的，因此配置文件中默认不开启，而是在初始通过IMU、里程计等预测位姿基础上，直接非线性优化修正位姿。 如果回环检测仍然是：大范围内6个循环穷举+分支定界的话，小范围都嫌慢，大范围更别提。    直接对位姿非线性优化？ 1.没有初值，会算到猴年马月。2.会落入局部最优值。 

回环检测后，就到优化了~而优化的本质，还是一个最小二乘问题。

### LOAM (Lidar Odometry and Mapping in Real-time)

LOAM是Ji Zhang早期开源的多线LiDAR SLAM算法。该代码可读性很差，作者后来将其闭源。其效果如下：
![](https://img-blog.csdnimg.cn/20210817224627415.gif)

上文介绍的cartographer主要解决室内问题，LOAM室内外都可以，但是没有回环检测。Cartographer的3D部分，更像是2D的扩展：即用2D的思路去做3D的事情。而LOAM则主要解决3D问题，其核心思路难以解决2D问题。虽然该算法发表于14年RSS，但是在KITTI上的Odometry模块（The KITTI Vision Benchmark Suite）的激光SLAM排行榜上，仍然霸占前列。

由于LOAM的工程已经闭源。本文在此处采用下面两个github工程来查看LOAM的效果～

其框架如下图所示
![](https://img-blog.csdnimg.cn/20210819143553576.png)

- Point Cloud Registration：点云不是同一时刻获取的，每一个帧点云，其中的每一个点，都是不同时刻获取的，因此把它进行运动补偿： 获取每个点的时间戳，位姿插值，把点云先投影到同一时刻；提取特征点。 
- Lidar Odometry： 估计两帧点云之间的位姿变换，获得两个时刻之间的相对位姿，频率较高 10Hz 
- Lidar Mapping：  建图模块，把连续10帧的点云数据和整个地图匹配，获得世界坐标系下的位姿，频率较低 1Hz。 
- Transform Intergration：实时利用世界坐标系下的位姿和两个时刻之间的相对位姿，更新各个时刻世界坐标系下的位姿。

Cartographer使用栅格地图，地图中存储着占据概率，通过把点云投影到栅格地图，计算匹配得分，找到最合适的投影，作为位姿变换。但是，LOAM使用的是点云地图，那么点云投影后，进行匹配的就还是点云地图。 两堆点云，如何匹配？

传统方法，使用ICP方法，即默认两堆点云中最近的点是匹配点，构建矩阵进行奇异值分解，得到变换矩阵后投影点云，然后再次寻找匹配点，重新计算投影……直至收敛。然而，对SLAM算法而言，要求同步定位和建图。这样随便根据距离选匹配点，计算太复杂，可能算来算去都不收敛，压根就不能实时，计算量实在是太大了。LOAM作者决定对点云提取特征，然后根据稀疏的特征来计算位姿变换。 作者决定提取两种特征点：平面点和边缘点。

对于平面点与边缘点，作者引入曲率计算方法

计算曲率听起来是一个很麻烦的事情，在高等数学中，一条曲线的曲率以如下公式进行计算：

$$
K=\frac{\left|y^{\prime \prime}\right|}{\left(1+y^{\prime 2}\right)^{\frac{3}{2}}}
$$

但事实上作者并不是这样算的。他直接利用激光的每条扫瞄线中，一个点前后各五个点，计算平均值到该点的距离。

$$
c=\frac{1}{|S| \cdot\left\|\mathbf{X}_{(k, i)}^{L}\right\|} \sum_{j \in S, j \neq i}\left\|\mathbf{X}_{(k, i)}^{L}-\mathbf{X}_{(k, j)}^{L}\right\|
$$

## 公众号

1. [【泡泡机器人成员原创-SLAM求职宝典】SLAM求职经验帖-泡泡机器人的文章-知乎](https://zhuanlan.zhihu.com/p/28565563)
2. [干货总结丨SLAM面试常见问题及参考解答-计算机视觉life的文章-知乎](https://zhuanlan.zhihu.com/p/66540565)
3. [视觉SLAM面试题汇总（2019年秋招题库参考）——第一部分-深蓝学院的文章-知乎](https://zhuanlan.zhihu.com/p/205008396)
4. [万字干货！视觉SLAM面试题汇总（19年秋招）——第二部分-深蓝学院的文章-知乎](https://zhuanlan.zhihu.com/p/212264860)

## 个人分享

1. [SLAM、定位、建图求职分享-小葡萄的文章-知乎](https://zhuanlan.zhihu.com/p/68858564)
2. [SLAM、3D vision笔试面试问题-wlwsjl的文章-知乎](https://zhuanlan.zhihu.com/p/63755692)
3. [SLAM、3D vision求职经历-wlwsjl的文章-知乎](https://zhuanlan.zhihu.com/p/56617825)
4. [SLAM暑期实习求职经验贴-熊勒个猫的文章-知乎](https://zhuanlan.zhihu.com/p/67818202)
5. [CS PhD的SLAM/无人车求职小结-Pickles Husky的文章-知乎](https://zhuanlan.zhihu.com/p/35348586)
6. [SLAM常见面试题（一）-小马恺文的文章-知乎](https://zhuanlan.zhihu.com/p/46694678)
7. [SLAM常见面试题（二）-小马恺文的文章-知乎](https://zhuanlan.zhihu.com/p/46696986)
8. [SLAM常见面试题（三）-小马恺文的文章-知乎](https://zhuanlan.zhihu.com/p/46697912)
9. [面试SLAM算法实习生总结-无能狂怒SLAM崔的文章-知乎](https://zhuanlan.zhihu.com/p/76280626)

## 代码面试

### C++知识总结

1. [interview-📚C/C++技术面试基础知识总结（一）-辉哈huihut的文章-知乎](https://zhuanlan.zhihu.com/p/114311142)
2. [校招C++大概学习到什么程度？-程序员内功修炼的回答-知乎](https://www.zhihu.com/question/290102232/answer/2094675219)
3. [C++常见面试题总结-Cpp小茶馆的文章-知乎](https://zhuanlan.zhihu.com/p/354382975)
4. [你遇到过哪些高质量的C++面试？-知乎用户的回答-知乎](https://www.zhihu.com/question/60911582/answer/1783988850)
5. [如果你是一个C++面试官，你会问哪些问题？-拓跋阿秀的回答-知乎](https://www.zhihu.com/question/451327108/answer/1868156551)
6. [C++学到什么程度可以面试工作？-牛客网的回答-知乎](https://www.zhihu.com/question/400543720/answer/1845364139)
7. [当面试官问我C++11新特性的时候，应该怎样回答？-程序喵大人的回答-知乎](https://www.zhihu.com/question/65209863/answer/1957019832)

### 算法总结

1. [程序员必须掌握哪些算法？-程序员客栈的回答-知乎](https://www.zhihu.com/question/23148377/answer/714596562)
2. [程序员必须掌握哪些算法？-帅地的回答-知乎](https://www.zhihu.com/question/23148377/answer/863990767)
3. [程序员必须掌握哪些算法？-力扣（LeetCode）的回答-知乎](https://www.zhihu.com/question/23148377/answer/602761180)
4. [程序员必须掌握哪些算法？-牛客网的回答-知乎](https://www.zhihu.com/question/23148377/answer/1012283025)

### 算法详解

1. [常用的排序算法总结-力扣（LeetCode）的文章-知乎](https://zhuanlan.zhihu.com/p/40695917)
2. [十大经典排序算法-菜鸟教程](https://www.runoob.com/w3cnote/ten-sorting-algorithm.html)
3. [排序算法总结-菜鸟教程](https://www.runoob.com/w3cnote/sort-algorithm-summary.html)

## GitHub

1. [jwasham/coding-interview-university](https://github.com/jwasham/coding-interview-university)
2. [CyC2018/CS-Notes](https://github.com/CyC2018/CS-Notes)
3. [labuladong/fucking-algorithm](https://github.com/labuladong/fucking-algorithm)
4. [azl397985856/leetcode](https://github.com/azl397985856/leetcode)
5. [geekxh/hello-algorithm](https://github.com/geekxh/hello-algorithm)
6. [huihut/interview](https://github.com/huihut/interview)
7. [youngyangyang04/leetcode-master](https://github.com/youngyangyang04/leetcode-master)
8. [amusi/Deep-Learning-Interview-Book](https://github.com/amusi/Deep-Learning-Interview-Book)
9. [0voice/campus_recruitmen_questions](https://github.com/0voice/campus_recruitmen_questions)
10. [forthespada/InterviewGuide](https://github.com/forthespada/InterviewGuide)
11. [DarLiner/Algorithm_Interview_Notes-Chinese](https://github.com/DarLiner/Algorithm_Interview_Notes-Chinese)
12. [Liber-coder/CV_Notes](https://github.com/Liber-coder/CV_Notes)

## 计算机系课程

1. [QSCTech/zju-icicles](https://github.com/QSCTech/zju-icicles)
2. [PKUanonym/REKCARC-TSC-UHT](https://github.com/PKUanonym/REKCARC-TSC-UHT)
3. [USTC-Resource/USTC-Course](https://github.com/USTC-Resource/USTC-Course)
4. [c-hj/SJTU-Courses](https://github.com/c-hj/SJTU-Courses)
5. [missing-semester/missing-semester](https://github.com/missing-semester/missing-semester)
6. [HuangCongQing/UCAS_Course_2019](https://github.com/HuangCongQing/UCAS_Course_2019)
7. [15172658790/Blog](https://github.com/15172658790/Blog)
8. [Salensoft/thu-cst-cracker](https://github.com/Salensoft/thu-cst-cracker)

## 传感器相关

### 激光雷达数据生成

1. [UMich-BipedLab/lidar_simulator](https://github.com/UMich-BipedLab/lidar_simulator)
2. [xiaoaoran/SynLiDAR](https://github.com/xiaoaoran/SynLiDAR)
3. [MartinHahner/LiDAR_fog_sim](https://github.com/MartinHahner/LiDAR_fog_sim)
4. [trripy/CARLA_ROS_SLAM](https://github.com/trripy/CARLA_ROS_SLAM)
5. [ntnu-arl/lidar_simulator](https://github.com/ntnu-arl/lidar_simulator)

### 激光雷达内参标定

1. [UMich-BipedLab/LiDAR_intrinsic_calibration](https://github.com/UMich-BipedLab/LiDAR_intrinsic_calibration)

### IMU噪声评估

1. [gaowenliang/imu_utils](https://github.com/gaowenliang/imu_utils)
2. [ccny-ros-pkg/imu_tools](https://github.com/ccny-ros-pkg/imu_tools)
3. [Kyle-ak/imu_tk](https://github.com/Kyle-ak/imu_tk)
4. [XinLiGH/GyroAllan](https://github.com/XinLiGH/GyroAllan)

### 激光雷达-相机标定

此类项目较多，搜索`LiDAR`+`camera`+`calibration`即可。

#### 基于MATLAB

1. [UMich-BipedLab/extrinsic_lidar_camera_calibration](https://github.com/UMich-BipedLab/extrinsic_lidar_camera_calibration)
2. [zhixy/Laser-Camera-Calibration-Toolbox](https://github.com/zhixy/Laser-Camera-Calibration-Toolbox)
3. [Aaron20127/Camera-lidar-joint-calibration](https://github.com/Aaron20127/Camera-lidar-joint-calibration)
4. [ccyinlu/multimodal_data_studio](https://github.com/ccyinlu/multimodal_data_studio)
5. [UMich-BipedLab/automatic_lidar_camera_calibration](https://github.com/UMich-BipedLab/automatic_lidar_camera_calibration)
6. MATLAB Lidar Camera Calibrator

#### 依赖项较多

1. [ankitdhall/lidar_camera_calibration](https://github.com/ankitdhall/lidar_camera_calibration)
2. [beltransen/velo2cam_calibration](https://github.com/beltransen/velo2cam_calibration)
3. [mfxox/ILCC](https://github.com/mfxox/ILCC)
4. [heethesh/lidar_camera_calibration](https://github.com/heethesh/lidar_camera_calibration)
5. [hku-mars/livox_camera_calib](https://github.com/hku-mars/livox_camera_calib)
6. [Livox-SDK/livox_camera_lidar_calibration](https://github.com/Livox-SDK/livox_camera_lidar_calibration)
7. [XidianLemon/calibration_camera_lidar](https://github.com/XidianLemon/calibration_camera_lidar)
8. [swyphcosmo/ros-camera-lidar-calibration](https://github.com/swyphcosmo/ros-camera-lidar-calibration)
9. [hku-mars/mlcc](https://github.com/hku-mars/mlcc)
10. [ram-lab/plycal](https://github.com/ram-lab/plycal)
11. [AbangLZU/cam_lidar_calibration](https://github.com/AbangLZU/cam_lidar_calibration)

### 激光雷达-IMU标定

1. [ethz-asl/lidar_align](https://github.com/ethz-asl/lidar_align)
2. [APRIL-ZJU/lidar_IMU_calib](https://github.com/APRIL-ZJU/lidar_IMU_calib)
3. [chennuo0125-HIT/lidar_imu_calib](https://github.com/chennuo0125-HIT/lidar_imu_calib)
4. [FENGChenxi0823/SensorCalibration](https://github.com/FENGChenxi0823/SensorCalibration)
5. [unmannedlab/imu_lidar_calibration](https://github.com/unmannedlab/imu_lidar_calibration)
6. [liyangSKD/lidar_rtk_calibration](https://github.com/liyangSKD/lidar_rtk_calibration)
7. [chengwei0427/calib_lidar_imu](https://github.com/chengwei0427/calib_lidar_imu)
8. [BohemianRhapsodyz/lidar_imu_calib](https://github.com/BohemianRhapsodyz/lidar_imu_calib)

### 相机-IMU标定

1. [ethz-asl/kalibr](https://github.com/ethz-asl/kalibr)
2. [hovren/crisp](https://github.com/hovren/crisp)
3. [urbste/OpenImuCameraCalibrator](https://github.com/urbste/OpenImuCameraCalibrator)

### 多激光雷达标定

1. [Livox-SDK/Livox_automatic_calibration](https://github.com/Livox-SDK/Livox_automatic_calibration)
2. [AbangLZU/multi_lidar_calibration](https://github.com/AbangLZU/multi_lidar_calibration)

### 多相机标定

1. [hengli/camodocal](https://github.com/hengli/camodocal)
2. [ethz-asl/kalibr](https://github.com/ethz-asl/kalibr)
3. [rameau-fr/MC-Calib](https://github.com/rameau-fr/MC-Calib)

### 多传感器联合标定

1. [tudelft-iv/multi_sensor_calibration](https://github.com/tudelft-iv/multi_sensor_calibration)
2. [PRBonn/extrinsic_calibration](https://github.com/PRBonn/extrinsic_calibration)
3. [plumewind/online_calibration](https://github.com/plumewind/online_calibration)
4. [zhixy/multical](https://github.com/zhixy/multical)
5. [zxl19/Hand_Eye_Extrinsic_Calibration](https://github.com/zxl19/Hand_Eye_Extrinsic_Calibration)

### 手眼标定

1. [zarathustr/SHERWCIC](https://github.com/zarathustr/SHERWCIC)

## ROS相关

### 系统状态监控

1. [ethz-asl/ros-system-monitor](https://github.com/ethz-asl/ros-system-monitor)
2. [ethz-asl/linter](https://github.com/ethz-asl/linter)
3. [ethz-asl/config_utilities](https://github.com/ethz-asl/config_utilities)

### Rviz可视化

1. [ros-visualization/rviz](https://github.com/ros-visualization/rviz)
2. [PickNikRobotics/rviz_visual_tools](https://github.com/PickNikRobotics/rviz_visual_tools)
3. [OctoMap/octomap_rviz_plugins](https://github.com/OctoMap/octomap_rviz_plugins)

### rosbag相关

#### 制作

1. [tomas789/kitti2bag](https://github.com/tomas789/kitti2bag)
2. [AbnerCSZ/lidar2rosbag_KITTI](https://github.com/AbnerCSZ/lidar2rosbag_KITTI)
3. [ethz-asl/kitti_to_rosbag](https://github.com/ethz-asl/kitti_to_rosbag)
4. [clynamen/nuscenes2bag](https://github.com/clynamen/nuscenes2bag)

#### 内容修改

1. [facontidavide/rosbag_editor](https://github.com/facontidavide/rosbag_editor)
2. [AtsushiSakai/rosbag_filter_gui](https://github.com/AtsushiSakai/rosbag_filter_gui)
3. [tier4/ros2bag_extensions](https://github.com/tier4/ros2bag_extensions)

#### 数据提取

1. [zxl19/LiDAR_Camera_Calibration_Preprocess](https://github.com/zxl19/LiDAR_Camera_Calibration_Preprocess)
2. [amc-nu/rosbag_image_pcd_exporter](https://github.com/amc-nu/rosbag_image_pcd_exporter)
3. [AtsushiSakai/rosbag_to_csv](https://github.com/AtsushiSakai/rosbag_to_csv)

## 数据标注

### 多类型信息多任务

1. [heartexlabs/label-studio](https://github.com/heartexlabs/label-studio)
2. [UniversalDataTool/universal-data-tool](https://github.com/UniversalDataTool/universal-data-tool)

### 点云

主要针对语义分割标注。

1. [Hitachi-Automotive-And-Industry-Lab/semantic-segmentation-editor](https://github.com/Hitachi-Automotive-And-Industry-Lab/semantic-segmentation-editor)
2. [jbehley/point_labeler](https://github.com/jbehley/point_labeler)
3. [halostorm/PCAT_open_source](https://github.com/halostorm/PCAT_open_source)
4. [yzrobot/cloud_annotation_tool](https://github.com/yzrobot/cloud_annotation_tool)
5. [RMonica/rviz_cloud_annotation](https://github.com/RMonica/rviz_cloud_annotation)
6. [hasanari/sane](https://github.com/hasanari/sane)

### 图像

主要针对目标检测标注。

1. [tzutalin/labelImg](https://github.com/tzutalin/labelImg)
2. [wkentaro/labelme](https://github.com/wkentaro/labelme)
3. [AlexeyAB/Yolo_mark](https://github.com/AlexeyAB/Yolo_mark)
4. [abreheret/PixelAnnotationTool](https://github.com/abreheret/PixelAnnotationTool)
5. [Cartucho/OpenLabeling](https://github.com/Cartucho/OpenLabeling)
6. [developer0hye/Yolo_Label](https://github.com/developer0hye/Yolo_Label)
7. [PaddleCV-SIG/EISeg](https://github.com/PaddleCV-SIG/EISeg)
8. MATLAB Image Labeler

## SLAM相关

### 位姿表示

1. [Eigen](http://eigen.tuxfamily.org/)
2. [strasdat/Sophus](https://github.com/strasdat/Sophus)
3. [petercorke/spatialmath-matlab](https://github.com/petercorke/spatialmath-matlab)
4. [brentyi/jaxlie](https://github.com/brentyi/jaxlie)
5. [ethz-asl/minkindr](https://github.com/ethz-asl/minkindr)

### 优化库

1. [ceres-solver/ceres-solver](https://github.com/ceres-solver/ceres-solver)
2. [RainerKuemmerle/g2o](https://github.com/RainerKuemmerle/g2o)
3. [borglab/gtsam](https://github.com/borglab/gtsam)
4. [stevengj/nlopt](https://github.com/stevengj/nlopt)
5. [baidu/ICE-BA](https://github.com/baidu/ICE-BA)
6. [ethz-adrl/ifopt](https://github.com/ethz-adrl/ifopt)
7. [MIT-SPARK/Kimera-RPGO](https://github.com/MIT-SPARK/Kimera-RPGO)
8. [xipengwang/AprilSAM](https://github.com/xipengwang/AprilSAM)
9. [zlthinker/STBA](https://github.com/zlthinker/STBA)
10. [srrg-sapienza/srrg2_solver](https://github.com/srrg-sapienza/srrg2_solver)
11. [ori-drs/isam](https://github.com/ori-drs/isam)
12. [ZJU-FAST-Lab/LBFGS-Lite](https://github.com/ZJU-FAST-Lab/LBFGS-Lite)
13. [helmayer/RPBA](https://github.com/helmayer/RPBA)
14. [bbopt/nomad](https://github.com/bbopt/nomad)
15. [utiasASRL/steam](https://github.com/utiasASRL/steam)
16. [cvg/pyceres](https://github.com/cvg/pyceres)

### 机器人学算法库

1. [MOLAorg/mola](https://github.com/MOLAorg/mola)
2. [onlytailei/CppRobotics](https://github.com/onlytailei/CppRobotics)
3. [zhujun98/sensor-fusion](https://github.com/zhujun98/sensor-fusion)
4. [ydsf16/TinyGrapeKit](https://github.com/ydsf16/TinyGrapeKit)
5. [MRPT/mrpt](https://github.com/MRPT/mrpt)
6. [AtsushiSakai/PythonRobotics](https://github.com/AtsushiSakai/PythonRobotics)
7. [petercorke/robotics-toolbox-python](https://github.com/petercorke/robotics-toolbox-python)
8. [AtsushiSakai/MATLABRobotics](https://github.com/AtsushiSakai/MATLABRobotics)
9. [MobileRoboticsSkoltech/mrob](https://github.com/MobileRoboticsSkoltech/mrob)

### 点云处理

1. [PointCloudLibrary/pcl](https://github.com/PointCloudLibrary/pcl)
2. [isl-org/Open3D](https://github.com/isl-org/Open3D)
3. [kzampog/cilantro](https://github.com/kzampog/cilantro)
4. [LiangliangNan/Easy3D](https://github.com/LiangliangNan/Easy3D)
5. [CloudCompare/CCCoreLib](https://github.com/CloudCompare/CCCoreLib)
6. [flann-lib/flann](https://github.com/flann-lib/flann)
7. [jlblancoc/nanoflann](https://github.com/jlblancoc/nanoflann)
8. [jbehley/octree](https://github.com/jbehley/octree)
9. [AcademySoftwareFoundation/openvdb](https://github.com/AcademySoftwareFoundation/openvdb)
10. [facontidavide/Treexy](https://github.com/facontidavide/Treexy)
11. [dimatura/pypcd](https://github.com/dimatura/pypcd)

### 点云匹配算法

搜索`point cloud registration`或者`iterative closest point`。

#### 基于几何特征

1. [ethz-asl/libpointmatcher](https://github.com/ethz-asl/libpointmatcher)
2. [MIT-SPARK/TEASER-plusplus](https://github.com/MIT-SPARK/TEASER-plusplus)
3. [SMRT-AIST/fast_gicp](https://github.com/SMRT-AIST/fast_gicp)
4. [neka-nat/probreg](https://github.com/neka-nat/probreg)
5. [ethz-asl/robust_point_cloud_registration](https://github.com/ethz-asl/robust_point_cloud_registration)
6. [koide3/ndt_omp](https://github.com/koide3/ndt_omp)
7. [nmellado/Super4PCS](https://github.com/nmellado/Super4PCS)
8. [yorsh87/nicp](https://github.com/yorsh87/nicp)
9. [YuePanEdward/GH-ICP](https://github.com/YuePanEdward/GH-ICP)
10. [STORM-IRIT/OpenGR](https://github.com/STORM-IRIT/OpenGR)

#### 基于深度学习

1. [zgojcic/3D_multiview_reg](https://github.com/zgojcic/3D_multiview_reg)
2. [overlappredator/OverlapPredator](https://github.com/overlappredator/OverlapPredator)

### 回环检测

1. [irapkaist/scancontext](https://github.com/irapkaist/scancontext)
2. [emiliofidalgo/ibow-lcd](https://github.com/emiliofidalgo/ibow-lcd)
3. [wh200720041/SRLCD](https://github.com/wh200720041/SRLCD)
4. [lhanaf/MILD](https://github.com/lhanaf/MILD)
5. [BigMoWangying/LiDAR-Iris](https://github.com/BigMoWangying/LiDAR-Iris)

### 图像处理

1. [opencv/opencv](https://github.com/opencv/opencv)
2. [ethz-asl/aslam_cv2](https://github.com/ethz-asl/aslam_cv2)

### 视觉重建

1. [openMVG/openMVG](https://github.com/openMVG/openMVG)
2. [colmap/colmap](https://github.com/colmap/colmap)
3. [ethz-asl/maplab](https://github.com/ethz-asl/maplab)
4. [open-mmlab/mmflow](https://github.com/open-mmlab/mmflow)
5. [Xbbei/super-colmap](https://github.com/Xbbei/super-colmap)

### 可视化

1. [CloudCompare/CloudCompare](https://github.com/CloudCompare/CloudCompare)
2. [facontidavide/PlotJuggler](https://github.com/facontidavide/PlotJuggler)
3. [MarkMuth/QtKittiVisualizer](https://github.com/MarkMuth/QtKittiVisualizer)
4. [orsalmon/KittiDatasetGPS-INSViewer](https://github.com/orsalmon/KittiDatasetGPS-INSViewer)

### 精度评估

1. [MichaelGrupp/evo](https://github.com/MichaelGrupp/evo)
2. [pamela-project/slambench2](https://github.com/pamela-project/slambench2)
3. [pamela-project/slambench1](https://github.com/pamela-project/slambench1)
4. [MobileRoboticsSkoltech/map-metrics](https://github.com/MobileRoboticsSkoltech/map-metrics)
5. [anastasiia-kornilova/mom-tools](https://github.com/anastasiia-kornilova/mom-tools)

### 地理坐标系

1. [GeographicLib](https://geographiclib.sourceforge.io/)
2. [ethz-asl/geodetic_utils](https://github.com/ethz-asl/geodetic_utils)
3. [wandergis/coordTransform_py](https://github.com/wandergis/coordTransform_py)
4. [ue4plugins/UEGeoCoordinates](https://github.com/ue4plugins/UEGeoCoordinates)
5. [Stellacore/peridetic](https://github.com/Stellacore/peridetic)
6. [chachi/GeoCon](https://github.com/chachi/GeoCon)

### 地图匹配定位

1. [koide3/hdl_localization](https://github.com/koide3/hdl_localization)
2. [at-wat/mcl_3dl](https://github.com/at-wat/mcl_3dl)
3. [HViktorTsoi/FAST_LIO_LOCALIZATION](https://github.com/HViktorTsoi/FAST_LIO_LOCALIZATION)
4. [robotics-upo/dll](https://github.com/robotics-upo/dll)
5. [rsasaki0109/ndt_mapping](https://github.com/rsasaki0109/ndt_mapping)
6. [AbangLZU/ndt_localizer](https://github.com/AbangLZU/ndt_localizer)

### EKF定位

1. [libing64/pose_ekf](https://github.com/libing64/pose_ekf)
2. [udacity/robot_pose_ekf](https://github.com/udacity/robot_pose_ekf)
3. [rsasaki0109/kalman_filter_localization](https://github.com/rsasaki0109/kalman_filter_localization)

### GNSS数据处理

1. [tomojitakasu/RTKLIB](https://github.com/tomojitakasu/RTKLIB)
2. [weisongwen/GraphGNSSLib](https://github.com/weisongwen/GraphGNSSLib)
3. [HKUST-Aerial-Robotics/gnss_comm](https://github.com/HKUST-Aerial-Robotics/gnss_comm)

## 场景仿真

1. [carla-simulator/carla](https://github.com/carla-simulator/carla)
2. [eclipse/sumo](https://github.com/eclipse/sumo)
3. [ucla-mobility/OpenCDA](https://github.com/ucla-mobility/OpenCDA)
## 参考

1. [GitHub开源计算机课程攻略yyds-逛逛GitHub的文章-知乎](https://zhuanlan.zhihu.com/p/447898788)


## 参考文献

【1】 https://blog.csdn.net/gwplovekimi/article/details/119711762
