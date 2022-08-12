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
### Qemu源码目录

- /vl.c: 最主要的模拟循环，虚拟机环境初始化，和 CPU 的执行。
- /target-arch/translate.c: 将 guest 代码翻译成不同架构的 TCG 操作码。
- /tcg/tcg.c: 主要的 TCG 代码。
- /tcg/arch/tcg-target.c: 将 TCG 代码转化生成主机代码。
- /cpu-exec.c: 主要寻找下一个二进制翻译代码块，如果没有找到就请求得到下一个代码块，并且操作生成的代码块。

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

下面分析stm32f405_soc_realize的实现.

### 初始化flash和sram
stm32f405_soc_realize主要实现了红框标注的三个内存区域

1、位于0x8000000处的flash区域

2、位于0x0处的区域，是flash的alias区域

3、位于0x20000000处的sram区域

函数入口首先设置了flash和sram.
```c++
static void stm32f405_soc_realize(DeviceState *dev_soc, Error **errp)
{
STM32F405State *s = STM32F405_SOC(dev_soc);
    MemoryRegion *system_memory = get_system_memory();
    DeviceState *dev, *armv7m;
    SysBusDevice *busdev;
    Error *err = NULL;
int i;

    memory_region_init_rom(&s->flash, OBJECT(dev_soc), "STM32F405.flash",
                           FLASH_SIZE, &err);
    if (err != NULL) {
        error_propagate(errp, err);
        return;
    }
    memory_region_init_alias(&s->flash_alias, OBJECT(dev_soc),
                             "STM32F405.flash.alias", &s->flash, 0,
                             FLASH_SIZE);

    memory_region_add_subregion(system_memory, FLASH_BASE_ADDRESS, &s->flash);
    memory_region_add_subregion(system_memory, 0, &s->flash_alias);

    memory_region_init_ram(&s->sram, NULL, "STM32F405.sram", SRAM_SIZE,
                           &err);
```

1、主要就是新建flash区域和flash_alias，然后通过memory_region_add_subregion把这两个区域放到对应的地址，这样0x0和0x8000000实际指向的是同一块RAM。

2、然后新建sram区域，并把sram放到0x20000000处。

根据需要模拟仿真固件的实际FLASH、SRAM参数可以配置不同的FLASH启动加载地址和大小、SRAM启动地址和大小。
```c++
#define FLASH_BASE_ADDRESS 0x08000000
#define FLASH_SIZE (1024 * 1024)
#define SRAM_BASE_ADDRESS 0x20000000
#define SRAM_SIZE (320 * 1024)
```

### 初始化外设
在初始化flash和sram后，会逐步初始化用到的外设，这里以UART外设为例进行介绍。
#### UART外设初始化
uart使用sysbus_mmio_map把外设的寄存器区域映射为mmio内存，然后使用sysbus_connect_irq初始化外设需要的irq。
```c++
/* Attach UART (uses USART registers) and USART controllers */
for (i = 0; i < STM_NUM_USARTS; i++) {

        dev = DEVICE(&(s->usart[i]));

        qdev_prop_set_chr(dev, "chardev", serial_hd(i));

        if (!sysbus_realize(SYS_BUS_DEVICE(&s->usart[i]), errp)) {
            return;
        }

        busdev = SYS_BUS_DEVICE(dev);

        sysbus_mmio_map(busdev, 0, usart_addr[i]);

        sysbus_connect_irq(busdev, 0, qdev_get_gpio_in(armv7m, usart_irq[i]));
}
```
s->usart在stm32f405_soc_initfn中创建

```c++
static void stm32f405_soc_initfn(Object *obj)
{
    STM32F405State *s = STM32F405_SOC(obj);
    int i;

    object_initialize_child(obj, "armv7m", &s->armv7m, TYPE_ARMV7M);

    object_initialize_child(obj, "syscfg", &s->syscfg, TYPE_STM32F4XX_SYSCFG);

for (i = 0; i < STM_NUM_USARTS; i++) {

        object_initialize_child(obj, "usart[*]", &s->usart[i],
                                TYPE_STM32F2XX_USART);
}
```
实际就是创建了TYPE_STM32F2XX_USART设备。
```c++
static const TypeInfo stm32f2xx_usart_info = {
    .name          = TYPE_STM32F2XX_USART,
    .parent        = TYPE_SYS_BUS_DEVICE,
    .instance_size = sizeof(STM32F2XXUsartState),
    .instance_init = stm32f2xx_usart_init,
    .class_init    = stm32f2xx_usart_class_init,
};
```

调用sysbus_init_child_obj函数初始化设备时会调用stm32f2xx_usart_init

```c++
static void stm32f2xx_usart_init(Object *obj)
{
    STM32F2XXUsartState *s = STM32F2XX_USART(obj);

    sysbus_init_irq(SYS_BUS_DEVICE(obj), &s->irq);

    memory_region_init_io(&s->mmio, obj, &stm32f2xx_usart_ops, s,
                          TYPE_STM32F2XX_USART, 0x400);
    sysbus_init_mmio(SYS_BUS_DEVICE(obj), &s->mmio);
}
```

函数做的工作如下

1、初始化设备的irq，保存到s->irq;

2、初始化s->mmio，设置memory_region的大小为0x400，mmio内存访问的回调函数由stm32f2xx_usart_ops指定;

3、sysbus_init_mmio主要是把s->mmio的指针保存到设备mmio数组中，以便后续使用sysbus_mmio_map把memory_region挂载到对应的地址。

#### mmio映射
stm32f405-soc实现了8个uart设备，设备mmio的起始地址分别为
```c++
static const uint32_t usart_addr[] = { 0x40011000, 0x40004400, 0x40004800,
                                  0x40004C00, 0x40005000, 0x40011400,
                                  0x40007800, 0x40007C00 };
```

然后在stm32f405_soc_realize函数里面会调用sysbus_mmio_map把设备的memory_region挂载到指定的位置。
```c++
sysbus_init_mmio(SYS_BUS_DEVICE(obj), &s->mmio);
```

### 中断仿真
#### qemu中断模型
qemu使用GPIO来实现中断系统，其简单的原理如下

Device.[GPIO_OUT] ->[GPIO_IN].GIC.[GPIO_OUT]->[GPIO_IN].core

1、首先CPU有GPIO_IN接口；

2、然后中断控制器（GIC）有GPIO_IN和GPIO_OUT， GPIO_OUT和CPU的GPIO_IN接口关联；

3、设备的GPIO_OUT和GIC的GPIO_IN关联；

4、当有中断发生时，设备通过GPIO_OUT通知GIC，GIC通过GPIO_OUT通知GPIO_IN。

中断依赖qemu_irq结构体：
```c++
struct IRQState {
    Object parent_obj;

    qemu_irq_handler handler; //irq handler
    void *opaque;
    int n; // irq num
};
```

要触发一个irq，可以使用qemu_set_irq函数

```c++
void qemu_set_irq(qemu_irq irq, int level)
{
    if (!irq)
        return;

    irq->handler(irq->opaque, irq->n, level);
}
```
GPIO_IN通过qdev_init_gpio_in初始化
```c++
void qdev_init_gpio_in(DeviceState *dev, qemu_irq_handler handler, int n)
```
初始化n个GPIO_IN接口，每个GPIO_IN接口的回调函数为handler，实际就是新建n个qemu_irq对象，qemu_irq的回调函数为handler。

GPIO_OUT初始化函数为sysbus_init_irq
```c++
/* Request an IRQ source.  The actual IRQ object may be populated later.  */
void sysbus_init_irq(SysBusDevice *dev, qemu_irq *p)
```
qemu使用sysbus_connect_irq将GPIO_OUT和GPIO_IN关联
```c++
void sysbus_connect_irq(SysBusDevice *dev, int n, qemu_irq irq)
```
把dev中的第n个gpio_out和irq关联，实际就是把irq保存为第n个gpio_out的值。

#### 实例分析
比如在armv7m_nvic_realize调用qdev_init_gpio_in初始化num_irq个GPIO_IN
```c++
static void armv7m_nvic_realize(DeviceState *dev, Error **errp)
{
    NVICState *s = NVIC(dev);

    /* The armv7m container object will have set our CPU pointer */
    if (!s->cpu || !arm_feature(&s->cpu->env, ARM_FEATURE_M)) {
        error_setg(errp, "The NVIC can only be used with a Cortex-M CPU");
        return;
    }

    if (s->num_irq > NVIC_MAX_IRQ) {
        error_setg(errp, "num-irq %d exceeds NVIC maximum", s->num_irq);
        return;
    }

qdev_init_gpio_in(dev, set_irq_level, s->num_irq);
```

uart设备在stm32f2xx_usart_init函数中通过sysbus_init_irq初始化一个GPIO_OUT

```c++
static void stm32f2xx_usart_init(Object *obj)
{
    STM32F2XXUsartState *s = STM32F2XX_USART(obj);

    sysbus_init_irq(SYS_BUS_DEVICE(obj), &s->irq);

    memory_region_init_io(&s->mmio, obj, &stm32f2xx_usart_ops, s,
                          TYPE_STM32F2XX_USART, 0x400);
    sysbus_init_mmio(SYS_BUS_DEVICE(obj), &s->mmio);
}

```

这样第0个GPIO就指向了s->irq。stm32f405_soc_realize会使用sysbus_connect_irq把设备的第0个GPIO和 nvic 的特定GPIO_IN进行关联。

实质上就是把 `s->irq = qdev_get_gpio_in(armv7m, uart_irq[i])`。

uart_irq 保存了每个uart设备需要使用的IRQ号
```c++
static const int usart_irq[] = { 37, 38, 39, 52, 53, 71, 82, 83 };
```

此外还有一个需要注意的点，这里的irq号和其在异常向量表中的位置存在以下关系

IRQ 号 = IRQ处理函数在异常向量表中的序号 - CPU内置异常数目

以stm32f405-soc为例，其使用的CPU为cortex-m4，CPU的内部中断数目为16个。

uart设备在stm32f2xx_usart_write中需要触发特定中断时会调用
```c++
 switch (addr) {
    case USART_SR:
        if (value <= 0x3FF) {
            /* I/O being synchronous, TXE is always set. In addition, it may
               only be set by hardware, so keep it set here. */
            s->usart_sr = value | USART_SR_TXE;
        } else {
            s->usart_sr &= value;
        }
        if (!(s->usart_sr & USART_SR_RXNE)) {
            qemu_set_irq(s->irq, 0);
        }
        return;
```

s->irq 在之前使用sysbus_connect_irq时就被设置成nvic中对应irq的qemu_irq结构

这里实际会调用set_irq_level通知nvic指定的中断到来

```c++
/* callback when external interrupt line is changed */
static void set_irq_level(void *opaque, int n, int level)
{
    NVICState *s = opaque;
    VecInfo *vec;

    n += NVIC_FIRST_IRQ;

    assert(n >= NVIC_FIRST_IRQ && n < s->num_irq);

    trace_nvic_set_irq_level(n, level);

    /* The pending status of an external interrupt is
     * latched on rising edge and exception handler return.
     *
     * Pulsing the IRQ will always run the handler
     * once, and the handler will re-run until the
     * level is low when the handler completes.
     */
    vec = &s->vectors[n];
    if (level != vec->level) {
        vec->level = level;
        if (level) {
            armv7m_nvic_set_pending(s, n, false);
        }
    }
}
```
主要就是根据IRQ号n，找到对应的异常信息 vec，然后判断vec的状态（高定平(level=1)，还是低电平(level=0)）。

如果是高电平，则会进入armv7m_nvic_set_pending通知CPU中断到来，实际也是调用CPU之前注册的GPIO_IN的回调函数通知。

因此qemu的中断实现其实是依赖于qemu_irq来实现，比如NVIC要通知CPU中断到来，实际就是调用CPU的qemu_irq中的回调函数实现。

### 固件加载

netduinoplus2_init在初始化stm32f405-soc后，调用armv7m_load_kernel加载二进制到内存
```c++
void armv7m_load_kernel(ARMCPU *cpu, const char *kernel_filename, int mem_size)
{
    int image_size;
    uint64_t entry;
    int big_endian;
    AddressSpace *as;
    int asidx;
    CPUState *cs = CPU(cpu);

#ifdef TARGET_WORDS_BIGENDIAN
    big_endian = 1;
#else
    big_endian = 0;
#endif

    if (arm_feature(&cpu->env, ARM_FEATURE_EL3)) {
        asidx = ARMASIdx_S;
    } else {
        asidx = ARMASIdx_NS;
    }
    as = cpu_get_address_space(cs, asidx);

    if (kernel_filename) {
        image_size = load_elf_as(kernel_filename, NULL, NULL, NULL,
                                 &entry, NULL, NULL,
                                 NULL, big_endian, EM_ARM, 1, 0, as);
        if (image_size < 0) {
            image_size = load_image_targphys_as(kernel_filename, 0,
                                                mem_size, as);
        }
        if (image_size < 0) {
            error_report("Could not load kernel '%s'", kernel_filename);
            exit(1);
        }
}

```
1、machine->kernel_filename通过命令的 -kernel 选项指定

2、armv7m_load_kernel首先尝试调用load_elf_as以elf格式加载

3、如果加载失败，就调用 load_image_targphys_as 直接把文件加载到0地址处

<table>
<thead>
<tr>
<th align="left">函数</th>
<th align="left">路径</th>
<th align="left">注释</th>
</tr>
</thead>
<tbody><tr>
<td align="left">main_loop</td>
<td align="left">/vl.c</td>
<td align="left">判断运行状态</td>
</tr>
<tr>
<td align="left">qemu_main_loop_start</td>
<td align="left">/cpus.c</td>
<td align="left">分时运行 CPU 核</td>
</tr>
<tr>
<td align="left">struct CPUState</td>
<td align="left">/target-xyz/cpu.h</td>
<td align="left">CPU 状态结构体</td>
</tr>
<tr>
<td align="left">cpu_exec</td>
<td align="left">/cpu_exec.c</td>
<td align="left">主要执行循环</td>
</tr>
<tr>
<td align="left">struct TranslationBlock</td>
<td align="left">/exec-all.h</td>
<td align="left">TB（二进制翻译代码块）结构体</td>
</tr>
<tr>
<td align="left">cpu_gen_code</td>
<td align="left">translate-all.c</td>
<td align="left">初始化真正的代码生成</td>
</tr>
<tr>
<td align="left">tcg_gen_code</td>
<td align="left">/tcg/tcg.c</td>
<td align="left">tcg 代码翻译成 host 代码</td>
</tr>

### 参考文档
[1] st公司发布stm32f4xx技术参考文档: https://www.st.com/resource/en/reference_manual/rm0090-stm32f405415-stm32f407417-stm32f427437-and-stm32f429439-advanced-armbased-32bit-mcus-stmicroelectronics.pdf

[2] 文章介绍了stm32f205 qemu实现: https://forum.butian.net/share/124

[3] 文章介绍了基于qemu实现监控基本块、指令级别的监控，支持观察点、断点的设置，支持mmio内存的申请等：https://forum.butian.net/share/123




