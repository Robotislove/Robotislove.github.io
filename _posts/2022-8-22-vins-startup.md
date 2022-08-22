---
layout: post
title: Vins论文阅读和实践笔记
date: 2022-08-22
author: lau
tags: [SLAM, Blog]
comments: true
toc: true
pinned: false
---
Vins作为经典的VIO系统，这里记录它的原理和实践。

<!-- more -->

## 实践
### VINS-Mono代码解读——启动文件launch与参数配置文件yaml介绍

#### 前言
一般我们通过以下命令运行VINS-Mono跑MH_01数据集。
```shell
roslaunch vins_estimator euroc.launch 
roslaunch vins_estimator vins_rviz.launch
rosbag play YOUR_PATH_TO_DATASET/MH_01_easy.bag 
```

那么，VINS的启动和运行需要提供哪些参数，若是想用自己的传感器运行VINS，又需要修改哪些参数？本文将以euroc.launch和euroc_config.yaml为例对VINS中的启动文件launch和参数配置文件yaml进行详细介绍，并讨论自己在实验中调参的感受。欢迎大家一起讨论。

#### 启动文件launch

VINS在正常工作时一般存在三个节点：“/fature_tracker”、“/vins_estimator”、“/pose_graph”，而VINS的启动文件launch在文件夹VINS-Mono/vins_estimator/launch中，启动时直接读取euroc.launch同时打开三个节点。

首先介绍一下ROS中的launch文件——通过XML文件实现多节点的配置和启动（可自动启动ROS Master）。以及会用到的几个关键字：

- <launch>：launch文件中的根元素采用标签定义
- <node>：启动节点
比如：<node pkg=“package-name” type=“executable-name” name=“node-name”>
- <param>：设置ROS系统运行的参数，存储在参数服务器中
比如：<param name=“output_farme” value=“odem”/>
- <rosparam>：加载参数文件中的多个参数
比如：<rosparam file=“params.yaml” command=“load” ns=“params”/>
- <arg>：launch文件内部的局部变量，仅限于launch文件使用
比如<arg name=“arg-name” default=“arg-value”/>
参数调用：<param name=“foo” value="$(arg arg-name)"/>

-------------------------------------------------------------------------------------------------------------
	
euroc.launch文件的内容很少：
  
1、设置局部变量"config_path"，表示euroc_config.yaml的具体地址；
  
```shell
<arg name="config_path" default = "$(find feature_tracker)/../config/euroc/euroc_config.yaml" />
	  <arg name="vins_path" default = "$(find feature_tracker)/../config/../" />
```
  
设置局部变量"vins_path"，在parameters.cpp中使用鱼眼相机mask中用到：
  
```shell
  
std::string VINS_FOLDER_PATH = readParam<std::string>(n, "vins_folder");
if (FISHEYE == 1)
        FISHEYE_MASK = VINS_FOLDER_PATH + "config/fisheye_mask.jpg";
```

2、启动"feature_tracker"节点，在该节点中需要读取参数文件，地址为config_file，即config_path；

```shell
 <node name="feature_tracker" pkg="feature_tracker" type="feature_tracker" output="log">
 <param name="config_file" type="string" value="$(arg config_path)" />
 <param name="vins_folder" type="string" value="$(arg vins_path)" />
 </node>
```

3、启动"vins_estimator"节点，内容同上；
	
```shell
<node name="vins_estimator" pkg="vins_estimator" type="vins_estimator" output="screen">
<param name="config_file" type="string" value="$(arg config_path)" />
<param name="vins_folder" type="string" value="$(arg vins_path)" />
</node>
```
4、启动"pose_graph"节点，除了参数配置文件的地址外还设置了4个参数：
	
```
    <node name="pose_graph" pkg="pose_graph" type="pose_graph" output="screen">
        <param name="config_file" type="string" value="$(arg config_path)" />
        <param name="visualization_shift_x" type="int" value="0" />
        <param name="visualization_shift_y" type="int" value="0" />
        <param name="skip_cnt" type="int" value="0" />
        <param name="skip_dis" type="double" value="0" />
    </node>
```
	
visualization_shift_x和visualization_shift_y表示在进行位姿图优化后，对得到的位姿在x坐标和y坐标的偏移量（一般都设为0）；
```shell
geometry_msgs::PoseStamped pose_stamped;
pose_stamped.header.stamp = ros::Time(cur_kf->time_stamp);
pose_stamped.header.frame_id = "world";
pose_stamped.pose.position.x = P.x() + VISUALIZATION_SHIFT_X;
pose_stamped.pose.position.y = P.y() + VISUALIZATION_SHIFT_Y;
pose_stamped.pose.position.z = P.z();
```

skip_cnt在pose_graph_node的process()中，表示每隔skip_cnt个图像帧才进行一次处理；

skip_dis也在pose_graph_node的process()中，目的是将距上一帧的时间间隔超过SKIP_DIS的图像创建为位姿图中的关键帧。

至此launch文件的内容就结束了。

------------------------------------------------------------------------------------------------------------

#### 参数配置文件yaml

从launch启动文件中可以看到，每个节点都会读取参数，例如在feature_traker_node.cpp中的main()通过以下函数实现参数读取：

```c++
void readParameters(ros::NodeHandle &n)
{
    std::string config_file;
    config_file = readParam<std::string>(n, "config_file");
    cv::FileStorage fsSettings(config_file, cv::FileStorage::READ);
    if(!fsSettings.isOpened())
    {
        std::cerr << "ERROR: Wrong path to settings" << std::endl;
    }
    //。。。
}
```
下面将具体介绍euroc_config.yaml每个参数的具体意义。

##### 1、通用参数
	
1）接收IMU和图像的topic，其中image_topic在节点/fature_tracker中被订阅，以进行角点的光流跟踪；imu_topic在节点/vins_estimator中被订阅，以进行IMU预积分。

2）output_path为输出文件的地址，输出以下内容：VINS的运行轨迹"/vins_result_no_loop.csv"与"/vins_result_loop.csv"，相机与IMU的外参估计 “/extrinsic_parameter.csv”（如果estimate_extrinsic不为0即需要对外参进行估计的话）。注意如果该文件夹不存在则不输出。

```c++
imu_topic: "/imu0"
image_topic: "/cam0/image_raw"
output_path: "/home/tony-ws1/output/"
```

##### 2、相机的基础信息
包括：

相机模型：PINHOLE（针孔相机）、KANNALA_BRANDT（一种常用的鱼眼相机模型）等。
图像的宽和高、相机的畸变系数（如果是鱼眼相机还要求mu、mv、u0、v0）、相机的内参。
相机的内参数据建议进行标定以获得较准确结果，以降低之后调参的难度。

```c++
model_type: PINHOLE
camera_name: camera
image_width: 752
image_height: 480
distortion_parameters:
   k1: -2.917e-01
   k2: 8.228e-02
   p1: 5.333e-05
   p2: -1.578e-04
projection_parameters:
   fx: 4.616e+02
   fy: 4.603e+02
   cx: 3.630e+02
   cy: 2.481e+02
```
	
##### 3、imu和相机之间的外参
这里需要注意的是其旋转矩阵ric、平移向量tic都是表示从camera坐标系到IMU坐标系的变换。

estimate_extrinsic：=0表示这外参已经是准确的了，之后不需要优化；=1表示外参只是一个估计值，后续还需要将其作为初始值放入非线性优化中；=2表示不知道外参，需要进行标定，在estimator.cpp的函数processImage()中调用了CalibrationExRotation()进行外参标定。

个人在实验时最直观的感受是：这个外参对VINS输出的精度和鲁棒性具有非常大的影响！尤其是初始化阶段，如果外参不准会对IMU的预积分和图像的SFM对齐以得到的尺度有很大偏差。强烈建议进行手眼标定，具体可以使用Kalibr工具进行IMU和相机的离线外参标定。
```shell
# Extrinsic parameter between IMU and Camera.
estimate_extrinsic: 0   # 0  Have an accurate extrinsic parameters. We will trust the following imu^R_cam, imu^T_cam, don't change it.
                        # 1  Have an initial guess about extrinsic parameters. We will optimize around your initial guess.
                        # 2  Don't know anything about extrinsic parameters. You don't need to give R,T. We will try to calibrate it. Do some rotation movement at beginning.                        
#If you choose 0 or 1, you should write down the following matrix.
#Rotation from camera frame to imu frame, imu^R_cam
extrinsicRotation: !!opencv-matrix
   rows: 3
   cols: 3
   dt: d
   data: [0.0148655429818, -0.999880929698, 0.00414029679422,
           0.999557249008, 0.0149672133247, 0.025715529948, 
           -0.0257744366974, 0.00375618835797, 0.999660727178]
#Translation from camera frame to imu frame, imu^T_cam
extrinsicTranslation: !!opencv-matrix
   rows: 3
   cols: 1
   dt: d
   data: [-0.0216401454975,-0.064676986768, 0.00981073058949]
```

##### 4、在节点/feature_traker中需要用到的参数

```shell
#feature traker paprameters
max_cnt: 150            # max feature number in feature tracking
min_dist: 30            # min distance between two features 
freq: 10                # frequence (Hz) of publish tracking result. At least 10Hz for good estimation. If set 0, the frequence will be same as raw image 
F_threshold: 1.0        # ransac threshold (pixel)
show_track: 1           # publish tracking image as topic
equalize: 1             # if image is too dark or light, trun on equalize to find enough features
fisheye: 0              # if using fisheye, trun on it. A circle mask will be loaded to remove edge noisy points
```

1）max_cnt为进行特征光流跟踪时保持的特征点数量，具体在FeatureTracker::readImage()，通过当前帧成功跟踪的特征点数，计算是否需要提取新的特征点，如果需要则用过goodFeaturesToTrack()提取。

max_cnt值增大在一定条件下会提高跟踪的鲁棒性（但在环境特征本来就不丰富的地方反而会提取不够鲁棒的特征点），但是会增加之后所有算法的运算时间，例如在我调试时取250已经无法达到实时性。
```c++
int n_max_cnt = MAX_CNT - static_cast<int>(forw_pts.size());
if (n_max_cnt > 0)
        {
            //...
            cv::goodFeaturesToTrack(forw_img, n_pts, MAX_CNT - forw_pts.size(), 0.01, MIN_DIST, mask);
        }
```
	
2）min_dist为两个相邻特征之间像素的最小间隔，目的是保证图像中均匀的特征分布。在FeatureTracker::setMask()中应用，该函数的作用是对跟踪点进行排序并去除密集点。
min_dist的实现原理是在mask中将当前特征点周围半径为MIN_DIST的区域设置为0，后面便不再选取该区域内的点。

```c++
cv::circle(mask, it.second.first, MIN_DIST, 0, -1);
```
这个参数取大一些，比如30，可以保证图像的特征点分布非常均匀，有利于图像全局的光流跟踪；如果在特征明显的场合下可以适当取小一些，保证跟踪的特征都聚集在特征明显的地方。

3）freq控制图像跟踪的发布频率。在回调函数img_callback()中应用，通过判断判断间隔时间内的发布次数控制发布频率。一般根据相机的运行速度以及其他参数调完后的实时运行情况进行调整。

```c++
if (round(1.0 * pub_count / (img_msg->header.stamp.toSec() - first_image_time)) <= FREQ)
    {
        PUB_THIS_FRAME = true;
        if (abs(1.0 * pub_count / (img_msg->header.stamp.toSec() - first_image_time) - FREQ) < 0.01 * FREQ)
        {
            first_image_time = img_msg->header.stamp.toSec();
            pub_count = 0;
        }
    }
    else
        PUB_THIS_FRAME = false;
```
	
4）F_threshold为ransac算法的门限值，在FeatureTracker::rejectWithF()通过计算基本矩阵去除图像特征跟踪的外点时使用，一般不修改。

```c++
cv::findFundamentalMat(un_cur_pts, un_forw_pts, cv::FM_RANSAC, F_THRESHOLD, 0.99, status);
```
	
5）show_track：将经过特征跟踪的图像进行发布，需要。

6）equalize: 如果图像整体太暗或者太亮则需要进行直方图均衡化，在FeatureTracker::readImage()中进行自适应直方图均衡化，需要。

```c++
if (EQUALIZE)
    {
        cv::Ptr<cv::CLAHE> clahe = cv::createCLAHE(3.0, cv::Size(8, 8));
        TicToc t_c;
        clahe->apply(_img, img);
        ROS_DEBUG("CLAHE costs: %fms", t_c.toc());
    }
    else
        img = _img;
```

7）fisheye：鱼眼相机一般需要圆形的mask，以去除外部噪声点。mask图在config文件夹中。
	
##### 5、在节点/vins_estimator中需要用到的参数

```c++
#optimization parameters
max_solver_time: 0.04  # max solver itration time (ms), to guarantee real time
max_num_iterations: 8   # max solver itrations, to guarantee real time
keyframe_parallax: 10.0 # keyframe selection threshold (pixel)
```
	
max_solver_time和max_num_iterations分别定义了ceres优化器的最大迭代时间和最大迭代次数，以保证实时性。在Estimator::optimization()中使用，并根据不同的边缘化方案而有改变。

```c++
options.linear_solver_type = ceres::DENSE_SCHUR;
options.trust_region_strategy_type = ceres::DOGLEG;
options.max_num_iterations = NUM_ITERATIONS;
if (marginalization_flag == MARGIN_OLD)
	options.max_solver_time_in_seconds = SOLVER_TIME * 4.0 / 5.0;
else
	options.max_solver_time_in_seconds = SOLVER_TIME;
```
	
keyframe_parallax定义了关键帧的选择阈值。在FeatureManager::addFeatureCheckParallax()中通过计算每一个点跟踪次数和它在次新帧和次次新帧间的视差确定是否是关键帧。这个参数影响着算法中关键帧的个数。
	
其中keyframe_parallax为像素坐标系，MIN_PARALLAX为归一化相机坐标系。
```c++
MIN_PARALLAX = fsSettings["keyframe_parallax"];
MIN_PARALLAX = MIN_PARALLAX / FOCAL_LENGTH;

    for (auto &it_per_id : feature)
    {
        if (it_per_id.start_frame <= frame_count - 2 &&
            it_per_id.start_frame + int(it_per_id.feature_per_frame.size()) - 1 >= frame_count - 1)
        {
            parallax_sum += compensatedParallax2(it_per_id, frame_count);
            parallax_num++;
        }
    }
    if (parallax_num == 0)
    {
        return true;
    }
    else
    {
        return parallax_sum / parallax_num >= MIN_PARALLAX;
    }
```
##### 6、IMU参数
```c++
#imu parameters       The more accurate parameters you provide, the better performance
acc_n: 0.08          # accelerometer measurement noise standard deviation. #0.2   0.04
gyr_n: 0.004         # gyroscope measurement noise standard deviation.     #0.05  0.004
acc_w: 0.00004         # accelerometer bias random work noise standard deviation.  #0.02
gyr_w: 2.0e-6       # gyroscope bias random work noise standard deviation.     #4.0e-5
g_norm: 9.81007     # gravity magnitude
```
包括加速度计和陀螺仪测量值的噪声标准差和随机偏置的导数标准差，以及重力加速度大小。因为在进行IMU预积分时我们假设传感器的噪声noise服从高斯分布，偏置bias的导数服从高斯分布。
这部分的参数对结果的精度有一定影响，建议读取IMU的datasheet中具体参数或者进行IMU标定直接确定，以降低之后调参的难度。


##### 7、闭环用到的参数
```c++
#loop closure parameters
loop_closure: 1                    # start loop closure
load_previous_pose_graph: 0        # load and reuse previous pose graph; load from 'pose_graph_save_path'
fast_relocalization: 0             # useful in real-time and large project
pose_graph_save_path: "/home/tony-ws1/output/pose_graph/" # save and load path
```
load_previous_pose_graph 为新功能：地图重用，通过载入先前的位姿图文件实现。
	
fast_relocalization常用于高实时性或者地图非常大、持续时间长的地方，得到的结果精度会降低。一般实验时先不打开。

##### 8、在线时差校准
这是VINS的新功能：因为大多数相机和IMU不是时间同步的，所以将estimate_td设置为1可以在线估计相机和IMU之间的时间差。
	
参数td在Estimator::optimization()里放在视觉残差函数中进行非线性优化。

```c++
#unsynchronization parameters
estimate_td: 0                      # online estimate time offset between camera and imu
td: 0.0                             # initial value of time offset. unit: s. readed image clock + td = real image clock (IMU clock)
```
##### 9、支持卷帘相机
这也是VINS的新功能：将rolling_shutter设置为0表示全局曝光相机；设置为1表示卷帘曝光相机，此时需要从传感器的datasheet中得到rolling_shutter_tr的值。不要使用网络摄像头webcam，不适合用。
```c++
#rolling shutter parameters
rolling_shutter: 0                  # 0: global shutter camera, 1: rolling shutter camera
rolling_shutter_tr: 0               # unit: s. rolling shutter read out time per frame (from data sheet). 
```
以上两个功能具体在以下代码中实现：
pts_i_td表示处理时间同步误差和Rolling shutter时间后，角点在归一化平面的坐标。
TR / ROW * row_i就是相机 rolling 到这一行时所用的时间
```c++
pts_i_td = pts_i - (td - td_i + TR / ROW * row_i) * velocity_i;
pts_j_td = pts_j - (td - td_j + TR / ROW * row_j) * velocity_j;
```
##### 10、RVIZ可视化参数
```c++
#visualization parameters
save_image: 1                   # 保存在位姿图中的图像，为0则关闭
visualize_imu_forward: 0        # 输出IMU前向递推的角度预测值，一般用于低延迟和高频率的应用要求下
visualize_camera_size: 0.4      # 在RVIZ显示中相机模型的大小
```
	
其中visualize_imu_forward在pose_graph_node中的imu_forward_callback()即imu前向递推的回调函数中用到，如果为1则接收IMU前向递推的角度预测值，经回环的偏移矫正并转换为世界坐标系下的相机姿态，发布；为0则什么都不做，不发布。

#### 总结
拿到一个新的相机-IMU组件，在安装对应的驱动和ROS包后，首先需要对yaml文件的传感器信息部分进行以下修改（这部分信息尽可能准确）：
	
1、确定相机和IMU各自发布的topic；
	
2、标定相机、IMU的内参，从相机到IMU的外参；
	
3、是否是硬件时间同步，如果不是的话需要进行在线同步时差估计；
	
4、相机是卷帘曝光还是全局曝光。
	
之后根据应用场景的要求，确定是否需要进行回环检测、快速重定位或是读取之前位姿图等功能，以及用于RVIZ可视化的参数。根据在实际场景运行后的情况再对/feature_traker和/vins_estimator中的参数进行调整，以获得较好的结果。当然这只是在实验阶段，要使其能真正落地应用，还需要进行不断的测试和优化。








