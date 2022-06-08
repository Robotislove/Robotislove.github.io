---
layout: post
title: Aloam+deeplabv3+构建语义地图+行人车辆检测(kitti数据集)
date: 2020-06-06
author: lau
tags: [语义SLAM, Archive]
comments: true
toc: true
pinned: false
---
基于两个开源库实现的语义SLAM。

<!-- more -->

本文首先用了一种非常简单粗暴的方法生成语义地图；先看看效果，之后逐渐改成了运行bag实时构建语义地图；这似乎也只能叫语义地图构建；不能叫语义slam因为语义信息并没有运行到前端匹配以及后段优化部分；

正确的思路应是语义信息可以帮助前端匹配部分添加一个约束帮助特征点的匹配增加系统的鲁棒性；这样在一些缺少文理的地形；即能利用相机能感知周围纹理的优势又能利用激光雷达感知距离的优势；后期逐渐完善

主要用到的两个github链接(感谢大佬)：1.融合相机的aloam，https://github.com/TianXiaoRui/color_map_loam. 2.deeplabv3+：https://github.com/VainF/DeepLabV3Plus-Pytorch.

## 1、kitti数据集
### 1.1、下载

好多文章只给了一个kitti的主业但是当打开时并不知道下载哪一个；kitti的数据十分详细分类很多，可以查阅一些kitti的解释博客；官方在rawdata页面也给出了解释的论文；重要的是耐心的看一般英语水平问题不大；

http://www.cvlibs.net/publications/Geiger2013IJRR.pdf

第一次下载需要先注册一下问题不大；

https://pan.baidu.com/s/1h4k4o3VfnUZZm2nl21FyvA

提取码: cmm5

### 1.2、kitti数据集解释

下载后是4个文件夹(别的是我解压出来的)；
![](https://img-blog.csdnimg.cn/2f421ca025574e7bbbe60c6db61850e3.png)

 calib是校准文件；包括
 
 ![](https://img-blog.csdnimg.cn/3b8eefae80bd4794995f04507b09c930.png)
 
 看一下kitti给出传感器位置图就能明白了
 
 ![](https://img-blog.csdnimg.cn/87c79c53a31844298b687f857e41a1d8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMTExMTExMTExMTEyNDU0NTQ1,size_17,color_FFFFFF,t_70,g_se,x_16)
 
cam1是右侧灰度摄像机；cam1是右侧彩色摄像机；cam0是左侧灰度摄像机；cam1是左侧彩色摄像机；

现在再看三个标定文件就比较清楚了

cam_to_cam是四个相机之间的标定文件；包括相机的内参；

imu_velo是imu到雷达的标定文件；本文用不到

velo_to_cam是雷达到相机的标定文件；

雷达到相机的投影公式如下：
![](https://img-blog.csdnimg.cn/257278ed71524c25847fe2a980122766.png)
![](https://img-blog.csdnimg.cn/f2c1d20020b844fab830ef19f314ed3c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMTExMTExMTExMTEyNDU0NTQ1,size_20,color_FFFFFF,t_70,g_se,x_16)

extract是原始数据；sync代表同步后的数据；

打开sync之后是这样的
![](https://img-blog.csdnimg.cn/1caceec804e147ea8ad2f8aee2d25b0f.png)

 image里存储着相机的图片；oxts里是imu和gnss的数据；velodyne里是雷达数据；
 
### 1.3、转成bag

在转换是extract和sync都可以转换成bag所有在转换时要确定好路径；自己能在kitti下载可以把标定文件放里面；

进入到这一个路径在extract和sync里都有；我们当然要用sync里的同步好的数据；打开终端

先安装kitti2bag

![](https://img-blog.csdnimg.cn/034a81d17ebd4cb48d4d3c4354d16812.png)

```c++
pip install kitti2bag
kitti2bag -t 2011_09_26 -r 0002 raw_synced
```

如果出现缺少pykitti的话尝试着换一个python环境；

我用conda创建了一个python3.6的环境就可以了；ubuntu18也自带py3.6；切换py2.7和py3.6的方式如下：

ubuntu切换python版本_Gone_float的博客-CSDN博客_ubuntu 切换python版本

过了几天在python3.6下又不行了这真奇怪；新建了一个py2.7安了一些包又行了；镇邪门；

https://www.csdn.net/tags/OtDaUg3sMTEwNTktYmxvZwO0O0OO0O0O.html

查看一下bag
```c++
rqt_bag  kitti_2011_09_26_drive_0002_synced.bag
```

![](https://img-blog.csdnimg.cn/b073b10910954af8ae1602142e667aa5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMTExMTExMTExMTEyNDU0NTQ1,size_20,color_FFFFFF,t_70,g_se,x_16)
选择一个话题右键选择view可以将其播放

## 2、融合相机的aloam

主要思路就是，将雷达点投影到相机从而获得相机像素坐标。将rgb值赋值给点云即可
2.1、对源码的一些更改

从github下载代码

https://github.com/TianXiaoRui/color_map_loam

我对他的代码做了几处更改

1、源代码使用的是cv::mat定义的矩阵运算，我总是编译不通过于是将其换成了Eigen

2、源代码似乎是从kitti的odom里下载的数据；只有R*T；我根据rawdata的论文将其改成了

![](https://img-blog.csdnimg.cn/e9c4d066ba4c45b08e50f9b7d6e69000.png)

### 2.2、运行

打开一个终端
```c++
roscore
```

再打开一个终端

```c++
source devel/setup.bash
roslaunch aloam_velodyne aloam_velodyne_HDL_64.launch 
```

再打开一个终端播放bag
```c++
rosbag play kitti_2011_09_26_drive_0002_synced.bag
```

效果
![](https://img-blog.csdnimg.cn/f26572f6c0e24205893383e062f9e546.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMTExMTExMTExMTEyNDU0NTQ1,size_20,color_FFFFFF,t_70,g_se,x_16)

原图
![](https://img-blog.csdnimg.cn/95331094d5fa4ebabf1d9a52dd3f1c29.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMTExMTExMTExMTEyNDU0NTQ1,size_20,color_FFFFFF,t_70,g_se,x_16)

也可以下载所有的bag；这个很大，任老自动驾驶定位教程给的链接；
3、deeplabv3+ 语义分割

kitti语义分割在如下的目录；但是由于此数据集只有200张训练集图片比较少；所有用于训练并不够；不过kitti的语义分割数据集是按照cityspace的数据集制作的所有可以用cityspaces的训练模型来预测kitti的图像；(效果也得却很好)
![](https://img-blog.csdnimg.cn/2662d51d508b41e097d6326bd40347cf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAMTExMTExMTExMTEyNDU0NTQ1,size_20,color_FFFFFF,t_70,g_se,x_16)

 kitti数据集一共有19个类别；每个类别代表什么以及颜色如下：
 ```c++
 labels = [
    #       name                     id    trainId   category            catId     hasInstances   ignoreInEval   color
    Label(  'unlabeled'            ,  0 ,      255 , 'void'            , 0       , False        , True         , (  0,  0,  0) ),
    Label(  'ego vehicle'          ,  1 ,      255 , 'void'            , 0       , False        , True         , (  0,  0,  0) ),
    Label(  'rectification border' ,  2 ,      255 , 'void'            , 0       , False        , True         , (  0,  0,  0) ),
    Label(  'out of roi'           ,  3 ,      255 , 'void'            , 0       , False        , True         , (  0,  0,  0) ),
    Label(  'static'               ,  4 ,      255 , 'void'            , 0       , False        , True         , (  0,  0,  0) ),
    Label(  'dynamic'              ,  5 ,      255 , 'void'            , 0       , False        , True         , (111, 74,  0) ),
    Label(  'ground'               ,  6 ,      255 , 'void'            , 0       , False        , True         , ( 81,  0, 81) ),
    Label(  'road'                 ,  7 ,        0 , 'flat'            , 1       , False        , False        , (128, 64,128) ),
    Label(  'sidewalk'             ,  8 ,        1 , 'flat'            , 1       , False        , False        , (244, 35,232) ),
    Label(  'parking'              ,  9 ,      255 , 'flat'            , 1       , False        , True         , (250,170,160) ),
    Label(  'rail track'           , 10 ,      255 , 'flat'            , 1       , False        , True         , (230,150,140) ),
    Label(  'building'             , 11 ,        2 , 'construction'    , 2       , False        , False        , ( 70, 70, 70) ),
    Label(  'wall'                 , 12 ,        3 , 'construction'    , 2       , False        , False        , (102,102,156) ),
    Label(  'fence'                , 13 ,        4 , 'construction'    , 2       , False        , False        , (190,153,153) ),
    Label(  'guard rail'           , 14 ,      255 , 'construction'    , 2       , False        , True         , (180,165,180) ),
    Label(  'bridge'               , 15 ,      255 , 'construction'    , 2       , False        , True         , (150,100,100) ),
    Label(  'tunnel'               , 16 ,      255 , 'construction'    , 2       , False        , True         , (150,120, 90) ),
    Label(  'pole'                 , 17 ,        5 , 'object'          , 3       , False        , False        , (153,153,153) ),
    Label(  'polegroup'            , 18 ,      255 , 'object'          , 3       , False        , True         , (153,153,153) ),
    Label(  'traffic light'        , 19 ,        6 , 'object'          , 3       , False        , False        , (250,170, 30) ),
    Label(  'traffic sign'         , 20 ,        7 , 'object'          , 3       , False        , False        , (220,220,  0) ),
    Label(  'vegetation'           , 21 ,        8 , 'nature'          , 4       , False        , False        , (107,142, 35) ),
    Label(  'terrain'              , 22 ,        9 , 'nature'          , 4       , False        , False        , (152,251,152) ),
    Label(  'sky'                  , 23 ,       10 , 'sky'             , 5       , False        , False        , ( 70,130,180) ),
    Label(  'person'               , 24 ,       11 , 'human'           , 6       , True         , False        , (220, 20, 60) ),
    Label(  'rider'                , 25 ,       12 , 'human'           , 6       , True         , False        , (255,  0,  0) ),
    Label(  'car'                  , 26 ,       13 , 'vehicle'         , 7       , True         , False        , (  0,  0,142) ),
    Label(  'truck'                , 27 ,       14 , 'vehicle'         , 7       , True         , False        , (  0,  0, 70) ),
    Label(  'bus'                  , 28 ,       15 , 'vehicle'         , 7       , True         , False        , (  0, 60,100) ),
    Label(  'caravan'              , 29 ,      255 , 'vehicle'         , 7       , True         , True         , (  0,  0, 90) ),
    Label(  'trailer'              , 30 ,      255 , 'vehicle'         , 7       , True         , True         , (  0,  0,110) ),
    Label(  'train'                , 31 ,       16 , 'vehicle'         , 7       , True         , False        , (  0, 80,100) ),
    Label(  'motorcycle'           , 32 ,       17 , 'vehicle'         , 7       , True         , False        , (  0,  0,230) ),
    Label(  'bicycle'              , 33 ,       18 , 'vehicle'         , 7       , True         , False        , (119, 11, 32) ),
    Label(  'license plate'        , -1 ,       -1 , 'vehicle'         , 7       , False        , True         , (  0,  0,142) ),
]
```

 在deeplabv3+的github人家已经给出了训练好的cityspaces的模型下载下来直接对kitti进行预测就可以了
 https://github.com/VainF/DeepLabV3Plus-Pytorch
 
 




