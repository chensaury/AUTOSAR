# 精华总结：一网打尽AUTOSAR MCAL模块

[ADAS与ECU之吾见](javascript:void(0);) *2024-01-08 08:20* *上海*

#### 一、简介

MCAL：微控制器抽象层；位于BSW层中的最下层；

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGPVy9iaQkEhXWbXctljwWRmDeyZbwIpQTnQiczy7XARhRspfzG2tzibTSQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

MCAL细分，可将驱动分为：微控制器驱动、存储器驱动、通信驱动、IO驱动：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGEG29ClvN4icGykjiag12nPibeibliaNqdq7eqtbMLMib0y6iahtTnwiaoWYe1Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 二、MCAL的配置（EB-Tresos）

**1.PORT**
我理解的PORT：MCAL层中的IO驱动组中的pin脚总体配置：

Port就是芯片上的每个pin脚，可以配置成DIO ADC PWM ICU等单引脚的功能，也能配置成CAN的TX或者RX、SPI的MOSI等等其他功能的单个pin脚功能；

总之，PORT就是芯片上的具体的某个引脚。

配置如下：

```
PortPinId:     逻辑上的Id值，从1递增
PinId：对应[芯片XX]芯片手册的pin引脚ID，根据实际使用选择对应的pin引脚
Mux: 选择PortPin用作哪个功能，最多八个，选择复用的功能需要查看TRM来选 择
InputSelect: 根据实际pin使用功能决定输入选择；比如Port用作IO Input 则选择SEL_NONE;比如用作CANFD1_Rx,则选择对应的CANFD1_Rx（参考 [芯片XX]_Procesor_TRM_Rev_00.06_For_xxx.pdf的IO Control/PINCTRL_SAFETY/Input Source Select）
PadSetting: 需要根据该Port用作的功能进行选择，如果是GPIO则选择PAD_SETTING_DEFAULT,如果是CAN则选择PAD_SETTING_CAN;有些pin比较特殊，建议沿用之前的配置。
OpenDrain: 是否启用开漏，选择是启用。
PortPinModeChangeable:是否启用在APP中更改PortPin的模式，一些特定场合会用到。
PortPinDirection: Port的方向，输入：PORT_PIN_IN, 输出：PORT_PIN_OUT
PortPinDirectionChangeable: 是否可以在程序运行过程中改变PortPin的方向（输入，输出）。
PortPinLevelValue:设置PortPin的初始化，只对Outout有效
PortPinInitialMode: 不需要配置
```



**2.DIO**

DIO一共分为五组，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqG15xcmRootNRmvGDvOpbauibjvM7NmardorOrTg6X7Ebo9CcW2BLQibDg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Dio没什么好配置的，只需要按照对应的ChannelId 更改下Name就好了。

**3.ADC**
[芯片XX]只有一个ADC内含8个通道，最大支持12位精度（8，10，12）；

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGJ1tqtLqeDicf9j7hsnEp1XxiaL65HQDwZzWYhHd1HHdYa3R7cQghq5Tw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

AdcPrescale: [公司]的[芯片XX]是填的199， BaseClock = 400MHz ,基于400MHz进行分频。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGFUQOSaMnYKMeV6bghNKIeNx4YbetYsu4dhBfehHXK2qYFD00mdwXicA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```
AdcLogicalChannelId: 逻辑通道从0递增
AdcPhysicalChannelId: 物理通道和逻辑通道保持一致，否则数据读取不正确
AdcChannelResolution: 选择ADC的采样精度8/10/12
AdcSampleFrequency(Hz): 通道的采样频率，ADC一共八个通道，代码中配置每个通道采样两次（MCAL暂时不能配置），内部FIFO的Water Level = 64, 按照配置中的800Hz来算   (1/800hz*16)*64 = 5ms
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGLH5SUn0PgvfrrmRhg2WJHS4Ykea9ZDHzN0XtBYicERxSiccJG0eW3bGg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)image-20240107211227399

```
AdcGroupConversionMode: 配置连续采样和单次采样，目前[芯片XX]只支持连续采样
AdcGroupTriggsrc: ADC_TRIGG_SRS_SW: 由软件API调用促发的组
ADC_TRIGG_SRC_HW: 由硬件触发的组
AdcNotification: [芯片XX]ADC采样必须使用中断模式，所以配置一个Notification进行数据处理。
```

**4.CAN**

**4.1 CAN-General**

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGBLInEzlz4kAdAD54lhpMXeAAgjYLIYIw7GA4ku6Ij7OG1NA02sjhcg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

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

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGINwWUIC9uR0de2my6zsI12Soibp8gvXQXY8vkYwwr6jmvBMI8ibw0BYA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```
CanControllerActivation: Channel 配置信息必须勾选此处才会生效
CanControlledId: 需和DaVinci中的ControlledId保持一致，不一致时，实际通信过程中CAN通道以DaVinci中的配置为准，会导致通道开启错误，进而无法通信的问题。
CanControllerBaseAddress: 要和CanControllerInstance保持一致，BaseAddress参考TRM手册。
例如：CAN1 0xF0030000      CAN 2 0xF0040000   CAN 3 0xF0050000   …
CanRxProcessing: INTERRUPT/POLLING
CanTxProcessing: INTERRUPT/POLLING
CanWakeupFunctionalityAPI: 没验证过该功能
CanWakeupProcessing: INTERRUPT/POLLING
CanWakeupSupport:没验证过该功能
CanIndividualRxMaskEnable: 勾选启用Rx filter mask功能
CanControllerDefaultBaudrate: 需要现在CanControllerBaudrateConfig配置波特率，然后才能选择
CanCpuClockRef: Clock时钟选择24M
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGzu29dibgoNpowHA1dgzxxS3UkxDYx1I7IWsibPdTXKue0t5kRUFkad7Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

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

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGHVBUEQgiaCic8O3uflDpZ6H8lNvnzRqMr8E5wjx1mzDkDogrA2YyyyBA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```
CanMessageBufferRegionName: 选择CAN_MB_REGION_0/CAN_MB_REGION_1,每个region有256byte
CanMessageBufferRegionSize: 选择CAN_MB_8_BYTES_PAYLOAD/CAN_MB_16_BYTES_PAYLOAD/CAN_MB_32_BYTES_PAYLOAD/CAN_MB_64_BYTES_PAYLOAD,每个region大小512byte,选择CAN_MB_8_BYTES_PAYLOAD一共可以接收512/（8+8）=32帧报文。如果配置成CAN_MB_32_BYTES_PAYLOAD一共可以接收512/（32+8）= 12
```

**4.2 CAN-CanHardwareObject**

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGGWbB69P5Z84Wegd4icd4hMakrVrBtDsKOabyoNrqib6t34VdiarSKWjRA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



```
在CanHardwareObject对CAN信号进行配置，该处配置需和DaVinci cfg的CanHardwareObject保持一致，否则协议栈处理会出现信号错位的问题。此处先讲解如何配置，然后再详细讲解如何和DaVinci cfg里的保持一致。
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGzavIaibPVFGQmcursbTIfyM8dkMAJm8nYsUSKzx51tZUy3PhqNyyuCg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



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

**5.SPI**

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqG5m757obDc2FWQ1arBX8ibrRiav3AnsEP9PjVfUCibRgwm2ayQBPujQSVg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGOst9nBL97zdtLVGlENSUibKxfyLQYoe08A7tv4IOgKpwPaicw6KM1d5w/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```
SpiMaxChannel: 与SpiChannel选项卡配置的Channel值保持一致
SpiMaxJob: 与SpiJob选项卡配置的Jobs值保持一致
SpiMaxSequence:与SpiSequence选项卡配置的Sequence值保持一致
SpiChannelBuffersAllowed: 0:1B ,  1:EB,   2 : IB&EB
SpiLevelDelivered: 0：1B ,  1: EB ,    2: IB&EB
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGfXZDNebRBcfTDdU8VET9ic7CgBaxGc9vzX9lzSicngic4nR65qicpd4gFA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```
SpiCsSelection: CS_VIA_PERIPHERAL_ENGINE/CS_VIA_GPIO选择SPI_SS或者GPIO作为CS, 选择CS_VIA_PERIPHERAL_ENGINE在SpiCsPin处选择Port的配置，选择CS_VIA_GPIO在SpiCsViaGpio处选择Dio的配置
SpiHwUnit: CSIB1-CSIB8对应SPI0-SPI7
```

**6.MCU**

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGDr0oibcOSEF3xy6jaAop11PRb5n00HkkqpT2aBZsLDXibylczHqYXAuw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGbziaMpIQXs5FaOM4f30olxSAolXu0zlia7AXXfEeFhxicFqyrPYMv50Ow/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGDxBiawmiawkZuFqzIhtqgmQicbwYKfZUFV6zR51WodXZPzibK8gMGMqc2g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqG7uIHW1SsqgPXdH177HEdel2K8BszDVHicJOafPtYKGicwtia67w9XUYbg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```
McuClockReferencePointFrequency: 期望的Clock频率和McuClockDefaultClock保持一致
McuClockDefaultClock:选项有MCU_CLOCK_UART_80M/MCU_CLOCK_TIMER_HIGH_FREQUENCY_400M/MCU_CLOCK_TIMER_LOW_FREQUENCY_24M/MCU_CLOCK_12C_133_3M/MCU_CLOCK_CANFD_80M/MCU_CLOCK_PWM_400M/MCU_CLOCK_PWM_EXT
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqG99x0ZHrLkSicONlHEnlTxyrrxwibpnBBZwh2TSVUwGGlrLsKpzcBD0vw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**我们使用了哪些外设模块就需要在此处Enable它，否则会导致该模块工作不正常或者初始化异常。**

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGP2MUxO6GYUjwvCFRvG5aT2OFJLnXtWSWNum0YukXGtyNqdemAsvUeA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果勾选了外设，则该外设只能由SECURE Doamin访问和使用，SAFETY Domain失去该模块的使用权限。

配置Mcu_InitRamSection的大小和写入值。（该截图里的值和[公司]的配置是一样的）。

**7.Gpt**
在[芯片XX] SOC 处理器中GPT模块配置的时钟是可以给其他模块使用的，例如在现有的项目开发中，Gpt有用作Os Timer, System timer ,和电源芯片定时喂狗中断等。

对于ICU模块来说只能使用GPT的配置作为时钟源。

[芯片XX]一共有8个Timer, 每个Timer有6个Channel,这6个Channel共享一个Timer时钟源和分频，换句话说，在APP中同一个Timer中最后生效的时钟源和分频是被最后一个初始化的Channel决定的。

6个Channel分别是：GPT_HW_TIMER_G0/GPT_HW_TIMER_G1/GPT_HW_LOCAL_A/GPT_HW_LOCAL_B/GPT_HW_LOCAL_C/GPT_HW_LOCAL_D, A/B/C/D共享一个中断号，G0/1共享一个中断号。支持使用同一个Timer的不能Channel，即使中断号共享[芯片XX]会自动识别到底是哪一个Chnnale触发的中断，进而去调用你所配置的Notification.

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGiaNqYjQBSs22iajrCQGXnwySA21wL9ibdQG0v1FwYI34IVsiaVJemktJ8Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqG3EbYReRSJYjjIVd2e9LQxicFsPmmMSmibIQmsukKiaUadr8y3WfvzhmqQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**Gpt基础配置，选择是否Enable某些功能和函数。**

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGEo0kH5yibVDuSJXia5XdlFGWiam2QIr94FvbxZ1z5AertSNFyIp8PpFiaA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```
GptHwModule: [芯片XX]一共有8个Timer,每个Timer有6个Channel,这6个Channel 共享一个Timer时钟源和分频，换句话说，在APP中同一个Timer中最后生效的时钟源和分频是被最后一个初始化的Channel决定的，更详细的介绍请参考[芯片XX]官方文档。
GptHwModuleChannel: GPT_HW_TIMER_G0…GPT_HW_LOCAL_D
GptChannelMode: Channel模式GPT_CH_MODE_CONTINUOUS/GPT_CH_MODE_ONESHOT
    Note：只有Local A/B/C/D可以配置成One shot模式
GptChannelTickFrequency：配置期望的频率，和GptChannelClkSrcRef保持一致
GptChannelTickValueMax：配置该GPT channel 最大的Ticks值产生中断或者其他
GptChannelClkSrcRef: 选择GPT 的时钟源
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGF4u3SVaWEeumItkPKPQlrmwNnm7dEosQqwE92EsP3zOIiapbCMYMc9g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

GptClockReference: 选择GPT可以选择配置的时钟源，只能选择已经在MCU模块配置好的时钟。

**8.ICU**
对于ICU模块来说只能使用GPT的配置作为时钟源

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGib0vpZZgqibbOtDI6wSau8dhbra5CjMy2Yvm117QA1py3Ikzp5bmZqfg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

ICU基础配置，选择是否Enable某些功能和函数.

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGW7sjMRtTH7t16apNic7YyCYHbxMopZ6PEkQiaIX0cDwdrFkejBfhIaIw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**9.PWM**
[芯片XX] 一共有8个PWM模块，每个pwm模块有四个子Channel，分别是A/B/C/D,四个子Channel共享同一个溢出值，所以子Channel的周期都一样的，占空比可以单独控制。更详细的可以参考官方文档。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGV6otpicyWrf6SLBR3NubEeULdkSc8DjIxwUehIU7cvJl2E97iakMIDoA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

PWM基础配置，选择是否Enable某些功能和函数

PwmIndex：暂时用不到

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqG8iariaOVVJYic5cVZyhwGJ1RKuzTBnc3RPv8vOcv0H6ZaCRVYwFbOIYjg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```
PwmHwModule: PWM_MODULE1/PWM_MODULE2/…/PWM_MODULE8
PwmPeriodDefault：设置PWM默认周期，我们通常在这里配置为0，如果配置成其他值且默认占空比也有配置，则初始化之后会立即输出PWM波
PwmMcuClockReferencePoint：Pwm的时钟源选择，只能选择在Mcu模块中已存在的配置，目前只能选择400MHz
PwmModuleFrequency：不可修改
PwmHwModulePrescaler: Pwm的分频系数

     400MHz/(PwmHwModulePrescaler+1) = 期望频率
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGAqp4r5xvGMYv0ictBqo4XhpC3kgMQzPbElhrT1ib1JN5qJDxjQFgTeLA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

PwmSubChannelId: 子ChannelID 0/1/2/3

DutycycleDefault: 默认占空比，通常配置为0x0

Polarity: Pwm的极性，根据项目需求配置

IdleState: Pwm空闲状态，通常与Polarity相反。



#### 三. 项目实践

**1.说明：**
项目实践中，MCAL需要配置两个新增功能，pwm和icu输入捕获。

功能描述：增加LSS8_EN（E12） / DI_AC_Wake(J4)PWM通道

（1）配置一个pin脚，让其输出pwm波形

（2）配置一个pin脚，让其捕获一个pwm波形

1. **查看PinMap表格：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGbyx3BdyUyITFts3DEvKUvChPIB933RewMvrYYjQM05VxYtebpXfekg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如图excel-PinMap表格描述了单片机中的两个引脚功能：

第一个：CPIO_C10引脚，配置成MIUX6的功能PWM3_CH2，Output模式的引脚，要输出信号，【功能描述】里的内容可以配置引脚名称时用。

第二个：GPIO_H3引脚，输入信号，使用的功能是MUX3，即TIM7_CH1，做输入捕获的功能。

**3.配置第一个功能：PWM输出**

**（1）配置PORT**

找到GPIO_C10 ，配置名称为DO_LSS8_Driver （截图示例为新建一个port）

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGEuiaJH393NlA3o9iamuNK8cQ0MiaPSJEPV1GRIOfHOpfqTekVmBpq8pnQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGtGa33WsfFXIaqokLh2JWEyuTBIYmib8VLLe7V9qmsu9qpVLU2F9nWZA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

根据【PinMap】文档中介绍的pin脚功能：配置。

**（2）配置DIO**

因为这个引脚十一输出的引脚 所以需要配置DIO （相当于GPIO 输出高电平或者低电平）

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGGFDTyDiaRMq2EHVfcUNaI2BBylxLqZ6f7VkibibdBA1cEV2aJkmEys29g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)image-20240107212653255

根据【PinMap】文档 ，查看MUX_0 = GPIO.IO58  ,配置IO58。

**（3）配置PWM**

引脚输出高电平的波形配置成PWM波形（有占空比 周期等参数的波形）

先配置模块，该芯片有8个PWM模块，每个模块有4个channel.

新增一个pwm模块（即第三个pwm模块） ，命名为PWMChannel_3 ,配置相关参数。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGJIPAwIic9Nlgib1ib01NEvjXbicLS26icm5VcLsHWNwqibjk9pMrgPRDozyA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqG4qGIicPT2hyoGMiamia2pFuuRt5JSC6RtmGYCFib5Z0290yXGbr8B0iabJw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

再配置子通道channel:

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqG152AnX54SeCtrdUSE8GYHR81qBS0fpum6cQhlFrSSTrgdiayIcLEdqw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上，完成【PinMap】文档中的PWM3CH2的配置。

**（4）配置MCU**

添加PWM3的使能

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqG3KmR1uTtNLuhIw2vsV0AcDRMObNs2OcAGjLz9icDlj6nmddIfIX9qUA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上，完成对引脚GPIO_C10的配置。

**4.配置第二个功能：ICU输入捕获**

**（1）配置PORT**

如【PinMap】文档，找到GPIO_H3 ，配置如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqG32ReeeZg4ibibAqmdeYHKaiagCTq1PXXyL8ZgBJicQu6cnSXSb9EAHMicNg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**（2）配置DIO**

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGKREg0cgiajID9EmelGIibJibh1kPbGtsyjwQd4jXy2toDzBrWAbN6pBUg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**（3）配置GPT**

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGJvQL7nbibQBN4p3A0FCEkrh76mfdOtacVaUVwCh5LuaQRiaZdwaoW8sw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

需要用到时钟驱动（【PinMap文档中的MUX功能】） MUC3 = TIM7_CH1

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGdAchUG2bYWaods9VUonhRp1nyHdOjxTxrYERfPdPJ5crzBUbyaA7Jg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

【+】新增 ，配置如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGtpia1n3WKKItYmQT4bvVEYy2MWrK1NRMlrKNMaiaLmybJVStffAJBnZw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**（4）配置ICU**

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGKCV3cD8kw1Z8Xbo3pDdK8iauYH6ayyiaw6oFODKYLQ8yAGtQhK7sdKyg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCLgj6NMF9RibxJ4MqCAVZqGrRXysrtk0pg7yAeIn19FN9gfsqJayVXqCKs2a02YkDPmeoeWZK0USA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

配置完成，生成代码即可。生成的代码是MCAL动态配置文件。

项目中，MCAL静态库和动态配置文件通常在不同路径下：

```
SDK包：     BSW\ShareUtiles\G9_SDK   :          
工程件：   BSW\ShareUtiles\MicroSarStatic_G9  : BSW层除MCAL外的其他模块代码 ： BsmW CanIF Dem等
DavinCi配置生成代码：Customer\Config\Source\MicroSarConfig ： bsw层除mcal外的其他模块的PBCfg.c和LCfg.c  (例如 Ea_Cfg.c  OS_xxx_Cfg.c等等)

MCAL静态库：  BSW\ShareUtiles\MCALStatic_G9    :adc.h adc.c  ...MCAL层的驱动文件
MCAL动态配置文件： Customer\Config\Source\McalConfig  : Adc_PBCfg.c  Port_Cfg.c  Pwm_Cfg.c等Mcal
```