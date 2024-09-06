---
layout: post
title: Intel FPGA开发流程
date: 2024-09-06
author: lau
tags: [FPGA,Archive]
comments: true
toc: false
pinned: false
---
Intel FPGA开发流程

<!-- more -->

本系列将带来FPGA的系统性学习，从最基本的[数字电路](https://www.eefocus.com/baike/1477913.html)基础开始，最详细操作步骤，最直白的言语描述，手把手的“傻瓜式”讲解，让电子、信息、[通信](https://www.eefocus.com/tag/%E9%80%9A%E4%BF%A1/)类专业学生、初入职场小白及打算进阶提升的职业开发者都可以有系统性学习的机会。

系统性的掌握技术开发以及相关要求，对个人就业以及职业发展都有着潜在的帮助，希望对大家有所帮助。后续会陆续更新 Xilinx 的 Vivado、ISE 及相关操作软件的开发的相关内容，学习FPGA设计方法及设计思想的同时，实操结合各类操作软件，会让你在技术学习道路上无比的顺畅，告别技术学习小BUG卡破脑壳，告别目前忽悠性的培训诱导，真正的去学习去实战应用，这种快乐试试你就会懂的。话不多说，上货。

### Intel FPGA开发流程

本篇将设计一个简单的二输入与门，来讲解整个设计流程。至于设计语言就不在单独列出一个章节去做特殊说明，语法、操作、实验将同时讲解，这样更具有带入性，便于读者阅读和学习。

## 1**************设计前准备**************

在设计之前我们需要在两个方面进行准备：硬件方面和软件方面。

**硬件方面**

开发FPGA设计，最终的产品是要落在使用FPGA[芯片](https://www.eefocus.com/tag/%E8%8A%AF%E7%89%87/)完成某种功能。所以我们首先需要一个带有Intel FPGA芯片的[开发板](https://www.eefocus.com/tag/%E5%BC%80%E5%8F%91%E6%9D%BF/)。

本文中设计将采用CYCLONE系列FPGA进行讲解，如果读者有其他系列（必须是Intel FPGA，否则无法在[Quartus](https://www.eefocus.com/baike/1533188.html)上开发），也可以进行学习，不同系列的开发流程基本相同。

**软件方面 **

我们需要综合工具-quartus 软件和仿真工具-modelsim软件。正确安装这两个软件是开发Intel FPGA的必要条件。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaEMCiabnVfoTcr2frerd3XDxW0YaySn46IzGnlJUdDaCHvzcmLSZVH2w%2F640%3Fwx_fmt%3Dpng&s=112156)**图1 ：Quartus 软件图标**

按照《quaruts prime 18.0标准版安装与破解-郝旭帅团队V1》的方式安装，Modelsim软件的图标将不会出现在桌面上。并不是没有安装上，而是此时Quartus软件认为Modelsim软件只是它的一个小程序包而已，所以就没有体现单独的软件图标。

当准备好软件后，笔者向大家推荐一个工程管理的方式，也是本文中做工程管理的方式，这样会比较清楚。

做设计的话，会编写很多设计文件、仿真文件以及综合器会给我输出很多的过程文件等等，那么这些文件最好都能有自己的一个“归宿”，不要乱放。

路径和命名中不允许出现非法字符（合法字符包括：数字、字母、下划线。特别说明：空格是非法的），笔者建议以字母开头。

我们是做FPGA开发设计的，首先我们将建立一个文件夹，专门用来放FPGA开发设计。例如：E：/fpga_design。

在后续的开发设计中，我们会做各种各样的设计。每个设计都有自己的名字，在上述文件内，用实验的名字命名一个文件夹。名字的话一定要带有某种含义，不建议随便给个字母序列当做名字。例如：第一个要做的二输入与门的设计，命名为and_gate2_1。

在建立完某项设计的文件后，依次在其里面新建四个文件夹，分别为：rtl、qprj、msim、doc。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaJH5rpSbUf9qDnTvnVEiaHByulHhU230xuujuKX4KFtiabicSjXSolVYzg%2F640%3Fwx_fmt%3Dpng&s=0e87dc)

**图 2 ：工程管理文件脉络示意图**

rtl文件夹用于存放设计的源文件。

doc文件夹用于存放设计的一些文档性的资料。

qprj文件夹用于存放quaruts 工程以及quartus生成的一些过程性文件。

msim文件夹用于存放仿真文件。

在 FPGA 设计时，主要是这四个文件的使用。某些时候我们也会新加一些文件，例如：FPGA板卡需要我们设计自己时，就会多一个文件夹[PCB](https://www.eefocus.com/tag/pcb/)，用于存放PCB相关的工程或者源文件等。

## 2**************建立工程**************

做好设计前准备后，就可以开始建立quartus 工程了。

在做设计时，都是以工程为主体的设计。在没有工程的情况下，利用quartus软件打开设计源文件等，也是不支持编译和综合的。

双击quartus 软件的图标（图4-1），打开quartus软件。在有的电脑上，软件启动的速度不是很快（在确保自己已经双击打开的前提下，可以等待1分钟），不要多次去双击图标，容易造成PC卡死或者启动了很多个quartus软件。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaTnicME9OrOniaiaNEfYDzvLJGBWhvynhO8D9Nlb3UGwjgjml70gJNN0mA%2F640%3Fwx_fmt%3Dpng&s=7c98d7)

**图3 ：quartus 18.0界面**

笔者这里不对每个界面进行单独介绍，后续用到那个功能或者界面时，会单独介绍说明。

点击左上角File，选择New Project Wizard…。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaMc4pRFu4wpqdCMcXQaLSRQHnZxjuhDpFmqW93wuiccDUic7LPwic9yARg%2F640%3Fwx_fmt%3Dpng&s=abda6a)

**图4 ：打开新工程创建向导**

打开新工程向导，首先出现一个工程向导介绍说明。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Weawa7cib9iaMPBh3IzKJWiaZAZ54ZOoYPDSzTVy6e9uj1eNWeDtPFuFM81Q%2F640%3Fwx_fmt%3Dpng&s=3f7428)

**图5 ：工程向导的介绍界面**

在工程向导中，我们会指定工程名称和位置，顶层实体的名称，工程文件和库文件，目标器件，[EDA](https://www.eefocus.com/tag/EDA/)工具。

在复杂设计时，会将电路分成各个小模块去做设计，最终还需要一个大模块将这些小模块包括进来，对外呈现都是大模块的接口。此时，这个大模块就是顶层实体（TOP level entity）。如果设计中只有一个模块，那么这个模块就是顶层实体。

点击Next。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaQTzK2QRP46oY5YLKAE5LAZTETkiaTp6X06GfKR6g7VYn1zqgK1ZJQXg%2F640%3Fwx_fmt%3Dpng&s=99d5c1)

**图6 ：指定工程位置、工程名称和顶层实体的界面**

将工程的位置指定到之前我们准备好的文件夹（qprj）中。点击编辑框后面的三个小点，进行文件搜索指定。

工程的名称就是采用之前我们做的设计文件夹的名字，这个名字可以是任意的，笔者建议和文件夹保持一致，因为当初建立文件夹时，就是选择用工程的名字。直接输入工程名称and_gate2_1即可。

顶层实体的名字会自动出现，与工程的名字保持一致。我们也可以重新指定一个新名字，笔者建议与工程名字保持一致。

点击Next。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea75KMI8b8Dw8MNzQIcOiczF1aWXrRSjvIGJQHRRM8SbNWq1sYZbOxISQ%2F640%3Fwx_fmt%3Dpng&s=8a84c4)

**图7 ：选择建立的工程的类型**

选择空白工程（默认空白工程），点击Next。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaYe2FFicDbVMS2iaOoexGAOichpRibP9gZKC7vpn1VuiajKs3Wib8CRKRzN8A%2F640%3Fwx_fmt%3Dpng&s=bbe7e7)

**图8 ：添加文件**

建立工程时，我们可以直接向工程中添加已有的文件。一般我们选择什么都不添加，后续设计中，如果有提前做好的文件，也是选择什么都不添加。建立完工程后，依然可以向工程添加文件。

点击Next。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaLcwxYThGkAFr7m35Hop38pJUwKwgwfica2Q85bwia27Gphu0as75sL7w%2F640%3Fwx_fmt%3Dpng&s=1d24ed)

**图9 ：选择目标器件**

FPGA设计最终是要落实到芯片内部，在这里要选择对应的芯片（自己手里开发板的FPGA芯片）。芯片的型号在FPAG的芯片上有描述，如果芯片上看不清楚，或者芯片在被其他东西挡住，可以查看开发板的资料，一般都有介绍。笔者手中开发板的FPGA的型号为EP4CE10F17C8N。后续所有的选择将按照笔者手中的型号进行设计和选择，读者不相同的，请自行改动。如果暂时还没有开发板的读者，可以跟着笔者选择继续下面的步骤（没有开发板的话，后面有一些步骤是做不了的）。

选择时，首先选择对应的系列。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Weah4gr3S8H1eA2C8hcMQibWXRMI3lvGxE00icg4FJn4BrvR5xibtpIso65w%2F640%3Fwx_fmt%3Dpng&s=d2d25c)

**图10 ：选择芯片所对应的系列**

选择对应的系列后，可以看到下面的器件列表（此列表的窗口是可以拉大的，可以直接扩大整个界面）中，就出现很多的器件。呈现的器件都是按照一定的规律进行排列的，可以很快的找到自己的芯片，然后单击选择芯片（先不考虑为什么没有EP4CE10F17C8N，而选择EP4CE10F17C8）。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Weanv7XLeiciagcxkp5F32WCwT3ICZlxomj1f5ibb2JG5KLOCwPpJNQMpjHA%2F640%3Fwx_fmt%3Dpng&s=b6c852)

**图11 ：器件列表**

如果每次新建工程都是这样去寻找芯片的话，是有一定的累人。好在这个软件给我们提供了筛选面板，我们可以把筛选条件输入进去。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea8SE7DIp489dviayaPnvaiag0BEn6NrX3P1oq19n3TBAZZU2fVDpgRa2g%2F640%3Fwx_fmt%3Dpng&s=7c0c0a)

**图12 ：器件筛选**

   设置好筛选条件后，器件列表中就只有四个器件了，很容易就可以找到我们这款芯片。

筛选条件中有[封装](https://www.eefocus.com/baike/492719.html)类型、管脚数目和速度等级。那么我们怎么知道自己手中的芯片的这些信息呢？

答案都隐藏在芯片的名字中。笔者手中的是CYCLONE IV系列的片子，下面就介绍一下这个系列的命名方式。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeavgcrKU0sPLyruK4tswjJwHLM7TvRZSs0QSSQWrXr1f9zI8yQohxRQw%2F640%3Fwx_fmt%3Dpng&s=547d4a)

**图13 ：CYCLONE IV E系列命名规则**

如果是其他系列，请自行参考所对应的芯片手册。

通过观察图4-13，EP4CE10F17C8N中末位N，只是表示无铅封装，和具体内部机构没有任何关系。故而选择不带N的，也是可以的。

注意无论是哪种方法，最后找到自己想要的芯片后，一定要点击它，选中它。

点击Next。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea0vmqgiaddSM3vmLvt5rutvs2lLk4CEHiclfhrl06K3pic2jXibl0FFOZvA%2F640%3Fwx_fmt%3Dpng&s=ca2e14)

**图14 ：选择EDA工具**

Quartus软件是一个综合工具，他可以关联一些其他的工具协助设计FPGA。在这里我们在simulation一栏，工具选择modelsim-altera，格式选择[verilog](https://www.eefocus.com/tag/verilog/) HDL。其他保持默认。

点击Next。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeanRfibPcPaONAmO8gCIfTUB2UYmDD7fBntJfAJ1mz75OkUDMH7bLCw9w%2F640%3Fwx_fmt%3Dpng&s=fd06e4)

**图15 ：工程向导配置总结**

这个总结显示出在新工程向导中，我们所做的所有的设置。大家可以检查一下，如果发现那一项和自己的要求不一致，就需要点击back，修改后，在回到此步骤。

点击Finsh，完成工程创建。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea6ngHcibUOu9cZibGrnPsTuZWGRKwC30YftZaHwPd273ZfIrjdQdAl6kg%2F640%3Fwx_fmt%3Dpng&s=99ee14)

**图16 ：工程建立完成后，工程向导界面**

工程建立完成后，工程向导界面显示出选择的器件和指定的顶层实体。

打开qprj文件夹。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea25ljajHWYVLMDkib6K9Q35QdDpbbCO4xJJ8iaTicEibzUEIV63HvO28ceQ%2F640%3Fwx_fmt%3Dpng&s=a175f0)

**图17 ：建立完工程后的qprj文件夹**

db文件夹为基础数据文件夹。

.qpf为quartus project file，quartus 工程文件。如果此时将quatus关闭了，双击此文件就可以打开工程。

.qsf为quartus settings file，quartus 设置文件。在quaruts里面做的大部分操作都会记录到此文件中。

Quarus 的版本很多，如果用一个版本建立的工程，用另外一个版本打开可能会出一些bug，所以建议采用使用已建好工程的版本打开。可以使用记事本的方式打开.qsf。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeabSzicx3rutVauGEticMjLq7fdogKDKib558PfbFD98A5ASdcgRGXIz3AQ%2F640%3Fwx_fmt%3Dpng&s=5f98db)

**图18 ：qsf文件的一部分**

通过查看.qsf文件，可以了解到工程的最初用什么版本建立，最后用什么版本打开（打开时，建议采用最后的版本打开）。

打开工程的方式，不建议采用双击.qpf文件。有时间一个PC上面，会有多个quaruts软件的情况，如果直接双击，就会采用某一个版本打开，这不一定是我们想要的。

建议打开工程的方式，首先查看应该选择的版本，启动对应的版本quartus软件，点击File，选择Open project（不要点击Open），找到工程，启动。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaI63Y4dvpM5CUd7hCqAIKC20IDDYFL044D69Yk2ic28q00wicrZ4dSAcA%2F640%3Fwx_fmt%3Dpng&s=33010e)

**图19 ：打开工程的途径**

### 3**************输入设计**************

当建立完工程后，就可以输入设计。输入设计的方法有三种：[原理图](https://www.eefocus.com/tag/%E5%8E%9F%E7%90%86%E5%9B%BE/)输入、HDL代码输入、原理图和HDL代码混入输入。

**原理图输入**

原理图输入就是以现有模块级联的方式实现功能。

点击File，选择New。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaOzZCNdnkFqVTgBsibMPZibffNMGyl4pIFhWketoUGhjUC2jufg8pTYPw%2F640%3Fwx_fmt%3Dpng&s=dd32e6)

**图20 ：新建模块**

 选择design file中的Block Diagram/Schematic File。点击ok。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea3npiaT8YFarJXatibGgcyYDOtibnjK4eaeGNLdicrmicH0wdlxicEqE9Yoqw%2F640%3Fwx_fmt%3Dpng&s=8a07c3)

**图21 ：新建文件列表**

对于新建的bdf文件立刻进行另存为。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeapJNB1zN4ibEyib3eJaNwK74Br7F7KggH6jtnrTFvzo5bmZH1ooBpnxEg%2F640%3Fwx_fmt%3Dpng&s=2b2a14)

**图22 ：另存为**

另存为到qprj文件夹。Bdf文件是只有quartus软件才能认可，不具有移植性，把它放到工程文件夹中。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaPibiaWgmxr5icjmic6uYzbfd7VMI1Bicvg9DRSNkSHomyPUtXWPsFtmvzfw%2F640%3Fwx_fmt%3Dpng&s=99e8ff)

**图23 ：保存路径和名称**

点击保存。特别注意保存路径和保存名称。不要盲目直接点击保存，一定要再三确认。

在BDF文件的空白位置，双击，打开添加原件的窗口。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Weaw6MUib54Q4ZZQpWr4JUz8P0w1uPAew3SDqp9Wd4bicq9ttPedkwXl3ow%2F640%3Fwx_fmt%3Dpng&s=2f8b74)

**图24 ：打开添加原件的窗口**

在libraries中，选择and2，点击ok。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Weanh5lmydtDPjnNKGxo8I4HmrXvHlwzmH3tfXtoRxM0z4oTn2kDuq3YA%2F640%3Fwx_fmt%3Dpng&s=4823e6)

**图25 ：添加二输入与门**

鼠标会拖动一个二输入与门的符号，此时点击鼠标左键，一个二输入的与门就放置在文件中了。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeawxkicgNQcCbOic4D6dSlJSQ2G3pR1yQg9o8bkvGO4iaMEemDicjxNUNGJw%2F640%3Fwx_fmt%3Dpng&s=16f512)

**图26 ：放置的二输入与门**

当添加完二输入与门后，也就意味着芯片内部的设计完成。现在需要将内部设计与外部相连接。芯片与外界相连接靠的是IO。

双击空白处，选择input（输入），点击ok。添加输入IO。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeafgyKpILA8txW6xRXjO5VwDEFv5SwHJWTgGo17FRbTjSoj2CTDdfMtw%2F640%3Fwx_fmt%3Dpng&s=441da0)

**图27 ：添加input管脚**

二输入与门，要有两个输入。同样的方法，再次添加一个。同时也要有一个输出，选择output（输出）（在input下面），添加一个输出。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeagtXgxXDjGqs1z1bOBaiaicMbgT4raGZ8BVdCpgtxnQqdvtEriaLF25N7w%2F640%3Fwx_fmt%3Dpng&s=252ff8)

**图28 ：添加两个输入和一个输出**

将管脚和二输入与门进行相连接。

把鼠标移至需要连接线的地方时，鼠标会自动变为十字状（不是十字状箭头），并且右下带有一个“7”形的导管。然后按住鼠标左键进行导线的引出。当连接到另外一个接口时，

就会出现一个小方框，证明已经连接上，此时释放鼠标左键即可。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaGZo4zht1ibNwgTNA6ibW7leNIHE8gQKibEWR8I2dhZvvEDiaBCfsU6qEiaQ%2F640%3Fwx_fmt%3Dpng&s=8dc75d)

**图29 ：连接完成图样**

此时默认两个输入的管脚的名字为pin_name1和pin_name2。在设计时，二输入与门我们将其输入命名为一个为A，一个为B，输出为Y。

右击pin_name1,选择properties。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaACc8o7MzbRlBEDgY27N9XHg2lXicVoyVgfl5sBvONITiatJsaBnJ1y1g%2F640%3Fwx_fmt%3Dpng&s=9cbd34)

**图30 ：选择属性**

将pin name改为A。点击ok。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaicxeunXHH9A5vYeXk76NFyFQTTramDMKLJEnTKEIzmIz9iaI5QuZzPLg%2F640%3Fwx_fmt%3Dpng&s=87e48e)

**图31 ：修改管脚名称**

将其他两个管脚也做对应修改。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeatlNz7SzoxxBfXiaeD6T9fjARn32GfK7JTMlpYxrHdKMJtwnLdTD3edA%2F640%3Fwx_fmt%3Dpng&s=7e613c)

**图32 ：修改完管脚名称的图样**

此时点击保存（ctrl + s），使用原理图的输入方式就完成了。

## **HDL代码输入**

用[计算机](https://www.eefocus.com/tag/%E8%AE%A1%E7%AE%97%E6%9C%BA/)语言设计一个数字电路系统，其实质就是用一种语言描述一个硬件模型，因此这样的语言又称为硬件描述语言（Hardware Description Language），或使用缩写HDL。虽然现在HDL已经有多种语言版本，而且还在发展中。但是在本书讨论的HDL仅包括现在最常使用的Verilog HDL和VHDL两种语言系统。

目前在国内做FPGA设计的公司中，使用Verilog HDL占据大多数，故而本书以Verilog HDL为主，在后续的章节中，专门设置一章来讲解VHDL。

选择File，New，选择design file中的verilog HDL file。点击ok。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaNCuCML7WRlWalJLpCAGGfcibx8ldmzYyrzZU1vqmxDIUlxsenOfTXOg%2F640%3Fwx_fmt%3Dpng&s=7ad3ba)

**图33 ：新建verilog HDL file**

新建完成后，立刻另存为，保存到rtl中。hdl文件的移植性比较高，无论在哪个平台都是通用的。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaMaiaIqmaxHrVBVnZTqNc5PJD5fUHzLosDRVibic9o7HEDPCwlrMnScibSw%2F640%3Fwx_fmt%3Dpng&s=c98f6d)

**图34 ：保存verilog HDL文件**

保存时，注意名字和保存路径。Verilog文件的后缀为.v。

建立完，verilog HDL文件后，就需要输入二输入与门所对应的verilog代码了。

Verilog 语法和C很相似，学习起来比较容易。下面我们按照做电路的方式讲解verilog语言。

做电路的话，首先需要拿出一块打的面包板，剪出合适大小的一块。相当于圈了一个地方，做设计只能在这块区域内。

对于verilog语言来说，需要用module和endmodule圈出一个区域，设计代码只能在这块区域中。Verilog语言区分大小写，我们一律采用小写。module和endmodule是verilog的关键字，在综合器中会变蓝。如果endmodule没有变蓝，请多打一个回车或者空格。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeauSzBbx36SASw3mK0nNQm4p5KCjl8YcVP3SLhh7d5UicYGV0Bcaqk0Tw%2F640%3Fwx_fmt%3Dpng&s=dce3e7)

**图35 ：verilog第一步**

当剪出合适大小的面包板后，需要其上面写一个名字。后面应用也好，说起来也好，好歹有个名字。

在verilog中，也需要有一个module name。

在verilog中命名的话，需要遵从一定的规则。由字母、数字、下划线构成；建议字母开头；不能够与verilog的关键字相同；命名是要有一定的意义。

对于module name来说，一般还有一个要求，与文件名称保持一致。那么此时我们要做二输入与门，文件名称是and_gate2_1。要求module name也写成and_gate2_1。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaIIPYZVfdSKQ6IaFMfLpIQb0WcSvFVgictqTGKoPKDunQFCOQianuWOkw%2F640%3Fwx_fmt%3Dpng&s=b1408b)

**图36 ：verilog第二步**

当对面包板命名后，需要给它添加输入和输出的端口（合理的布局接口）。

二输入与门有两个输入，一个为A，另外一个为B；一个输出为Y。在verilog中，布置接口的方式有两种。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaZ211ibicU30v3bxNGcqcXX8M9Y4rQc9S0gNMSdLrP77hY4uULFZhm4KA%2F640%3Fwx_fmt%3Dpng&s=32d57f)

**图37 ：布置接口的第一种方式**

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaT2U46zGuILldia1vskCaMlicWmMdhoYe0icicXibv7YTJOPrVAjC9YRnDMA%2F640%3Fwx_fmt%3Dpng&s=a1522a)

**图38 ：布置接口的第二种方式**

在verilog中，module name（and_gate2_1）之后的那个括号中的内容成为端口列表。

Verilog布置接口的第一种方式为1995标准，第二种方式为2003标准。目前大多数平台都可以支持这两种方式。笔者建议用2003标准。

端口列表中，描述端口时，用逗号隔开，最后一个端口后面不加逗号。在端口列表的括号后面有一个分号。

对于描述端口来说，有最基本的四项：方向、类型、位宽、名称。

input表示输入，output表示输出，inout表示输入输出。

类型中比较常用的有两种：一种是wire，另外一种reg。wire类型时，wire可以省略不写。另外input必须是wire类型。笔者建议wire不省略，都写上。

在做电路时，位宽表示有几根线。有时候为了方便，会将同一种类的线进行同时命名，此时就需要用到位宽。例如：5位的地址线。可以单独命名5次，但是比较麻烦。位宽用中括号括起来，例：[3:0]，[3:1]，[2:5]。如果位宽为1的话，省略不写。笔者建议位宽的右侧为0，左侧为位宽减一。

名称就是为这个输入命名了一个名字。命名时要遵从verilog命名规则。

在做完端口后，需要在面包板上做出符合功能的设计，然后用连接线将设计和输入输出管脚相连接。

二输入与门的设计是需要在中间放一个[组合逻辑电路](https://www.eefocus.com/baike/503889.html)二输入与门。

Verilog中，描述组合逻辑的第一种方式是利用assign语句进行描述。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaUl63IWCoawTV3fOcFJDVJ1WoDuVxVG27DWZpoXgp2jkR7UEVHMNiciaQ%2F640%3Fwx_fmt%3Dpng&s=5e639b)

**图39 ：assign语句描述二输入与门**

assign语句要求被赋值变量（Y）为wire类型，中间采用阻塞赋值（=）的方式，最后面是赋值表达式，在verilog中，算术与用&来表示（后续介绍算术运算和逻辑运算的区别）。

至此，二输入与门的HDL输入就完成了。

**原理图和HDL代码混入输入**

在复杂设计时，我们可以用HDL代码生成底层模块，用原理图的方式，将底层模块进行连接。此方式在后续的章节中介绍。

## 4**************综合和分析**************

当设计输入完成后，需要对设计进行综合分析，同时也检查一下其中是否存在错误。可以单击综合分析按钮，可以双击综合分析选项，也可以利用快捷键（Ctrl + K）。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeakKQn9tCplzVZjMaSw4TdO98xIGvaYOYJ3F7T9EwR0tlMmicpapic6sQQ%2F640%3Fwx_fmt%3Dpng&s=81aa2d)

**图40 ：综合分析按钮**

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaO6hQnpFkbLc7f2yQmPltNGKo7iahkgwSvvFoOjHRcGmO3vtynS32USA%2F640%3Fwx_fmt%3Dpng&s=929755)

**图41 ：综合分析选项**

进行综合分析时，有时会提出一个提示：

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaMnALdXGRpJ8Uic8YczLelpT0eJ44CP6AsG0wwqE0uE0GWAKVgZicfTmg%2F640%3Fwx_fmt%3Dpng&s=2ad9d5)

**图42 ：某文件被改变，是否要保存**

出现上述提示，就证明我们在设计时，修改了某些文件后，没有点击保存。此时点击Yes即可。但是这是一个非常不好的习惯，建议大家修改完任何设计都要及时保存。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaZMianCLbRfEm8XaH7UcGgNMRB92ntsYpQT8P3LfTGVZrSEXLfK9dPng%2F640%3Fwx_fmt%3Dpng&s=bdffa0)

**图43 ：Task的综合分析前面的进度条和后面的已用时间**

经过一段时间后，会出现错误。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaDf415tpJIcSnjUw2nggib92fTIVic0ibOXvEBWx0aNHroVpvNCUz13vHw%2F640%3Fwx_fmt%3Dpng&s=9ed65b)

**图44 ：出现错误**

出现错误后，可以观察quartus 的下部massage窗口。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea8fyM4ib6icTev8qwmLvvp4OrXsORzk59PDbEIpTNdaGqpEZ5VmwqdSQQ%2F640%3Fwx_fmt%3Dpng&s=7e2993)

**图45 ：massages窗口报错信息**

报错信息为：错误(12049):无法将实体“and_gate2_1”的重复声明编译到库“work”中。报错的原因是我们在本工程中声明了多个and_gate2_1，在电路中是不允许出现多个电路模块是同样的名字。

在设计时，为了演示原理图输入和HDL代码输入，工程中存在原理图输入的and_gate2_1和HDL代码输入的and_gate2_1。

在工程向导界面，选择Hierarchy，选择Files。Hierarchy为结构，显示工程的电路模块的结构；Files为文件，显示工程中存放的所有文件。可以利用此方式查看，工程中是否存在多个and_gate2_1。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WealAcDRKQImnvLib3944icpFfzic5dwml4BEORrs1qJH12czswlWCpxuoAA%2F640%3Fwx_fmt%3Dpng&s=990c09)

**图46 ：选择Files界面**

在Files界面中，可以发现工程中确实存在两个and_gate2_1。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaSFKQDvTD1wziaolb7PEGdW2UArq7lk50iaawDHDAtHl2Ar52PtS6cWmg%2F640%3Fwx_fmt%3Dpng&s=28759c) **图47 ：Files界面**

此时右键选择第and_gate2_1.v，选择Remove file from project。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaNkR0G7wagic9XvjZGhY7CJVGPJrfc9rjUAf3PWdEe2BiclpgPIIQRO9Q%2F640%3Fwx_fmt%3Dpng&s=da33f7)

**图48 ：选择移除**

随后在确定界面点击Yes。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaoL4phudMKnpAXQicUlMm6yDGufCmAOteJ90z1o4bSpU8KP257bicGibxw%2F640%3Fwx_fmt%3Dpng&s=e9607b)

**图49 ：确认移除**

将其移除工程后，就会只剩下一个原理图输入的and_gate2_1。然后进行综合分析，等待结果完成。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaXGIYZnOkkCVrRGIqJTSiazVjia0AZ19npvsib033g3sJhW8hl0UNCemSQ%2F640%3Fwx_fmt%3Dpng&s=1b663b)

**图50 ：综合分析成功**

如果中间有错误的话，请参考输入设计中的原理图输入，查看自己的步骤是否正确。

再一次选择到Files界面，将and_gate2_1.bdf移除。将HDL代码输入的and_gate2_1.v

重新添加回来。在Files上右键，选择Add/Remove Files in project（添加或移除文件）。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaB1kEiau4iaBpXaKMczrGqCJlia0G9lekZ4nQMM8DdwpBGubq4YpAzMjzA%2F640%3Fwx_fmt%3Dpng&s=63ecc8)

**图51 ：从工程中添加或移除文件**

选择编辑框后面三个点。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea7CniciaDTuX6aesmOZiaIuZjeKaOFic9S0mj4IUzsZoQ9Io8FYk8MahsFw%2F640%3Fwx_fmt%3Dpng&s=7d7fd1)

**图52 ：寻找添加文件**

找到and_gate2_1.v，选中，点击打开。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeafbpqpW3rUicQQGaQtI470wHibKMWdLvVpd6iaFc3bofzsFtcdibBficlAHw%2F640%3Fwx_fmt%3Dpng&s=bc0cf2)

**图53 ：添加选中文件**

注意添加文件的路径，and_gate2_1.v是存放在rtl文件夹中。

然后点击apply，OK。就添加成功。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea6Pv8CicB1OxPzLOXCBpa4N7SSN7nzTNtWbg7J9dzfmALOmr0FoGmic1A%2F640%3Fwx_fmt%3Dpng&s=529cef)

**图54 ：确认添加文件**

点击综合分析，确认综合分析成功。如果综合分析失败，请参考输入设计中的HDL输入，查找错误的地方。

无论是哪一种输入方式，综合分析成功。双击RTL视图选项，打开RTL视图，查看quartus综合出的电路模型。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeawXhqIGbS9jWs88YGoxSp1hJEiawbV5VLHBlcO2KqAWqOmeqpEialsL2w%2F640%3Fwx_fmt%3Dpng&s=1d5b18)

**图55 ：RTL视图的选项**

在RTL视图中，综合出来的[电路图](https://www.eefocus.com/tag/%E7%94%B5%E8%B7%AF%E5%9B%BE/)，只是电路模型而已。在FPGA中是没有与门的，有的只是LUT等效的二输入与[门电路](https://www.eefocus.com/baike/528785.html)。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaZZuM3buzSskJ8FuDib8ZGwW7n4EvpTiaBVgQunUdSos6JiaXf8Go8ybPQ%2F640%3Fwx_fmt%3Dpng&s=20d180)

**图56 ：RTL视图的二输入与门**

综合分析成功后，会产生一个报告。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea6stBwXy2drUHuV4Ho0jsETYJNzVFXtuvO3eRdVdMKFnibWTECVCd8Zw%2F640%3Fwx_fmt%3Dpng&s=92a472)

**图57 ：综合分析报告**

在报告中，可以看出综合状态、软件信息、工程版本信息、顶层实体、器件系列、目标器件、时序模型、逻辑单元数量、[寄存器](https://www.eefocus.com/baike/502591.html)数量、管脚数量、虚拟管脚数量、[存储器](https://www.eefocus.com/tag/%E5%AD%98%E5%82%A8%E5%99%A8/)大小、[嵌入式](https://www.eefocus.com/tag/%E5%B5%8C%E5%85%A5%E5%BC%8F/)乘法器的使用个数、[锁相环](https://www.eefocus.com/baike/481846.html)使用个数。

只是设计了一个二输入的与门，所以使用一个逻辑单元，3个管脚，其他都没有涉及到。

## 5**************RTL仿真  **************

在综合分析完成后，对于简单的设计，通过查看RTL视图中综合出来的电路模型，就能够知道所做设计是否正确。但是对于复杂的设计，电路模型比较复杂，无法直接判断是否设计正确。

如果直接将不知道正确与否的设计加载到板卡中，很多时候无法通过结果直接看出来是否设计比较严谨。所以要求设计者在软件环境下对所做设计进行一定的测试。

仿真是利用模型复现实际系统中发生的本质过程，并通过对系统模型的实验来研究存在的或设计中的系统。

当所研究的系统造价昂贵、实验的危害性大或需要很长的时间才能了解系统参数变化所引起的后果时，仿真是一种特别有效的研究手段。

仿真其实就是模拟实际情况。对于电路来说，就是给予合适的输入，观测输出是否和设计时所预想的相同。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaalDeFpCyxic0E56PgFsT34cRfO8aWBoic5gYq8eXuOCEpXuKvTaib8QKw%2F640%3Fwx_fmt%3Dpng&s=b18288)

**图58 ：仿真的示意图**

电路的输入、中间过程和输出，都是[数字信号](https://www.eefocus.com/baike/1546930.html)，用波形来表示比较直观。

在真正的电路中，是存在电路延迟的。在仿真时，如果加载的综合出来的电路模型，那么此时验证的内容主要是测试模型的逻辑功能是否正确，不考虑延时信息。这种仿真被称为功能仿真、RTL仿真、前仿真、前仿。

仿真的途径有两个，一个是quaruts 自带的仿真环境，一个是modelsim的仿真环境。

**Quartus 自带的仿真环境**

点击File，New，选择Uiversity Program VWF。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaMPOota2ZjWmqzp1s0cEiaKqcxr3ickPFR8o8ZiaE2XKLc7UHMIicE3h2Pw%2F640%3Fwx_fmt%3Dpng&s=881e48)

**图59 ：新建VWF文件**

点击OK。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeafgfgMB3bkqmM1qbCeSOfflt4sDJEj7yjzwBpVCHUkAaRcm5tVJ3nbA%2F640%3Fwx_fmt%3Dpng&s=f81081)

**图60 ：VWF初始化界面**

在“空白位置”双击。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaLZAX1lTzibVA5ZASMcbkaDwdtFz9QHFtJa0RVsVjbFia2zqib2b3Z6J5A%2F640%3Fwx_fmt%3Dpng&s=9a2e16)

**图61 ：插入节点或者[总线](https://www.eefocus.com/baike/1542179.html)**

点击“节点查找”。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaUUveDPv9eC888eS6Yw49UUSW46ehmJ3SR9tTzgLFIl7otmXRXUQiawg%2F640%3Fwx_fmt%3Dpng&s=bd506b)

**图62 ：节点查找**

点击“list”。在“filter”中，默认选择的是“Pins：all”，当点击“List”后，“Nodes Found”的界面中会出现所有的端口。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea7us17GOsQNiaOelokiaOMxfhdZzsYva0ZTX8tb15Wmmib4ZxwMIrBEyDg%2F640%3Fwx_fmt%3Dpng&s=bf4b8f)

**图63 ：查找需要激励和观测的信号**

选中所有的端口，点击“>>”，全部添加到“Selected Nodes”中。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea3PpR5Xicvpd9GT6FVvCBJWClDOMpdliaWibYHlwICqF25NSVllKic9Hotw%2F640%3Fwx_fmt%3Dpng&s=1c6bb2)

**图64 ：选择观测和[激励信号](https://www.eefocus.com/baike/1658524.html)**

点击“OK”。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaIpaxWs4YEFrJ4uiaeyG0tdxiagL68NgO34HadSeiaQnP1vu9Wb5Oxcggw%2F640%3Fwx_fmt%3Dpng&s=9533db)

**图65 ：确定选择的信号**

点击“OK”。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaTmDIjQ7c9OwJQVPpBSWrzLWcUX5xciblIJiaMgRicv1fy5r1CQicjqnHGA%2F640%3Fwx_fmt%3Dpng&s=311700)

**图66 ：选择信号完成**

A端口和B端口是二输入与门的两个输入，只要给A、B两个端口加载上合适的信号，观测Y端口的输出是否正确即可。

A端口和B端口一共只有四种情况“00”、“01”、“10”和“11”，只要将四种情况全部加载一遍即可。

利用鼠标左键，选中A端口信号一段或者B端口信号一段，利用上方的置1或者置0按钮，将输入信号做成下图所示：

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeafVR7WCsnR693wBRWPscOWaf6gaGkXh8HPiahyYz7jK8tnoicI8uKnAaQ%2F640%3Fwx_fmt%3Dpng&s=e64315)

**图67 ：设置输入信号**

设置好输入信号后，点击保存，将其保存到qprj文件中。虽然其为仿真文件，但是此文件依然只是quartus软件能识别，可移植性太差。

点击Simulation -> Run Functional Simulation（运行功能仿真）。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Weaukgiczw9tw0Rmj1O8g06swXDyftCUp5A90XDN6icTPjXnhCRI9R9IyOg%2F640%3Fwx_fmt%3Dpng&s=b7780a)

**图68 ：选择运行功能仿真**

经过一段时间的运行。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea5nKaSoYubJq6FyCHUNdRbYUYQ9v5FPuAmkiaMDpNdX6kOgBDJGqazTA%2F640%3Fwx_fmt%3Dpng&s=39c340)

**图69 ：运行功能仿真的过程**

在运行中，如果有报错，请认真回顾上述步骤。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea2EIpVzPq0gcICIJS6yNrk9j8zlXZfb1LXb7VPzb10wcS2KRq3iazdFw%2F640%3Fwx_fmt%3Dpng&s=15a7dc)

**图70 ：功能仿真结果图**

通过分析上述结果图，容易得出结论：AB“00” – >  Y“0”， AB“01” – >  Y“0”， AB“10” – >  Y“0”， AB“11” – >  Y“1”。通过波形的图的分析，二输入与门的功能仿真是没有任何问题的。

利用quartus 自带的[仿真器](https://www.eefocus.com/baike/1572820.html)，可以支持原理图输入和HDL代码输入；可移植性不强；对于一些复杂的输入信号，利用这种驱动方式较为复杂；在企业设计研发中，很少有人会用这个工具。

**利用modelsim仿真**

Mentor公司的ModelSim是业界最优秀的HDL语言[仿真软件](https://www.eefocus.com/baike/1573507.html)，它能提供友好的仿真环境，是业界唯一的单内核支持VHDL和Verilog混合仿真的仿真器。

利用quaruts 自带的仿真器仿真时，是利用绘制波形的方式进行输入信号的驱动。但是这种方法移植性不好，无法在modelsim中充当激励。

在开发中用的比较多的方式是利用HDL的方式进行充当激励，modelsim软件会自动抓取HDL代码中的信号进行绘制波形，用于设计者的观测。

Modelsim的软件无法为原理图的输入方式进行仿真，所以要将HDL代码输入的方式添加到工程，将原理图输入的方式移除工程。在企业工程开发时，不建议使用原理图的方式输入，移植性太低。所以在后续的章节中，将全部采用代码的输入方式进行设计。

新建一个verilog文件，命名为and_gate2_1_tb，保存到msim中。这个verilog文件是当做测试文件的，命名时，建议名字设置成为被测试模块的名字，然后后面加上“_tb”。tb为testbench的简写。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea0EkSYTT6kYg5HVkB7JHQiaKmBRYytAtIw3lCBibyqTFzZxnBXjYiaIwdw%2F640%3Fwx_fmt%3Dpng&s=68ae62)

**图71 ：and_gate2_1_tb代码**

`timescale是verilog中定义时间标度的关键字。1ns/1ps中的1ns表示时间的单位，在veirlog中不支持直接写出单位，例：5 ns，ns等时间单位是不被识别的。当定义了1ns为时间单位后，表示时间时，可以直接写出，例：表示10ns时，可以直接写10即可。1ns/1ps中的1ps表示时间的精度，由于精度的存在我们可以写小数。例：表示5.5ns时，可以直接写5.5。但是也正是由于精度的存在，小数不能无限制向下描述。例：表示5.523ns时，可以成5.523，如果表示5.258963ns，那目前的精度是到不了这么精确的。精度是1ps，因此小数的位数最多能有三位。在设计中，很少用到比ps还要精确的单位，所以一般的时间标度都是1ns/1ps。

Testbench文件也是verilog文件，所以也必须遵从verilog的标准。

在tb文件中，是没有端口的。在测试时，输入的信号都由内部产生，[输出信号](https://www.eefocus.com/baike/1572723.html)只要引出到内部即可，仿真器会自动捕获。所以tb的模块是没有端口的。

在测试文件中，需要将被测试元件例化进来。例化的方式如下：

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaNaML1EELyFBsROD0Rq5tAP8ccico2VaXia8qNz4WGUt8VuK6t2Z9pW4w%2F640%3Fwx_fmt%3Dpng&s=679ab5)

**图72 ：例化方式**

在例化时，首先是模块名称（and_gate2_1），后面是例化名称，这个名字可以任意名字，笔者建议例化名称要和模块名称有一定的关系，笔者采用模块名称后加上_inst，表示例化的意思。后面的括号是端口列表，每一个端口的前面加上一个“.”，后面加上一个“（）”，此时表示这个端口可以连接线了，连接线放到“（）”里面就是连接上了。

对于连接线来说，命名也是任意的，笔者建议连接线的名字和要连接的端口的名字要有一定的关系，笔者采用端口名字前面加tb_，表示tb中的连接线。

所有的连接线都需要提前定义。在定义时，都可以采用“wire”类型（后续会有更改）。

当例化完成，连接线定义和连接完成后，就需要开始测试了。而测试就是给模块的输入赋值，观测输出是否正确。

在测试时，我们需要顺序性的给出激励，verilog提供了一种比较简单的方式“initial”语句。在这个语句中，我们只需要顺序性的给出激励就可以了。“#”表示延时，延时在verilog中是不可综合的，但是在仿真中，是存在的。

“1’b0”中的1表示位宽为1，’b表示后面的数码为2进制，0表示数码为0。

在赋值时，建议采用位宽+进制+数码的方式进行赋值。

Verilog中，begin end表示中间的语句是一个整体，verilog和C类似，每一个函数或者语句下只能包含一个语句块。C语言中采用大括号，verilog中采用begin end。

上述的initial语句块中，描述了tb_A和tb_B被先赋值00，延迟20ns，被赋值为01，延迟20ns，被赋值为10，延迟20ns，被赋值为11。

Verilog语法规定，在initial语句中被赋值的变量，应该定义为reg类型。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeawtVoXWVVLiayzGt1EsTuMJnN6wcgIwje86abTxZGC0DkKib9VovqFs9g%2F640%3Fwx_fmt%3Dpng&s=b77f93)

**图73 ：定义变量（连接线）**

在写完testbench后，可以综合分析一下。保证没有任何的语法错误。

在仿真之前，需要指定仿真文件。

选择assignments -> settings。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeacvdVQUibDw2cSdtAZOr6ibsNolIvmU6wCWicppPCFlD6cfoyJIk8fIyEg%2F640%3Fwx_fmt%3Dpng&s=5d5ebc)

**图74 ：选择settings**

选择simulation。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaIvSTbtUic2BmFUncJibhevd0tm9pFTqxib5M56aHibsgWhS4DAdHqAwDMQ%2F640%3Fwx_fmt%3Dpng&s=2415d8)

**图75 ：选择simulation**

选择compile test bench 后，选择后面的test benches…。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeasdblWvmribcA2BsSZuhYeMzrTAwR1LwVfmnyNiaScUm0aWNvoAicyk8ZA%2F640%3Fwx_fmt%3Dpng&s=3a92c9)

**图76 ：选择test benches**

选择New，新建一个test bench。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea7V0Gojf8ySdcWBUOIrz15XJASxGEqodswSicCEpbichOE5VCLN9mycag%2F640%3Fwx_fmt%3Dpng&s=70bf44)

**图77 ：新建test bench**

在test bench name的对话框中输入：and_gate2_1_tb。默认模块中的顶层实体也是相同的。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaVS4k4ak0Kia6KZQ6rQHTJfJFuhfUCybnw0SJr4IhnPibLwkOwPyBibXKw%2F640%3Fwx_fmt%3Dpng&s=6c1d14)

**图78 ：输入testbench名字**

点击file name后面的三个点，寻找and_gate2_1_tb文件。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaWDtEJI4ribZHsKmkicZbKDo2mpbia5XjynP8Aibe3BFJT5jx8qibl49giaUw%2F640%3Fwx_fmt%3Dpng&s=73c910)

**图79 ：查找testbench文件**

找到msim文件夹中的and_gate2_1_tb，选中后，点击open。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea8icvOoPyDAH7TbGfGTqEbXWFOibuTR3bKYOe69ichFbvaSIEbmZh7NuOA%2F640%3Fwx_fmt%3Dpng&s=5d8951)

**图80 ：确定testbench文件**

点击ADD。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaXwWu2YAE7mMQcWFeJVDhtwp6XHrMtuCygJFh37uoxJuvMibOh6CsCsQ%2F640%3Fwx_fmt%3Dpng&s=38c363)

**图81 ：添加testbench**

添加成功后，点击ok。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaNMAamiaYmkgkf4o4shzO1Owdwr2eXgtLB9zrzpJQNcWk8bgU1IwkYVQ%2F640%3Fwx_fmt%3Dpng&s=a1d1a2)

**图82 ：添加成功**

点击ok。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaNlvXBGSGCbUexnAhucia4tSLu0RvMgFaBtDqhVD5wCwc2anXlibje2yg%2F640%3Fwx_fmt%3Dpng&s=750a01)

**图83 ：新建成功一个testbench**

点击OK。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea9cfd93998YWichspRN3J4GM4xe48Ga4YfLZ0aVOYpZqia8GlibSA6B2cg%2F640%3Fwx_fmt%3Dpng&s=7fc620)

**图84 ：设置完成**

在设置完成后，进行综合和分析。

综合和分析完成后，点击tools  -> Run Simulation -> RTL Simulation。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wearf6R3ylIFCxzALM0n3nXGWWrQLmcgEjofARTjxiak7RypQW7gx0joeQ%2F640%3Fwx_fmt%3Dpng&s=7a5ed8)

**图85 ：进行RTL 仿真**

稍微等一段时间，modelsim软件会自动启动。如果没有启动的话，并且报错的话，请查看modelsim的关联位置。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea6ZowBMKDmpJSRg1jMYAOSKtqlASIghFiccjw0XfzWWESHFaRD6IkX7w%2F640%3Fwx_fmt%3Dpng&s=13e335)

**图86 ：无法连接到modelsim软件**

点击tools 中的options。选择EDA Toos options。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea9F2fDUStGZqyu7MlenVEn1cf7jkz4ZvoTAezVVF2aQZ1bNS8CUDqqQ%2F640%3Fwx_fmt%3Dpng&s=5ebcfe)

**图87 ：配置关联路径**

在modelsim-Altera中，看看路径是否正确（这是笔者的安装路径，请自行查看自己的安装路径），注意最后那个“”，很多的系统中，没有它也是不对的。

配置完成后，点击ok。

重新点击RTL 仿真。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Weae2eKvVSgiaoicYJCWxy4cuSlOmWrY18CnUOGwVpZL7rw6zluJRmOMTxA%2F640%3Fwx_fmt%3Dpng&s=f57e7a)

**图88 ：modelsim的基本界面**

在wave界面中，点击一下鼠标左键，就会出现一个黄色的光标。

点击“全局缩放”。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeazHhmo2iaG5hzHX8GKA4TX6ln7lxCq42IcKnDWXWz5poxBXnp8qDoCIA%2F640%3Fwx_fmt%3Dpng&s=973e26)

**图89 ：放大缩小的图标**

在全局缩放前面一个为缩小，后面一个为放大。两头的两个图标，暂时不考虑。

全局缩放后，所有的波形都显示到wave窗口中，经过分析，设计正确。

此时就可以关闭modelsim软件。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeabvdrXxmjoziaiaWfCsJoQibezGh2gxiaL1V1qMbhRL8lWiceRqWKIymOrsQ%2F640%3Fwx_fmt%3Dpng&s=6ef555)

**图90 ：是否确定关闭modelsim**

点击“是”即可关闭。

## 6**************锁定管脚**************

输入设计后，经过综合和分析以及RTL仿真后，证明设计的逻辑功能是没有任何错误的。但是设计是在芯片内部进行实现的，如果想要发挥功能，势必要与外部的逻辑电路进行相连接。

在上述例子中，设计了二输入与门。我们可以将两个输入绑定到任意的两个管脚上，将输出绑定到任意一个管脚上。经过下载后，可以在输入的管脚上加载[电平](https://www.eefocus.com/baike/1465710.html)，测量输出管脚的电平，验证设计是否正确。

在FPGA学习开发板上，大部分都会有一些按键和LED，这些按键就可以为输入提供高低电平，LED就可以检测输出的电平值。

所以最好的验证方法是，将输入的管脚分配到连接有按键的管脚上，将输出分配到带有LED的管脚上。

自己制作或者购买的开发板，都会有原理图。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaM9IfE3UJepsHj3IuTAkibGQjtBK6oesvEiaD6BgbQibksTE1opicIjXiasQ%2F640%3Fwx_fmt%3Dpng&s=e2704d)

**图91 ：四位按键的电路原理图**

经过分析，key1的网络是直接连接到FPGA芯片上的；按键释放时，key1网络为[高电平](https://www.eefocus.com/tag/%E9%AB%98%E7%94%B5%E5%B9%B3/)，按下时，key1网络为低电平。其他按键原理相同。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Weao63icvvaAMaQu3icML5ZhjNjiaict2ZvesG2kleFRQYtYn82Emb5o4XVYw%2F640%3Fwx_fmt%3Dpng&s=ca6327)

**图92 ：LED电路**

经过分析，LED1的网络是直接连接到FPGA芯片上的；当FPGA将LED1网络置成高电平时，LED是熄灭的；当FPGA将LED1网络置成低电平时，LED是点亮的。其他的LED原理类似。

不同的电路原理，后续会得出不同的结果。请认真分析原理。

通过查看各个网络与FPGA的芯片连接关系，就可以得出按键、LED电路与FPGA的线连接的IO。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaGFcgM2OnNxibrZqq5iak8pCc4WMR1HWV4IZETZjkKuUdWOcwFhuN7myA%2F640%3Fwx_fmt%3Dpng&s=fbc83c)

**图93 ：外部网络与FPGA连接示意图**

经过查看，两个按键分别选择M15和M16。LED选择G15。

点击assignments  ->  pin planner。打开管脚规划器。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Weam7kzTLJ20uB8nciaJNfytw9YKxyezUj2HpC6zO3wSArkWwjQySsKzcA%2F640%3Fwx_fmt%3Dpng&s=e39c56)

**图94 ：打开管脚规划器**

在对应端口的Location标签下的空白窗位置双击。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeajkydxaPE6zMseADs7k2P5Rc3DfCY5Trib50UYCDTAEHup502PvkoTxw%2F640%3Fwx_fmt%3Dpng&s=85740c)

**图95 ：准备输入管脚号**

输入对应的管脚号。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaTyT2psy3mkG0HnzIiaibH5kmaSFzZrCwKpmjfLkicEYm6sEX4gWPicnBfw%2F640%3Fwx_fmt%3Dpng&s=7fa8ea)

**图96 ：输入对应的管脚号**

输入完成后，点击关闭即可。

点击综合和分析，等待综合分析完成。

## 7**************布局布线**************

综合分析只是将外部的输入转换成为对应的电路模型或者对应FPGA的电路模型，我们可以对电路模型进行RTL仿真，来排除逻辑功能的错误。

在FPGA片内实现的话，就需要对模型进行“实地”布置，利用FPGA片内的资源来实现模型，并且要固定好位置和连接线。这部分操作称为适配，也被称为布局布线。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaxhibrCVL7iaf1RBBd244FYERMq9NFytGwyFd4oyiaNPbEKv44QXM7GBow%2F640%3Fwx_fmt%3Dpng&s=aa2716)

**图97 ：布局布线选项**

双击Fitter即可进行布局布线。

布局布线后形成的报告有可能和之前综合分析形成的报告中的资源利用有所出入，最终我们以布局布线后的资源为准。

## 8**************时序仿真**************

在真正的电路中，是存在电路延迟的。在仿真时，如果加载的综合出来的电路模型，那么此时验证的内容主要是测试模型的逻辑功能是否正确，不考虑延时信息。这种仿真被称为功能仿真、RTL仿真、前仿真、前仿。

在仿真时，也可以加载布局布线后的电路模型，那么此时验证的内容主要是测试模型的延时是否能够达到我们的要求。这种仿真被称为时序仿真、后仿真、后仿、门级仿真。

双击EDA Netlist writer，产生后仿真所需要的模型。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeakmmURQKO04bk8vcjsYGb81OQ464kRC2XPdoDR7EaWAvpTVueC7d3JQ%2F640%3Fwx_fmt%3Dpng&s=1b6041)

**图98 ：EDA Netlist Writer选项**

启动后仿真。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea3kMyDntuxiaX61ghRyZcgs7yqnj1ZDEHETicTOG3HAc91qaT2XfxKqTA%2F640%3Fwx_fmt%3Dpng&s=c458db)

**图4-89 ：启动后仿真**

直接默认选择，点击RUN。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Weas34iaGWHeT3pNpWbV03QhA9DXCzBReusejzlppaficd7e22FpyEHt94A%2F640%3Fwx_fmt%3Dpng&s=9235a5)

**图99 ：选择时序模型**

综合器给出了外部不同条件下的时序模型，这里先不叙述他们各自的作用，后续时序分析中，会专门提到各个模型的作用。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaYDKibrGDdTDcPGHvDqllrJd4ctw4WAACK57lz6uRrpgJpKfF1n13Jaw%2F640%3Fwx_fmt%3Dpng&s=bdf0f2)

**图100 ：二输入与门后仿真波形图**

在输入信号都变为高后，输出信号没有立刻变化为高，而是延迟了一段时间后，才变为高电平。

在二输入与门中，电路的延迟对于我们来说是可以接受的，没有任何的要求。在后续的一些复杂的电路中，就要求电路的延迟不能太大，否则就会影响电路的功能。

时序仿真在企业设计中用的不太多，对于时序的很多问题，采用静态时序分析来查看。所以在后续的设计中，时序仿真将不在重点叙述。

## 9**************生成配置文件并下载**************

在布局布线后，需要将对应的结果下载到FPGA片内。对于模型来说是无法下载的，只有通过编译，形成某种文件才可以下载。

双击Assem[ble](https://www.eefocus.com/tag/ble/)r（Generate programming files），产生配置文件。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaLAGB8tgCuqopEwN30KeZrmQ2bQ3ic57gZl38QCH6qth5xibWSHmOE2YQ%2F640%3Fwx_fmt%3Dpng&s=78e9a1)

**图101 ：产生配置文件选项**

利用下载电缆连接FPGA开发板和PC。Intel FPGA的下载器为[usb](https://www.eefocus.com/tag/usb/) blaster ，当连接到PC后，需要安装驱动。

将FPGA开发板通电。

打开设备管理器。在通用串行[总线控制器](https://www.eefocus.com/baike/1571932.html)的下面，观看有没有Altera USB blaster。如果有的话，证明已经有了驱动，不需要再次安装。如果没有Altera USB blaster，在其他设备中，观测有没有“其他设备”。如果没有“其他设备”的话，就需要认真的检查开发板的各个连接线的连接情况。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaR1f35jkbZloTS5tfhIGptZRe1DKECfG1zfrObAN4Qd8cfP11vzGrjA%2F640%3Fwx_fmt%3Dpng&s=89ea03)

**图102 ：Altera USB blaster驱动**

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea9Sa91dVjnxP2qKuHbGC9aM2nhRw2OsZNiazdP2jjMRO8fwyP2dcq9Qg%2F640%3Fwx_fmt%3Dpng&s=9d5e9b)

**图103 ：其他设备**

选中USB-Blaster，右击，选择更新[驱动程序](https://www.eefocus.com/baike/1571085.html)。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaPC7KLakNFzlmlqnKeReR0nJf56jricJ5ofVe5bQe77tyYoPoqPkXPpw%2F640%3Fwx_fmt%3Dpng&s=434d2d)

**图104 ：如何搜索驱动程序软件**

选择“浏览计算机以查找驱动程序软件”。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeafyeMcicsUCIN1eHMB7fLcMl1uBz7k6ib33DVnf5ICm45jHe8L7a48Bkw%2F640%3Fwx_fmt%3Dpng&s=136d7d)

**图105 ：浏览计算机上的驱动程序文件**

点击“浏览”，按照图4-96中所示路径（quartus 的安装路径，读者请寻找自己的路径，后面的路径是相同的）选择文件夹，然后点击确定。

点击下一步，开始安装驱动。安装过程中，PC会询问是否安装，点击安装即可。

安装完成后，可以将下载线从PC上重新插拔一下。在通用串行总线控制器中就有Altera USB Blaster的驱动。

安装好驱动后，点击quartus 界面中tools的programmer。点击Hardware setup。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Weaermfib92TXYfxVfdYQ8ju5icLYuEsyJoyEZhudj7lhNQH6et4ZpchCag%2F640%3Fwx_fmt%3Dpng&s=cc8b47)

**图106 ：下载界面**

    选择USB Blaster后，点击close。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaiawWUZk9nkhVdXGZCr1FgKB5oxsnIgiaQyTD1LyNJF4icmukxicbvDvfmw%2F640%3Fwx_fmt%3Dpng&s=021857)

**图107 ：选择USB Blaster**

选择USB Blaster后，下载界面中显示出，USB Blaster。此时直接Start即可开始下载。右上角会有下载进度条。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaictF0CMM7ptUenibUujJej2L2QC0tibic13iaj8GGVUgicDsmhmLRHhm7HPA%2F640%3Fwx_fmt%3Dpng&s=11abf3)

**图108：下载界面**

如果下载界面没有可下载的文件，可以点击add files，在qprj中的output files文件夹中，有一个后缀为.sof文件。选择后，下载即可。

下载完成后，此界面就可以关闭。询问是否保存时，选择否即可。

当配置完成后，我们就可以进行验证。按下按键，分析LED的灯的状态。我们做的是二输入与门，它的真值表如下：

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wear9yMiadCTU0iaIzE9l5PWtqw1vyiaE0kIzwCcjGQKrIebpUuUXoslbV4Q%2F640%3Fwx_fmt%3Dpng&s=553126)

**图109 ：二输入与门真值表**

通过记录按键和LED的状态，也会得到一组真值表。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaVTu21CxEvVGSicoia9dYwlDYu9Xh5xa5lCEd8oficGoVaetJTVaTGMLRA%2F640%3Fwx_fmt%3Dpng&s=9dd06f)

**图110 ：二输入与门的按键和LED功能真值表**

通过分析按键和LED的工作原理，可以化简真值表。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Weas07E6bGH9zickkicQaU7OTKhK2eaLXKEwEugKoSFuWQKOosNjgrV6ZLA%2F640%3Fwx_fmt%3Dpng&s=5ca9d0)

**图111 ：二输入与门的按键和LED化简真值表**

通过分析两个真值表，确认功能正确。

不同的按键和LED原理，可以对应去分析。

大多数FPGA的内部实现是用SRAM等效出来的电路，SRAM是一种掉电丢失的器件。所以FPGA下载成功后可以正常运行，但是掉电后，FPGA会丢失之前配置的所有信息。

这种情况非常不利于产品的研发，设备断电时常有的事情，而断电后再上电，还是希望FPGA能够正常工作的。

正是由于FPGA掉电丢失所有信息，所以在FPGA的周边会配置一块掉电不丢失的存储器（flash），可以将配置信息存储到存储器中，FPGA每次上电后读取存储器的内容即可。

向flash中存储信息，需要将上述.sof文件转换为.jic文件。

在quartus 界面中，点击file  -> Convert Programming file。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Weakl61G86AUrUJQJewzjoFFJmhiaU8vic2G9nL1eGN66pwOTTZGq1zZ7Og%2F640%3Fwx_fmt%3Dpng&s=d55d60)

**图112 ：转换配置文件**

在programming file type中选择jic文件，在configuration device中选择EPCS16。EPCS16是altera 公司推出的一款flash的型号，国内大多数开发板上不是altera公司推出的芯片，但是能够兼容。EPCS16是一个16Mbit的flash。读者需要观察自己手中开发板的flash型号，找到与之对应的altera公司的兼容产品，选择即可。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1Wea9grDkDv02XtXGtsaGic44rKDMUXIgx3GRrYPic2lTZfBwxtQsaLhlIaA%2F640%3Fwx_fmt%3Dpng&s=9f1371)

**图113 ：添加flash loader**

选择flash loader，点击add device，选择ep4ce10，单击OK。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeauhbnV8wXUS9bocHaWvmFibiayn9AWOA3Bas2eaxdUFuOSsGe7WI1Psfg%2F640%3Fwx_fmt%3Dpng&s=77250e)

**图114 ：选择FPGA系列**

选择SOF data，点击add file，找到之前的配置文件.sof，点击open。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaoL7hfA3RRDqecNvBY57TnVThG6eWLMvZ17ic2iaA0XkKNTcCupCicnVEQ%2F640%3Fwx_fmt%3Dpng&s=ea6a55)

**图115 ：选择配置文件**

点击generate，开始生成.jic文件。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeafIhL5hLFFlHfv5Xgr2e2icLFvbjtYwI3G4AODb7ZTTSsjCKA2yrOjBA%2F640%3Fwx_fmt%3Dpng&s=65631e)

**图116 ：生成文件成功**

点击OK后，关闭界面即可。

重新打开下载界面。选择下载文件，点击delete。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaBeW7WKwAn5PibwawNkh8AGbHGzDbpqwuqQ4fujLalmV9SIB9jLnAYvw%2F640%3Fwx_fmt%3Dpng&s=c71d8b) **图117：移除默认下载文件**

点击add files，将生成的.jic文件（在qprj中的outputfiles文件中）添加进来，勾选program/configure，然后点击start。

![](https://wximg.eefocus.com/forward?url=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FaU04XPq8pdheMicpr3prrwuQgU51u1WeaIozibx3E2macmEXE6VUQG8csueziaapb2USpjGgPMyXnbPwbtbpZgydA%2F640%3Fwx_fmt%3Dpng&s=63013b)

**图118：下载.jic文件**

下载此文件速度比较慢，请耐心等待。

下载后，FPGA不能够正常工作，需要断电后上电，FPGA就可以正常工作了。

以后每次断电再上电，都可以正常工作。

本文的1到9小节就是正常的开发流程。在此过程之外，还有可能会加入一些其他的流程，例：功耗分析、时序约束等等。
