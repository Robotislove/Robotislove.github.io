---
layout: post
title: 激光雷达和相机联合标定汇总笔记
date: 2022-05-15
author: lau
tags: [calibration, Archive]
comments: true
toc: true
pinned: false
---

激光雷达和相机联合标定汇总笔记。

<!-- more -->

## 激光雷达和相机联合标定

### 相机标定

相机将三维世界中的坐标点（单位为米）映射到二维图像平面（单位为像素）的过程能够用一个几何模型进行描述。这个模型有很多种，其中最简单的称为针孔模型。针孔模型很常用，而且是有效的模型，它描述了一束光线通过针孔之后，在针孔背面投影成像的关系。遗憾的是，真实的针孔由于不能为快速曝光收集足够的光线，因此使用透镜来收集更多的光线。由于透镜的存在，会使得光线投影到成像平面的过程中会产生畸变。

Camera-LIDAR 联合标定分为 2 步：

1. 相机内参标定
2. 雷达相机外参标定

相机内参标定可以参考：https://blog.csdn.net/learning_tortosie/article/details/79901255。

本项目将利用摄像机标定（camera calibration），来矫正（数学方式）因使用透镜而给针孔模型带来的主要误差。摄像机标定的重要性还在于它是摄像机测量与真实三维世界测量的联系桥梁。摄像机标定的过程既给出**摄像机几何模型**，也给出**透镜的畸变**的模型。这两个模型定义了相机的**内参数**（intrinsic parameter）。

![image-20220513193839716](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220513193839716.png)



### 一、使用calibration_toolkit进行相机和三维激光雷达的联合标定(16线速腾雷达)

**系统配置：**

- Ubuntu18.04系统 安装
- ROS Melodic 安装
- opencv 3.2.0
- rslidar 16

calibration_toolkit，autoware 1.10.0版本中的标定工具箱单独编译安装。

calibration_toolkit是autoware1.10版本以前内置标定工具，autoware是与apllol类似的自动驾驶模拟系统。

网上很多教程都会先绍autoware的安装然后单独编译标定工具。我根据官网的教程安装autoware，可能是我网络原因和系统配置的原因，一直不能成功将git工程克隆到本地，走了很多弯路。实际上我发现如果只需要标定工具可以单独安装编译calibration_toolkit这个是大佬从autoware系统中单独“剥”出来的，可以直接安装使用。

#### 1、安装nlopt

网上很多安装教程都是ubuntu16.04的，我的系统是18.04的文章前面也已经介绍了系统配置，接下来介绍标定工具的比版以安装。

编译标定工具箱依赖于nlopt，需要先行安装，安装教程参考：https://github.com/stevengj/nlopt

#### 2、安装标定工具箱

标定工具箱安装教程参考：https://github.com/XidianLemon/calibration_camera_lidar，教程很详细有点ros基础的都会，把工程克隆到自己本地工程的src目录下编译即可。

可能出现的问题及解决方式：

此时直接运行标定程序，会报找不到可执行文件 calibration_camera_lidar的问题。因为我的系统是ubuntu18.04对应支持的ROS是melodic，因此需要对calibration_camera_lidar功能包下的CMakeLists.txt进行修改，添加ROS的melodic版本的支持。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210318162743197.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h0ZHhfeHR5,size_16,color_FFFFFF,t_70#pic_center)

home/x/catkin_ws/devel/lib/calibration_camera_lidar/calibration_toolkit: error while loading shared libraries: libnlopt.so.0: cannot open shared object file: No such file or directory
解决办法：

```shell
$ cd /etc/ld.so.conf.d
$ sudo touch libnlopt.conf
$ sudo gedit libnlopt.conf 
```

在libnlopt.conf文件中添加内容为：/usr/local/lib

```shell
$ sudo ldconfig
```

编译问题CMakeFiles/calibrationtoolkit.dir/CalibrationToolkit/calibrationtoolkit.cpp.o: In function nlopt::opt::get_errmsg() const': /usr/local/include/nlopt.hpp:516: undefined reference tonlopt_get_errmsg’ #10，我把nlopt安装在根目录下，实际上这个错误应该是我重复安装冲突了，解决办法：

```shell
$ sudo apt remove libnlopt-dev
```

在编译过程中，我遇到了opencv的问题，有些函数在opencv2和opencv3的表示不一样，我的系统安装了不同版本的opencv，所以我在CMakeLists.txt需要指定opencv的版本find_package(OpenCV 3.2.0 REQUIRED)避免冲突。

Could not find the required component ‘jsk_recognition_msgs’新开终端执行以下语句

```shell
$ sudo apt-get install ros-melodic-jsk-recognition-msgs
```

#### 3、开始标定（离线标定）

编译通过后可以启动标定程序
**（1）启动录制的数据包**
录制的数据包需要注意

- 标定板要同时在相机和雷达的视野内
- 这个工具订阅的是velodyne_points,所以需要指定rslidar_points:=/points_raw
- 不停调整标定板的角度，并且在同一个角度停留久一点，接下来标定捕获图片与点云振才能顺利。

```shell
rosbag play bagName.bag /rslidar_points:=/points_raw
```

**（2）启动标定工具**

```shell
roscore
source ~/catin_ws/devel/setup.bash
rosrun calibration_camera_lidar calibration_toolkit
```

出现以下选择图像以及标定类型的界面，选择你自己的相机以及标定类型，从标定类型看，该工具可以单独标定相机、雷达以及联合标定。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210318174952694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h0ZHhfeHR5,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210318174927281.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h0ZHhfeHR5,size_16,color_FFFFFF,t_70#pic_center)

**（3）执行标定步骤**

首先在pattern size设置标定板规格，我的标定板是6x8，小方格的size是0.245X0.245,这里给出下载链接[下载](http://wiki.ros.org/camera_calibration/Tutorials/MonocularCalibration?action=AttachFile&do=view&target=check-108.pdf)。

先调节点云界面，右上框的颜色，默认是黑色，点一下该框，按下B键，换成浅色调（便于观察）。

qweasd六个健用于旋转，将右边框角度调到与左边一致。

- step1.调好角度后，按下空格暂停数据播放，点击grab按钮
- step2.在右下图选择标定版所在的点云，点击calibrate标定按钮等待一会右边会出现外参矩阵等数据。没有出现下面的图像也没关系，重新播放在grab点云帧。
- step3.点击project按钮左下图中红点会散落到标定板，你可以根据标定版上红点投影情况以及重投影误差确认标定结果。

不断重复以上两个步骤，达到标定目的。

点击save保存标定结果，可以保存相机，雷达各自的内参标定结果以及联合标定结果（说明这个工具是可以用于单个传感器的标定的，联合标定不需要提前标定相机的内参）。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210318175356322.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h0ZHhfeHR5,size_16,color_FFFFFF,t_70#pic_center)

雷达-相机的联合标定结果如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021031819292173.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h0ZHhfeHR5,size_16,color_FFFFFF,t_70#pic_center)

### 二、使用lidar_camera_calibration进行相机和三维激光雷达的联合标定(16线速腾雷达)

作为无人驾驶中常用的传感器，激光雷达和相机都有自己特有的优势，然而它们也各有缺点和不足，现在主流的感知算法都向传感器融合方法靠拢，这就涉及到不同传感器之间的标定问题，本博文主要针对一个开源项目讲解激光雷达和相机之间的外参标定，即如何通过实验的方法得到激光雷达和相机之间的旋转平移矩阵。

项目地址： https://github.com/ankitdhall/lidar_camera_calibration。

项目描述：该功能包用来找到激光雷达和相机之间的旋转平移变换，该包使用aruco_ros和一个稍微修改的aruco_mapping包为依赖，lidar_camera_calibration/pointcloud_fusion可以用来融合点云和两个双目相机的信息，两个双目相机都和激光雷达进行过外参标定。
在使用该包之前，需要先做好相机内参标定，因为需要一些相机内参参数的一些信息。

#### 1.lidar_camera_calibration ROS包配置过程记录

依赖配置：
1、先将整个Github包clone下来，放在已经建好的ROS工作空间下，clone完成后生成文件夹lidar_camera_calibration；

2、将文件夹lidar_camera_calibration下的dependencies路径下的两个目录aruco_mapping和aruco_ros拷贝到ROS工作空间的src路径下；
准备编译：由于各个包之间有依赖关系，必须按以下顺序安装

3、先编译aruco_ros包

```shell
catkin_make -DCATKIN_WHITELIST_PACKAGES="aruco_ros"
```

4、再编译lidar_camera_calibration包

```shell
catkin_make -DCATKIN_WHITELIST_PACKAGES="lidar_camera_calibration"
```

此时可能会报错，缺少`velodyne_msgs`包，需要安装参考以下网站：
http://wiki.ros.org/velodyne/Tutorials/Getting Started with the HDL-32E

#### 2. 安装过程：

项目依赖velodyne ROS包，需要下载安装：

```shell
sudo apt-get install ros-kinetic-velodyne
```

5、再次编译`lidar_camera_calibration`包

```shell
catkin_make -DCATKIN_WHITELIST_PACKAGES="lidar_camera_calibration"
```

6、编译`aruco_mapping`包

```shell
catkin_make -DCATKIN_WHITELIST_PACKAGES="aruco_mapping"
```

#### 3. 运行过程

编译通过后，通过find_transform.launch文件启动所需节点和设置节点参数。

在运行launch文件之前，需要修改launch文件中的部分内容：

1、launch文件中默认将ArUco mapping节点注释掉了，可能需要启动该节点，并修改其中的重映射命令使得获取正确的相机图片话题：

```shell
 <remap from="/image_raw" to="/frontNear/left/image_raw"/>
```

修改相机标定文件路径参数（.ini格式文件），aruco标记的个数，标记尺寸（单位为米）

**注：**图片话题消息类型为sensor_msgs/Image类型

aruco mapping用来在图像中找到aruco marker，并发布6DOF的位姿。

http://wiki.ros.org/aruco_mapping

2、lidar_camera_calibration_rt话题是由aruco_mapping包发送出来的，包含了利用aruco marker计算出来的旋转平移矩阵数据启动标定节点前，需要保证aruco标记能够在相机画面内可见，并且需要使这些标记的ID号从左到右按升序排列，运行过程第一次需要手动标记line-segments。

**标记过程：**

每块板上都有四个线段，需要从左到右依次标记每个板上的直线段；此处对于每个板上的直线标记是在该线段的周围绘制一个四边形，每次点击一个点，并按下键盘确认，得到四个点之后就得到了包围一根直线的一个四边形，按这个方法把一个板上的直线从左上按顺时针依次标记。

标记完所有的直线段之后，就得到了最终的变换矩阵，中间值存储在conf/points.txt。

points.txt文件

**输出：** 3x1的平移矩阵和3x3的旋转矩阵，设置迭代多少次，就有多少个这样的矩阵生成。通过对n次迭代生成的平移矩阵求平均，即可得到最终的平移矩阵，通过对n个旋转矩阵转换成四元数，然后求平均，即可得到最终的旋转矩阵，从而得到最终的标定矩阵。


### 参考文献

相机内参标定方法

- http://wiki.ros.org/camera_calibration/Tutorials/MonocularCalibration
- https://blog.csdn.net/learning_tortosie/article/details/79901255

外参标定：lidar_camera_calibration

- https://github.com/ankitdhall/lidar_camera_calibration
- https://tumihua.cn/2019/02/20/698adb0e68a9eaa19a52d52c69f34bea.html
- https://blog.csdn.net/a2281965135/article/details/79785784

外参标定：but_calibration_camera_velodyne

- https://github.com/robofit/but_velodyne/tree/master/but_calibration_camera_velodyne
- https://blog.csdn.net/learning_tortosie/article/details/82385394

外参标定：Autoware

- https://blog.csdn.net/learning_tortosie/article/details/82347694
- https://blog.csdn.net/AdamShan/article/details/81670732
- https://blog.csdn.net/X_kh_2001/article/details/89163659?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task
- https://zhuanlan.zhihu.com/p/65226371
- https://blog.csdn.net/mxdsdo09/article/details/88370662
- http://s1nh.org/post/calib-velodyne-camera/

外参标定：Apoll

- https://blog.csdn.net/learning_tortosie/article/details/82351553

LIDAR-IMU 联合标定

- https://blog.csdn.net/Crystal_YS/article/details/90763723#lidarimulidar_align_12
- https://github.com/ethz-asl/lidar_align

Camera-IMU 联合标定

- https://blog.csdn.net/zhubaohua_bupt/article/details/80222321
- https://github.com/ethz-asl/kalibr
