---
layout: post
title: Qemu 固件模拟仿真技术-基于arm cortex-m4仿真实践笔记
date: 2022-08-12
author: lau
tags: [Markdown, Archive]
comments: true
toc: true
pinned: false
---

Qemu 固件模拟仿真技术-基于arm cortex-m4仿真实践。

<!-- more -->

## 为什么要做Qemu仿真？
为了加快开发速度，另外单板或者芯片的硬件资源是有限的，为了模拟多级或大量单板、芯片的性能，可以进行提前进行功能性仿真或者性能性仿真。

QEMU是目前最先进的动态二进制翻译跨平台仿真软件，它可以模拟x86、ARM、ARM64、MIPS、PowerPC等架构。QEMU的原理主要是将ELF格式的可执行文件翻译成中间形式，然后根据中间形式，拷贝编译好的微操作代码，形成目标基本块，最后再执行此基本块。它的总体结构如图所示
![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress-image-174126-1646122955.png)

QEMU主要有两种仿真方式：

- 用户模式仿真：允许一个（Linux）进程执行在不同架构的CPU上，该模式下，QEMU 可以作为进程级虚拟机

- 系统模式仿真：允许仿真完整的系统，包括处理器和配套的外设，该模式下，QEMU 也可以作为系统虚拟机


## Qemu安装
### Qemu下载
Qemu 官方下载地址：https://www.qemu.org/download/
选择source code源码下载，并选择qemu最新版本
```
wget https://download.qemu.org/qemu-6.1.0.tar.xz
tar xvJf qemu-6.1.0.tar.xz
cd qemu-6.1.0
./configure --prefix=/home/xx/xxx/qemu
make –j32
make install
```
./configure --prefix配置本地安装路径.

### Qemu仿真系统
首先我们需要从debian官网下载kernel和image，地址如下：
```
https://people.debian.org/~aurel32/qemu/mipsel/
```
「为什么我们这里知道使用mipsel呢，你可以在文件系统内随便找一个ELF文件，然后使用file命令查看一下」

将目录中的所有文件下载到一个kernel内即可，同时也将固件解压放到同一目录

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress-image-174126-1646122956.png)

首先安装虚拟网络设备tun
```
sudo apt-get install uml-utilities
```
为root用户添加网卡tap0
```
sudo tunctl -t tap0 -u root
```
设置IP地址
```
sudo ifconfig tap0 192.168.3.1/24
```
查看一下我们设置的IP地址
```
ifconfig
```
![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress-image-174126-16461229561.png)

进入kernel目录，并使用如下命令启动qemu：
```
sudo qemu-system-mipsel -M malta -kernel ./vmlinux-3.2.0-4-4kc-malta -hda ./debian_wheezy_mipsel_standard.qcow2 -append "root=/dev/sda1 console=tty0" -net nic -net tap,ifname=tap0,script=no,downscript=no -nographic -s
```

命令解析如下
![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress-image-174126-1646122958.png)

效果如图所示
![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress-image-174126-1646122960.png)


## stm32f405-soc实现

为了仿真某个设备，我们需要通过阅读硬件文档或者通过逆向程序逻辑来获取外设的行为，然后再在qemu中进行模拟，stm32f405的手册可以直接在网上下载.

```
https://www.st.com/resource/en/reference_manual/rm0090-stm32f405415-stm32f407417-stm32f427437-and-stm32f429439-advanced-armbased-32bit-mcus-stmicroelectronics.pdf
```

qemu提供的模拟设备中名字为netduinoplus2的Machine使用到了stm32f405-soc这个设备，可以使用 -M 指定使用该设备
```
qemu-system-arm -M netduinoplus2
```

netduinoplus2初始化函数为netduinoplus2_init

```c++
static void netduinoplus2_init(MachineState *machine)
{
    DeviceState *dev;

    /*
     * TODO: ideally we would model the SoC RCC and let it handle
     * system_clock_scale, including its ability to define different
     * possible SYSCLK sources.
     */
    system_clock_scale = NANOSECONDS_PER_SECOND / SYSCLK_FRQ;

dev = qdev_new(TYPE_STM32F405_SOC);

qdev_prop_set_string(dev, "cpu-type", ARM_CPU_TYPE_NAME("cortex-

	m4"));

    sysbus_realize_and_unref(SYS_BUS_DEVICE(dev), &error_fatal);

armv7m_load_kernel(ARM_CPU(first_cpu),

                       machine->kernel_filename,

                       FLASH_SIZE);
}

```

函数逻辑如下：

1、首先创建stm32f405-soc设备，然后设置cpu-type为 cortex-m4

2、然后通过设置 realized 触发stm32f405_soc_realize函数的调用

3、最后armv7m_load_kernel把命令行-kernel指定的文件加载到虚拟机内存。

```c++
static void stm32f405_soc_class_init(ObjectClass *klass, void *data)
{
    DeviceClass *dc = DEVICE_CLASS(klass);

dc->realize = stm32f405_soc_realize;

    device_class_set_props(dc, stm32f405_soc_properties);
    /* No vmstate or reset required: device has no internal state */
}

static const TypeInfo stm32f405_soc_info = {
.name          = TYPE_STM32F405_SOC,
.parent        = TYPE_SYS_BUS_DEVICE,
.instance_size = sizeof(STM32F405State),
    .instance_init = stm32f405_soc_initfn,
    .class_init    = stm32f405_soc_class_init,
};

```




