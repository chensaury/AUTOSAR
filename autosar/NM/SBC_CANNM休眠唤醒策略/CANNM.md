Autosar CAN开发05（Autosar的CanNM----网管报文在汽车上的作用、“同起同睡”）


​        网管报文的作用：

        要理解网管报文的作用，我们需要先明白车上的节点------“Node”。
    
        所谓节点，对一台新能源汽车来说，车上有着几十个节点，如OBC节点、VCU节点、BMS节点。如果这些不知道的话，再简单点来说，一个节点可以理解为一台车上的一个MCU控制器（大家都把一个节点的控制器称为ECU），这个控制器负责车上的一部分功能，如BMS节点就是负责车上的电池管理这部分功能。因此车上几十个节点，每个节点都负责一部分功能，这就是节点。
    
        比如下面这张图

![img](https://img-blog.csdnimg.cn/1c14c871761540d8a2a31d8cdfe8d1d1.png)

        好了，明白了什么是节点，接下来就是要知道这样一件事：一台车上那么多的节点，当车要正常工作，不可能每个节点都自己玩单机游戏吧：电池模块不管车上其它用电设备，想放电就放电，不想放电就死也不输出、汽车尾门不管汽车是否处于在路上跑着的状态，想开尾门就开尾门。这样想想都滑稽对吧。所以，车上的每个节点之间都是要相互通讯的，以使得各个节点获取自己需要的讯息然后才考虑要不要进行动作（通讯方式如CAN通讯、LIN通讯、以太网等。本篇是讲CAN通讯，文章接下来所有的节点都认为是CAN通讯）。
    
        明白了汽车节点间要进行通讯，另外还需要知道汽车各个节点都有休眠机制，简而言之，就是没有汽车没有工作需求的时候节点就要休眠下去，以减少功耗。
    
        好了，接下来你想一下，当汽车上的所有节点都处于休眠状态，这时候节点A被触发了唤醒机制（比如车子被车主踩下踏板------这个栗子也不知道合不合适，反正就是假设节点A是主动起来的）。现在的情况是：节点A主动唤醒了，但车子正常工作不能就一个节点A醒来干活啊，其它几十个节点还全部处于休眠状态，这可咋整？
    
        其实很简单，节点A向外喊一声不就行了。就像大学宿舍那样，大家要么在睡觉，要么出去玩，而且格外团结，每次有人出去玩都会叫人其他人一起。好了，现在你想出去玩了，你要怎么让在睡觉的他们跟你一起出去？很简单，你只需要大声吼一句：“兄弟们，走了，出去耍”，然后宿舍所有人听到了就起床一起出去玩了。
    
        汽车也是这样，节点A要叫醒其它节点，只需要节点A向整车网络广播网管报文就可以了。这个作用就类似于那句“兄弟们，走了，出去耍！”。

![img](https://img-blog.csdnimg.cn/a19eafaa9c4b4840903a0e37f1acff0f.png)

        但是注意，节点A要是向外广播的不是网管报文而是别的报文呢？用上面那个栗子，就会是这样：你叫大家去玩，但是，你不是吼要出去玩，而是吼一句：“兄弟们！看我看我！我是个帅比！”。那么效果就是舍友已迅雷不及掩耳之势把你丢出去，然后马上回去继续睡大觉。所以，要是节点A广播应用报文出去，那么其它节点就会上一下子电，然后跑到检测唤醒源代码，检测到不是有效唤醒，最后就会马上又休眠下去。
    
        好了，现在所有节点都起来了。但是注意，节点A是主动唤醒，因为节点A有主动工作需求，其它节点是被迫叫起来的，其它节点本身是没有自己主动工作需求的。他们起来只是因为节点A可能需要这些节点的某些数据。所以，其它节点是被动唤醒。 
    
        换句话说，当节点A主动工作完成后，其它节点的任务也就完成了。所以，在节点A有主动工作需求的整个过程中，节点A会一直向外发出NM报文，以使得其它节点一直处于唤醒状态，其它节点是不会发出网管报文的（被网管唤醒的节点在被唤醒的开始几秒会发出几帧NM报文，然后它就停发NM了）。当节点A主动工作需求结束后，它就停发网管了，此时若车上其它节点也没有主动工作需求，那么车上就没有节点向外发网管报文了，当车上的节点没有接收到网管报文一段时间后，就会进入休眠状态。

![img](https://img-blog.csdnimg.cn/77a00b268980429f955325494fa15727.png)

        上面说了那么多，各个节点从休眠状态到唤醒状态再回到休眠状态，人家专业人士概况起来其实就四个字：“同起同睡”。
    
        讲到这里，你应该明白了CANNM报文到底是干啥用的了。

提前贴一张官方文档下的CANNM报文状态机的截图：

![img](https://img-blog.csdnimg.cn/0c8ca6f4d385483f99c65c32069f72c5.png)

Autosar CAN开发06（Autosar的CanNM----CanNM状态机）

        在了解了CANNM在汽车上的作用之后，我们来看CANNM的状态机是如何实现的。

一、CANNM状态机：

        1、首先我们先看一下CanNM的状态机及各个状态下报文发送的情况（一个汽车的ECU在CANNM处于不同状态时，对于CAN应用报文和CANNM报文有着不同的发送要求。比如：在Bus-Sleep-Mode状态，应用报文和网管报文都不往can总线上发送。在Read Sleep State状态，应用报文要往can总线上发送，但CANNM报文不往can总线上发）

![img](https://img-blog.csdnimg.cn/fb3de4a3d6c5417ab2cc31000038afd8.png)

![img](https://img-blog.csdnimg.cn/7140df6399f843c9a64e917d9ae6e15f.png)

2、CANNM各个状态跳转条件

        理解CANNM状态机的跳转，需要再理解一下ECU的休眠唤醒机制。在一辆汽车整车上，某个ECU的休眠唤醒定义一般是这样的（比如ECU 1）：

休眠：ECU1不向CAN总线外发出报文

唤醒：ECU1向CAN总线外发出报文

        我在开发过程中，经常遇到有同事认为休眠是指ECU没电、唤醒是指ECU有电。虽然这样理解很多情况下也没有问题，但实际上对于整车来说，一个ECU醒没醒，是通过CAN报文来看的，你向外发出CAN报文，就认为你醒着，你停发CAN报文，就认为你休眠了。如：当CANNM状态处于Prepare Bus-Sleep Mode 的时候，是不发CANNM报文和应用报文的，有些企业的需求认为此时ECU已处于休眠状态，当处于此状态时有别的唤醒源唤醒ECU时，需要ECU认为此时更换了唤醒源，但ECU此时并未下电。
    
        只不过正常情况下，为了减小功耗，只要ECU没有外发CAN报文的需求就要立即下电，ECU有外发CAN报文的需求就要立即上电。）
    
        好了，那么什么时候ECU要醒来，什么时候ECU要休眠呢？其实情况不多，就两种：被动唤醒和、主动唤醒。而当被动唤醒和主动唤醒都释放的时候，就需要休眠。被动唤醒和主动唤醒解释如下：
    
        ①被动唤醒（自己没有主动工作需求，是由于别的节点有主动工作需求，自己才被迫唤醒）：ECU节点接收到其他节点的网管报文。
    
        ②主动唤醒（自己有主动工作需求，会通过网管报文唤醒别的节点）：ECU节点有主动工作需求。如OBC节点检测到充电的插枪动作。


​       

接下来理解CANNM的状态机就好理解了：

（当进入Repeat Message State、Normal Opearation State、Ready Sleep State时，都成CANNM处于Network Mode）

①Bus-Sleep Mode：
就是CANNM状态机处于睡眠状态。CANNM状态机处于这个状态一般有两个情况：

        一：ECU被唤醒刚上电初始化，程序还没跑到处理网管状态跳转的时候（无论是主动唤醒还是被动唤醒）
    
        二：是ECU准备进入休眠的时候，即程序跑到即将下电前。

![img](https://img-blog.csdnimg.cn/4d19a2a0f6fb40369ad4bfcccaaee82c.png)

②Repeat Message State：
从上面的状态机可以看出，ECU被唤醒后，必须先经过Repeat Message State。

![img](https://img-blog.csdnimg.cn/0490185cb30e47739ca266a6369f77dc.png)

        Repeat Message是指重复发送CANNM报文，那么为什么在这各状态下要重复发NM报文呢？正如前面所说，ECU有主动唤醒和被动唤醒。如ECU1：
    
        当ECU1有主动唤醒需求时，ECU1是第一个醒来的，它需要整车其他节点快速起来配合工作。因此，ECU1需要重复快发CANNM报文，使得其他ECU节点快速唤醒。
    
        当ECU1有被动唤醒需求时，ECU1被总线其他节点的NM报文唤醒后，ECU1在该状态下需要发送几帧CANNM报文，作用就类似于告诉别人：我起来啦！
    
        可以看出主动唤醒和被动唤醒时，Repeat Message State发出的CANNM报文的作用是不一样的。因此在该状态下，主动唤醒和被动唤醒发出的NM报文的周期也不一样。一般来说，主动唤醒需要向外快发NM报文，如20ms一帧，连续快发5帧（如20ms一帧），连续快发5帧。被动唤醒则按正常周期发送NM报文（如500ms一帧。这里的正常是指发出的NM报文周期与CANNM处于Normal状态时发出的NM周期一致）。
    
        一般来说，Repeat Message State状态的停留时间较短，如某车企需求为1.5s，1.5s过后，就要跳到Normal Opearation State或Ready Sleep State。

![img](https://img-blog.csdnimg.cn/242dbb47b1194ef79b9a499ba9f37165.png)

        注意：“主动请求”，当ECU处于休眠状态可被主动请求唤醒，当ECU已经处于唤醒的状态时，也是时刻在检测主动请求的。

③Normal Opearation State： 
        进入Normal Opearation State：

![img](https://img-blog.csdnimg.cn/3faeb035879842ff842be9236cf2e864.png)


        1、从Repeat Message State跳转至Normal Opearation State条件：
    
       ①当ECU主动唤醒且当Repeat Message State的时间参数满足后，主动请求还未释放时，状态跳转至Normal Opearation State
    
       或②当ECU被动唤醒且在Repeat Message State检测到主动请求，则当Repeat Message State的时间参数满足后，状态跳转至Normal Opearation State
    
        2、从Ready Sleep State跳转至Normal Opearation State条件：
    
        ①处于Ready Sleep State时检测到主动请求，状态跳转至Normal Opearation State 
    
        所谓Normal Opearation State，即正常工作模式，从上面所说的跳转条件可以看出，即只有存在主动请求时，才会跳转到Normal状态。而处于该状态时会持续发出NM报文，至于原因也很好理解，如上篇文章所说的同起同睡机制：当ECU有主动请求一直唤醒时，必须要使其他的ECU节点也保持唤醒，因此有主动请求的ECU必须持续发出CANNM报文唤醒使得其他节点不睡下去。
    
        退出Normal Opearation State：

![img](https://img-blog.csdnimg.cn/a7b2c12f8c324a009b700bdc536ef1cf.png)

    1、从Normal Opearation State跳转至Repeat Message State条件：
    
    根据Autosar的CAN网络管理标准，当CANNM处于Normal Opearation State或Ready Sleep State时接收到总线上CANNM报文的Byte1的Bit0置1时，需要把状态跳转至Repeat Message State。这个功能的作用实际上是用来检测总线上还有哪些ECU节点在线，因为如前面所说，处于Repeat Message State时需要发出CANNM报文。
    
    2、从Normal Opearation State跳转至Ready Sleep State条件：释放主动请求。

④Ready Sleep State：
        当处于Ready Sleep State时，即准备休眠状态，从字面意思也能理解，本ECU此时肯定没有主动唤醒请求，因此不向外发出CANNM报文。但若此时别的ECU有主动唤醒请求（总线持续存在其他ECU的CANNM报文），由于同起同睡机制，我们此时不能进入休眠状态，必须保持唤醒，并向外发出应用报文，持续停留在Ready Sleep State状态。

        在该状态，有个时间参数为NM-Timeout Timer（如NM-Timeout Timer = 2000ms），当接收到NM报文时，程序会将该计数器清0，若但该时间参数到达时仍未接收到CANNM报文，则认为总线上所有ECU都已经没有主动请求，所有ECU需要进入休眠状态。（另外需要注意的是，该时间参数只要CANNM进入Network Mode就会开始计时，接收或发送一帧CANNM报文时该时间参数就会清0）
    
        进入Ready Sleep State：

![img](https://img-blog.csdnimg.cn/d2093a987af045af9ee2e8d0f6288a5d.png)

    1、从Repeat Message State跳转至Ready Sleep State条件：
    
    当ECU是被动唤醒且Repeat Message State的时间参数已到达后，CANNM状态从Repeat Message State跳转至Ready Sleep State。
    
    2、从Normal Opearation State跳转至Ready Sleep State条件：
    
    本ECU释放主动请求。
    
    退出Ready Sleep State：

![img](https://img-blog.csdnimg.cn/955ba69985ad425aa8f706798bb8c3f0.png)

1、从Ready Sleep State跳转至Repeat Message State条件：

 （跳转条件与Normal Operation State跳到Repeat Message State一样）根据Autosar的CAN网络管理标准，当CANNM处于Normal Opearation State或Ready Sleep State时接收到总线上CANNM报文的Byte1的Bit0置1时，需要把状态跳转至Repeat Message State。这个功能的作用实际上是用来检测总线上还有哪些ECU节点在线，因为如前面所说，处于Repeat Message State时需要发出CANNM报文。

2、从Ready Sleep State跳转至Normal Operation State条件：

检测到主动请求

3、从Ready Sleep State跳转至Prepare Bus-Sleep Mode条件：

未接收到网管报文时间超过NM-Timeout Timer时间。此时认为总线上所有ECU都已经没有主动请求，所有ECU需要进入休眠状态



⑤Prepare Bus-Sleep Mode：
        即预休眠状态，在该状态下ECU停发应用报文和网管报文。此时的时间参数为Wait BusSleep Timer（如Wait BusSleep Timer = 5000ms），当该时间参数到达后，则CANNM进入休眠状态。

        进入Normal Opearation State条件：

![img](https://img-blog.csdnimg.cn/f7aa53a9803d46b09b5b7ac97e7f433e.png)

    见上面------“从Ready Sleep State跳转至Prepare Bus-Sleep Mode条件”
    
    退出Normal Opearation State条件：

![img](https://img-blog.csdnimg.cn/a46fc18339f04b3c9cdcb073376560eb.png)


        1、从Prepare Bus-Sleep Mode跳转至Repeat Message State条件：
    
        检测到唤醒源，主动唤醒或被动唤醒
    
        2、从Prepare Bus-Sleep Mode跳转至Bus-Sleep Mode条件：
    
        Wait BusSleep Timer已到达(在该过程中未检测到唤醒源)。

各个状态的时间参数示例如下（以某大型车企为例，不同的车企，CANNM的时间参数需求一般都不一致）

![img](https://img-blog.csdnimg.cn/1f4fb07aadec46fa9c0e5e0814b420df.png)




CANNM状态的解释大概就这些了。

下一章我将讲解CANNM报文8个字节的用途。





































