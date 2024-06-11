# AUTOSAR基础篇之EcuM

### 前言

当你看到ECU从启动状态至正常运行状态，再从正常运行状态至休眠或关闭的过程时，你是否曾想过以下一些问题呢？

- ECU是怎么启动或关闭的呢？
- ECU启动方式有没有一般规律呢？
- 按照AUTOSAR标准，ECU启动过程又可分为哪几个阶段呢？
- 。。。。。。

今天，我们来一起探讨并回答这些问题。为了便于大家理解，以下是本文的主题大纲：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBCd2PB8ApibpqECibgnzvvR1BiceDgXicNbAHjnicCBhE3BOam6suyENTJWpb3NmKHu15ULlxjujnichJw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**熬夜肝文系列之技术干货，阅读稍长，但应有尽有，一文搞懂EcuM爽文，认真看完，也必定让你有所收获！**

------

### 正文

#### EcuM模块总体介绍

##### 主要功能

**EcuM**模块作为AUTOSAR中的标准模块，全称为（**ECU State Management）**。故名思义，指的就是ECU 的状态管理，不过需特别强调的是ECU上下电流程的状态管理，具体可以简单概括为以下五个方面的内容：

- **Startup 初始化流程状态管理；**
- **ECU运行状态管理；**
- **ShutDown流程状态管理；**
- **Sleep流程状态管理**
- **Wakeup Source管理；**

##### 总状态机（Flexible 与 Fixed）

在具体介绍上述5个状态管理过程之前，我们有必要对ECU启动过程有个总体的感性认识，以便于对后续各个阶段的之间的关系有个较为清晰的了解。如下图1所示，描述了一般情况下ECU的启动流程。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBCd2PB8ApibpqECibgnzvvR19QnRHXlB9Jyiatib1ibSP3hKM9TujcDOrq74D3ibLBAnNTibOPvy09HUibxg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图1 ECU一般启动流程

在上述的ECU启动过程中，可以看出ECU的一般启动过程涉及到Boot，C_Init,  EcuM，OS等模块，在这些模块的共同接力下保证BSW及RTE成功初始化，进而使得整个SW-C处于正常running的过程。

ECU启动时，首先通过中断向量表运行引导程序（俗称BootLoader），Bootloader在满足一定条件下跳转至APP程序中的C_Init处并指向Main函数。

在Main函数中首先完成堆栈空间的初始化，然后调用**EcuM_Init**函数进入到后续的StartPreOS，StartOS阶段。

在开启OS的初始化函数中调用**EcuM_StartupTwo**进行第二启动阶段的初始化，最后就是进入StartPostOS阶段，如完成BswM模块的初始化，进而将控制权转交给BswM模块。

由于接力赛中首棒很关键，因此本文将重点关注EcuM模块的启动与关闭过程，按照AUTOSAR定义，EcuM可分为两种模式：**Flexible与Fixed模式**。

- **Flexible 总状态机**，如下图2-1所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBCd2PB8ApibpqECibgnzvvR1wiajWQojXb3YnHP0sgEpzvEmEJuOiaaic51XVLCOV1j3UokItiadlwDGPg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图2-1 EcuM Flexible 总状态机

在上图中，Startup阶段按照开始OS节点作为分水岭，可分为StartPreOS与StartPostOS两个阶段。经历过Startup阶段之后，则会进入到UP阶段。

在UP阶段则是正常运行状态，当条件满足时，可以根据CPU是否进入到低功耗状态还是OFF状态，相应进入到Sleep阶段与ShutDown阶段，当然如果是Reset，那么也是先进入到Shutdown阶段，最后跳转至Startup阶段。

若进入到Sleep阶段之后，也存在着两种CPU低功耗模式：Poll与Halt模式，后者比前者更节约电能且无需运行代码，具体采用哪个则可根据当初的系统设计而定。

在该阶段不会关闭OS，OS始终低功耗的running状态，同时也会不断的对唤醒源进行监控，若唤醒源满足，则会直接跳转至RUN阶段。

若进入到Shutdown阶段，会经历两个阶段：OffPreOS与OffPostOS阶段，前者则是为Shutdown OS之前所做的准备，后者则是关闭OS之后，选择对应的函数执行关闭ECU还是重启ECU的操作。

- **Fixed 总状态机**,如下图2-2所示：

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBCd2PB8ApibpqECibgnzvvR1uYqFf3ISTy51ia7237mQsxUHh117DlLTJNu0FvHuAPQ0Y7XaFgGACng/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图2-2 EcuM Fixed 总状态机

在上图2-2中，较为清晰的描述了EcuM Fixed模式下五种状态**Startup，Shutdown，RUN，Sleep，Wakeup**的状态组成以及状态切换的过程，**其中OFF，Sleep，RUN是稳态，而Startup跟Wakeup则是暂态**。

在Startup阶段，同样按照Flexible 模式中开启OS为界限，分为Startup I与Startup II两个阶段；

当唤醒事件能够控制CPU供电时，则需要进入Wakeup阶段验证Wakeup Event是否有效，相反如果不带电源控制，则直接进入RUN阶段。

若进入到RUN阶段，可分为两个阶段：RUN II与RUN III两个阶段。其中RUN II指的是正常运行阶段，RUN III则是SW-C为即将进入到ShutDown所需要做的前提准备。

若进入到ShutDown阶段，首先会进入到PreShutDown阶段，然后按照Shutdown的目标不同，可以分为reset，OFF，Sleep三条路径。

如果Target为Sleep，则进入到Go Sleep阶段，若在该阶段检测到唤醒事件，那么直接跳转至Wakeup Validation阶段。

如果Target为OFF或Reset，则需经历Go OFF I与Go OFF II两个阶段，reset则会重新跳转至Startup阶段，而OFF则是直接关闭ECU。

若进入到Wakeup阶段，则需要进行四个阶段的唤醒源验证，主要分为Wakeup I，Wakeup Validation，Wakeup Reaction，Wakeup II阶段；

若进入到Sleep阶段，则可以分为两种Sleep模式：Sleep I 与Sleep II，一般两者选其一。其中Sleep I阶段（Halt），此阶段不运行代码， 等待唤醒事件，然后跳转至Wakeup阶段；

其中Sleep II阶段则为Polling阶段，这个阶段则会低功耗运行代码，并且等待唤醒事件，如果存在，则进入到Wakeup阶段。

**Fixed与Flexible模式区别与联系**，从上述EcuM Fixed Mode与Flexible Mode的描述，便可知两者存在着很多的相似点，同时也存在着彼此之间的差异，因此小T我将两者的区别与联系展现如下表1所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBCd2PB8ApibpqECibgnzvvR1GOPic56UxcD8MfHm8ibxdMDt3M38mvD8ZOxNicu0KkSRudQHEpQQLufmQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



表1 EcuM Fixed与Flexible模式区别与联系

由上分析可知，EcuM Flexible可以兼容Fixed模式，是传统ECU的启动过程的扩展，也可理解Flexible是Fixed模式的更高一层抽象，Fixed则可以称作Flexible模式的一种表现形式。

同时Fixed模式明确了各个阶段的状态及状态切换过程，而Flexible则更为灵活，可以实现多核启动，局部快速启动等特性，为了更好的了解Flexible模式的启动思想，本文将以重点介绍Fixed模式下各状态机的状态机及切换过程，举一反三。

按照EcuM的主体功能，对应的将从以下五个过程来展开讲解**EcuM Fixed Mode**下的各状态机状态及状态切换过程。

- **Startup Sequence** : 完成启动过程的初始化；

- **Run Sequence** ：正常运行及退出运行状态阶段

- **ShutDown Sequence**：shutdown 或Reset ECU的阶段；

- **Sleep Sequence**：ECU休眠阶段;

- **Wakeup Sequence**: ECU 验证唤醒源阶段；

  

#### Startup Sequence

STARTUP阶段的目的就是初始化基础软件模块，主要可分为两个阶段：启动OS之前的初始化以及启动OS之后的初始化，如下图3所示，为Startup Sequence的顶层设计。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBCd2PB8ApibpqECibgnzvvR11WCEiacliaBPXib2FOSvCUhUveOPia7WKPnDbsonMd0q0XUwcRleJGwGwQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图3 Startup Sequence顶层设计

##### STARTUP I

如上图3所示，通过调用EcuM_Init函数则进入到STARTUP I阶段，在该阶段主要会调用下列两个Callout函数完成OS启动前的初始化工作；

- **EcuM_AL_DriverInitZero**：完成无需OS支持的底层硬件驱动的初始化或者其他低水平的初始化（无需postconfig），将这部分驱动的初始化称为Init Block 0；

- **EcuM_AL_DriverInitOne**：完成无需OS支持的底层硬件驱动的初始化或者其他低水平的初始化，将这部分驱动的初始化称为Init Block 1;

  

##### STARTUP II

在STARTUP II阶段则是在start os函数中调用EcuM_AL_DriverInitTwo ，随后开启RTE，最后调用函数EcuM_AL_DriverInitThree最后初始化那些需要NVM数据的BSW模块。

- **EcuM_AL_DriverInitTwo** ：需要OS支持但是无需使用NVM的BSW模块初始化，并将此部分驱动的初始化称为Init Block II；
- **EcuM_AL_DriverInitThree：**需要OS支持同时也需要使用NVM的BSW模块初始化，并将此部分驱动的初始化称为Init Block III；

特别需要注意的是，STARTUP 1主要用于为start OS而作的驱动函数初始化，启动时间应当尽可能短，而START UP II尽可能完成所有所需模块的初始化。

且中断一般不允许在startup I阶段使用，如果需要使用，也只能使用Category I，不能使用Category II。

为了加深大家对Startup两个阶段的驱动模块初始化的认识与理解，特此将其总结如下表2所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBCd2PB8ApibpqECibgnzvvR187kN2dRUddl6Mcjm0CibQ3aFKUj7OIBmvaATkbLbUyTVpp9atZyBzPw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



表2 StartUp阶段驱动初始化列表



#### RUN Sequence

RUN阶段可以划分为以下两个阶段，一个是RUN II，表示正常工作状态，另一个是RUN III，表示为进入到ShutDown所作的前提准备，顶层设计如下图4所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBCd2PB8ApibpqECibgnzvvR1xPIXLS5l9UtU1k9QhibS6JzfG6MWZHqm6iaoI5qiciaOwgXhtkkIEJd1ibg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图4 RUN Sequence顶层设计

##### RUN II

在RUN I阶段则表明已完成了所有BSW模块（包括OS及RTE）的初始化，开始运行SW-C程序。在该阶段，将主要完成以下几种操作：

- 通过调用函数ComM_CommunicationAllowed来使得相应的通信通道允许通信；
- 在该阶段，EcuM将允许保持一个最小的运行事件EcuMRunMinimumDuration，以便让SW-C有机会向EcuM模块请求RUN Request；
- 在该阶段也需要进行休眠总线的唤醒源验证工作；
- 除非没有通信请求，否则ComM不会释放RUN Request，也就不会退出RUN II阶段；

##### RUN III

当最后一个Run Request被释放之后，EcuM就会进入到RUN III阶段（即Post RUN 阶段）。在PostRUN主要完成以下几种操作：

- 在RUN III阶段，如果Sw-C请求PostRun，那么就会停留在该状态，SW-C可以运行其相应的代码如存储重要的数据等，直至释放PostRun Request；
- 若在该阶段存在RUN Request，那么就会立刻跳回到RUN II阶段；
- 若既不存在RUN Request，也不存在PostRun Reqest，那么就会直接进入到ShutDown阶段中的PreShutdown阶段；



#### ShutDown Sequence

在ShutDown阶段，主要根据ShutDown Target不同而进入不同的状态机处理流程。如下图5所示，总体上体现了根据Target不同而做出的不同状态机处理。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBCd2PB8ApibpqECibgnzvvR1pI7Miajfw2ria62mEUl7Ix87H3RNSKx21aiaoC85cNqr9ZjzicfxpGTGpg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图5 ShutDown Sequence顶层设计

从上图可知，不管ShutDown Target是什么，都会经历PreShutdown阶段，进入到该阶段，主要完成以下操作：

- De_Init所有的SW-C，同时保证通信协议栈处于关闭状态。
- 清除所有的Wakeup Event；
- 关闭Dem模块；
- 根据不同的ShutDown目标进入不同的状态（Sleep或者OFF或者Reset）；

##### ShutDown Target

在ShutDown阶段，ShutDown Target非常重要，因为其决定了ShutDown阶段应当走何种路线。ShutDown Target可分为以下三种：

- **OFF：**CPU掉电；
- **RESET：**这属于一个暂态，CPU Reset；
- **Sleep：**CPU处于低功耗状态，未掉电；

默认的ShutDown Target可以通过配置得到，当然SW-C可以直接调用函数接口 **EcuM_SelectShutdownTarget**来覆盖掉默认的ShutDown Target。

##### Go Sleep

当ShutDown Target为Sleep时，那么就会进入到Go Sleep阶段，在该阶段主要完成以下操作：

- 调用NvM_WriteAll函数完成写操作，同时开启NVM写超时计数器；
- 调用函数EcuM_EnableWakeupSources使能Wake up事件接收；
- 在该阶段，OS并没有关闭，处于正常Running状态；
- 若此阶段存在Pending Wakeup Event，则直接调用函数NvM_CancelWriteAll取消写操作，然后直接跳转Wakeup阶段的Wakup Validation子状态；
- 当Nvm_WriteAll成功执行完或者写超时，则直接进入到Sleep阶段；

##### Go OFF I

当ShutDown目标为OFF或者RESET时，则首先进入到该状态。在该阶段，主要完成以下几种操作：

- 仅设置LIN的通信状态为FALSE；
- 完成ComM，BswM的Deinit操作；
- 调用NvM_WriteAll函数完成写操作，并开启写超时计数器；
- 等待NvM写成功或者NvM写超时，调用函数**ShutdownOS**关闭OS；
- 在ShutDown OS的过程中通过shutdown hook函数调用**EcuM_ShutDown**来进入OFF II阶段；

##### Go OFF II

当ShutDown Target为OFF或者RESET时，经过OFF I阶段就会最终调用**EcuM_ShutDown**进入到该阶段，在该阶段，主要完成以下几种操作：

- 如果ShutDown Target是**OFF**，则调用Callout函数**EcuM_AL_SwitchOff**来直接断掉CPU供电；
- 如果ShutDown Target是**RESET**，则调用Callout函数**EcuM_AL_Reset**进而调用MCAL标准函数**Mcu_PerformReset**来重启CPU；



#### Sleep Sequence

当ShutDownTarget为Sleep，经历了Go Sleep阶段后，便会直接进入到Sleep阶段，Sleep阶段的总体流程如下图6所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBCd2PB8ApibpqECibgnzvvR1Kcg6XbZHeHPVGaicThKibhWUslVxawtfGQslYCsVQYHwUBQa5GiaI5z9g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图6 Sleep Sequence顶层设计

如果所有的RUN Request没有被释放，则不会进入到Sleep阶段，也就意味着进入到Sleep阶段了，表示当前已没有RUN Request。

在进入Sleep状态之前，EcuM模块应当将所有的通信接口处在Standby状态，且需要使能必要的Wakeup Source。

进入到Sleep模式后，可以选择MCU Halt模式，等待Wakeup Event触发，也可以选择Polling模式，主动查找当前有无唤醒事件，两者根据系统设计选择其中一种即可。

##### Sleep I

在Sleep I阶段，即Halt模式，在该低功耗模式下，无需运行代码，但需要存在某种CheckSum算法来保证唤醒前后RAM空间的数值不会遭到破坏。

即通过调用**EcuM_GenerateRamHash**生成对应的Hash值，接收到唤醒事件后，则调用**EcuM_CheckRamHash**来完成前后RAM一致性检查。

若一致，则进入到Wakeup阶段，若不一致，则调用Dem模块的Event ID来上报故障并触发重启来保证安全。

##### Sleep II

在Sleep II阶段，即Polling模式，在该低功耗模式下，会降低系统时钟频率来运行代码，并实时检查有没有相应的唤醒源。

通过调用Callout函数**EcuM_SleepActivity**以及**EcuM_CheckWakeup**来检查是否存在唤醒源。



#### Wakeup Sequence

如上图2-2所示，无论是在Go Sleep阶段还是Sleep阶段或者是带有电源控制的唤醒阶段，如果监测到Wakeup Event就会进入到该阶段，目前Wakeup Sequence可以分为以下四个基本阶段：

- Wakeup One：
- Wakeup Validation
- Wakeup Reaction：
- Wakeup Two：

如下图7为Wakeup Sequence的总体流程图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBCd2PB8ApibpqECibgnzvvR1iaZcnMXnT6GYxXjibGz0HjDJnI5j2BnzZHjgxx2wJlUQricQx9EodPUPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图7 Wakeup Sequence顶层设计

##### Wakeup I

当从Sleep状态进入到Wakeup阶段时，首先进入到Wakeup I阶段，在Wakeup I阶段主要完成以下几种操作：

- 设置MCU模式为Normal Mode；
- 抑制当前pending的Wakeup Event；
- 调用函数**EcuM_AL_DriverRestart**重新启动驱动，主要初始化**Block I**与**Block II**；
- 使能Run Reqest以及PostRun Request；
- 解锁Scheduler并可能重新运行OS；

##### Wakeup Validation

当从Go Sleep或者通过待电源控制的唤醒条件下启动时，则会进入到该阶段，在该阶段主要会进行以下操作：

- 获取当前Pending Wakeup Event并调用函数**EcuM_ValidateWakeupEvent**开启验证；
- 如果validate超时，则可以通过调用函数EcuM_StopWakeupSources停止验证工作；
- 在该阶段，存在以下5种唤醒源在任何时刻都**无需验证**：
- **WKSOURCE_POWER；**
- **WKSOURCE_RESET**
- **WKSOURCE_INTERNAL_RESET；**
- **WKSOURCE_INTERNAL_WDG ；**
- **WKSOURCE_EXTERNAL_WDG；**

##### Wakeup Reaction

经过Wakeup Validation阶段后，肯定会进入到该阶段，在该阶段主要会进行以下几个操作：

- 根据event Validation之后的结果选择进入不同的阶段，一种是验证有效，进入**RUN II**阶段，另外一种是验证无效，进入**Go Sleep**阶段；

##### Wakeup II

当经过Wakeup Reaction之后，如果验证成功就会进入到该阶段，在该阶段主要完成以下几类操作：

- 如果是从Sleep阶段跳转至该阶段，则首先要调用Dem_Init函数来完成Dem模块初始化，因为是新一轮operation cycle；
- 如果是从Startup阶段跳转至该阶段，则可能需要等待NvM readall操作完成；
- 最后可直接跳转至RUN II阶段直接运行；



#### 常用函数接口

为了更好的使用该模块函数以及遇到问题时方便调试该模块，特将BswM模块中较为重要的常用函数列举如下表3所示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBCd2PB8ApibpqECibgnzvvR1MFMYffSVSWLktJTJs3ESNNMkKh2HfkficQ5F9iby3TGibfSwHW6picSCPg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



表3 EcuM模块常用函数列表



# AUTOSAR基础篇之BswM

原创 汽车小T8 [ADAS与ECU之吾见](javascript:void(0);) *2021-05-31 08:20*

### 前言

首先，请问大家几个小小问题，你清楚：

- 你知道BswM是做甚的吗？
- 常说的APP Mode或者System状态机与BswM关系又是如何的呢？
- BswM模块作为AUTOSAR的一个标准模块，内部工作机制如何实现？
- BswM与各SW-C以及各个BSW模块又是如何交互的呢？
- 。。。

今天，我们来一起探索并回答这些问题。为了便于大家理解，以下是本文的主题大纲：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icA4qGVlYEibdiaEXHCpebDmZVV1u5nXVbicM6AiafRvxAkuXkIWSuENhu7xkyCIrTSB3VEelicYeLTPCdQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

------

### 正文

#### 总体设计框架

顾名思义，**BswM**全称为基础软件管理模块（即Bsw Management）。该模块根据来自BSW或者SW-C特定的输入，在满足一定的规则条件下执行直接对各个BSW模块的序列化操作。

从我刚刚总结的话语中不难得出BswM具有以下三个显著的特点：

- **输入来源于各SW-C或者各BSW模块；**
- **执行时需要满足一定的规则条件；**
- **实现对各BSW模块的序列化操作；**

按照AUTOSAR规范，我们可以将前两者叫做模式仲裁过程，而最后的步骤称为模式控制过程。

为了便于大家理解，首先分别针对上述模式仲裁过程与模式控制过程做总体性介绍。

##### 模式仲裁过程

如下图1所示，BswM模块将会接收来自SW-C或者BSW模块的**Mode Request**或者**Mode Indication**作为模式仲裁的两种输入方式。

通常Mode Request来源于SW-C模块，当然也不排除BSW模块，比如DCM请求ComM进行full communication的时候。Mode Indication则总是来源于BSW模块，比如CANSM，EcuM，WdgM等。

在图中也可以看到定义了模式请求的一些规则条件，如只有Normal_Mode为TRUE且IFC1_Bus_Off为False时，条件成立为TRUE，其他情况则为False。

由此可见，模式仲裁过程中的规则条件也就是真值表达式，而真值表达式的结果就是模式仲裁之后的结果，并作为下一环节模式控制过程的输入。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icA4qGVlYEibdiaEXHCpebDmZVXzjWkTT8pueffLichlOyhQLLiaemyfUVSDRlqXtGjYOtiaQ1u5ia5XXGMA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 模式仲裁过程

上述过程中提到的Mode Indication 与 Mode Request都是以同等的地位在BswM中接收仲裁，且这两种类型都可以在AUTOSAR标准配置项**BswMModeRequestSource**得以体现。

同时BswM模块会通过调用其他BSW模块如EcuM，COM，ComM，SM，PduR的标准接口来获取每个模块的当前状态作为模式仲裁的输入。

但需要注意的是如果在调用这些标准接口之后返回的结果为错误（E_NOT_OK），那么BswM模块可以设置DTC来告知上层这一错误或者取消当前正在执行的Action List。

##### 模式控制过程

模式控制指的是基于上述上述模式仲裁的结果（TRUE or FALSE）执行相应的行为序列。

这些行为序列通常称为“**Action List**”，且这些Action List都是有序的，也就意味着可根据项目需要自由配置各类Action的执行顺序。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icA4qGVlYEibdiaEXHCpebDmZVZyqozDJiayMApQqOwJFNqSoV8b640G6L9RdvpVY0DbfCGuicV0NAYSBQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图2 模式控制过程

如上图2所示, 每一个Action List可以对应1个或者多个行为，AUTOSAR支持实现不同行为的自由组合与复用。每一种行为都可以分为以下三种方式来实现：

- 直接调用其他BSW模块或者RTE模块的**标准接口**来实现一系列的控制（即**BSWM_ATOMIC**）；

- - ComM：设置对应通信接口的通信模式或允许在对应的通道上通信；
  - COM：实现IPDU报文的切换；（带有初始值或者不带有初始值）；
  - COM: 使能或者关闭信号的deadline timeout monitoring；
  - NM：开启或者关闭NM通信；

- 链接其他Action List中的内容（即**BSWM_LIST**）；

- 执行某些仲裁规则（即**BSWM_RULE**）；

除此以外，模式控制在可控制多个行为的同时，也可以通过ID号来保证各个Action的执行顺序，所以说Action List是有序行为列表。某些仲裁规则可以作为从属规则成为某个主规则的Action List的一员。

从整体上介绍了模式仲裁与模式控制的过程之后，相信大家对BswM模块的控制流有了基本的了解。

接下来，将针对模式仲裁与模式控制过程中的具体细节信息展开讲述，以便大家对BswM模块的配置过程能够更为清晰。

#### 模式仲裁

##### 模式请求来源(ModeRequestPorts)

模式请求来源指的是模式仲裁过程中需要判断的数据源是什么，而这些数据源的获取都是已定义好的标准接口。

你只需要进行相应配置就能完成针对其他Bsw模块数据获取，即BswM会提供并生成相应的函数接口来获取对应Bsw模块的数据输入，举例说明常见的数据请求来源如下表1所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icA4qGVlYEibdiaEXHCpebDmZVLicXslqjNEHAnic9Jr0nkCoV2WeDBJLQEJxKATs57ibicWhlSxL7eahTcw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



表1 模式请求来源举例

针对上述的模式请求来源，BswM存在以下两种方式去查询这些请求来源的状态：

- **事件触发型**

- - 在该模式下，只有接收到Mode Indication或者Mode Request才会执行，适用于模式请求来源数据变化不大较为稳定的场合；

- **轮询遍历型**

- - 在该模式下，BswM会在自身Mainfunction函数中主动查询Mode Indication以及Mode Reqest的状态，适用于Indication或者Request变化较为频繁的场合，一般情况下，都可以直接采用该模式去查询请求数据来源的状态；

上述两种状态可通过配置参数**BswMRequestProcessing**来进行选择。

##### 模式条件(ModeCondition)

模式条件指的是根据上述模式请求来源与设定的值相比较的**单一表达式**。比如若模式请求来源为X，N为设定的常量状态，那么模式条件就是配置X ==N或者X ！=N的表达式。

##### 逻辑表达式(LogicExpressions)

逻辑表达式是相比模式条件而言,它可以实现多个模式条件的逻辑组合。举例说明如下：

若Logic Expression仅需要两个Mode Indication 1与Mode Indication 2的逻辑组合，当然理论上可支持n个单一表达式的逻辑组合，取决于实际情况的需要。

Mode Condition 1：X == 3；

Mode Condition 2：Y == 4；

Logic Expression A = （Mode Condition 1 (OR或AND或XOR或NAND)Mode Condition 2 ）；

##### 模式规则(ModeRules)

模式规则指的是根据上述逻辑表达式的结果（TRUE or FALSE）来执行相应的Action List。也就意味着模式规则是为了实现逻辑表达式与相应Action List 的Mapping关系。

为了对模式仲裁过程中的三大构件（**模式请求来源、模式条件、逻辑表达式**）的相互reference关系及执行对应关系有个更好的理解，如下图3所示则较为清晰地表示了彼此之间的关联关系。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icA4qGVlYEibdiaEXHCpebDmZVicsTBqUEO0oDUNjiblYevE2HWLa68uhJV2nYdR6lwiagbwWU6ZjygrAQg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图3 模式仲裁三大构件的组织关系

##### 模式规则的初始化

模式规则可以设定其Map的逻辑表达式的结果初始值为TRUE或者FALSE，以便默认执行相应的Action List。

该初始值可通过参数**BswMModeInitValue**来进行配置，同时该参数属于BswMRules的子参数。

如果没有配置初始值，则模式仲裁的初始值处于undefined的状态，在该状态下意味着BswM在完成其初始化之后默认就会执行一次。绝大多数情况下，模式规则的初始值为FALSE。



#### 模式控制

##### 模式控制基本流程

模式控制基本流程就是根据模式仲裁的结果去执行相应的Action List。如下图4所示，以SW-C组件模式请求为例：

- S1：SW-C通过sender port经由RTE向BswM模块发送模式请求信息；
- S2：BswM模块通过receive port接收请求并根据已配置好的模式仲裁规则得出相应的仲裁结果；
- S3：执行相应的Bsw相关的Action List及RTE状态切换；
- S4：经过RTE传输的状态将会传递给到需要使用该模式的SW-C以及返回请求该模式的SW-C；

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icA4qGVlYEibdiaEXHCpebDmZVhUcMAMULsvxrLiaqpWkp7kONgPStqic9TGKBibxX6jicibIXRvFwwweUR9w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图4 模式控制基本过程

##### 模式行为

模式行为作为模式控制的基本执行单元，在执行Action List的过程中也存在着以下两种方式：

- 循环执行（BswM_Condition），只要模式仲裁规则成立，一直连续不断的执行;
- **事件触发**（BswM_Trigger），仅在模式规则变化的情况下，才会去执行相应的Action List；

这两种执行方式可以通过模式控制过程中的参数**BswMActionListExecution**来进行配置。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icA4qGVlYEibdiaEXHCpebDmZVuDaUlMgibFwmZ0TQ5FnuyJ0da4MFj2nmlU4gzmXmjbEIP5JiavdXtb9w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图5 模式行为的触发方式

如上图5展示了BswMruleInitValue与模式行为的触发方式在不同组合情况下执行Action List的结果，以便我们在配置过程中能够注意到它们之间的关联。

需要注意的是如果在执行Action List的过程中返回结果为E_NOT_OK，那么BswM模块应当终止Action List的执行，如果需要实现，那么需要设置配置参数**BswMAbortOnFail**结果为TRUE。

在模式控制的配置选项中，一般会存在BswMAction与BswMActionList两类选项，解释如下：

- **BswMAction:** 则用来配置各种各样标准的行为；（如Com模块报文切换等）
- **BswMActionList**: 用来定义action的集合（即从BswMAction中取任意个自由组合）；

#### 常用函数接口

为了更好的使用该模块函数以及遇到问题时方便调试该模块，特将BswM模块中较为重要的常用函数列举如下表2所示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icA4qGVlYEibdiaEXHCpebDmZVGBLncGg0KDz32MQFdUX2SmibEuTia4eDw19luBTKqPGp7OAtHeXpMnDg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

表2 Bsw模块常用函数列表

