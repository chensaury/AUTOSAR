# AUTOSAR基础篇之OS(上)

原创 汽车小T8 [ADAS与ECU之吾见](javascript:void(0);) *2021-10-11 08:16*

### 前言

首先，请问大家几个小小的问题，你清楚：

- 为什么汽车电子ECU需要使用OS呢，它的必要性在哪里？
- ECU软件运行过程中是如何实现任务切换的吗？
- 多核系统OS又是如何协同启动或者关闭的呢？
- 。。。。。。

今天，我们来一起探索并回答这些问题。为了便于大家理解，以下是本文的**主题大纲**：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAIhoJwIvTS80jsP69pCsnNEN9rr2GticI9zg9fRfuAeL9H5gEqjfeP5LUfEOaZNCpQleoOQEYn0hA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 正文

#### 为啥要用OS？

我们知道传统所说的“**裸机编程**”就是不带操作系统的编程，在系统需求相对比较简单的情况下使用裸机编程可以满足要求。

但是随着系统需求越来越复杂，此时就需要用到模块化设计方法以及多任务编程思想，否则后期软件升级维护成本将会急剧增加。

虽然我们可以采用传统编程方式（如计数器与状态机）来实现简单多个任务的调度，但是当涉及到多个任务之间的状态切换，优先级，现场保护，执行时间控制等方面就显得极为吃力，开发效率低下且极容易出错。

此时迫切需要一种机制来替我们完成各个任务之间的调度功能，使得开发人员能够更关注于应用软件的开发，提高软件开发效率。

为此**OS(Operation System)**便应运而生！实际上，OS主要是为我们解决了以下几个基本问题：

- 改变各任务的执行频率；
- 改变各任务的执行时间；
- 设定各任务的优先级，保证高优先级任务能够及时执行；
- 任务切换时的现场保护与恢复；
- 共享资源的安全访问机制等；

其中调度功能则是OS的核心组件，主要功能就是负责任务的切换。在一个单核系统中，多任务只能并发执行，通过**时间片轮转法**来切换任务。

通过时钟中断或者软中断的方式来触发一次任务的切换，从而打断当前执行任务，调度器抢夺CPU控制器，来进行任务调度并切换至新任务开始执行，如下图1所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAIhoJwIvTS80jsP69pCsnNuKpYIiaxwTaHBFgoFADZKFuoVe2UxMjw6vLRlEVia3DyfZt6yzzXmVMg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 OS调度器功能

对于传统汽车电子开发领域，早期使用的OS则是**OSEK OS**, 其中OSEK是德文的缩写，译为**汽车电子开放系统及接口**。

OSEK OS是一个为满足汽车电子可靠性、实时性、成本敏感性等需求而打造的**实时单核操作系统(RTAOS)**。

该实时单核操作系统具备以下基本特性以及基本系统服务,如下图2所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAIhoJwIvTS80jsP69pCsnNd2QlAvicaOsVYZkAuoFIUAkmxfP4p2AM8Z0CZSEw2Cia0sA2iavzCTbwg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图2 OSEK OS基本特点



#### AUTOSAR OS与OSEK OS联系

首先，**AUTOSAR OS是基于OSEK OS继承发展而来**，所以上述的OSEK OS的基本特点在AUTOSAR OS都能够得到满足，所以AUTOSAR OS是向后兼容的，也就意味着在OSEK OS上能够运行的应用程序同样也可以在AUTOSAR OS上运行。

除此之外，AUTOSAR OS也存在自身的一些独特的基本特点，下面将从该OS的基本属性与系统基本服务两个方面展开：

**基本属性**

AUTOSAR OS在OSEK OS的基础上，除了上述的基本特点之外，仍需要确保具备以下几点十分重要的属性：

- 是静态配置以及伸缩扩展；
- 支持实时性能推理；
- 提供基于优先级的调度策略；
- 在运行时提供保护功能(内存、定时等)；
- 在低端控制器上是可托管的；

**系统服务**

AUTOSAR OS继承OSEK OS，在OSEK OS的基础上又特别明确了AUTOSAR OS至少需要提供的系统服务如下：

- 基于优先级的调度；
- 及时的中断处理的能力；
- 中断优先级必定高于task；
- 通过StartOS()与StartOSHook()来创建启动接口；
- 通过ShutdownOS()与ShutdownOSHook()来创建关机接口；
- 能够在OSEK OS中跑的APP自然也能够在AUTOSAR OS运行，但同时Autosar os也同时限制了OSEK OS的一些基本使用；

#### AUTOSAR OS基本对象

AUTOSAR OS总共包含以下**5****大基本对象：Counter，Alarm，Schedule Table，Task，ISRs**。

这5个基本对象必须归属于一个OS Application，可以简单理解为**OS Application是上述5大基本对象的容器。**

而归属于同一OS Application的基本对象则可以互相访问，来自其他OS Application的基本对象则需要通过配置来限制性访问。

如下图3所示，列举了5大基本对象的基本含义：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAIhoJwIvTS80jsP69pCsnNSv3CCIRE5ITH2ge9cfP9UudStsJpOlqMY2u4BE108QsG8k35RdmAmg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图3 AUTOSAR OS 5大基本对象

这5大基本对象，OS Application，Core这三者存在着一定的关系，为了更好的理解它们之间彼此的关系，如下图4就较为清晰地描述了者三者之间的关系。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAIhoJwIvTS80jsP69pCsnNjxch4wibsfiayGUyhicAvwKADWzR4Qhnc49U7HZkQQr63mFCMgWyB1QDw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图4 Core中基本对象关系

如上图所示，以单核为例，**每一个Core可包含1~N个OS Application，而每一个OS Application可包含0~N个基本对象**。

每个基本对象必须从属于某个OS Application，否则会出现错误。同时OS Application可分为Trusted与Not Trusted这两种类型。

Trusted 与Not Trusted 的OS Application在运行过程中可以配置相应的监控与保护机制。

从属于Not Trusted OS Application的OS基本对象对存储器和API的访问将受到限制。

通常会将一些基础软件的模式管理主函数映射到Not Trusted OS Application中的任务，如EcuM_Mainfunction，BswM_Mainfunction, Can_Mainfunction_Mode()等周期性状态查询函数，当然前提这些软件模块的安全级别为QM。

接下来，将针对这5大基本对象分别展开讲述各个对象的基本特点与用途，以便大家在对OS配置的过程中有一个更为清晰的认识。



**Task**

**基本任务与扩展任务**

AUTOSAR OS中存在两种任务：基本任务(Basic Task)和扩展任务（Extended Task）。基本任务则存在以下三种状态：

- **运行状态(Running)**：处于运行状态的任务可能被高优先级任务或者中断抢占从而进入就绪状态，且同一Core中任何时刻只会存在一个任务处于运行状态，任务运行结束后则将自己挂起进入阻塞状态；
- **就绪状态(Ready):** 处于就绪状态的任务由调度器决定是否启动进入运行状态，且该状态时任务切换至运行状态的前提；
- **阻塞状态(Suspend):** 处于阻塞状态的任务是被动的，可以由API函数或Alarm激活进入就绪状态；

扩展任务与之相比，则多了一个等待状态(Waiting),解释如下：

- **等待状态(Waiting)：**当任务的运行需要等待某一或某些事件被置位时，任务进入就绪状态。

基本任务的代码示例如下：

```
#include “OS.h”
TASK(BasicTask)
{
  ...
  /*User Code*/
  ...
  TerminateTask();
}
```

扩展任务的代码示例如下

```
#include “OS.h”
TASK(ExtendedTask)
{
 for(; ;)
 {
     WaitEvent(Event1);
     /*Perform some actions in specific condition*/
     ClearEvent(Event1);
 }
}
```

基本任务与扩展任务的状态机切换如下图5所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAIhoJwIvTS80jsP69pCsnNF2gW6QibVtUHPuvTy728vLPJlYJ7xegzjZQYV2kZIFnmiaPEf6hWbNIw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图5 基本任务与扩展任务状态切换图

如上图所示，基本任务没有等待状态，所以只能在任务启动与终结时进行同步，基本任务的优点就是占用较小的任务与执行时间。

扩展任务则包含多个同步点，没有同步请求的麻烦，当进一步的条件无法满足时，任务则会切换至等待状态，其缺点也很明显，会占用较多的内存和执行时间。

**Task的符合类(Conformance Class,简称CC)**

AUTOSAR OS根据不同的软硬件需求，根据每个优先级可能具备的任务个数以及需要的是基本任务还是扩展任务等来定义了四种符合类分别为**BCC1，BCC2，ECC1以及ECC2**, 各种符合类及属性其如下图6所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAIhoJwIvTS80jsP69pCsnN9H1fLgqY5Q6W3mwFxesQQM4UiaFib8uorkoQ7zz55cBMArF0vlxiaLwjQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图6 Task符合类

由上表可知，基本符合类BCC1与BCC2仅支持基本任务，扩展符合类ECC1与ECC2基本任务与扩展任务均支持；

BCC1与ECC1不支持多次任务激活请求且每个优先级只能有一个任务；BCC2与ECC2既支持多次任务激活请求，同时也支持每个优先级可以有多个任务。各种符合类之间的兼容关系如下图7所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAIhoJwIvTS80jsP69pCsnNWh3CHj7ffB8C1bgniaAplPn44ia7NoxhVx3jt1n0BG18TBqsFxkr284g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图7 各符合类的兼容关系

**Task调度策略**

AUTOSAR OS是基于优先级进行任务调度，所以每个任务必定有一个优先级，而每个任务都是根据其自身特点来定义一个优先级且需要配置其可抢占属性。

可抢占属性可分为不可抢占与全抢占，这里所说的抢占指的是内核抢占。AUTOSAR OS可根据各个任务的可抢占属性配置，来提供不同的调度策略，调度策略可分为以下三种：

- **完全抢占式**：OS所有任务均是可抢占类型；
- **非抢占式**：OS中所有任务均是不可抢占的；
- **混合抢占式**：OS部分任务是可抢占类型，部分任务是不可抢占类型；

1.对于完全抢占式任务调度策略而言，当前运行的任务可在任何时刻被高优先级任务打断而被迫释放处理器控制权，具备最高优先级的任务从就绪状态转入运行状态，而当前任务被抢占从而进入就绪状态，同时保留现场环境，待下次运行时恢复。

如下图8所示为完全抢占式任务调度策略，TaskA为扩展任务，TaskB与TaskC为基本任务，优先级TaskA > TaskB > TaskC。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAIhoJwIvTS80jsP69pCsnNaW5A5RkRumEvYPN2oJ5h0aaN4JcZu1R6MzniawicXQtMfCZnGNKDLesA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图8 完全抢占式调度策略

**Case1：**

当前TaskC处于运行状态，当激活TaskB进入到就绪状态时，由于TaskB优先级高于TaskC，所以TaskC被迫释放处理器控制权，调度器开始调度TaskB从就绪状态变为运行状态，直到TaskB运行完成之后，在调度TaskC继续运行。

**Case2：**

当前TaskC处于运行状态，激活TaskA与TaskB分别进入就绪状态，由于TaskA优先级高于TaskB，所以TaskA抢占内核运行, 但是由于Resource1仍被TaskC占用，而TaskA无法访问到共享资源Resource1，则被迫进入到等待状态，TaskB开始运行。

TaskB运行结束后挂起之后则重新运行TaskC，TaskC运行结束后释放Resource1，进入TaskA得以由等待状态转入运行状态。

此时你会发现高优先级的任务TaskA由于共享资源被占用的原因导致不能先于TaskB运行的现象，该现象也被称为**优先级反转现象**。

为了解决该问题，在此需要提到AUTOSAR OS的**优先级天花板模式**：**即将访问共享资源的任务优先级在占用资源的过程中提升至共享资源任务的最高优先级之上，从而避免优先级反转现象的发生。**

即若TaskC运行过程中占用共享资源Resource1，此时即使存在需占用共享资源的高优先级任务TaskA被激活，也必须保证TaskC运行结束之后才能执行TaskA，也就意味着在重要代码执行之前，应采用资源保护机制，以免被高优先级的任务打断。

\2. 若采用非抢占式调度策略，那么当前运行状态的任务在任何时刻都不会其他高优先级任务所抢占，任务的切换只会发生在任务完成时。 

非抢占式调度策略的问题在于任务执行时间不确定，系统调度实时性较差。如下图9所示为非抢占式调度策略，可见即使高优先级任务 TaskB被激活切换至就绪状态，也必须等到TaskC执行结束之后才能够被调度。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAIhoJwIvTS80jsP69pCsnNcLlBTic9iawibOA3YsojX3ibplYzNL5tVbAXs4bHvj0oBzRduOf5mVicxiaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)图9 非抢占式调度策略

3.若采用混合抢占式，则OS的调度策略就取决于当前任务的可抢占属性，如果为非抢占，则执行非抢占式调度策略，如果为抢占式则执行完全抢占式调度策略。



**Counter**

Counter概念的引入是为了实现对硬件计数器以及软件计数器的管理，为Alarm与Schedule table提供支持。

**即多个Alarm可以共用一个Counter，一个Schedule Table只能由一个Counter来驱动。** Counter按照AUTOSAR定义可分为以下两种：

- **Hardware Counter:** 该Counter的增加由硬件外设驱动，如Gpt或者timer等；
- **Software Counter:**Counter的增加通过调用API函数IncrementCounter来实现，且每次只能增加1；

**基本原则：**优先使用Hardware Counter，因为可以根据Task的激活状况来减少无意义的时钟中断；

如下图10所示，则较为清晰的表现了Counter，Schedule Table以及Alarm三者之间的关系。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAIhoJwIvTS80jsP69pCsnNwsElzflgjic5XX1LDMurk8Y1evWob1SpzHduWAHlooFeFu2dyiavJw5g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图10 OS Counter，Schedule Table，Alarm三者之间的关系

**Alarm**

在计数器的基础上，AUTOSAR OS为应用软件提供了闹钟机制，多个闹钟可以连接一个Counter。

当到达Alarm所对应的计数器设定值时，则可以激活一个任务，设定一个event，调用callback或者增加计数器等功能，但只能是一对一。

不能像Schedule Table那样，能够在Expiry point同时设定多个Task或者多个Event，这也是为什么引入Schedule Table的原因。

**一个软件Counter +多个Alarm队列就可以实现静态定义的任务激活机制。****但随着Schedule Table的引入，因此一般建议能用Schedule Table就不要用Alarm。**

**Schedule Table**

如上Alarm所述，当计数器的计数值依次达到各个Alarm设定的计数值时，各个Alarm被触发，但很难保证各个Alarm有特定的时间间隔；

且每个Alarm只能激活一个Task或者Event，所以需要多个Alarm来协作实现在同一时刻触发多个Task或者Event，因此Schedule Table应运而生！

Schedule Table会定义一系列终结点(Expiry Point)，且每个调度表都有一个以Tick为单位的持续时间(Duration)。

每个终结点则是以Tick为单位的距离起始点的偏移量(Offset)，在每个终结点可以实现多个Task或者Event的设置。

与报警器类似，一个调度表只能由一个Counter驱动，同时调度表存在以下两种调度方式：

- **单次执行(Single-Shot):** 调度表启动之后 只运行一次，到达调度表终点则终止，即每个终结点只运行一次；
- **循环执行(Repeating):** 调度表启动后可反复执行，到达调度表终点后重头开始执行，则每个终结点会被周期性的执行，一般情况下激活任务采用此模式。

如下图11所示，较为清晰了描述了调度表中Expiry Point与Task，Event激活之间的时序关系。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAIhoJwIvTS80jsP69pCsnNz255IZzxBXoCsyIx37fgHmZM2Um8CjM8LB5v2ZdGmGoOP13WlmwiaqQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图11 Schedule Table时序图

**注意点：**

- 每一个终结点必须配置至少一个Task或者Event；
- 每一个调度表至少存在一个终结点(Expiry Point)；
- 在每一个Expiry Point优先激活Task，随后设置Event；

**ISRs**

在AUTOSAR中定义了两类中断服务程序(Interrupt Service Routine)。分别为一类中断(Category I)与二类中断(Category),两者之间的区别定义如下：

- **Category I**：此类中断服务程序不能够使用OS提供的系统服务，当中断执行完成之后则会重新跳转至产生中断的地方继续执行，不会影响到任务的执行，因此占用系统资源较少。
- **Category II**：该类中断则可以调用OS系统服务，如激活任务或者设置事件等。

在AUTOSAR OS中，**中断的优先级始终高于任务的优先级，即最低优先级的中断都可以打断最高优先级的任务，即使该任务不可抢占也不例外。**

因此，中断服务子程序的执行时间不宜过长，否则会影响到整个系统的实时性。

**Resource Management**

Resouce作为OS调度过程中一个十分重要的对象，Resource管理就显得尤为重要。

资源管理就是为了协调具有不同优先级的多个任务或者中断对共享内存（如内存或者硬件等）的并发访问。

不过幸运的是AUTOSAR OS采用上述的优先级天花板模式来避免任务优先级反转以及死锁问题的发生，即资源的上限优先级必须高于所有该资源的任务以及中断的优先级，但是应低于不访问该资源的任务的最低优先级。

其中为了保护共享资源而提出的锁机制-自旋锁(Spin Lock)。该自旋锁一般用于多核操作系统解决资源互斥的问题。

当内核控制必须访问共享数据结构或进入临界区时，如果自旋锁已经被别的执行单元保持，调用者就一直循环在那里看是否该自旋锁的保持者已经释放了该锁，从而达到某共享资源的互斥作用。

#### 常用函数接口

为了便于大家日常软件调试，我将常见的OS相关API函数接口及其相应功能表述如下图12所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icAIhoJwIvTS80jsP69pCsnNvia1kL1Chme03E7Byd5NtQNeXiaNPwOtgicQR4dnourY0gDWPfCJEWP2w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图12 常用函数接口列表



# AUTOSAR基础篇之OS(下)

原创 汽车小T8 [ADAS与ECU之吾见](javascript:void(0);) *2021-11-08 08:19*

![图片](https://mmbiz.qpic.cn/mmbiz_png/jvibaTuwDCBHkp4Ocd4ibfkBJqRgkiaxZ8bWqDzrhReRrRcczPEsAV9qwuhESMWxNrGaxvGiax4Jky0thTibbQNOF2A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

小T觅食记

![图片](https://mmbiz.qpic.cn/mmbiz_png/kjOBCbXenyDJl0zqzjZSohdOpooZGtE9t5kznNnwDxY8GftM54027Qm9wgg7V1xCAhlon8RSlxUpDZb4OTVTSQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

PICK OF THE WEEK





![图片](https://mmbiz.qpic.cn/mmbiz_png/TN05MmJLxMoECPkMMOX5e1ziaTV5tsEzibFykKI0oqyEFX6CCOicW8ibA2kl5CVd90iaxdBs49ibLic1Y8IbGdQeib2J6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





![图片](https://mmbiz.qpic.cn/mmbiz_png/c6gqmhWiafyogBy7xAQibchicFl7vsK9o0NrsaRzibujyVFLwCa3Bsw0SjMo04TdTLibtGUwGauYj0e0ZnvB5sn9bpQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

小T我也是吃货一枚，平常有时间就会外出探店寻觅美食，也希望每次给大家分享知识的同时也带来美食分享！(PS:纯属个人分享，不是美食推广哈)。

这两天降温了，小T今天就给小伙伴们推荐一家滋补鸡汤海鲜火锅小店，既能喝鸡汤又能涮海鲜火锅！

**美食店名**：威皇广福和小海鲜(乌鲁木齐南路店)

**特点评价**：香气逼人的招牌滋补醉鸡煲，甜的爆的基围虾海鲜，难得吃到的鲜嫩西洋菜！

**免责声明**：小T吃完瞬间感觉头发也变多了，味道回味无穷，内心实在难以掩盖分享的激动！在上海的小伙伴可以试试，尽量早点去不然要排队等很久，如果有小伙伴觉得不好吃，口味差异，可不能揍我，好心推荐哈![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKubuD8aQzAjm9qEvicPjTyNk6BmowcW1cfatcIEUpVRz8LmzqziaCN9qA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

小伙伴们要是知道上海其他好吃的店，欢迎后台加我微信留言推荐。

![图片](https://mmbiz.qpic.cn/mmbiz_png/6aVaON9Kibf56gaTAFbn1qM9hPmo70lDI4auaHlE5hcMAcCgJ5gWibWfAtJQtNRmb9iaRccKYjvibfjFficRNUzI64A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKtfL1IYzoKxick9bKlb80dXtprdDFySpADJCrzv2ZtkCVvBkqtsHNChg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrK1w56ytCawMLvmJS9eDGOsYmmwZ98Cgda4hdTHMiaS39icbCpEUvjibfIw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



------

### 言归正传 

首先，请问大家几个小小的问题，你清楚：

- 你知道多核OS在什么场景下使用吗？
- 多核系统OS又是如何协同启动或者关闭的呢？
- AUTOSAR OS存在哪些功能安全等方面的要求呢？
- 多核OS之间的启动关闭与单核相比又存在哪些异同呢？
- 。。。。。。

今天，我们来一起探索并回答这些问题。为了便于大家理解，以下是本文的**主题大纲**：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKsiauttBGeppMS91d90rMJErAcGMBe0yJRHW1e6zicrR1UDJsYWqFuDwA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

------

### 正文

#### AUTOSAR OS 保护

在上篇文章[AUTOSAR基础篇之OS(上)](http://mp.weixin.qq.com/s?__biz=MzU3OTI4NzY0OQ==&mid=2247488043&idx=1&sn=baf40b482d75a8e9a72bf0b1deca505f&chksm=fd693d25ca1eb433883335816e22aec70a8cd8e2a88d66924764336ac8e7bca95d16b9522487&scene=21#wechat_redirect)中我们可以了解到AUTOSAR OS的基本特点，基本对象以及各个对象之间的彼此关联，本篇文章将承前启后，在此基础上来简单谈谈我对AUTOSAR 多核OS的理解与认识。鉴于本人水平能力有限，如存在错误之处，还请大家多多批评指正。

我们已知道AUTOSAR OS来源于OSEK OS，随着汽车电子信息安全，功能安全等需求的不断提出，传统的OSEK OS已无法满足当前的需求，**因此AUTOSAR组织在OSEK OS的基础上为不同的用户提供四类不同功能安全的OS可裁剪类型，分别为SC1-SC4**。

**AUTOSAR OS 可裁剪类型**

AUTOSAR OS的四种可裁剪类型分别为SC1-SC4，具体含义如下：

- **SC1:** OSEK OS + Schedule Table；
- **SC2:** OSEK OS + Schedule Table + Timing Protection;
- **SC3:** OSEK OS + Schedule Table + Memory Protection;
- **SC4:** OSEK OS + Schedule Table + Timing Protection + Memory Protection；

如下图1所示，较为清晰了描述了这四种不同可裁剪类型的区别与联系。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKflyFAaM9zaBiaF0Uo48icTYgtOVktT7Pic7AjvbtQKW1ly1CGS4XhuicBQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 AUTOSAR OS可裁剪类型

**AUTOSAR OS 时间保护**

从AUTOSAR OS四种可裁剪类型可以看出，时间保护(Timing Protection)是一项非常重要的功能保护机制。如之前文章所示，AUTOSAR OS作为一实时操作系统，那么就需要在预定的时间内完成特定的任务，但有时由于某些原因导致超时错误，OS必须采用有效的方式来预防超时任务的发生，而这类措施则可以称为时间保护。

一种较为常见的时间保护就是Deadline Monitoring。即当OS检测到某一任务的运行时间超过其截止时间时，则会调用相应的Hook函数向系统报错，但是AUTOSAR OS并不是通过监控截止时间方式来实现时间保护的，因为针对截止时间的保护并不能准确识别出当前错误的原因。具体解释如下：

- **现象：**任务1的运行时间超过其截止时间时，任务A本身可能并没有出错；
- **过程：**在执行任务1之前的任务2频繁的抢占或者过长的阻塞资源的访问；
- **原因：**正由于任务2的上述行为，进而导致任务1执行超时，从而直观的认为任务1发生错误便停止任务1，反而让真正的罪魁祸首任务2继续逍遥法外，这就不合情理，也起不到对OS中各任务的时间保护。

下面为了进一步加强大家对上述简单的监控截止时间造成运行错误的理解，假设存在一个操作系统中存在下列任务A，B，C，并明确各自任务的优先级，执行时间，Deadline 如下表1中所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKDICwUt2DBq0Q1aC3tRibyymicwEnNRNYmcqKx7mavwSy0skX3oPbuAmg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

表1 操作系统任务的基本属性假设

假设所有任务在0时刻均处于就绪状态，期望运行的任务执行时序如下图2所示。具体的任务执行时序如下所述：

- 由于任务A优先级最高，因此任务A先执行；
- 1个Tick之后任务B开始执行，3个Tick之后任务C执行；
- 当任务C执行1个Tick后被任务A打断，任务A执行完毕后，任务C继续运行；
- 到第10个Tick任务C执行完毕，周而复始，整个过程并未出现超时现象，并且仍有一个Tick的空闲状态；

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKAHCUNsPEaWX2ziaupWhbtGIupuiaNpLicYLXxLkyF8WibRyfMUHuSpX0zA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图2 任务期望时序图

接下来，如果发生如下图3所示的异常状况，那么我们看会发生什么呢？

- 任务A的第二个周期与任务B的第一个周期都出现运行时间过长的现象，但并没有超过其截止时间。
- 任务B在第二个周期提前进入运行状态，但也未超过其截止时间；
- 任务C按照正确的方式运行，但由于任务A与任务B的出错导致任务C运行超时，则发生超时错误；

若此时采用简单的超时监控机制只能监控到任务C超时，这时操作系统调用钩子函数，由于出错原因并未被检测到，从而操作系统采取的措施将无法有效解决错误。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKFL1HTUEWLiby4bicvI7ibh3EP9oLCIzP9NhTUqEIMicac0THZMVibibmvkCg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图3 任务超时现象

因此，从上述例子分析可以得出任务或者中断能否满足其截止时间需取决于以下三大基本要素：

- **1.静态任务或者中断的执行时间上限；**
- **2.被低优先级任务锁住共享资源或屏蔽中断所引起的阻塞时间；**
- **3.任务或者中断之间的间隔执行时间；**

针对上述三大基本要素，AUTOSAR OS采取了下列三种时间保护机制：

对于1：AUTOSAR OS为任务或者二类中断服务设定了运行时间上限；

对于2：AUTOSAR OS设定了共享资源被任务或者二类中断锁定的时间上限，设定了OS中断被任务或者二类中断中断挂起的时间上限，设定了所有中断被任务或者二类中断挂起或者屏蔽的时间上限；

对于3：AUTOSAR OS设定了任务或者二类中断执行间隔的时间下限；

特别的，需要注意的是AUTOSAR OS时间保护存在一些基本特性：

- **时间保护仅仅用于任务或者二类中断，对于一类中断不起作用；**
- **在OS未开启之前，时间保护将不起作用；**
- **对于Trusted OS Application, OS应当有能力提供一种基于任务或者二类中断的时间保护，而对于Non-Trusted OS Application，OS必须提供为这个非信任的OS Application中的每一个任务或者二类中断提供时间保护；**

**AUTOSAR OS内存保护**

AUTOSAR OS的内存保护需要特定的硬件支持，即处理器应该存在MPU单元(Memory Protection Unit), 例如英飞凌AURIX单片机则具备这个特性。鉴于内存保护可以有效防止出错的应用模块影响到其他模块，降低系统完全瘫痪的风险，因此，AUTOSAR OS提供以下三种形式的保护：

**栈保护**

- 即每一个OS Application和其中的OS Object都有各自的私有栈，不同的OS Object无需存在共享栈。栈保护能够更为快速的检测出栈溢出，同时栈保护也是划分OS Application的一种方式和依据。

**数据保护**

- 每一个OS Application和其中的Objects都具备各自私有数据，同时OS Application的私有数据区就是从属于该OS Application的Objects的共享数据区；

**代码保护**

- 代码区既可以被私有，也可以被共享，在没有代码保护的前提下，错误代码的执行会导致内存，时间和服务上的出错。

OS通过MPU监控内存的访问权限，其中AUTOSAR的访问权限可分为受信任与非受信任的Object(Trusted and Non Trusted)两级，受信任的Object有读写大部分内存的权限，但没有读取其他非激活栈的权限。

而非受信任的Object仅有读写少数内存的权限，包括当前活跃的栈，当前OS Application的数据以及LMU的共享数据。



#### AUTOSAR多核OS启动与关闭

在AUTOSAR软件基本架构下，无论是单核操作系统还是多核操作系统，都与EcuM模块和BswM模块息息相关。因为这两类模块决定了OS启动，初始化，运行，关闭等状态及其过程。

从之前文章[AUTOSAR基础篇之EcuM](http://mp.weixin.qq.com/s?__biz=MzU3OTI4NzY0OQ==&mid=2247487807&idx=1&sn=8f6ade36c059be313841bb38d7b41782&chksm=fd693e31ca1eb727beb4151889a4e2b932c5285c514890f7897137d63107d628459d29641ddd&scene=21#wechat_redirect)我们可以知道ECU工作过程可分为启动(STARTUP), 运行(UP), 睡眠(SLEEP)以及关闭(SHUTDOWN)四种状态。

ECU在上电前处于SHUTDOWN阶段，上电后便进入STARTUP阶段，主要包括StartPreOS与StartPostOS两个子阶段。

在StartPreOS子阶段主要完成一些OS启动之前的一些准备工作，如初始化MCU，IO，WatchDog等模块；

在StartPostOS子阶段则是启动OS之后的阶段，该阶段主要执行初始化BSW的调度器以及初始化BswM模块两个动作，如下图4所示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKmkFj8I2P5nn0wagZvTX9XkodKXucbiaic6U4SeickcUNUJ57sicVVvTFxA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图4 ECU启动流程

由于ECU详细的启动过程在[AUTOSAR基础篇之EcuM](http://mp.weixin.qq.com/s?__biz=MzU3OTI4NzY0OQ==&mid=2247487807&idx=1&sn=8f6ade36c059be313841bb38d7b41782&chksm=fd693e31ca1eb727beb4151889a4e2b932c5285c514890f7897137d63107d628459d29641ddd&scene=21#wechat_redirect)文章中已有大量篇幅介绍，本文就不再赘述，只是作为一个引子，大家如果有兴趣，可以阅读了解一下。

**多核OS启动过程**

当然无论是否存在OS，多核的启动过程相比单核与硬件关系则更为密切。

**通常情况下，硬件会首先启动一个核作为主核(Master Core)，而从核(Slave Core)则由软件启动，这种启动方式被称为主从模式。AUTOSAR规范定义了多核启动方式应该为主从模式。**

值得注意的是：即使硬件支持多核同时启动，AUTOSAR规定也需要通过软件模拟的方式来实现主从模式来启动多核系统。

如下图5所示，非常生动形象的描述了主从模式的各个环节的启动时序及相互之间的交互关系。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKcx5YDMG5S5MoMOb0ro6VUAHDvagz2B62dRqFHXs5DiaIk1TVPWVAMiaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图5 多核OS的启动过程

在上图中，Core 0作为主核，其他核则作为从核。

- 主核Core 0完成前期的硬件初始化之后启动从核Core 1 并随后调用Start OS函数来启动OS，OS完成初始化之后在第一个同步点等待所有从核完成OS的启动。 
- 从核Core 1被主核启动之后，首先完成硬件相关的初始化，然后激活从核Core2，Core3，并在第一个同步点等待其余从完成OS的启动；
- 从核Core2，Core3被Core 1激活之后，首先完成各自的硬件相关初始化，然后调用StartOS完成OS的初始化并在第一个同步点进行同步；
- 在完成第一个同步点之后，主从核便分别执行Startup Hook函数之后在第二个同步点进行同步，然后所有核的Kernel将一起运行，只有这样才能够更好的保证整个系统的稳定性与鲁棒性。

**值得注意的是如果某从核运行的OS不是AUTOSAR OS时，此时则不能使用AUTOSAR OS API StartCore来启动该从核，而应当使用StartNonAutosarCore来实现该从核的启动。**

**多核OS关闭过程**

与单核OS关闭过程类似，多核OS的关闭也是通过EcuM来完成，如果在关闭过程中出现唤醒事件，ECU则需要关闭之后立即重启。

在关闭过程中可以选择ShutDown Target分别为关闭(OFF), 睡眠(Sleep),和复位(Reset),此处仅简要描述其基本过程，如下图6所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKkic0EfDG3lAPkDxDE2bj20LjC1zDxjNQC7vsONYDdea42n46Lvb0H8w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图6 ECU关闭过程

如上图的ECU关闭过程主要分为以下几个阶段：

- 反初始化BswM以及BSW调度器；
- 检查是否存在唤醒事件发生；
- 选择ShutDown Target；
- 关闭OS；

如下图7所示，则较为生动形象的描述了多核OS的关闭过程。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKVJrp0vA8mmaluvYQfepoAVibplUMJShGEbqQomAZR1adaQiaoMPvumSA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图7 多核OS的关闭过程

在多核系统中，关闭过程存在以下几个特点：

- AUTOSAR4.x不支持仅仅关闭单个核，即若发送关闭指令或者致命错误时所有核必须全部关闭，具体的关闭过程如上图所示；
- 若某一任务拥有调用ShutDown All Cores的权限时，关闭信号将会同步发送至所有核；
- 当关闭过程启动后，所有的中断服务和任务都不能被激活，关闭前必须完成的程序由EcuM保证完成；
- 关闭完成前，各自的OS Application调用各自的Shutdown Hooks函数完成对应的回调程序，然后等待到同步点所有核执行关闭回调函数。





#### AUTOSAR多核OS调度

**基本特点**

多核OS调度相比单核OS调度而言并没有什么区别，都是**根据任务或者中断的优先级作为首要因素来决定调度顺序**。在同一处理器内核上，优先级越高的任务或者中断优先调度。如果优先级相同，那么就根据激活顺序进行调度。

需要明白一点的是对于单核系统，CPU每时每刻只能运行一个任务或者中断，采用时间片轮转法来实现看上去的**并发执行**。

对于多核系统而言则不同，由于存在多核所以可以同时运行多个任务或者中断，至于能够同时运行几个任务或者中断由CPU内核数量来决定，我们把这种执行方式称为**并行执行**。

同时对于AUTOSAR操作系统而言，任务及中断的优先级是提前静态分配的，在运行过程中不支持动态更改。

**调度方式**

如下图8所示，清晰的表现了 多核OS调度任务的过程。根据调度规则，若在同一核上多个任务被同时调度，即这些任务均处于就绪状态，那么高优先级任务会被率先执行，如图中的Core0上的任务T2，Core1上的任务T3以及Core2上的任务T5，三个任务同时进入运行状态，各个内核上的任务独立运行，互不干扰，其优先级并没有相互影响。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKPC3YAj8KsF2mLKLjXFm6KiadOoERjR13FCjRndhRZfkaQWgaAica43RA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图8 多核OS任务调度图

同时，对于AUTOSAR 多核OS支持任务选择调度模式，可分为全调度和拒绝调度模式。对于全调度模式，任务在运行过程中，可以被高优先级的任务或者中断抢占，而对于拒绝调度模式下，该任务不能够被任何其他任务或者中断抢占。

**因此在任务分配优先级的过程中，应当将起到关键作用的重要任务分配较高的优先级，重要程度类似的任务周期越短，分配的优先级越高。**

#### AUTOSAR多核OS通信(IOC)

AUTOSAR规范定义了内部通信与外部通信两种方式，其中内部通信包括核内通信，核间通信。

每当不同内核中的用用程序需要进行数据交互时，操作系统就需要开辟单独的内存区域，通信双方可以实现该内存区域的读写，从而完成数据的传输。

但需要注意的是共享内存区域有可能在读取数据的过程中发生数据更新，这样就会发生数据一致性问题。

为解决上述数据不一致问题，AUTOSAR多核OS提供了应用于核间通信的**IOC(Inter OS Application Communication)**通信机制。

IOC不同于核内通信，核内通信则是通过RTE来实现。

**在使用IOC的过程中如果应用程序对这一共享内存区域进行读写时，会申请占用一个自旋锁(SpinLock),以防止被其他内核的应用程序同时访问，这样便可以保证核间数据读写的一致性。**

IOC仅提供Send-Recerver的通信方式，但是RTE可以将Client-Server的请求与应答可间接转换为Send-Recerver模型，因此IOC支持1：1，1：N，N：M的通信。

在每次传输过程中可以传输一个数据项，该数据项可以是基本类型的值或者复杂数据结构的参考，而如果是复杂数据结构，那么就必须被实现为单个内存块，这样IOC便无需得知内部的数据结构 ，仅需知道内存地址和长度便可以完成该结构体数据的传输。

**Send-Receiver模型**

如下图9所示为也不带通知的1：1的Sender-Recerver的通信模型。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrK3lFPgVyK4x2DewGLWfMNlLwRMOlibHfE3tiaQor8FCtibNI1FDpia8uIhQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图9 不带通知的RTE通信模式

如上图，Core0中SW-C将数据发送至Core1中的SW-C模块，以下是核间通信的具体步骤：

- 接收端实体被周期性调用通过Rte_Receive从RTE接收来自Core0的数据；
- 发送端调用函数Rte_Send函数发送数据，进而调用Ioc_Send函数写入数据到Buffer中；
- 接收端便会通过Ioc_Read函数读取共享内存中的Buffer数据，并传递给到Rte_Read函数中供Core1中的SW-C使用。

IOC生成器会生成所有的发送及接受函数，为了优化目的，这些函数被定义为宏指令，这种不带通知的通信方式适用于以下场景：

- **Send/Receiver通信；**
- **队列或非队列通信；**
- **1：1通信；**

**Client-Server模型**

如下图10所示，为带通知的N：1的Client-Server模型。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKeVXu5Eic0kT5YIeFZSFdP523NF9VY1OKSMT1BT8gG9N9OibEPDHbniaHQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图10 带通知的IOC模型

如上图为Core 0 中的SW-C请求Core1中的服务操作。RTE通过将client-Server调用映射为send-Receiver来实现客户端的服务，因为需要跨核通信，所以RTE调用IOC将数据从Core0传输至Core1,具体传输过程如下所示：

- 发送端调用函数Rte_Call函数进而调用函数IocSend函数，将数据写入IOC内部队列缓存中；
- Rte_Call函数使用OS调用激活接受从核的服务任务来通知接收端；
- 接收端被激活的该任务将负责调用IocReceive函数从IOC共享内存Buffer中读取数据并将数据传输至服务端的运行实体中；
- Core1中服务函数的结果会被反向传输至Core0的客户端中。

这种带通知的通信方式适用于以下场景：

- **带通知的Send-Receiver通信；**
- **Client-Server通信，在此类情况下，RTE需要将服务转换为1：1的Send-Receiver模型通信，同时将服务结果反向传输到客户端的过程转换为另外一组Send-Receiver通信；**
- **队列或者非队列通信；**
- **1：1通信，如果接受端没有被周期性的调用；**
- **N：1通信；**

**核间元素交互与任务同步**

我们知道多核OS由任务，中断，报警器和事件等元素组成，它们之间的相互作用使得操作系统能够有条不紊的运作。

从一般意义上来讲，AUTOSAR OS本质上就是基于事件驱动的操作系统。如下可简要描述核间各元素的交互关系：

- 报警器可以激活基本任务A或者设置某事件1;
- 扩展任务B等待事件1那边可以激活基本任务C；
- 基本任务C可以设置某事件2，中断可以设置某事件3；
- 扩展任务D等待事件2和事件3之后便可以开启执行；
- 基本任务E可以通过IOC机制实现与基本任务A之间的数据交互；

如上可知，在多核OS中事件是可以跨核传输的，这也就意味着核与核之间的同步可以通过事件触发的方式来实现。

如下图11所示，当Core0中的任务T1执行时，可以通过设置Event来激活位于Core1中的T2，这样便完成了T1与T2之间的任务同步，该方法适用于事件触发的任务或者定期执行的任务之间的同步，当然如果是定期执行的任务只需使用Alarm或者调度表定期触发Event即可。

![图片](https://mmbiz.qpic.cn/mmbiz_png/cr5sh0Hw1icBMBwDlvibWia6AmgL3yLxkrKS51yRKdNTSOltBicLUE9sV8DtiaPeDhjMJab2aGxcOiaTU31pXQQcopGw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图11 多核OS任务基于事件同步示意图

除此以外，我们还可以通过**调度表来实现任务同步**，由于报警器只能一次触发一个任务，因此不能实现任务同步，而调度表则可以实现同时触发不同核的多个任务，而该方法仅用于定期执行的任务的同步。

