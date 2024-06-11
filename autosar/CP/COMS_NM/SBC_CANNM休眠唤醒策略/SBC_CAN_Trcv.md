# AutoSAR之基础篇CanNM

### 前言

首先，问大家几个问题，你清楚：

- 为什么要引入网络管理呢？上电同时启动，下电同时关闭，它不香吗？
- 你知道车上的ECU节点可以分为哪几种类型吗？
- 汽车启动时，ECU之间怎么保持同步唤醒的呢？
- 下电时，ECU又是怎样协同罢工的呢？
- 汽车熄火后，什么样的ECU会继续工作呢？

这篇，我们来一起探索并回答这些问题。为了便于大家理解，以下是本文的主题大纲：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCMiagFFPR0xTxrta0ibXwVbxgP5SAXaq53Ywpib6lo0GYGiawfbVD7dwMfVVqhSdhFkYSBnjAdicicyauA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

------

### 正文

#### 网络节点类型

汽车上ECU节点千千万万，不可能ignition On时所有ECU都正常工作，而是当用户需要请求相关功能时，参与该功能的相关ECU节点才需要启动起来，否则带来的只是过多对电池的无用消耗。

为了更好的去利用整车的能源，防止出现不必要的电池浪费，网络管理（Network Management，以下简称NM）便可以很好的解决此类问题，最大可能的高效利用整车电池能源，节约用车成本，延长电池使用寿命。

虽然汽车上网络总线类型多种多样，有CAN，FlexyRay、Lin、Ethernet等，但基本原理相似，本文将以最为常见的CAN总线的NM来讲述，举一反三，对于其他总线的NM，AUTOSAR也有相关规范，大家可以自行去阅读学习。

一般而言，按照唤醒方式，我们可以将ECU网络节点类型划分为两大类：**本地唤醒与远程唤醒**。

**本地唤醒：**唤醒源来源于自身模块，比如常说的KL15硬线唤醒或者hardware sensor感知唤醒等；

**远程唤醒：**唤醒源来源于自身ECU节点所在的网络报文，该节点可以处于完全休眠状态，一般无静电消耗；

根据这两大类唤醒方式，可以组合出下列四种子唤醒方式，每种方式的供电方式如下图所示：

##### 仅本地唤醒节点

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCMiagFFPR0xTxrta0ibXwVbxbkPPsd52afaJz73XMBavvLKn4A7IKIryMVZ0stBLrbQo7Pmv830IQQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





图1 本地唤醒节点

**
**

**特点：**ECU始终保持after-running状态，当检测到本地唤醒源之后，就会开启网络通信；

##### 仅网络唤醒节点

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCMiagFFPR0xTxrta0ibXwVbx230wMh1GOcfR9DicUujB92qPmxwtRROIE8pyD9pNMlnpntKmmldwr0g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图2 仅网络唤醒节点

**特点：** 只有当Transceiver监控到总线电平变化或者接收到特定报文时，就会接通Vreg给到芯片供电，从而唤醒ECU；

##### 本地+网络唤醒节点

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCMiagFFPR0xTxrta0ibXwVbxBI2ykZhCN8aY0WF6tkibR2Xr6A1sh25N6EXq5KNWJkibXbkwgqys9HLg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图3 本地+网络唤醒节点

**特点：** ECU始终保持After-Running模式，当接收到本地唤醒源或者网络唤醒源时，就会启动ECU；

##### 类KL15唤醒节点

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCMiagFFPR0xTxrta0ibXwVbxNZw1icJ64xjJ9CXURfZTJqfkRDHLtz8gpyhllqeFvXsdndjbNRofKGw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图4 KL15电唤醒节点

**特点：** 仅当Power supply 连接至ECU时，ECU才会唤醒，比如常说的KL15启动唤醒。

#### NM状态机

本文对于本地唤醒方式不作过多讨论，因为本地唤醒本身就是一种供应商自定义的一种唤醒方式，比较直接，不需要其他ECU节点参与，同时也不会对其他ECU节点造成影响。

但是网络唤醒是以网络管理报文为基础来协同整个网络“**同睡同醒**”，采用**分布式的直接网络管理方式**来发送自身节点所需的网络管理请求及自身网络管理状态，并接受来自网络上其他ECU节点的网络管理请求与状态。

因此，对于这些NM状态的变化我们有必要进一步研究NM状态机的状态及它们之间状态转换关系。该状态机的状态类型可分为“**三大三小**”。其中“三大”指的是Bus Sleep Mode、Network Mode、Prepare Bus-Sleep Mode；而“三小”则指得是Network Mode下的三个子状态：Repeat Message State、Normal Operation Mode、Ready Sleep Mode。

##### Bus Sleep Mode

当没有远程唤醒或者本地唤醒请求时，ECU的Controller应当切换至Sleep模式，电流消耗将降低至最低水平，该Mode是ECU启动时的起始状态或者是ECU睡眠时的最终状态。

在该模式下，NM报文以及应用报文都应该被禁止发送，但是可以被网络上的报文唤醒。

在此特意说明一点，当Transceiver支持并使能了特定帧唤醒时，该ECU只会接受到特定的NM报文才会正常唤醒，否则就会一直处于休眠状态，能够不受网络上其他报文的干扰。

如果Transiver不支持特定帧唤醒，那么网络的任意报文都可以唤醒该ECU，如果唤醒条件不满足，又会走休眠流程继续睡下去，这样“睡醒交替”的方式就是不支持特定帧唤醒的Transiver的典型特征。当然，如果整车上的NM都可以正常运作，那么就不会频繁出现这种“睡醒交替”的方式，这种方式一般都是在做测试时才会较多的体现出来。

##### Network Mode

一旦进入Network Mode，计时器**T_NM_Timeout**就会启动，只要成功接收到来自总线上的NM报文或者成功发送至总线NM报文，都会将该计时器**T_NM_Timeout**重置。

该模式又进一步细分为以下三种子状态，**RMS、NOS、RSS**。

- **Repeat Message State（RMS）**

- 该状态能够确保当ECU的状态机从Bus-Sleep Mode或者Prepare Bus-Sleep mode切换至Network Mode时能够及时的被网络上其他ECU节点发现，也就是告诉其他ECU，“大家注意了，我成功上线了，请多多指教！”

- 当成功进入到RMS状态时，该节点就会重新发送NM报文并开启计时器**T_REPEAT_MESSAGE**，应用报文则需要等待第一帧网络管理报文发送之后再发送。

- 当然，第一帧NM报文可以通过配置参数**MSG_CYCLE_ OFFSET**来延迟发送，降低在同一时间内的总线负载，这个配置参数默认是0 ，一般根据测试结果来做适当的调整。

- 在计时器**T_REPEAT_MESSAGE**超时之前，该节点就会一直保持在该状态，否则将会离开该状态。

- 在该状态下也存在着两个子状态：

- - **NM Immediate Transmit State**

    在该模式下，ECU的目的是快速唤醒整个网络，同时该节点将会以配置参数**T_NM_ImmediateCycleTime**的周期发送NM报文，而发送次数则是由配置参数**N_ImmediateNM_TIMES**来决定，每一次成功发送，该参数就会减1，直至为0，退出该子状态；

  - **NM Normal Transmit State**

    在该模式下，ECU节点将会以正常报文周期**T_NM_MessageCycle**的方式来发送NM报文。

- **Normal Operation State（NOS）**

  只要ECU节点自身存在网络通信的需要，那么ECU就会一直工作在NOS的状态下。该状态下NM报文的发送将会以**T_NM_MessageCycle**的周期来发送报文，每次报文的成功发送或接收或者计时器**NM-Timeout**超时都会重置该计时器**NM-Timeout**；

  在该状态下的NM报文以及应用报文都应该正常收发通信。

- **Ready Sleep State（RSS）**

  在该模式下，ECU节点应当停止发送NM报文。每次成功接受到来自网络上的NM报文，计时器**T_NM_TIMEROUT** 就会重置，一旦**T_NM_TIMEROUT** 超时，那么就会离开该状态转而进入**Prepare Bus-Sleep**状态。

##### Prepare Bus-Sleep Mode

一旦进入该模式，计时器**T_WAIT_BUS_SLEEP**就会启动。在该模式下禁止网络管理报文的发送，允许接受NM报文。应用报文已经在buffer中的一般允许继续发送，但最终应该是silent bus，该ECU的Controller的状态应当处于operational mode。一旦**T_WAIT_BUS_SLEEP**超时，就会进入到Bus-Sleep阶段。

##### Passive Mode

在该模式下只接受NM报文，但不发送任何的NM报文。该模式可以通过配置得到，同时该模式应只存在于开发或者调试过程中，在正式SOP的软件中禁止出现此种模式。

##### 报文发送与接受状态

在测试的过程中，需要针对网络管理每一个状态下的NM报文与APP报文接收与发送进行测试。如下图所示，体现了在不同NM子状态下的报文发送与接受状态。

- **Bus-Sleep**阶段，只接收NM报文唤醒，不发送任何报文；
- **Pre-Bus-Sleep**阶段，同样仅允许接收NM报文，对于早已在发送Buffer中的APP报文应发送完毕后立刻停止APP报文；
- 在**Network Mode**模式下，除了在Ready Sleep阶段不允许发送NM报文之外，其余阶段APP报文与NM报文正常收发；

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCMiagFFPR0xTxrta0ibXwVbx7icEgdrAqUWtnxsZvnticvLicsJAS68A7LP3WWLJQprYOH5rib0oe2tT9A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图5 NM过程中报文收发状态

##### 状态机时间参数总结

鉴于在网络管理各子状态的切换过程中都依赖于各种计时器，为了便于后续状态机切换的讲述以及后续查表方便，将相关参数总结如下，以供参考。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCMiagFFPR0xTxrta0ibXwVbx1TaOiclPpF18EIExkOfRaJhTrWXDUnc1rtOslXIopACk8nbQSvjUGqw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图6 NM计时器参数表

**NM状态机切换**

NM状态机是整个网络管理的核心。从上述内容可知NM管理状态机总共分为3种模式：Bus-Sleep、Pre-Bus-Sleep以及Network Mode。

其中Network Mode 又可分为3个子状态：Repeat Message State、Normal Operation State以及Ready Sleep State。

其中Repeat Message State又可分为Immediate Transmit State与Normal Transmit State。

如图7所示，根据以下状态切换路径来一一讲解各个状态的切换。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCMiagFFPR0xTxrta0ibXwVbxlYuyabX7s1jJCn3xwibQcz3dIVKc5feIITibCv9dCPnKOM9z9xT210ibw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图7 NM状态机切换

唤醒源一般可分为本地唤醒源与远程唤醒源。

- **本地唤醒源：**主要指的是基于内部计时器、传感器、按钮或者硬线连接或者基于内部状态的自身请求等。
- **远程唤醒源：** 主要指的是来自网络上的NM报文或者其他相关信号，比如点火开关信号等。

在ECU被唤醒前，软件内部状态机应当充分实现唤醒源的可靠性检查，如果检查失败，应当禁止进入Network Mode。

绝大部分情况下，需要支持NM的ECU都始终连接着KL30。在这种情况下，ECU是否被启动（存在静态电流消耗），则取决于网络上是否存在任意报文或者存在特定的NM报文。需要注意的是特定的NM报文唤醒方式则取决于是否采用支持特定帧唤醒的Transiver。

- **NM_01(系统初始化):**

  当系统KL30启动或者接收到来自网络的唤醒源时，则会执行系统初始化，在初始化的过程中，则会执行CanNM_Init来实现NM的初始化。如果唤醒条件不满足，ECU就会一直停留在Bus-Sleep 阶段，直至满足条件休眠或者被正常唤醒。

- **NM_02(Bus-Sleep to RMS):**

  当ECU处于Bus-Sleep阶段时，如果接收到有效的NM报文，则会进入到Normal Transmit State。当进入到该阶段后，在**T_REPEAT_MESSAGE** 超时前，ECU将按照**T_NM_MessageCycle**周期来传输报文，同时**T_MESSAGE_TIMEOUT**也会启动。

- **NM_03(Bus-Sleep to RMS):**

  当ECU在Bus-Sleep阶段，存在本地唤醒请求时，ECU应当主动激活网络，并进入Immediate Transmit State阶段，同时将发送的NM报文中的Active Wake up bit置为1。

  在该状态下，应当按照**N_ImmediateNM_TIMES**的次数发送报文周期为**T_NM_ImmediateCycleTime**的网络管理报文。

- **NM_04:**

  当**N_ImmediateNM_TIMES** 等于0之后，NM状态就会从Immediate Transmit State进入到Normal Transmit State。

- **NM_05(RMS to RMS):**

  在RMS阶段，如果**T_NM_TIMEROUT**超时，当前NM状态不会被改变，但是**T_NM_TIMEROUT**会被重置。当**T_MESSAGE_TIMEOUT**超时后，则会调用相应的exception函数通知上层进行处理。

- **NM_06(RMS to NOS):**

  当处于RMS阶段时，**T_REPEAT_MESSAGE**超时后，ECU需要继续保持网络通信的需要，即通过调用**CanNM_NetworkRequest**函数进入到NOS阶段，而ECU则会继续按照**T_NM_MessageCycle**来发送NM报文。

- **NM_07(NOS to RMS):**

  在NOS阶段，有两个RMS子状态可以到达：Immediate  Transmit  State 与 Normal  Transmit  State。如果自身节点有repeat message的需要，那么则会进入到Immediate  Transmit  State。如果接收到的NM中repeat message bit置1，则进入到Normal  Transmit  State。

- **NM_08(NOS to NOS):**

  在NOS阶段，如果**T_NM_TIMEROUT**超时，当前NM状态不会被改变，但是**T_NM_TIMEROUT**会被重置。当**T_MESSAGE_TIMEOUT**超时后，则会调用相应的exception函数通知上层进行处理。

- **NM_09(NOS to RSS):**

  当休眠条件满足时，ECU就会通过**CanNm_NetworkRelease**函数来实现从NOS至RSS状态。在RSS状态下应当停发NM报文。

- **NM_10(RSS to NOS):**

  在RSS状态下，如果存在本地唤醒请求，则可以通过**CanNm_NetworkRequest**函数来切换至NOS状态。

- **NM_11(RSS to RMS):**

  在RSS状态下，RMS存在两种子状态可供进入：Immediate Transmit State与 Normal  Transmit  State。当自身节点存在repeat message请求时，则会进入前者；当接受到外部的NM报文中repeat message bit为1时，则进入后者。

- **NM_12(RMS to RSS):**

  当ECU处在RMS状态中的Normal  Transmit  State状态下，如果**T_REPEAT_MESSAGE**超时且满足休眠条件时，则会进入RSS状态。

- **NM_13(RSS to Pre-Bus-Sleep):**

  在RSS状态下，如果没有本地唤醒请求或者远程唤醒请求，在计时器T_NM_TIMEROUT 超时之后就会进入Pre-Bus-Sleep 阶段，同时**T_MESSAGE_TIMEOUT**置为0，启动T_WaitBusSleep计时器。

- **NM_14(Network Mode to Network Mode):**

  在Network Mode下，当成功接受或者发送NM报文时**T_NM_TIMEROUT** 就会被重置，重置该定时器的行为就发生在调用函数**CanNM_RxIndication**或者**CanNM_Txconfirmation**接口中。

- **NM_15(Pre-Bus-Sleep to RMS):**

  在Pre-Bus-Sleep模式下，如果存在远程唤醒请求，则会进入到RMS阶段中的Normal Transimit State。同时启动**T_REPEAT_MESSAGE**。

- **NM_16(Pre-Bus-Sleep to RMS):**

  在Pre-Bus-Sleep模式下，如果存在本地唤醒请求，即调用函数接口**CanNm_NetworkRequest**来进入到RMS中的Immediate Transmit阶段，应当按照**N_ImmediateNM_TIMES**的次数发送报文周期为**T_NM_ImmediateCycleTime**的网络管理报文。

- **NM_17(Pre-Bus-Sleep to Bus-Sleep):**

  在Pre-Bus-Sleep模式下，如果不存在本地唤醒或者远程唤醒请求，则进入到Bus-Sleep状态。至于何时关闭电源控制，则可以根据自身节点类型来shutdown。



#### 网络管理报文结构

NM报文作为传递自身节点的NM状态的重要载体，有必要仔细研究报文中每一位Bit的含义。

接下来就以CAN NM报文为例，来一同解析其报文总体结构以及各个Bit的相应含义，以便了解各个ECU节点的协同唤醒与休眠方式。

##### NM报文总体结构解析

如下图8所示，根据最新的AutoSAR官方文档的介绍：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCMiagFFPR0xTxrta0ibXwVbxzcDaEB3b0Vdfwaldhck0t7pZHHt7BMoGo9BhtBNaVsytDZGdhDBm8A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图8 NM报文结构

由图可知，Byte 0表示当前节点的Source ID，比如如何当前节点发送的NM报文ID为0x514，那么该Source ID就为0x14；Byte1则为CBV，该字节中传递着当前节点的网络状态，是非常重要的字节；其余字节作为User Data给到客户自行定义，如定义当前唤醒源有哪些？网络管理状态机切换过程等。

当然上述报文内容定义只是AutoSAR举例说明，Source ID可以被分配至Byte0，Byte1或者没有，而CBV也是可被分配至Byte0，Byte1或者没有。这些字节内容的分配与定义一般均可以在BSW工具链中进行配置。

##### CBV详解

鉴于CBV的重要性，如下图9所示，讲解每一个Bit在使用中的具体含义：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCMiagFFPR0xTxrta0ibXwVbxUROtCKlGp3wgauBtGSPt8fiaPW172lKx9IzvcfZTn0dK8WTUmIETuHw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图9 CBV Byte定义

**Bit 0: Repeat Message Request Bit**

- 0: 代表存在Repeat Message Request ；
- 1：代表不存在Repeat Message Request ；

**Bit 1：PN ShutDown Request Bit（PNSR）**

- 0：NM报文中不包含同步局部网络管理休眠请求；
- 1：NM报文中包含同步局部网络管理休眠请求；

**Bit 3：NM Coordinator Sleep Bit**

- 0：未被主协调NM节点请求开始同步休眠；
- 1：已被主协调NM节点请求开始同步休眠；

**Bit 4：Active Wakeup Bit**

- 0：节点没有唤醒过网络，属于被动唤醒；
- 1：节点唤醒过网络，属于主动唤醒；

**Bit 5: PN Learning Bit(PNL)**

- 0: PNC learning被请求
- 1: PNC learining未被请求

**Bit 6 PN Information Bit(PNI)**

- 0: NM报文中包含PN 信息；
- 1:NM报文中未包含PN 信息；

上述几个Bit中，经常使用到的也就Bit0，Bit3，Bit4, Bit6这4位，需要重点掌握。

#### 常用函数接口

为了更好的使用该模块函数以及遇到问题时方便调试该模块，特将CanNM模块中较为重要的常用函数列举如下图10所示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icCMiagFFPR0xTxrta0ibXwVbxbuOI9Ty3SicAibWvxxN3SLsxpyQ97bA4IpSTl4ZNUc1RQPw7xP2gicnPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



对于NM这个模块，还有相关知识比如PNC，因为篇幅有限，不宜深入展开，后续会专题讲解，敬请期待。



感谢您阅读至此，愿给你带来些许帮助。码字不易，绘图不易，您的每次**【点赞】、【在看】、【转发】、【关注】**将是我继续深夜码字的强大动力！

 

经常有小伙伴们在后台给我留言咨询相关技术问题，在这里为了方便大家的技术交流，扫一扫，一起学习，一起进步！





# 一文搞懂ECU休眠唤醒之利器-TJA1145

### 前言

首先，小T请教大家几个小小问题，你清楚：

- 什么是TJA1145吗？
- 你知道休眠唤醒控制基本逻辑是怎么样的吗？
- TJA1145又是如何控制ECU进行休眠唤醒的呢？
- 使用TJA1145时有哪些注意事项呢？

今天，我们来一起探索并回答这些问题。为了便于大家理解，以下是本文的主题大纲：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAOIOIUzVCPGPofXf2UBS1moghrnVklUbOVCNsfVbP9zhPeIFIgia04Bciar9cD1LQwNTbGhvNHsRKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

------

### 正文

#### TJA1145简介

TJA1145是NXP公司为汽车电子领域量身定做的高速CAN收发器，提供了CAN控制器与物理CAN双绞线之间的接口，相比其他CAN收发器，它具备如下几个特点：

- **在Standby与Sleep状态下能保持极低功耗，其中Sleep状态下功耗比Standy状态下更低；**
- **可通过选择性唤醒功能支持符合ISO11898-2:2016标准的CAN部分网络；**
- **针对TJA1145T/FD与TJA1145TK/FD这两种TJA1145/FD变种而言，支持CANFD-Passive功能，能够实现在CANFD与CAN网络共存的前提下保证节点正常休眠不会被总线CANFD总选错误唤醒；**
- **TJA1145提供接口与3.3V或者5V微控制器相连，通过SPI可用来控制CAN收发器控制以及状态获取；**
- **TJA1145物理层实现满足了ISO11898-2:2016与SAE J2284标准，且能够实现CANFD 2M通讯速率的稳定通信；**

TJA1145上述这些特性使得其能够作为控制ECU整个供电系统的关键利器，通过休眠特性最大可能地降低ECU整体电流消耗，仅在应用层存在需求时才会被唤醒。

如下图1所示为TJA1145的基本功能框图，各个引脚的基本功能解释如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAOIOIUzVCPGPofXf2UBS1mNZUZUfuF7ode03iaKvUMb1F7qaJuuiaEIpfvv3ibTBlLhMOzibvqmVVOCQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 TJA1145 基本功能框图

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAOIOIUzVCPGPofXf2UBS1mD5DAQxRzGvByvLeyemX4UibWPvUC0oN5tj29uiak1JyN6gU1CV6T2k9g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图2 TJA1145功能引脚定义

如下图3所示为TJA1145内部三大供电细节详解：

- **BAT：** 该电源用于给TJA1145系统状态维护进行供电，只要BAT一直有点，那么TJA1145相关状态寄存器值就不会丢失,且**CAN接收器由BAT供电；**
- **VCC：** 该电源一方面作为5V电源输入给到TJA1145系统模块进行监控是否过压或欠压，另一方面则给到CAN总线供电，且**CAN发送器由VCC供电；**
- **VIO:**   该电源一方面作为电源输入到TJA1145系统模块进行监控是否过压或欠压，另一方面则作为SPI通信的电平转换；

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAOIOIUzVCPGPofXf2UBS1m9pViabVnyNC1ydUV8pQalWELkDCQn0QraQfwfjFmSb63vC0r0UoUrEg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图3 TJA1145内部供电详解图



#### ECU休眠唤醒概念

为了让大家更好的了解休眠唤醒控制原理，小T将按照AUTOSAR文档中针对休眠唤醒的定义给大家做个总结，可分为如下三种场景：

**冷启动式休眠唤醒**

冷启动式休眠唤醒具备如下几个特点：

- **MCU处于掉电状态；**
- **ECU的部分外围电路如CAN发器处于供电状态(如KL30供电)；**
- **唤醒事件能够被CAN收发器识别；**
- **CAN收发器能够根据唤醒源决定是否唤醒MCU，给到MCU供电；**

**CAN通道式休眠唤醒**

CAN通道式休眠唤醒具备如下几个特点：

- **MCU始终处于正常供电状态；**
- **至少ECU的部分外围电路如CAN收发器处于供电状态；**
- **CAN收发器处于Standby状态；**
- **唤醒事件能够被CAN收发器识别；**
- **CAN收发器识别到有效唤醒源后能够产生一个软中断唤醒MCU或者MCU周期性的去检查是否存在有效唤醒源；**

**CAN通道与MCU式休眠唤醒**

CAN通道与MCU式休眠唤醒具备如下几个特点：

- **MCU处于低功耗状态；**
- **至少ECU的部分外围电路如CAN收发器处于供电状态；**
- **CAN收发器处于Standby状态；**
- **唤醒事件能够被CAN收发器识别；**
- **CAN收发器识别到有效唤醒源后能够产生一个软中断唤醒MCU；**

#### 休眠唤醒控制原理

通过上述对休眠唤醒类型的总结，想必大家都可以集合自己工作中碰到的休眠场景一一对应起来，小T就结合最为常见的**冷启动式休眠唤醒**从硬件与软件两个层面跟大家一起交流休眠唤醒控制基本原理。

**硬件层面：**

如下图4所示，小T将从MCU芯片供电以及TJA1145状态获取控制两方面来讲解硬件层面的休眠唤醒控制原理：

**S1：**MCU满足休眠条件时，通过发送SPI相应指令**让TJA1145进入Sleep状态**；

**S2**：TJA1145**进入到Sleep状态后，INH引脚就会拉低，控制5V或者3V关闭电源输出，间接导致MCU整个系统处理掉电状态**，此时TJA1145始终处于供电状态（由于BAT始终有电），整个ECU成功进入到休眠状态；

**S3**：TJA1145虽然处于Sleep状态，属于极低功耗状态，**同步也检测着网络是否存在有效唤醒源**；

**S4**：当TJA1145发现有效唤醒源之后，**就会自动从Sleep状态切换成Standby状态**，在Standby状态下INH引脚拉高，此时5V与3V便会正常输出，从而MCU被正常供电，程序开启正常运行；

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAOIOIUzVCPGPofXf2UBS1mTJp7lRXgFaK76sKVsicaBibp2nwrfBnZ288FA6VevaYe8cn4CUHEjdJg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图4 TJA1145休眠唤醒机制硬件实现

**软件层面：**

基于AUTOSAR软件架构，休眠唤醒的检测处理机制统一有EcuM模块来实现，有关EcuM的详细介绍可参考文章《[AUTOSAR基础篇之EcuM](http://mp.weixin.qq.com/s?__biz=MzU3OTI4NzY0OQ==&mid=2247487807&idx=1&sn=8f6ade36c059be313841bb38d7b41782&chksm=fd693e31ca1eb727beb4151889a4e2b932c5285c514890f7897137d63107d628459d29641ddd&scene=21#wechat_redirect)》。关于通讯模块的开启则是由ComM模块来负责，BswM在此阶段可实现一些自定义的控制行为来满足各个项目的特别要求。

本文不会过多阐述细节，仅在说明整个休眠唤醒过程在软件逻辑层面具体如何来实现，参考如下图5所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAOIOIUzVCPGPofXf2UBS1mSjOz3ADvUARQ9bpGHpzf5NuF3ZuCqf5CbzyKIQLdiaXdU17qeybNcRw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图5 休眠唤醒软件控制逻辑

**S1**：当MCU Power ON之后由EcuM模块便会检查唤醒源是否有效，若有效，则通知ComM模块开启通信，**进而通过SPI通信控制TJA1145进入Normal状态**，同时通知BswM模块开启其他BSW模块的控制以及让EcuM进入到RUN模式；

**S2**：当MCU PowerON之后由**EcuM模块识别唤醒源无效，便会直接走下电流程，最终控制TJA1145进入到Sleep状态；**

**S3**：在系统正常工作后如果满足休眠条件（如外界没有NM报文），MCU便会控制TJA1145进入到Sleep状态；

**S4**：当TJA1145处于Sleep状态下会检测休眠前设定的唤醒源，**如果唤醒源满足条件，TJA1145切换至Standby状态，从而Power ON MCU，最终走向EcuM唤醒源检测验证流程；**



#### TJA1145控制

如上述过程MCU通过SPI总线接口实现了针对TJA1145状态的控制与获取。针对TJA1145的控制过程可分为TJA1145 Operating Mode的控制以及内部CAN Operating Mode两种类型。

**TJA1145 Operating Mode**

在TJA1145内部存在一系统控制器包含如下五种运行状态机：**Normal，Standby，Sleep，Overtemp，Off**状态。接下来将针对这五种状态进行一一讲解每个状态的基本特征以及相应的SPI控制指令。

- **Normal:** 该模式下TJA1145处于全功能模式下，所有功能均可用，可通过**MC=111从Standby或者Sleep状态切换至Normal状态；**

- **Standby:** 该模式下TJA1145处于低功耗状态，不过不能够正常收发数据，但是INH引脚可以始终保持拉高状态；

  **如果此时寄存器CWE=1，那么此时receiver会持续监控总线电平状态，如果寄存器CPNC=PNCOK=1，那么就开启了特定帧唤醒，否则就是标准CAN唤醒（010101切换就可唤醒，即所说的任意帧唤醒）；**

- 当电压满足特定下限时便会自动从Off状态切换至Standby状态；

- 当监测到温度没有再次发生过温便会自动从Overtemp状态切换至Standby状态；

- 在Sleep状态下检测下唤醒源便会自动切换至Standby状态；

- 通过SPI指令MC=100将状态从Normal或者Sleep状态切换至Standby状态；

- 在Normal状态下发送指令MC=001准备切换至Sleep状态，但此时存在唤醒源或者所有的唤醒源检测全部关闭，该特性从一定程度上避免了死锁；

- **Sleep:** 该模式下TJA1145处于最低功耗状态下，且不能正常收发数据，同时INH引脚处于高阻状态，电源供电一般会通过该引脚进行拉低关闭输出。同时其状态变化存在如下几种可能：

- 一个有效唤醒源或者中断事件(**除去SPIF事件**)或者**SPI指令(SPI通讯速率不能过高)**则可以唤醒TJA1145从Sleep状态切换至Standby状态；

- 在Normal或者Standby状态下通过发送MC=001便可以进入到Sleep状态，同步须确保至少存在一个唤醒源使能(如CAN唤醒或者Wake pin唤醒)且没有pending状态下的唤醒源；

- **如果VCC或者VIO持续一段事件低于某个阈值，那么该低电压事件将会强制让TJA1145进入到Sleep状态，与此同时所有的Peding的唤醒源将会被清除，CWE=1以及WPFE=WPRE=1使能同时特定帧唤醒功能将会被Disable(CPNC=0);**

  **该强制进入到Sleep状态可通过TJA1145主状态寄存器(03h)中的FSMS状态位来获取，如果FSMS为1表示最近一次进入到Sleep状态是由于低电压进入到Sleep状态，否则是通过SPI指令完成的状态切换。**

- **Overtemp: **  该状态是由于TJA1145放置温度过高导致被损坏的一个功能，一旦在Normal状态下温度超出一定阈值就会自动切换至OverTemp保护状态；

- **为了放置数据的丢失，TJA1145在温度超过预警阈值时会主动触发一个Warning，该Warning发生时就会触发OTWS置位，中断产生(即OTW=1)如果OTWE使能的前提下；**

- 在该模式下，CAN收发器将不能正常工作，CAN pin脚始终处于高阻状态，唤醒源将不会被检测，但是如果存在Pending的唤醒源，那么RXD引脚就会被拉低；

- 当温度低于特定阈值时，那么该状态便会自动切换至Standby状态；

- 如果Bat电压引脚低于某个特定阈值，则会自动从该状态切换至OFF状态；

- **Off:** 在该状态下电压由于低于特定阈值导致供电不足，当Bat首次连接时这是TJA1145的初始状态，**只要Bat电压低于某特定阈值，那么TJA1145将会从任意状态直接切换至Off状态**。在Off状态下，CAN pin脚以及INH引脚始终处于高阻状态；

  当电压高于某个特定阈值时，TJA1145便会重新启动触发初始化过程，从而经历特定时候后进入到Standby状态；

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAOIOIUzVCPGPofXf2UBS1mLaK0WC5GbN1OaXdRZx2aMOZbjbZyIhQTlKWRxVdqia2PNoBHic9u1IdA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图6 TJA1145 Operating Mode状态迁移图

如下图7所示，展示了TJA1145 Operation Mode与SPI通信，INH引脚，内部CAN模块状态以及RXD引脚的之间的关系。从中特别值得关注的有以下几点：

- **Standby状态下INH引脚可以拉高控制MCU各级电源输入，CAN无法正常收发通信；**
- **Normal状态下SPI可正常通信，INH引脚拉高，CAN模块的状态取决于CMC值，仅在Active状态下才能够开启正常通信，且仅在CMC=01/10/11下才能够正常收发；**
- **Sleep状态下INH引脚处于高阻状态，此时MCU各级电源可关闭，CAN始终处于Offline状态，无法正常收发通信；**
- **在Off状态与Overtemp状态下SPI都无法正常通信，在Sleep状态仅在VIO供电正常的情况下才可以，且通信速率不能太高；**
- **仅在Standby与Sleep状态下才检测唤醒事件，即仅在CAN处于Offline模式下才能够开启唤醒事件的检测；**

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAOIOIUzVCPGPofXf2UBS1mHsZImWXGsIJT5o43IIS2u2Tyv400bkKN6eX9YjiaCViaLNAM5eJEnFjA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图7 TJA1145各模式状态关系

**Can Operating Mode**

TJA1145内部集成的CAN收发器存在四种状态：**Active，Listen-Only，Offline**，**Offile Bias**状态。存在如下两种基本组合：

- **当TJA1145处于Normal状态时，CAN收发器状态取决于CMC值，如进入Offline或者Active或者Listen-only等；**
- **当TJA1145处于Standby或者Sleep状态，则CAN收发器始终处于Offline状态；**

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAOIOIUzVCPGPofXf2UBS1muceTrOFVjiaEn3hvpfVvuibELkgUIRFvrt3UeZGt5xibp3ibK6n9KsSYRw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图8 TJA1145内部CAN收发器状态迁移图

**CAN Active Mode**

- CAN收发器在此模式下能够正常发送或接收数据；
- 当**CMC=0x01**时，CAN收发器处于Active状态，VCC低压检测使能，如果出现VCC电压，就会直接切换到CAN Offline或者Offline Bias状态；
- 当**CMC=0x10**时，CAN收发器处于Active状态，且VCC电压检测被抑制，如果出现VCC电压异常，不会影响到CAN Active正常切换；(**实际应用过程中优先统一配置成CMC=0x10，以降低状态不断反复切换**)

**CAN Listen-only Mode**

- 该模式下CAN发送器被关闭，**当TJA1145处于Normal状态且CMC=0x11时，CAN收发器状态就会处于Listen-only状态**；
- 在该模式下如果TJA1145处于Normal状态且CMC=0x1，同时VCC电压低于90%，那么就会始终在这个Listen-Only状态；

**CAN Offline与Offline Bias Mode**

- 在CAN Offline模式下CAN收发器便会检测CAN总线以查看是否存在唤醒事件，同时CWE=1，CAN H与CAN L始终偏置接地；
- 在CAN Offline Bias Mode下也会检测CAN总线是否存在唤醒事件，只不过CAN H与CAN L会偏置2.5V，当超过一段事件总线没有活动，便会回归至CAN Offline模式下；
- **当TJA1145切换至Standby或者Sleep状态则会直接切换成此状态；**
- **当TJA1145状态为Normal且CMC=0x0，则会将状态切换成CAN Offline模式；**
- **当TJA1145状态为Normal且CMC=01同时VCC<90%,则也会将状态切换成CAN Offline模式；**

**CAN Off Mode**

- 当TJA1145状态为Off状态或者Overtemp状态时；
- 当Bat电压低于CAN接收器某个特定阈值电压时，当BAT电压回升至某个特定阈值时，便会进入到CAN Offline模式下；

**常见寄存器配置说明**

在对TJA1145进行正常使用时，无论是初始化过程还是正常对其进行控制，我们都需要针对常见的TJA1145寄存器进行控制，因此小T结合项目实战需要给大家列举了常用寄存器以及相关功能，如下图9所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAOIOIUzVCPGPofXf2UBS1mlibleKJa4VhjgicuZibuB2R6oUk1gEicwWRTiae5at1Y9ITH3omp5wwXIJg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图9 常见寄存器配置说明



#### TJA1145注意事项

- 使用TJA1145时，如果需要使用其CAN FD Passive功能(**0x2F寄存器设置 CFDC=1**)，那么需要选用型号TJA1145/FD才具备；
- TJA1145默认仅支持**CAN帧格式的特定帧唤醒**；
- 如果使能了VCC/VIO电压欠压监控，如出现在缓升缓降的过程中，就会导致TJA1145状态直接切换成sleep状态，因此一般优先选择CMC=0x10，则抑制了电压欠压检测。





**STANDBY MODE**

STANDBY MODE是TJA1043的一级节电模式。在STANDBY MODE下，收发器无法收发数据，低功率接收器被激活以监控总线活动。INH引脚为高电平。

![图片](SBC_CAN_Trcv.assets/640)

▲图3 Standby Mode

**LISTEN MODE**

在LISTEN MODE下，收发器的发送功能被禁用，接收仍正常，INH引脚为高电平。

**NORMAL MODE**

在NORMAL MODE下，收发器可以通过总线CANH和CANL进行传输和接收数据。总线上输出信号的斜率被控制和优化，以保证最低的EME。引脚INH为高电平。

![图片](SBC_CAN_Trcv.assets/640)

▲图4 Normal或Listen Mode

**GO TO SLEEP MODE**

该模式是进入睡眠模式的过程路径。在进入睡眠模式前，收发器表现为在待机模式下，并附加了一个向收发器发出进入睡眠的命令。在进入睡眠模式之前，收发器将保持在最短的保持时间(20~50us)进入Sleep模式。



如果STB_N或脚EN引脚的状态发生改变，或者在过去之前设置了唤醒标志，则收发器将不会进入休眠模式。



**SLEEP MODE**

该模式是TJA1043的二级节电模式。睡眠模式通过进入睡眠模式进入，当VCC或VIO上的欠压检测时间在相关电压水平恢复之前经过时也会进入。在睡眠模式下，收发器为待机模式，引脚INH设置为浮动。由此引脚控制的电源芯片将关闭。

![图片](SBC_CAN_Trcv.assets/640)

▲图5 Sleep Mode





# CAN网络管理（TJA1145如何实现MCU的休眠唤醒）

### **节点唤醒方式**

#### **本地唤醒：**

唤醒源来源于自身模块，比如常说的KL15，控制器由KL15线供电，即只能在钥匙置于“ACC”或者“ON”档时运行软件和维持CAN通信

- 对于正在运行的CPU软件，无论它处在什么状态，只要Hardware OFF，PMIC供电立即切断，3.3V，5.0V立即消失，程序立即停止运行，ECU系统进入OFF模式，不存在Sleep模式。该状态下PMIC也不消耗电能，ECU系统的电能消失是0，比较KL30节点的Sleep模式最节省电能。
- KL15节点没有下电流程，随时可能终止运行，没有时间进入AfterRun模式和做Eeprom最终存储。WakeUp信号的消失后ECU直接进入OFF模式，不存在Sleep模式,不耗费电能。
- KL30节点有完整的下电流程，软件根据WakeUp信号的消失可以控制自己按步骤进入AfterRun模式，存储数据到flash，设置PMIC进入Sleep模式。ECU进入Sleep模式只有PMIC在耗电。AutoSar网络管理只针对KL30节点。(参考BSWM和ECUM的上下电流程)

![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3Lph3UUaHFzJWPUYtj9HszHLkY3WnUz2BQ96ctjmelHibnBztVLoxmPTibg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### **远程唤醒：**

唤醒源来源于自身ECU节点所在的网络报文

- **网络唤醒是以网络管理报文为基础来协同整个网络“同睡同醒”**，CanNM采用分布式的直接网络管理方式来发送自身节点所需的网络管理请求及自身网络管理状态，并接受来自网络上其他ECU节点的网络管理请求与状态。**"同睡同醒"机制的目的是确保所有节点在睡眠和唤醒操作上保持同步。当一个节点准备进入睡眠模式时，它会通过网络发送一个特殊的同步消息，通知其他节点它即将进入睡眠状态。其他节点接收到该消息后，会做出相应的响应，以确保整个系统在同一时间进入睡眠状态。同样地，当一个节点准备唤醒时，它会发送一个唤醒消息来通知其他节点。其他节点接收到唤醒消息后，会做出相应的响应，以确保整个系统在同一时间唤醒。**
- 该状态机的状态类型可分为“三大三小”。

“三大”指的是**Bus Sleep Mode、Network Mode、Prepare Bus-Sleep Mode**；

“三小”则值得是Network Mode下的三个子状态：**Repeat Message State、Normal Operation Mode、Ready Sleep Mode**。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3Lp2ZejL69xyW4gpJwql7mScAibLDcUJ6xPJErjp6qC6GhfewxwpLxVw6A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 一旦进入Network Mode，计时器T_NM_Timeout就会启动，只要成功接收到来自总线上的NM报文或者成功发送至总线NM报文，都会将该计时器T_NM_Timeout重置。一旦T_NM_TIMEROUT 超时，那么就会离开该状态转而进入Prepare Bus-Sleep状态。
- 报文发送与接受状态。

**“Bus-Sleep”**阶段，只接收NM报文唤醒，不发送任何报文；

“**Pre-Bus-Sleep**”阶段，同样仅允许接收NM报文，对于早已在发送Buffer中的APP报文应发送完毕后立刻停止APP报文；

“**Network Mode**”模式下，除了在Ready Sleep阶段不允许发送NM报文之外，其余阶段APP报文与NM报文正常收发；

![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3LpGQoAl5vVf5AibZQPiaD2s5aNsgpfup1Ettj6eNcAnU6xD47UCDejNsibg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

CanNM报文周期性的向MCU发送，如果一旦一段时间没有收到，MCU就通过SPI向TJA115的寄存器写数据，要进入sleep模式，之后TJA1145在向PMIC拉低。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3Lp467g7C5eOqhQHvm2KnPdt8MznPUWBN806Jhfhrl7XqBOtnY51ut8YQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### **如何让TJA1145进入sleep和wakeup状态**

下图以TJA1043简单说明can报文如何使MCU进入到休眠唤醒状态：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/3g8Dklb9TwicSxaImwssicTdlDQhBPu3LpmLJtMYDFM6yW8br6PianZL9Q7fIjoYYPlSgKuS0LsE2XmED10ult8FQ/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/3g8Dklb9TwicSxaImwssicTdlDQhBPu3LpR0knhk8Md3IQPG0tjtzN7H50zXMc8AtVAxS4sA4v37bdC3FfovKlDQ/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3LpIP80RicdFiaHCswwQVjbPn4qT6LNdFUcSnbZubO5PckzS8ljKYibcjX9A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- **S1**：MCU满足休眠条件时，通过发送SPI相应指令让TJA1145进入Sleep状态；

![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3LpOTibYibibzibFhSCOzQ0ricMWJNUHrcIEsicACcTpkphMTpFKxKJYIveyiaKQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

例如下面的用法：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3LphvLSF61LPicfBS3UrJtkH0oWqDkD9GuItEMUqe7JibcnCdhCle9w1afw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3LpFIUpgib6iaSweavLloeEsvFsbp5IibtTzpdporibRRgSZX5kBW01JFFn9g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3Lpic6BdGeSNrDWPPYEL0loMwkPhQ3jN0LuwQjyPIloSWBicyArDjbAfOnA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3LphCTqg4NAibmDZ3KnYfn6rkAZGqTia58uweyTqY8buGMLib1uch2D0kfyA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3LpY9iaCRsbYV1xRQT2MYmyEu0vJcYzyDug8pNBhxibEDBquaQd2w5K7w3g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3LpWfj00ck3nGRFwhHGT2Uy6rFHiaVvibS1IkD8MnMHqw0qH3MJSTBxZQkA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3LpXEMM2u1ZEjick9qjvctvJJicMDpA3dzsBAnLQbMf4UTibdJINP4cb55Qg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

简单的指令代码实现就是下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3g8Dklb9TwicSxaImwssicTdlDQhBPu3LpFW0c6dFh5EvCb4cKOHu3YYIaPumUs79IsOLkZfwRNDib7KOvW7y7HSw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- S2：TJA1145进入到Sleep状态后，INH引脚就会拉低，控制5V或者3V关闭电源输出，间接导致MCU整个系统处理掉电状态，此时TJA1145始终处于供电状态（由于BAT始终有电），整个ECU成功进入到休眠状态；
- S3：TJA1145虽然处于Sleep状态，属于极低功耗状态，同步也检测着网络是否存在有效唤醒源；
- S4：当TJA1145发现有效唤醒源之后，就会自动从Sleep状态切换成Standby状态，在Standby状态下INH引脚拉高，此时5V与3V便会正常输出，从而MCU被正常供电，程序开启正常运行；





# 详解AUTOSAR：AUTOSAR CAN网络管理/CAN NM

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwGicByPTTRiak5HvyzufxzPJp5zYaXB2AQJwnRsWSE3VJcrPyibSnCeDo78CgK08PsSprqoeNIhCFp7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



AUTOSAR CAN网络通信中有三种模式和三种状态，如下图所示：



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwGicByPTTRiak5HvyzufxzPJpuEeDHRFRYdKVoenp7gvrva8jwo2vPC9V7MykyayukLcjOvReMs00oQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



3种运行模式：

1. 睡眠模式（Bus-Sleep Mode）；
2. 预睡眠模式（Prepare Bus-Sleep Mode）；
3. 网络模式（Network Mode）。



## 睡眠模式（Bus-Sleep Mode）

当CAN网络中没有远程唤醒或者本地唤醒请求时，ECU应处于睡眠模式（Bus-Sleep Mode），将功耗降低至最低水平，这种模式是ECU启动时的起始状态或者是ECU睡眠时的最终状态。



在该模式下，网络管理报文和应用报文都禁止发送，但是可以被网络上的报文唤醒。



CAN收发器应当支持设定唤醒帧（如果有CAN收发器的情况下），ECU只会接受到特定的NM报文才会正常唤醒，否则就会一直处于休眠状态，能够不受网络上应用报文的干扰。



## **预****睡眠模式（Prepare Bus-Sleep Mode）**

ECU进入预睡眠模式（Prepare Bus-Sleep Mode）后禁止网络管理报文的发送，允许接收网络管理报文。应用报文已经在buffer中的一般允许继续发送，进入到预睡眠模式（Prepare Bus-Sleep Mode）计时器CanNmWaitBusSleepTime就会启动，一旦计时器CanNmWaitBusSleepTime超时，就会进入到睡眠模式（Bus-Sleep Mode）。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwGicByPTTRiak5HvyzufxzPJppZqpDcVq4MxNsfXdvcPHicWosG3ILfI86ZMic0IiaXA0etfQvuDcUEwdA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



本项目AUTOSAR配置中设定计时器CanNmWaitBusSleepTime为2秒。

## 网络模式（Network Mode）

当CAN网络处于开启或者工作情况下会进入网络模式，ECU进入网络模式（Network Mode）后计时器CanNmTimeoutTime就会启动，只要成功接收到来自CAN总线上的网络管理报文或者成功发送至CAN总线网络管理报文，都会将计时器CanNmTimeoutTime重置。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwGicByPTTRiak5HvyzufxzPJpFicDlNmvic3m3ryBf5QpJyeLrHtg3uEibicypwo5dKn2jnOHrAH97C9vqw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



本项目AUTOSAR配置中设定计时器CanNmTimeoutTime为2秒，所以0X505网络管理报文的发送周期要在2秒内，超时会进入预睡眠模式（Prepare Bus-Sleep Mode）。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwGicByPTTRiak5HvyzufxzPJphWM0421rkeg6vjae3Exx9qUhTDjUyQO6JhzItiaDqxNJLvCgra44GMQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



网络模式（Network Mode）包含3种状态：

1. 报文重复状态（Repeat Message State）；
2. 常规运行状态（Normal Operation State）；
3. 准备睡眠状态（Ready Sleep State）。



## **报文重复****状态（Repeat Message State）**

当ECU从其他模式进入网络模式（Network Mode）时，默认进入报文重复状态（Repeat Message State）。该阶段是CAN网络正式开始工作前的准备阶段，用来等待CAN网络中所有相关节点进行网络通信的准备时间。



该模式下计时器CanNmRepeatMessageTime规定了重复发送网关管理报文的时间，CanNmImmediateNmTransmissions规定了发送网络管理报文的次数。

在报文重复状态（Repeat Message State）ECU使用计时器CanNmMsgCycleTime周期时间发送网络管理报文。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwGicByPTTRiak5HvyzufxzPJpWLiaY86ySHnHJNkAd5S9BslrHHHg9GiboJF4w6qxVmo2aGOt5DfxqlRw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwGicByPTTRiak5HvyzufxzPJpOv8vgRicic3JCZPgzFGW9VzsiaD25cppiaicwrKwWTbiaC4hNZ7nkcPLKPoA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



本项目AUTOSAR配置中发送网络管理报文的周期是640毫秒，重复次数为5，总时常3.2秒。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwGicByPTTRiak5HvyzufxzPJpicW3chIQVWBYON0aiaXGb1TISocFa6ghSfydPuasDaIbURyby9CM4y7A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## **常规运行****状态（Normal Operation State）**

ECU进行正常CAN通信时会处于常规运行状态（Normal Operation State），该阶段中，节点要按照计时器CanNmMsgCycleTime时间周期发送网络管理报文。每次成功发送或者接收CAN网络报文计时器CanNmTimeoutTime就会重置。

在常规运行状态（Normal Operation State）下的网络管理报文和应用报文都应该正常收、发通信。



## **准备****睡眠状态（Ready Sleep State****）**

在准备睡眠状态（Ready Sleep State）ECU应当停止发送网络管理报文，每次成功接受到来自CAN网络上的网络管理报文，计时器CanNmTimeoutTime就会重置，一旦CanNmTimeoutTime超时，就会进入预睡眠模式（Prepare Bus-Sleep Mode）。



在AUTOSAR中规定了各种模式和状态下计时器的默认时间：



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwGicByPTTRiak5HvyzufxzPJpyky1XL3maM4KIoyCFO6XXWkXB8n4RUvpq096NCQNw7nnmUYT5H3CPg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



配置参数在AUTOSAR代码中体现如下所示：



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwGicByPTTRiak5HvyzufxzPJp4fUZ28c8pvrr2QXiaib8ib1KibkVTqUswAT1PjkIuML5xBwnWlIicnzwUAQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



下图梳理了所有网络管理的模式转换情况，通常控制器的状态转换如蓝色箭头所示：



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/WU9hWnL9BwGicByPTTRiak5HvyzufxzPJpg4YYgwbkxiczicDibQ1q5HP2XsAMeQicEYH0y9Mdw5CN1JCgYKtTQLPF4Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)