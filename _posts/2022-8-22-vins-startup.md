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

