---
layout: post
title: 嵌入式AI TinyML学习笔记(一)
date: 2023-11-07
author: lau
tags: [AI, Archive]
comments: true
toc: false
pinned: false
---

嵌入式AI TinyML学习汇总笔记。

<!-- more -->

## 1 前言

云端智能物联网（Artificial Intelligence + Internet of Thing， AI + IoT = AIoT）已是非常成熟的行业了，通常会以非常平价的微处理器（Microprocessor， uP）或单芯片（Micro Controller Unit， MCU）当作本地端（Local / Edge）的主要核心，接收各式「感应元件」（如温湿度、光照、心率、影像、声音等）及控制各式「致动元件」（LED， 开关、马达等），接着再利用长短距离「通讯模块」（如WiFi， BT， 4G/5G等）来传送数据和接收命令，最后再和云端的运算、控制、储存并和用户的移动通信装置连结就能完成整个AIoT系统。 其中所时序预测、维修提醒、异常侦测、影像对象侦测及辨识等智能分析工作通常都交由云端服务器来协助完成。

![](https://1.bp.blogspot.com/-R8jzLJeNiN0/YULD2eNqRXI/AAAAAAAAEtk/8M8WTfgaL5o_2e9PjcSXGmsNkgpNYX7EQCPcBGAYYCw/s1654/iThome_Day_01_Fig_01.jpg)

Fig. 1-1 智能物联网（AIoT）架构 （OmniXRI整理绘制， 2021/9/9）

不过在强调「低延迟反应」、「高数据隐私」、「低功耗」、「低成本」的需求下**边缘智能（Edge AI）**的需求就日益增加，所以大家就想把AI的功能加到MCU上，希望藉此能得到又便宜又好用的智能设备。 但不幸地是，脚踏车是难以和机车甚至汽车、飞机匹敌的。 同样地要用几十到几百元台币的产品去对抗几千到几十万甚至上千万台币的产品更是天方夜谭。 所以要认清最适合脚踏车的应用范围，那就能获得最大的收益。

为了让大家更了解MCU如何实现AI，在接下来的30天会帮大家介绍一项名为「微机器学习tinyML」的技术，同时介绍两大核心技术MCU及AI基本原理，并说明如何实现在Arm Cortex-M系列MCU上及如何应用通用开发平台来完成自定义AI应用。 就好比一口吃下爆酱濑尿虾牛丸（tinyML）就能同时享受牛肉丸（MCU）的鲜美、濑尿虾（AI）的甘甜，更可让大家在享受美食时一并了解它如何制作及开发出更适合自己口味的新丸子。

这班tinyML的列车就要出发了，还不赶紧跟上。

「tinyML」就字面上意思就是微小的机器学习（Tiny Machine Learning， tinyML），但它又和人工智能（Artificial Intelligence， AI）及微控制器单元（Micro-Controller Unit， MCU）（又称单芯片）有什么关连呢？

首先说明几个常见的名词定义及主要差异。

- 【人工智能（Artificial Intelligence， AI）】：又称人工智能，泛指能产生类似人类思考、判断结果的人为程序、算法。 其中最常见的应用包含计算机视觉（Computer Vision， CV）、自然语言处理及理解（Natural Language Processing / Understanding， NLP / NLU）及数据分析/挖矿（Data Analysis / Mining）几大领域。

- 【机器学习（Machine Learning， ML）】：为人工智能下的一部份，泛指透过数学统计进而模仿人类学习数据中的规则，进而推论未知数据的结果。 常见的分类有监督式（Supervised）、非监督式（Unsupervised）及增强式（Reinforcement）学习，其主要功能为分类、回归及聚类。

- 【深度学习（Deep Learning， DL）】：为机器学习下的一个分支，类神经网络（Neural Network， NN），它可透过更深及更宽的网络层及巨大量参数（权重）来学习各种影像、声音及数据特征，进而达到和人类似的学习、辨识及推论能力，为近年来当红技术。 由于其能力显著，因此当许多非专业人士提及「人工智能」时，通常指的就是「深度学习」技术，而非原始名词定义。

![](https://1.bp.blogspot.com/-XuWHULtnDgk/YULD2a9lvCI/AAAAAAAAEto/d45PphaZEYw5sI7W6M9vZ2d7lCw0cL9owCPcBGAYYCw/s1658/iThome_Day_02_Fig_01.jpg)

Fig. 1-1 人工智能、机器学习、深度学习 （OmniXRI整理绘制，2018/5/24）

通常来说「机器学习」能处理的特征数量及数学模型通常不会太复杂，因此所需的计算量也不会太大。 但深度学习就反其道而行，越复杂的模型及越巨量的参数就能令推论准确率得到更好的提升，因此需要极高性能的运算设备来辅助训练和推论的计算。 所以如果要将AI应用放到运算能力很低、内存很少的MCU上时，就只选择较微型的AI应用（如智能感测器等）或者较小型的机器学习算法甚至超微型的深度学习模型来推论，因此「tinyML」需求因运而生。

基于上述AI微型化理念，许多CPU，GPU， MCU， AI 加速计算芯片大厂、AI开发工具及应用厂商纷纷响应，于2019年成立微型机器学习基金会（tinyML Foundation） ，每年定期会举办高峰会，让厂商、学界、社群都能共襄盛举。 2021高峰会有近六十个赞助商，其中台湾也有四家，分别是奇景光电（Himax）、装智（On-Device AI）、原相（PixArt）及威盛（VIA）。

![](https://1.bp.blogspot.com/-Wpwl2a-dODE/YULVCd9mTVI/AAAAAAAAEts/TgqLKeF_bIYcLOQ75kxx3D0SxvTjH9eqgCLcBGAsYHQ/s1783/iThome_Day_02_Fig_02.jpg)

Fig. 1-2 tinyML基金会2021赞助商列表 （OmniXRI整理绘制，2021/8/14）

根据该基金会对tinyML的定义：「**微型机器学习（tinyML）**为一个快速发展的机器学习技术和应用领域，包括硬件、算法和应用软件。 其能够以极低功耗执行设备上的传感器（Sensor）的数据分析，通常在mW（毫瓦特）以下范围，进而实现各种永远上线（或称常时启动）（Always On）的应用例及使用电池供电的设备。」 这里的ML虽然指的是「机器学习」，不过亦可延伸解释到「深度学习」甚至「人工智能」、「边缘智能」等名词。 从上述定义可得知tinyML 几乎是锁定MCU及低阶CPU所推动的Edge AI。

目前tinyML基金会并没有明确的定义那些项目才算是其范围，也没有制定特定的开发框架及函式库（如机器人操作系统ROS），而是开放给硬件及开发平台供应商自行解释及彼此合作。 目前较常见的应用，包括振动侦测、手势（运动感测器）侦测、感测器融合、关键字侦测（声音段分类）、（时序讯号）异常侦测、影像分类、（影像）对象侦测等应用，而所需算力也依序递增。 一般来说，以Arm Cortex-M系列为例，智能传感器（如声音、振动、温湿度等）大约Arm Cortex-M0+， M3左右就能满足，而智能影像传感器（小尺寸影像）要完成影像分类及对象侦测工作，则需要Cortex-M4， M7甚至要到Cortex-A， R系列。

![](https://1.bp.blogspot.com/-7-kEJyqoElQ/YULX3GGseHI/AAAAAAAAEt0/uZbff2UnOns1pE-11D_DgRjvzmxIcsX_gCLcBGAsYHQ/s1654/iThome_Day_02_Fig_03.jpg)

Fig. 1-3 Arm MCU等级芯片智能运算能力与适用情境 （OmniXRI整理绘制，2021/8/14）

引用链接 ：

tinyML基金会 https://www.tinyml.org/

## 2 TinyML开发板

由于MCU规格大小性能差异颇大，所以可以运行何种模型及速度是否满足，须以实际布署为准。

- Arduino Uno R3
- Arducam Pico4ML(Raspberry Pi RP2040)
- Seeeduino XAIO (SAMD21G18)
- Eta ECM 3532
- Silicon Labs Thunderboarrd Sense 2
- Nordic nRF52840
- Arduino Nano 33 BLE Sense (nRF52840)
- Arduino Protenta H7 (STM32H747XI)
- OpenMV Cam H7 (STM32H743VI)
- Himax WE-I Plus （台湾厂商奇景光电）

![](https://1.bp.blogspot.com/-lo9RGRcEGW8/YULrONzVvOI/AAAAAAAAEuA/WMx2JuDkfBQ6Fj3TSggyqeLBmUDiFjR0ACPcBGAYYCw/s1658/iThome_Day_03_Fig_01.jpg)

Fig. 2-1 常见tinyML开发板。 （OmniXRI整理绘制， 2021/8/14）

在Arm Cortex-M系列中，皆为32bit MCU，依指令集效能（非工作时脉速度）来排名，大概为M0， M0+， M1， M3， M4， M7， M23， M33， M35P， M55，而其内建的程式码区（Flash）和静态随机内存（SRAM）通常不多，仅有数百KB到数MB而已，并会随着不同厂商及产品线会有不同配置。 不过相较于一般仅有数十KB Code Flash及数KB的SRAM，这样的配置已相当不错，可做出相当多的应用。

从上面表格中可看出，Cortex-M的MCU的工作时脉通常不高，内存也不多，这使得运行tinyML前就要考虑是否能将模型及参数塞进代码区，运行时所需的变量内存是否够用，同时要评估工作时脉（含平行指令数）推论速度是否能满足实际应用。 当然这些评估工作有些亦由开发平台商提供的工具代劳，不须使用者头痛，待后面章节再行介绍。

目前在这么多开发板中，其中又以Arduino Nano 33 BLE Sense被最多tinyML平台商支持，其主要原因如下所示：
- 主芯片为Nordic nRF52840，其中以Arm Cortex-M4为主要核心，可支持浮点数运算，工作时脉64MHz， 1MB Flash， 256KB SRAM。
- Arm Cortex-M4可支持Arm Mbed操作系统及Cortex单芯片软件界面标准CMSIS（Common Microcontroller Software Interface Standard），其中亦包括CMSIS-NN神经网络加速运算库。
- 板子上有很多传感器，包括运动感测模组、麦克风（声音）、手势、色彩、近接（光电）、气压、温湿度等。
- 具有2.4GHz 蓝牙低功耗模块（BT 5.0， BLE）可轻松连接到笔电或其它移动设备，方便传送数据及接收命令。
- 板子体积很小，仅有45mm x 18mm，非常适合直接做成产品原型机。

另外Arduino Nano 33还有两片兄弟板，分别为Nano 33 IoT， Nano 33 BLE，原则上和Nano 33 BLE Sense只差在传感器的支持数量不同，其它使用上都相同，更完整规格及使用说明可参见文末连结。

最后补充几个重要的tinyML开发平台商所支持的开发板清单。

Edge Impulse https://docs.edgeimpulse.com/docs/fully-supported-development-boards

- ST B-L475E-IOT01A (IoT Discovery Kit)
- Arduino Nano 33 BLE Sense
- Eta Compute ECM3532 AI Sensor
- Eta Compute ECM3532 AI Vision
- OpenMV Cam H7 Plus
- Himax WE-I Plus
- Nordic Semiconductor nRF52840 DK
- Nordic Semiconductor nRF5340 DK
- Nordic Semiconductor nRF9160 DK
- Silicon Labs Thunderboard Sense 2
- Sony's Spresense
- TI CC1352P LaunchPad
- Arduino Portenta H7 + Vision shield (preview support)
- Raspberry Pi 4
- NVIDIA Jetson Nano
- Seeed Wio Terminal (ATSAMD51)
- Agora Product Development Kit
- Arducam Pico4ML TinyML Dev Kit (PR2040)
- Blues Wireless Swan (STM32L4+)
AITS (cAInvas) https://www.ai-tech.systems/cainvas/

- Raspberry Pi 3
- Arduino Nano 33 BLE Sense
- STM32F4
- STM32L4
- STM32F3
- Microchip AT91SAM9260
- Infineon PSoC 6
- NXP i.MX RT1060
- NXP LPC5500
SensiML https://sensiml.com/documentation/firmware/

- Arduino Nano 33 BLE Sense
- Arm GCC Cortex M4/M7/A53
- Microchip SAMD21 ML Eval Kit (SAM-IoT WG）
- Nordic Thingy
- QuickLogic Chilkat
- QuickLogic QuickAI
- QuickLogic QuickFeather
- Raspberry Pi
- Sillicon Labs Thunderboard Sense 2
- SparkFun QuickLogic Thing Plus - EOS S3
- ST SensorTile
- ST SensorTile.Box
- x86 Processors
Google TensorFlow Lite Microcontroller https://www.tensorflow.org/lite/microcontrollers

- Arduino Nano 33 BLE Sense
- SparkFun Edge
- STM32F746 Discovery
- Adafruit EdgeBadge
- Adafruit TensorFlow Lite for Microcontrollers
- Adafruit Circuit Playground Bluefruit
- Espressif ESP32-DevKitC
- Espressif ESP-EYE
- Wio Terminal：ATSAMD51
- Himax WE-I Plus EVB
- Synopsys DesignWare ARC EM Software Development Platform
引用链接 ：

- Arm Cortex-M 维基百科 https://zh.wikipedia.org/wiki/ARM_Cortex-M
- Arduino Nano 33 IoT https://store-usa.arduino.cc/collections/boards/products/arduino-nano-33-iot
- Arduino Nano 33 BLE https://store-usa.arduino.cc/collections/boards/products/arduino-nano-33-ble
- Arduino Nano 33 BLE Sense https://store-usa.arduino.cc/collections/boards/products/arduino-nano-33-ble-sense

## 3 tinyML整合开发平台介绍

在传统中大型AI应用程序中，几乎都在同一个平台（如云端服务器加AI加速运算装置）或同一部电脑（如PC加Nvidia GPU或树莓派加INTEL神经运算棒）上就能完成数据标注、模型建置、训练调参、模型优化和布署测试，只需会Python语言就几乎能全部搞定，包括人机界面。 但遇到像tinyML这种只能在MCU上运作，资源超级少的平台上，很多工作就要分散到其它平台上去开发，只留下推论部份。 若再加上先前提过，每个MCU供应商可能有自己的开发平台，那就要学习更多技术（含硬件控制）才能上手，这就导致许多只会Python的AI工程师大呼吃不消，甚至碰不了，因为多数的MCU都是以C语言甚至是ASM（组合）语言开发的。

如图Fig. 3-1所示，为了解决这个麻烦，许多厂商纷纷推出（No-Code， 不用写程序）或少码（Low-Code， 只需少量程序）的tinyML开发解决方案，包括整合型开发平台和依MCU框架开发两大类方案。 前者把所有AI开发步骤全部整合在云端，用户只需按着步骤就能完成一些常见的AI应用，如声音辨识，异常振动、影像分类、影像对象侦测等，优点是不需写任何代码就能布署到MCU上，且提供很多的可视化工具帮忙调整模型，但缺点就是弹性较小。 而后者虽然各家MCU都有自己的开发工具，但如果是使用Arm Cortex-M系列，则Arm有提供Mbed OS操作系统和CMSIS标准函式库，其中更有专门用于神经网络加速运算的CMSIS-NN函式库，方便同样是Arm Cortex-M系列的MCU跨厂牌使用。 另外像Google 也有提供TensorFlow Lite Micro （TFLM or TFLu， Microcontroller专用），方便使用自家TensorFlow开发出的模型，使其可以轻易的转成MCU可以运作的代码。 当然目前支持tinyML的开发板多半是Arm Cortex-M系列MCU，所以转换出来的MCU程序也多半是由CMSIS-NN函式来完成。 使用MCU框架开发有很大弹性，还可充份融合感测器及外围模组，不过缺点也很明显，就是没有一定MCU程序开发功力的人，一开始会搞得一头雾水，不知从何下手。 在接下来的章节中会尽量帮大家介绍不要无脑使用，也不要烧脑使用tinyML on MCU。

![](https://1.bp.blogspot.com/-OTkL4kXg1T4/YU1PjbPL9WI/AAAAAAAAEwY/sNxKlwVcB9YWYKOzALa2mbfLTjpDclRpACLcBGAsYHQ/s1658/iThome_Day_10_Fig_01.jpg)

Fig. 3-1 传统AI及tinyML（MCU AI）应用程式开发流程图。 （OmniXRI整理绘制， 2021/8/14）

### 3.1 tinyML整合型开发平台

tinyML整合型开发平台提供商为了让用户不用懂太多AI技术，也不用会写MCU程序，所以通常提供一站式云服务，申请一个帐号加上网页浏览器就能开始使用，主要整合了下列项目：
- 资料收集：可实时收录声音、运动传感器信号、网络摄影机取像，或自行准备预录或影像数据集上传。
- 资料标注：可分割连续数据并标注，或者直接标注上传的数据集。
- 模型选用：通常只有数种常用模型可选，可作部份网络结构微调。 通常不支持从常见框架（如TensorFlow， PyTorch， ONNX... ）建立的模型转换或者仅支持特定框架。
- 训练调参：提供数据简单滤波（去除噪声）、数据分布图表、训练结果正确率比较表等各种可视化图表，方便调整特定超参数。
- 模型优化：一般中大型AI应用如果没有布署空间限制问题，通常不一定需要这个步骤，但对于MCU这种资源很有限的执行推论装置一定要加入。 通常这个部份可使用量化、剪枝、蒸馏、压缩等技术来减化模型结构及参数（内存使用）量，同时要确保优化后推论准确率只能掉个几%。
- 布署测试：不管训练结果如何，最后还是要布署到MCU上。 有些是将优化后的模型结构和参数转成C语言程序（*.c， *.h），再由原MCU开发平台当成一部份程序一起编译，这样方便整合原先的MCU程序。 另一种则是安装一些套件直接帮你刻录到特定开发板的MCU中，再使用虚拟序列端口（COM）将推论结果从MCU输出回电脑上，单纯把tinyML的开发板当成一种智能传感器，不用写半行MCU程序。 最后如果测试结果不理想或准确率过低，则需回到训练调参，甚至是数据收集、标注步骤重新来过，直至满意为止。

目前常见的tinyML整合型开发平台供应商，如Fig. 3-2所示。 当然不只这些，只是这几个平台提供较多的学习资源及参考资料，更重要地是「免费」（不知道何时会开始收费）。 

![](https://1.bp.blogspot.com/-2lbjwVn0arc/YU11XeDfw4I/AAAAAAAAEwo/Nq-n7-maSBQ9vp2EH7h7Fz5Of2cikF5-ACLcBGAsYHQ/s1658/iThome_Day_10_Fig_03.jpg)

- Edge Impules(https://docs.edgeimpulse.com/docs)
- Fraunhofer IMS AIfES（另可支持Arduino UNO R3）(https://www.ims.fraunhofer.de/en/Business-Unit/Industry/Industrial-AI/Artificial-Intelligence-for-Embedded-Systems-AIfES.html)
- AITS cAInvas（另有提供类似APP方式的tinyML应用程序）(https://www.ai-tech.systems/cainvas/)
- SensiML(https://sensiml.com/resources/information/)

### 3.2 依MCU框架开发tinyML

如果你是一般的MCU工程师，那大概对Fig. 3-3的程序开发堆栈（Stack）不会太陌生，只是最上面应用层和C Code中间多了一些东西，接下来就帮大家补充说明一下。

一般从事AI应用开发的人多半会选择较多人使用的框架（建模、训练、布署），如TensorFlow， PyTorch， ONNX等。 不过这样的框架都太大，通常连小型单板微电脑（如树莓派、手机、平板等）都塞不进去。 所以需要一些优化工具来把模型简化、参数减小，如Google TensorFlow Lite， Nvidia TensorRT， Intel OpenVINO等。 不过这些工具转换后的内容对于tinyML所使用的MCU来说还是太大，所以才有像Google TensorFlow Lite for Microcontroller这类的工具出现，企图用更精简的方式来产生MCU所需执行的代码。

我们都知道在电脑上可用多线程或单指令多数据流（SIMD）指令集的CPU或者是具有超多乘加器的GPU来加速运算（训练及推论），但在一般MCU上就很难有这些，只能靠CPU慢慢算。 曾介绍过Arm Cortex-M4以上MCU就有支持单指令周期乘加器、硬件除法和SIMD指令，这可大幅提升运算速度。 但这些超强功能，没有写组合语言根本很难作到，所以Arm CMSIS就提供了通用界面程式方便大家使用，尤其更提供的CMSIS-NN来完善神经网络加速运算，把常用的基本元素都已封装好。 因此像TFLM就能轻松使用这样的接口来转换成MCU通用的C语言程序。

在MCU开发中，通常写好C语言后配合GCC或者专属编译器（Compiler）就能产生能在MCU上运行的执行文件（*.bin或 *.hex）。 有时为了更方便管理工作排程、资源冲突、档案管理等高阶动作，有时也会MCU上加入超迷你的操作系统，如Arm Mbed OS， freeRTOS等。 当然不使用通用型要自己开发作业也是OK的，那又是另一个神级工作，这里就不加讨论了。
![](https://1.bp.blogspot.com/-3kKHyjm_9Tw/YU1PjQqi9TI/AAAAAAAAEwc/pZU473RAiKQr_DiboxOsG6jAz_FiKSqkgCLcBGAsYHQ/s1658/iThome_Day_10_Fig_02.jpg)

参考链接

- Arm Mbed OS(https://os.mbed.com/)
- Arm Mbed OS Github(https://github.com/ARMmbed/mbed-os)
- Arm Common Microcontroller Software Interface Standard (CMSIS) Documentation(https://arm-software.github.io/CMSIS_5/General/html/index.html)
- Arm CMSIS Version 5 Github(https://github.com/ARM-software/CMSIS_5)

## 4 tinyML开发框架（一）：TensorFlow Lite Micro初体验（上）

说到tinyML不得不说起「TinyML Machine Learning with TensorFlow Lite on Arduino and Ultra-Low-Power Microcontrollers」一书的作者Pete Warden，他应该是最早将这个概念整理成书的作者（2020/1/7 初版）， 并且在Youtube上也有多段影音教学影片（如文末参考链接【TinyML Playlist】）。 目前这本书也有中文译本在台发售，碁峯于2020/7/31出版，书名「tinyML TensorFlow Lite机器学习 应用Arduino与低耗电微控制器」，如Fig. 4-1所示。

![](https://1.bp.blogspot.com/-h4rxcATnPWY/YVCxbltbgTI/AAAAAAAAExI/cx8jwEBdmswyQQ8ZyN_jVMR8ESAJGjWogCLcBGAsYHQ/s1658/iThome_Day_12_Fig_01.jpg)

>「TinyML」中文版书中对两位作者的介绍：Pete Warden 是移动及嵌入式TensorFlow的技术主管，也是TensorFlow团队的创始成员之一。 他曾经是Jetpac的CTO和创始人，该公司在2014年被Google收购。Daniel Situnayake 是Google的首席开发布道师，并且协助运作tinyML聚会小组。 他也是Tiny Farms的共同创办人，Tiny Farms是美国第一家大规模自动生产昆虫蛋白的公司。

相信可能很多人第一次接触tinyML应该都是跟着这本书来实作和练习，但说实在的，步骤有点多，有很多开发环境要设置，所以对于新手会有些吃力，因为除了AI部份外，还有MCU部份要处理。 好在AI的世界进步速度颇快，才过了一年多，就有了不少改进，简化了不少。 接下来就让我们跟着Google官网「TensorFlow Lite for Microcontrollers(https://www.tensorflow.org/lite/microcontrollers?hl=zh-tw)」（以下简称TFLM）的说明来实际操练一下。

什么是TFLM呢？ 一般大家在开发深度学习模型时多半会使用Google TensorFlow框架，但到了手机或单板微电脑（如Arm Cortex-A， Cortex-R或树莓派Cortex-A53/A57等）时代，这样的框架太大塞不进这些系统中，于是Google又推出TensorFlow Lite，它不仅更短小精悍，同时还提供的模型优化工作， 使得在推论准确率只损失数个百分点甚至几乎没有损失的情况下，让模型缩小十倍以上，不仅更容易塞进这些开发板，同时加快了推论速度。 不幸地是，当遇到像MCU（Cortex-M）等级的开发板时，又遇到塞不进的问题，所以Google才会再推出TensorFlow Lite for Microcontroller。 表面上名字很像Lite，但实质上为了牵就MCU的开发架构，因此本质上有很大不同，无法直接取代。 原则上这些都是开源码，有兴趣研究源代码的朋友可参考文末参考链接【TFLM Github开源码】。 另外目前TFLM可支持的开发板可参考[Day 03] tinyML开发板介绍的说明。

接下来就跟着Google TFLM的Hello World来建立基本操作观念。 这个范例主要分成两个部份，如下所示。

- 训练模型：使用Python，在Google Colab （jupyter notebook）完成训练、转换及优化模型。
- 执行推论：使用C++ 11，在开发板上使用C++库执行模型推论。

**训练模型（Python + Google Colab + GPU）**

首先点击Google提供的Colab范例程序(https://colab.research.google.com/github/tensorflow/tflite-micro/blob/main/tensorflow/lite/micro/examples/hello_world/train/train_hello_world_model.ipynb)，免下载，可直接运行，Jupyter Notebook操作环境，说明文字和程序一起存在，方便学习，只需在每个程序字段左上角按下黑色箭头（或者点想执行的字段按Ctrl + Enter亦可）即可单步执行，但切记要按照顺序把每个步骤都执行完，不能跳过任何一步骤。 由于开启后会看到先前运行结果都被保留在执行结果字段，为了更清楚看到所有动作，可执行主菜单的[编辑]─[清除所有输出字段]，将所有输出字段清除。

首先说明这个「Hello World」程序主要想展示如何将一个TensorFlow建立好的模型转换到TFLM，为了方便说明，并没有使用现成常用的数据集，而是以正弦波加乱数方式产生一个数据集，然后训练出一个模型（正弦波函数），使得输入X位置就能推论出Y位置。

接下来就快速摘要一下整个程序在做什么？ 程序部份请参考原始程序，这里仅作重点摘要及补充说明，方便大家更容易理解。

- 配置模型工作路径名称。
- 安装TensorFlow 2.4.0版环境，并导入Python所需套件。
- 产生数据
    - 产生正弦波基本数据，在0~2 PI间产生1000点当作x，再将其顺序打乱，并依正弦（Sine）函式计算出对应的y值，得到1000组原始资料，（x，y）。
    - 增加噪声，为了使数据更接近真实世界随意变动，所以将原始数据的y值随机加入10%的变动。
    - 分割数据集，为了后续训练、验证及测试模型的效能，所以将数据集分割成60%， 20%， 20%。
- 模型训练
    - 设计模型，一般来说像正弦波这类数据，采用「机器学习」的「多项式回归」方法大概就能解出不错的结果。 不过这里故意使用一个全连结的神经网络来代替，输入层就只有一个神经元，输入值就是x，输出层也只有一个神经元，而隐藏层使用8个神经元，每个神经元的输出激励函数采用「ReLu」，最终训练时判定损失的函数则采用MSE（Mean Square Error， 均方误差），度量方法采MAE（Mean Absolute Error， 平均绝对误差）， 而调整优化方法则采「Adam」。 整体模型采用TensorFlow的Keras方法来建构。 更多损失函数和优化方法可参考NTUST Edge AI Ch6-1 模型优化与布署-模型训练优化(https://omnixri.blogspot.com/p/ntust-edge-ai-ch6-1.html)。
    - 训练模型，设定好模型后就可以开始训练，由于这个范例很小，所以原始档案并没有启动GPU加速计算，如果有需要可选择主菜单的[编辑]─[笔记本设定]─[硬件加速器]将[None]改成[GPU（Nvidia）]或[TPU（Google）]即可。 这个范例指定训练迭代次数（Epochs）次为500次，意思是不管是否达到满意程度，到达设置就停止。 批量大小（batch_size）则是一次取多少笔数据来训练，以600笔数据批量大小为64为例则要取10次才能完成一次迭代，所以理论上批量大小大一点会好些，但会随数据性质及多样性可能会造成模型训练结有很大不同，所以可自行调整大小来加速训练速度及测试收歛速度。
    - 绘制图表，为了更清楚知道训练过程中损失收歛的速度，可将训练过程的输出值绘成图表来观察，以图标约在100次迭代后就几乎收歛，只有很小幅度的下降。 若想更清楚看到变化细节，可忽略前50笔（可自行调整）。 另外除了损失率外亦可将平均绝对误差（MAE）当成观察内容。 接着为了知道训练了500次迭代的模型效能好不好，此时就可把测试集拿来当输入求预测值，再将实际值和预测一起绘出，很明显差了很多，还不满足需求。 表示训练看起来好像不错但实际上根本不能用。
- 训练一个大模型
    - 设计模型，接着把隐藏层变成两层，且两层都使用16个神经元进行全连结，其它条件和先前一样。
    - 训练模型，同样地训练500次迭代。
    - 绘制图表，同样绘出LOSS和MAE，可看出大约在350次迭代后才趋于收歛。 最后实际值和预测值也趋于拟合，已可看出一条很接近正弦波的曲（红）线。 代表这个模型已可应付大部份的状况。
- 产生TensorFlow Lite模型
    - 产生有量化和没有量化的模型，原始TensorFlow模型是以FP32（32bit浮点数）格式进行训练，接着将其转成TensorFlow Lite格式，接着若能把训练好的参数降到INT8（8bit整数）则模型大小就能变为1/4。 但前提是预测的结果不能变化太大才有意义。
    - 比较模型效能，在比较之前需将模型以不同参数格式加载，为了正确配置相关记忆体，需初始化TFLite的直译器（Interpreter）再加载，最后将有无量化的比较结果绘制成图表。 由图可看出绿色X和蓝色X几乎重叠，代表量化后的结果几乎没有影响。 再经由数字来看，原始TensorFlow的模型损失率1.02%和转成TensorFlow Lite 未优化前几乎相同，而经过量化后变成1.08%，变化极小。 此时再来看模型（参数）大小，原始TensorFlow使用了4096 Byte，经转成TFLite后缩小至2788（减少1308）Byte，而经量化后，只剩2488（再减少300）Byte，和原本模型相比仅为60.7%。 虽然离目标的25%（1/4大小）还很远，这是因为这个模型不算太复杂且很小，所以暂时无法发挥太明显效益。
- 产生TFLM模型，接着要安装xxd工具程序，利用它将TFLite已量化的模型再转成TFLM格式，即C语言格式档案，预设会于/hello_world/train/models下产生一个 model.cc 的档案，打开看就是一个2488 Byte的int格式阵列。
- 布署到MCU上，最后把 model.cc 和其它MCU相关的程序（*.cc）一起编译就能布署到MCU上，不过这部份不在Colab上完成，MCU程序编译和TFLM执行推论部份就留待下回分解。

![](https://1.bp.blogspot.com/-DyJqL-GwQK0/YVFS-U3CPxI/AAAAAAAAExQ/BTp9VIC6_vIdo20ZJhMQKJtmxun6cZzkwCLcBGAsYHQ/s1658/iThome_Day_12_Fig_02.jpg)

参考链接

- Google TensorFlow Lite for Microcontrollers 中文学习指南(https://www.tensorflow.org/lite/microcontrollers?hl=zh-tw)
- Pete Warden Youtube， TinyML Book Screencasts Playlist 影片列表(https://youtu.be/Fdt9xunlyCQ?list=PLtT1eAdRePYoovXJcDkV9RdabZ33H6Di0)
- TensorFlow Lite for Microcontrollers Github 开源码(https://github.com/tensorflow/tflite-micro)
- TFLM Get started with microcontrollers， The Hello World example(https://www.tensorflow.org/lite/microcontrollers/get_started_low_level?hl=zh-tw)
- Experiments with Google = TensorFlow Lite for Microcontrollers 案例分享(https://experiments.withgoogle.com/collection/tfliteformicrocontrollers)

## 5 tinyML开发框架（一）：TensorFlow Lite Micro初体验（下）

### 5.1 执行推论（C/C++ + Arduino IDE + MCU）

上章在训练完成，在最后一个输出字段出现一大堆数字，这就是大的那个模型经优化及INT8量化后的结果。 在训练过程中，其实有把TensorFlow， TensorFlow Lite和TensorFlow Lite for Microcontroller的模型储在云端硬盘暂存区中，只需点击左侧最下方那个档案夹符号就能看到相关输出文件，如图Fig. 5-1所示。 其中 model.cc （C/C++语言格式）就是要给MCU使用的。 要特别注意地是，由于这些档案是存放在Google Colab临时性的云端档案夹中，所以要先以鼠标右键点击档案，下载到自己的电脑或云端硬盘中，以免断线后就消失了。

![](https://1.bp.blogspot.com/-vbUx0oWi3DM/YVHNw4aHcBI/AAAAAAAAExc/BtwjF8jEiTo8-1S8NKcIVOxIq0V82LgzgCLcBGAsYHQ/s1658/iThome_Day_13_Fig_01.jpg)

为了省去一大堆编译及设定的指令，在MCU端最简单的方法就是使用Arduino IDE。为了让Arduino IDE能顺利使用TensorFlow Lite，首先要点击主菜单的[草稿码]─[导入库]─[管理库...]，然后输入「TensorFlowLite」，此时选择「Arduino_TensorFlowLite」，按下「安装」即可完成。 回到主屏幕后，可到主菜单[档案]─[范例]下检查一下是否已出现[Arduino_TensorFlowLite]选项，如图Fig. 5-2所示。 当点下[hello_world]就会自动产生完整示例及所需配套档案，如下所示。

- hello_world.ino - 范例主程序
- arduino_constants.cpp
- arduino_main.cpp
- arduino_output_handler.cpp
- constants.h
- main_functions.h
- model.cpp - 模型参数文件
- model.h - 模型参数头文件文件
- output_handler.h
ps. 这里暂时只会用到hello_world.ino和model.cpp，其它的档案就暂时忽略说明。

![](https://1.bp.blogspot.com/-pr7b_PkELXs/YVHNwm2QGRI/AAAAAAAAExY/5zYTyKNIXcAe1okmU-5V_Mnfe2Nk6JkMwCLcBGAsYHQ/s1658/iThome_Day_13_Fig_02.jpg)

接着把Arduino Nano 33 BLE Sense开发板连接到电脑上，并检查一下开发板是否都已正确设定及序列埠通讯正常，接着就可按左上角[上传]（右箭头符号），它会自动重新编译程序并上传到开发板上。 如果一切顺利，会在下方讯息区显示写入大小及花费时间。 板子会自动重置启动，如果看到板上橘色LED一直闪烁，就表示程序已顺利运作。 此时就可以到[工具]开启[序列端口监控窗口]观察板子送回电脑上的数值，或者使用[序列绘图家]可直接看到波形图，更方便侦错，如图Fig. 5-3所示。

![](https://1.bp.blogspot.com/-ENGVVSFMMFI/YVHNwzgFwqI/AAAAAAAAExg/O3MZ5S4GcXQyKqRNwrfR46-G-NVcL-S8gCLcBGAsYHQ/s1658/iThome_Day_13_Fig_03.jpg)

本来作到这里就该收工，但前先训练了老半天的结果 model.cc 怎么没用到，MCU就能自己跑起来了呢？ 这是因为范例已自带一个模型参数文件 model.cpp 。 不过仔细比对会发现 model.cpp 和 model.cc 除了文件名不同外，数值设定方式也稍有不同，所以可以只复制 model.cc 大括号{}中间的值再贴到 model.cpp 后再重新编译即可，如下所示。

```c++
// Google Colab訓練好的原始模型 model.cc
unsigned char g_model[] = {
  0x1c, 0x00, 0x00, 0x00, 0x54, 0x46, 0x4c, 0x33, 0x14, 0x00, 0x20, 0x00,
  ... （略） // 數值內容會隨訓練結果而有不同
  0x00, 0x00, 0x00, 0x09
};
unsigned int g_model_len = 2488;
```

```c++
// Arduino_TensorFlowLite hello_world範例中的預設模型 model.cpp
#include "model.h"

alignas(8) const unsigned char g_model[] = {
    0x1c, 0x00, 0x00, 0x00, 0x54, 0x46, 0x4c, 0x33, 0x14, 0x00, 0x20, 0x00,
    ... （略） // 數值內容會隨訓練結果而有不同
    0x00, 0x00, 0x00, 0x09};
const int g_model_len = 2488;
```

接下来就把hello_world.ino作一些简单中文注解，希望能帮助大家更容易理解程序运作原理。 其主要步骤可参考官网说明「Get started with microcontrollers - Run inference(https://www.tensorflow.org/lite/microcontrollers/get_started_low_level?hl=zh-tw)」。

```c++
/*  
Copyright 2020 The TensorFlow Authors. All Rights Reserved.
hello_world.ino MCU端推論主程式
在Arduino IDE安裝Arduino_TensorFlowLite程式庫後，
點擊主選單的[檔案]─[範例]─[Arduino_TensorFlowLite]─[hello_world]產生。
程式會將推論結果由序列埠送回PC，同時板上橘色LED會一直閃爍代表有在通訊，
可使用工具「序列埠監看視窗」觀察數值，或使用「序列繪圖家」直接觀發波形。
*/

// 導入TensorFlowLite相關及必要之頭文件
#include <TensorFlowLite.h> 
#include "main_functions.h"
#include "tensorflow/lite/micro/all_ops_resolver.h" // 提供解釋器用來運行模型的操作。
#include "constants.h" // 定義常用常數
#include "model.h" // 模型頭文件
#include "output_handler.h"
#include "tensorflow/lite/micro/micro_error_reporter.h" //輸出除錯日誌。
#include "tensorflow/lite/micro/micro_interpreter.h" // 包含加載和運行模型的代碼。
#include "tensorflow/lite/schema/schema_generated.h" 包含 TFLite FlatBuffer模型文件格式的架構。
#include "tensorflow/lite/version.h" // 為 TFLite 架構提供版本信息。

// 宣告命名空間，方便後續程式呼叫使用
namespace {
tflite::ErrorReporter* error_reporter = nullptr; // 除錯日誌指標
const tflite::Model* model = nullptr; // 模型指標
tflite::MicroInterpreter* interpreter = nullptr; // 直譯器指標
TfLiteTensor* input = nullptr; // 輸人指標
TfLiteTensor* output = nullptr; // 輸出指標
int inference_count = 0; // 推論計數器

// 宣告全域變數（配置記憶體）
constexpr int kTensorArenaSize = 2000; 
uint8_t tensor_arena[kTensorArenaSize];
}  // namespace

// 設定腳位用途及模組初始化（只在電源啟動或重置時執行一次）
void setup() {
  // 建立除錯日誌
  static tflite::MicroErrorReporter micro_error_reporter;
  error_reporter = &micro_error_reporter; 

  // 載入模型
  model = tflite::GetModel(g_model);
  if (model->version() != TFLITE_SCHEMA_VERSION) {
    TF_LITE_REPORT_ERROR(error_reporter,
                         "Model provided is schema version %d not equal "
                         "to supported version %d.",
                         model->version(), TFLITE_SCHEMA_VERSION);
    return;
  }

  // 建立解析器(resolver)和直譯器(interpreter)
  static tflite::AllOpsResolver resolver; 
  static tflite::MicroInterpreter static_interpreter(
      model, resolver, tensor_arena, kTensorArenaSize, error_reporter);
  interpreter = &static_interpreter;
  
  // 配置張量所需記憶體
  TfLiteStatus allocate_status = interpreter->AllocateTensors();  
  if (allocate_status != kTfLiteOk) {
    TF_LITE_REPORT_ERROR(error_reporter, "AllocateTensors() failed");
    return;
  }

  // 建立輸出入
  input = interpreter->input(0);
  output = interpreter->output(0);

  // 清除推論計數器
  inference_count = 0;
}

// 設定無窮循環程式 （會一直依序重覆執行）
void loop() {
  // 設定輸入值x
  float position = static_cast<float>(inference_count) /
                   static_cast<float>(kInferencesPerCycle);                   
  float x = position * kXrange;
  int8_t x_quantized = x / input->params.scale + input->params.zero_point;
  input->data.int8[0] = x_quantized;

  // 運行推理
  TfLiteStatus invoke_status = interpreter->Invoke();  
  if (invoke_status != kTfLiteOk) {
    TF_LITE_REPORT_ERROR(error_reporter, "Invoke failed on x: %f\n",
                         static_cast<double>(x));
    return;
  }

  // 將結果x,y值輸出到序列埠
  int8_t y_quantized = output->data.int8[0];
  float y = (y_quantized - output->params.zero_point) * output->params.scale;
  HandleOutput(error_reporter, x, y);

  // 推論計數值加1，若超過預設次數就清零
  inference_count += 1;  
  if (inference_count >= kInferencesPerCycle) inference_count = 0;
}
```

参考链接

- Google TensorFlow Lite for Microcontrollers 中文学习指南(https://www.tensorflow.org/lite/microcontrollers?hl=zh-tw)
- Pete Warden Youtube， TinyML Book Screencasts Playlist 影片列表(https://youtu.be/Fdt9xunlyCQ?list=PLtT1eAdRePYoovXJcDkV9RdabZ33H6Di0)
- TensorFlow Lite for Microcontrollers Github 开源码(https://github.com/tensorflow/tflite-micro)
- TFLM Get started with microcontrollers， The Hello World example(https://www.tensorflow.org/lite/microcontrollers/get_started_low_level?hl=zh-tw)
- Experiments with Google = TensorFlow Lite for Microcontrollers 案例分享(https://experiments.withgoogle.com/collection/tfliteformicrocontrollers)

## 6 tinyML开发框架（二）：Arm CMSIS 简介
Arm CMSIS全名Common Microcontroller Software Interface Standard，顾名思义就是Arm为了让自家MCU （IP）能跨厂牌（如STM， NXP， 新唐、盛群等）、跨开发平台（MDK， Keil， IAR， eIQ， STM32CubeIDE， Arduino IDE等）、 跨家族（Cortex-M0/M0+/M1/M3/M4/M7/M23/M33/M35P/M55/A5/A7/A9）所推出来的软件开发接口，减少用户的学习曲线。 它是一种硬件抽象层（Hardware Abstraction Layer， HAL）的概念，就是不直接驱动硬件而采间接驱动的程序写作概念，这样就可达成上位软件写作统一化的好处，类似使用OpenGL，OpenCL跨GPU开发绘图软件一样。

由Fig. 6-1图可看出CMSIS的基本架构，最下方为实际硬件，包括微处理器（CPU，暂存器、旗标等）、通讯外围（I2C， SPI， UART等）、特殊外围（如ADC， DAC， PWM等）及一些除错相关的硬件。 中间层则负责硬件抽象描述，处理所有核心计算及外围信号处理。 上一层则为高级集成性功能，如实时操作系统、神经网络运算等，最后才是组合成完整的应用程序码。 目前CMSIS可以支持的组件非常多，有6009项，其中Cortex-M4最多，占了39.1%，Cortex-M3次之，占25.6%，第三为Cortex-M0+，占14.9%，三者合计占79.6%。 而支持的开发板高达441种，相关软件也多达682项，可见其普及性及便捷性。

![](https://1.bp.blogspot.com/-hUmF0DiGW7c/YVKPqUKGX6I/AAAAAAAAExw/HRRn5cGv9-I12ZBGFjaLDGeAsnI_HpawgCLcBGAsYHQ/s1658/iThome_Day_14_Fig_01.jpg)

Fig. 6-1 ARM CMSIS架构图。 （OmniXRI整理制作， 2021/9/28）

Arm CMSIS为开源项目(https://github.com/ARM-software/CMSIS_5)，主要组成组件（Components）如下所示。

- 核心（Core）：又分为Cortex-M用的Core（M）和Cortex-A用的Core（A），提供标准API及核心运算指令。
- 驱动程序（Driver）：用于连接实际硬件接口的中间软件（Middleware），以实现软件堆栈（Stack）。
- 数字信号处理器（Digital Signal Processor， DSP）：有超过60个以上函数，可处理定点（q7， q15， q31）、32 bit浮点及单指令多资料流（SIMD）指令应用。
- 神经网络（Neural Network， NN）：高性能神经网络计算库，执行tinyML、深度学习重要库。
- 即时操作系统（Real Time Operation System， RTOS）：又分为V1（Cortex-M0/M0+/M3/M4/M7）和V2（Cortex-M/A5/A7/A9）。
- 包装（Pack）：描述软件组件、设备参数和开发板支持的交付机制。 它简化了软件重用和产品生命周期管理 （PLM）。
- 建置（Build）：一组可提高生产力的工具、软件框架和工作流程，例如使用连续整合（CI）。
- 系统视角描述（System View Description， SVD）：可用于在除错工具及创建外设感知设备的描述。
- 除错存取端口（Debug Access Port， DAP）：提供调试存取端口介面所需的的韧体。
- 区域（Zone）：定义方法来描述系统资源并将这些资源划分为多个项目和执行区域。

透过前面章节及上图Fig.6-1可知，Arm Cortex-M4（F）是最适合用来开发tinyML的家族，因为它价格及性能在整个Cortex家族中间，相较M0/M0+/M3等低阶MCU配置有较多的程式码区（Flash）和随机内存（SRAM）可用来储放模型及参数，同时支持定点、浮点（FP16/FP32）及整数（INT8）运算， 也支持单指令周期DSP及SIMD指令集，可让CMSIS-DSP数字信号处理（含高阶数学运算及机器学习常用统计算法）和CMSIS-NN神经网络函式库发挥更多功效。 前面章节使用的Arduino Nano 33 BLE Sense开发板的主芯片内核正是Arm Cortex-M4F。

接下来就帮大家介绍一下CMSIS-DSP和CMSIS-NN的主要组件和功能。

### 6.1 CMSIS-NN 神经网络库

曾提及TensorFlow Lite for Microcontroller （TFLM） 会以CMSIS-NN函式库为基础来产出对应的代码，方便MCU端可以执行。 虽然CMSIS-NN也可运行在没有浮点运算及SIMD指令的Cortex-M0/M0+/M3 CPU上，但其执行效率会和M4以上家族差上数倍到数十倍（含CPU工作时脉速度提升）。

以CMSIS-NN库发展出的tinyML应用，包括有：
- 多层全连接的深度神经网络（Deep Neural Network， DNN）：如智能传感器、手势（动作）辨识、唤醒词检测等应用，
- 卷积神经网络（Convolutional Neural Network， CNN）：如影像分类、对象侦测等视觉应用。

CMSIS-NN库相关函数 ：

- 卷积函数（1x1， 3x3，定点7位、定点15位、有号8位等）
- 激活函数（sigmoid， tanh， relu等）
- 全连接层函数（定点7位、有号8位、无号8位）
- SVDF层函数（有号8位）
- 池化函数（平均、最大，定点7位、有号8位）
- Softmax函数（定点7位、有号8位、无号8位）
- 基本数学函数（元素相加、相乘，重塑形等）

![](https://1.bp.blogspot.com/-T5zNsaQsH9A/YVM9Dz68ZjI/AAAAAAAAEx4/igLgMFNeYD0e6JQA3rHEDPWWQk0ZkMJQQCLcBGAsYHQ/s1658/iThome_Day_14_Fig_02.jpg)

Fig. 6-2 Arm CMSIS-NN库架构图。 （OmniXRI整理绘制， 2021/9/28）

至于如何直接使用CMSIS-NN来开发CNN，可参考下列文章介绍，或者等后面章节再补充介绍。 假若你不想这么深入了解底层运作方式，那使用TFLM或其它tinyML云端一站式整合型开发平台即可。

参考链接

- Arm Common Microcontroller Software Interface Standard （CMSIS）文件说明(https://arm-software.github.io/CMSIS_5/latest/General/html/index.html)
- Arm CMSIS-5 Github开源码(https://github.com/ARM-software/CMSIS_5)
- Arm Developer - Converting a Neural Network for Arm Cortex-M with CMSIS-NN(https://developer.arm.com/documentation/102591/0000)
- Arm Developer - Deploying a Caffe Model on OpenMV using CMSIS-NN(https://developer.arm.com/documentation/ARM0629486814402664/0100)
- Arm Youtube - Image recognition on Arm Cortex-M with CMSIS-NN in 5 steps(https://youtu.be/EkYp0glSenE)
- Arm-Software ML-exsamples - CMSIS-NN CIFAR10 Example(https://github.com/ARM-software/ML-examples/tree/master/cmsisnn-cifar10)
- Arm CMSIS-NN: Efficient Neural Network Kernels for Arm Cortex-M CPUs(https://slideplayer.com/slide/17731119/)

## 7 Arm CMSIS 牛刀小试一下
Arm CMSIS 简介已初步帮大家介绍了Arm CMSIS的基本框架及重要函式库，其中Arm CMSIS-DSP中有很多矩阵及机器学习相关计算，这和tinyML运算效能有很大关系。 如果你是喜欢把机器性能榨干的朋友，或者是吃完咖哩饭还想更了解厨师的到底是用了什么步骤和掺了什么秘方的人，那一定要花点时间了解一下这些底层运作方式。 但如果你只是想好好点个喜爱口味的咖哩饭来享用，或者是只想学习tinyML如何快速应用的朋友，则可忽略过这个章节。

在CMSIS-DSP中和tinyML最相关的莫过于矩阵演算，常用的矩阵加法、减法、乘法、转置和反矩阵计算都有，还有依不同数值精度，提供有浮点数（f16， f32， f64）及定点数（q7， q15， q31）不同函式（注：不是所有函数都有6种数值精度）。 其基本計算如图Fig. 7-1所示。 更完整的函数使用可参考官网CMSIS-DSP-Reference-Matrix Functions(https://arm-software.github.io/CMSIS_5/DSP/html/group__groupMatrix.html)。

![](https://1.bp.blogspot.com/-i9GpaXvmisk/YVQkOsH0dMI/AAAAAAAAEyA/gQUAXHGOIjcuRScpDoysw2V2ctFmQL0DwCLcBGAsYHQ/s1658/iThome_Day_15_Fig_01.jpg)

Fig. 7-1 Arm CMSIS-DSP常用矩阵函数。 （OmniXRI整理制作， 2021/9/29）

在CMSIS-DSP函式中，矩阵相关函数并不是直接使用指标访问数据，而是另外定了一个arm_matrix_instance_xxx结构（xxx表数值精度）来访问，其中包含行数、行数及数据起始位置，而不同的数值精度提供了不同的结构来呼叫。 使用前需直接给定或者通过初始化函数来完成。 写入和读出都透过指标来完成，可参考下列程序段说明。

```c++
// CMSIS-DSP 矩陣運算時所需的資料結構原始定義
// 其定義已宣告在arm_math.h中的dsp/matrix_functions.h
// 不用再主程式或子程式中重新宣告
typedef struct
{
 uint16_t numRows; // 矩陣列數
 uint16_t numCols; // 矩陣行數
 float32_t *pData; // 矩陣資料起點指標
} arm_matrix_instance_f32; // 32bit浮點數矩陣實例結構名稱

// 宣告原始資料內容方式，可為f16, f32, q15, q31等格式陣列
float32_t data[資料數量] = {資料內容};

// 取得資料所在指標方式
float32_t *pData = &data;

// 從原始資料轉換到矩陣實例，以利矩陣相關運算。
arm_matrix_instance_f32 S = {nRows, nColumns, pData};

// 或者可使用初始化函數將原始資料轉入矩陣實例中，其格式可為f16, f32, q15, q31四種。
arm_mat_init_f32 (arm_matrix_instance_f32 *  S,
		          uint16_t     nRows,
		          uint16_t     nColumns,
		          float32_t *  pData 
) 

// 矩陣相關計算完成後，取出矩陣實例S座標(i,j)數值方式
value = S.pDate[i*nColumns + j];
```

在Github上的Arm CMSIS开源码主要不是给所有MCU及开发工具用的，所以有时要自己手动修改一些设定值，原则上可把这些/include下的头文件（.h）及/source下的代码（.c）直接复制到各家MCU开发工具上，再进行编译即可，有些可能需要修改一些参数，详见各函式库的 readme.md 说明。

下面为一个简单的矩阵乘法测试，准备两个矩阵资料，相乘后得到X=AxB，再将结果从序列监视控制台秀出结果。

```c++
/*
  CMSIS-DSP_Test.ino （for Arduino）
  測試在Arduino IDE下使用Arm CMSIS-DSP函式庫
  使用前需先安裝Arduino_CMSIS-DSP函式庫（最新版本為5.7版）
  求矩陣A乘以矩B得到結果X = A x B
  
  程式作者：OmniXRI Jack, 2021/9/29
*/

#include "arm_math.h" // 導入CMSIS-DSP中最主要的函式頭文件

// 宣告一組32 bit浮點數陣列A，作為輸入值，大小為1x4
const float32_t A_f32[4] =
{
  1.0, 2.0, 3.0, 4.0
};

// 宣告一組32 bit浮點數陣列B，作為輸入值，大小為4x4
const float32_t B_f32[16] =
{
  1.0,  2.0,  3.0,  4.0,
  5.0,  6.0,  7.0,  8.0,
  9.0,  10.0, 11.0, 12.0,
  13.0, 14.0, 15.0, 16.0,
};

// 宣告一組32 bit浮點數陣列X，存放輸出值，大小為1x4
float32_t X_f32[4];

arm_status status;  // CMSIS-DSP運算結果狀態
arm_matrix_instance_f32 A; // 宣告CMSIS-DSP 32bit浮點數矩陣實例指標
arm_matrix_instance_f32 B; // 宣告CMSIS-DSP 32bit浮點數矩陣實例指標
arm_matrix_instance_f32 X; // 宣告CMSIS-DSP 32bit浮點數矩陣實例指標
uint32_t srcRows, srcColumns; // 宣告行(Columns)、列(Rows)數變數

void setup() {
  Serial.begin(9600); // 初始化序列埠，方便監看輸出結果
  
  // 將列陣A_f32初始化為矩陣實例A
  srcRows = 1;
  srcColumns = 4;
  arm_mat_init_f32(&A, srcRows, srcColumns, (float32_t *)A_f32);
  
  // 將列陣B_f32初始化為矩陣實例B
  srcRows = 4;
  srcColumns = 4;
  arm_mat_init_f32(&B, srcRows, srcColumns, (float32_t *)B_f32);
  
  // 將列陣X_f32初始化為矩陣實例X
  srcRows = 1;
  srcColumns = 4;
  arm_mat_init_f32(&X, srcRows, srcColumns, (float32_t *)X_f32);
}

void loop() {
  // 計算矩陣A乘以矩陣B並將結果存放在X中
  status = arm_mat_mult_f32(&A, &B, &X);

  // 將計算結果從序列埠監控視窗印出
  // 得到字串 X[0,0] = 90.00	X[0,1] = 100.00	X[0,2] = 110.00	X[0,3] = 120.00	
  for(int i=0; i<srcRows; i++){      // 指定結果矩陣列數
    for(int j=0; j<srcColumns; j++){ // 指定結果矩陣行數
      Serial.print("X[");
      Serial.print(i);
      Serial.print(",");
      Serial.print(j);
      Serial.print("] = ");
      Serial.print(X.pData[j]); // 將矩陣實例中的結果取出，轉回float32_t格式
      Serial.print("\t"); // 列印TAB（跳格）指令
    }
    Serial.print("\n"); // 列印換行指令
  } 

  delay(3000); // 為方便觀察，讓程式重覆每隔3秒重新執行一次，通常會以 While(1); 程式碼將程式停住。
}
```

其它矩阵相关函式可自己玩看看，这里就不多作说明了。

参考链接

- Arm CMSIS DSP Software Library(https://arm-software.github.io/CMSIS_5/DSP/html/index.html)
- Arduino-Reference-Libraries-Arduino_CMSIS-DSP （注：官方说明文件已停用）(https://www.arduino.cc/reference/en/libraries/arduino_cmsis-dsp/)
