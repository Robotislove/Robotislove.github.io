---
layout: post
title: AutoSAR MCAL原理与实战
date: 2024-09-06
author: lau
tags: [AutoSAR,Archive]
comments: true
toc: false
pinned: false
---
AutoSAR MCAL原理与实战

<!-- more -->

一、简介MCAL：微控制器抽象层；位于BSW层中的最下层；MCAL细分，可将驱动分为：微控制器驱动、存储器驱动、通信驱动、IO驱动：二、MCAL的配置（EB-Tresos）1.PORT我理解的PORT：MCAL层中的IO驱动组中的pin脚总体配置：Port就是芯片上的每个pin脚，可以配置成DIO ADC PWM ICU等单引脚的功能，也能配置成CAN的TX或者RX、SPI的MOSI等等其他功能。


## 一、简介

MCAL：微控制器抽象层；位于BSW层中的最下层；

![](https://i-blog.csdnimg.cn/blog_migrate/fab3a0d9c51a761c36c889613a5cb55f.png)

MCAL细分，可将驱动分为：微控制器驱动、存储器驱动、通信驱动、IO驱动：

![](https://i-blog.csdnimg.cn/blog_migrate/614c2ccada2d33278635d29e7d89bb97.png)

## **二、MCAL的配置（EB-Tresos）**

### 1.PORT

****我理解的PORT：MCAL层中的IO驱动组中的pin脚总体配置：****

Port就是芯片上的每个pin脚，可以配置成DIO ADC PWM ICU等单引脚的功能，也能配置成CAN的TX或者RX、SPI的MOSI等等其他功能的单个pin脚功能；

总之，PORT就是芯片上的具体的某个引脚。

配置如下：

![](https://i-blog.csdnimg.cn/blog_migrate/fe052f91535fec09ffca0a374b8cb402.png)

```
PortPinId: 	逻辑上的Id值，从1递增
PinId：对应[芯片XX]芯片手册的pin引脚ID，根据实际使用选择对应的pin引脚
Mux: 选择PortPin用作哪个功能，最多八个，选择复用的功能需要查看TRM来选 择
InputSelect: 根据实际pin使用功能决定输入选择；比如Port用作IO Input 则选择SEL_NONE;比如用作CANFD1_Rx,则选择对应的CANFD1_Rx（参考 [芯片XX]_Procesor_TRM_Rev_00.06_For_xxx.pdf的IO Control/PINCTRL_SAFETY/Input Source Select）
PadSetting: 需要根据该Port用作的功能进行选择，如果是GPIO则选择PAD_SETTING_DEFAULT,如果是CAN则选择PAD_SETTING_CAN;有些pin比较特殊，建议沿用之前的配置。
OpenDrain: 是否启用开漏，选择是启用。
PortPinModeChangeable:是否启用在APP中更改PortPin的模式，一些特定场合会用到。
PortPinDirection: Port的方向，输入：PORT_PIN_IN, 输出： PORT_PIN_OUT
PortPinDirectionChangeable: 是否可以在程序运行过程中改变PortPin的方向（输入，输出）。
PortPinLevelValue:设置PortPin的初始化，只对Outout有效
PortPinInitialMode: 不需要配置
```

### 2.Dio

****D********io********一共分为五组，如下图所示：****

![](https://i-blog.csdnimg.cn/blog_migrate/89dd996be93d61d5d3064c207da2e1b8.png)

****D********io********没什么好配置的，只需要按照对应的Channe********lId ********更改下Name就好了。****

### ****3.ADC****

****[芯片XX]只有一个ADC********内含8个通道，最大支持12位精度（8，10，12）；****

![](https://i-blog.csdnimg.cn/blog_migrate/998f4f0171b4efa9fc6142c4d2c9e54e.png)

****A********dcPrescale: [公司]的[芯片XX]是填的199， B********ase**** ****Clock **** ****=**** ****400********MH********z**** **** ,**** ****基于40********0MH********z进行分频。****

![](https://i-blog.csdnimg.cn/blog_migrate/859af8c546f07650e285f2a9ca0bd556.png)

```
AdcLogicalChannelId: 逻辑通道从0递增
AdcPhysicalChannelId: 物理通道和逻辑通道保持一致，否则数据读取不正确
AdcChannelResolution: 选择ADC的采样精度8/10/12
AdcSampleFrequency(Hz): 通道的采样频率，ADC一共八个通道，代码中配置每个通道采样两次（MCAL暂时不能配置），内部FIFO的Water Level = 64, 按照配置中的800Hz来算   (1/800hz*16)*64 = 5ms
```

![](https://i-blog.csdnimg.cn/blog_migrate/52cbb60cd4ee31968e656438ec44cbce.png)

```
AdcGroupConversionMode: 配置连续采样和单次采样，目前[芯片XX]只支持连续采样
AdcGroupTriggsrc: ADC_TRIGG_SRS_SW: 由软件API调用促发的组
ADC_TRIGG_SRC_HW: 由硬件触发的组
AdcNotification: [芯片XX]ADC采样必须使用中断模式，所以配置一个Notification进行数据处理。
```

### 4.CAN

#### 4.1 CAN-General

![](https://i-blog.csdnimg.cn/blog_migrate/2fc2d30b65590d8cb3044c562c50f31c.png)

```
VirtualCanEnable: 指定CAN消息是否由（SDPE）半驱动器包引擎路由。如果启用，所有的CAN驱动程序将由SDPE处理
CanDevErroDetect:指定是否在每个API中启用错误检测
CanIndec: 对于[芯片XX]系列CAN驱动，该参数应该始终是0
CanLPduReceiveCalloutFunction:当收到帧时调用用户回调函数
CanMainFunctionBusoffPeriod:指定调用Can_MainFunction_BusOff的周期
CanMainFunctionWakeupPeriod:指定调用Can_MainFunction_Wakeup的周期
CanMainFunctionModePeriod:指定调用Can_MainFunction_Mode的周期
CanMultiplexedTransmission: 是否支持多路传输，多路传输用于防止传输帧时的优先级反转
CanTimeoutDuration:指定阻塞功能的超时时间，例如模块的enable/disable, freeze/unfreeze在控制器的初始化 ，注意：目前不支持此配置
CanVersionInfoApi: 指定是否支持Can_GetVersionInfo函数
CanSupportTTCANRef:[芯片XX]系列不支持TTCAN，因此不使用此配置。
```

---

```
CanControllerActivation: Channel 配置信息必须勾选此处才会生效
CanControlledId: 需和DaVinci中的ControlledId保持一致，不一致时，实际通信过程中CAN通道以DaVinci中的配置为准，会导致通道开启错误，进而无法通信的问题。
CanControllerBaseAddress: 要和CanControllerInstance保持一致，BaseAddress参考TRM手册。
例如：CAN1 0xF0030000  	CAN 2 0xF0040000   CAN 3 0xF0050000   …
CanRxProcessing: INTERRUPT/POLLING
CanTxProcessing: INTERRUPT/POLLING
CanWakeupFunctionalityAPI: 没验证过该功能
CanWakeupProcessing: INTERRUPT/POLLING
CanWakeupSupport:没验证过该功能
CanIndividualRxMaskEnable: 勾选启用Rx filter mask功能
CanControllerDefaultBaudrate: 需要现在CanControllerBaudrateConfig配置波特率，然后才能选择
CanCpuClockRef: Clock时钟选择24M
```

---

![](https://i-blog.csdnimg.cn/blog_migrate/8d6ecabaa6ab40c32868912231e7a36f.png)

```
在 CanControllerBaudrateConfig 选项卡中配置CAN的波特率和采样点等。
CanControllerBaudRate:直接填写期望的波特率，在驱动中会自动进行分频计算
CanControllerBaudRateConfigID:ID从0开始递增
CanControllerPropSeg:广播同步段
CanControllerSeg1:同步缓冲段1
CanControllerSeg2:同步缓冲段2
CanControllerSyncJumpWidth: 同步跳转段。
Note:采样点值的确定需根据客户的输入来确定，采样点计算方法：
	（1+CanControllerPropSeg+CanControllerSeg1）/(1+CanControllerPropSeg+CanControllerSeg1+CanControllerSeg2) * 100% = 采样点
在计算采样点参数时要注意这四个参数的关系，具体请参考百度或者J1939定义，否则EB不能生成代码。
```

---

![](https://i-blog.csdnimg.cn/blog_migrate/f40b3c20d206cebef3a544e67a8f622a.png)

```
CanMessageBufferRegionName: 选择CAN_MB_REGION_0/CAN_MB_REGION_1,每个region有256byte
CanMessageBufferRegionSize: 选择CAN_MB_8_BYTES_PAYLOAD/CAN_MB_16_BYTES_PAYLOAD/CAN_MB_32_BYTES_PAYLOAD/CAN_MB_64_BYTES_PAYLOAD,每个region大小512byte,选择CAN_MB_8_BYTES_PAYLOAD一共可以接收512/（8+8）=32帧报文。如果配置成CAN_MB_32_BYTES_PAYLOAD一共可以接收512/（32+8）= 12
```

#### 4.2 CAN-CanHardwareObject

![](https://i-blog.csdnimg.cn/blog_migrate/9aec5840b92699844ca779c64fa4a5ef.png)

```
在CanHardwareObject对CAN信号进行配置，该处配置需和DaVinci cfg的CanHardwareObject保持一致，否则协议栈处理会出现信号错位的问题。此处先讲解如何配置，然后再详细讲解如何和DaVinci cfg里的保持一致。
```

![](https://i-blog.csdnimg.cn/blog_migrate/9827b5b65fbea7e3ff852bf28254d1fc.png)

```
此处以一个Tx信号为例：
CanHandleType: BASIC/FULL
CanHwObjectCount: 配置成Tx并选择BASIC,配置决定该HTH可以使用几个MailBoxs,此处配置为32，第一个Region全部用作了发送
CanIdType: STANDARD/EXTENDED/MIXED
CanObjectId:需要和DaVinci CFG里面的保持一致
CanObjectType: TRANSMIT/RECEIVE
CanControllerRef: 该信号属于哪路Cantroller就选哪路
CanMessageBufferRegionRef: 选择使用哪一个BufferRegion,一定要注意每个Region最多配置32个8Byte的报文
```

![](https://i-blog.csdnimg.cn/blog_migrate/97b96e67fe7da1d82dd3a33b95f03055.png)

```
对于发送来讲是不需要配置Filter的，以该信号为例CAN ID = 0x7DF, 则需在Filter处配置CanHwFilterCode = 0x7DF, CanHwFilterMask = 0x7ff ,滤波就是Code&Mask = ID&Code, 所以在Driver层会自动计算写入寄存器。
如果是RxBasic 则需要计算出来Code&Mask配置好即可
```

### 5.SPI

![](https://i-blog.csdnimg.cn/blog_migrate/1714ff7e1a1fc27b5242c7dbe51307f9.png)

![](https://i-blog.csdnimg.cn/blog_migrate/8a6937062d3c32ecb2f97316ae34c432.png)

```
SpiMaxChannel: 与SpiChannel选项卡配置的Channel值保持一致
SpiMaxJob: 与SpiJob选项卡配置的Jobs值保持一致
SpiMaxSequence:与SpiSequence选项卡配置的Sequence值保持一致
SpiChannelBuffersAllowed: 0:1B ,  1:EB,   2 : IB&EB
SpiLevelDelivered: 0：1B ,  1: EB ,    2: IB&EB
```

![](https://i-blog.csdnimg.cn/blog_migrate/76b5173bb453ae0edb46f15727440aba.png)

```
SpiCsSelection: CS_VIA_PERIPHERAL_ENGINE/CS_VIA_GPIO选择SPI_SS或者GPIO作为CS, 选择CS_VIA_PERIPHERAL_ENGINE在SpiCsPin处选择Port的配置，选择CS_VIA_GPIO在SpiCsViaGpio处选择Dio的配置
SpiHwUnit: CSIB1-CSIB8对应SPI0-SPI7
```

### 6.MCU

![](https://i-blog.csdnimg.cn/blog_migrate/534b55d05b52f8be1a9d828a132310a0.png)

![](https://i-blog.csdnimg.cn/blog_migrate/a202c52fcac19118ed95a113bc326274.png)

![](https://i-blog.csdnimg.cn/blog_migrate/6dd2b76e9941ae5afbe3b3613900ce91.png)

 ![](https://i-blog.csdnimg.cn/blog_migrate/6370cef59af282d32ea2d7770b6a4235.png)

```
McuClockReferencePointFrequency: 期望的Clock频率和McuClockDefaultClock保持一致
McuClockDefaultClock:选项有MCU_CLOCK_UART_80M/MCU_CLOCK_TIMER_HIGH_FREQUENCY_400M/MCU_CLOCK_TIMER_LOW_FREQUENCY_24M/MCU_CLOCK_12C_133_3M/MCU_CLOCK_CANFD_80M/MCU_CLOCK_PWM_400M/MCU_CLOCK_PWM_EXT
```

![](https://i-blog.csdnimg.cn/blog_migrate/f8f94efa4c0244cc52087d7c3d020982.png)

 ****我们使用了哪些外设模块就需要在此处********E********nable它，否则会导致该模块工作不正常或者初始化异常。****

![](https://i-blog.csdnimg.cn/blog_migrate/9060d469fea1a1ba38f9c08185f79771.png)

 ****如果勾选了外设，则该外设只能由S********ECURE D********o********amin**** ****访问和使用，**** ****SAFETY D********o********main********失去该模块的使用权限。****

![](https://i-blog.csdnimg.cn/blog_migrate/0da3536fcff80b6912d89675a1e0e14f.png)

 ****配置Mcu**** ****_InitRamSection**** ****的大小和写入值。 （该截图里的值和[公司]**** ****的配置是一样的）。****

### ****7.Gpt****

 ****在**** ****[芯片XX]******** SOC ********处理器中G********PT********模块配置的时钟是可以给其他模块使用的，例如在现有的项目开发中，Gpt有用作Os******** T**** ****imer****  ****, System timer ,**** ****和电源芯片定时喂狗中断等。****

****对于I********CU********模块来说只能使用G********PT********的配置作为时钟源。****

 ****[芯片XX]****  ****一共有8个Timer**** ****, **** ****每个Timer有6个Channel****  ****,**** ****这6个Ch********annel********共享一个Timer时钟源和分频，换句话说，在********APP********中同一个Timer中最后生效的时钟源和分频是被最后一个初始化的********C********hannel决定的。****

****6个Channel分别是：G**** ****PT_HW_TIMER_G0****  ****/**** ****GPT_HW_TIMER_G1/GPT_HW_LOCAL_A/GPT_HW_LOCAL_B/GPT_HW_LOCAL_C/GPT_HW_LOCAL_D, A/B/C/D********共享一个中断号，G********0/1********共享一个中断号。支持使用同一个********T**** ****imer的不能Channel，即使中断号共享****  ****[芯片XX]**** ****会自动识别到底是哪一个Chnnale触发的中断，进而去调用你所配置的Notifi********cation.****

![](https://i-blog.csdnimg.cn/blog_migrate/b9277c95a9f98134b342d1f6f566cb0a.png)

![](https://i-blog.csdnimg.cn/blog_migrate/6541ca94cf20edd20ea1afdae4cc0b95.png)

****Gp********t********基础配置，选择是否Enable某些功能和函数****

 ![](https://i-blog.csdnimg.cn/blog_migrate/7f75f4b06cff53255fdcb4879d631377.png)

```
GptHwModule: [芯片XX]一共有8个Timer,每个Timer有6个Channel,这6个Channel 共享一个Timer时钟源和分频，换句话说，在APP中同一个Timer中最后生效的时钟源和分频是被最后一个初始化的Channel决定的，更详细的介绍请参考[芯片XX]官方文档。
GptHwModuleChannel: GPT_HW_TIMER_G0…GPT_HW_LOCAL_D
GptChannelMode: Channel模式GPT_CH_MODE_CONTINUOUS/GPT_CH_MODE_ONESHOT
    Note：只有Local A/B/C/D可以配置成One shot模式
GptChannelTickFrequency：配置期望的频率，和GptChannelClkSrcRef保持一致
GptChannelTickValueMax：配置该GPT channel 最大的Ticks值产生中断或者其他
GptChannelClkSrcRef: 选择GPT 的时钟源
```

![](https://i-blog.csdnimg.cn/blog_migrate/001528450574be1dfb386584de6e1d9f.png)

****Gpt********C********lock********Reference: ********选择G********PT********可以选择配置的时钟源，只能选择已经在M********CU********模块配置好的时钟。****

## 8.**ICU**

****对于I********CU********模块来说只能使用G********PT********的配置作为时钟源****

![](https://i-blog.csdnimg.cn/blog_migrate/bbc6a0b8918fbae151c58857eba191f0.png)

****I********CU********基础配置，选择是否En********able********某些功能和函数.****

![](https://i-blog.csdnimg.cn/blog_migrate/10b87b5e21caa75dd9abce25be6f5935.png)

****Icu********HardwareChannelRef: ********配置Icu的时钟源，需要先在Gpt模块配置好之后才能选择。****

### 9.**PWM**

****[芯片XX]**** ****一共有8个P********WM**** ****模块，每个pwm模块有四个子Channel，分别是A****  ****/B/C/D,**** ****四个子Channel共享同一个溢出值，所以子Channel的周期都一样的，占空比可以单独控制。更详细的可以参考官方文档。****

![](https://i-blog.csdnimg.cn/blog_migrate/c69b8dd990deba84d6bbe98267c70710.png)****P********WM********基础配置，选择是否Enable某些功能和函数****

****Pwm**** ****Index**** ****： 暂时用不到****

![](https://i-blog.csdnimg.cn/blog_migrate/0ae6a691fb09899fecf7073a3487286c.png)

```
PwmHwModule: PWM_MODULE1/PWM_MODULE2/…/PWM_MODULE8
PwmPeriodDefault：设置PWM默认周期，我们通常在这里配置为0，如果配置成其他值且默认占空比也有配置，则初始化之后会立即输出PWM波
PwmMcuClockReferencePoint：Pwm的时钟源选择，只能选择在Mcu模块中已存在的配置，目前只能选择400MHz
PwmModuleFrequency：不可修改
PwmHwModulePrescaler: Pwm的分频系数

 	400MHz/(PwmHwModulePrescaler+1) = 期望频率
```

![](https://i-blog.csdnimg.cn/blog_migrate/26f7a24919562945fa08aeb244f6e184.png)

 ****Pwm********SubChannelId: ********子Channel********ID ********0/1/2/3****

****Duty********cycleDefault: ********默认占空比，通常配置为0x********0****

****P********olarity: Pwm********的极性，根据项目需求配置****

****Idle********State: Pwm********空闲状态，通常与Po********larity********相反。****

## 三、项目实践

### 1.说明：

项目实践中，MCAL需要配置两个新增功能，pwm和icu输入捕获。

功能描述：增加LSS8_EN（E12） / DI_AC_Wake(J4)PWM通道

（1）配置一个pin脚，让其输出pwm波形

（2）配置一个pin脚，让其捕获一个pwm波形

### 2.查看PinMap的excel文档：

![](https://i-blog.csdnimg.cn/blog_migrate/0a1885edf79051c33e0fc6e594cc14a7.png)

 如图excel-PinMap表格描述了单片机中的两个引脚功能：

第一个：CPIO_C10引脚，配置成MIUX6的功能PWM3_CH2，Output模式的引脚，要输出信号，【功能描述】里的内容可以配置引脚名称时用。

第二个：GPIO_H3引脚，输入信号，使用的功能是MUX3，即TIM7_CH1，做输入捕获的功能。

### 3.配置第一个功能：PWM输出

#### （1）配置PORT

找到GPIO_C10 ，配置名称为DO_LSS8_Driver （截图示例为新建一个port） ![](https://i-blog.csdnimg.cn/blog_migrate/9b7140e0908b8138c07261310c25463a.png)

 ![](https://i-blog.csdnimg.cn/blog_migrate/7be56998b10b3c1578bc5a3344501f40.png)

 根据【PinMap】文档中介绍的pin脚功能：配置。

#### （2）配置DIO

 因为这个引脚十一输出的引脚 所以需要配置DIO （相当于GPIO 输出高电平或者低电平）

![](https://i-blog.csdnimg.cn/blog_migrate/2ca2dd81aadb1a814b6a89ff6bd70663.png)

 根据【PinMap】文档 ，查看MUX_0 = GPIO.IO58  ,配置IO58。

#### （3）配置PWM

引脚输出高电平的波形配置成PWM波形（有占空比 周期等参数的波形）

先配置模块，该芯片有8个PWM模块，每个模块有4个channel.

新增一个pwm模块（即第三个pwm模块） ，命名为PWMChannel_3 ,配置相关参数。

![](https://i-blog.csdnimg.cn/blog_migrate/5c862fa156950d9cac5e2ef07e251efe.png)

 ![](https://i-blog.csdnimg.cn/blog_migrate/459b812ef5eea6143fa3f8684d5e92ae.png)

 再配置子通道channel:

![](https://i-blog.csdnimg.cn/blog_migrate/ff93d7c144928ae08ba14de9a4898978.png)

 如上，完成【PinMap】文档中的PWM3CH2的配置。

#### （4）配置MCU

添加PWM3的使能

![](https://i-blog.csdnimg.cn/blog_migrate/a8ec2675a88a708c1fafd19d931ca9d0.png)

 如上，完成对引脚GPIO_C10的配置。

### 4.配置第二个功能：ICU输入捕获

#### （1）配置PORT

如【PinMap】文档，找到GPIO_H3 ，配置如下：

![](https://i-blog.csdnimg.cn/blog_migrate/c8eafbf28de0f3de807c1f6bb4cd5541.png)

#### （2）配置DIO

![](https://i-blog.csdnimg.cn/blog_migrate/fec4f2a01e5c57707a3e638bd082dd6c.png)

#### （3）配置GPT

![](https://i-blog.csdnimg.cn/blog_migrate/57470f6413b02714e847d8edd53e87db.png)

需要用到时钟驱动（【PinMap文档中的MUX功能】） MUC3 = TIM7_CH1

![](https://i-blog.csdnimg.cn/blog_migrate/5901b26a3c459d95f1787fc873408cb9.png)

 【+】新增 ，配置如下：

![](https://i-blog.csdnimg.cn/blog_migrate/d6370d7090881212bc8ee3e279e70a1e.png)

#### （4）配置ICU

![](https://i-blog.csdnimg.cn/blog_migrate/35c094b8085346c05c32e139b4124f95.png)

 ![](https://i-blog.csdnimg.cn/blog_migrate/4d9f9e050bdbb85e2c64bb506ccb5789.png)

配置完成，生成代码即可。生成的代码是MCAL动态配置文件。

 项目中，MCAL静态库和动态配置文件通常在不同路径下：

```
SDK包：     BSW\ShareUtiles\G9_SDK   :  
工程件：   BSW\ShareUtiles\MicroSarStatic_G9  : BSW层除MCAL外的其他模块代码 ： BsmW CanIF Dem等
DavinCi配置生成代码：Customer\Config\Source\MicroSarConfig ： bsw层除mcal外的其他模块的PBCfg.c和LCfg.c  (例如 Ea_Cfg.c  OS_xxx_Cfg.c等等)

MCAL静态库：  BSW\ShareUtiles\MCALStatic_G9    :adc.h adc.c  ...MCAL层的驱动文件
MCAL动态配置文件： Customer\Config\Source\McalConfig  : Adc_PBCfg.c  Port_Cfg.c  Pwm_Cfg.c等Mcal驱动的配置后生成的代码 ：EB-Tresos配置生成的代码
```
