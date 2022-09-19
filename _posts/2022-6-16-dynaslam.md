---
layout: post
title: 动态环境下SLAM学习笔记汇总
date: 2022-06-16
author: lau
tags: [动态SLAM, Archive]
comments: true
toc: false
pinned: false
---
动态环境SLAM是目前slam方向的一个热门研究领域。

<!-- more -->

## Related survey papers:

- [**A survey: which features are required for dynamic visual simultaneous localization and mapping?**](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8285453/pdf/42492_2021_Article_86.pdf). Zewen Xu,CAS. 2021
- [**State of the Art in Real-time Registration of RGB-D Images**](https://cg.cs.uni-bonn.de/aigaion2root/attachments/StateoftheArtinReal-timeRegistrationofRGB-DImages.pdf). Stotko, Patrick. University of Bonn. 2016
- [**Visual SLAM and Structure from Motion in Dynamic Environments: A Survey**](https://dl.acm.org/doi/pdf/10.1145/3177853).  University of Oxford. 2018
- [**State of the Art on 3D Reconstruction with RGB-D Cameras**](https://www.cg.informatik.uni-siegen.de/data/Publications/2018/star1009-main.pdf). Michael Zollhöfer. Stanford University. 2018
## Related Article:

- https://www.zhihu.com/question/47817909
- [deeplabv3+ slam](https://www.shangyexinzhi.com/article/4766362.html)
- [SOF-SLAM：一种面向动态环境的语义视觉SLAM（2019，JCR Q1， 4.076）](https://blog.csdn.net/ainitutu/article/details/107540299)
## 中文工作汇总

- 1.DynaSLAM（IROS 2018）
  - 论文：DynaSLAM: Tracking, Mapping and Inpainting in Dynamic Scenes
  代码：https://github.com/BertaBescos/DynaSLAM
  主要思想：（语义+几何）
  1.使用Mask-CNN进行语义分割；
  2.在low-cost Tracking阶段将动态区域（人）剔除，得到初始位姿；
  3.多视图几何方法判断外点，通过区域增长法生成动态区域；
  4.代码中将多视图几何的动态区域与语义分割人的区域全都去除，将mask传给orbslam进行跟踪；
  5.背景修复，包括RGB图和深度图。
  **创新点：** 语义分割无法识别移动的椅子，需要多视图几何的方法进行补充
  **讨论：** DynaSLAM与下面的DS-SLAM是经典的动态slam系统，代码实现都很简洁。Dyna-SLAM的缺点在于：1.多视图几何方法得到的外点，在深度图上通过区域增长得到动态区域，只要物体上存在一个动态点，整个物体都会被“增长”成为动态 2.其将人的区域以及多视图几何方法得到的区域都直接去掉，与论文不符。
  ![](https://img-blog.csdnimg.cn/20210530120315904.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  ![](https://img-blog.csdnimg.cn/20210530120328187.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FpbnFpbnhpYW5zaGVuZw==,size_16,color_FFFFFF,t_70)

- 2.DS-SLAM（IROS 2018， 清华大学）
  - 论文：DS-SLAM: A Semantic Visual SLAM towards Dynamic Environments

代码：https://github.com/ivipsourcecode/DS-SLAM
主要思想：（语义+几何）
1.SegNet进行语义分割（单独一个线程）；
2.对于前后两帧图像，通过极线几何检测外点；
3.如果某一物体外点数量过多，则认为是动态，剔除；
4.建立了语义八叉树地图。
讨论：这种四线程的结构以及极线约束的外点检测方法得到了很多论文的采纳，其缺点在于：1.极线约束的外点检测方法并不能找到所有外点，当物体沿极线方向运动时这种方法会失效 2.用特征点中的外点的比例来判断该物体是否运动，这用方法存在局限性，特征点的数量受物体纹理的影响较大 3.SegNet是2016年剔除的语义分割网络，分割效果有很大提升空间，实验效果不如Dyna-SLAM
![](https://img-blog.csdnimg.cn/20210530120144856.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FpbnFpbnhpYW5zaGVuZw==,size_16,color_FFFFFF,t_70)

- 3.Detect-SLAM（2018 IEEE WCACV， 北京大学）

论文：Detect-SLAM: Making Object Detection and SLAM Mutually Beneficial
代码：https://github.com/liadbiz/detect-slam
主要思想：
目标检测的网络并不能实时运行，所以只在关键帧中进行目标检测，然后通过特征点的传播将其结果传播到普通帧中
1.只在关键帧中用SSD网络进行目标检测（得到的是矩形区域及其置信度），图割法剔除背景，得到更加精细的动态区域；
2.在普通帧中，利用feature matching + matching point expansion两种机制，对每个特征点动态概率传播，至此得到每个特征点的动态概率；
3.object map帮助提取候选区域。

![](https://img-blog.csdnimg.cn/20210530120440760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FpbnFpbnhpYW5zaGVuZw==,size_16,color_FFFFFF,t_70)

- 4.VDO-SLAM（arXiv 2020）

论文：VDO-SLAM: A Visual Dynamic Object-aware SLAM System
代码： https://github.com/halajun/vdo_slam
主要思想：
1.运动物体跟踪，比较全的slam+运动跟踪的系统；
2.光流+语义分割。
![](https://img-blog.csdnimg.cn/20210530120519434.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FpbnFpbnhpYW5zaGVuZw==,size_16,color_FFFFFF,t_70)

- 5.Co-Fusion（ICRA 2017）

论文：Co-Fusion: Real-time Segmentation, Tracking and Fusion of Multiple Objects
代码：https://github.com/martinruenz/co-fusion
**主要思想：** 学习和维护每个物体的3D模型，并通过随时间的融合提高模型结果。这是一个经典的系统，很多论文都拿它进行对比
![](https://img-blog.csdnimg.cn/20210530120626236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FpbnFpbnhpYW5zaGVuZw==,size_16,color_FFFFFF,t_70)

- 6.Learning Rigidity in Dynamic Scenes with a Moving Camera for 3D Motion Field Estimation（2018，ECCV，NVIDIA）

论文：Learning Rigidity in Dynamic Scenes with a Moving Camera for 3D Motion Field Estimation
代码：https://github.com/NVlabs/learningrigidity.git
主要思想：
1.RTN网络用于计算位姿以及刚体区域，PWC网络用于计算稠密光流；
2.基于以上结果，估计刚体区域的相对位姿；
3.计算刚体的3D场景流；
此外还开发了一套用于生成半人工动态场景的工具REFRESH。

![](https://www.guyuehome.com//Uploads/Editor/202106/20210618_83839.jpg)

- 7.ReFusion（2019 IROS）

论文：ReFusion: 3D Reconstruction in Dynamic Environments for RGB-D Cameras Exploiting Residuals

代码：https://github.com/PRBonn/refusion
主要思想：
主流的动态slam方法需要用神经网络进行分实例分割，此过程需要预先定义可能动态的对象并在数据集上进行大量的训练，使用场景受到很大的限制。而ReFusion则使用纯几何的方法分割动态区域，具体的：在KinectFusion稠密slam系统的基础上，计算每个像素点的残差，通过自适应阈值分割得到大致动态区域，形态学处理得到最终动态区域，与此同时，可得到静态背景的TSDF地图。
讨论：为数不多的不使用神经网络的动态slam系统
![](https://www.guyuehome.com//Uploads/Editor/202106/20210618_48914.jpg)

- 8.RGB_D-SLAM-with-SWIAICP（2017）

论文：RGB-D SLAM in Dynamic Environments Using Static Point Weighting
代码：https://github.com/VitoLing/RGB_D-SLAM-with-SWIAICP
主要思想：
1.仅使用前景的边缘点进行跟踪（ Foreground Depth Edge Extraction）；
2.每隔n帧插入关键帧，通过当前帧与关键帧计算位姿；
3.通过投影误差计算每个点云的静态-动态质量，为下面的IAICP提供每个点云的权重；
4.提出了融合灰度信息的ICP算法–IAICP，用于计算帧与帧之间的位姿。
**讨论：** 前景物体的边缘点能够很好地表征整个物体，实验效果令人耳目一新。但是文中所用的ICP算法这并不是边缘slam常用的算法，边缘slam一般使用距离变换（DT）描述边缘点的误差，其可以避免点与点之间的匹配。论文Robust RGB-D visual odometry based on edges and points提出了一种很有意思的方案，用特征点计算位姿，用边缘点描述动态区域，很好地汲取了二者的优势。

![](https://img-blog.csdnimg.cn/20210530120722763.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FpbnFpbnhpYW5zaGVuZw==,size_16,color_FFFFFF,t_70)

- 9.RDS-SLAM（2021，Access）

论文：RDS-SLAM: Real-Time Dynamic SLAM Using Semantic Segmentation Methods
代码：https://github.com/yubaoliu/RDS-SLAM.git
主要思想：克服不能实时进行语义分割的问题
1.选择最近的关键帧进行语义分割；
2.基于贝叶斯的概率传播；
3.通过上一帧和局部地图得到当前帧的外点；
4.根据运动概率加权计算位姿。

![](https://www.guyuehome.com//Uploads/Editor/202106/20210619_99878.jpg)

- 靠谱的工作
  - DS-SLAM Dynaslam:https://github.com/zhuhu00/DS-SLAM_modify； https://blog.csdn.net/qq_41623632/article/details/112911046；
## Dynamic Object Detection and *Removal*

- Pfreundschuh, Patrick, et al. “**[Dynamic Object Aware LiDAR SLAM Based on Automatic Generation of Training Data.](http://arxiv.org/abs/2104.03657)**” (*ICRA* *2021*)
  - **ETH ASL**,code, [video](https://youtu.be/LcDxd97r1Gc), [**dataset**](https://projects.asl.ethz.ch/datasets/doals), Lidar
  - 作者基于deep-learning（3D-MiniNet网络）进行实时3D动态物体检测，滤除动态物体后的点云被喂给LOAM，进行常规的激光SLAM。提供了学习的方法是属于无监督的方法。
  
- Canovas Bruce, et al. “[**Speed and Memory Efficient Dense RGB-D SLAM in Dynamic Scenes.**](https://doi.org/10.1109/IROS45743.2020.9341542)” (*IROS* 2020)
  - **GIPSA-lab**, [code](https://github.com/BruceCanovas/supersurfel_fusion), [video](https://youtu.be/hzzVVHUAO74)
  
- Yuan Xun and Chen Song. “[**SaD-SLAM: A Visual SLAM Based on Semantic and Depth Information.**](http://ras.papercept.net/images/temp/IROS/files/0092.pdf)” *(IROS 2020)*
  - **USTC**, code, video
  
- Dong, Erqun, et al. “[**Pair-Navi: Peer-to-Peer Indoor Navigation with Mobile Visual SLAM.**](https://cswu.me/papers/infocom19_pairnavi.pdf)” (ICCC 2019)
  - Tsinghua, [Code](https://github.com/Horacehxw/Dynamic_ORB_SLAM2), Video, [Slides](https://slidetodoc.com/pairnavi-peertopeer-indoor-navigation-with-mobile-visual-slam/).
  
- Ji Tete, et al. “**[Towards Real-Time Semantic RGB-D SLAM in Dynamic Environments](https://arxiv.org/abs/2104.01316v1)**.” (ICRA 2021)

- Palazzolo Emanuele, et al. “**[ReFusion: 3D Reconstruction in Dynamic Environments for RGB-D Cameras Exploiting Residuals.](https://arxiv.org/abs/1905.02082v3)**” (IROS 2019)
  - [code](https://github.com/PRBonn/refusion),[video](https://youtu.be/1P9ZfIS5-p4).University of Bonn.
  
- Arora Mehul, et al. ***[Mapping the Static Parts of Dynamic Scenes from 3D LiDAR Point Clouds Exploiting Ground Segmentation](https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/arora2021ecmr.pdf)***. p. 6.

- Chen Xieyuanli, et al. “**[Moving Object Segmentation in 3D LiDAR Data: A Learning-Based Approach Exploiting Sequential Data](https://www.ipb.uni-bonn.de/pdfs/chen2021ral-iros.pdf)**.” *IEEE Robotics and Automation Letters*, 2021
  - [code](https://github.com/PRBonn/LiDAR-MOS). [video](https://www.youtube.com/watch?v=NHvsYhk4dhw). University of Bonn.
  
- Zhang Tianwei, et al. “[**FlowFusion: Dynamic Dense RGB-D SLAM Based on Optical Flow.**](http://arxiv.org/abs/2003.05102.)”(ICRA 2020)
  - code. [video](https://youtu.be/6yPGDdwKFLA).
  
- Zhang Tianwei, et al. “**[AcousticFusion: Fusing Sound Source Localization to Visual SLAM in Dynamic Environments.](http://arxiv.org/abs/2108.01246)**”,IROS 2021
  - [video](https://youtu.be/8eNikzp9LIQ). 结合声音信号
  
- 1. Liu Yubao and Miura Jun. “[**RDS-SLAM: Real-Time Dynamic SLAM Using Semantic Segmentation Methods.**](https://doi.org/10.1109/ACCESS.2021.3050617)” *IEEE Access* 2021

  2. Liu Yubao and Miura Jun. “RDMO-SLAM: Real-Time Visual SLAM for Dynamic Environments Using Semantic Label Prediction With Optical Flow.” *IEEE Access*, vol. 9, 2021, pp. 106981–97. *IEEE Xplore*, https://doi.org/10.1109/ACCESS.2021.3100426.

  - [code](https://github.com/yubaoliu/RDS-SLAM.git
    ), video.

- Cheng Jiyu, et al. “**Improving Visual Localization Accuracy in Dynamic Environments Based on Dynamic Region Removal.**” *IEEE Transactions on Automation Science and Engineering*, vol. 17, no. 3, July 2020, pp. 1585–96. *IEEE Xplore*, https://doi.org/10.1109/TASE.2020.2964938.

- Soares João Carlos Virgolino, et al. “**[Crowd-SLAM: Visual SLAM Towards Crowded Environments Using Object Detection.](https://doi.org/10.1007/s10846-021-01414-1)**” *Journal of Intelligent & Robotic Systems* 2021

  - [code](https://github.com/virgolinosoares/Crowd-SLAM), video

- Kaveti Pushyami and Singh Hanumant. “**[A Light Field Front-End for Robust SLAM in Dynamic Environments.](http://arxiv.org/abs/2012.10714)**”.

- Kuen-Han Lin and Chieh-Chih Wang. “**[Stereo-Based Simultaneous Localization, Mapping and Moving Object Tracking.](https://doi.org/10.1109/IROS.2010.5649653)**” IROS 2010

- Fu, H.; Xue, H.; Hu, X.; Liu, B. **[LiDAR Data Enrichment by Fusing Spatial and Temporal Adjacent Frames](https://doi.org/10.3390/rs13183640)**. *Remote Sens.* **2021**, *13*, 3640. 

- Qian, Chenglong, et al. ***RF-LIO: Removal-First Tightly-Coupled Lidar Inertial Odometry in High Dynamic Environments***. p. 8. IROS2021, **XJTU**

- K. Minoda, F. Schilling, V. Wüest, D. Floreano, and T. Yairi, “**[VIODE: A Simulated Dataset to Address the Challenges of Visual-Inertial Odometry in Dynamic Environments,](https://doi.org/10.1109/LRA.2021.3058073)**”RAL 2021

  - 动态环境的数据集，包括了静态，动态等级的场景，感觉适合用来作为验证。
  - 东京大学，[code](https://github.com/kminoda/VIODE)
  
- W. Dai, Y. Zhang, P. Li, Z. Fang, and S. Scherer, “**[RGB-D SLAM in Dynamic Environments Using Point Correlations](https://doi.org/10.1109/TPAMI.2020.3010942)**,” *IEEE Transactions on Pattern Analysis and Machine Intelligence*, pp. 1–1, 2020

  - 浙大，使用点的关联进行去除。

- C. Huang, H. Lin, H. Lin, H. Liu, Z. Gao, and L. Huang, “YO-VIO: Robust Multi-Sensor Semantic Fusion Localization in Dynamic Indoor Environments,” in 2021 International Conference on Indoor Positioning and Indoor Navigation (IPIN), 2021.
  - 使用yolo和光流对运动对象进行判断，去除特征点后进行定位
  - VIO的结合

- Dynamic-VINS：RGB-D Inertial Odometry for a Resource-restricted Robot in Dynamic Environments. 
  - 分割+运动点置信度，[code](https://github.com/HITSZ-NRSL/Dynamic-VINS),[video](https://www.bilibili.com/video/BV1bF411t7mx)



## Dynamic Object Detection and ***Tracking***

- “**[AirDOS: Dynamic SLAM benefits from Articulated Objects,](http://arxiv.org/abs/2109.09903)**” Qiu Yuheng, et al. 2021(Arxiv)
  - [code](https://github.com/haleqiu/airdos)(TBA), [Paper](https://arxiv.org/abs/2109.09903), Video. **CMU RI**, Vision.
  
- “[**DOT: Dynamic Object Tracking for Visual SLAM**](http://arxiv.org/abs/2010.00052).” Ballester, Irene, et al.(ICRA 2021)
  - code, [video](https://youtu.be/9hWChyQGKJk), **University of Zaragoza**, Vision
  
- Liu Yubao and Miura Jun. “[**RDMO-SLAM: Real-Time Visual SLAM for Dynamic Environments Using Semantic Label Prediction With Optical Flow**](https://doi.org/10.1109/ACCESS.2021.3100426).” *IEEE Access*. 

- Kim Aleksandr, et al. “[**EagerMOT: 3D Multi-Object Tracking via Sensor Fusion.**](http://arxiv.org/abs/2104.14682)” (*ICRA 2021*)
  - **TUM**, [code](https://github.com/aleksandrkim61/EagerMOT), [video](https://youtu.be/k8pKpvbenoM), Sensor Fusion.
  
- 1. Shan, Mo, et al. “**[OrcVIO: Object Residual Constrained Visual-Inertial Odometry.](http://moshan.cf/orcvio_githubpage/0072.pdf)**” *(IROS2020)*
  2. Shan, Mo, et al. “**[OrcVIO: Object Residual Constrained Visual-Inertial Odometry.](http://arxiv.org/abs/2007.15107)**” (IROS 2021)

  - [code](https://github.com/shanmo/OrcVIO), [video](http://moshan.cf/orcvio_githubpage/), [Project page](http://moshan.cf/orcvio_githubpage/).

- Rosen, David M., et al. “[**Towards Lifelong Feature-Based Mapping in Semi-Static Environments.**](https://doi.org/10.1109/ICRA.2016.7487237)” *(ICRA 2016)*

- 1. Henein Mina, et al. “[**Dynamic SLAM: The Need For Speed**](https://arxiv.org/abs/2002.08584v2).” *(ICRA 2020)*
  2. Zhang Jun, et al. “**[VDO-SLAM: A Visual Dynamic Object-Aware SLAM System](http://arxiv.org/abs/2005.11052)**.” (ArXiv 2020)
  3. **[Robust Ego and Object 6-DoF Motion Estimation and Tracking](https://arxiv.org/abs/2007.13993v1)**,Jun Zhang, Mina Henein, Robert Mahony and Viorela Ila. IROS 2020([code](https://github.com/halajun/multimot_track))

  - [code](https://github.com/halajun/VDO_SLAM), [video](https://drive.google.com/file/d/1PbL4KiJ3sUhxyJSQPZmRP6mgi9dIC0iu/view), Vision
  
- Minoda, Koji, et al. “**[VIODE: A Simulated Dataset to Address the Challenges of Visual-Inertial Odometry in Dynamic Environments.](https://arxiv.org/abs/2102.05965v1)**” (RAL 2021)

  - [**code_and dataset**](https://github.com/kminoda/VIODE), [video](https://youtu.be/LlFTyQf_dlo).

- Vincent, Jonathan, et al. “[**Dynamic Object Tracking and Masking for Visual SLAM.**](https://arxiv.org/abs/2008.00072v1)”, (IROS 2020)

  - [code](https://github.com/introlab/dotmask), video, 

- Huang, Jiahui, et al. “**[ClusterVO: Clustering Moving Instances and Estimating Visual Odometry for Self and Surroundings](http://arxiv.org/abs/2003.12980)**.” (CVPR 2020)

  - Tsinghua, code, [video](https://youtu.be/paK-WCQpX-Y).[slides](https://cg.cs.tsinghua.edu.cn/people/~huangjh/clustervo-slides.pdf).

- Liu, Yuzhen, et al. “**[A Switching-Coupled Backend for Simultaneous Localization and Dynamic Object Tracking.](https://github.com/zhuhu00/Awesome_Dynamic_SLAM/raw/main/pdfs/liu_2021_SwitchingCoupled.pdf)**” (RAL 2021)

  - Tsinghua

- Yang Charig, et al. “**[Self-Supervised Video Object Segmentation by Motion Grouping](http://arxiv.org/abs/2104.07658)**.”(ICCV 2021)

  - [Project page](https://charigyang.github.io/motiongroup/).

- Long Ran, et al. “**[RigidFusion: Robot Localisation and Mapping in Environments with Large Dynamic Rigid Objects](http://arxiv.org/abs/2010.10841)**.” ,(RAL 2021)

  - [**project page**](https://conferences.inf.ed.ac.uk/rigidfusion/).code, video, 

- Yang Bohong, et al. “**[Multi-Classes and Motion Properties for Concurrent Visual SLAM in Dynamic Environments.](https://doi.org/10.1109/TMM.2021.3110667)**” *IEEE Transactions on Multimedia*, 2021

- Yang Gengshan and Ramanan Deva. “**[Learning to Segment Rigid Motions from Two Frames.](http://arxiv.org/abs/2101.03694)**” CVPR 2021

  - [Project page](https://www.contrib.andrew.cmu.edu/~gengshay/cvpr21rigidmask.html).

- Thomas Hugues, et al. “**[Learning Spatiotemporal Occupancy Grid Maps for Lifelong Navigation in Dynamic Scenes.](http://arxiv.org/abs/2108.10585)**” 

  - [code](https://github.com/utiasASRL/Deep-Collison-Checker).

- Jung Dongki, et al. “**[DnD: Dense Depth Estimation in Crowded Dynamic Indoor Scenes.](http://arxiv.org/abs/2108.05615)**” (ICCV 2021)

  - code, video.
  
- Luiten Jonathon, et al. “[**Track to Reconstruct and Reconstruct to Track.**](https://arxiv.org/abs/1910.00130v3)”, (RAL+ICRA 2020)

  - [code](https://github.com/tobiasfshr/MOTSFusion), [video](https://youtu.be/PMOYkpBwE78). **Reconstruct**.

- Grinvald, Margarita, et al. “**[TSDF++: A Multi-Object Formulation for Dynamic Object Tracking and Reconstruction.](http://arxiv.org/abs/2105.07468)**”(ICRA 2021)

  - [code](https://github.com/ethz-asl/tsdf-plusplus), [video](https://youtu.be/dSJmoeVasI0).

- **Wang Chieh-Chih, et al. “[Simultaneous Localization, Mapping and Moving Object Tracking.](https://www.ri.cmu.edu/pub_files/pub4/wang_chieh_chih_2007_1/wang_chieh_chih_2007_1.pdf)” *The International Journal of Robotics Research 2007***

- Ran Teng, et al. “**[RS-SLAM: A Robust Semantic SLAM in Dynamic Environments Based on RGB-D Sensor](https://doi.org/10.1109/JSEN.2021.3099511)**.” 

- Xu Hua, et al. “OD-SLAM: Real-Time Localization and Mapping in Dynamic Environment through Multi-Sensor Fusion.” * (ICARM 2020)* https://doi.org/10.1109/ICARM49381.2020.9195374.

- Wimbauer Felix, et al. “**[MonoRec: Semi-Supervised Dense Reconstruction in Dynamic Environments from a Single Moving Camera.](http://arxiv.org/abs/2011.11814)**” (CVPR 2021)

  - **[Project page](https://vision.in.tum.de/research/monorec)**. [code](https://github.com/Brummi/MonoRec). [video](https://youtu.be/-gDSBIm0vgk). [video 2](https://youtu.be/-gDSBIm0vgk).

- Liu Yu, et al. “Dynamic RGB-D SLAM Based on Static Probability and Observation Number.” *IEEE Transactions on Instrumentation and Measurement*, vol. 70, 2021, pp. 1–11. *IEEE Xplore*, https://doi.org/10.1109/TIM.2021.3089228.

- P. Li, T. Qin, and S. Shen, “**[Stereo Vision-based Semantic 3D Object and Ego-motion Tracking for Autonomous Driving](http://arxiv.org/abs/1807.02062)**,” arXiv 2018

  - 沈邵颉老师组

- G. B. Nair *et al.*, “**[Multi-object Monocular SLAM for Dynamic Environments](http://arxiv.org/abs/2002.03528)**,” IV2020

- M. Rünz and L. Agapito, “[**Co-fusion: Real-time segmentation, tracking and fusion of multiple objects,**](https://doi.org/10.1109/ICRA.2017.7989518)” in *2017 IEEE International Conference on Robotics and Automation (ICRA)*, May 2017, pp. 4471–4478.

  - [code](https://github.com/martinruenz/co-fusion), 

- [TwistSLAM: Constrained SLAM in Dynamic Environment](https://arxiv.org/pdf/2202.12384),
  - S3LAM的后续，使用全景分割作为检测的前端



## Researchers

### 🥼1. Berta Bescos

> **主页：[谷歌学术](https://scholar.google.hk/citations?user=8koVpHwAAAAJ&hl=it) | [个人主页](https://bertabescos.github.io/) | [GitHub](https://github.com/BertaBescos)**

> **博士学位论文： [Visual slam in dynamic environments](https://zaguan.unizar.es/record/100672)**

> **代表性工作**

- **[1]** Bescos B, Fácil J M, Civera J, et al. [DynaSLAM: Tracking, mapping, and inpainting in dynamic scenes](https://arxiv.org/pdf/1806.05620.pdf)[J]. IEEE Robotics and Automation Letters, **2018**, 3(4): 4076-4083.
- **[2]** Bescos B, Neira J, Siegwart R, et al. [Empty cities: Image inpainting for a dynamic-object-invariant space](https://arxiv.org/pdf/1809.10239.pdf)[C]//2019 International Conference on Robotics and Automation (ICRA). IEEE, **2019**: 5460-5466.
- **[3]** Bescos B, Cadena C, Neira J. [Empty cities: A dynamic-object-invariant space for visual SLAM](https://arxiv.org/pdf/2010.07646.pdf)[J]. IEEE Transactions on Robotics, **2020**, 37(2): 433-451.
- **[4]** Bescos B, Campos C, Tardós J D, et al. [DynaSLAM II: Tightly-coupled multi-object tracking and SLAM](https://arxiv.org/pdf/2010.07820.pdf)[J]. IEEE Robotics and Automation Letters, **2021**, 6(3): 5191-5198.

### 🥼2. Yubao Liu

[Walk Into AI World](https://www.ybliu.com/)

> 主页：[谷歌学术](https://scholar.google.com/citations?user=P4M6ru0AAAAJ&hl=zh-CN) | [GitHub](https://github.com/yubaoliu)

> 代表性工作：

1. RDS-SLAM: real-time dynamic SLAM using semantic segmentation methods
2. KMOP-vSLAM: Dynamic Visual SLAM for RGB-D Cameras using K-means and OpenPose
3. RDMO-SLAM: Real-time Visual SLAM for Dynamic Environments using Semantic Label Prediction with Optical Flow
#### 3. Guoquan Huang(SLAMMOT)


#### 4. Shenshao Jie(MOT)
