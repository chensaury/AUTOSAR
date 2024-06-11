# XCP概念和基本原理介绍



ASAM接口模型描述了Slave和Master之间发送和接收命令和数据。为了独立于特定的物理[传输层](https://so.csdn.net/so/search?q=传输层&spm=1001.2101.3001.7020)，XCP被细分为协议层和传输层。

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/38388cc3a2d346e3bcdf67a6dff749df.png)



根据传输层的不同，可分为XCP ON CAN、XCP ON Ethernet等。早在2005年XCP ON [FlexRay](https://so.csdn.net/so/search?q=FlexRay&spm=1001.2101.3001.7020)首次亮相时，对新传输层的可扩展性就得到了证明。XCP协议的当前版本是1.3版本，于2015年获得批准。

在设计该协议时优先考虑遵守以下原则:

 

- ECU 中的资源使用最少
- 高效沟通
- 简单的从机实现
- 即插即用配置，只需少量参数
- 可扩展性

XCP的一个关键功能是允许对Slave的内存进行读写访问。

读访问让用户测量一个内部 ECU 参数的时间响应。ECU 是具有离散时间行为的系统，其参数仅在特定的时间间隔内发生变化：仅当处理器重新计算值并在 RAM 中更新它时。XCP 在于获取测量值从同步变化的RAM到ECU中处理流程或事件，相关机制将在后面详细说明。

写访问允许用户在Slave中优化算法参数。访问是面向地址的，即内存中主引用地址和从引用地址之间的通信。所以，一个参数的测量本质上是作为一个Master向Slave的请求实现的:“给我内存位置0x1234的值”。参数的校准—写访问—到Slave，意味着:“将地址0x9876的值设置为5”

XCP Slave并不一定需要在ecu中使用。它可以在不同的环境中实现:从基于模型的开发环境到hardware-in-the-loop 和software-in-the-loop件环境，再到通过JTAG、NEXUS和DAP等调试接口访问ECU内存的硬件接口。

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/6856f463fc344ffc98d9ba324d093090.png)

 



如何通过对ECU的读写访问来优化算法?这样做有什么好处?为了能够在ECU运行时修改单个参数，必须有访问它们的权限。并不是每种类型的内存都允许这个过程。只可能对RAM中的内存地址执行读写访问(这里有意排除EEPROM)。以下是个人记忆技术之间的差异的简要总结:对它们的知识是非常重要的理解在本书的进一步过程。

Memory基本原理

如今，flash内存存通常集成在ecu的微控制器芯片中，即使没有电源供应，也能长期存储代码和数据。Flash内存的特殊之处在于，对单个字节的读访问确实可以在任何时候进行，但对新内容的写入只能以块的方式进行，通常是以相当大的块进行。

Flash内存的寿命是有限的，这是指定的擦除周期的最大数量(取决于具体的技术，最大可达100万个周期)。这也是写周期的最大数量，因为在再次写入内存之前，必须始终将内存擦除。

当这种擦除程序重复多次时，绝缘层(“隧道氧化膜”)可能会损坏。这意味着电子会慢慢泄漏，随着时间的推移，一些信息会从1变为0。因此，在ECU中允许的闪存周期的数量受到严重限制。在生产ECU中，它往往只在个位数的顺序上。这个限制由Flash Boot Loader监控，它使用一个计数器来跟踪已经执行了多少Flash操作。当超过指定的数量时，Flash Boot Loader拒绝另一个Flash请求。

随机存取存储器(RAM)需要一个永久的电源供应;否则它会丢失内容。Falsh内存用于应用程序的长期存储，而RAM用于缓冲计算数据和其他临时信息。关闭电源会导致RAM内容丢失。与flash内存相比，RAM很容易读取和写入。

这个事实很清楚:如果需要在运行时更改参数，必须确保它们位于RAM中。理解这种情况是非常重要的。这就是为什么我们将基于下面的例子来看看ECU中应用程序的执行:



在应用程序中，y参数是从传感器值x计算出来的。

/ Pseudo-code representation

a = 5;

b = 2;

y = a * x + b;

果应用程序在ECU中闪烁，控制器在启动后按如下方式处理该代码:x参数的值对应传感器的值。因此，在某些时候，应用程序必须轮询传感器值，然后将该值存储在分配给x参数的内存位置中。因为这个值总是需要在运行时重写，所以内存位置只能位于RAM中。

计算参数y。a和b，作为因子和偏移量，存储在flash内存中。它们被存储为常量。y的值必须存储在RAM中，因为这是唯一可以进行写访问的地方。在运行编译器/链接器时，会设置参数x和y在RAM中的位置，以及a和b在flash中的位置。这就是对象被分配到唯一地址的地方。对象名称、数据类型和地址之间的关系记录在链接器映射文件中。链接映射文件是由编译器运行生成的，可以以不同的格式存在。然而，所有格式的共同点是，它们至少包含对象名称和地址。

在这个例子中，如果偏移量b和因子a依赖于特定的车辆，那么a和b的值必须单独适应车辆的特定条件。这意味着算法保持不变，但参数值会随着车辆的不同而改变。

在ECU的正常工作模式下，应用程序从flash内存运行。它不允许对单个对象进行任何写访问。这意味着位于flash区域的参数值不能在运行时修改。如果在运行时可以更改参数值，则要修改的参数必须位于RAM中，而不是闪存中。现在，参数和它们的初始值是如何进入内存的呢?如何解决需要修改更多的参数，而不能同时存储在RAM中的问题?这些问题将我们引向标定概念的话题。

可以通过XCP协议的机制对内存内容进行读写访问。访问是以面向地址的方式进行的。读访问允许测量RAM中的参数，而写访问允许校准RAM中的参数。XCP允许与ECU中的事件同步执行测量。这确保了测量值相互关联。每次重新开始测量时，可自由选择要测量的信号。对于写访问，需要校准的参数必须存储在RAM中。这需要一个标定概念。



### 02Xcp协议层介绍

XCP 数据在 Master 和 Slave 之间以基于消息的方式进行交换。整个“XCP 消息帧”嵌入在[传输层](https://so.csdn.net/so/search?q=传输层&spm=1001.2101.3001.7020)的帧中（XCP ON Ethernet 嵌入UDP报文中）。XCP报文包括三部分：XCP头、XCP包和XCP尾。

下图中，部分消息用红色表示，用于发送当前的 XCP 帧。XCP头和XCP尾取决于传输协议。

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/7726ae7bd7ef4913920ae915245070e5.png)



XCP包本身独立于所使用的传输协议。它总是包含三个组件:“标识字段”、“时间戳字段”和当前数据字段“数据字段”。每个标识字段以标识数据包的PID (Packet Identifier)开始。

下面显示已经定义了的PID：

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/7f7b809dbb8d4c7b9472a92ff6a0feab.png)

 



XCP通信分为两种方式，一种是命令 ([CTO](https://so.csdn.net/so/search?q=CTO&spm=1001.2101.3001.7020))，一种是发送同步数据 (DTO) 。

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/11b3ecc71cd3434d8b180bbf1425869b.png)

 



首字母缩略词代表

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/51faa20147c14694909c5b2f249ff0b2.png)

 



通过CTO（命令传输对象）交换命令。例如，Master以这种方式发起请求。Slave必须始终以RES或ERR响应CMD。其他CTO消息是异步发送的。数据传输对象（DTO）用于交换同步测量和激励数据。

 

### 标识段

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/c883409a082c43c5a44524c8526085ff.png)

 



当交换消息时，Master和Slave都必须能够确定对方发送了哪条消息。这在标识领域中完成。这就是为什么每个消息都以包标识符(PID)开始的原因。

在发送CTO时，PID字段完全足以识别CMD、RES或其他CTO数据包。可以看出，从Master到Slave的命令使用了一个从0xC0到0xFF的PID。XCP Slave用从0xFC到0xFF的pid响应或通知Master。这将导致一个独一无二的PID分配给单独发送的CTO。传输DTO时，将使用标识字段的其他元素。

### 时间戳字段

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/d6bf526050584afea05795d1ea323f88.png)

 



DTO包使用时间戳，但在CTO消息的传输中这是不可能的。Slave使用时间戳来提供测量值的时间信息。也就是说，Master不仅有测量值，还有测量值获取的时间点。测量值到达主服务器所花费的时间不再重要，因为测量值和时间点之间的关系直接来自于从服务器。

从Slave传输时间戳是可选的。这个主题在ASAM XCP第2部分协议层规范中有进一步的讨论。

### 数据字段

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/08514233d10a42abb5c2e803573f0bfa.png)

 



最后，XCP包还包含存储在数据字段中的数据。对于CTO报文，数据字段由不同命令的具体参数组成。DTO报文包含从Slave发送的测量值，当STIM数据被发送时，则包含从Master发送的值







## 03 Xcp CTO 命令介绍

# 

[CTO](https://so.csdn.net/so/search?q=CTO&spm=1001.2101.3001.7020)被用来将命令从Master发送到Slave，以及将Slave的响应发送到Master。

### [XCP]命令结构

Slave 收到来自 Master 的命令，必须对其做出正面或负面的响应。

**命令 (CMD):**

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/e5d83560aa3c46af98873eefa43a65ee.png)



每条命令都有一个唯一的编号，此外，还可以随命令发送其他具体参数，这里定义的最大参数个数为MAX_CTO-1，MAX_CTO表示CTO包的最大长度，单位为字节。

**肯定响应：**



![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/656dda330bbe4310b4b08635d8212751.png)

 

**否定响应：**

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/a313ce84161242c0b73786bfa3e1f939.png)

 



具体参数也可以作为补充信息传递给消极的回应，而不仅仅是积极的回应。举个例子，在Master和Slave之间建立连接。在Master和Slave开始通信时，Master向Slave发送一个连接请求，而从Slave必须积极响应，从而产生一个连续的点对点连接。

Master –>Slave: 连接

Slave –> Master: 肯定响应

**连接命令：**

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/93d2c9c2ebf44857b74c6d522e56d0c1.png)

 



模式 00 表示 Master 希望与 Slave 进行正常XCP 通信。如果 Master 在建立连接时使用 0xFF 0x01，则 Master 正在请求与 Slave 进行 XCP 通信。同时，它通知 Slave 它应该切换到特定的 - 用户定义- 模式。

**Slave****肯定响应：**

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/a551969ce3f84b2aa8bd40165be0e903.png)

 



Slave的肯定响应可以采取更广泛的形式。当与Slave建立连接时，它已经向Master发送了特定于通信的信息。例如，RESOURCE是Slave提供的关于是否支持页面切换或是否可以在XCP上闪烁的信息。使用MAX_DTO，从端通知主端它支持的传输测量值的最大数据包长度，等等。您将在ASAM XCP第2部分协议层规范中找到有关参数的详细信息。

XCP允许在主从之间交换命令和反应的三种不同模式:Standard模式、Block模式和Interleaed模式。

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/05834d06dbb542cea0d3953d3b3b3029.png)

 



在标准的通信模型中，每个对Slave的请求后面都有一个响应。除了XCP ON CAN，它不允许多个slave对Master的命令做出反应。因此，每个XCP消息总是可以追溯到一个唯一的Slave。这种通信模式就是Standard模式。

Block传输模式是可选的，节省了大量数据传输(例如上传或下载操作)的时间。尽管如此，在Slave方，在这种模式下必须考虑性能问题。因此，需要维护两个命令之间的最小间隔时间(MIN_ST)，并将命令总数限制在MAX_BS的上限值。另外，Master可以使用GET_COMM_MODE_INFO从Slave读取这些通信设置。前面提到的限制在Master方Block传输模式中不需要观察，因为PC机的性能几乎总是足以接受来自微控制器的数据。

由于性能原因，还提供了交错模式。但是这种方法也是可选的，与块传输模式相反，它在实践中没有相关性。



### CMD

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/b81b0cce56da43a7ad64e6afceebb3b9.png)

 



Master通过CMD向Slave发送一个通用请求。PID (Packet Identifier)字段包含命令的标识号。附加的特定参数在数据字段中传输。然后Master等待Slave肯定或者否定响应。

XCP的实现也具有很强的可伸缩性，因此不需要实现每个命令。在A2L文件中，把可用的cmd列在所谓的XCP IF_DATA中。如果在A2L文件中的定义和在Slave中的实现之间有差异，Master可以根据Slave的响应确定，Slave甚至不支持该命令。如果Master发送了一个没有在Slave上实现的命令，那么Slave必须响应ERR_CMD_UNKNOWN并且没有在Slave上发起任何活动。这会让Master快速知道一个可选命令还没有在Slave上实现。

命令中还包括其他一些参数。请从文档ASAM XCP第2部分的协议层规范中获取精确的细节。

命令按组划分: Standard, Calibration, Page, Programming 和 DAQ测量命令。如果不需要某个组，则不需要实现该组的命令。如果组是必要的，那么某些命令必须始终在Slave中可用，而组中的其他命令是可选的。

下面以概述为例。Page组中的SET_CAL_PAGE和GET_CAL_PAGE命令被标识为非可选。这意味着在一个支持页面切换的XCP Slave中，至少必须实现这两个命令。如果在Slave中不需要切换页面，则不需要执行这些命令。

**Standard** **命令:**

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/f5c4108448144f62bba1739e7f47a31b.png)

 

Calibration**命令：**

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/2cd2ec15d2ef4db8aefea79bd1e7a7c1.png) 



**Page命令：**![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/fea833c990d84d208086acc49cb7f009.png)

 



**周期数据交换—****基础：**

 

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/dbeaa73851bc4e1d858d8aea517ef567.png)

 



**周期数据交换—****静态配置**

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/88666122375f41afbec4222f20ab95db.png)

 

 



**周期数据交换—****动态配置**

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/4a09ffb2bc1f449fa8aa75de4a90cdc8.png)

 



**Flash programming****：**

![img](https://img-blog.csdnimg.cn/14c9ccce8a674984ab39b8ea80984fd9.png)

 



### RES

如果Slave能够成功地满足Master的请求，Slave 会使用RES表示肯定响应。

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/acbb88b0a16247ff90a44d4c51831466.png)

 



在ASAM XCP第2部分协议层规范中找到有关参数的更详细信息

### EV

如果Slave想要通知Master一个异步事件，可以发送一个EV来做这件事。它的实现是可选的。

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/a2eac1a6c1f9450ab5ae9056f136d338.png)

 



在ASAM XCP第2部分协议层规范中找到有关参数的更详细信息

我们将更多地讨论与测量和激励有关的事件。这与XCP Slave发起发送事件的动作没有任何关系。相反，它涉及Slave报告诸如特定功能故障之类的干扰。

### SERV

Slave可以使用这种机制请求Master执行一个服务。

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/e52ed4db501e42718060d56a86d37dc2.png)

 



在ASAM XCP第2部分协议层规范中找到有关参数的更详细信息







## 04 Xcp 标定过程介绍

要更改[XCP](https://so.csdn.net/so/search?q=XCP&spm=1001.2101.3001.7020) Slave中的参数，XCP Master必须将参数的位置以及值本身发送给Slave。

XCP总是用5个字节定义地址:4个字节用于实际地址，1个字节用于地址扩展。基于CAN传输，XCP消息只有7个有用的字节可用。例如，如果[标定](https://so.csdn.net/so/search?q=标定&spm=1001.2101.3001.7020)工具设置了一个4字节的值，并且想要在一个CAN消息中同时发送这两个信息，那么就没有足够的空间来完成这一操作。由于总共需要9个字节来传输地址和新值，这个变化不能在一个CAN消息(有用的7个字节)中传输。因此，标定请求是由Master向Slave发送两条信息完成，Slave必须确认这两个消息。

下图显示了Master和Slave之间的通信，需要设置一个参数值。实际信息解释被隐藏，通过用鼠标“展开”它来显示。

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/0cfa7a785eeb4896ba0d0922f7774209.png)



在Master的第一个消息中，Master发送了SET_MTA命令给Slave，该命令将写入一个新的地址值。在第二条消息中，Slave对命令表示肯定Ok: SET_MTA。

第三条消息下载传输十六进制值以及有效的字节数。在本例中，由于是浮点数，有效的字节数是4。Slave在第四个消息中同意给予了肯定响应。

这就完成了当前的标定过程。在Trace显示中，您可以识别到最后一个命令SHORT_UPLOAD。，这是Vector的测量和标定工具[CANape](https://so.csdn.net/so/search?q=CANape&spm=1001.2101.3001.7020)的一个特殊方面。为了确保标定成功，在标定结束后再次读出该值，并将显示更新为读出值。这可以让用户直接识别是否执行了标定命令。这个命令也得到了一个肯定的确认:SHORT_UPLOAD。

当参数在ECU的RAM中发生变化时，应用程序将处理新的值。然而，重新启动ECU会导致该值被擦除，并用flash中的原始值覆盖RAM中的值。那么，如何永久保存修改后的参数呢?

基本上有两种可能:

 

### 保存在ECU中

例如，RAM中改变的数据可以保存在ECU的EEPROM中:当ECU掉电时自动保存，或由用户手动保存。前提条件是数据可以存储在Slave的非易失性内存中。在ECU中，这将是EEPROM或flash。然而，具有数千个参数的ECU很少能够提供这么多未使用的EEPROM内存空间，因此这种方法很少见。

另一种可能是将RAM参数写回ECU的flash内存。这种方法比较复杂。Flash内存必须先被擦除，然后才能被重写。反过来，这只能作为一个块来完成。因此，这不是简单地回写单个字节的问题。

### 将参数以文件的形式保存到PC上

更常见的是将参数存储在PC上。所有参数(或它们的子集)都以文件的形式存储。不同的格式是可用的;最简单的情况是ASCII文本文件，它只包含对象的名称和值。其他格式还允许保存其他信息，例如关于修订历史参数的成熟度级别等。

场景:完成工作后，标定员希望享受一个自由的夜晚。因此，标定工具将执行的更改，以参数设置文件的形式保存在ECU的RAM中。第二天，标定员想要继续他停止的工作。标定工具启动ECU。在引导时，参数在RAM中初始化。然而，ECU使用存储在flash中的值来做到这一点。这意味着前一天的变化在ECU中不再可用。为了继续前一天停止的工作，标定工具使用DOWNLOAD命令通过XCP将参数设置文件的内容传输到ECU的RAM中

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/123fd4e4a8d040898a835110038d55fe.png)

 



### 保存参数设置文件Hex文件和刷新

刷新ECU flash是另一种改变flash参数的方法。当ECU启动时，它们被作为新参数写入RAM。参数文件也可以被传输到C或H文件中，并在运行编译器/链接器时被制成新的flash文件。但是，根据代码的参数，生成flash十六进制文件的过程可能需要相当长的时间。此外，标定工具可能没有任何ECU源代码，这取决于工作过程。

作为一种替代方案，标定工具可以将参数设置文件复制到现有的flash文件中。

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/7d137cea99fa4316a20e46db31adc515.png)

 



在flash文件中，有一个包含地址和值的十六进制文件。现在参数文件可以复制到十六进制文件中。为此，CANape从参数文件中获取地址和值，并在Hex文件的相关位置更新参数值。这将产生一个新的Hex文件，其中包含更改的参数值。现在这个Hex文件现在必须重新写入flash中，并且在重新启动后，新的参数值在ECU中是可用的。



# XCP实战系列介绍16-XCP标定过程指令解析

### 本文框架

- [1.前言](https://blog.csdn.net/initiallizer/article/details/130231086#1_1)
- [2. XCP标定过程指令解析](https://blog.csdn.net/initiallizer/article/details/130231086#2_XCP_4)



# 1.前言

前面几篇文章我们介绍了[XCP](https://so.csdn.net/so/search?q=XCP&spm=1001.2101.3001.7020)底层原理，配置方法及基于CANape，CANoe或Vehicle SPY进行观测或标定的方法，在本篇中我们将对标定过程的指令进行解析，希望能加深大家对标定过程的理解。 此外，大家如果对标定的指令过程能理解到位的话有助于后续通过[CANoe](https://so.csdn.net/so/search?q=CANoe&spm=1001.2101.3001.7020) 的CAPL语言编写对应的脚本文件从而达到不需要专业标定工具就能实现标定的目的。

# 2. XCP标定过程指令解析

在测试时，通过[CANape](https://so.csdn.net/so/search?q=CANape&spm=1001.2101.3001.7020)工具作为上位机，同时打开了Trace window界面，通过Trace window界面对报文指令进行查看，截图如下图： ![在这里插入图片描述](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/35ee2f07b5ad43e0878d403bd5bfcdeb.png) 1）对于CANape工具而言，在建立连接到online模式后，上位机会首先自动发送GET_CAL_PAGE，询问从设备ECU当前激活的页面和XCP访问的页面。

2）在获取GET_CAL_PAGE肯定响应后，上位机继续发送SET_CAL_PAGE进行切面，执行对应的过程如下，将激活页切换到WorkingPage，即映射到的Ram区域： ![在这里插入图片描述](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/3eedb4265f9440e9b9ac37c8e60dff61.png) 3）上位机发送SET_MTA操作，该命令中包含应写入新值的地址，从机在随后消息中用Ok:SET_MTA对命令进行肯定确认。 ![在这里插入图片描述](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/9f0a26087b094901b023e0b5859d54f0.png) 4）上位机发送F0 DOWNLOAD命令，传输需要下载的十六进制值以及有效字节数。在这个例子中，有效的字节数是四。 ![在这里插入图片描述](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/d047ab5ceeb744bb82209f2452ed010c.png)





# MDA项目XCP标定使用说明v1.0

### 一、测量量内存段指定

```c
     section_layout :vtc:linear
    {
        group(ordered, contiguous, run_addr = 0xb0075000, align = 8, attributes=rw)
        {
            section "calibration_section_ram_mes" (size = 8k, overflow = "symbol", attributes=rw)
            {
                select ".zbss.calibration_parameter_mes_cal"(attributes =+rw);
                select ".bss.calibration_parameter_mes_cal"(attributes =+rw);
           }
         }
    }
```

测量量放在LMU（ram）区域的calibration_parameter_mes_cal段，大小为8K，起始地址为0xb0075000,

在链接文件中已对该段做类型指定，可存放.bss（默认为0的初始化全局变量）

#### 测量量声明方式：

```c
#pragma section all "calibration_parameter_mes_cal"
volatile uint16  mes_0;
uint16  mes_1;
#pragma section all restore
```

编译后map文件：

![image-20240111105133364](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/image-20240111105133364.png)

另：如果定义方式为前两种，则会定义在指定段以外的地方

```c
#pragma section all "calibration_parameter_mes_cal"
volatile const __attribute__((protect)) uint16  mes_1 = 0x1234;
volatile uint16  mes_2 = 0x4567;
volatile uint16  mes_3;
uint16  mes_4;
#pragma section all restore
```

编译后map文件：

使用mes_2变量时，可以看到其分配的地址不在段内

![企业微信截图_17049423016588](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049423016588.png)

如果mes_2未使用时，变量虽然在相关的段内，但不会分配地址

![企业微信截图_17049417861225](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049417861225.png)

![image-20240111105923511](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/image-20240111105923511.png)

### 二、标定量内存段指定

```c
     section_layout :vtc:linear
    {
        group(ordered, contiguous, run_addr = 0xb0060000, align = 8, attributes=rw)
        {
            section "calibration_section_ram_cal" (size = 64k, overflow = "symbol", attributes=rw)
            {
                select ".zdata.calibration_parameter_wp_cal"(attributes =+rw);
                select ".data.calibration_parameter_wp_cal"(attributes =+rw);
                select ".zrodata.calibration_parameter_wp_cal"(attributes =+rw);
                select ".rodata.calibration_parameter_wp_cal"(attributes =+rw);
            }
        }
        group(contiguous, ordered, load_addr = 0x80100000)
        { 
            select ".zdata.calibration_parameter_wp_cal";
            select ".data.calibration_parameter_wp_cal";
            select ".zrodata.calibration_parameter_wp_cal";
            select ".rodata.calibration_parameter_wp_cal";

        }       
    }
```

1、标定量的变量地址放在LMU（ram）区域的calibration_parameter_wp_cal段，大小为64K（大小可根据需要调整），起始地址为0xb0060000，在链接文件中已对该段做类型指定，作为.data（已初始化全局变量）或.rodata（常量）的变量地址段

2、标定量的变量初始值放在Pflash0（rom）区域的calibration_parameter_wp_cal段，大小同上，起始地址为0x80100000，在链接文件中已对该段做类型指定，可存放.data（已初始化全局变量）或.rodata（常量）的初始值

#### 标定量声明方式：

```c
#pragma section all "calibration_parameter_wp_cal"
volatile uint16  calibration_P0 = 0x4567;
uint16  calibration_P1 = 0x4567;
#pragma section all restore
```

编译后map文件：

1.变量地址：变量未使用时，两种

![企业微信截图_17049426615889](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049426615889.png)

2.映射到pflash，显示calibration_P2和calibration_P3这两个已初始化全局变量在该地址

![企业微信截图_17049428009174](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049428009174.png)

![企业微信截图_17049473795342](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049473795342.png)

查看hex文件，可以看到上述两个变量的初始值存在0x80100000开始的区域内

![企业微信截图_17049429487516](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049429487516.png)

另：如果定义方式为前两种，则会定义在指定段以外的地方

```c
#pragma section all "calibration_parameter_wp_cal"
volatile const __attribute__((protect)) uint16  calibration_P0 = 0x1234;
volatile const uint16  calibration_P1 = 0x4567;
volatile uint16  calibration_P2 = 0x4567;
uint16  calibration_P3 = 0x4567;
#pragma section all restore
```

编译后map文件：

1.变量地址，前两个变量会自动分配地址，且直接放在flash上

![企业微信截图_17049433776108](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049433776108.png)

查看hex相关地址可以看到其初始值

![企业微信截图_17049440065584](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049440065584.png)



### 三、CANape指定相关的数据段

注：新建标定工程等步骤省略

需要将原始hex中的标定数据段替换为当前标定数据，具体步骤如下：

#### （1）设置memory segments

根据下图对memory segments进行配置（可根据需要改变flash和ram的地址）

![企业微信截图_17049411597658](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049411597658.png)

在标定量的设置页面查看标定量的地址，由于已在设备中进行整体的内存偏移映射，故此处无需设置偏移量

![企业微信截图_17049413533294](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049413533294.png)

#### （2）更新A2L文件

打开ASAP2编辑工具，如果提示没有map文件，点ok忽略即可（下面会介绍重新添加方式）

![企业微信截图_17049483565423](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049483565423-1710393545541.png)

查看已经添加的相关标定量和测量量

![企业微信截图_17049485015218](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049485015218-1710393558495.png)

更新map、elf文件或需添加新的标定量、观测量

![image-20240318112834419](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/image-20240318112834419.png)

![image-20240111125843893](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/image-20240111125843893.png)

打开后，按Ctrl+f搜所相关变量，右键设置添加属性为标定量还是观测量，在overview界面查看，也可设置变量所处的group、备注等，然后保存关闭

![企业微信截图_17049502813699](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049502813699-1710393613711.png)

关闭后会提示是否应用前面的设置，点击yes，上述对A2L 的更新，变量的添加或删除变会更新到工程中

![image-20240111132108066](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/image-20240111132108066.png)

可以看到更新后的标定量有两个

![企业微信截图_170495063958](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_170495063958-1710393626539.png)

#### （3）标定数据

开始标定前设置测量模式为polling

![企业微信截图_17049549782648](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049549782648-1710393641213.png)

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1704951995464.png)

标定结束，保存标定数据

![企业微信截图_17049522738340](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049522738340-1710393688269.png)

提示存储，重命名为xxxx.par文件

![image-20240111135311130](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/image-20240111135311130.png)

根据需要设置标定数据文件的备注属性等内容

![image-20240111135431231](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/image-20240111135431231.png)

#### （4）加载标定文件

如果今天未完成标定任务，存为par文件，明天加载历史标定数据文件xxxx.par可继续昨天的标定

![image-20240111135758935](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/image-20240111135758935.png)

#### （5）vCDM工具

1.上述（1）中设置了memory segments，在数据合并过程中需要ram到flash的映射

选择CDM studio，进入界面后选择tools > options，找到extended ASAP2 Setting，勾选地址映射选项，如下图：

![企业微信截图_1704952970631](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1704952970631.png)

2.打开历史标定数据文件xxxx.par

![image-20240111140656660](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/image-20240111140656660.png)

前面已经在extended ASAP2 Setting，勾选地址映射选项，因此在CDM界面添加原始hex文件，此时会弹出如下界面，在地址映射方式里选择xcp，如下

![企业微信截图_17049553727006](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049553727006-1710393720189.png)

![image-20240111144732044](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/image-20240111144732044.png)

以下显示内容，第一个为正在标定的数据，可以直接添加到最右侧的hex文件中，第二个为标定的历史数据文件，也可添加到右侧hex文件中

![image-20240111145009985](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/image-20240111145009985.png)

![企业微信截图_17049558804442](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17049558804442-1710393733742.png)

存储新合并的.hex文件

![image-20240111145342963](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/image-20240111145342963.png)

查看合并前后的.hex对比

![image-20240111150259390](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/image-20240111150259390.png)

#### （6）下载新的.hex文件

下载好后，重新连接，calibration_P0的初始值已经由0x4567变为0x0

完成标定

 







# MCAL知识点（二十六）：英飞凌OVC（OverLay）地址重定向功能及实战

[1、概述](https://blog.csdn.net/weixin_43580890/article/details/131305461#t0)

[2、OverLay数据手册概述](https://blog.csdn.net/weixin_43580890/article/details/131305461#t1)

[2.1、手册概述](https://blog.csdn.net/weixin_43580890/article/details/131305461#t2)

[2.2、数据重定向](https://blog.csdn.net/weixin_43580890/article/details/131305461#t3)

[2.3、目标内存](https://blog.csdn.net/weixin_43580890/article/details/131305461#t4)

[2.4、OverLay内存](https://blog.csdn.net/weixin_43580890/article/details/131305461#t5)

[2.5、全局OverLay控制](https://blog.csdn.net/weixin_43580890/article/details/131305461#t6)

[2.6、OverLay配置的更改](https://blog.csdn.net/weixin_43580890/article/details/131305461#t7)

[2.7、访问保护、属性、并发匹配](https://blog.csdn.net/weixin_43580890/article/details/131305461#t8)

[2.8、Overlay控制寄存器](https://blog.csdn.net/weixin_43580890/article/details/131305461#t9)

[2.9、模块控制寄存器](https://blog.csdn.net/weixin_43580890/article/details/131305461#t10)

[3、OverLay代码解析](https://blog.csdn.net/weixin_43580890/article/details/131305461#t11)

[3.1、初始化](https://blog.csdn.net/weixin_43580890/article/details/131305461#t12)

[3.2、使能与关闭](https://blog.csdn.net/weixin_43580890/article/details/131305461#t13)

[3.3、内存同步](https://blog.csdn.net/weixin_43580890/article/details/131305461#t14)

[4、测试结果](https://blog.csdn.net/weixin_43580890/article/details/131305461#t15)

------



## 1、概述

​    为了[标定](https://so.csdn.net/so/search?q=标定&spm=1001.2101.3001.7020)，很多控制器提供了overlay的功能，将flash数据加载到RAM区域，这个过程被称为flash仿真或flash重定向。用来XCP的页切换功能。

​    假设遇到多个FLASH区域，对应一个RAM区域，就可以通过将单次单个FLASH区域的数据放置到RAM里面，实现页切换与数据映射功能。

​    因为这是特定于目标的，所以用户需要参考目标微控制器的参考手册来实现(本文锚点是TC275单片机为例)该功能。然而，该函数应具有以下功能:

·指定[overlay](https://so.csdn.net/so/search?q=overlay&spm=1001.2101.3001.7020)闪存和RAM内存的地址和大小

·将overlay闪存中的数据复制到目标RAM中

·为overlay初始化全局寄存器

·这个功能需要放在[XCP](https://so.csdn.net/so/search?q=XCP&spm=1001.2101.3001.7020)的初始阶段。

Overlay对页切换的作用

该特性由Overlay的全局寄存器控制。该函数应具有以下功能:

当主机请求进入工作页时，通过Xcp_SetCalPage命令切换到对应页码的工作页RAM区域。

当主机请求进入参考页时，通过Xcp_SetCalPage命令切换到对应页码的参考页Flash区域。

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\f2205246df564ea794f8645ee9c46fd1.png)

 展示代码如下

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\9e4c060de85645f9ae2caf725c9b2056.png)

## 2、OverLay数据手册概述

### 2.1、手册概述

​    数据overlay功能提供了将选定的数据访问重定向到overlay内存的能力。TriCore对程序闪存、在线数据采集空间或EBU空间的数据访问可以重定向。overlay存储器可以位于本地存储器(如果存在)、仿真存储器(仅限仿真设备)或DPSR/PSPR存储器中。覆盖功能使其成为可能，例如，在程序运行期间修改应用程序的测试和校准参数(通常存储在闪存中)。注意，只有读和写数据访问被重定向。访问重定向的执行没有性能损失。

​    注意:由于地址转换是在DMI中实现的，它只对TriCore的数据访问有效。由TriCore获取指令或访问任何其他主控(包括调试接口)不受影响!

**功能总结**

1、重定向数据访问地址到PFLASH，OLDA或外部EBU空间。

2、支持重定向到覆盖内存位于:

-本地内存(LMU)(如果存在)

-仿真内存(仅限仿真设备)

-DSPR或PSPR内存

3、支持多达4 MB的覆盖内存地址范围

4、在每个TriCore实例中可以使用最多32个覆盖范围(“块”)。

5、overlay块大小从32字节到128 k字节

6、多达4 MB的重定向空间(32个区间x 128 k字节)

7、对每个overlay块单独选择overlay内存位置和块大小;

8、可以通过一个寄存器写入访问启用或禁用多个overlay块;

  SCU_OVCCON.U = 0x00050007;/*Overlay Start, OVSTRT = 1, OVSTP = 0*/

所属寄存器章节位置

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\ccc26681f2384c08b9ac83e71c1bbcb7.png)

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\5cb75a3ce28a4c0d82486b2cae3d0818.png)

 注意一个名词DMI: 5.11 章节 Data Memory Interface (DMI)

9、在DMI中控制数据缓存的可编程刷新(无效)控制

10、overlay开始/停止同步数据负载

11、每个核都有独立的overlay系统

### 2.2、数据重定向

​    从原始目标存储器(“目标地址PFLASH”)重定向数据访问到overlay存储器(“重定向地址RAM”)的原理如下所示。

​    简而言之就是将已有的PF目标地址重定向到其他位置（重定向地址RAM）提供XCPO的工作页。

​    ![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\7ba7ff45ca6f44b3ab13721394e1ab50.png)

 从上图可以看出，overlay Memory定义的时是RAM区域。

数据访问overlay是通过overlay范围定义的，每个overlay块定义了一个连续的地址空间范围，访问被重定向。每个overlay块配置如下参数:

1、overlay块目标基地址-目标地址范围的开始地址（PF）;

2、overlay块大小-地址的大小将被重定向

3、overlay块重定向地址-重新定向的开始地址(RAM)。

在TC27x中,可以在每个TriCore实例中使用最多32个覆盖范围

每个覆盖块有3个相关寄存器用于独立配置这些参数。覆盖参数配置如下:

·目标基本地址配置为OTARx寄存器：一般是PFLASH

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\07a23ccf2d76491eb3b26c00685b5089.png)

 ·overlay块大小配置了OMASKx寄存器：对应位是1的地方让定向的地址与基地址作比较，要是要定向的地址与基地址一样可以重定向，不一样不可以重定向，下文有描述

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\854382c760c640f1b12c718ef060948e.png)

 ·重定向地址配置为RABRx寄存器：一般是RAM

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\d9534128af204b6fad5af2f13f31e79f.png)

​     覆盖内存块的大小可以是2n x 32字节（这里是2的n次方，下文有解释来源），n = 0到12。这给出了块大小从32字节到128 kb的范围。块的起始地址只能是已编程块大小的整数倍(自然对齐边界)。如果OTAR寄存器值或RABR寄存器值与块大小不一致，则忽略最低有效位值并将其视为零。

   重定向地址由RABRx寄存器的两个字段决定:

1、RABRx.OMEM 选择overlay内存

2、RABRx.OBASE选择基地址内存

​    通过RABRx.OVEN位每个overlay块可以单独激活或者去激，也可以组合形式的实现。

地址重定向过程如下图所示。

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\716554c3ff5f4a7e8d7b54b714db7f2f.png)

​     对段8H或段AH的任何数据访问都要对照所有激活的overlay blocks进行检查。对于每个激活的overlay blocks ,地址位27...5与目标base地址(OTARx)进行比较,这种位比较由OMASKx寄存器的内容限定（限定的是块大小）。如果相应的OMASKx位设置为1 ,则地址位参与比较。如果当前访问的地址中OMASKx为1的位的值等于OTARx寄存器中的相应位,则访问被重定向。（下面先简单解释一下，后面例子里面会详细解释）。  

​    段8H或段AH ：指PF内存的高位，例如PF 0x8000 0000 （携带Cache）与0xA000 0000 （不携带Cache）。此处有没有Cache选择关联着OVCCON.DCINVAL

​    地址位27...5与目标base地址(OTARx)进行比较：理解上其实就是将基地址的数据27...5填入Tbase位域，举例如下

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\0b781b7da710468898b9c610ff66487d.png)

 再举个其他地方看到的例子：

​    我们设定的flash base地址为0x80180000,size为32Kbyetes,对应的mask位5-16位的值110000000000 ,当访问的内存地址为0x80180004,此时为1的位都相等(15,16位都为0，与基地址的是相等的)，会进行重定向，当访问的内存地址为0x80189000时，mask为1的位不相等(当前访问的15位为1，设定的为0),不会进行重定向，从这个例子可以看出，想重定向，需要再OMASK为1的位置，需要重定向的地址与设定的地址bit位一致，其实也就是范围上的限制。

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\f227c986346c4e50aff7b0c268613e4a.png)

**重定向地址如下:** 

1、地址位31…22bit的设置根据overlay内存选择（RABRx.OMEM）与原始地址的缓存能力，也就是Cache。

2、对于地址位21…5

·如果设置了相应的OMASKx位，则地址位的值取自RABRx.OBASE位域;

·如果清除对应的OMASKx位，则地址位的值取自原地址。

·地址位4..0总是直接取自原始地址。

| 31…22                                                        | 21…5                | 4…0              |
| ------------------------------------------------------------ | ------------------- | ---------------- |
| 举例：RABRx.OMEM = 8为LMU段地址开始B/9000 0000H假设为B000 0000前面的10个bit拆解1011 0000 00 B  0 | RABRx.OBASE位域的值 | 直接取原地址的值 |
| 决定地址的寄存器主要是：OMASKRABR                            |                     |                  |

举个其他地方看到的例子

​    目标地址0x80180000和size为32Kbyetes,我们设置重定向base地址为0x60008000(为core1的ram,需要设置RABRx.OMEM为1，也就是Core1 DSPR/PSPR,对应的RAM位0x6000 0000开头的地方)，设置RABRx.OBASE为0x400，若访问的内存为0x80180004,则重定向地址最高位为6 ( core1 DSPR/PSPR ) ，重定向地址为0x60080004。

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\ed9bce6b98054b49b4288a56626fa65b.png)

### 2.3、目标内存

​     对段8H或段AH内任何内存的数据访问可以重定向到overlay内存。特别是，对以下内存的访问可能会被重新定向:

• Program Flash;
• Data Flash;
• OLDA space;
• External EBU space

**在线数据采集**

虚拟OLDA内存范围还支持校准。虚拟OLDA内存范围的基址为A/8FE7 0000H。

### 2.4、OverLay内存

​    在下面,描述了所有支持的Overlay内存。注意,依赖于设备类型的仅是列出的覆盖内存的子集。

Overlay内存通过寄存器RABRx.OMEM位域进行独立选择的。

***\*Local Memory\****

​    如果存在，可以选择本地内存(LMU)进行Overlay。Overlay块x重定向选择本地内存，如果是RABRx.OMEM值为6。LMU的基址为B/9000 0000H。在地址转换过程中，前10位地址位被设置为B0H00B(目标段A)或90H00B(目标段8H)。

***\*Emulation Memory\****

​    如果存在，可以选择仿真内存(EMEM)进行Overlay。Overlayx重定向选择仿真内存，如果是RABRx.OMEM值为7。仿真内存的基址为B/9F00 0000。在地址转换过程中，前10位地址位被设置为BFH00B(目标段Au)或9FH00B(目标段8H)。

***\*DSPR & PSPR Memory\****

​    核0、1、2的数据暂存缓存器（DSPR）与程序暂存缓存器（PSPR）可以被选择为Overlay，DSPR或PSPR为叠加块x重定向选择,假设RABRx.OMEM值为0，1，2。基地址是 7000 0000H,6000 0000H, or 5000 0000H. 在地址传输期间,高10位地址设置为70H00B,60H00B,或50H00B。基于RABRx.OBASE位的值DSPR（偏移地址是0）PSPR（偏移地址10 0000H）可以供选择。

### 2.5、全局OverLay控制

​    Overlay可以为每个Core单独禁用或启用,使用OVCENABLE寄存器实现。如果OVCENABLE.OVENx位被清除后,无论剩下的寄存器设置如何，Core x上都不允许地址重定向。写入OVCENABLE寄存器不会改变任何剩余的寄存器值。

​    每个OverLay块都可以通过位RABRx.OVEN单独激活与停用，有一个专用的功能，可以激活与停用多个块。这有助于在多个内存区域维护数据一致性，为了实现并发激活和停用OverLay块, 分两个阶段选择OverLay块

1、使用OVCx_OSEL寄存器分别为每个Core x选择激活和去激活的单独块;

2、通过寄存器的OVCCON.CSEL位选择核。

​    可以通过寄存器位OVCCON.OVSTRT来控制多个块的激活与停用。当OVCCON.OVSTRT设置为1时。 表示处于激活状态。

·当OVCCON.CSELx = 1、OVCx_OSEL.SHOVENy = 1  OverLay块Y在Core x里面被激活，此时应该设置OVCx_RABRy.OVEN = 1。

·当OVCCON.CSELx = 1、OVCx_OSEL.SHOVENy = 0  OverLay块Y在Core x里面被停用，此时应该设置OVCx_RABRy.OVEN = 0。

·当OVCCON.CSELx = 0，OverLay在Core x里面的设置是无效的。

​    上面列出的操作是并发（同时）执行的。否则不会改变覆盖配置。有了这个功能，可以直接从一组覆盖块切换到另一组覆盖块。

多个OverLay块可以被同时停用，通过设置寄存器位OVCCON.OVSTP。当OVCCON.OVSTP = 1时。

·当OVCCON.CSELx = 1，所有的OverLay块在Core x都会被停用，所有的OVCx_RABRy.OVEN将会被清除。

·当OVCCON.CSELx = 0，OverLay在Core x里面的设置是无效的。

上面列出的操作是并发执行的。否则不会改变覆盖配置。

注意！注意！注意！

​    **假设通过寄存器RABRx.OVEN控制单个块，就不应该使用全局寄存器OVCENABLE，OVSTRT与OVSTP是处理并发情况的。**

​    当寄存器OVCCON.DCINVAL = 1，也就是使能Dcache时，所有在Cache里面未修改的数据均是无效的。已修改的数据不受影响。没有设置OVCCON.CSEL位的核是不受影响的（核都没选择，那肯定不起作用）。数据缓存失效可以与OVSTP或OVSTP操作相结合。此函数有助于确保CPU在激活或停用覆盖块后访问新数据。

注意！注意！注意！

​    **CSEL, OVSTRT, OVSTP and DCINVAL在同一个寄存器里面，\**不保留写入值，意味着读取的时候始终是0\****

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\13d18942ce704dd79e4b3d304a5698c3.png)

​    OVCCON.OVCONF位是预留的用户控制位，与OVCCON.POVCONF一样，不影响OverLay的功能。

​    当OVCCON寄存器的CSEL与OVSTRT或OVSTP被设置了，与此同时OVCx_RABRy寄存器被写入，OVCx_RABRy.OVEN的值将不会被定义，应该避免这种情况同时发生。

​    在Core的IDLE状态OVSTRT、OVSTP、DCINVAL将不会被执行。

**全局OverLay同步设置**

​    当请求OVSTRT, OVSTP或DCINVAL动作时，其执行可能会延迟，以防止在正在进行的数据负载期间更改OverLay配置。

​    在请求一个动作之后，在请求另一个动作之前，应该有足够的时间。如果一个新的操作被请求，而之前的操作仍然挂起(由于与CPU负载同步)，一些操作可能会丢失。

### 2.6、OverLay配置的更改

​    在操作OTARx,OMASKx或RABRx寄存器之前应该通过清除它的RABRx.OVEN位，禁用OverLay块。否则，可能会发生意外的访问重定向。只有在目标地址、覆盖内存选择、重定向地址和掩码都配置了预期值的情况下，才应该启用覆盖块。

注意！注意！注意！

​    OverLay控制不能防止错误地配置转换逻辑。特别是，不阻止重定向到未实现或禁止的地址范围。

​    如果需要数据一致性，则需要特别注意将OverLay重定向更改同步到执行的指令流。

​    外部访问可以在CPU中进行缓冲。尚未完成的外部访问仍可能受到OverLay配置更改的影响。因此，建议在任何OverLay范围被激活或去激活之前，确保完成所有挂起的访问(例如，通过执行DSYNC指令)。

​    当启用OverLay块并且通过目标地址空间写入相同的内存位置并通过重定向地址空间读取相同的内存位置时，反之亦然，则需要强制执行访问同步(如果适用，则使用DSYNC和数据缓存回写)。

### 2.7、访问保护、属性、并发匹配

当使用Overlay重定向数据访问时，使用的访问保护如下:

·目标地址受CPU内存保护(MPU)验证;

·重定向地址受安全保护验证。（是OVCCON的保护寄存器？）

重定向访问的物理内存属性是根据目标地址段(参见CPU物理内存属性章节)。

​    不支持多个已启用的Overlay块中的并发匹配。当一个地址与两个或多个已启用的Overlay块匹配时，将引发异常并且不执行内存访问。在Overlay范围上具有多个匹配的负载操作会引发数据访问同步错误(DSE)陷阱，而存储操作会引发数据访问异步错误(DAE)陷阱。在这种情况下，相关的trap信息寄存器:数据同步trap寄存器(DSTR)，数据异步trap寄存器(DATR)和数据错误地址寄存器(DEADD)被更新。

### 2.8、Overlay控制寄存器

​    OVC块控制寄存器位于支持数据访问Overlay的每个模块中。OVC全局控制寄存器位于SCU中。OVC寄存器访问可以通过安全寄存器保护来限制。

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\7724b7b38e3d4b5a80a97acee1fb93ea.png)

寄存器的基地址如下

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\4f81488880c54935be87b93fd3afc932.png)

 偏移量如下

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\835b7adfae1e400082178313f6ef1080.png)

符号P:带安全保护的写访问

符号SV:管理员模式下允许访问

符号U:允许在用户模式0或1下访问

符号32:只允许32位字访问

### 2.9、模块控制寄存器

​    对于32个Overlay内存块中的每一个(由索引x表示)，三个寄存器控制Overlay操作和内存选择:

·RABRx,设置Overlay内存，包含了激活位，此处假设是常规使用英飞凌单片机位RAM区域地址设置。

·OTARx，Overlay目标地址寄存器，显示OverLay内存的基地址

·OMASKx，它决定哪些位（来源于RABRx）作为基地址（对Overlay的内存以及块）与哪个位(原始数据地址)直接被用作块中的偏移量。

​    此外，OverLay范围选择寄存器OSEL决定在OVCCON时启用哪些块，禁用哪些块。设置了OVSTRT位。

​    随着应用程序复位，所有OverLay块控制寄存器被重置为其默认值。不考虑特殊的调试重置。   ![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\302495b2f8b44546b77eab69f70b67d8.png) ![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\f4fd5c9dcca84f66a9fe32c01d6c92e3.png)

 ***\*CSELx\*******\*位存在疑问，此处是为了以此控制多个\*******\*OverLay\*******\*而存在还是因为其他的呢？\****

 ![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\342f738b07924ad6bcb94be1b5793cb3.png)

## 3、OverLay代码解析

下面例子与测试使用的非一套代码，但是都是有效代码，注意识别

### 3.1、初始化

```c
void Xcp_OverlayMem_Init(void)
{
	/* The calibration size is 24KB,
	 * Flash CAL: 0xAF05_8000H - 0xAF05_DFFFH, 24KB (16KB, 0xAF05_8000 - 0xAF05BFFF; 8KB, 0xAF05C000 - 0xAF05DFFF)
	 * Ram CAL  : 0x7000_0000H - 0x7000_5FFFH, 24KB (16KB, 0x7000_0000 - 0x70003FFF; 8KB, 0x70004000 - 0x70005FFF)
	 */
	/* The calibration size is 24KB, but supported overlay block are 8,16,32,64KB...
	 * Here 2 overlay blocks are configured
	 * Block 0: size 16KB
	 * Block 1: size 8KB
	 */
	Mcal_ResetCpuENDINIT(-1);
	Mcal_ResetCpuENDINIT(0);
	SCU_OVCENABLE.B.OVEN0 = 1;
    /*假设此处需要处理其他两个核，
    * 开启其他核即可，另外下面的地址与OMASK配置也是开启其他核就行
    *SCU_OVCENABLE.B.OVEN1 = 1;
    *SCU_OVCENABLE.B.OVEN2 = 1;
    */
	Mcal_SetCpuENDINIT(0);
	Mcal_SetCpuENDINIT(-1);
	/* ----------------------------Start Configuration for Block 0 -----------------------------------*/
	OVC0_OSEL.B.SHOVEN0 	= 1; /* Enable overlay on Block 0 */
	OVC0_OMASK0.B.OMASK 	=  0xE00; /* 1110 0000 0000B, 16K block size */
	/* Base Address
	 * 								  |<-- OMASK--->|
	 * 0xAF05_8000= 1010|1111|0000|0101|1000|0000|0000|0000
	 * OMASK      = 0000|1111|1111|1111|1100|0000|0000|0000 (0xE00)
	 * TBASE      = ****|1111|0000|0101|1000|0000|000*|****
	 * 			  = 	  111 1000 0010 1100 0000 0000 = 0x782C00
	 */
	OVC0_BLK0_OTAR.B.TBASE 	= 0x782C00;
	/* Redirection to Core 0 DSPR/PSPR memory for Block 0 - 4kB */
	OVC0_BLK0_RABR.B.OMEM 	= 0x0;  /* 0, Core 0 DSPR; 1, Core 1 DSPR; 6 - Redirect to LMU memory; 7 - Redirection to EMEM; 3..5H Reserved, do not use */
	/* Overlay Address
	 * 								  |<-- OMASK--->|
	 * 0x70000000 = 0111|0000|0000|0000|0000|0000|0000|0000
	 * OMASK      = 0000|1111|1111|1111|1100|0000|0000|0000 (0xE00)
	 * OBASE      = ****|****|**00|0000|0000|0000|000*|****
	 * 			  =		        00 0000 0000 0000 000 =0x00
	 */
	OVC0_BLK0_RABR.B.OBASE 	= 0x00; 
	/* ---------------------------- End Configuration for Block 0 -------------------------------------*/
	/* ---------------------------- Start Configuration for Block 1 -----------------------------------*/
	OVC0_OSEL.B.SHOVEN1 	= 1; /* Enable overlay on Block 1 */
	OVC0_OMASK1.B.OMASK 	=  0xF00; /* 111100000000B, 8K block size */
	/* Base Address

	 * 								  |<-- OMASK---->|
	 * BaseAddr   = 1010|1111|0000|0101|1100|0000|0000|0000 (0xAF05C000)
	 * OMASK      = 0000|1111|1111|1111|1110|0000|0000|0000 (0xF00)
	 * TBASE      = ****|1111 0000 0101 1100 0000 000*|****
	 * 			  = 	  111 1000 0010 1110 0000 0000 = 0x782E00
	 */
	OVC0_BLK1_OTAR.B.TBASE 	= 0x782E00;
	/* Redirection to Core 0 DSPR/PSPR memory for Block 1 - 8kB */
	OVC0_BLK1_RABR.B.OMEM 	= 0x0;  /* 0, Core 0 DSPR; 1, Core 1 DSPR; 6 - Redirect to LMU memory; 7 - Redirection to EMEM; 3..5H Reserved, do not use */
	/* Overlay Address
	 * 								  |<-- OMASK---->|
	 * OverAddr   = 0111|0000|0000|0000|0100|0000|0000|0000	(0x70004000)
	 * OMASK      = 0000|1111|1111|1111|1110|0000|0000|0000 (0xF00)
	 * OBASE      = ****|****|**00 0000 0100 0000 000*|****
	 * 			  =		        00 0000 0100 0000 000 =0x200
	 */
	OVC0_BLK1_RABR.B.OBASE 	= 0x200;
	/* ---------------------------- End Configuration for Block 1 -------------------------------------*/
	SCU_OVCCON.B.CSEL0 = 1; /* Select CPU0 */
	SCU_OVCCON.B.DCINVAL = 1; /* only use non-cached access */
	SCU_OVCCON.U |= 0x03000000;
}
```

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\7fb453c144b94626a9235cb6b6553da7.png)

###  3.2、使能与关闭

```c
void Xcp_OverlayMem_SetEnabled(boolean enabled)
{
	if(enabled)
	{

		/* XcpApp_CurrentCalPage is WORKING_CAL_PAGE; */
		SCU_OVCCON.U = 0x00050001;//Overlay Start, OVSTRT = 1,Select CPU0,DCINVAL
		OverlayMem_Flag = TRUE;
	}else
	{
		/* XcpApp_CurrentCalPage is REFERENCE_CAL_PAGE; */
		SCU_OVCCON.U = 0x00060001; //Overlay Stop, OVSTP = 1,Select CPU0,DCINVAL
		OverlayMem_Flag = FALSE;
	}
}
```

### 3.3、内存同步

```c
void Xcp_OverlayMem_Sync(void)
{
    Xcp_MemCopy((uint32*)CALRAM_START_ADDR,
                (uint32*)CALFLASH_START_ADDR,
                CAL_MEM_SIZE);
}
```

Xcp_MemCopy（）是XCP生成的源码部分

所处位置

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\6cf50d075ea441ba9a24c178960dd9d0.png)

 

## 4、测试结果

1、正常调用XCP的OVERLAY使用

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\cc588222d93c41a5aef72f1331729aba.png)

2、去除同步代码

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\a9a1b52843e1440ba4243e0277f2017e.png)

 未进行同步之前

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\9739d65bf7ff40afbc0a33e253c0053c.png)

 同步之后

![img](C:\Users\Administrator\Desktop\xcp内存重定向.assets\6908cfc6aba5456eb70c793bcdc46094.png)



# tricore 汇总

1：指定输入段

\#if defined(__HIGHTEC__)
\#pragma section
\#pragma section ".start" x    /* hightec 编译器下声明函数 fun() 放入名为 .start 的指定输入段中，除了 .start 外全是关键字，.start 可以随意命名 */
\#endif


\#if defined(__TASKING__)
\#pragma protect on
\#pragma section code "start"  /* tasking 编译器下声明函数 fun() 放入名为 .code.start 的指定输入段中，段名会自动添加 .code. ，除了 start 外全是关键字，start 可以随意命名 */

\#endif


\#if defined(__DCC__)
\#pragma section CODE ".start" X   /* 类似上面 */

\#endif

 

\#if defined(__ghs__)
\#pragma ghs section text=".startup"  /* ghs 编译器下声明函数 fun() 放入名为 .text.startup 的指定输入段中，除了 .startup 外全是关键字，.startup 可以随意命名 */
\#endif

void fun(void)
{
}

if defined(__HIGHTEC__)
\#pragma section               /* hightec 编译器下声明结尾，必须和开头的声明成对存在，全是关键字无须修改 */
\#endif
\#if defined(__TASKING__)
\#pragma protect restore
\#pragma section code restore     /* tasking 编译器下声明结尾，必须和开头的声明成对存在，全是关键字无须修改 */
\#endif
\#if defined(__DCC__)
\#pragma section CODE          /* dcc 编译器下声明结尾，必须和开头的声明成对存在，全是关键字无须修改 */
\#endif
\#if defined(__ghs__)
\#pragma ghs section text=default  /* ghs 编译器下声明结尾，必须和开头的声明成对存在，全是关键字无须修改 */
\#endif

 

 2：在AURIX中每个核心都有自己的DSPR和PSPR，每个区域都可以使用以下两个地址访问；1：全局地址：无论代码在哪个内核上执行，这个地址范围都指向同一个内存；2：本地地址：此地址将引用特定于核心的ram，并将根据执行代码的核心而更改。例如 : CPU0 DSPR从0x70000000开始，CPU1 DSPR从0x60000000开始。在代码中，如果您使用0x70000000，它将引用CPU0 DSPR，而不管访问是来自CPU0还是CPU1，这叫做全局地址；相反，如果您在代码中使用0xD0000000，如果代码从CPU0执行，它将访问0x70000000，如果它从CPU1执行，它将访问0x60000000，这叫做本地地址。提供这种功能是为了使代码相对于cpu具有可移植性。对于DSPR，本地地址从0xD0000000开始，对于PSPR，本地地址从0xC0000000开始

REGION_MAP( CPU0 , ORIGIN(dsram0_local), LENGTH(dsram0_local), ORIGIN(dsram0))  定义dsram0的全局访问和本地访问地址

 

3：CORE_ID = CPU5; + CORE_SEC(.text) => 名为 .CPU5.text 的输出段； 而 CORE_ID = GLOBAL; + CORE_SEC(.text) => 名为 .text 的输出段

 

4："output\objs\asw\ASW_IoHwAb\SensorActuator_ADC.o(.text.ADC_Get_func)"
   "output\objs\asw\ASW_IoHwAb\SensorActuator_DIO.o(.text.DIO_Set_func)"
   "output\objs\asw\ASW_IoHwAb\SensorActuator_DIO.o(.text.DIO_Get_func)"
   通配符查找命令为 ("out*.*text) 其中（）和 *.* 是主要关键字，("out*.*）是替换一整行， (^\n) 替换所有空行

 

5：单步可以而全速运行不行的情况一般出现在时序上，通过加入一小段延时，问题得以解决。而 printf 往往是一个需要阻塞运行很久的一个函数

 

6：提示警告 cast increase required alignment of target type，解决方法为类型强转前加 (void*) 比如 udp_fakehead = (struct udp_fakehead *)(void *)ptr; 

7：![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/3068294-20231007102204258-2070591289.png) trace32软件 Var下的show stack可以显示栈嵌套调用的深度，图中只看黑色的标识(-000 -001 -002)不看那个灰色的标识，标识数最大到002因此嵌套2层，因此当前csa的值为base值70EEC+2。LCX表示csa的上限值，FCX表示csa的当前值，每嵌套调用一层函数该值加1，FCX的最大值是LCX-1，不能等于LCX；本例中FCX的base值是70EEC；PCXI = FCX-1 表示上一层的csa栈值。PCXI中的bit20表示 上文1/下文0 的context；csa值与真实ram的换算：ram = FCX & 0x000F0000 << 12 + FCX & 0xFFFF << 6；

 

8：一个context是64B，一个task的上文+下文=128B，因此1K可以保存128个context或64个task的信息

9：PSW寄存器的bit[6:0]表示调用深度，因此最深128级，因此最多需要128个csa，因此csa最大为8KB

 

10：指令 MFCR 读 核心寄存器值；指令 MTCR 写 核心寄存器值；

11：硬复位和软复位的区别是：硬复位会将cpu寄存器的值重置为复位值，同时将ram置为0；软复位则只会将cpu寄存器的值重置为复位值，ram中的值保持复位前的值不变

 

12：tc397下 分别将一个u32的数放入数据和地址寄存器2中的汇编代码

​    ![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/3068294-20231007103630350-535235632.png)

 

13：tricore 机器码与汇编代码的转换；

这里证明了即使没有编译，汇编，链接器，直接将汇编代码人工翻译为指令机器码，再通过烧录器存入rom中机器也能执行，即纸带代码。这也是代码套娃中从无到有的实现思想。![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/3068294-20231019110810486-752537259.png)

机器码在rom    |    指令机器码        |    人能书写和理解

中的起始地址   |    字节序排列        |    的机器汇编代码

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/3068294-20231019110816440-1908028111.png)  ![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/3068294-20231019113924718-600377801.png)

机器码 JA 后跟 disp24(B) 形式，且机器码 JA 用 9D 表示，因此 disp24[23:16] == 88; disp24[15:0] == 00 80(不是80 00) ; 

但是我们要跳转的地址是80100000，应该是 9D 80 10 01 00 才对呀？原因在于特定于硬件架构的硬件操作，上图粉色标记表示此硬件架构会硬件自动

将24位的数据按上图的格式转为32位数据。类似 x86 下将16位的地址硬件右移4位变为20位的段基地址。

因此 disp[24:0] = 1000 1000 0000 0000 1000 0000 b;

PC = 1000 0000000 1000 0000 0000 1000 00000 b -> 1000 | 0000 | 0001 | 0000 | 0000 | 0001 | 0000 | 0000 b == 0x80100100

# 两种标定方式

### 1.Flash中的标定参数

我们在开发的时候，对于常数其实会用修饰符进行限定的，如下：

```
const uint32 Parameter_A = 1;
```

在进行编译链接之后，编译器会把这个Parameter_A分配到Flash的区间(链接文件里的ROM段)并给定一个地址(通过Map文件可以查找)，同时也从编译出来的hex的地址去找到这个值 1，如下图所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/aHaxFGLTT2UN6LPFrNXFpY7IncsQWibW7tTzGbOP7hibc7fAOKibM9kyAnATxpZKFuYu2tEIg3d8tZSQ2rML8eQOQ/640?wx_fmt=png&from=appmsg)

那么要修改这些参数，应该怎么办呢？有两个办法：

> 1. 通过在源码中修改常数值，重新进行编译，然后刷进ECU里。这样很麻烦，遇到稍微大一点的工程，编译都得十几二十分钟，很显然这种方法是不符合现在的开发流程的。
> 2. 通过FlashDriver对存放Paremeter_A的Flash空间进行重编程，这种方式比上述重新编译要好一点，但是还是实时性不够，并且目前的FLash特性是没有办法按照Byte进行擦除的，意味着要修改一个标定参数就必须擦掉整个Sector，重新刷写所有的标定参数，这显然是不可接受的。当然，如果ECU外挂的EEPROM，那么就不会存在这个问题了，但是访问速度和相应的开发成本也是阻碍参数的实时标定。

另外，还需要提一点的就是，如果编译器优化等级开的比较高，作为常量的Parameter_A有可能会被优化，直接作为一个数值出现嵌入到代码中，而不会出现在map文件里。因此我们在定义标定参数这种类型常量时，通常会按照如下定义：

```
volatile const uint32 Parameter_A = 1;
```

volatile可以有效防止被编译器优化 ；

此外，为了方便管理和能迅速定位到标定参数，我们通常也会在链接文件里定义一块单独的空间，在代码里使用#pragma把参数放到该空间里，如下：

```
#pragma section "Cal_Flash"
const uint32 Parameter_A = 1;
```

这里我们简单讲了标定参数只在Flash里的时候应该如何修改，实际上这种方式并不能支持我们在ECU运行过程中动态修改

所以，我们能不能想个办法把这些参数搬到一段RAM中，在RAM里实时修改，修改完成后再把标好的参数重新刷进Flash。

显然这个想法是成立的。

------

### 2.RAM中的标定参数

我们定义一个带初值的变量，如下图：

```
uint32 Parameter_A = 1;
```

编译器会给这个参数分配RAM空间地址，并且初值存放在Flash中。我做过一个试验，用英飞凌TC2xx系列，在不修改链接文件的情况下，编译器给标定参数分配一个RAM地址，但实际上存在Flash里，如下图：

![img](https://mmbiz.qpic.cn/mmbiz_png/aHaxFGLTT2UN6LPFrNXFpY7IncsQWibW7p2sZOIicU1NeicrcMs38xicMMvp0ib9iad5KDpiczTugicxKglo39Z8rh0KwQ/640?wx_fmt=png&from=appmsg)

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

这是怎么实现的呢？我们查看链接文件关于.data段的定义： 

![img](https://mmbiz.qpic.cn/mmbiz_png/aHaxFGLTT2UN6LPFrNXFpY7IncsQWibW7tBnXxMXTqG8rqLxFd4Z2ZUcCbMmssFKCQIWRCqkMGTicJ07gD7wEOicw/640?wx_fmt=png&from=appmsg)



这行代码的含义就是加载运行在RAM(DSPR)，但是初值存放在PFlash，上电时启动代码会将Flash中的值拷贝到RAM里。

控制算法去获取Parameter_A的值也是从RAM去拿值的，所以这种情况我们就可以在ECU运行过程中动态修改标定参数的值。

这个时候还不是完全体，通常我习惯是拿出一段单独的Flash空间方便标定数据的管理，借鉴上面链接文件的修改，就可以在RAM和Flash单独定义一块空间专门给标定使用，因此在链接文件里有了如下定义:

![img](https://mmbiz.qpic.cn/mmbiz_png/aHaxFGLTT2UN6LPFrNXFpY7IncsQWibW7JnQMjgDCUl0rLbbu8gczWM1Wxlw0enVl1Ufz4xB0XraVr2Q5nywOUw/640?wx_fmt=png&from=appmsg)


\#pragma section "calDataOvc"

```
const uint32 Parameter_A = 1;
```

但是这样又有一个问题，因为标定数据的地址是RAM地址，那么使用标定上位机导出的标定数据hex文件还是RAM的空间。

这个时候我们想要把标定数据刷到Flash，还需要对hex文件做一个地址的偏移，最后通过UDS或者XCP自带编程指令集进行刷写。



# 汽车标定技术--标定量与#pragma的趣事

在之前不会使用overlay机制的时候，我们想要做汽车标定，标定常量编译出来的地址一般都应该是ram的地址，而且在链接文件中都会指定一段区域来存放标定量和观测量。

那么为什么要提出这样奇怪的问题呢？

起因是在向客户询问标定量存放在在ram的哪个位置时，客户说不需要指定特定的段。

这就有点疑惑了，在标定中明确说了，标定过程会分为两页：workingpage和referencepage；WP:可以进行数据修改的页，通常是ram段；RP：不能修改的页，通常是flash段；在ETAS的文档里更明确的表示：

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)



所以不指定位置的话，标定数据会存放到哪里呢？那么我用#pragma来做了如下试验，分享给大家。



01

—

### 不添加#pragma

不添加上述语句，则不指定标定数据具体会放在什么位置；

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

经过编译（此时未给变量分配地址）、链接（分配地址）之后，结果如下： 

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

可以看到，编译器将变量放在了0xd0000840这个位置，结合ld文件 

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

发现它把变量放在了DMI_DSPR（ram）段；所以有理由来谈谈变量在MCU放的位置：

MCU能存放数据的地方有三处：register、rom、ram，涉及到预定义的：

> .text段  ：存放代码
>
> .rodata段 ：存放只读数据
>
> .noinit段  ：存放不需要初始化数据
>
> .bss段  ：存放默认初始化数据（一般为0）
>
> .data段  ：存放已初始化数据
>
> CSTACK段 ：栈
>
> HEAP段  ：堆

​     下面来看一些变量的例子：

| 属性                | 位置                                                         | 操作                                                         | 举例                                                         |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 宏变量              | 预编译期间被汇编进.text段；                                  | 运行已不存在                                                 | ![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640) |
| 常量                | 放在.rodata段                                                | 程序访问在.rodata读取                                        | ![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640) |
| 未初始化全局变量    | 放在.noinit段；一般在ram                                     | 在.noinit                                                    | _no_init uint32_t ni_global_var;                             |
| 默认0初始化全局变量 | 存放在.bss段；一般放在ram                                    | 启动时将bss清零；程序访问时在.bss段存取                      | ![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640) |
| 已初始化全局变量    | 初始化值存放在.data_init段，一般是ROM；变量本身是存在.data段，一般放在ram | 启动时将初值从.data_init段复制到.data段；程序访问时均是在.data段存取 | ![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640) |

 注：（1）观测量是放在.bss段的

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

（2）标定量本身应该是放在.data段的

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

可以看到，calibration1变量本身是放在ram里的，在程序上电但未运行时，ram里肯定是为0的，所以必须有一个从rom把值拷贝到ram指定位置的操作：

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

根据链接文件里，可以看到，DMI_DSPR是从PFLASH1l里读取值，所以有理由相信，在未指定ram区域给标定量时，**初始化值**是存在PFLASH1，且变量本身是放在ram里，位置由链接文件指定。那么这个值是存在flash里的具体位置应该如何找：

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

可以看到.data_start是从0x802a20a8开始，那么0x802a20a8肯定是calibra1的初始值：[1,1,1,1,1]，查看hex文件里：

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

------

02

—

### 添加#pragma语句

在添加上述语句之后，正常情况下标定量和观测量是会放到我们指定的区间的，

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)



通过链接文件给标定量分别划分了ram区和flash区：

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

在这里，给标定量划分的flash区间是：0Xaf004000，共80K；给标定量划分的RAM区间是：0x60000000，也是80k；

同时也给观测量划分了ram区间是：0x60015000，共4K。

那么现在就看如何将标定量观测量放到指定区间了；

首先看看结果：

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)



calibration1被放到了区域：.calDataOvc；这是一块什么区域呢？来看看链接文件进一步解释： 

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)



可以看到，在单片机开始运行之后，单片机会把RP_CAL0中的值复制到WP_CAL0，并且是变量名和值是一一对应。

此时我们来看hex文件，在AF004000处应该是calibration1的初值：[1,1,1,1,1]

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

 

------

03

—

### 不指定ram空间

在链接文件这样写：

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)



因为没有指定映射到ram的具体地址，所以在map文件里会出现如下现象：

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

对比加上>WP_CAL0 AT>RP_CAL0，

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)



可以看到，这个变量本身是放在flash里的，也就起不到标定的作用了。

而标定量初始值如下，没有变化：

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)



为了比较，不修改rpcal1，如下：

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)



在相应位置添加#pragma语句： 

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

编译之后，在map文件中，calibration4的位置在60001000；

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

而在hex中，af005000能找到其初始值： 

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

也就是说，通过语句> WP_CAL1 AT > RP_CAL1，将标定量本身放在了ram里，标定量初始值放在了flash里，在程序上电运行后，通过CALINIT函数把flash的值拷贝到指定的ram区；

当然也有直接在内核初始化的时候将flash的值copy到ram里（hightec的ld文件）；

这里就要修改链接文件，如下图：

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)

使用copy_table函数，将指定的flash段的数据拷贝至与之匹配的ram段；

Copy_table函数是在mcal的coreinit函数里；

![img](xcp%E6%A0%87%E5%AE%9A%EF%BC%88%E4%BD%BF%E7%94%A8%E3%80%81%E5%86%85%E5%AD%98%E9%87%8D%E5%AE%9A%E5%90%91%E3%80%81lsl%E4%BF%AE%E6%94%B9%E7%AD%89%EF%BC%89.assets/640)



------

04

—

### 总结

从以上结果来看，如果只是给标定量确定了flash的位置和大小，而不确定ram的大小，那么编译器会直接把标定量本身以及值都会存放在指定的flash里面，并且无法映射到ram，因为没有做这个操作；所以需要给ram去指定一个区间存放变量名，把值放到flash；这样我们就能做标定操作了。