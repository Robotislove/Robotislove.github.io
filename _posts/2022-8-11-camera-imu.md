---
layout: post
title: 双目相机和IMU联合标定实验笔记
date: 2022-08-11
author: lau
tags: [Archive]
comments: true
toc: true
pinned: false
---
双目相机和IMU联合标定实验笔记。

<!-- more -->

## 前言

标定的目的是获取**双目参数**，**imu参数**以及二者直接之间的**转换矩阵**，分为三个阶段：
- 双目标定获取双目相机内参以及双目直接的转换矩阵
- imu标定获取噪声密度、随机游走
- imu+双目联合标定获取imu与双目之间的转换矩阵

需要用到的知识包括（ros的基本操作：工作空间与功能包，rosbag记录数据集； 相机模型；）

## 标定过程

### 双目相机标定

**双目标定采用Kalibr标定工具**

1. 首先创建一个新的ros的工作空间，命名为kalibr_ws,将Kalibr的源码（功能包）下载到src文件夹下，用catkin_make命令进行编译.

2. 下载标定板，里面有三种类型的标定板(Aprilgrid, Checkerboard, Circlegrid). 由于Aprilgrid能提供序号信息, 能够防止姿态计算时出现跳跃的情况, 所以建议采用Aprilgrid进行标定.

3. 下载例子，包括一个Multiple camera calibration（多相机标定）和一个IMU-camera calibration（imu+相机联合标定），这里我们先用前者标定相机，里面包括相机参数文件camchain.yaml标定结果文件target.yaml,录制的数据集.bag文件

4. 录制.bag数据集

为了之后imu和相机联合标定，这里我们采集双目和imu的数据。如果你已经有了通过 ROS 发布 image 和imu消息的节点, 我们只需要使用 rosbag record 工具将拍摄到的标定板图像制作成 bag 文件就行了。

**注意: ** 

通常设备采集的频率为 20-60 hz, 这会使得标定的图像过多, 而导致计算量太大. 最好将ros topic的频率降低到4hz左右进行采集.

ROS 提供了改变 topic 发布频率的节点throttle, 分别打开两个终端，运行以下命令以更改图片发布的频率 ：
```
rosrun topic_tools throttle messages /mynteye/left/image_raw 4.0 /left
rosrun topic_tools throttle messages /mynteye/right/image_raw 4.0 /right
```
其中/mynteye/left/image_raw和messages /mynteye/right为原始的话题名称，/left和/right为更改频率后的话题名称

然后就可以制作bag了，具体方法参考[视频](https://www.bilibili.com/video/BV1rC4y1p7ma?from=search&seid=12889677312177442999&vd_source=acc036f52075e7e5c8044d47a94d4660)
```
rosbag record -O stereo_calibra.bag /left /right /imu
```

5. 运行标定程序
进入下载的例子Multiple camera calibration文件夹下，修改camchain.yaml文件中的内容，包括话题名称，频率等信息，标定板尺寸信息（视频里已经讲解的很清楚了）

打开一个终端，首先source一下，然后输入命令（注意路径）

```
source devel/setup.bash
rosrun kalibr kalibr_calibrate_cameras --bag /home/heyijia/stereo_calibra.bag --topics /left /right --models pinhole-equi pinhole-equi --target /home/heyijia/april_6x6_80x80cm_A0.yaml
```

内容包括–bag（数据集 bag文件） --topics （相机话题）–models （相机模型）–target（参数输出文件）

6. 标定结果

标定完的参数储存在了.yaml文件里，主要包括相机内参、畸变参数、基线等内容。

```xml
cam0:
  cam_overlaps: [1]
  camera_model: pinhole
  distortion_coeffs: [0.2783471016021346, 0.6964790605940689, -1.7636165117280103,
    1.9185066948675424]
  distortion_model: equidistant
  intrinsics: [464.46660755800724, 465.36764628863784, 345.1018538803555, 229.48918054311704]
  resolution: [640, 480]
  rostopic: /left
cam1:
  T_cn_cnm1:
  - [0.9996465213666637, -0.02537787899957037, 0.007924366031799834, -0.11251195025612559]
  - [0.02547649136646326, 0.9995959921960228, -0.012601617884125324, -0.0017042839059835384]
  - [-0.0076013621922192826, 0.012799048524251695, 0.9998891956860474, -0.0005072054140188229]
  - [0.0, 0.0, 0.0, 1.0]
  cam_overlaps: [0]
  camera_model: pinhole
  distortion_coeffs: [0.2682739040565608, 0.7579325851647005, -2.0708675842774156,
    2.3962970091118696]
  distortion_model: equidistant
  intrinsics: [461.8506067496413, 462.2467047389745, 325.2758830849641, 247.10152372631737]
  resolution: [640, 480]
  rostopic: /right
```

![](https://img-blog.csdnimg.cn/20200917103906185.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FpbnFpbnhpYW5zaGVuZw==,size_16,color_FFFFFF,t_70#pic_center)

### imu标定

imu标定采用[imu_utils](https://github.com/gaowenliang/imu_utils)

1. 安装依赖项


```
sudo apt-get install libdw-dev
```
2. 下载imu_utils和code_utils并放入工作空间进行编译


- imu_utils下载地址为：https://github.com/gaowenliang/imu_utils
- code_utils下载地址为： https://github.com/gaowenliang/code_utils

**注意：**

- 在此之前需要安装Ceres库
- 先创建一个工作空间（这里为imu_ws），将code_utils放到src文件夹下编译，通过后再将imu_utils放到src文件夹下编译，不能一起编译，因为后者依赖于前者

```
sudo apt-get install libdw-dev
```

在code_utils下面找到sumpixel_test.cpp，修改`#include "backward.hpp`"为 `#include “code_utils/backward.hpp`”，再编译

编译报错：执行`sudo apt-get install ros-kinetic-bfl`过程中无法定位包，(找不到链接了，回来补上，按照ubuntu无法定位包的问题来搜索).

4. 录制imu.bag


保持IMU静止不动至少两个小时，录制IMU的bag
```
rosbag record /imu -O imu
```

5. 根据需求修改launch文件


根据自己的需求对src/imu_utils-master/launch文件进行修改：主要包括名字、时长之类的； 比如: mynt_imu.launch
![](https://img-blog.csdnimg.cn/20200917105258708.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FpbnFpbnhpYW5zaGVuZw==,size_16,color_FFFFFF,t_70#pic_center)

6. 运行标定程序


运行bag文件
```
roscore
rosbag play -r 200　imu_utils/imu.bag
```
运行launch文件
```
cd imu_ws
source ./devel/setup.bash
roslaunch imu_utils mynt_imu.launch
```
其中 -r 200是指200速播放bag数据，不可能真等两个小时2333333333333

7. 标定结果

标定完成后，可在data文件中找到对应的yaml文件src/imu_utils/data/m210_imu_param.yaml

![](https://img-blog.csdnimg.cn/20200917104954512.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FpbnFpbnhpYW5zaGVuZw==,size_16,color_FFFFFF,t_70#pic_center)

标定好的参数包括各方向的随机噪声与随机游走，如下所示：
```
%YAML:1.0
---
type: IMU
name: m210
Gyr:
   unit: " rad/s"
   avg-axis:
      gyr_n: 1.1016440992489866e-03
      gyr_w: 5.7968344392887427e-06
   x-axis:
      gyr_n: 1.2601308952358316e-03
      gyr_w: 6.6549144466689008e-06
   y-axis:
      gyr_n: 9.1639862355727669e-04
      gyr_w: 4.5920115983527386e-06
   z-axis:
      gyr_n: 1.1284027789538512e-03
      gyr_w: 6.1435772728445881e-06
Acc:
   unit: " m/s^2"
   avg-axis:
      acc_n: 5.1380013730273288e-02
      acc_w: 1.7544697355051581e-03
   x-axis:
      acc_n: 3.1145378555559291e-02
      acc_w: 2.9961069437766212e-03
   y-axis:
      acc_n: 7.0380818146931270e-02
      acc_w: 1.9377334256450410e-03
   z-axis:
      acc_n: 5.2613844488329294e-02
      acc_w: 3.2956883709381275e-04
```

### imu+双目联合标定

与kalibr标定双目的过程类似，这次在IMU-camera calibration例子的基础上标定imu与camera

首先以上两次标定结果修改.yaml参数,包括标定结果，话题类型，频率等信息

运行命令进行标定

```
kalibr_calibrate_imu_camera --target april_6x6.yaml --cam camchain.yaml --imu imu_adis16448.yaml --bag /home/dji/db/stero_dji_kalibr/stereo/imu_stereo_subset.bag --bag-from-to 5 45
```

运行时需要注意路径

**标定结果：**

.txt文件中的部分结果,最重要的是相机与imu之间的变换矩阵

```
Calibration results
===================
Normalized Residuals
----------------------------
Reprojection error (cam0):     mean 0.126389437034, median 0.117608381877, std: 0.0673478862055
Reprojection error (cam1):     mean 0.132576172002, median 0.12130080264, std: 0.0739372479366
Gyroscope error (imu0):        mean 0.000146638755352, median 3.16696236165e-07, std: 0.0014737840839
Accelerometer error (imu0):    mean 1.14000452874e-06, median 1.02723705118e-08, std: 6.0143590628e-06

Residuals
----------------------------
Reprojection error (cam0) [px]:     mean 0.126389437034, median 0.117608381877, std: 0.0673478862055
Reprojection error (cam1) [px]:     mean 0.132576172002, median 0.12130080264, std: 0.0739372479366
Gyroscope error (imu0) [rad/s]:     mean 5.18446291471e-06, median 1.11969028084e-08, std: 5.21061359864e-05
Accelerometer error (imu0) [m/s^2]: mean 8.06104932854e-08, median 7.26366284774e-10, std: 4.2527940778e-07

Transformation (cam0):
-----------------------
T_ci:  (imu0 to cam0): 
[[-0.02093225 -0.99971259  0.01168657 -0.00021865]
 [ 0.13518962 -0.01441203 -0.99071492 -0.00086208]
 [ 0.99059861 -0.01915798  0.13545244  0.00014753]
 [ 0.          0.          0.          1.        ]]

T_ic:  (cam0 to imu0): 
[[-0.02093225  0.13518962  0.99059861 -0.00003417]
 [-0.99971259 -0.01441203 -0.01915798 -0.00022818]
 [ 0.01168657 -0.99071492  0.13545244 -0.00087151]
 [ 0.          0.          0.          1.        ]]

timeshift cam0 to imu0: [s] (t_imu = t_cam + shift)
0.000336885304337
```

![](https://img-blog.csdnimg.cn/20200917105744831.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FpbnFpbnhpYW5zaGVuZw==,size_16,color_FFFFFF,t_70#pic_center)


### 参考文献

[1] [Kalibr 之 Camera-IMU 标定 (总结)](https://github.com/cggos/cggos.github.io/blob/c691cf45efa30a669f0ccd7b23c86c12d6e1152f/source\_/posts/robotics/2018-10-23-kalibr-camera-imu.md)

[2] [Kalibr使用：imu+camera联合标定过程详解](https://blog.csdn.net/qinqinxiansheng/article/details/108629530?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-108629530-blog-124291667.pc_relevant_multi_platform_whitelistv3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-108629530-blog-124291667.pc_relevant_multi_platform_whitelistv3&utm_relevant_index=6)

[3] [双目相机 -- IMU联合标定](https://blog.csdn.net/weixin_48592526/article/details/124291667)
