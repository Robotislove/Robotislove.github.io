---
layout: post
title: Linux 设备驱动的软件架构总结学习笔记
date: 2022-09-29
author: lau
tags: [Linux, Blog]
comments: true
toc: false
pinned: false


---

Linux 设备驱动的软件架构总结学习笔记。

<!-- more -->

## Linux驱动的软件架构

基本思想：将驱动与设备分离，具体来说，驱动只管驱动，设备只管设备，总线负责匹配设备和驱动，而驱动以标准途径拿到板级信息。

设备指与项目有关的板级信息，项目用到的板级资源；驱动指对芯片、对硬件的操作。

Linux字符设备驱动需要编写file_operations成员函数，并负责处理阻塞、非阻塞、多路复用、SIGIO等。但我们面对一个真实硬件驱动时，并不像干所有事情，只需要实现我们关注的事情。这样就需要一个分层思想，因为所有输入设备都是一样的，可以提炼一个中间层，将软件进行分层：提炼一个input的核心层，把跟Linux接口以及整个一套input事件的汇报机制都在这里实现。

驱动与设备分离示意图：

![](https://img2022.cnblogs.com/blog/741401/202207/741401-20220711145242214-694984848.png)

Linux驱动分层：

![](https://img2022.cnblogs.com/blog/741401/202207/741401-20220711145244468-1913084011.png)

在单片机中，我们可能有多个 SPI控制器，如YYY1，YYY2。如果对每个SPI控制器都单独设一个驱动API，那么会有：

```c++
spi_client_yyy1_work1()
spi_client_yyy2_work2()
```

如果有数个spi控制器，那么就会多个驱动API。类似地，其他主机控制器可能也有多个，这样驱动API数量就会爆炸性增长，而它们的操作是类似的，有没有什么办法解决这个问题？
可以将Linux设备驱动的主机控制器与外设驱动分离，主机控制器不用关心外设，而外设也不用关心主机，外设只是访问核心层的通用API进行数据传输，主机和外设间可以进行任意组合。

Linux设备驱动的主机与外设驱动分离：

![](https://img2022.cnblogs.com/blog/741401/202207/741401-20220711145258812-1597625463.png)

## platform_device 总线设备

Linux2.6后，驱动模型只用关心3个实体：总线、设备、驱动。

- 总线
  总线将设备和驱动绑定。系统每注册一个设备时，会寻找与之匹配的驱动；相反，系统中每注册一个驱动时，会寻找与之匹配的设备。
  Linux并没有实体的总线，而是发明了虚拟总线，也称为platform总线。一个设备、驱动都挂接在一种总线上。
- 设备
  一个platform_device对象代表一个总线设备。platform_device并不是与字符设备、块设备、网络设备并列概念，而是Linux系统提供的一种附加手段，如SoC内部集成的I2C，RTC、LCD，Watchdog等控制器都归纳为platform_device，而这些本身就是字符设备。
- 驱动
  一个platform_driver对象代表一个总线驱动。

引入platform概念的好处：
1）使得设备被挂接在一个总线上，符合Linux2.6以后内核的设备模型。使配套的sysfs节点、设备电源管理都成为可能。

2）隔离BSP和驱动。BSP中调用platform设备和设备使用的资源、具体配置，而驱动中只需要通过通用API获取资源和数据。如此，板相关代码和驱动代码分离，驱动具有更好的可扩展性和跨平台性。

3）让一个驱动支持多个设备实例。

## 总线设备、驱动相关结构体

platform_device结构体：

```c++
struct platform_device {
    const char    *name; // 设备名称
    int         id; // 设备号
    bool        id_auto;
    struct device    dev;
    u32        num_resources;
    struct resource    *resource;

    const struct platform_device_id    *id_entry;
    char *driver_override; /* Driver name to force a match */

    /* MFD cell pointer */
    struct mfd_cell *mfd_cell;

    /* arch specific additions */
    struct pdev_archdata    archdata;
};
```

platform_driver 结构体：

```c++
struct platform_driver {
    int (*probe)(struct platform_device *);
    int (*remove)(struct platform_device *);
    void (*shutdown)(struct platform_device *);
    int (*suspend)(struct platform_device *, pm_message_t state); // 挂起 用于电源管理, 过时
    int (*resume)(struct platform_device *); // 恢复 用于电源管理, 过时
    struct device_driver driver;
    const struct platform_device_id *id_table;
    bool prevent_deferred_probe;
};
```

platform_driver 包含probe、remove、一个device_driver实例、电源管理函数suspend、resume。

电源管理可以用suspend, resume，但已经过时，更好的办法是实现device_driver中的dev_pm_ops结构体成员。

device_driver定义

```c++
struct device_driver {
    const char        *name;
    struct bus_type        *bus;

    struct module        *owner;
    const char        *mod_name;    /* used for built-in modules */

    bool suppress_bind_attrs;    /* disables bind/unbind via sysfs */
    enum probe_type probe_type;

    const struct of_device_id    *of_match_table;
    const struct acpi_device_id    *acpi_match_table;

    int (*probe) (struct device *dev);
    int (*remove) (struct device *dev);
    void (*shutdown) (struct device *dev);
    int (*suspend) (struct device *dev, pm_message_t state);
    int (*resume) (struct device *dev);
    const struct attribute_group **groups;

    const struct dev_pm_ops *pm;

    struct driver_private *p;
};
```

跟platform_driver 地位对等的i2c_driver、spi_driver、usb_driver、pci_driver中都包含了device_driver实例成员。描述了各种xxx_driver（xxx代表总线名）在驱动意义上的一些共性。

系统为platform总线定义了一个bus_type 实例platform_bus_type，位于drivers/base/platform.c。

```c++
struct bus_type platform_bus_type = {
    .name          = "platform",
    .dev_groups    = platform_dev_groups,
    .match         = platform_match,
    .uevent        = platform_uevent,
    .pm            = &platform_dev_pm_ops,
};
EXPORT_SYMBOL_GPL(platform_bus_type);
```

重点是match()，该函数确定了platform_device和platform_driver之间如何匹配。

```c++
static int platform_match(struct device *dev, struct device_driver *drv)
{
    struct platform_device *pdev = to_platform_device(dev);
    struct platform_driver *pdrv = to_platform_driver(drv);

    /* When driver_override is set, only bind to the matching driver */
    if (pdev->driver_override) // 匹配platform_device的driver_override和驱动名
        return !strcmp(pdev->driver_override, drv->name);

    /* Attempt an OF style match first */
    if (of_driver_match_device(dev, drv)) // 基于设备树风格的匹配
        return 1;

    /* Then try ACPI style match */
    if (acpi_driver_match_device(dev, drv)) // 基于ACPI风格匹配
        return 1;

    /* Then try to match against the id table */
    if (pdrv->id_table) // 匹配ID表
        return platform_match_id(pdrv->id_table, pdev) != NULL;

    /* fall-back to driver name match */
    return (strcmp(pdev->name, drv->name) == 0); // 匹配platform_device设备名和驱动名
}
```

platform_device（总线设备）和platform_driver（总线驱动）匹配有5种方式，按顺序排列：
1）匹配platform_device的driver_override和驱动名；（不推荐）
2）基于设备树风格的匹配；（推荐）
3）基于ACPI风格匹配；
4）匹配ID表，即platform_device设备名是否出现在platform_driver的ID表内；（推荐）
5）匹配platform_device设备名和驱动名；（推荐）

如何添加一个platform_device？
Linux 2.6 ARM，platform_device定义为一个数组，通过platform_add_devices()注册总线设备。它内部调用platform_device_register() 注册单个设备。

```c++
int platform_add_devices(struct platform_device **devs, int num);
```

Linux 3.x后，根据设备树内容自动展开platform_device，而不再推荐通过手动编码填写platform_device和注册。

## platform 设备资源和数据

### platform 设备资源

platform_device 定义中，由resource结构体描述资源。resource定义如下：

```c++
struct resource {
    resource_size_t start; // 资源开始值
    resource_size_t end;   // 资源结束值
    const char *name;
    unsigned long flags;   // 资源类型, 值可为 IORESOURCE_IO/IORESOURCE_MEM/IORESOURCE_IRQ/IORESOURCE_DMA
    unsigned long desc;
    struct resource *parent, *sibling, *child;
};
```

通常关注start、end、flags 这3个字段，分别表示资源开始值、资源结束值、资源类型。start、end含义随flags不同而不同，例如：

- flags为IORESOURCE_MEM，start、end表示该设备占据的内存起始地址、结束地址；
- flags为IORESOURCE_IRQ，start、end表示该设备使用的中断号开始值和结束值（左闭右闭）；

多份同类型资源，可停用多个start、end、flags。

如何获取设备资源？可通过platform_get_resource()，获得该dev中某种类型（type）的第num个资源。num从0开始计数。

```c++
#include <linux/platform_device.h>

struct resource *platform_get_resource(struct platform_device *dev, unsigned int type, unsigned int num);
```

对于IRQ类型资源，还有一个变体platform_get_irq()。

```c++
#include <linux/platform_device.h>

int platform_get_irq(struct platform_device *dev, unsigned int num);
```

例，在arch/arm/mach-at91/board-sam9261ek.c板文件中，为DM9000网卡定义了如下resource：

```c++
static struct resource dm9000_resource[] = {
    [0] = {
        .start = AT91_CHIPSELECT_2,
        .end   = AT91_CHIPSELECT_2 + 3,
        .flags = IORESOURCE_MEM
    },
    [1] = {
        .start = AT91_CHIPSELECT_2 + 0x44,
        .end   = AT91_CHIPSELECT_2 + 0xFF,
        .flags = IORESOURCE_MEM
    },
    [2] = {
        .flags = IORESOURCE_IRQ | IORESOURCE_IRQLOWEDGE | IORESOURCE_IRQ_HIGHEDGE,
    },
};
```

在DM9000网卡驱动中，通过下面办法获得这3份资源：

```c++
db->addr_res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
db->addr_res = platform_get_resource(pdev, IORESOURCE_MEM, 1);
db->addr_res = platform_get_resource(pdev, IORESOURCE_IRQ, 0);
```

### platform_data 设备数据

设备除了在BSP中定义资源（resource）外，还能附加一些数据信息，因为对设备的硬件描述除了中断、内存等标准资源外，可能还有一些配置信息，而这些配置信息依赖于板，不适宜放在设备驱动上。因此，platform提供platform_data支持。

platform_data 由每个驱动自定义。定义时，可以只存放一个结构体指针；解析时，根据结构体类型来解析。
例如，对于DM9000网卡，为platform_data定义一个dm9000_plat_data结构体，定义完后，可以将MAC地址、总线宽度、板上有无EEPROM信息等放入platform_data中。

```c++
static struct dm9000_plat_data dm9000_platdata = {
    .flags = DM9000_PLATF_16BITONLY | DM9000_PLATF_NO_EEPROM,
};

static struct platform_device dm9000_device = {
    .name = "dm9000",
    .id = 0,
    .num_resources = ARRAY_SIZE(dm9000_resource),
    .resource = dm9000_resource,
    .dev = {
        .platform_data = &dm9000_platdata, // 自定义设备数据
    }
};
```

如何获取设备数据？
可使用dev_get_platdata()。例如，在DM9000网卡驱动的probe()中，可这样拿到platform_data：

```c++
struct dm9000_plat_data *pdata = dev_get_platdata(&pdev->dev);
```

# 如何基于总线设备、驱动写程序？

以100ask使用2个GPIO驱动2个led灯的简单例子进行说明。主要分为两类工作：定义设备和驱动。

## 总线设备

总线设备指定项目用到的板级资源，因此放到与板配置相关目录board_A_led.c中，主要任务是定义并注册platform_device对象。

注意：现在通常不手动定义总线设备，而是通过设备树（dts）文件来定义设备使用的资源，因此实际项目中可能看不到这部分代码。

1）定义资源对象（resource）
定义2个GPIO口，为IORESOURCE_IRQ类资源。

```c++
/* 定义资源 */
static struct resource resources[] = {
    {
        .start = GROUP_PIN(3, 1), /* GPIO3_1 */
        .flags = IORESOURCE_IRQ,  /* flags 表示哪一类资源 */
    },
    {
        .start = GROUP_PIN(5, 8), /* GPIO5_8 */
        .flags = IORESOURCE_IRQ,
    },
};
```

2）定义设备对象（platform_device）

```c++
/* 把定义的资源添加到总线设备中
*
* 还需要定义一个与platform_device匹配的platform_dirver
*/
static struct platform_device board_A_led_dev = {
    .name = "100ask_led",                   /*总线设备名称  */
    .num_resources = ARRAY_SIZE(resources), /* 资源个数 */
    .resource = resources,
};
```

3）注册设备
通过board_A_led模块加载函数，注册总线设备（platform_device对象）。

```c++
static int led_dev_init(void)
{
    int err;
    /* register device */
    err = platform_device_register(&board_A_led_dev);
    return 0;
}
```

4）反注册设备
利用board_A_led模块卸载函数，反注册总线设备（platform_device对象）。

```c++
static void led_dev_exit(void)
{
    int err;
    /* unregister device */
    err = platform_device_unregister(&board_A_led_dev);
}
```

5）其他工作：指定入口函数、出口函数

```c++
module_init(led_dev_init);
module_exit(led_dev_exit);

MODULE_LICENSE("GPL");
```

其中，pdev为platform_device指针。

## 总线驱动

与板子无关，与具体的芯片和用到的外设相关，因此放到与芯片相关的chip_demo_gpio.c。主要任务是定义并注册platform_driver对象。

1）定义总线驱动结对象
需要与总线设备配对，即名称相同。

```c++
/*
* 总线驱动
* 需要与platform_device(总线设备)配对
*/
static struct platform_driver chip_demo_gpio_drv = {
    .driver = {
        .name = "100ask_led", /* 总线驱动名字, 要与总线设备名字配对 */
    },
    .probe    = chip_demo_gpio_led_probe,  /* 当发现配对的platform_device时, 就会调用probe函数 */
    .remove    = chip_demo_gpio_led_remove, /* 当把platform device去掉时, 就会掉调用remove函数 */
};
```

2）在probe函数中，获取资源，并创建逻辑设备（device_create）

```c++
static int chip_demo_gpio_led_probe(struct platform_device *dev)
{
    int i = 0;
    struct resource *res;
    
    while (1) { /* 因为用到了多个引脚资源, 所有用while循环获取 */
        res = platform_get_resource(dev, IORESOURCE_IRQ, i++);
        if (!res)
            break;
        /* save pin */
        g_ledpins[g_ledcnt] = res->start;

        /* device_create */
        led_device_create(g_ledcnt);
        g_ledcnt++
    }
    
    return 0;
}
```

led_device_create和led_device_destroy是上层leddrv.c中定义的，分别用于创建和销毁逻辑设备的函数。

为什么要在上层leddrv.c中定义？
因为调用device_create需要led_class（设备逻辑类 class_create创建），是static类型，只在leddrv.c中可见。

```c++
void led_device_create(int minor)
{
    device_create(led_class, NULL, MKDEV(major, minor), NULL, "100ask_led%d", minor);
}

void led_device_destroy(int minor)
{
    device_destroy(led_class, MKDEV(major, minor));
}
EXOPRT_SYMBOL(led_device_create);
EXOPRT_SYMBOL(led_device_destroy);
```

3）在remove函数中，销毁逻辑设备（device_destroy）

```c++
static int chip_demo_gpio_led_remove(struct platform_device *pdev)
{
    int i = 0;
    /* device_destroy */
    for (i = 0; i < g_ledcnt; i++) {
        led_device_destroy(i);
    }
    g_ledcnt = 0;

    return 0;
}

```

4）注册总线驱动
利用chip_demo_gpio模块的加载函数chip_demo_gpio_drv_init，来注册总线驱动。

```c++
static int chip_demo_gpio_drv_init(void)
{
    int err;
    err = platform_driver_register(&chip_demo_gpio_drv);   /* 注册总线驱动 */
    register_led_operation(&board_demo_led_opr);
    return 0;
}

```

5）反注册总线驱动
利用chip_demo_gpio模块的卸载函数chip_demo_gpio_drv_exit，来反注册总线驱动。

```c++
static void chip_demo_gpio_drv_exit(void)
{
    int err;
    err = platform_driver_unregister(&chip_demo_gpio_drv); /* 反注册总线驱动 */
}

```

6）登记chip_demo_gpio模块的入口函数、出口函数

```c++
module_init(chip_demo_gpio_drv_init);
module_exit(chip_demo_gpio_drv_exit);
MODULE_LICENSE("GPL");
```

完整源码示例参见：[04_led_drv_template_bus_dev_drv | gitee](https://gitee.com/fortunely/imx6study/tree/master/source/02_led_drv/04_led_drv_template_bus_dev_drv)

# 附：与总线设备、驱动有关的常用函数

## 注册/反注册platform_device

platform_device_register 注册总线设备；
platform_device_unregister 反注册总线设备；

```c++
#include <linux/platform_device.h>

int platform_device_register(struct platform_device *);
void platform_device_unregister(struct platform_device *);

```

## 注册/反注册platform_driver

platform_driver_register 注册总线驱动；
platform_driver_unregister 反注册总线驱动。

```c++
#include <linux/platform_device.h>

#define platform_driver_register(drv) \
    __platform_driver_register(drv, THIS_MODULE)

int __platform_driver_register(struct platform_driver *,
                    struct module *);
void platform_driver_unregister(struct platform_driver *);

```

## 注册多个device

platform_add_devices 一次注册多个device

```c++
int platform_add_devices(struct platform_device **, int);

```

## 获得设备资源

platform_get_resource 获得该dev中某类型（type）的第num个资源。num从0开始计数。

```c++
#include <linux/platform_device.h>

struct resource *platform_get_resource(struct platform_device *dev, unsigned int type, unsigned int num);

```

platform_get_irq 获得该dev中某类型（type）的第num个中断。num从0开始计数。

```c++
#include <linux/platform_device.h>

int platform_get_irq(struct platform_device *dev, unsigned int num);

```

platform_get_resource_byname 通过名字获得该dev的某类型（type）资源。

```c++
#include <linux/platform_device.h>

struct resource *platform_get_resource_byname(struct platform_device *dev,
                          unsigned int type,
                          const char *name)
```

通过名字（name）返回该dev的中断号。

```c++
#include <linux/platform_device.h>

int platform_get_irq_byname(struct platform_device *dev, const char *name);

```

# 参考

[1]宋宝华. Linux设备驱动开发详解[M]. 人民邮电出版社, 2010.
