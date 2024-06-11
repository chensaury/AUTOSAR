# AUTOSAR实战教程 - 通信协议栈CAN_CANIF_PDUR_CANTP_COM_XCP_ECUC配置一网打尽

通讯[协议栈](https://so.csdn.net/so/search?q=协议栈&spm=1001.2101.3001.7020)几乎是CP AUTOSAR中最庞杂的一块。由于其涉及的模块比较多(仅实现CAN信号的收发就需要ECUC/CAN/CANIF/CANTP/PDUR/COM/XCP这么多模块的协作！)，且名词概念众多，入门很难。网络上关于各个模块的详细介绍浩如烟海，其深度也让人叹为观止。但没有一篇文章把这些模块串起来！

这就导致对于初学者来说，往往耐心的把各个模块的详细介绍都看完，甚至把[AUTOSAR](https://so.csdn.net/so/search?q=AUTOSAR&spm=1001.2101.3001.7020)标准文档读完，依然不能建立一个全局的思路。导致在配置通讯协议栈时候，导入DBC之后，一看那么多错误，无从下手或者解决了CANIF的错误，PDUR又出现了新的错误提示，解决了PDUR错误，ECUC又报错...按下葫芦浮起瓢(就像当下的防疫形势)，这种窘境，我相信绝对是每个AUTOSAR初学者都遇到过的。

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/63abd4ca32aa44c9bda2c8a64b433622.png)

本文试图从一个全局的高度，自顶向下逐步细化开来。让你在建立全局观之后熟悉通讯协议栈各模块之间的关联然后高屋建瓴学会配置每一项！也希望在此抛砖引玉，彼此交流心得,共同进步。

------



## DBC属性与信号流

### DBC属性决定报文类型

不同的DBC属性决定不同功能的报文, 一般实际项目中涉及的报文为4类:应用报文，诊断报文，网络管理报文，[XCP](https://so.csdn.net/so/search?q=XCP&spm=1001.2101.3001.7020)报文。**不同作用的报文其在协议栈中的信号流路径是不同的**。

参考Vector给出的《TechnicalReference_DbcRules_Vector》文档，在DBC文件中对关键属性Attributes的规定如下。

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/0be0f5deca6c47ea9de468c9af550fa9.png)



- **应用报文**：GenMsgILSupport:Yes
- **网络管理报文**:NmAsrMessage:Yes
- **XCP报文**：

1. 根据《TechnicalReference_DbcRules_Vector》规定只要Message中含有大写XCP字样，即可在导入DBC后被Vector的工具自动识别为XCP报文。其他属性同“应用报文：GenMsgILSupport:Yes”
2. 如果不用1的方式，也可以在CANIF模块里手动设置其上层模块Upper Layer(PduUserTxConfirmationUL)为XCP模块。其他属性同“应用报文：GenMsgILSupport:Yes”

- **诊断报文**：

1. 功能寻址:DiagState:Yes
2. 物理寻址请求:DiagRequest:Yes
3. 物理寻址响应:DiagResponse:Yes(物理寻址和功能寻址的区别请自行摆渡)

### 报文类型决定信号流路径

以TX报文为例：

**普通报文路径：**CAN->CANIF->PDUR->COM

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/a886da6eafdc4f9fabe6fb4b8a3ff196.png)

**诊断报文路径:**CAN->CANIF->CANTP->PDUR->DCM

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/3b804bc654ca476db19d97e92de1e900.png)



**XCP报文路径:**CAN->CANIF->XCP

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/139f063f8ce548ba8c6e754d49cfc82e.png)



**网络管理报文路径:**CAN->CANIF->CANNM

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/4cafc762fb054c93a471e4566094e216.png)

之所以把PDUR标红，是因为在下面的配置中方便我们识别PDUR的相关模块，这个要在PduRBswModules配置项中选择的！从这里也可以直接确定，PDUR的PduRBswModules上下文最多只有CANIF,COM,CANTP,DCM。

------

## 配置实践

DBC如下： 

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/cc06e65c04194f33ab46b6cbe8e2f469.png)

我习惯将DBC中所有报文简单罗列到一个表中，按报文功能进行分类。这样结合上面我们的总结，就对于每个报文的路径有一个全局的了解。如果项目比较大，报文较多的情况，建议将普通报文之外的报文(NM报文,XCP报文,诊断报文)列出来，因为他们特殊啊！

通过观察DBC属性，制作报文分类表格：

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/d630d379ca724eb28510c48a9c254e1b.png)

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/5a836378e82f410da0f3752340af7379.png)

 好，接下来进入我们的实战环节。

导入DBC,Update工程, 现在看工具自动配置中遇到的错误还是比较多的, 所以我们接下来的任务就是将这些模块的错误全部Fixed掉!

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/a34e93ca994e4dbcb38d663d38219916.png)

## 

## 搞定信号路径

------

### ECUC模块

EcucPduCollection这个Container的作用.数据在CAN通信协议栈各层间都是以PDU形式传输的，为了将各层PDU关联起来，则需要定义全局 PDU（Global PDU）。由于全局PDU不属于任何一个标准BSW模块，所 以AUTOSAR提出了一个EcuC模块来收集一些配置信息。在EcuC模块中定义全局PDU时不需要关心其数据类型，只需要定义PDU长度即可。

所以我们先对照DBC对照检查以下,ECUC/EcucPduCollection对各个PDU(PDU是啥?你可以简单理解成一个PDU就对应总线上的一个Message再附上一个地址信息的这么一个玩意--虽然这种说法不准确,但是它能有助于你去理解)的长度定义是否正确,至于长度之外的错误,先忽略之,后面其他模块配好之后,ECUC中相关错误一般就自动消失了.

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/9a6c7c135e2242b7befb7b50e1a8b409.png)

------

### CAN模块

CAN模块是直接面向硬件的, 所以CAN模块主要的配置分2部分:

- **对CAN控制器的配置,包括,参考时钟, 波特率,采样点,帧类型,处理方式Polling/Interrupt;**
- **和CANIF的联系,即对Hoh和MailBox和Filter的配置)**

**CAN控制器的配置**

本阶段我们只关注CAN控制器的配置! (在后面的步骤中再重点配置Hoh和MailBox和Filte,所以本阶段这三方面的错误先忽略!)

CAN控制器的配置还是比较容易的,如果有什么错误一般根据工具里面给出的提示即可轻易解决。这里科普2个基本知识点, 也是CAN模块一个稍微难懂的概念 - CAN的时钟, CAN的重同步和采样点.

**CAN时钟**

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/b0c087020fc242b4bd34f3353738c552.png)

Can/CanConfigSet/CanControllers/Clock Frequecy这个值是从芯片的时钟树分频而来, 在MCAL的MCU模块中指定.

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/1e2a80c149f0407baacbc9e186e30ad7.png)

 /Can/CanGeneral/Clock Divider是对上面Can/CanConfigSet/CanControllers/Clock Frequecy的分频, 他们相除的结果在CanControllerBaudrateConfig/CanBaudrateClock中, 比如

Clock Frequecy = 40M, Clock Divider = 1, 则CanBaudrateClock= 40M = 40000KHz.

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/8f1d9a989ef04ec699fc9adc06469b85.png)

**重同步和采样点**

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/1a7f6116f70b473d87fa4ebe8735ee71.png)

 参考文献《CAN总线学习笔记（5）- CAN通信的位定时与同步》这篇博文有非常详尽的介绍（ 如果是Tir1，一般OEM会给出具体的采样点参数值, Autosar工具也会给出参考值）我在这就蜻蜓点水说以下计算原则。

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/14bf7e6b662946d69de5b78a76ff0e3a.png)

**Sync Seg**(同步段):长度固定为1Tq, 所以配置工具中没有它的配置.

在Vector的配置工具中, 定义Prop+Seg1 = TSeg1, Seg2 = TSeg2,一开始感觉后别扭,后来发现这样也好,计算采样点位置更加方便了,比如采样点为80%:

**(同步段(1) + TSeg1)/(同步段+Tseg1+Tseg2) = 80%,**

如果一个BitTime中Tq总和固定了,比如为16个Tq,

**同步段(1) + TSeg1 + TSeg2 = 16**

根据这个二元一次方程组则很容易算出各段的值.

Sync Seg固定为1, TSeg1 = 11, Seg2 = 4.

**SyncJumpWidth:**它的值是用于调整相位缓冲段1和相位缓冲段2的值, 用于CAN的同步,比如相位缓冲段1向前增长了3个,则相位缓冲段2向后减少3个Tq.---也就是一次同步中相位缓冲段改变的长度.所以Sync Jump Width的设置有2个原则:

Sync Jump Width <= 3,

Sync Jump Width <= Min(Seg1, Seg2), 因为一次同步调整的幅度不能超出相位缓冲段1和2中任意一个!

敲黑板了,下面画重点:

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/626727ea43de426a93d1339d0cd861ca.png)



好了，截止目前，我们把CAN模块的1/2错误都消掉了， 剩下CanHardwareObjects这个容器里面的错误，我们先放下。继续下一步。

------

### CANIF模块

CANIF的配置主要分2部分

- **向上:指定各个PDU的上层模块**
- **向下:对Hoh的配置(配置PDU的HOh,对应MailBox和Bufffer,CAN帧的类型)**

这一步我们只关注它"**向上:指定各个PDU的上层模块**"的功能.

**检查各个PDU的上层模块**

主要配置/CanIf/CanIfInitCfg/CanIfRxPduCfgs和/CanIf/CanIfInitCfg/CanIfTxPduCfgs这两个小container

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/3ac960bddf2348a09fba18543b009b23.png)

结合我们上面讲的知识, 检查Davinci Cfg工具/CANIF/Pdu User Tx/Rx Confirmation UL这个配置项对PDU的上层配置是否正确, 即:

- 诊断报文: CANIF之上是CANTP,(CAN->CANIF->CANTP->PDUR->DCM)
- NM报文:CANIF之上是CANNM,(CAN->CANIF->CANNM)
- XCP报文:CANIF之上是XCP,(CAN->CANIF->XCP)
- 普通报文:CANIF之上是PDUR, (CAN->CANIF->PDUR->COM)

如果出现如下错误：

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/86fc61c4c3584a03bdb05a187619e41a.png)

如果不需要Confirmation功能，则可以将Confirmation UL配置项中设为NONE -- 只要到对应模块中检查该PDU确实存在。比如：普通应用报文PDUa，它的上层应该是PDUR， 我们去PDUR中检查，如果它确实被映射到PDUR中了， 则可以在CANIF中将它的Confirmation UL设为NONE.

该容器(/CanIf/CanIfInitCfg/CanIfRxPduCfgs和/CanIf/CanIfInitCfg/CanIfTxPduCfgs)下其他的一些小错误根据工具提示修改即可.

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/3f5db87db22c4d4da052484d548d168f.png)

剩下的错误在后面的操作中解决。

------

###  XCP模块

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/02a77b5004d94733aee9ffccf505a0b0.png)

主要是配置XCP中用于接收和发送的PDU，如果XcpPdus这一块有错误，则检查你在DBC中和CANIF中指定的XCP收发报文是否已经在XCP中Mapping上了，其他小错误根据提示修改即可。

------

### PDUR模块

PDUR主要有2个作用：对信号的路由，对不同总线信号的网关。

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/9a61438b0ba7466a863c3e3eb3d96e4c.png)

**PduRBswModules指定PDUR的上下文模块**

根据我们上面的描述，PDUR向下向上的模块分别是：

普通报文: CANIF->**PUDR**->COM

诊断报文:CANTP->**PDUR**>DCM

XCP报文和NM报文绕过PDUR。

所以如果你的网咯中没有诊断报文，则PDURBswModules中，PDUR的上下层是CANIF和COM

如果有诊断报文，则PDURBswModules中，PDUR的上下层是CANIF,COM,DCM,CANTP.

**PduRRoutingTables**

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/ce9a45cd835647c8a569d8e16b452eb8.png)

 一般工具自动生成的配置，出现错误就在这三个地方。

**PduR Transmission Confirmation**这个错误主要是由于PDUR的上下层Confirmation没有一致，比如一个TX信号，CANIF中将Confirmation UL指定为PDUR，而在PDUR中将Transmission Confirmation设为False，则自然会报错；又或者在CANIF中将Confirmation UL设为NONE, 而在PDUR中将Transmission Confirmation设为True，则自然会报错。

其他小错误根据提示修改即可。

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/fa7fb98e237c4e3ba1b98398c70ed5da.png)

------

### COM模块

COM模块非常简单,其作用就是将总线上的Msg进行卸货或者装车,装车:将信号组装到Msg里面;卸货:将Msg拆分成一个个的信号,给应用层或者CDD使用.

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/28854211ac0a499db0b41892df0400eb.png)

------



### CANTP模块

因为诊断协议中有多帧连续帧的概念,有些报文一帧是发不完的, 所以CANTp模块的主要作用是对CAN I-PDU进行分段和重新组装，使得I-PDU的长度不大于8个字节，对CAN FD而言，CAN I-PDU不大于64个字节。

这里面的难点应该就是一些时间参数的设定, 这个要结合UDS的14229/15765/11898和主机厂释放的网络规范进行设定.

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/dc44634ba9914bba8b180f55f7dad3ad.png)

------



## 搞定Hoh和MailBox

（有朋友反应这一块有很多错误，好吧，我们先讲这一块）

CAN模块下面的CanHardwareObjects其实就是MailBox，是硬件上的存在。CANIF下面的Hoh包含Hrh(接收)和Hth(发送)是报文收发的句柄，是一个软件概念。

结合我们上面的工作， 我接下来主要是对

- CAN部分MailBox和Filter的配置
- CANIF部分Hoh的配置

------

### CAN模块中MailBox配置

**CanHardwareObjects**

先检查CanHardwareObjects这个容器下面, 检查HardwareObject的数量.注意此时HardwareObject还没有和CANIF中的PDU建立任何关系!--这模块的HardwareObject我习惯叫它MailBox!

根据DBC中Message个数, 设置CAN模块下面每个CanHardwareObjects(就是MailBox)的CanHandleType,设为Full CAN还是Basic CAN.

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/fd1736ee45af49c4a91c1f0dcda6c422.png)



 **Full CAN和Basic CAN**

先说结论:

- Full CAN一个Hoh对应一个MailBox而Basic CAN一个MailBox可以处理多个PDU.
- Full CAN是硬件滤波而Basic CAN软件滤波,因此配成Basic的要设置滤波.
- Full CAN一个Buffer对应一个ID报文,无缓存功能而Basic CAN以FIFO的方式接受特定的多个报文,有缓存功能.

因此:

- **对于诊断报文和NM报文的接收报文必须配置成Basic Can,**
- **其他报文最好配成高效的Full CAN.**

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/dfa96cf78e214fe59e4db5a7ad0fb6f7.png)



关于Full CAN和Basic CAN, 这篇文章讲的很详细[《【AUTOSAR-CAN】CAN的 “BasicCAN架构” 和 “FullCAN架构”》](https://blog.csdn.net/tim_hoven/article/details/115097781), 这里我说一下我的理解, 不一定很准确,但有助于理解.

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/26b0a5e6f04e4b72b2ad3ff0a860e838.png)



如果你在CanHardwareObjects这个容器下面配置的**BasicCAN**个数>1(Tx MailBox>1个**或者**Rx的MailBox>1个)这个时候你应该会遇到一个报错:

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/898049bf8db9406f940f5934651b582d.png)

翻译成人话就是你没有使能Multi BasicCAN或者你么有更高级的授权, 而这个时候你进入CanGeneral这个容器下面却发现不允许使能Multi BasicCAN!!![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/4feba46aa1ca4846ab69547cad44890a.png)

是不是很崩溃?---没关系, 按下面这样做:

将所有Tx的BasicCAN删除到只剩一个, Rx的BasicCAN删除只剩一个,然后命名(随个人喜好)TxBasicCanMailBoxCommon和RxBasicCanMailBoxCommon.**然后设置其Size大小为之前所有BasicCAN的MailBox总和!**

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/85bfddd568304b58a198c01308387f82.png)

最后别忘了给接收的BasicCAN设置滤波,并绑定:

在CanFilterMasks下面设置滤波, 在BasicCAN的MailBox下面设置映射:

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/772b4c7faded4d028b2a884d4592ace8.png)

再科普以下滤波的设置:

**滤波参数**

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/a2bb438042e142caad2d33f63bb9bf7e.png)

 白名单模式计算原则是: **received ID & Mask == Code & Mask.**

有一个简便的方法就是,Code Value里面填写ID大的那个ID值, Mask Value里面填写ID小的那个ID值两个数按位与后的值.

例如:我只想接受0x7DF和0x7D4这两个报文,将其他报文过滤掉. 根据计算公式,对于0x7DF报文, 

0x7DF & 0x7D4 == 0x7DF & 0x7D4

对于0x7D4报文, 0x7D4 & 0x7D4 == 0x7DF & 0x7D4

好了,纵然现在千般错, 先放过.去CANIF模块!

------

### CANIF模块中的PDU(Rx和Tx PDU)

进入/CanIf/CanIfInitCfg/CanIfInitHohCfgs/CanIfInitHohCfg/CanIfHrhCfgs这个下面,

**将诊断Rx PDU和网络管理的Rx PDU(他们是Basic Can)都映射到CAN模块下面的RxBasicCanMailBoxCommon上!并勾选CanIfHrhSoftwareFilter.**

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/c4862c5a7c934455bfd11e16404f50a5.png)



**将XCP报文和普通应用报文与CAN模块下面的MailBox进行一对一映射!--因为他们是FULL CAN!**

**并取消CanIfHrhSoftwareFilter.**

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/4f7c88136d834344b361f171cf263ab1.png)

进入/CanIf/CanIfInitCfg/CanIfInitHohCfgs/CanIfInitHohCfg/CanIfHthCfgs这个下面,安装上面的步骤操作即可!

接下来为Tx的PDU配置Buffer即可!

其他一些错误根据工具提示修复即可.这一块相互绑定关系我做个图谱:

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/cdac073990524bf7a16f8169fafbd233.png)



截止目前CAN和CANIF的错误就全部消除了

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/2688ce1550324cfcbd03c653ceea739e.png)

------

参考文献

------



# CAN的 “BasicCAN架构” 和 “FullCAN架构”

（将本文讨论的 “BasicCAN” 和 “FullCAN” 称为 ”BasicCAN[架构](https://so.csdn.net/so/search?q=架构&spm=1001.2101.3001.7020)“ 和 ”FullCAN架构”，具体原因后面解释）

## 1.“BasicCAN架构” 和 “FullCAN架构”

CAN的Basic和Full类型，在配置Can的时候，这个配置项困扰了我很久。

> 摘自 [Specification of CAN Transceiver Driver](https://www.autosar.org/fileadmin/user_upload/standards/classic/20-11/AUTOSAR_SWS_CANTransceiverDriver.pdf) 4.0.3
>
> https://www.autosar.org/fileadmin/user_upload/standards/classic/4-0/AUTOSAR_SWS_CANDriver.pdf
>
> ![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/20210322205000242.png)
>
> Basic：一个Hardware Object可以处理多个L-PDU
>
> Full：一个Hardware Object只可以处理一个L-PDU
>
> 这个参数只会被CanIf使用，用于配置FilterMask和ID。

查了一圈，资料不算多，这个Basic和Full常常会让人和 CAN2.0A 和 CAN2.0B 混淆，然后在这个网站找到了比较靠谱的解释。

> http://www.can-wiki.info/doku.php?id=can_faq:can_faq_basic_full
>
> ![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/20210322210235158.png)
>
> 最开始只有一种CAN Controller，它被设计成了具有一定数量的报文buffer的形式。比如已经作古的Intel 82526（有5个message buffer），以及他的继任者82527拥有15个message buffer。于是飞利浦想着降低成本，就有了只有两个接收buffer和一个发送buffer的82C200，当然也有部分掩码过滤。为了区分这两种CanController，有些人开始叫最开始的为 “FullCAN架构” ，后来的低成本为 “BasicCAN架构” 。
>
> 对于 “FullCAN架构” 还是 “BasicCAN架构” 其实在式样上没有一个明确的定义，取决于生产商。“FullCAN架构” 应该被叫做 “DPRAM” 架构，而 “BasicCAN架构” 则应该被称作“FIFO”架构。并且应当注意“FullCAN”并不是“BasicCAN”的某种完全版本
>
> 原初的82527的实现后来被西门子用在了他们的控制器里，现在可以在C505C/C515C/C164/C167中找到身影。
>
> 新的CanController（BasicCan的那个）扩展了基础功能，比如扩展到了32个buffer，可以用于实现FullCan模式，或者对于几个message相对拥有了较大的FIFO，比如飞利浦的SJA1000的PeliCAN，实现了BasicCAN模式（~实在是太绕了，SJA1000本身有一个BasicCAN模式，是和PeliCAN相对的，结果这个PeliCAN才是”BasicCAN架构“~）。对于大多数的控制器是融合了上面的两种CanController的。他们大多数是以FullCAN来实现的，但是也可以用其中的部分Buffer来实现BasicCAN架构。
>
>  于是就常常会问道，那种CANController更好
>
> “FullCAN”和“BasicCAN”是两种不同的CANController的架构。“FullCAN架构”通常有多于一个的message buffer。可以以这种方式对接收寄存器进行编程，只有一个特定的消息（或一组）传递到该Buffer中。但大多数的实现并不会缓存接收队列的message，意味着后续的message通过接受过滤后会替换之前的，而之前的就会被丢失。如果两个ID相同但是数据不同的message要以很快的速度发送，那么CPU就必须快速的发送出去避免新的message的覆盖
>
> 典型的“BasicCAN架构”控制器（飞利浦的SJA1000）本身具有接收队列，而没有缓存区
>
> 那种架构更好，取决于应用场合。如果只是用少数的message，并且小于所具有的message buffer的话，也许“FullCAN架构”更好。较新的CPU具有两种CANController。另一方面，如果你想看到所有的报文，则FIFO模式的CANController则更好。
>  

大意就是，这个Basic和Full是针对CAN Controller的缓存架构来说的，而 CAN2.0A 和 CAN2.0B 是CAN的通信协议。

"BasicCAN架构“的主要特征是以FIFO的方式buffer特定ID的报文，可以缓存一定的历史报文。

”FullCAN架构“的主要特征是，一个Buffer对应一个ID的报文，而且新的报文会覆盖旧的报文，并不会缓存。

> ![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/20200331110106404.png)
>
> 可以看到只有一个Transmit Buffer和FIFO类型的Recieve Buffer。对于CPU的负担来说稍重 

> ![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/20200331110113677.png)
>
> 有多个message Object Buffer的存在，对于CPU的负担来说稍低。

## 2.回归问题

那么根据实际的项目来看，一般的Com报文都会被要求配置成Full，而诊断的报文不论接收都要配置成Basic模式，然后NM网络管理的报文，接受的要配置成Basic，而发送则是Full和Basic都可以。

结合架构上的区别来看，可能对于一般的Com报文，并不需要历史报文的保留。所以对于某一个ID的报文并不需要buffer它。

而诊断的报文则是遵循诊断的一个要求，首先诊断的报文以接收为例，是只有一个ID，然后基于UDS的协议在这一个ID的报文里做文章，所以如果采用”FullCAN“的新报文覆盖旧报文的架构的话明显是不合理的，而且（具体我还得查证）UDS是有要求接收到的报文都要处理不能丢弃的。

另一方面，网络管理的报文，对于接受来说其实是一个报文区间，”FullCAN“对此也并不合理。而发送目前是配置成了Full，因为对于发送来说就一个特定的ID的报文。理应是Full。

 

## 3.CAN的各种分类

| Classical CAN                                | CAN-FD                                                       |                                 |
| -------------------------------------------- | ------------------------------------------------------------ | ------------------------------- |
| CAN 2.0A                                     | CAN 2.0B                                                     | 不同于Classical CAN的另一个东西 |
| 8Byte数据场，只支持“11bit”的ID，称为标准模式 | 8Byte数据场可以是“11bit”的标准模式，也可以是“29bit”，的扩展模式 | 最大64Byte的数据场              |



## TC397Can_ram资源

3个MCMCAN组件的起始和终止地址为:

| Module | Base Address | End Address |
| ------ | ------------ | ----------- |
| CAN0   | 0xF0200000   | 0xF0208FFF  |
| CAN1   | 0xF0210000   | 0xF0218FFF  |
| CAN2   | 0xF0220000   | 0xF0228FFF  |

其中分配给Message RAM的空间为:

- CAN0 Module的前`0x8000 Bytes => 32768 Bytes` 给了Message RAM
- CAN1 Module的前`0x4000 Bytes => 16384 Bytes` 给了Message RAM
- CAN2 Module的前`0x4000 Bytes => 16384 Bytes` 给了Message RAM

每个MCMCAN组件共4个节点所能使用的最大Message RAM资源如图所示:

![在这里插入图片描述](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/20210225162434398.png)

如果全部元素都启用, 将占用 `128 + 128 + 1152 + 1152 + 1152 + 64 + 576 + 128 = 4480 words = 17920 Bytes = 17.5KB`, 超过了`16384 bytes = 4096 words = 16 KB`的最大Ram限制. 所以还是要有所取舍, 用到的分配, 不用的不分配.

下面会用到的参数总结如下:

- 1 个 11-bit filter element => 1 words, 4个节点的11-bit filter element数量之和不超过128
- 1 个 29-bit filter element => 2 words, 4个节点的29-bit filter element数量之和不超过64
- 1 个 Rx FIFO 0 element => 18 words, 4个节点的Rx FIFO 0 element数量之和不超过64
- 1 个 Tx Buffer element => 18 words, 4个节点的Tx Buffer element数量之和不超过32

`0x4000 Bytes`空间可以对四路CAN节点平均分配, 每路0x1000 Bytes, 所有资源也可以平分, 每路可以分:

- 32 个 11-bit filter element
- 16 个 29-bit filter element
- 16 个 Rx FIFO 0 element
- 16 个 Rx FIFO 1 element
- 8 个 Tx Buffer element
- …

当然一般不这么搞, 比如我这里只用了CAN22, 其它路不用, 完全可以分多点资源来用:

- 32 个 11-bit filter elements

- 16 个 29-bit filter elements

- 20 个 Rx FIFO 0 elements

- 20 个 Tx Buffer elements

  例如使用lld库时的分配操作

```c
#define MODULE_CAN0_RAM    0xF0200000
#define MODULE_CAN1_RAM    0xF0210000
#define MODULE_CAN2_RAM    0xF0220000
#define NODE0_RAM_OFFSET   0x0
#define NODE1_RAM_OFFSET   0x1000
#define NODE2_RAM_OFFSET   0x2000
#define NODE3_RAM_OFFSET   0x3000

g_mcmcan22.canNodeConfig.messageRAM.baseAddress = MODULE_CAN2_RAM;
//Standard Frame, 32 elements => 32 words => 128 Bytes => 0x80
g_mcmcan22.canNodeConfig.messageRAM.standardFilterListStartAddress = 0x0 + NODE2_RAM_OFFSET;
//Extended Frame, 16 elements => 32 words => 128 Bytes => 0x80
g_mcmcan22.canNodeConfig.messageRAM.extendedFilterListStartAddress = 0x80 + NODE2_RAM_OFFSET;
//RxFIFO0, 20  elements => 360 words => 1440 Bytes => 0x5A0
g_mcmcan22.canNodeConfig.messageRAM.rxBuffersStartAddress = 0x100 + NODE2_RAM_OFFSET;
//Tx FIFO Buffers, 20  elements => 360 words => 1440 Bytes => 0x5A0
g_mcmcan22.canNodeConfig.messageRAM.txBuffersStartAddress = 0x6A0 + NODE2_RAM_OFFSET;
//Total: 128 + 128 + 1440 + 1440 = 3136 Bytes => 0xC40 Bytes
```

![在这里插入图片描述](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/20210225162448257.png)









# 一、达芬奇Configurator导入DBC初步

1. 介绍

本文档为[AutoSAR](https://so.csdn.net/so/search?q=AutoSAR&spm=1001.2101.3001.7020)通讯部分配置文档，配置工具为Vector公司DaVinci Configurator Pro。

1. 模块

   1. BSW架构![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/b24d8c8d98bf4e87b5a642272e55a0b3.png)

   

2. 1. 通讯功能

3. CAN通讯，通过接口层到PDU Router模块;（路径：CanDrv--CanIf--PduR--Com）

4. UDS服务，通过接口层到CANTp模块;（路径：CanDrv--CanIf--CanTp--PduR--Dcm）

5. XCP服务，通过接口层到XCP模块。（路径：CanDrv--CanIf--XCP)![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/a9c20c2b33d047748abf80b750030af9.png)



1. 1. Can通讯发送接收流程
2. 应用层Send一个数据进COM
3. COM写信号进PDU Buffer中
4. PDU被PDU Router立刻发送或按周期发送（每个PDU都有一个独立的ID），之后PDU Router辨认总线种类，并把PDU发向不同的下级模块
5. Interface根据不同的通道，把报文写入不同的队列
6. Driver根据报文的优先级立刻发送报文
7. ![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/03b80907c74f4a1589ef7196b8692bec.png)

1. 硬件接收报文
2. 由Driver发出Rx中断（函数），之后通过RxIndication，数据被传递到Interface
3. 传递到PDU Router
4. 传递到COM（如果SWCs使用Data ReceptionTrigger，就通知RTE；否则暂存到Buffer中）
5. 信号被RTE读取，然后应用层读取

![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/46b83eb8b24248948a5736eb642fca1f.png)





1. 1. 各层级间交互

由CAN Driver收取报文生成L-[PDU](https://so.csdn.net/so/search?q=PDU&spm=1001.2101.3001.7020),而后进入CAN Interface进行抽象隔离处理，生成I-PDU，进入PDUR进行分配，根据地址信息（PCI）将I-PDU传入COM，COM对I-PDU的数据信息SDU进行解析，生成signals，signals通过RTE传输给APP层，发送则正好相反。



![https://img-blog.csdnimg.cn/20191125103908121.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5ZnhfZmh3,size_16,color_FFFFFF,t_70](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/20191125103908121.png) 



1. 具体配置

本章节主要为[DaVinci](https://so.csdn.net/so/search?q=DaVinci&spm=1001.2101.3001.7020) Configurator Pro的配置。

1. 1. 新建工程以及EB Mcal的导入

配置顺序：没有严格要求，一般过程是先MCAL，再导入dbc(包含诊断报文)，然后配置COM，CANIF,PDUR,然后再导入诊断cdd数据库，再配dcm，dem。

1. 1. 1. 新建工程

填写相应的工程名、路径和作者等，NEXT

![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/e7ed217dfa674b61b7f3d9cbe7d4cfc6.png)



选择版本和编译工具等信息（Davince会根据这些信息生成动态代码），Next

![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/7e14593e111c48e88ac35e05a9b37817.png) 

![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/11baba8b3ae041f680172d1c81cedfe9.png) 



![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/6e480247dbfd439695408cce8492d699.png) 



1. 1. 1. 导入DBC文件

（Input Files->Open the Input Files Assistant->Add->ECU Instance修改为MyECU（当前ECU的节点名叫MyECU（这个根据DBC文件不同而有差异））->Finish->Update Configuration）

![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/fd81056adf0d4c85935e1a8467d05a7a.png)



![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/e9bb90e875b34b1c97f59da2c8f20553.png) 



导入dbc文件后，自动生成Com、ComM、CanIf、PduR等通信相关的模块的部分配置

（注意：导入时需要将DaVinci Developer软件关闭）



1. 1. 1. 导入EB中生成的Mcal的arxml文件

在导入之前,进入Basic Editor将MCU模块删掉

![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/9ea5a8ab6b3b46a29d9c2d6293b2c8d8.png)



否则会因为DaVinci与EB兼容性的问题，出现两个Mcu（如下图）



![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/2821ef83ec1245d1bfb1f0803767cca4.png) 

File->Import

![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/5c0341ebc610420a93f830b7fc8aee9b.png) 



选择EB中生成的Mcal的arxml文件

![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/5944d871ea474458bf941fc3493579f4.png) 

![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/300fff850731474d938140932373fbc1.png) 

因为是第一次导入，所以全选添加（如果是配置变更的导入，需要将Import Mode由Add改成replace再导入），Finish

![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/ec9e73e889ab4c4e92229759a744804e.png) 

1. 1. 1. 对工程进行配置

对从Mcal导入的模块进行配置，implementation variant按照实际情况修改





 

 

然后点击下方黄色Synchronize now，进行同步

![img](../../AUTOSAR%25E5%25BC%2580%25E5%258F%2591%25E7%25AC%2594%25E8%25AE%25B0/%25E8%25BE%25BE%25E8%258A%25AC%25E5%25A5%2587cfg%25E9%2585%258D%25E7%25BD%25AEcan%25E9%2580%259A%25E4%25BF%25A1%25E7%25AD%2589%25E6%25A8%25A1%25E5%259D%2597.assets/13c46de7741846b3b3664fa51bc5999f.png) 

至此新建工程以及EB MCAL的导入结束  

 

# 二、达芬奇Configurator导入DBC后，配置CAN步骤

接上一篇《[达芬奇Configurator导入DBC初步_dbc导入_Skypine-故都的秋的博客-CSDN博客](https://blog.csdn.net/leiyijing/article/details/127735739)》，导入[DBC](https://so.csdn.net/so/search?q=DBC&spm=1001.2101.3001.7020)后，基本的框架Configurator都已经给你生成好了，下面就按照AUTOSAR通信架构一步一步来配置即可；

1.先配置Controller，Vector工具中，CAN从Driver开始都是由Vector提供的，所以MCAL提供的Driver用不上，但是如果是EB配合[达芬奇](https://so.csdn.net/so/search?q=达芬奇&spm=1001.2101.3001.7020)开发，最好先将EB和MCAL导入到达芬奇的SIP包中，这样就不需要打开EB来回倒ARXML同步MCAL的配置了，只需要在Configurator中配置MCAL即可，怎么导入，可以看官方的帮助文件，有详细的步骤，当然各个SIP包的导入方法不太一样，但是都大同小异，英飞凌的先要安装MCAL到EB比较复杂，NXP的相对简单一些，一键安装，傻瓜式操作，不会的可以私信。

先点开左侧的Communication，然后选择Bus Controller，

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/f52a34f3733f433a98b36e7639af545a.png)



 选中你需要配置的CAN Controller，打开，例子中有两路CAN，一路CAN和CANFD，随便选中一个即可 ，Buss-off Processing、wake-up processing、Rx Processing、Tx processing这些可以按需配置，可以配置成polling，也可以配置成中断，但是汽车电子一般都配置成polling的方式，因为[AUTOSAR](https://so.csdn.net/so/search?q=AUTOSAR&spm=1001.2101.3001.7020)为多任务实时操作系统，如果对于任务之间的交互及共享内存把握不好，很容易产生篡改变量（非重入性问题），汽车电子相对于其它软件，相对来说都是比较注重时序性的，所以最好是配置成polling模式，当然如果是全部用达芬奇Developer开发模型，然后用Configurator和Developer配合集成，没有手写代码，不存在冲入行问题，也可以选择中断，中断实时性较好，且能保证CPU的Load不是太高

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/d004c5746cb14c3f893b4c6aef267e50.png)

然后配置CAN Controller相关的硬件，根据MCU手册和原理图配置，配置收发器的ID、CAN收发管脚等

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/851c79a0e25f409393de708fea9acd7f.png)

配置好上面的参数后，就可以到Basic Editor中去配置波特率、采样点等参数了，这些都比较简单，按照客户需求配置即可，如果想要配置成CAN FD，需要在CANGeneral中去勾选即可

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/53cc98ce767546e2a4d08cda7eb12030.png) ![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/6aef0c7eddb54e93967086db09c231f6.png)

当然，如果是DBC文件做的比较好，CANFD的参数是可以直接生成的，不需要手动去修改，调度周期也可以在CAN General中去修改，目前都是1ms时基，CAN模块基本的配置就这么多，后续继续更新CAN IF、PDUR、CANTP、COM等模块的配置，敬请期待。 





# 三、达芬奇CAN配置----CANIF



CANIF是属于承上启下的一层，为CAN抽象层的范畴，对接MACL的CAN驱动层，

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/a4721117b621468281829b046ca88774.png)

对接CANIF的上层为多种，PDUR、CANTP、CANSM、CANNM、ECUM，具体上层往哪边传，需要你在配置的过程中选，[DBC](https://so.csdn.net/so/search?q=DBC&spm=1001.2101.3001.7020)中对相应的信号也要配置好，如：哪些是诊断报文、哪些是网络管理报文、XCP不需要在DBC中做特殊的处理，在配置的过程中选到XCP协议即可

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/6f6ea9f12d594bdda9a988261fcebcc9.png)

在导入 dbc后，配置完CAN模块，从Communication->PDUs->Modules->CanIf,首先点开CanIfRxPduCfgs，配置接收模块中的每个报文的配置：

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/6ef7e96203c04ffd8da097dd01cd1382.png)

主要是长度检测，Basic和Full 选择，每条报文都必须配置，还有一个比较重要的就是Upper Layer选择，如果是普通的CAN报文，我们都会默认选PDUR，当然你也可以选择 CDD去手动处理，特别是做网关路由开发，有的需要这样选择，至于为什么选择，这个需要单独讨论。

有时候选择upper Layer到CAN TP会报错，选不到，那是因为你的其它模块没有配置好，这个具体问题后面再讨论，CANIF的发送配置也类似，打开CanIfTxPduCfgs，与接收类似，需要配置Tx Buffer，如果选择了Full CAN，tx Buffer则会自动给你生成，这里要注意，需要先在Basic CAN的状态，选择发送的Basic邮箱，再点击Full复选框，这样它就会基于你选的Basic CAN这一路来创建FUll CAN的邮箱，要不然它随机生成的FULL Buffer可能不对，生成到其它路CAN去了，这一点切记

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/ecaffe65b336472ea9894365b500bf5c.png)

 诊断报文配置，我们的项目是CANFD诊断报文，既支持CANFD，又支持CAN诊断，所有会生成两个RX和两个TX，这个不是错误，之前没理解，一直以为是错误，如果只想要CANFD诊断，需要在DBC中修改相应的参数，CANFD-Only打开即可。

![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/6f06e6dbb42d4ea7b2fb39b7a12e3bc5.png)



[XCP](https://so.csdn.net/so/search?q=XCP&spm=1001.2101.3001.7020)配置比较简单，直接upper layer选中到XCP即可。 ![img](Autosar%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE.assets/d317d6f3912b4f009495da296d49aedc.png)

当然配置过程中也可能会出现乱七八糟的错误，具体问题具体分析，总体思路跟大致如此。 







# 傻傻分不清楚的CANFD 一次采样点与二次采样点？

**1.采样点的定义**

采样点是CAN控制器**读取总线电平，并解释各个比特的逻辑值的时间点**。



首先我们需要了解**Tq**的概念，**Tq**是can控制器的**最小时间周期称作时间份额（Time quantum，简称Tq）**,它是通过芯片晶振周期分频而来。传输的个bit位由若干个Tq组成，根据功能传输一个BIT位需要分成四个阶段：**同步段、传输段、相位缓冲段1和相位缓冲段2.**



![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxicrnFjslHVrVEzeF6s2Ngz7Sq4yaJVib5KGZZLTNfdVzLlUlAfIia01cJbtossXozInVNtTpU5ufvg/640?wx_fmt=png&from=appmsg&wxfrom=5&tp=webp&wx_lazy=1&wx_co=1)





这4个阶段的功能如下：



1.**同步段（Sync_Seg）**：用于实现时序调整，总线上各个节点的跳变沿产生在同步段内，通常为1个Tq；



2.**传播段(Prop_Seg)**：用于补偿网络上的物理延迟时间。这些延迟时间包含信号在总线上的传输延迟和CAN节点内部的处理延迟。传播段保证了2倍的信号在总线上的延迟时间；



3.**相位缓冲段1(Phase_Seg1)**和相位缓冲段2(Phase_Seg2)：用于补偿跳变沿的相位误差，其长度会在重同步的实现过程中延长或缩短。



**采样点位于相位缓冲段1的结尾**。由于相位缓冲段1和相位缓冲段2能够延长或缩短，采样点也能够随之变化。



**2.can采样点的计算**

**![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxicrnFjslHVrVEzeF6s2NgzUUCnIMPpicDqeYE11LTYmH92Qb64DxmgmGRwqvqBrpfVMUjlicQicLkRw/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)**



**晶振时钟周期**：是由单片机振荡器的晶振频率决定的，指的是振荡器每震荡一次所消耗的时间长度，也是整个系统中最小的时间单位；



**下面以TC377为例进行计算**:



通过查看配置我们的canfd的，采样分为仲裁场和数据场。仲裁场的波特率为**500Kbit/s**,数据场为**2Mbit/s**，can时钟频率为**4.0E7**。



查看TP377手册Tq = (DBRP + 1) clock cycles



**Tq=(1+1)1/4.0E7（s）=50(ns)**



**仲裁场计算采样点**：



对于仲裁场**500k**，传输一个bit位的时间**1/500000s=2us**,所以分配

**2us/50ns=40个tq**



如果采样点设置为80%，则**sync_seg+prop_seg+phase_seg1=4080%=32Tq**



在ET tresos中可以这样配置



![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxicrnFjslHVrVEzeF6s2Ngzpzf2iceTzE63PO8svOfpMTEa2wEU8icEicibEfN7bibicsnkTobm8E1y60tA/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)



**数据场采样点计算：**

对于数据场**2Mkbs**，传输一个bit位的时间**1/2000000s=500ns**,所以分配

**500ns/50ns=10个tq**,

如果采样点设置为80%，则**sync_seg+prop_seg+phase_seg1=10\*80%=8Tq**

则对应在EB TRESOS中配置如下



![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxicrnFjslHVrVEzeF6s2NgzBFelBqm1fpe2asCk8ibia4bmT8DibjENkjeslaY3z6JCgKSbYkiaQ6k2mQ/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)



**3. CAN-FD与CAN发送速率的不同**

CAN最大传输速率**1Mbps**,CAN-FD速率可变，仲裁比特率最高1Mbps（与CAN相同），数据比特率最高**8Mbps**。**BRS**位速率切换为，**BRS位为0**时CANFD速率保持恒定速率、**BRS位为1**时CANFD的数据段会被切换到高速率。



![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxicrnFjslHVrVEzeF6s2NgzBUrDBvLEYNxDOVjicjEZTMaS8suZGCtvM1JeDIl96CdSkCwJp4Q0N2w/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)



**ESI错误状态指示位**：CAN报文中发送节点的错误状态只有该节点自己知道，CANFD报文中可以通过ESI标志位来告诉其他节点该节点的错误状态，当ESI为1时表示发送节点处于被动错误状态、当ESI为0时表示发送节点处于主动错误状态。



![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxicrnFjslHVrVEzeF6s2Ngz4A2HGlh4qf2gmiaZ0Z8wY4p4ic7eCFJbctgXdiagSsHbvSz3Q8bx0ygWA/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)



**4. 发送延迟补偿**

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxicrnFjslHVrVEzeF6s2NgzVia0T2YQSDrruMiaegjW87qNUtJg2rejlSoVSrJumnO5N2OvXHNMPGfw/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)



CAN控制器发送信号时，是经过收发器后发往CAN总线后，再经过收发器反馈总线信号。那么发送过程中，**控制器发送位信号到接收位信号就不可避免地存在环路延迟**。**发送延迟时间的总和**如下：



1 ). CAN控制器内部产生TX信号到Tx引脚的传播延迟；



2 ). Tx引脚到收发器TxD引脚的传播延迟；



3 ).收发器环路延迟TxD到RxD;



4 ). 收发器RxD引脚到CAN控制器Rx引脚延迟；



5 ).CAN控制器Rx引脚到控制器内部收到Rx信号的延迟



**CAN协议中规定**：发送方发送位时，需检测接收到的位与发送是否一致，若不一致则产生错误帧（位错误）。**如果发送延迟过长，则将直接导致发送与接收位不一致而产生错误帧**。由于传统CAN协议规定最高波特率为**1Mbps**，即位宽**1us**，正常情况下，传输延迟不会超过位宽的采样点（当然具体延迟取决于收发器环路延迟、传输距离、传输线缆质量等），因此不会因为发送延迟而产生错误。



在CANFD中，数据段的波特率是比CAN更高的（BRS位为隐性时），此时**波特率越高，位宽越小，在发送报文时发送延迟影响越大，越容易产生位错误。**由于发送延迟无法避免，此时就需要一种机制来保证发送与接收的位对应上，以避免产生位错误。这种机制就是发送延迟补偿了。



**5. 发送延迟补偿（TDC）**

**TDC****实际上就是在发送BRS位为隐性的CANFD报文时（BRS隐性即开启数据域波特率），在发送时延迟一定时间后，在第二采样点采样接收位，以正确采样到发送位对应的接收位**。



**6.发送延迟测量**

**那么延迟采样的延迟时间是多久呢？**实际上，开启TDC后，控制器将自动测量Tx信号线上FDF位到r0位下降沿与Rx信号线上FDF位到r0位边沿的之间的延迟时间，如下图中所示，TDCV即为延迟时间。发送延迟测量的时间单位为CAN控制器时钟（TDC寄存器中一般对TDCV的值有限制，若超过寄存器最大位数，则发送延迟测量失败）。



![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxicrnFjslHVrVEzeF6s2NgzPqCYibHaLsx26BDD9y3y56KYH7iapnCZxibibMqcIRhAKVOR1xrAL0D11A/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)

不同于采样点，第二采样点在CAN FD控制器接收其他节点发送报文的过程中并不会起到任何作用。第二采样点的作用，是在不改变传输延迟补偿的情况下，实现CAN FD在数据场的位错误检测要求。



根据TC377手册

![图片](https://mmbiz.qpic.cn/mmbiz_png/CJj2CPh6RcxicrnFjslHVrVEzeF6s2Ngzwtao9mn6ZzahPUsOoDqta2EMJ469afxXBQpTLKl8puzv2TtibsfJKkQ/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)



根据手册TDCV是接收到数据时的发送延迟时间，是当TDC使能的时候会自己自动计算的。



TDCO就是设置的正常采样点的时间。

