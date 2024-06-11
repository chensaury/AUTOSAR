# CDD复杂驱动

**Linux驱动是真正的驱动，复杂驱动只是在autosar范畴的说法，内容不一定是驱动，可能同时包含底层、算法、应用层等其他东西” 简而言之，没办法统一标准的东西，都放在复杂驱动了；例如功能安全类似的；**



**AutoSar官方推荐的CDD的构建思路是：使用标准函数构建基础的框架，例如ADC实现基本的初始化、开启通道转换、获取通道值等功能；然后在框架中添加组件，撰写算法实现更多需要的功能，例如硬解码、软解码、电压电流监测等等功能；**

******CDD是啥？**

首先看字面意思，CDD(Complex Device Driver or Complex Driver)是复杂设备驱动/复杂驱动的缩写，**但是它不仅限用于驱动模块**，也可以是应用或者与芯片、ECU相关的其他模块。

一个CDD模块是AUTOSAR里面的一个软件实例，属于AUTOSAR的一部分，但是该模块没有被AUTOSAR标准化。它可以通过AUTOSAR接口访问AUTOSAR里面其他基础模块（BSW），同样也可以被其他BSW或者RTE访问，SWCs与CDD之间不能直接进行访问，但可通过RTE进行。

CDD虽然是没有被AUTOSAR标准化的模块，但其可看作AUTOSAR软件[架构](https://so.csdn.net/so/search?q=架构&spm=1001.2101.3001.7020)里面的一种机制或者一种理念。

**为啥有CDD?**

CDD的主要目标是实现复杂的传感器采集和/或执行器控制，使用特定的中断和/或复杂的微控制器外设，外部设备（通信收发器，ASIC …）直接访问微控制器，以满足特殊的功能和时序要求。

## 一、Introduction to CDD

CDD：
CDD是复杂设备驱动的首字母缩写
主要为驱动程序或复杂驱动程序，但不限于驱动程序。

复杂驱动程序是一种未被[AUTOSAR](https://so.csdn.net/so/search?q=AUTOSAR&spm=1001.2101.3001.7020)标准化的软件实体，可以通过AUTOSAR接口和/或基本软件模块api访问。

 CDD是位于基础软件复杂驱动层的特定模块，与标准BSW模块或Rte交互。

 CDD可能需要与分层软件体系结构的模块连接

 分层软件架构的模块可能需要连接到CDD

 CDD可能需要通过Rte连接swc

![在这里插入图片描述](https://img-blog.csdnimg.cn/0a764d77640f44b9bcb3a1929b6a8fbe.png#pic_center)
CDD的主要目标是通过使用特定的中断和/或复杂的微控制器外围设备、外部设备(通信收发器)直接访问微控制器来实现复杂的传感器评估和执行器控制。ASIC…)以满足特殊的功能和时序要求。

此外，它还可以用于实现增强的服务/协议或封装非autosar系统的遗留功能。

CDD的实现可能取决于应用程序、µC和ECU。

最后，CDD可以作为将现有概念或新概念引入AUTOSAR软件体系结构的迁移机制。

某些语气词使用的约定：

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL
NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and
“OPTIONAL” in this document are to be interpreted as:

***SHALL***: 这个词意味着这个句子是AUTOSAR规范的绝对要求。因此，设计者应尊重原始AUTOSAR规范的要求。（意思必须按AutoSar要求来）

***SHALL NOT***: 这个词表示这句话绝对禁止使用AUTOSAR规范。因此，设计者应尊重原始AUTOSAR规范的要求。（意思必须按AutoSar要求来）

***MUST***: 这个词的意思是由于法律问题，这句话是AUTOSAR规范的绝对要求。因此，设计者应尊重原始AUTOSAR规范的要求。（意思必须按AutoSar要求来）

***MUST NOT***: 这句话的意思是，由于法律的限制，这句话是绝对禁止的规范。因此，设计者应尊重原始AUTOSAR规范的要求。（意思必须按AutoSar要求来）

***SHOULD***: 这个词或形容词是“推荐”的意思，在特定情况下，可能存在忽视某一特定项目的正当理由，但在选择不同的路线之前，必须了解并仔细权衡其全部影响。（意思推荐你这么做，不这么做也行，出了错自己负责）

***SHOULD NOT***: 这个短语或短语是“不推荐”的意思，在特定的情况下，当特定的行为是可以接受的，甚至是有用的，可能存在有效的理由，但在实施任何带有这个标签的行为之前，应该理解全部的影响，并仔细权衡情况。（意思推荐你这么做，不这么做也行，出了错自己负责）

***MAY***: 这个词，或者形容词“OPTIONAL”，意思是一个项目是真正可选的。一个供应商可能会选择包含该项目，因为一个特定的市场需要它，或者因为供应商觉得它可以增强产品，而另一个供应商可能会省略相同的项目。一个不包含特定选项的实现必须准备好与另一个包含该选项的实现互操作，尽管功能可能会减少。以同样的方式，一个包含特定选项的实现，
必须准备好与另一个不包含该选项的实现进行互操作(当然，选项提供的特性除外)。
(意思这里你有很多选择，不做强制要求。自己了解这么选择会产生哪些影响就好了)

## 二、CDD设计建议(CDD开发需要注意的事项）

为了在AUTOSAR体系结构中接口和简化CDD集成，设计人员应考虑以下几点

### 2.1、文档

**用户手册**（写完CDD应该写哪些文档让别人理解）

CDD设计者应提供用户手册以简化集成并向客户提供信息:

 CDD介绍和概述
 功能操作描述(初始化、正常、关机、故障操作……)
 描述与其他BSW模块、SchM和Rte的关系和需求;例如，NvM的内存块，需要配置的临界区。
 文件结构和依赖关系
 接口(包括服务)的描述:名称、描述、可重入性、参数(名称、类型、范围、值)、返回值(名称、类型、范围、值)、配置类。
 非功能需求描述:时间和行为需求、资源使用、与其他BSW模块或SWC的行为……
 描述Dem错误，可选Det错误，调试变量
 配置参数(名称、类型、范围、值)的描述。
 —内存映射需求描述(Flash、RAM)
 使用限制和开放问题
 与其他模块的集成约束和要求
 例子

**执行**（应该遵守的一些要求）
对于CDD实现，AUTOSAR几乎没有什么限制。至少:
 CDD应尊重输入规格[3]，[4]，[5]，[6]，[7]，[9]，[10]，[11]
 CDD应保护其关键资源，定义可由SchM或OS机制处理的关键部分。
 CDD模式可由EcuM和BswM模块管理。
 CDD可以使用内存映射机制来处理它的内存段。
 CDD可以使用Det或Dem模块报告错误。

**CDD Files** （建议你这样设计文件结构）

本节只是一个建议，并没有完全定义模块文件的结构。
Code file(s) ：
除了[4]AUTOSAR基本软件模块通用要求和[5]BSW模块通用规范要求外，CDD模块的代码文件结构不固定。

 至少应该提供一个CDD_.c
 中断函数可以放在CDD_*Irq.c。
 Callout函数可以放在CDD**Callout.c。
 根据需要，在Link time从配置中生成的C对象可以放在CDD**Lcfg.c文件中。
 根据需要，在构建后从配置生成的C对象可以放在CDD*_PBcfg.c文件中。
 如果CDD模块的实现需要额外的代码文件，则可以自由地包含它们

**Header file(s)**
下图包含了CDD模块定义的AUTOSAR头文件层次结构
![在这里插入图片描述](https://img-blog.csdnimg.cn/42e24b03ff5f4584abf27af190069588.png#pic_center)

-  CDD模块应该提供一个头文件结构，这样CDD模块的用户只需要包含CDD_.h文件。
-  如果某些回调函数必须由其他BSW模块处理，CDD模块可能会提供CDD__Cbk.h头文件。
-  根据需要，由配置生成的C对象声明可以放在CDD_*Cfg.h,
  CDD**PBcfg.h，CDD* < MODULENAME > _Lcfg.h文件。
-  如果CDD模块的实现需要额外的头文件，则可以自由地包含它们。头文件是自包含的，这意味着它们将包括它们所需的所有其他头文件
-  CDD模块可能包括Det .h和/或Dem.h头文件来报告错误
-  如果必须定义一些内存映射区域，其中是模块实现前缀，CDD模块可能包括_MemMap.h头文件。
-  如果配置了与Rte的接口，CDD模块可能包含Rte_CDD_.h头文件。

### 2.2行为和接口描述

一些CDD不仅具有到其他BSW模块或集群的接口，而且还具有通过Rte从应用程序sw - c访问的更抽象的接口。（CDD不仅能连RTE，也可以连BSW）

在这种情况下，需要一个SW-C类型的CDD来连接Rte, CDD应遵守文档[10]BSW模块规范的要求。（SWC类型的CDD？还没理解）

**描述模板**

这个描述文件应该包含:

-  CDD服务描述
-  类型和端口
-  描述内部行为和可运行实体
-  可运行实体所需触发事件的描述
-  独占共享资源保护区域描述
-  内存映射

这里需要的更抽象的接口称为AUTOSAR接口，通过软件组件模板(SWCT)来描述，它们由端口、端口接口及其进一步的详细信息组成

用于为CDD描述这些元素的SWCT的根类是ComplexDeviceDriverSwComponentType。

从Rte到这些CDD的函数调用应该被建模为也包含在SWCT中的可运行实体。类的根类
用于描述RunnableEntities(以及其他一些东西)的SWCT被调用SwcInternalBehavior。

**提示:CDD可运行程序应该设计成减少Rte开销，例如:**
服务器可运行对象最好是可重入的:可以并发调用=TRUE（可重入？意思是可多次调用？）
可运行签名为:void或StdReturnType RunnableName(void或参数)

### 2.3参数配置

如果必须使用AUTOSAR GCE配置参数，CDD应遵守文档[11]ECU配置规范的要求

至少：
 模块的AUTOSAR和软件版本应由配置文件标识。
 Det不应该包含在生产阶段，因此在配置中需要一个参数来禁用错误报告。

## 三、与其他模块的接口

介绍基础软件与其他模块的对应关系。

### 3.1 与Rte和SWC接口

CDD可能需要通过Rte与sw -c连接:
 “应规定所需的端口和接口，并据此实现AUTOSAR (AUTOSAR接口)。（意思SWC要预留给CDD的端口）

 在某些情况下，CDD必须使用Rte定义的一些端口特定参数

### 3.2 与库的接口

CDD可以使用AUTOSAR库。

例如:CDD可以使用端到端库机制来传输防止数据损坏或丢失的通信保护。（就是E2E校验）

### 3.3 接口到标准BSW模块

CDD可能需要连接到分层软件体系结构中的其他模块，或者分层软件体系结构中的模块可能需要连接到CDD。如果是这种情况，以下建议适用:

从分层软件体系结构的模块到CDD的接口:（BSW到CDD需要注意的事项）

CDD应提供通用配置的可接入AUTOSAR模块的接口（大概是要CDD使用BSW中定义的Autosar标准函数）

一个典型的例子是PDU路由器:一个CDD可以实现一个新的总线系统的接口模块。在PDU的配置中已经考虑到了这一点。

**从CDD到分层软件体系结构模块的接口:**
只有当分层软件体系结构的各个模块提供接口，并准备好供CDD访问时，才允许这样做。通常这意味着

CDD应负责接口的可重入性。对于不可重入接口，只有一个调用者可以访问该接口。对于有条件的可重入接口，如果多个调用方使用不同的id，则可以同时访问该接口。
可重入性：
重入一般可以理解为一个函数在同时多次调用，例如操作系统在进程调度过程中，或者单片机、处理器等的中断的时候会发生重入的现象。
可重入的函数必须满足以下三个条件：
（1）可以在执行的过程中可以被打断；
（2）被打断之后，在该函数一次调用执行完之前，可以再次被调用（或进入，reentered)。
（3）再次调用执行完之后，被打断的上次调用可以继续恢复执行，并正确执行

- 如果使用回调例程（call back routines），则名称是可配置的。
- 不存在上层模块来管理模块的状态(并行访问会在不被上层模块发现的情况下改变状态)

CDD应提供满足其他AUTOSAR模块所需的所有配置参数，例如，如果Dem被调用来报告生产错误，则Dem错误代码必须在CDD配置中符合Dem错误码定义的配置标准

N多核架构的情况，请参考§7.4章节

一般来说，可以访问以下模块:

#### 3.3.1 Interfacing with MCAL modules

CDD可以直接访问微控制器资源(例如硬件定时器)。如果所需的资源由MCAL模块管理，并且没有特定的约束(例如实时需求)，CDD应该使用MCAL。强烈建议这样做以避免冲突(例如，并行访问同一组/通道/等)。大多数情况下是不允许的，因为DIO服务不能重入)。

（CDD没有实时需求或者其他的，就使用MCAL标准函数。避免并行访问硬件。尽量避免重入，像DIO就无法重入）
（**解决DIO重入问题的一种想法**：使用指针，通过形参传入地址，使返回值存放在不同的地方？）
在这种情况下，CDD应该使用MCAL模块的标准API来访问MCAL模块

#### 3.3.2 Interfacing with ECU State Manager fixed

EcuM应是状态管理的唯一接入点，以防ECU State Manager fixed被使用：

-  Init和De-Init函数应该由EcuM单独调用。
-  CDD模式的变更应由EcuM处理
-  如果CDD处理唤醒源，则必须遵循文档[13]ECU State Specification中指定的处理唤醒事件的协议。
-  具有固定状态机的管理器。

#### 3.3.3 Interfacing with BSW Mode Manager & ECU State Manager Flexible

如果使用灵活的ECU状态管理器，EcuM和BSW模式管理器应该是模式管理的专有访问点

**ECU状态管理器应灵活用于**:

-  Init和De-Init函数应唯一由EcuM和/或BswM模块调用。
-  如果CDD处理唤醒源，则必须遵循文档[14]ECU State Manager. Specification中指定的处理唤醒事件的协议

**BSW模式管理器应该用于:**

-  CDD模式改变了管理
-  BswM(在主核心上)确定ECU应关闭，并将适当的模式开关分配到每个核心。从核上的CDD必须捕获此模式的开关；适当地取消初始化，并向BswM发送适当的信号以指示它们准备就绪。

#### 3.3.4 Interfacing with Memory Stack

直接访问外部NVRAM管理器是可能的，如果它是专门由CDD管理的。如果CDD使用标准内存堆栈，NVRAM管理器是内存堆栈的独占访问点:CDD应该使用NVM的API来访问内存。

#### 3.3.5 Interfacing with Watchdog Stack

看门狗管理器可以监督一个或多个可运行程序的执行CDD作为监督实体。看门狗管理器需要配置，可运行的CDD需要调用文档[21]看门狗中描述的看门狗API

看门狗管理器是看门狗堆栈的独占访问点。CDD不应该直接与看门狗管理器交互，而是通过Rte定义的端口进行交互。通常，Rte负责传播检查点信息CDD中的被监督实体到看门狗管理器模块。监督部门Manager模块使用Runtime Environment的服务将监控状态的变化通知CDD。

为了控制CDD的状态依赖行为，Rte提供了模式端口机制。模式管理器可以在模式端口中定义的不同模式之间切换。连接到模式端口的CDD可以通过两种方式使用模式信息:
—CDD可通过模式端口查询当前模式。
CDD可以声明Rte由于模式变化而启动或停止的可运行对象。

在失败的情况下，看门狗管理器可以通过Rte模式机制将监督失败通知CDD被监督实体。然后CDD监督实体可以采取行动从该失败中恢复。

#### 3.3.6 Interfacing with Communication Stack

有几个可能的接入点:

- 可以连接到PDU Router模块来处理IPDU。
- 可以连接到接口。
- 可以连接到NM模块。
- 可以连接TcpIp模块。
- 它可以直接接口到Com模块，因为它可以有信号接口。

一般不适合混合接入点，即同时使用PduR接入点和Com接入点或接口

处理通信并可能触发pdu传输的CDD应该提供一个API来启用/禁用传输。这将使Dcm在相应的诊断请求中禁用整个通信。CDD提供的这些函数可以在链接到该函数的配置动作列表中调用。例如，对于这些函数，请参考通信堆栈中的类似API

##### 3.3.6.1 Interfacing with PDU Router

PduR是IPDU通信栈的独占总线和协议独立访问点。
CDD通过PduR模块的标准接口访问IPDU。
CDD与PduR交互时，每个CDD在PduR内配置一个容器。
详情请参见文档《[22]PDU路由器规格》。

##### 3.3.6.2 Interfacing Interfaces modules

接口模块是通信堆栈的专用总线访问点。
CDD使用接口模块的标准api访问IPDU。
当CDD与接口交互时，CDD使用为接口定义的访问函数，并且接口回调应根据
CDD的需要。<总线>接口应配置为包括
CDD__Cbk.h头文件。
详细信息请参见接口规范和用户手册

##### 3.3.6.3 Interfacing with Com Module

如果CDD处理Com信号，则CDD应使用Com模块的标准api或Rte定义来访问信号。
详细信息请参考文档[23]Specification of Communication。

##### 3.3.6.4 Interfacing with Com Manager

如果CDD使用Com信号，CDD应使用Com管理器的标准api来请求“通信模式”。

如果CDD处理的不是AUTOSAR标准，状态应该由ComM处理以协调总线通信堆栈。
详细信息请参见文档[24]Specification of Communication Manager

##### 3.3.6.5 Interfacing with Network Management Interface module

如果CDD处理的不是AUTOSAR标准，状态应该由Nm_CDD模块处理。
Nm_CDD应该向Network Manager提供服务进行管理
<总线>。
详情请参见《[25]网管接口规范》

##### 3.3.6.6 Interfacing with TcpIp module

TcpIp模块是通信堆栈的基于套接字的独占访问点。
CDD应该使用TcpIp模块的标准api来访问套接字。
详细信息请参考文档[26]Specification of TCP/IP Stack

#### 3.3.7 Interfacing with XCP module

如果CDD处理的<总线>不是AUTOSAR标准，XCP可以接入<总线>_CDD来转发数据。

XCP模块提供可供CDD使用的可配置接口:
<Cdd_Transmit>请求通过CDD发送PDU
<Xcp_CddTxConfirmation> API确认成功传输
PDU<Xcp_CddRxIndication> CDD调用的API服务表示成功接收到LPDU。

XCP模块应配置为允许CDD功能:
XcpOnCddEnabled参数将被激活。
如果需要，CDD可以调用回调函数Xcp_RxIndication

#### 3.3.8 Interfacing with Diagnostic Log and Trace

如果CDD处理的<总线>不是AUTOSAR标准，Dlt可以接口
<总线>_CDD转发数据。

Dlt将数据转发到Dcm或使用串行接口的CDD。
Dlt不定义特定的通信接口。Dlt规范定义了一个到内部Dlt通信模块的API。这取决于实现者，如何实现这个通信模块，以及它如何与可能的对象通信
CDD(如串行或USB)

#### 3.3.9 Interfacing with Default Error Tracer and Development Error

Manager

CDD应使用文档[19]中描述的Det, Dem报告错误
AUTOSAR标准误差的描述。
CDD应使用Det和Dem模块的标准api。
CDD应与任何BSW模块反应一致。错误ID应该在CDD模块中本地定义。CDD负责启动内部恢复。

#### 3.3.10 Interfacing with OS

通常，只有BSW Scheduler和Rte可以使用OS对象或OS服务。
因此，CDD应该只访问GetCounterValue和
操作系统的GetElapsedCounterValue服务。
只要使用的OS对象没有被其他BSW模块使用，CDD就可以访问OS，例如CDD可以创建OS告警并使用。

OS可以通过OsRestartTask通知CDD某个OS- application已经被终止并重新启动。CDD将不得不采取适当的清理行动。
详细信息请参考文档[15]Specification of Operating System

### 3.4 CDD in multi-cores system

CDD可用于多核架构。
在多核架构的情况下，CDD可以驻留在任何核心上，遵循以下规则:

跨分区和核心边界只允许用于模块内部通信，使用主/卫星实现。

因此，如果CDD需要访问的标准化接口BSW，它需要驻留在同一个核心上。

如果CDD位于不同的核心上，它可以使用正常的端口机制来访问AUTOSAR接口和标准化的AUTOSAR接口。这将调用Rte, Rte使用操作系统的IOC机制将请求传输到另一个核心。
“但是，如果CDD需要访问BSW的标准化接口，并且不在同一个核心上，
提供标准化接口的卫星可以在CDD所在的核心上运行，并将调用转发到另一个核心o，或者CDD的存根部分需要在另一个核心上实现，并且需要使用操作系统的IOC机制(类似于Rte所做的)组织CDD-local通信。

此外，在后一种情况下，CDD的初始化部分也需要驻留在不同核心上的存根部分中

### 3.5 CDD module of the MCAL

微控制器驱动程序的CDD可以执行，但它不能访问其他标准模块，因为它是在下层除Det, Dem, SchM…
一般来说，如果某些限制应用于特定的层，那么它也适用于CDD