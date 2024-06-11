# [AUTOSAR笔记：ECU级开发——RTE、BSW（五）](https://www.cnblogs.com/fortunely/p/17464661.html)



目录

- ETAS RTA系列工具
  - [RTA-BSW简介](https://www.cnblogs.com/fortunely/p/17464661.html#rta-bsw简介)
  - [RTA-RTE简介](https://www.cnblogs.com/fortunely/p/17464661.html#rta-rte简介)
  - [RTA-OS简介](https://www.cnblogs.com/fortunely/p/17464661.html#rta-os简介)
- RTAS RTA系列工具入门
  - [RTA系列工具安装](https://www.cnblogs.com/fortunely/p/17464661.html#rta系列工具安装)
  - [RTA系列工具界面](https://www.cnblogs.com/fortunely/p/17464661.html#rta系列工具界面)
- CAN协议栈概念与配置方法
  - [CAN协议栈概念](https://www.cnblogs.com/fortunely/p/17464661.html#can协议栈概念)
  - [CAN通信协议栈配置方法](https://www.cnblogs.com/fortunely/p/17464661.html#can通信协议栈配置方法)
- [EcuM模块概念与配置方法介绍](https://www.cnblogs.com/fortunely/p/17464661.html#ecum模块概念与配置方法介绍)
- [BswM模块概念与配置方法介绍](https://www.cnblogs.com/fortunely/p/17464661.html#bswm模块概念与配置方法介绍)
- [BSW模块代码生成](https://www.cnblogs.com/fortunely/p/17464661.html#bsw模块代码生成)
- [服务SWC、应用层SWC端口连接](https://www.cnblogs.com/fortunely/p/17464661.html#服务swc应用层swc端口连接)
- RTE配置、代码生成
  - [RTE Contract阶段生成](https://www.cnblogs.com/fortunely/p/17464661.html#rte-contract阶段生成)
  - [RTE配置](https://www.cnblogs.com/fortunely/p/17464661.html#rte配置)
  - [RTE Generation阶段生成](https://www.cnblogs.com/fortunely/p/17464661.html#rte-generation阶段生成)
- AUTOSAR OS概念、配置方法
  - [AUTOSAR OS概念](https://www.cnblogs.com/fortunely/p/17464661.html#autosar-os概念)
  - [RTA-OS工程创建](https://www.cnblogs.com/fortunely/p/17464661.html#rta-os工程创建)
  - [AUTOSAR OS配置方法](https://www.cnblogs.com/fortunely/p/17464661.html#autosar-os配置方法)
  - [RTA-OS工程编译](https://www.cnblogs.com/fortunely/p/17464661.html#rta-os工程编译)
- [小结](https://www.cnblogs.com/fortunely/p/17464661.html#小结)



根据AUTOSAR方法论，完成了系统级SWC设计，还需配置目标ECU（ECU级设计）。该阶段主要针对运行时环境（RTE）、基础软件层（BSW）模块的配置。BSW包含很多模块，可根据实际需求选择配置。

根据示例需求，A、B车灯控制器所用BSW模块：

- 系统服务层中的操作系统（Operating Systme，OS）、基础软件模式管理器（Basic Software Mode Manager，BswM）、ECU状态管理器（ECU State Manager，EcuM）、通信管理模块（Communication Manager，ComM）；
- 通信服务中的通信模块（Communication，Com）、CAN状态管理模块（CAN State Manager，CanSM）、协议数据单元路由模块（PDU Router，PduR）；
- ECU抽象层中的I/O硬件抽象，通信硬件抽象中的CAN接口模块（CAN Interface，CanIf）；
- MCU抽象层中的MCU驱动、GPT驱动，通信驱动中的CAN驱动，I/O驱动中PORT驱动、DIO驱动、ADC驱动、PWM驱动、ICU驱动。

其中，I/O硬件抽象层作为AUTOSAR非标准模块，实现方法已介绍，这里不再赘述。

本文介绍ETAS针对AUTOSAR ECU级开发推出的工具、ECU级开发中除MCAL以外的模块概念与配置方法。

# ETAS RTA系列工具

ETAS RTA系列工具针对AUTOSAR ECU级设计与开发，主要包括RTA-BSW（AUTOSAR基础软件）、RTA-RTE（AUTOSAR运行时环境生成器）和RTA-OS（AUTOSAR实时操作系统）。

## RTA-BSW简介

RTA-BSW是一套高质量基础软件，能提供：
1）一套全面的AUTOSAR栈，包括通信、存储、诊断、标定、复杂驱动等；
2）为ECU应用开发提供全面的AUTOSAR R4.x平台，易于配置、集成、测试，支持将应用运行于真实ECU硬件/虚拟ECU平台（e.g. ISOLAR-EVE）；

RTA-BSW特点：
1）基于ISO 26262功能安全标准中ASIL-D的流程开发而成；
2）为ECU软件开发项目提供即用型解决方案；
3）除标准模块外，还包括对标准模块的功能扩展；
4）支持自动配置和代码生成，最大限度减少开发复合AUTOSAR规范的基础软件的工作量；
5）借助RTA-BSW自动测试功能，可在诸如ISOLAR-EVE等虚拟ECU平台上进行软件前期验证；
6）能与MCAL一起使用，从而构建一套完整BSW。

## RTA-RTE简介

RTA-RTE（AUTOSAR运行时环境生成器）可为符合AUTOSAR规范（R4.x、R3.x）的ECU软件提供运行时环境，提供配置生成运行时环境的多种选择：可检测arxml文件的正确性，以确保开发过程的高质量；可输出OS配置文件，以集成运行时环境和OS。

使用RTA-RTE优势：
1）通过ISO 26262（ASIL-D）认证；
2）生成的运行时环境独立于目标ECU和MISRA，C代码可在不同开发环境和ECU平台使用；
3）可优化运行时环境，精确匹配应用需求；
4）与各种类型的编译器、目标ECU硬件兼容，可在广泛的ECU平台上使用；
5）通过使用虚拟功能总线（VFB）进行跟踪，易于查错。

## RTA-OS简介

RTA-OS用于嵌入式系统的实时操作系统（RTOS），符合AUTOSAR、OSEK/VDX、ISO26262（ASIL-D）、MISRA C最新标准。

RTA-OS以ETAS RTA-OSEK RTOS开发经验为基础可用于单核与多核MCU，资源开销小。

RTA-OS可与RTA-RTE、RTA-BSW结合使用，开发出一个符合AUTOSAR规范的软件平台，但RTA-OS也可用于非AUTOSAR软件。基于ISOLAR-EVE虚拟ECU平台，即使在目标ECU硬件还不可用的情况下，也可以对OS进行虚拟验证。

------

# RTAS RTA系列工具入门

## RTA系列工具安装

安装过程简单，按提示即可。详细步骤略。

## RTA系列工具界面

ISOLAR-A中可添加BCT（ECU Conf+Code Generation）附件，如下图。该界面大致分3块：左边ECU Conf Navigator 展示已配置BSW模块；中间Outline可进行BSW模块的添加与新配置项的添加；右边界面通过表格等形式列出个模块的配置参数子项。

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607211730466-697065315.png)

RTA-OS工具界面如下图（布局类似于ISOLAR-A）：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607211738863-441300595.png)

------

# CAN协议栈概念与配置方法

## CAN协议栈概念

AUTOSAR通信栈位于运行时环境（RTE）与MCAL之间，可简化ECU之间的通信服务，实现不同类型或速率总线间的数据交互，如下图：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607211748860-178999038.png)

位于服务层（上图1）的有通信模块（Communication，Com）、诊断通信管理模块（Diagnostic Communication Manager，Dcm）协议数据单元路由模块（Protocol Data Unit Router，PduR）、协议数据单元复用模块（I-PDU Multiplexer，IpduM）、总线相关的传输模块（如CanTp、FrTp等）以及通信和网络管理相关的模块；

位于ECU抽象层（上图2）的是与总线相关的接口模块（如CanIf、LinIf等）；

位于MCAL（上图3）的是与总线相关的驱动模块（如Can、Lin等）；

**AUTOSAR通信栈的意义是什么？**
对应用层隐藏总线相关的协议和报文的属性。以CAN通信为例，发送过程描述如下：
1）Com模块获取应用层的信号（Signal），经一定处理封装为I-PDU（Interaction Layer Protocol Data Unit）发送到PduR模块；
2）PduR根据路由协议中所指定的I-PDU目标接收模块，将接收到的I-PDU经过一定处理后发送给CanIf；
3）CanIf将信号以L-PDU（Data Link Layer Protocol Data Unit）的形式发送给CAN驱动模块；

最终，实现基于CAN总线的基本数据发送，反之亦然；

## CAN通信协议栈配置方法

整车CAN信息可以在ISOLAR-A中通过DBC文件来获取。
ISOLAR-A主界面中，点击下图RTA-BSW Configuration Generation按钮，进行CAN协议栈中Com、PduR、CanIf、ComM、CanSM等模块的预配置。
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607211826259-1877876360.png)

点击RTA-BSW Configuration Generation按钮后，会弹出下图>选择待开发ECU的ECU Instance>OK：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607211833993-700549785.png)

完成预配置后，切换到BCT（ECU Conf+Code Generation）界面，可看到一些模块的配置信息，如下图。基于这些预配置模块，可以开始ECU级开发：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607211843556-34533671.png)

下面介绍CAN协议栈相关模块的配置：EcuC、Com、PduR、CanIf、ComM、CanSM。Can模块（驱动）属于MCAL，后面MCAL配置单独介绍，这里不介绍。

1）EcuC模块

数据在CAN协议栈各层间都是以PDU（Protocol Data Unit）形式传输，为将各层PDU关联，需要定义全局PDU（Global PDU）。由于全局PDU不属于任何标准BSW模块，所以AUTOSAR提出一个EcuC模块来收集一些配置信息。

在ECU Conf Navigator界面，右键EcuC“EcuC”>Open In Editor，如下图。
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607211852753-542199430.png)
Outline界面可看到EcuC模块具体配置：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607211859761-524864053.png)

在EcuC模块中定义全局PDU时，不需要关心其数据类型，只需要定义PDU长度即可：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607211907955-1873702699.png)

2）Com模块
Com模块位于运行时环境RTE与PduR模块之间，主要功能包括：
①将信号装载到I-PDU中发送，从接收到的I-PDU中解析出信号；
②提供信号路由功能，将接收到的I-PDU中的信号打包到发送I-PDU中；
③通信发送控制（启动/停止I-PDU组）；
④发送请求的应答等；

先前生成的Com模块配置如下图，主要有ComIPDus（I-PDU）、ComIPduGroups（I-PDU工作组）、ComSignals（信号）、ComTimeBases（时间基准）等。

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607211916047-834797521.png)

CompIPduGroups配置如下图，其中创建了2个I-PDU工作组，即ComIPduGroup_Tx和ComIPduGourp_RX，其Id分别为0、1；
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607211926790-1182043236.png)

其次，Com层中Signal是应用层通过Com模块发的基本单元，也是Com层内信息交互的基本单元，需要引用系统信号（System Signal）。I-PDU作为Com层与下层网络交互的基本单元，可由一个或多个Signal信号组成，各信号在Com模块中装载和解析。

每个ComSignal需要配置信号对初始值（ComSignalInitValue）、发送属性（ComTransferProperty）、数据类型（ComSignalType）、字节顺序（ComSignalEndianness）、字节大小（ComSignalLength）、系统信号引用（ComSystemTemplateSystemSignalRef）等。
对于发送属性，主要可分为OENDING、TRIGGERED、TRIGGERED_ON_CHANGE，特点：
①PENDING：写入信号值不能触发该信号相关的I-PDU的发送；
②TRIGGERED：根据信号相关的I-PDU的发送模式，写入该信号值可以触发该信号相关的I-PDU的发送；
③TRIGGERED_ON_CHANGE：根据信号相关的I-PDU的发送模式，当写入该信号值并且该信号值有变化时，才会触发该信号相关I-PDU的发送；

此外，为保证应用层发送到复杂类型数据（如结构体）的完整性，有时还可以定义信号组（Com Signal Group），免去数据过于复杂需要拆解而导致的完整性破坏。

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607211947787-413289363.png)

对于每个Com I-PDU需要设定I-PDU的传输方向（ComIPduDirection）、信号处理方式（ComIPduSignalProcessing）、类型（ComIPduType）、所属的I-PDU工作组（COmIPduGroupdRef）、Com信号引用（ComIPdurSignalRef）、全局PDU引用（ComPduIdRef）等。
最终配置：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607211956646-1942477279.png)

3）PduR模块
主要为通信接口模块、传输协议模块、诊断通信管理模块，通信模块提供基于I-PDU的路由服务。在通信协议栈中起承上启下的作用，为上层服务基础软件模块和应用屏蔽网络细节，使得上层模块和应用无需关心运行于哪种总线网络之上。

同时，PduR模块提供了基于I-PDU的网关功能，使得不同总线之间的通信成为可能。

PduR模块自动生成的配置：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212005846-158179080.png)

在PduR模块中，首先需要添加PduRBswModules，即添加所用到的通信协议栈中的相关模块，并勾选相关属性。示例用到了Com模块、CanIf模块、PduRBseModules配置如下：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212014043-1715027681.png)

需要定义路由路径（PduRRoutingPaths）。路由路径为源PDU（Source PDU）到目标PDU（Destination PDU）的描述，它们都需要通过引用前述EcuC中定义的全局PDU来关联：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212022662-580850323.png)

最终PduR模块的PduRRoutingPaths配置：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212030965-1833809765.png)

4）CanIf模块
CAN接口层（CanIf）是访问CAN总线的标准接口。CanIf抽象类CAN控制器的信息，并向上提供了一个平台无关接口（无需关心CAN Controller是片外还是片内设备）。

CanIf模块主要功能：完成对CanIf和控制器中全局变量及配置缓冲区的初始化：发送请求服务，为上层提供在CAN网络发送PDU的接口；发送确认服务，发送成功后通知上层，或发送取消后确认存于发送缓存；接收指示服务，成功接收PDU后通知上层。

CanIf模块主要配置为迎接对象句柄（Hardware object handle，Hoh），包括Hth（Hardware transmit handle）和Hrh（Hardware receive handle），它们需要引用Can模块中定义的CAN硬件对象（CanHardwareObject），CanHardwareObject是对CAN邮箱（MailBox，MB）的抽象。CanIf模块还需要配置CanIf层的PDU，每个PDU需要引用一个Hth或者Hrh，即完成PDU向MB的分配。CanIf模块主要配置如下：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212040014-1176765879.png)

Hrh、Hth配置：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212047116-2105313880.png)

CanIfRxPduCfgs配置：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212055452-1082189390.png)

5）ComM模块
通信管理模块（Communication Manager，ComM）可简化总线通信栈的初始化、网络管理等，并可收集/协调总线通信访问请求。主要提供3种通信模式：
①COMM_FULL_COMMUNICATION：FULL模式，此状态既能接收又能发送；
②COMM_SILENT_COMMUNICATION：SILENT模式，只能接收；
③COMM_NO_COMMUNICATION：NO通信模式，不能通信。

ComM模块的配置生成结果：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212104349-1302154105.png)

ComMChannel配置中可以配置ComMMainFunction()周期，默认10ms：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212111839-1809376514.png)

6）CanSM模块
CAN状态管理器（CAN State Manager，CanSM）负责实现CAN网络控制流程的抽象，为ComM模块提供API来请求CAN网络进行通信模式当切换，其配置生成：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212120348-174379555.png)

CanSMGeneral配置中，可配置CanSMMainFunction()周期，默认10ms：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212128094-134182917.png)

------

# EcuM模块概念与配置方法介绍

EcuM（ECU State Manager）模块属于系统服务层，它在系统服务层中具体位置：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212143607-1169198852.png)

EcuM模块负责初始化（Initialize）和反初始化（De-initialize）一些BSW模块。AUTOSAR ECU模式管理分为Fixed和Flexible两种方式。
Fixed有如下确定的模式：
1）STARTUP；
2）RUN；
3）POST_RUN；
4）SLEEP；
5）WAKE_SLEEP；
6）SHUTDOWN；

Flexible模式则允许其他的情况，如快速/分部（Fast/Partial）启动、多核管理（Muticore Manager）等。

另外，EcuM模块还可以配置ECU睡眠模式（Sleep Modes）、下电原因（Shutdown Causes）、复位模式（Reset Modes），管理所有ECU唤醒源（Wakeup Sources）。

在BCT界面中，可以完成EcuM模块的添加和配置：Outline界面，右键Ecu State Manager Module>Create 'EcuM' with mandatory containers with va 新建EcuM模块的配置：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212154517-1033355030.png)
最终EcuM模块配置：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212200690-490702669.png)

EcuM模块需要初始化一些BSW模块，所以需要定义一些列初始化列表。对于Fixed模式，可以定义4个初始化列表：
①初始化列表0（Driver Init List Zero）；
②初始化列表1；
③初始化列表2；
④初始化列表3。

其中，列表0、1在OS启动之前完成，而列表2、3则需要OS支持，故而在其启动之后完成。
对于Flexible模式，EcuM只需要完成列表0、1中各模块的初始化，列表2、3模块初始化需要由BswM模块来实现。

下面是一个推荐的模块初始化顺序猎豹：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212209635-1824964806.png)
①用于Fixed EcuM；
②用于Flexible EcuM；

这里用Flexible EcuM完成EcuM模块的配置

1）EcuMDriverInitListZero配置
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212217749-1743991721.png)

2）EcuMDriverInitListOne配置

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212223619-1009499223.png)

这老展示最终生成的初始化列表EcuMDriverInitListZero、EcuMDriverInitListOne的部分代码。其中，EcuM_AL_DriverInitOne函数中调用的各MCAL模块初始化函数将在MCAL配置部分讲解。

```c
FUNC (void, ECUM_CODE)EcuM_AL_DriverInitZero(void)
{
　　…
　　Det_Init();
　　…
}

FUNC (void, ECUM_CODE) EcuM_AL_DriverInitOne(const EcuM_ConfigType* ConfigPtr)
{
　　…
　　Mcu_Init(ConfigPtr->ModuleInitPtrPB.McuInitConfigPtr0_cpst);
　　McuFunc_InitializeClock();
　　Port_Init(ConfigPtr->ModuleInitPtrPB.PortInitConfigPtr0_cpst);
　　Dio_Init(ConfigPtr->ModuleInitPtrPB.DioInitConfigPtr0_cpst);
　　Gpt_Init(ConfigPtr->ModuleInitPtrPB.GptInitConfigPtr0_cpst);
　　Adc_Init(ConfigPtr->ModuleInitPtrPB.AdcInitConfigPtr0_cpst);
　　Can_Init(ConfigPtr->ModuleInitPtrPB.CanInitConfigPtr0_cpst);
　　Icu_Init(ConfigPtr->ModuleInitPtrPB.IcuInitConfigPtr0_cpst);
　　Pwm_Init(ConfigPtr->ModuleInitPtrPB.PwmInitConfigPtr0_cpst);
　　…
}
```

3）EcuMWakeupSources配置

在EcuMCommonConfiguration中需要配置EcuMWakeupModes，配置如下：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212415467-1481223487.png)

4）EcuMResetModes配置
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212422347-828068596.png)

5）EcuMShutdownCauses配置
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212429405-1273262532.png)

6）EcuMDefaultAppMode配置
点击EcmMCommonConfiguration>选择EcuMDefaultAppMode为OSDEFAULTAPPMODE：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212437665-1530533565.png)

7）EcuMGeneral配置

EcuMGeneral是EcuM模块整体功能的配置，配置EcuM_MainFunction()调用周期=10ms。并且，需要添加自定义头文件McuFunc.h：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212447798-1202192424.png)

------

# BswM模块概念与配置方法介绍

BswM（Basic Software Mode Manager）模块属于系统服务层，位置如下：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212457671-691975101.png)

BswM模块作用：
1）模式仲裁（Mode Arbitration）：根据SWC或其他BSW模块发来的模式请求（Mode Request）或者模式指示（Mode Indication），通过规则发起相应的模式切换。

2）模式控制（Mode Control）：通过执行动作列表（Action List）里面的动作实现模式切换。

模式仲裁和模式控制各司其职，模式仲裁决定是否要进行模式切换。如果是，那么模式控制就会执行（配置阶段预定义好的）动作列表。

模式仲裁基于规则（Rule）来进行的，每个规则中需要包含逻辑表达式（Logic Expression）和动作列表（Action List）。其中，逻辑表达式由一些列的模式请求条件（Mode Condition）通过逻辑运算符（AND/NAND/OR/XOR）组合起来；动作列表由一些列动作（Actions）组合而成。

BswM模块相关概念如下图；
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212507733-996276416.png)
BswM模块工作过程示意图：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212514246-1637294.png)

BswM实现过程中，会在如下三种情况进行模式仲裁：
1）SWC发出模式切换请求——SWC通过调用BswM模块交互的接口函数，BswM模式仲裁机制会根据当前已有的请求，以及当前请求的类型选择立即仲裁（IMMEDIATE）或者推后仲裁（DEFERRED）；

2）EcuM、WdgM等BSW模块的模式标识发生改变，进而向BswM模块发出一个模式仲裁请求；

3）通过周期性地调用BswM_MainFunction()，在BswM主函数中进行周期性的模式仲裁，该方式一般用来处理推后仲裁类型的请求。

模式控制最终要执行动作列表中的动作，动作可以是调用其他BSW模块的服务，或调用RTE，执行其他的动作列表或引起另外的规则仲裁。

另外的规则仲裁：
1）条件执行（CONDITION）：每次进行规则检查时，都会根据设定的期望结果来执行动作列表；
2）触发执行（TRIGGER）：仅当当前规则检查结果和上次规则检查结果不同时，才会执行动作列表；

为简化BswM模块的设计，AUTOSAR预定义一些标准动作（Standard Actions）：
1）ComM：设置一个通信接口的通信模式；
2）ComM：限制通信模式；
3）ComM：使能一个ComM通道的通信；
4）LinSM：设置LIN调度表（Schedule Tables）；
5）FlexRay：切换到“All Slot Mode”；
6）Com：激活和停用I-PDU组（I-PDU Group），重设或不重设信号初始值；
7）Com：使能或禁用截止时间超时监控（Deadline Timeout Monitoring）；
8）Com：触发I-PDU发送；
9）EcuM：设置ECU运行模式（ECU Operation Mode）；
10）EcuM：关闭所有运行请求（Run Requests）；
11）Network Management：使能或禁用网络管理通信（NM Communication）；
12）PduR：使能或禁用PDU路由路径（PDU Routing Path）；
13）RTE、BSW：调度器的模式切换；

示例BswM模块主要完成初始化工作，接收应用层EcuBaseSWC的启动/关闭CAN请求。配置总览如下：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212546404-1241213924.png)

BSW模块的初始化工作，主要基于EcuM等BSW模块的模式标识改变而向BswM模块发出模式仲裁请求，发生在BSW模块间。下面介绍应用层EcuBaseSWC发起启动/关闭CAN请求，感受BswM模块各主要概念、运行机制。

1）BswMModeRequestPort配置
首先，建立一个与应用层SWC交互的服务端口——添加BswMModeRequestPort：点击“+”Add，名为BswM_MRP_ApplicationRequestPort，采用推后仲裁DEFERRED模式：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212556661-292510522.png)

其次，需要为端口定义属性。该端口与应用层SWC交互，因此新建BswMGenericRequest，请求类型RequestType为SWC，共有2种模式（启动/关闭CAN）——配置BswMRequestedModeMax为2：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212605110-589370106.png)

3）Action与ActionList配置

在定义ActionList时，需要定义Action。定义2个：BswM_AI_AppReqFullCom、BswM_AI_AppReqNoCom，分别使用ComM模块预定义的标准动作，请求COMM_NO_COMMUNICATION和COMM_FULL_COMMUNICATION，配置如下：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212614051-482992722.png)
定义2个ActionList：BswM_AL_AppReqFullCom，都配置成TRIGGER模式，且分别引用引用BswM_AI_AppReqFullCom和BswM_AI_AppReqNoCom。Action List配置成下图：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212627207-317466130.png)

4）Rule配置
定义Mode Condition、Logical Expression、Action、Action List后，就可以定义Rule。这里新建一个名为BswM_AR_AppReqFullNoCom的Rule，配置如下：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212635157-441618822.png)

5）BswMGeneral配置

BswMGeneral配置主要是对BswM模块属性的配置，配置BswM_MainFunction()周期10ms：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607212646008-62035513.png)

下面展示最终BswM模块生成的部分代码，便于理解模式仲裁和模式控制的实现过程。规则检查函数BswM_Rule_BswM_AR_AppReqFullNoCom，以通过该规则检查为例进行分析：若规则检查结果True，则调用BswM_ActionList_BswM_AI_AppReqFullCom()，即触发相应ActionList，而ActionList函数中将调用Action函数BswM_Action_BswM_AI_AppReqFullCom。

```c
FUNC (void, BSWM_CODE) BswM_Rule_BswM_AR_AppReqFullNoCom(void)
{
　　if(BSWMLOGEXP_BSWM_LE_APPREQFULLCOM)
　　{
　　　　/* True Action list */
　　　　/* Triggered */
　　　　if(BswM_Prv_RuleState［6］!= BSWM_TRUE)
　　　　{
　　　　　/* Make a call to the corresponding action list item-BswM_AL_AppReqFullCom */
　　　　　BswM_ActionList_BswM_AL_AppReqFullCom();
　　　}
　　　/* Change the state of the rule-BswM_AR_AppReqFullNoCom */
  　BswM_Prv_RuleState［6］= BSWM_TRUE;
　　}
　　else
　　{
　　　　/* False Action list */
　　　　/* Triggered */
　　　　if（BswM_Prv_RuleState［6］!= BSWM_FALSE）
　　　　{
　　　　　　/* Make a call to the corresponding action list item-BswM_AL_AppReqNoCom */
　　　　　　BswM_ActionList_BswM_AL_AppReqNoCom();
　　　　}
　　　　/* Change the state of the rule-BswM_AR_AppReqFullNoCom */
　　　　BswM_Prv_RuleState[6] = BSWM_FALSE;
　　}
}

FUNC (void, BSWM_CODE) BswM_ActionList_BswM_AL_AppReqFullCom(void)
{
　　VAR(Std_ReturnType, AUTOMATIC) action_RetVal_u8;

　　/* Invoke all the available actions in the assending order of ActionListItem index */
　　/* Execute the Available Action-BswM_AI_AppReqFullCom */
　　BswM_Action_BswM_AI_AppReqFullCom(&action_RetVal_u8);
　　action_RetVal_u8 = BSWM_NO_RET_VALUE; /* Avoid variable un-used compiler warning when only Rule or Action list is configured as action */
}

FUNC(void, BSWM_CODE) BswM_Action_BswM_AI_AppReqFullCom(P2VAR(Std_ReturnType, AUTOMATIC, BSWM_APPL_DATA) action_RetVal_pu8)
{
　　/* Initialize to"no return value" */
　　*action_RetVal_pu8 = BSWM_NO_RET_VALUE;
　　/* Switch the communication mode for a ComM User-ComMUser_Can_Cluster_Channel */
　　*action_RetVal_pu8=ComM_RequestComMode(0, COMM_FULL_COMMUNICATION);
}
```

------

# BSW模块代码生成

BCT界面配置完所需BSW模块后，可进行BSW模块代码、描述文件但生成：点击ISOLAR-A主菜单“→”右边箭头>选择Run Configurations：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213057904-1756919883.png)

弹出Run Configurations界面：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213106377-2110198021.png)

在BSWGen Lanch Configuration菜单下新建名为LightECU_BSW的配置，即可进入BSW生成相关配置界面。其中，需要选择RTA-BSW工具版本、工程以及BSW代码生成路径，勾选需要生成的模块>点击Run，生成BSW模块。

Console显示生成情况信息：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213115848-1570563318.png)

# 服务SWC、应用层SWC端口连接

切换到ISOLAR-A系统级设计界面，会发现产生一些基础软件模块的SWC（BswM，ComM，Det，EcuM，etc.）：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213124123-989950000.png)

如果涉及服务SWC与应用层SWC的交互，就需要为应用层SWC添加相应的服务端口。在配置BswM模块配置阶段，已经设计了相应的BswMModeRequestPort，名为BswM_MRP_ApplicationRequestPort，打开BswM SWC可看到端口信息：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213135587-1748448914.png)

按需求，该端口是与应用层EcuBaseSWC交互的，所以按基于ISOLAR-A SWC设计方法中提到的方法为EcuBaseSWC添加一个Port，其Port Interface为BswMSwcGenericRequest_ClientServerInterface：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213142773-396928732.png)

完成相关应用层SWC的端口添加后，再次按前述系统级设计与配置方法，将这些新生成的SWC都添加到VehicleComposition，完成Assembly Connectors添加，并基于VehicleSystem完成SWC To ECU Mapping。最后，再次进行ECU Extract，此时LightECU_FlatView Composition如下图：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213155789-871792405.png)
最终，生成EcuBaseSWC模板，并在其中调用它和BswM SWC的模式请求RTE接口函数，编写相应模式切换逻辑即可。

------

# RTE配置、代码生成

RTE生成器工作在2个阶段：合同阶段（Contract Phase）、生成阶段（Generate Phase）。

## RTE Contract阶段生成

合同阶段：输入文件是SWC描述文件，输出.h文件，主要包括类型定义和接口定义信息。

合同阶段生成示意：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213217747-711761396.png)

点击ISOLAR-A主界面“R”RTE contract phase...，如下图：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213225549-1178275284.png)

弹出合同阶段生成界面：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213232870-1064268395.png)

选择RTE工具的路径、文件生成路径，指定Additional Commands>Finish，完成RTE合同阶段生成。

## RTE配置

生成RTE前需要配置RTE：1）ECU级模块配置信息的收集（Ecuc Value Collection）；2）各运行实体到OS任务的映射（RE To Task Mapping）。

1）ECU级模块配置信息收集

RTE是应用层与基础软件层交互的桥梁，因此生成RTE前需要收集所有ECU级模块的配置信息（Ecuc Value Collection）：右键Bsw文件夹>Create BSW Module Descriptions>Create Ecuc Value Collections>Elements ：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213246003-1542683298.png)

弹出下图，用于创建AR Element：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213252564-1813600550.png)

双击LightECU_EcucValueCollection，弹出下面界面，选择一个ECU Extract（选择EXTR_LightECU）：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213303643-1598597813.png)

新建Ecuc Value Collection后，就可以向其中添加各模块的配置信息。右键LightECU_EcucValueCollection>New Child>Ecuc Values | Ecuc Module，可新建一个引用，选择需要引用的模块配置信息即可：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213310727-921496124.png)

由于RTE和OS两个模块的配置在BSW模块配置阶段还未添加，双击LightECU_EcucValueCollection，切换至RTE Configuration界面>点击Generate即可创建Rte模块；切换至OS Task Properties>点击Create OSAppMode即可创建Os模块。

最终，示例ECU级模块配置信息收集结果：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213320062-1591817066.png)

2）运行实体到OS任务映射

由于运行实体是用户程序的最小划分，而OS任务是运行实体的载体。换言之，OS无法直接调度运行实体，而是需要通过将运行实体映射到不同的任务中，通过对任务进行调度进而实现各个特定的时刻执行特定的运行实体。因此，需要完成运行实体到OS任务的映射（RE To Task Mapping）。

这里需要映射到运行实体 包括应用层SWC + 基础软件模块。运行实体到OS任务的映射关系设计如下（共7个任务）：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213328122-380786549.png)

根据上表，可完成运行实体到OS任务的映射工作。双击LightECU_EcucValueCollection，切换至Entity To Task Mapping界面：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213335791-1738076671.png)

在创建上述所有Task后，切换到Os Task Priorities界面可看到所有Task，可以为其配置相关属性：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213344741-214088952.png)

## RTE Generation阶段生成

在完成了RTE配置后，就可以进行RTE生成阶段（Generation Phase）的生成，该阶段将主要生成如下文件：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213354539-32673586.png)

①Rte_Type.h：通用Type定义；
②Rte_.h：应用API声明；
③Rte__Type.h：SWC特定的Type定义；
④Rte.c：API定义、数据结构定义；
⑤Rte_Lib.c：静态RTE库函数；
⑥.c：Task主体函数；
⑦osNeeds.arxml：OS配置描述；

由于RTE是应用层软件和基础软件的桥梁，所以一般RTE Generation阶段在ECU级开发末期生成，即此时已完成了基本所有的SWC与BSW模块的设计。
点击ISOLAR-A主界面“R”Generate RTE code in Generate phase：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213402561-2029110095.png)

将弹出：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213411570-1766584433.png)

选择RTA-RTE工具的路径、文件生成路径，并选择一个ECU Instance，加入Additional Commands>Finsh，完成RTE Generation阶段生成。

由于RTA-RTE在RTE Generation阶段将生成部分OS的配置，它们可以导入RTA-OS工具，简化OS的配置过程，所以在OS配置完成前先完成RTE Generation阶段生成。

下面展示RTE Generation阶段生成的一些代码。

首先展示LightControlSWC向外发送车灯当前状态的RTE接口实现，

```c
FUNC (Std_ReturnType, RTE_CODE)
Rte_ImplWrite_LightControlSWC_PPortSendLightState_DESendLightState(VAR(UInt8, AUTOMATIC)data)/* 1 */
{
　VAR（Std_ReturnType，AUTOMATIC）rtn=RTE_E_OK；
　/* SpecReq：Send signal begin */
　/* The signal is LightState */
　if(((VAR（StatusType，AUTOMATIC))E_OK) != Com_SendSignal(((VAR(Com_SignalIdType，AUTOMATIC))1), &data))
　{
　　　rtn = ((VAR(Std_ReturnType，AUTOMATIC))RTE_E_COM_STOPPED)；
　}
　/* SpecReq：Send signal end */
　/* Send complete */
　return rtn；
}
```

其次展现的是Client/Server模式的具体实现。示例采用同步模式，所以当Client端调用接口Rte_Call_LightControlSWC_RPortGetLightState_OPGetLightState时，即调用Server端名为RE_GetLigthState()：

```c
FUNC (Std_ReturnType, RTE_CODE)
Rte_Call_LightControlSWC_RPortGetLightState_OPGetLightState(CONSTP2VAR(UInt8, AUTOMATIC, RTE_APPL_DATA)DEGetLightState) /* 1 */
{
　VAR (Std_ReturnType, AUTOMATIC) rtn;
　/* SpecReq：Activate RE via Queue begin */
　/* Parameter DEGetLightState has direction OUT */
　RE_GetLightState(DEGetLightState);
　rtn= ((VAR(Std_ReturnType, AUTOMATIC))RTE_E_OK);
　/* SpecReq：Activate RE via Queue end */
　return rtn;
}
```

最后，Task主体函数如下，在名为OsTask_BSW_1ms的Task主体含税中调用了先前映射到它上面的三个运行实体函数：

```c
TASK（OsTask_BSW_1ms）
{
　/* Box：Implicit Buffer Initialization begin */
　/* Box：Implicit Buffer Initialization end */
　　　/* Box：Implicit Buffer Fill begin */
　/* Box：Implicit Buffer Fill end */
　{
　　/* Box：BSWImpl6_BSWIMPL_Can begin */
　　Can_MainFunction_Read();
　　/* Box：BSWImpl6_BSWIMPL_Can end */
　}
　{
　　/* Box：BSWImpl6_BSWIMPL_Can begin */
　　Can_MainFunction_Write();
　　/* Box：BSWImpl6_BSWIMPL_Can end */
　}
　{
　　/* Box：CPT_EcuAliveIndicatorSWC begin */
　　SetEcuAlive1();
　　/* Box：CPT_EcuAliveIndicatorSWC end */
　}
　/* Box：Implicit Buffer Flush begin */
　/* Box：Implicit Buffer Flush end */
　TerminateTask();
} /* OsTask_BSW_1ms */
```

------

# AUTOSAR OS概念、配置方法

## AUTOSAR OS概念

AUTOSAR OS（Operating System, OS）源于OSEK/VDX OS，属于系统服务层，它是一种多任务的实时OS（RTOS），并且是静态OS，即不可以在运行时动态创建任务。RTOS对实时性要求较高，需要保证在特定的时间内处理完相应的事件或数据。

1）OS的任务（Task）及状态

OS的任务（Task）主要可分为用户任务、系统任务。用户任务需要根据不同应用场景进行自定义。用户任务的划分及其优先级的选取，是OS配置的重点，这影响着程序的执行效率和结果。而系统一般有空闲任务（Idle Task），它的优先级最低，并且执行时间是衡量CPU负载率的重要参数。所以，OS的任务设计一般指用户任务的设计，AUTOSAR OS定义了两种用户任务：
①基本任务（Basic Task）；
②扩展任务（Extended Task）；
其中，基本的任务状态包括运行状态（Running）、就绪状态（Ready）和挂起状态（Suspended），任务切换只发生在这3种状态之间；而扩展任务除了具有基本任务的3种状态，还有等待状态（Waiting），如下图：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213504102-835174927.png)

对于上面提及的任务状态，分别有如下特征：
①运行状态（Running）：处于运行状态，处理器资源被分配给该任务，该任务的指令被执行。在同一个处理器上，任何时候只有一个任务处于运行状态，而处于其他状态的任务则可以有多个。

②就绪状态（Ready）：处于就绪状态是任务转换到运行状态的前提，此时任务等待处理器的资源分配，由调度器来决定哪个就绪任务将被执行。

③挂起状态（Suspended）：处于挂起状态时，任务是被动的，可以被激活。

④等待状态（Waiting）：任务因等待一个或多个事件而无法继续执行。

基本任务只有在以下情况下，才会释放处理器资源：
①情况1：该任务运行结束时；
②情况2：OS切换到更高优先级的任务时；
③情况3：发生了一个中断，处理器切换到该中断对应的中断服务程序。

基本任务代码：

```c
#include“OS.h”
TASK (BasicTask)
{
　　...
　　/* User code */
　　...
　　TerminateTask();
}
```

扩展任务在运行状态通过调用WaitEvent()切换为等待状态，直到所等待的事情发生。扩展任务在等待状态下会释放处理器资源，OS会执行处于就绪状态且任务优先级最高的任务，而不需要终止该扩展任务。
所以，扩展任务比基本任务更复杂，占用更多系统资源。

扩展任务代码：

```c
#include "OS.h"
TASK (ExtendedTask)
{
　　for(; ;)
　　{
　　　　WaitEvent(Event1);
　　　　/* perform actions */
　　　　ClearEvent(Event1);
　　}
}
```

2）任务的调度策略（Scheduling Policy）

AUTOSAR OS任务调度基于优先级（Priority），每个任务根据其特性定义一个优先级，配置可抢占性，可抢占属性分为非抢占与全抢占。
OS提供3种调度策略（Scheduling Policy）：
①非抢占式（Non-preemptive）：所有任务都被定义为不可抢占的。
②完全抢占（Preemptive）：所有任务都被定义为可抢占的。
③混合抢占：有的任务被定义为可抢占的，有的被定义为不可抢占的。

若采用全抢占式任务调度策略，运行的任务在任何时刻都有可能由于更高优先级任务而被迫释放处理器，最高优先级的就绪任务将会被调度，低优先级任务从运行太切换到就绪态，并保存当前运行环境，待下次继续运行时恢复；

若采用非抢占式任务调度策略，任务执行期间不会被高优先级任务抢占，任务的切换只发生在当前任务完成时。最大缺点：任务响应时间不确定，导致系统实时性较差。

若采用混合式任务调度策略，则调度策略取决于当前任务的可抢占属性。

3）计数器（Counter）与报警器（Alarm）

OS提供处理重复事件的服务，基于计数器、报警器实现。
计数器：反复出现的事件，由特定的计数器来记录。
报警器：在计数器的基础上，OS向应用软件提供了报警机制。多个报警器可以连接到同一个计数器。当达到报警器相对应的计时器设定值时，可激活一个任务、设置一个事件或调用一个回调函数。

4）调度表（Schedule Table）

一个计数器和一个基于该计数器的报警器队列，可以实现静态定义的任务激活机制：当计数器的值 >= 报警设定值时，报警器被触发。但这样存在一个缺点：很难保证个报警器之间具有特定的时间间隔，且由于每个报警器只能激活一个任务或者设置一个事件，所以需要定义多个报警器来实现在同一时刻激活多个任务或设置多个事件。

为解决该文件，OS引入调度表（Schedule Table）的概念。
调度表中可定义一系列终结点（Enpiry Point）。每个调度表都有一个以Tick为单位的持续时间（Duration）。其中，每个终结点都有一个以Tick为单位的距离调度表起始点的偏移量（Offset）。在每个终结点可以进行一个或多个激活任务或设置事件的操作。

类似于报警器，一个调度表由一个计数器驱动。调度表有2种运行模式：
①单次执行（Single shot）：调度表启动后只运行一次，并在调度表的终点自动停止。每个终点只处理一次。
②重复执行（Repeating）：调度表启动后可重复运行，即当到达调度表终点时，又再回到起点重复运行。此时，每个终结点将以调度表的持续时间为周期，周期性地被处理。

5）中断（Interrupt）处理

AUTOSAR定义2类中断服务程序（Interrupt Service Routinge，ISR）：
①一类中断（Category 1 Interrupt）：不能使用OS的服务，中断服务程序结束后，处理程序将从产生中断的地方继续执行。这类中断不影响任务的管理，占用系统资源较少。

②二类中断（Category 2 Interrupt）：可以使用一部分OS提供的服务，如激活任务、设置事件等。

OS中，任务优先级 < 中断优先级，即最低优先级的中断可以打断最高优先级的任务。所以，中断服务程序的执行时间不易太长，以免影响重要任务的执行，降低OS实时性。

6）资源管理

资源管理被用来协调有着不同优先级的多个任务对共享资源（如内存或硬件等）的并发访问（Concurrent Access）。

OS采用优先级置顶协议（Priority Ceiling Protocol）来避免优先级倒置（Priority Inversion）和死锁（Deadlock）问题。在系统初始化阶段，每个资源拥有的上限优先级是静态分配的，资源的上限优先级必须高于所有要访问该资源的任务和中断的最高优先级，但是低于不访问该资源的任务的最低优先级。

优先级倒置问题，参见[RTOS 优先级倒置](https://www.cnblogs.com/fortunely/p/17457230.html)。

如果一个任务要访问一个资源，且该任务的优先级比该资源的优先级上限低，则将该任务的优先级提升到要访问的资源的上限优先级；当该任务释放资源后，其优先级再回到要求访问该资源前的优先级。

7）自旋锁（Spin Lock）

自旋锁是一种保护共享资源的锁机制，一般用于多核处理器。当内核控制路径必须访问共享数据结构或进入临界区时，如果自旋锁已被其他执行单元保持，那么调用者就一直循环等待锁被释放（等待时不会主动释放CPU资源）。

8）一致类（Conformance Class）

OS支持4种一致类，开发者根据实际需求灵活配置OS的调度程序。划分方法：根据每个优先级可能具有的任务个数、需要的是基本任务还是扩展任务等来决定。大类可分为：基础一致类（Basic Conformance Class，BCC）、扩展一致类（Extended Conformance Class，ECC）。每个大类又可分为2小类：
①BCC1：每个优先级只有一个任务，基本任务的激活数只能为1次，仅支持基本任务；
②BCC2：每个优先级可有多个任务，基本任务的激活数可为多次，仅支持基本任务；
③ECC1：每个优先级只有一个任务，基本任务的激活数只能为1次，支持基本任务和扩展任务；
④ECC2：每个优先级可有多个任务，基本任务的激活数可为多次，支持基本任务和扩展任务。

9）可扩展性等级（Scalability Class）
为迎合不同OS功能的不同需求，OS根据可扩展性等级（Scalability Class，SC）分为4类：
①SC1：在OSEK OS基础上加入调度表（Schedule Table）；
②SC2：在SC1基础上加入时间保护（Timing Protection）；
③SC3：在SC1基础上加入存储保护（Memory Protection）；
④SC4：在SC1基础上加入时间保护和存储保护；

## RTA-OS工程创建

OS配置后，需要新建一个RTA-OS工程：点击RTA-OS主菜单File>New Project：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213545007-1983583319.png)

将弹出：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213603357-679464170.png)

配置过程本质是对配置描述文件修改的过程，所以新建工程阶段需要完成XML Settings，即对其中一些元素进行命名，依次为AR Package Name、ECU Configuration Name、OS Configuration Name、Release。输入相应名字 > OK。

RTA-OS主界面OS Configuration界面将显示新工程的所有配置项：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213612716-728296473.png)

下面先对OS进行整体性配置，即General配置，部分配置如下图（SC1等级OS，使能Startup、Shutdown、Error钩子函数，便于debug）：
![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213620706-390301153.png)

运行目标芯片设为MPC5744P：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213636456-161308441.png)

## AUTOSAR OS配置方法

下面介绍基于RTE的OS配置方法：

1）描述文件导入
切换到Project Files菜单>右键OS工程文件LightECU.rtaos>Add Existing File，如下图。选择添加OsNeeds.arxml（RTE生成）、OsCfg.arxml（RTE配置阶段创建，包含与OS相关的配置信息）。

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213644474-1938697024.png)

上面2个描述文件导入后，如下图，其中粗体为当前正在被修改的描述文件：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213653271-1356640319.png)

再切换到OS Configuration界面，将看到大部分与用户任务相关的配置信息已导入：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213701101-1399326496.png)

2）Counter配置

由于在ISOLAR-A中进行RTE配置时，未定义OS Counter具体实现相关的属性，所以需要添加。示例将Counter计数基准配置为1ms，即将Rte_TickCounter中的Seconds Per Tick设置为0.001，并将Ticks Per Base设置为1：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213710210-1209490501.png)

3）ISR配置

ISR与具体实现密切相关，且与MCU相关，需要另行配置。
右键ISRs>New新建一个中断服务函数，名字即为中断服务函数名。
这里以产生Os Tick的通用定时器（GPT）中断配置为例，进行说明：新建ISR名为Gpt_STM_0_Ch_0_ISR，将其配置为二类中断（CATEGORY_2），定义一个优先级（Priority），并需要选择一个中断向量号（Address/Vector），采用STM_0_CH_0来实现GPT，所以选择System Timer 0 Channel 0（INTC_36）即可。

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213719488-276282421.png)

除了产生OS Tick的GPT中断，根据MCAL中一些模块以及硬件通道的选择，还需要完成一些ISR配置。
对于A车灯，还需要配置ADC的ADC_EOC中断，其中断函数名Adc_Adcdig_EndGroupConvUnit0；
对于B车灯，还需要配置输入捕获（ICU）相关的中断，其中断函数名为ETIMER_2_CH_4_ISR，配置方法与上述GPT中断配置方法一致。

4）Schedule Table配置
示例用一个Schedule Table实现各Task调度，自动生成的配置如下图。驱动计数器为Rte_TickCounter，定义成重复模式，且持续时间20ms。Schedule Table一共定义20个终结点，在每个终结点定义若干操作。

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213727366-40962863.png)

5）Task配置
在导入的Task配置信息的基础上，还需要新建一个初始化任务，具有最高优先级，且只运行一次。OS启动后，首先被调用，完成一些初始化工作，如调用EcuM_StartupTwo()完成BSW模块初始化。

右键Tasks>New 新建一个Task，命名为ECU_StartupTask。在General配置菜单中可配置该Task的基本属性：
这里将该Task优先级配置为所有Task最高（50），数值越高，优先级越高；
激活方式（Activation）配置为1，表示该任务在任何时候只允许激活一次；
可抢占属性（Task Preemptability）配置为NON，即该Task配置为非抢占式。

> 切换到Application Modes界面，添加一个Application Mode OSDEFAULTAPPMODE，即将ECU_StartupTask配置为在OSDEFAULTAPPMODE模式下自启动的Task，即在OS启动后自动启动该Task。
> ECU_StartupTask配置：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213735586-1262803214.png)

最终OS Task配置：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213743976-562603238.png)

## RTA-OS工程编译

RTA-OS工具可以直接调用编译器对OS相关代码进行编译。在完成OS所有配置后，可以切换到Builder>Setup界面，对生成文件的路径、包含的头文件进行设置后，工具会自动生成Build脚本：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213754079-1765721598.png)

配置完后，切换到Builder>Build界面>点击Build Now，开始OS工程编译（过程RTA-OS将调用编译器完成OS相关代码编译）：

![img](AUTOSAR%E7%AC%94%E8%AE%B0%EF%BC%9AECU%E7%BA%A7%E5%BC%80%E5%8F%91_RTE%E3%80%81BSW.assets/741401-20230607213804821-156714335.png)

------

# 小结

1）介绍ETAS针对AUTOSAR运行时和基础软件层开发工具RTA-RTE、RTA-BSW、RTA-OS的基础上，对ECU级开发中除MCAL以外部分进行全面的讲解。
2）从各模块概念着手，结合工具配置方法和一些生成的代码进行剖析。