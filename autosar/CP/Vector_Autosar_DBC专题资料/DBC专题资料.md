

# Vector_Autosar_DBC专题

[TOC]



# 【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC

## 0关键字/术语描述

**CANdb++****：**

通常在数据库中管理在网络总线系统中处理的所有信息以及信息单元之间的相互关系。CANdb ++是一个数据管理程序，可用于创建和修改这些数据库。

CANdb ++软件窗口中排列了以下元素（见图0-1）：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133532302.png)图0-1

 

**Communication Matrix**：通信矩阵

在Communication Matrix中，信号，消息和网络节点之间的相互关系以表格形式显示。比如DBC转换成Excel表格的形式显示。

**DBC**：CANdb network file (Data Base for CAN)

描述CAN网络所需的所有信息都存储在CANdb network file中。CANdb network file的文件扩展名为DBC，可以使用CANdb编辑器（CAN数据库编辑器）或CANdb ++编辑器创建。

**Multiplexing****：**

通过信号复用，可以根据复用值在消息中的相同数据字节上传输不同的信号。包含多路复用值的信号称为**Multiplexor Signal**（模式信号）。根据多路复用值发送的信号称为**Multiplexed**（与模式有关）**Signals**（在此示例中为Signal_S1，...，Signal_S6）。

在MDC标准多路复用器概念中，要一起发送的多路复用信号必须每个都合并到一个**Multiplex Group**中（例如，示例中的Signal_S1和Signal_S2）。

举例：

如果多路复用值等于0，则发送信号Signal_S1和Signal_S2；如果等于1，则信号Signal_S3和Signal_S4； 如果等于2，则发送信号Signal_S5和Signal_S6（见图0-2）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133532224.png)图0-2



***\*下面来详细介绍如何使用\**\**CANdb++ Editor\*******\*创建并制作一个\**\**DBC\*******\*。\****

 

## 1 启动“CANdb++ Editor”

双击CANdb ++ Editor图标（见图1-1），启动CANdb ++后，将显示Overview窗口，在其工作区中不包含任何窗口（见图1-2）。

![图1-1](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2020121913361111.png)图1-1

 

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2020121913361110.png)图1-2

 

## 2 创建一个新的DBC（CANdb network file (Data Base for CAN)）

按照以下步骤创建新的CAN数据库文件：

a)依次选择菜单栏FileàCreate Database…(见图2-1)，打开“Template”模板对话框（见图2-2）；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219134909880.png)图2-1

 

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2020121913361115.png)图2-2

 

b) 在“Template”模板对话框中，根据需要选择合适的模板，这里以CANTemplate.dbc模板为例，单击“OK”按钮；

c)执行完b)后，弹出“New Database FIle”对话框，找到合适的路径，存放新建的DBC(CAN数据库文件)，并给这个数据库文件命名，最后单击“保存”按钮（见图2-3）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2020121913361142.png)图2-3

 

d)执行完c)后，默认打开这个新建的DBC(CAN数据库文件)，并默认显示Overview窗口（见图2-4）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2020121913361151.png)图2-4

 

## 3 创建CAN网络当中的Network nodes网络节点

在Network nodes中定义CAN网络当中的节点。

 

按照以下步骤新建CAN网络中的节点：

a)选中“Network nodes”并鼠标右键，在其上下文中选择“New”(见图3-1)，弹出“[Node](https://so.csdn.net/so/search?q=Node&spm=1001.2101.3001.7020) ‘New_Node_1’” 对话框(见图3-2)；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133611105.png)图3-1

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133611107.png)图3-2

 

b) 在“Node ‘New_Node_1’”对话框下的Definition子选项卡中定义节点的名称（见图3-3）；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133611102.png)图3-3

 

c) 依次类推，新建OBD和VCU两个网络节点（见图3-4）；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133611127.png)图3-4

 

***\*注意：\****

在现实项目当中，可更具项目实际情况定义节点的个数，这里仅仅是演示，才定义了两个节点。

 

## 4 创建CAN网络当中的Message消息

 

这里Message（消息）可以称之为Frame(帧)，也可称之为CAN ID，其用来表示CAN的标识符。一个CAN网络当中往往存在多个Message，但CAN网络中Message的CAN ID必须是唯一的，即每个CAN ID在一个CAN网络中只能使用一次。

 

按照以下步骤新建Message信号：

a)选中“Message”并鼠标右键，在其上下文中选择“New”(见图4-1)，弹出“Message‘New_ Message_1’”对话框(见图4-2)；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133635785.png)图4-1

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133635787.png)图4-2

 

b) 在“Message‘New_ Message_1’”对话框下的Definition子选项卡中定义Message的属性（见图4-3）；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133635792.png)图4-3

 

***\*注意：\****

因为在这儿之前没有添加Attribute Definitions，此时无法编辑Tx Method和Cycle Time。这将在第9章进行描述。

 

c) 在“Message‘New_ Message_1’”对话框下的Transmitters子选项卡中,单击“add”按钮（见图4-4），将该Message添加到对应的发送节点（见图4-5，图4-6）；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133635801.png)图4-4

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133635799.png)图4-5

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133635800.png)图4-6

 

c) 在“Message‘New_ Message_1’” 对话框下的Comment子选项卡中,可根据需要对该Message添加一些备注（见图4-7）；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133635803.png)图4-7

 

d) 最后单击左下角的“确定”按钮，完成Message的编辑。

 

***\*注意：\****

其它Message消息的新建，可参照该过程。这里仅描述一个Message消息。

 

## 5 创建Message消息中Signals信号

一个Message消息常存在多个Signal信号。

 

## 5.1 创建Signal信号

按照以下步骤新建Signal信号：

a)选中“Signals”并鼠标右键，在其上下文中选择“New”(见图5-1)，弹出“Signal ‘New_Signal_1’” 对话框(见图5-2)；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133635977.png)图5-1

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133635909.png)图5-2

 

b) 在“Signal ‘New_Signal_1’” 对话框下的Definition子选项卡中定义Signal信号的属性（见图5-3）；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133635982.png)图5-3

 

***\*注意：\****

因为在这儿之前没有添加Attribute Definitions，此时无法编辑Init_Value。这将在第9章进行描述。

 

b) 在“Signal ‘New_Signal_1’” 对话框下的Messages子选项卡中, 单击“add”按钮（见图5-4），将该Signal信号添加到对应的Message（见图5-5,图5-6）；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133635890.png)图5-4

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2020121913363646.png)图5-5

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133635994.png)图5-6

 

c) 在“Signal ‘New_Signal_1’” 对话框下的Comment子选项卡中,可根据需要对该Message添加一些备注（图5-7）；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219135020999.png)图5-7

 

d) 最后单击左下角的“确定”按钮，完成Signal的编辑。

 

***\*注意：\****

其它Signal信号的新建，可参照该过程。这里仅描述一个Signal消息。照此方法，Test_ID_211给添加以下信号：Voltage_value、Current_value和OBD_status（见图5-8）；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2020121913363656.png)图5-8

 

## 5.2 在Message中调整Signal信号的位置

在5.1中，我们己经将创建的Signal信号添加对应的Message消息Test_ID_211中，但CANdb ++ Editor在将Signal信号添加到Message消息时，往往不会用户期望的那样排列，这时会导致Signal信号在Message消息中的Startbit起始位置出错。这时需要手动去调整。

 

常见的调整方法有两种，下面一一说明：

a) 以Message消息Test_ID_211中Voltage_value为例。选中Voltage_value并鼠标右键，在其上下文中选择“Edit mapped Signal…”(见图5-9)。或者双击Test_ID_211下的Voltage_value信号，也有同样的效果。在弹出“Message Signal ‘Test_ID_211：：Voltage_value’”对话框下的“Definition”子选项卡中（见图5-10），查看Signal信号的Startbit起始位置是否有问题。其它的信号也按照此法一一确认。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2020121913363648.png)图5-9

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133635963.png)图5-10

 

b) 以Message消息Test_ID_211为例。选中Test_ID_211并鼠标右键，在其上下文中选择“Edit Message…”(见图5-11)。或者双击Test_ID_211消息，也有同样的效果。在弹出“Message ‘Test_ID_211（0x211）’”对话框下的“Layout”子选项卡中（见图5-12），查看各Signal信号的位置是否有异常，若存在，可手动拖曳相应信号到正确的位置。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2020121913363653.png)图5-11

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2020121913363655.png)图5-12

 

## 6 在Message中编辑每个Signal信号的接收节点

以Message消息Test_ID_211中Voltage_value为例。选中Voltage_value并鼠标右键，在其上下文中选择“Edit mapped Signal…”(见图6-1) 。或者双击Test_ID_211下的Voltage_value信号，也有同样的效果。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655261.png)图6-1

 

在弹出“Message Signal ‘Test_ID_211：：Voltage_value’”对话框中，选择“Receiver”子选项卡（见图6-2），单击“add”按钮，在弹出的“Choose Objects”的选项卡中找到信号的接收节点（见图6-3），最后单击“确定”按钮，完成Signal信号添加到对应的接收节点（见图6-4）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655265.png)图6-2

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655267.png)图6-3

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655269.png)图6-4

 

照此方法，给Test_ID_211下的其它信号添加接收的节点：

 

## 7 创建Value tables

 

Value tables可用于将符号名称分配给Signal信号或环境变量的值。举个例子，可以使用Value tables将符号名称red、yellow和green分配给Colors信号的具体值1、2和3。

 

Value tables常用来描述信号的值是否有效，以及用布尔量直观显示信号的含义。

 

下面来介绍Value tables常见的两种用法。

 

## 7.1 给信号Voltage_value添加“无效值”描述

按照以下步骤给信号添加“无效值”描述：

a)依次选择菜单栏ViewàValue Tables（见图7-1），打开Value Tables窗口（见图7-2）；

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655335.png)图7-1

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655337.png)图7-2

 

b)在空白的处，鼠标右键，在其上下文中选择“New”（见图7-3）,弹出“Value Table‘New_Value_Table_1’”对话框（见图7-4）；

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655383.png)图7-3

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655391.png)图7-4

 

c) 在“Value Table‘New_Value_Table_1’”对话框下的Definition子选项卡（见图7-5）中，定义创建的Value Tables的符号名称和备注。在Value Descriptions子选项卡（见图7-6）中，枚举出信号其中部分的值，并添加描述（下图中表示，当信号值等于0xFFFF时，用Invalid voltage代替显示）。单击“确定”按钮，完成值描述编辑；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655340.png)图7-5

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655382.png)图7-6

 

d) 信号Current_value也可以仿照此方法添加值描述。

 

## 7.2 给信号OBD_status添加“状态”描述

按照以下步骤给信号添加 “状态”描述：

a)前2个步骤与“7.1中中的a)和b)相同”；

 

b) 在“Value Table‘New_Value_Table_1’”对话框下的Definition子选项卡（见图7-7）中，定义创建的Value Tables的符号名称和备注。在Value Descriptions子选项卡（见图7-8）中，枚举出信号所有可能出现的值，并添加描述（下图中表示，当信号值等于0x0时，用Initialization代替显示；当信号值等于0x1时，用Run代替显示；当信号值等于0x2时，用Shutdown代替显示）。单击“确定”按钮，完成值描述编辑；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655407.png)图7-7

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655334.png)图7-8

 

在Value Tables窗口（见图7-9），会显示添加完成的值描述；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219134447829.png)图7-9

 

## 8 将Value tables添加至相应的Signal信号当中

以信号Voltage_value为例。在Overview窗口，双击Signals下的Voltage_value信号（见图8-1），在弹出的对话框下的Definition子选项卡中的“Value Table:”选择“Voltage_State”（见图8-2）。接着单击“确定”按钮，完成“Value Tables值描述”添加。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655408.png)图8-1

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133655419.png)图8-2

 

 

其它Signal信号的“Value Tables值描述”添加，也遵循该方法。这里不在一一描述。

 

***\*注意\****：如果项目当中没有“Value Tables”要求，可以忽略第7~8章节。因为项目的差异，“Value Tables”不一定是必要的。

 

## 9 添加Attribute Definitions属性定义

“Attribute Definitions属性定义”窗口包含用户定义属性的列表。可以添加新的用户定义属性，并可以在此处编辑现有属性。

 

常见的Attribute Definitions见下表，这一般更具项目的需要的添加响应的属性。

| **Attribute Name**      | **Attribute Type** | **Value Type** | **Value Range**                                              | **Default**           |
| ----------------------- | ------------------ | -------------- | ------------------------------------------------------------ | --------------------- |
| ProtocolType            | Network            | String         | J1939                                                        | -                     |
| BusType                 | Network            | String         | CAN                                                          | -                     |
| NmStationAddress        | Node               | Integer        | 0..254                                                       | 254                   |
| NmJ1939AAC              | Node               | Integer        | 0..1                                                         | 0                     |
| NmJ1939IndustryGroup    | Node               | Integer        | 0..7                                                         | 0                     |
| NmJ1939System           | Node               | Integer        | 0.127                                                        | 0                     |
| NmJ1939SystemInstance   | Node               | Integer        | 0..15                                                        | 0                     |
| NmJ1939Function         | Node               | Integer        | 0..255                                                       | 0                     |
| NmJ1939FunctionInstance | Node               | Integer        | 0..7                                                         | 0                     |
| NmJ1939ECUInstance      | Node               | Integer        | 0..3                                                         | 0                     |
| NmJ1939ManufacturerCode | Node               | Integer        | 0..2047                                                      | 0                     |
| NmJ1939IdentityNumber   | Node               | Integer        | 0..2097151                                                   | 0                     |
| ECU                     | Node               | String         | -                                                            | -                     |
| SigType                 | Signal             | String         | DefaultRangeRangeSignedASCIIDiscreteControlReferencePGNDTCStringDelimiterStringLengthStringLengthCtrlMessageCounterMessageChecksum | -                     |
| SPN                     | Signal             | Integer        | 0..524287                                                    | 0                     |
| GenMsgILSupport         | Message            | Enum           | NoYes                                                        | Yes                   |
| GenMsgSendType          | Message            | Enum           | CyclicNotUsedNotUsedNotUsedNotUsedNotUsedNotUsedIfActiveNoMsgSendType | NoMsgSendType         |
| GenSigILSupport         | Signal             | Enum           | NoYes                                                        | Yes                   |
| GenSigSendType          | Signal             | Enum           | CyclicOnWriteOnWriteWithRepetitionOnChangeOnChangeWithRepetitionIfActiveIfActiveWithRepetitionNoSigSendType | NoSigSendType         |
| GenMsgDelayTime         | Message            | Integer        | 0..1000 [ms]                                                 | 0                     |
| GenMsgStartDelayTime    | Message            | Integer        | 0..100000 [ms]                                               | 0                     |
| GenMsgFastOnStart       | Message            | Integer        | 0..1000000 [ms]                                              | 0                     |
| GenMsgNrOfRepetition    | Message            | Integer        | 0..1000000 [ms]                                              | 0                     |
| GenMsgCycleTime         | Message            | Integer        | 0..60000 [ms]                                                | 0                     |
| GenMsgCycleTimeFast     | Message            | Integer        | 0..1000000 [ms]                                              | 0                     |
| GenMsgRequestable       | Message            | Integer        | 0..1                                                         | 1                     |
| GenSigInactiveValue     | Signal             | Integer        | 0..1000000                                                   | 0                     |
| GenSigStartValue        | Signal             | Integer        | 0..10000                                                     | 0                     |
| GenSigEVName            | Signal             | String         | -                                                            | Env@Nodename_@Signame |

 

下面举几个常见的Attribute Definitions。

 

按照以下步骤新建Attribute Definitions：

a)依次选择菜单栏ViewàAttribute Definitions（见图9-1），打开Attribute Definitions窗口（见图9-2）。在空白的处，鼠标右键，在其上下文中选择“New”（见图9-3）,弹出“Value Table‘New_Value_Table_1’”对话框（见图9-4）；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714389.png)图9-1

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714386.png)图9-2

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714397.png)图9-3

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714407.png)图9-4

 

b) 常见的Attribute Definitions描述见第9.1~9.9章节。

 

## 9.1 添加GenericFrameRequirementNb

**属性应用**：添加该属性，可以用来给Message定义需求编号（见图9-5）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714410.png)图9-5

 

## 9.2 添加GenMsgCycleTime

**属性应用**：添加该属性，可以用来给Message定义周期，也就是CAN ID的周期（见图9-6）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714450.png)图9-6

 

## 9.3 添加GenMsgSendType

**属性应用**：添加该属性，可以用来给Message定义触发的类型，也就是CAN ID发送方式：周期帧发送，事件帧发送，混合帧发送（见图9-7）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714454.png)图9-7

 

## 9.4 添加ProjectFrameRequirementNb

**属性应用**：添加该属性，可以用来给Message定义需求编号（见图9-8）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714516.png)图9-8

 

## 9.5 添加Manufacturer

**属性应用**：添加该属性，可以用来定义制造商名称（见图9-9）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714527.png)图9-9

 

## 9.6 添加GenericSignalRequirementNb

**属性应用**：添加该属性，可以用来给Signal信号的需求编号（见图9-10）。

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714467.png)图9-10

 

## 9.7 添加GenSigStartValue

**属性应用**：添加该属性，可以用来给Signal信号的定义初始值（见图9-11）。

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714519.png)图9-11

 

## 9.8 添加ProjectSignalRequirementNb

**属性应用**：添加该属性，可以用来给Signal信号的需求编号（见图9-12）。

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714531.png)图9-12

 

定义好上述属性后，在Attribute Definitions窗口显示的效果如下（见图9-13）：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714539.png)图9-13

 

## 9.9 对比Overview下的Message和Signals显示差异

通过图9-14和9-15可以发现，这时才可以编辑Signal信号的初始值，CAN ID的发送周期，发送方式等常见的属性。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714549.png)图9-14

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133714600.png)图9-15

 

## 10 Consistency check一致性检查

为了确定CAN数据库的对象及其相互关系是否一致，可以使用CANdb ++执行自动一致性检查，查看创建/修改的DBC是否有误错误。

 

选择“FileàConsistency check”命令以启动自动一致性检查(见图10-1)。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133729343.png)图10-1

 

一致性检查的结果显示在“Consistency check”窗口中。

\>如果CANdb ++在CAN数据库中未发现任何不一致，则“一致性检查”窗口中将没有任何条目(见图10-2)；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133729350.png)图10-2

 

\>如果CANdb ++在CAN数据库中发现不一致，则会在“一致性检查”窗口中列出(见图10-3)。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219133729419.png)图10-3

**自此DBC的创建就完成了。**

 



# 【DBC专题】-2-CAN Signal信号的Multiplexor多路复用在DBC中实现

在“【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC”一文中，我们已经掌握了DBC的创建，下面我们来介绍DBC中存在的另一种应用“Signals信号的多路复用”。



## 0 关键字/术语描述

**Multiplexing****：**

通过信号复用，可以根据复用值在消息中的相同数据字节上传输不同的信号。包含多路复用值的信号称为**Multiplexor Signal**（模式信号）。根据多路复用值发送的信号称为**Multiplexed**（与模式有关）**Signals**（在此示例中为Signal_S1，...，Signal_S6）。

在MDC标准多路复用器概念中，要一起发送的多路复用信号必须每个都合并到一个**Multiplex Group**中（例如，示例中的Signal_S1和Signal_S2）。

 

举例：

如果多路复用值等于0，则发送信号Signal_S1和Signal_S2；如果等于1，则信号Signal_S3和Signal_S4； 如果等于2，则发送信号Signal_S5和Signal_S6（见图0-1）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742522.png)图0-1

 

## 1有关“多路复用”概念

 

DBC数据库中的标准“多路复用器”概念。在一条消息中，一个信号正好可以承载multiplexor value，这就是***\*multiplexor signal\****。在“Message–Signal”对话框中设置。这涉及在“Multiplexor Type”框中选择“Multiplexor Signal type”（见1-1）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742527.png)图1-1

 

**Multiplexor type**：

- Signal

​     信号没有多路复用

- Multiplexor Signal (short: Multiplexor)多路复用信号（简称：多路复用）

​    此消息的其他信号复用到其值的信号

- Multiplexed Signal多路信号

​    仅当复用信号的值与多路复用值一致时，才发送的一种信号。

 

然后，消息中具有“Multiplexor Signal type”的所有信号都取决于multiplexor signal的值。 在“Multiplex Value”输入框中设置特定值。

 

***\*注意：\****

a)每条消息的multiplexor signals的数量只有一个；

b)信号可以是multiplexor signal或multiplexed signal，但不能同时是两者。

 

## 2 创建Message中“信号多路复用”

假设Message中的Test_ID_212存在以下信号（见图2-1）：

信号Package_Num(Length：8bit);

信号Voltage_1_Value(Length：16bit)

信号Voltage_2_Value(Length：16bit)

信号Voltage_3_Value(Length：16bit)

信号Voltage_4_Value(Length：16bit)

信号Voltage_5_Value(Length：16bit)

信号Voltage_6_Value(Length：16bit)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742748.png)图2-1

 

Message的Test_ID_212新建参照 “[【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC](https://blog.csdn.net/qfmzhu/article/details/111403266)”一文。当Package_Num = 0，后面的信号表示Voltage_1_Value，Voltage_2_Value，Voltage_3_Value；当Package_Num = 1，后面的信号表示Voltage_4_Value，Voltage_5_Value，Voltage_6_Value。

 

通过上面的信息我们可以看出：信号Package_Num是Multiplexor Signal类型；信号Voltage_1_Value、Voltage_2_Value、Voltage_3_Value、Voltage_4_Value、Voltage_5_Value和Voltage_6_Value是Multiplexed Signal类型。

 

### 2.1 给Multiplexor Signal类型的信号创建合适的Value tables

 

“[【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC](https://blog.csdn.net/qfmzhu/article/details/111403266)”一文中第7章的讲述了如何创建Value tables，这里不再重复的叙述。

 

在“Value Table‘New_Value_Table_5’”对话框下的Definition子选项卡（见图2-2）中，定义创建的Value Tables的符号名称和备注。在Value Descriptions子选项卡（见图2-3）中，枚举出信号所有可能出现的值，并添加描述（下图中表示，当信号值等于0x0时，用No.1代替显示；当信号值等于0x1时，用No.2代替显示；）。单击“确定”按钮，完成值描述编辑（见图2-4）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742580.png)图2-2

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742590.png)图2-3

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742599.png)图2-4

 

### 2.2 将创建Value tables的添加到Multiplexor Signal类型的信号

 

以Message消息Test_ID_212中Package_Num为例。选中Package_Num并鼠标右键，在其上下文中选择“Edit mapped Signal…”(见图2-5) 。或者双击Test_ID_212下的Package_Num信号，也有同样的效果。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742605.png)图2-5

 

在弹出“Message Signal ‘Test_ID_212：：Package_Num’”对话框中，选择“Signal”子选项卡，在“Value Table:”处选择刚刚新建的**Value tables** “Package_Num_Value”（见图2-6）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742606.png)图2-6

 

在弹出“Message Signal ‘Test_ID_212：：Package_Num’”对话框中，选择“Definition”子选项卡，在“Multiplexortype”处选择“Multiplexor Signal”（见图2-7）。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742617.png)图2-7

 

### 2.3 配置Message中的其它Multiplexed Signal类型的信号

以Message消息Test_ID_212中Voltage_1_Value为例。选中Voltage_1_Value并鼠标右键，在其上下文中选择“Edit mapped Signal…”(见图2-8) 。或者双击Test_ID_212下的Voltage_1_Value信号，也有同样的效果。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742652.png)图2-8

 

在弹出“Message Signal ‘Test_ID_212：：Voltage_1_Value’”对话框中，选择“Definition”子选项卡，在“Multiplexortype”处选择“Multiplexed Signal”；在“Multiplex Value”处填“0x0”（见图2-9）。按照此方法依次配置信号Voltage_2_Value、Voltage_3_Value、Voltage_4_Value、Voltage_5_Value、Voltage_6_Value。

注意：配置其它信号时唯一的差异是：

信号Voltage_2_Value的“Multiplex Value”处填“0x0”；

信号Voltage_3_Value的“Multiplex Value”处填“0x0”；

信号Voltage_4_Value的“Multiplex Value”处填“0x1”；

信号Voltage_5_Value的“Multiplex Value”处填“0x1”；

信号Voltage_6_Value的“Multiplex Value”处填“0x1”；

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742659.png)图2-9

 

### 2.4 在Message中调整Signal信号的位置

“[【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC](https://blog.csdn.net/qfmzhu/article/details/111403266)”一文中第5.2章的讲述了如何调整Signal信号的位置，这里不再重复的叙述。

 

Signal信号位置调整完整后，在Message—>Layout中显示效果见下图2-10、图2-11、图2-12。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742684.png)图2-10

 

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742686.png)图2-11

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742696.png)图2-12

 

## 3 Consistency check一致性检查

“[【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC](https://blog.csdn.net/qfmzhu/article/details/111403266)”一文中第10章讲述了如何进行一致性检查，这里不再重复的叙述（见图3-1）。

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201219155742701.png)图3-1

 

***\*自此信号的多路复用就完成了。\****

## 4 结尾





# 【DBC专题】-3-利用CANdb++ Editor在DBC文件添加帧CAN_ID和信号CAN_Signal

**DBC（Data Base CAN）**文件用于描述单个CAN网络的通信，DBC文件格式比较固定、不会产生歧义和理解误差，便于交流。下面在已有的DBC中增加帧[Frame](https://so.csdn.net/so/search?q=Frame&spm=1001.2101.3001.7020) ID和信号Signal。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NQUFB2ck9YbHh6UHBVQlR0eXE5OGFUVnFBQm15YnZpYThhSG9uOXh6ZkVQMFBoOUJRUXlPVFFBLzY0MA)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1N6V2w3ZjdoT0RVY3ZoVm5sMUVsb1NuUWRQbGlhQUl2VVljaWM1MnpNYkppY3NXaHN4bng4Vnl6NVEvNjQw)

 

## 1 打开“CANdb++ Editor”，在”Signals”中增加一个信号；

1.1 右击“Signals”，在上下文中选择“New”，弹出如下对话框；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NNeXk5eWljVFJVdUI3TnN6aWNvZDloMUpRc1VzdU9EdWw1V2dwM0RJbG03S0RpYVJYMzYxZ0xFS3cvNjQw)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NNWkN4SnBtVjdabnJpYjBnSE1nRm55SDdzZHJMeHNHWmpTakYzYXpqTnJkaWNJS3BtdVNjT2ljSHcvNjQw)

 

1.2 编辑“Signal‘New_Signal_6’”对话框中的信息；

1.2.1 编辑信号基本信息：

修改前：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NNWkN4SnBtVjdabnJpYjBnSE1nRm55SDdzZHJMeHNHWmpTakYzYXpqTnJkaWNJS3BtdVNjT2ljSHcvNjQw)

 

修改后：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NGVXhLZm9OZjRHaWJ1ZWwxVWxKbUd6U1JOOVVSdDlYbXNwUE9TajdTU2xVdmhMVDN6VmJUWDNRLzY0MA)

**注：**

“Name:”表示该信号的名称；

“Length[Bit]:”表示该信号的长度，以Bit度量；

“Byte Order：”表示数据格式，有“Motorola(大端模式)”，“Intel(小端模式)”可选，根据实际情况选择；

“Value Type:”表示数据是有/无符号类型（若偏移量为0，需要表示负数，则该项选择Signed; 若偏移量为负数，需要表示负数，则该项选择Unsigned。）；

“Factor:”表示分辨率；

“Offset:”表示偏移量；

“Minimum:”和“Maximum”表示该信号实际范围；

“Init.Value:”表示该信号的初始值；

“Unit:”表示单位。

解析数据时：实际的信号物理值 = 分辨率 * CAN信号值 + 偏移量

 

1.2.2 对该信号添加备注：

修改前：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1M3RDBTTFVEV1N6QXFPaWFLS2lhZWdCb0Z5emlhdlU1SFBpYWNXZVRVaWNBZHVkRGF6b21heDExdVVRQS82NDA)

 

修改后：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NBTE1BVVlid0loRFFMZTV5bW9BNmNBbmt1N3RNdEdUQ25KY2ljaHNOVFFPdUdQV3lKY1lBaWFDQS82NDA)

 

## 2 在“CANdb++ Editor”右侧的”Message”中增加一个ID；

2.1 右击“Message”，在上下文中选择“New”，弹出如下对话框；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NzV0RIOWRxSTdkUHBjZ2ZzQ2xaSDF3MG02cnc1RnloSXR0dzB1bWZtSW8xN2VCTTVGSm1POVEvNjQw)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NHeFB5dzY2Q3F2WVVqUGFtdGp5dTlOZDZLQXBaMTExaWJRUmpPY0NOSHVORmJmYnlWVThrRTJ3LzY0MA)

 

2.2 编辑“Message‘New_Message_4’”对话框中的信息；

2.2.1 编辑ID基本信息：

修改前：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1M0UFdhbEZYemZiaEZpYWZRQTQwMzQ5bHExQ1RrMlJwdWRhd29YdnBLWGgwaGQzYW1JaEo2N3lRLzY0MA)

 

修改后：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1N6bWpJUFZhSWwwQk9adU9sNUdFTHpGYzI3dkZMa0FPRllMN0VieHc3blFhYUdlZ2libjZ5YmljUS82NDA)

 

**注：**

“Name:”表示新增ID的名称；

“ID：”用十六进制表示，如果是标准帧，范围：0~0x7FF;

“DLC：”表示新增ID实际的长度，最大为8.

 

2.2.2添加该ID的信号：

修改前：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NVa1FkSUxuZ2haOVBuUjlBVU9wQnl0QzdFOU9KRjIxNGRVRThRcGVpYjYwY1pNaWFqRHFjSVh4US82NDA)

 

修改后：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NTTjhYYkFMQTFHNERma2RTc3F5VDlOaWJpYzU1Rkh0WDVIRU1pYkFIbFJIUE56ZFF0WmRpY0dYbW5nLzY0MA)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1M5WGVQZTU4aWNBTDJ5VzNwZlRPUU40WFNMZm90MFZYSnVqT1h5TmppYWliN3IxVmswMzVxbnppY25nLzY0MA)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NNVkpHalJ2a2lhMmNpY090TGU0NkxYNG1HdkdQajZEQ0dOcllqdDIwNEIzS0xHeWJ3anRrcG1QUS82NDA)

 

2.2.3定义该ID发送节点：

修改前：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NxeEw4d2JUZ2ljWTFQWlNscklmbmljN2R1Y1JLbHpqOGZadFdRMU1PaWFrdUcwbGliMjJ3ODZNOGVBLzY0MA)

 

修改后：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NRa3dDWDI2NkVMcFRpYTBndHVrbE9SdEpjRzh2dnFoYW9DQnh4WWRNREFXZ3dUdjNUYzZRdzFBLzY0MA)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1N3M0hpYWppYTEwRkVyZUZGcWVwTHYwQk9CWmliT0l6RUxWYXVtNzFZSDVpYXloSGhvdTNXaFE3RHNnLzY0MA)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1M5Y2gzdUlmaWN4MkVvaWJZRkJvVHBwbW4yZ3ltNXNPS21pYjU2TUJUdXNncG1QUVJpY3U5akpZMkh3LzY0MA)

 

2.2.4将信号拖动到合适的位置

修改前：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NZaWNRejFhaWJBbXljcUhCVkxDWFJISmE3TkhaWHR3Ujc3azNkd3d2ajBrc1FTMVRQazNRM2lhaEEvNjQw)

 

修改后：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NyRGt6UEFQVlhSSjRwaWNMRVpHZWJPeW9pYzR4UHFwamRFM1RzRkNvRVQyekVwYVRxYVg4QWIzUS82NDA)

 

2.2.5定义该ID的周期和发送类型

修改前：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NXYlptNVFJaFZ2ajlsQ1hqM2liRWVFT0kwNU9QY204NTJaWGlhRGdPNWNuMVc3NGY5aWFNTWFFaWJBLzY0MA)

 

修改后：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1M3M2hsVmpBRnZ0RDFOZTVrV0IzcVFRSHhDWkd0a0daUDA5dUkzRk5lY0VFY21pYUx3OG5ZaEF3LzY0MA)

 

2.2.6给该ID添加备注：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NrbEhvWlJrMnNZUmU4RGlhWkVUOU5rM2pVQmNvOGZrQ3RKelJJVGhZYm4zdmNUdVhqSEhHMmt3LzY0MA)

 

## 3 对ID中的信号添加接收的节点。

3.1 双击Message里ID中的信号：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NzUWV0Z0hxa0JFalgxajdUVXV1STloWGxtUFU2R1I5WVc5UlA3SmliUWc1QWI2M2haZ01PZGljUS82NDA)

 

修改前：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1MyRGlidEl3Tmw5MGtwNlNaZDRISmZscm9ZNU1QY1pBZDFMVExEWUF4SjAzWGtxYnR4QXBsVUt3LzY0MA)

 

修改后：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NWbGtqUHZlWUh6eG1FMUoyQmVGcUFTc2ZQb1hVazR0d1ZQWVFHdEg1ZThVY0gxN3RYQjk0b3cvNjQw)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NNUkY1S3FSZ2ljR013Q1k2YjhPSUNocHBJZkNRYnUzd2t3TFZ3alc0VGppYmNFb2wxa1I3RlpVQS82NDA)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1MxUWRqWkppYVNWbklGaWMzREpLWklwNndpYnB2Vk9sV1BIV2ZVbVhRaWJMRGZpYmdRRGtMTkhpYTZ1eEEvNjQw)

 

## 4 对ID中的信号增加“值描述”。

4.1 在菜单栏中选择“View”à“Value Tables”

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NQRXhpYzh6MGJXOGRBaWM1ZVIxVmpubmVwbXoxWURWSlRSM0lGVGZrUGNCRnhLOHFqbFlzU0xzZy82NDA)

 

4.2 在空白的位置右击，并在上下文中选择“New”;

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NBUUNpYUtIZUFjbTlQYXJXNU9ZNnpNOUNlSHc5bU1KMkhDbUZTZTU3d0NabjZwcnRZUk05VFpBLzY0MA)

 

4.3 在弹出的对话框中，编辑相关信息；

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NMRDNuOHd6ZzBHWmNJa1hQOWxwRlNielJybmtJU29BRkRNNHl4aWJpYU9zMnMybU9pYXV0QWdnaWJ3LzY0MA)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NyT3VRYVNiVTBRaWNpYmZWRUJKaWJXbmxzTnZhOWtuaHZRblVvZFRTVGFmdzdET2hYWkRSSVAxMmcvNjQw)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NQaWJXWTlZcktEN2F2SDk3cTFlbk4wRmliOWFBaWNPV1c3MEQ4Z1NjNmNOdDk0SEFEMHQzVkxNUFEvNjQw)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1MxVzRhWkk1a0R4Qk9pY1hpY1dtdXkyNExZVEJHaWFRZEpUM0ZMOHlaaWFBWmsxdzV2UVJIY3YyYkFBLzY0MA)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NZanROYzVaZUVYaWJUa1c2VXVpYm5GUk5scEZpYVVuNFdUcmdUUHE1ajM0N3RyanRaRTBIZ3JLNWcvNjQw)

 

4.4 对Message中的信号增加“值描述”。

4.4.1 双击Message里ID中的信号：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NzUWV0Z0hxa0JFalgxajdUVXV1STloWGxtUFU2R1I5WVc5UlA3SmliUWc1QWI2M2haZ01PZGljUS82NDA)

4.4.2 对该信号增加“值描述”

修改前：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NWT2lhYnRhckhLOG9ubnNrUEtpY2dlYnhjRDIzaWF0QTAxcHdoZWliNFREdFVpY2VKc3JSVnVKM2tSdy82NDA)

 

修改后：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1M4TjZZRlduaFIyRTExZHZkZUw5eExDZlRQd0pvcnRWVXBJNzloM3AxU3NjejNienJmTWZPaWJBLzY0MA)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9lRU9hN0p0clBxb1p4Z3MwdG1nZWV5ODVsNXRjU0JpY1NHM2JySGVKTEVheXQ4NTFITFBGQmlhcHNRaER1OE5ZZXlsTXBOZnN0VDRqZEpvY2NBTWljNVBCQS82NDA)

 

**综上，新增CAN_ID和CAN_Signal已经完成。**

 

## **5、END**

# 【DBC专题】-4-DBC文件中的Signal信号字节顺序Motorola和Intel介绍

## 关键字：

Intel: Little endian小端
Motorola: Big endian大端
Start Bit：CAN信号的起始位
Byte order:字节顺序

## 0 引言

Message/CAN_ID中的Signal信号的“Byte order字节顺序”有两种模式：Intel和Motorola。在排列方式上，Intel模式和Motorola模式没有孰优孰劣之分，只不过是设计者的习惯。
一般情况下，发送CAN信号时，首先发送Byte0，然后Byte1，Byte2，....Byte7顺序发送，其位的编号可以参照CANdb++ Editor的Layout子选项卡中CAN数据域分布，见如图0-1。

注意：
右下角的Inverted一般默认不勾选，符合正常的二进制阅读习惯。若勾选右下角的Inverted，其显示的位排列是相反的。
![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2020122216535729%5B1%5D.png)
图0-1

在这种情况下，首先发送Byte0，最后发送Byte7的发送顺序，则在上图中可直接按照从左至右，从上至下的顺序依次对信号进行排布即可。
下面介绍Intel 格式和Motorola 格式这两种“字节顺序”排列的不同之处。

## 1 小端（Intel）编码格式

### 1.1 Signal信号不跨字节

信号在一个字节内实现（信号没有跨字节）时，该信号的高位（MSB）放在该字节的高位，该信号的低位（LSB）放在该字节的低位(见图1-1)。
起始位在该信号的低位（LSB）(见图1-2)。
![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2020122216581915%5B1%5D.png)
图1-1
![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201222165831863%5B1%5D.png)
图1-2

### 1.2 Signal信号跨字节

信号在多个字节内实现（信号跨字节）时，该信号的高位（MSB）放在高字节的高位，该信号的低位（LSB）放在低字节的低位(见图1-3)。
起始位在该信号的低位（LSB）(见图1-4)。
![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201222165851801%5B1%5D.png)
图1-3
![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201222165905254%5B1%5D.png)
图1-4

## 2 大端（Motorola）编码格式

### 2.1 Signal信号不跨字节

信号在一个字节内实现（信号没有跨字节）时，该信号的高位（MSB）放在该字节的高位，该信号的低位（LSB）放在该字节的低位(见图2-1)。
起始位在该信号的低位（LSB）(见图2-2)。
![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201222165920457%5B1%5D.png)
图2-1
![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201222165931454%5B1%5D.png)
图2-2

2.2 Signal信号跨字节

信号在多个字节内实现（信号跨字节）时，该信号的高位（MSB）放在低字节的高位，该信号的低位（LSB）放在高字节的低位(见图2-3)。
起始位在该信号的低位（LSB）(见图2-4)。
![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201222165945535%5B1%5D.png)
图2-3
![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201222165956756%5B1%5D.png)
图2-4

## 3 小结

当信号在一个字节内实现（信号不跨字节）时，Intel模式和Motorola模式的信号字节顺序，完全一样：信号的高位（MSB）放在该字节的高位，信号的低位（LSB）放在该字节的低位。
当信号在多个字节内实现（信号跨字节）时，Intel模式和Motorola模式的信号字节顺序，明显不同：
Intel模式：信号的高位（MSB）放在高字节的高位，信号的低位（LSB）放在低字节的低位；Motorola模式：信号的高位（MSB）放在低字节的高位，信号的低位（LSB）放在高字节的低位。俗称：小端模式“高在后，低在前”；大端模式“高在前，低在后”。

另：不管是Intel模式，还是Motorola模式，起始位都该信号的低位（LSB）。

# 【DBC专题】-5-DBC文件格式解析

**DBC**：CANdb network file (Data Base for CAN)

描述CAN网络所需的所有信息都存储在CANdb network file中。CANdb network file的文件扩展名为DBC，可以使用CANdb编辑器（CAN数据库编辑器）或CANdb ++编辑器创建。

 

通过前两个【DBC专题】中，我们已经掌握了DBC的创建和制作。在一些高阶应用（如DBC转Excel，DBC转XML，DBC转ARXML等等）中，了解这些显然是不够的，需要熟知其文件格式，毕竟“CANdb++ Editor”是参照某个标准，生成DBC文件的一个工具而已。

 

DBC文件描述单个CAN网络的通信。完成“[【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC](https://blog.csdn.net/qfmzhu/article/details/111403266)”一文中的第2章节，我们会得到一个Classic CAN的数据库DBC文件空模板（见下图左侧）；完成第9章节，我们会得到一个较为完成DBC文件（见下图右侧），二者的差异下图。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201223203531101.png)

 

一眼看去感觉有点“蒙”，下面我们就逐一来介绍这些“关键字”。

 

## 0 DBC文件中“符号字符串”命名要求

 

DBC文件中除了固定格式的关键字外，还有一些用户自定义的“符号字符串”，像[Node](https://so.csdn.net/so/search?q=Node&spm=1001.2101.3001.7020)，Message，Signal…的命名，其要求为：

- 以字母字符或下划线开头，并且可能进一步由字母数字字符和下划线组成。
- 长度最多为128个字符。为了与较旧的工具兼容，长度不得超过32个字符。

 

## 1 DBC文件的标头

 

**关键字**：VERSION

**关键字**：NS_  ，全称：new symbols

 

**描述：**

DBC文件包含带有版本（**关键字**：VERSION）和新符号条目（**关键字**：NS_）的标头。**关键字**：VERSION后面的双引号中通常是空的，或者是CANdb++编辑器使用的字符串。

 

该部分基本是固定的。

 

## 2 Bit Timing[波特率](https://so.csdn.net/so/search?q=波特率&spm=1001.2101.3001.7020)定义

 

**关键字**：BS_

 

Bit Timing定义了网络的波特率和BTR寄存器的设置。该部分已过时，不再使用。但是，关键字“BS_”必须出现在DBC文件中。

 

**举例：**

BS_：

 

## 3 Node节点定义

 

**关键字**：BU_

 

**格式：**

BU_: node_1_name node_2_name…

 

**描述：**

a)CAN网络汇中所有节点的名称在此处定义，且定义的名称必须唯一;

b)节点与节点之间以“空格”分隔；

c)节点的命名必须满足“符号字符串”要求。

 

**举例：**

在“[【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC](https://blog.csdn.net/qfmzhu/article/details/111403266)”一文的第3章节我们定义两个节点：VCU和OBD，其在DBC文件中的描述为:

BU_: VCU OBD

 

## 4 Value Table值表定义

 

**关键字**：VAL_TABLE_

 

**格式：**

VAL_TABLE_ value_table_name value_table_value “value_description” …0 “value_description”;

 

**描述：**

a)一个value table中以“空格”分隔；

b)value_table_name表示value table的名称, 命名必须满足“符号字符串”要求;

c) value_table_value表示value table的值，十进制表示;

d) value_description表示value table的值描述;

e)当一个value table存在多个值描述时，以（value_table_value “value_description”）的形式接着追加，value table内的两个值描述以“空格”分隔;

f)完成一个value table定义，需以“分号；”结尾；

g)多个value table需要换行。

 

**举例：**

在“[【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC](https://blog.csdn.net/qfmzhu/article/details/111403266)”一文的第7章节我们定义两个value table：Voltage_state和OBD_status_description，其在DBC文件中的描述为:

VAL_TABLE_ OBD_status_description 2 "Shutdown" 1 "Run" 0 "Initialization" ;

VAL_TABLE_ Voltage_state 65535 "Invalid voltage" ;

 

## 5 Message消息定义

 

**关键字**：BO_

 

**格式**：

BO_ message_id message_name: message_size transmitter

 

**描述：**

a)message_id表示CAN_ID，以十进制表示。在DBC文件中，CAN-ID必须唯一。如果CAN_ID的类型是“CAN Extended”,则在DBC文件中表示是“0x8000 0000 + 十六进制CAN_ID”的十进制转换；

b)message_name表示CAN_ID的消息名称，在在Message中必须是唯一, 命名必须满足“符号字符串”要求;

c) “： ” 冒号不能少；

d)message_size表示Message消息的长度，以字节为单位，十进制表示;

e)transmitter表示该Message的发送节点, 如果Message没有定义发送节点，则必须在此处输入字符串“Vector__XXX”;

 

### 5.1 标准帧Message定义举例

 

在“[【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC](https://blog.csdn.net/qfmzhu/article/details/111403266)”一文的第4章节我们定义一个Message：Test_ID_211，其在DBC文件中的描述为:

BO_ 530 Test_ID_212: 8 OBD

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201223203530968.png)

 

### 5.2 扩展帧Message定义举例

 

与4.1章节的差异是Type中选择了CAN Extended，注意在DBC文件中CAN_ID描述：

 

BO_ 2147484177 Test_ID_211: 8 OBD

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201223203530976.png)

 

## 6 Signal信号定义

 

**关键字**：SG_

 

**格式**：

SG_ signal_name multiplexer_indicator : start_bit|signal_size@byte_order+value_type (factor，offset) [minimum|maximum] “unit” receiver

 

**描述：**

a) signal_name表示信号的名称。一个Message消息中的信号名称必须唯一。多个Message消息之间可以存在相同名称信号。命名必须满足“符号字符串”要求;

b) multiplexer_indicator含义：

1. “ M”（大写）将信号定义为Multiplexor Signal。一个消息中只有一个信号可以是Multiplexor Signal；
2. “m”（小写）后跟无符号整数，将信号定义为Multiplexed Signal；
3. “空格”将信号定义为普通的Signal，表示没有复用。

c)“：”冒号不能少；

d)start_bit值指定信号在帧的数据域内的位置：Intel（小端）模式这里表示信号的LSB；Motorola（大端）模式这里表示信号的MSB；

e)signal_size表示信号的长度，以位为单位，十进制表示;

f)“@”不能少；

g)byte_order表示字节顺序。如果信号的字节顺序为Intel（小端），则byte_order为1；如果字节顺序为Motorola（大端），则byte_order为0；

h)value_type表示值类型。“+”表示无符号；“-”表示有符号；

i)(factor，offset)表示分辨率（不能为0）和偏移量，这两个值于该信号的原始值与物理值之间的转换。转换如下：物理值=原始值* factor + offset；

j)[minimum|maximum]表示最小值和最大值，定义了信号的有效物理值的范围；

k) unit表示信号的单位，为字符串类型；

l) receiver表示信号的接收方，信号的接收方可以是多个节点，接收方节点名称必须在**关键字**BU_中定义。如果信号没有定义接收方，则必须在此处输入字符串“Vector__XXX”。

 

**举例：**

在“[【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC](https://blog.csdn.net/qfmzhu/article/details/111403266)”一文的第6章节我们定义三个信号：OBD_status、Current_value和Voltage_value，其在DBC文件中的描述为:

 

BO_ 529 Test_ID_211: 8 OBD

 SG_ OBD_status : 39|8@0+ (1,0) [0|255] "-" VCU

 SG_ Current_value : 23|16@0- (0.1,0) [-300|300] "A" VCU

 SG_ Voltage_value : 7|16@0+ (0.1,0) [0|6553.5] "V" VCU

 

其在第5章生成的DBC文件效果如下：

BO_ 529 Test_ID_211: 8 OBD

 SG_ OBD_status : 39|8@0+ (1,0) [0|255] "-" Vector__XXX

 SG_ Current_value : 23|16@0- (0.1,0) [-300|300] "A" Vector__XXX

 SG_ Voltage_value : 7|16@0+ (0.1,0) [0|6553.5] "V" Vector__XXX

 

**以第****6****章为基础，拓展一下**，新定义一个DCDC节点，同时DCDC也接收者三个信号：OBD_status、Current_value和Voltage_value，其在DBC文件效果如下：当信号被多个节点接收时，在DBC中的描述，节点与节点之间用“逗号，”隔开。

BU_: DCDC VCU OBD

 

BO_ 529 Test_ID_211: 8 OBD

 SG_ OBD_status : 39|8@0+ (1,0) [0|255] "-" DCDC,VCU

 SG_ Current_value : 23|16@0- (0.1,0) [-300|300] "A" DCDC,VCU

 SG_ Voltage_value : 7|16@0+ (0.1,0) [0|6553.5] "V" DCDC,VCU

 

在“[【DBC专题】-2-CAN Signal信号的Multiplexor多路复用在DBC中实现](https://blog.csdn.net/qfmzhu/article/details/111406108)”多路复用信号在DBC文件中的描述如下：

VAL_TABLE_ Package_Num_Value 1 "No.2" 0 "N0.1" ;

 

BO_ 530 Test_ID_212: 8 OBD

 SG_ Voltage_6_Value m1 : 55|16@0+ (0.1,0) [0|6553.5] "V" VCU

 SG_ Voltage_5_Value m1 : 39|16@0+ (0.1,0) [0|6553.5] "V" VCU

 SG_ Voltage_4_Value m1 : 23|16@0+ (0.1,0) [0|6553.5] "V" VCU

 SG_ Voltage_3_Value m0 : 55|16@0+ (0.1,0) [0|6553.5] "V" VCU

 SG_ Voltage_2_Value m0 : 39|16@0+ (0.1,0) [0|6553.5] "V" VCU

 SG_ Voltage_1_Value m0 : 23|16@0+ (0.1,0) [0|6553.5] "V" VCU

 SG_ Package_Num M : 7|8@0+ (1,0) [0|1] "-" VCU

 

## 7 Value Table指标的绑定Signal信号

 

**关键字**：VAL_

 

**格式：**

VAL_ message_id signal_name  value_table_value “value_description” …… 0 “value_description”;

 

**描述：**

a) message_id同Message中的关键字描述;

b) signal_name同Signal中的关键字描述；

c) value_table_value “value_description” …… 0 “value_description”同Value Table中的关键字描述。

d)完成一个value table与信号绑定，需以“分号；”结尾；

f)多个value table与信号绑定需要换行。

 

**举例：**

在“[【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC](https://blog.csdn.net/qfmzhu/article/details/111403266)”一文的第8章节将Value Table绑定了信号，其在DBC文件中的描述为:

 

VAL_TABLE_ OBD_status_description 2 "Shutdown" 1 "Run" 0 "Initialization" ;

VAL_TABLE_ Voltage_state 65535 "Invalid voltage" ;

 

BO_ 529 Test_ID_211: 8 OBD

 SG_ OBD_status : 39|8@0+ (1,0) [0|255] "-" VCU

 SG_ Current_value : 23|16@0- (0.1,0) [-300|300] "A" VCU

 SG_ Voltage_value : 7|16@0+ (0.1,0) [0|6553.5] "V" VCU

 

VAL_ 529 OBD_status 2 "Shutdown" 1 "Run" 0 "Initialization" ;

VAL_ 529 Voltage_value 65535 "Invalid voltage" ;

## 8 Comment注释定义

 

**关键字**：CM_

 

### 8.1 Node节点的注释举例

 

**格式：**

CM_ BU_ node_name “Comment”;

 

**举例**：

CM_ BU_ VCU "VCU node in can network";

CM_ BU_ OBD "OBD node in CAN network";

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201223203530972.png)

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/202012232035316.png)

 

### 8.2 Message消息的注释举例

 

**格式：**

CM_ BO_ message_id “Comment”;

 

**举例**：

CM_ BO_ 529 "The CAN_ID of the test message is 0x211.";

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/202012232035317.png)

 

### 8.3 Signal信号的注释举例

 

**格式：**

CM_ SG_ message_id signal_name “Comment”;

 

**举例**：

CM_ SG_ 529 OBD_status "OBD working status signal";

CM_ SG_ 529 Current_value "OBD current signal.";

CM_ SG_ 529 Voltage_value "OBD voltage signal.";

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/202012232035317.png)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/202012232035316.png)

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/202012232035317.png)

 

## 9 User Defined Attribute用户定义的属性定义

 

User defined attributes是扩展DBC文件的对象属性的一种方法。这些附加属性必须使用具有属性默认值的属性定义来定义。对于具有为属性定义的值的每个对象，必须定义一个属性值。 如果未对对象定义属性值，则该对象的属性值为该属性的默认值。

 

### 9.1 Attribute属性定义

 

**关键字**：BA_DEF_

**关键字**：BA_DEF_DEF_

 

**格式**：

BA_DEF_ object_type “attribute_name”attribute_value_type ;

BA_DEF_DEF_ attribute_name attribute_value ;

 

**描述：**

a) object_type表示BU_、BO_、SG_、EV_等关键字；

b) attribute_name表示属性的名称，一般比较固定，参照相应的标准；

c) attribute_value_type表示INT、FLOAT、STRING、ENUM、HEX等类型，各自的表示发如下：

1. INT signed_integer signed_integer;
2. HEX signed_integer signed_integer;
3. FLOAT double double;
4. STRING ;
5. ENUM “char_1_string ”, “char_2_string”, “char_3_string”;

d) attribute_name同b)；

e) attribute_value表示unsigned_integer 、 signed_integer、double |char_string

f)完成一个Attribute，需以“分号；”结尾；

g)多个Attribute与信号绑定需要换行。

 

**举例：**

BA_DEF_ SG_ "ProjectSignalRequirementNb" STRING ;

BA_DEF_ SG_ "GenSigStartValue" HEX 0 0;

BA_DEF_ SG_ "GenericSignalRequirementNb" STRING ;

BA_DEF_ "Manufacturer" STRING ;

BA_DEF_ BO_ "ProjectFrameRequirementNb" STRING ;

BA_DEF_ BO_ "GenMsgSendType" ENUM "Cyclic","OnEvent","Cyclic_And_OnEvent";

BA_DEF_ BO_ "GenMsgCycleTime" INT 0 65535;

BA_DEF_ BO_ "GenericFrameRequirementNb" STRING ;

BA_DEF_DEF_ "ProjectSignalRequirementNb" "";

BA_DEF_DEF_ "GenSigStartValue" 0;

BA_DEF_DEF_ "GenericSignalRequirementNb" "";

BA_DEF_DEF_ "Manufacturer" "BMW";

BA_DEF_DEF_ "ProjectFrameRequirementNb" "";

BA_DEF_DEF_ "GenMsgSendType" "Cyclic";

BA_DEF_DEF_ "GenMsgCycleTime" 0;

BA_DEF_DEF_ "GenericFrameRequirementNb" "";

 

### 9.2 Attribute属性值

 

**关键字**：BA_

 

**格式**：

BA_ attribute_name BU_ node_name attribute_value;

BA_ attribute_name BO_ message_id attribute_value;

BA_ attribute_name SG_ message_id signal_name attribute_value;

BA_ attribute_name EV_ env_var_name attribute_value;

 

**举例**：

BA_ "GenMsgSendType" BO_ 530 2;

BA_ "GenMsgCycleTime" BO_ 530 1000;

BA_ "GenMsgSendType" BO_ 529 1;

BA_ "GenMsgCycleTime" BO_ 529 100;

BA_ "GenSigStartValue" SG_ 530 Voltage_6_Value 3000;

BA_ "GenSigStartValue" SG_ 530 Voltage_5_Value 3000;

BA_ "GenSigStartValue" SG_ 530 Voltage_4_Value 3000;

BA_ "GenSigStartValue" SG_ 530 Voltage_3_Value 3000;

BA_ "GenSigStartValue" SG_ 530 Voltage_2_Value 3000;

BA_ "GenSigStartValue" SG_ 530 Voltage_1_Value 3000;

BA_ "GenSigStartValue" SG_ 529 OBD_status 2;

BA_ "GenSigStartValue" SG_ 529 Current_value 0;

BA_ "GenSigStartValue" SG_ 529 Voltage_value 768;

 







# 【DBC专题】-6-Signal信号字节顺序Motorola_LSB/MSB/Sequential/Backward，Intel_Standard/Sequential等6类格式详解 

**关键字：**

***\*MSB\****：信号的最高位；

***\*LSB\****：信号的最低位；

***\*Motorola Forward LSB\****；

***\*Motorola Backward\****；

***\*Motorola Sequential\****；

***\*Motorola Forward MSB\****；

***\*Intel Standard\****；

***\*Intel Sequential\****；

***\*Intel\****：little endian小端；

***\*Motorola\****：big endian大端；

***\*Start bit\****：起始位；

***\*Byte order\****：字节顺序。

**推荐阅读（单击下方文字即可跳转至对应博文）：**

## 0 概述

## 1 Bit Index位索引

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20210219232154105.png)

## 2 Intel 和Motorola的细分介绍

 

### 2.1 字节顺序 “Intel Standard”

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20210219232208356.png)

### 2.2 字节顺序 “Intel Sequential”

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20210219232219359.png)

### 2.3 字节顺序 “ Motorola Forward LSB”

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2021021923223038.png)

### 2.4 字节顺序 “ Motorola Forward MSB”

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20210219232239444.png)

### 2.5 字节顺序 “Motorola Sequential”

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20210219232250777.png)

### 2.6 字节顺序 “ Motorola Backward”

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20210219232304592.png)

## 3 小结

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20210219232314151.png)



# Can报文的字节排序（Motorola Forward MSB和Motorola Forward LSB的区别）

本篇只描述Motorola格式的字节排序方式，Intel格式的不作介绍。
首先以下面的表格来表示字节顺序和位顺序，用红色背景表示高位MSB，蓝色背景表示地位LSB，绿色为LSB到MSB的过渡。
![在这里插入图片描述](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/3ff1ef4d87ff46218bd6a9e2f1c4d153.png)
下面以起始位位34，长度位12的信号来做演示来区分Motorola Forward MSB和Motorola Forward LSB的区别。
Motorola Forward MSB（大端在前）：
[矩阵](https://so.csdn.net/so/search?q=矩阵&spm=1001.2101.3001.7020)文档中起始位置则为MSB的起始位34，往高字节借位。
![在这里插入图片描述](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/3f1beef7afbc4ae39def1259af82f309.png)
填入0xB79,即101101111001，如下图：
![在这里插入图片描述](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/a7ae9799597544a8a64168af4e084c74.png)

Motorola Forward LSB（小端在前）：
矩阵文档中起始位置则为LSB的起始位34，往低字节借位。
![在这里插入图片描述](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/dd477e2cc5aa4feba028e15ec080536d.png)
填入0xB79,即101101111001，如下图：
![在这里插入图片描述](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/dc5391e393c744adae2c199a852edde5.png)
注意的是Motorola格式主要在于跨字节的区别，如果计算错了会导致发送的报文或解析的报文异常。其次要注意起始位是从第几位开始计算，Motorola Forward MSB以高位MSB为起始位，Motorola Forward LSB则以低位LSB为起始位。

**知识补充1：什么是内存的高低地址？**
如下图所示，以8个字节长度为例，Byte0为低字节，Byte7为高地址。
![在这里插入图片描述](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/130b2d9142df4311b184fea419f16636.png)
**知识补充2：什么是高低字节？**
比如说对于0xFF22来说，FF就叫做数据的高字节部分，22就是低字节部分。

**知识补充3：什么是MSB和LSB？**
MSB(most significant bit)即最高有效位，LSB(least significant bit)即最低有效位。字节计算就是从LSB到MSB的计算过程。



# [CAN总线常见的两种编码格式(Intel/Motorola)](https://www.cnblogs.com/MySunday/p/15656645.html)

CAN总线信号的常见的两种编码格式：**Intel格式与Motorola格式**。

先了解一些**大端模式**和**小端模式**，会对后面理解这两种编码格式有很大的帮助。

**一、大端模式和小端模式**

**大端模式(Big-Endian):**高字节存低地址，低字节存高地址

**小端模式(Little-Endian):**高字节存高地址，低字节存低地址

单纯的从概念描述上可能比较难理解，我们来看一个实例，十六进制数---0x12345678,分别来看一下这个数据在两种模式下的存储情况：

数据0x12345678,共四个字节，**从高字节到低字节依次为12、34、56、78**

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2593628-20211207104048654-1672589795.png)

 

将这个数据以**大端**的方式存放在数组data[3]中为：

  ![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2593628-20211207104753080-457877281.png)

 

将这个数据以小**端**的方式存放在数组data[3]中为：

 

  ![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2593628-20211207104909753-1916704361.png)

 

**二、\**Intel格式与Motorola格式\****

1.当一个信号的长度不超过1个字节(8bit),且不跨字节时，Intel格式与Motorola格式编码结果是完全一样的，如图：

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2593628-20211207114932119-1846262477.png)

 

 

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2593628-20211207121711346-296837997.png)

 

 

2.当一个信号的长度不超过1个字节(8bit),但是跨字节时，Intel格式与Motorola格式编码结果是不一样的

**MSB：高位字节  LSB：低位字节**

Motorola格式(类似于大端模式)：从高地址开始存储

 ![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2593628-20211207160833051-1039398315.png)

 

Intel格式(类似于小端模式)：从低地址开始存储

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2593628-20211207155301222-873446294.png)





# 【DBC专题】-7-在DBC中创建一个Signal Group信号组

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/original.png)

[汽车电子助手](https://blog.csdn.net/qfmzhu)![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/newCurrentTime2.png)于 2021-09-04 16:28:04 发布

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/articleReadEyes2.png)阅读量5.2k![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/tobarCollect2.png) 收藏 25

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/newHeart2023Black.png)点赞数 4

分类专栏： [# DBC](https://blog.csdn.net/qfmzhu/category_9753587.html) [# CANFD/经典CAN/CANXL](https://blog.csdn.net/qfmzhu/category_9753585.html) [Vector工具链](https://blog.csdn.net/qfmzhu/category_9753586.html) 文章标签： [dbc](https://so.csdn.net/so/search/s.do?q=dbc&t=all&o=vip&s=&l=&f=&viparticle=)

版权

[![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20190918140129601.png)CANFD/经典CAN/CANXL同时被 3 个专栏收录![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/newArrowDown1White.png)](https://blog.csdn.net/qfmzhu/category_9753585.html)

32 篇文章288 订阅

订阅专栏

[![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20190918140145169.png)Vector工具链](https://blog.csdn.net/qfmzhu/category_9753586.html)

29 篇文章120 订阅

订阅专栏

[![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20201014180756930.png)DBC](https://blog.csdn.net/qfmzhu/category_9753587.html)

14 篇文章267 订阅

订阅专栏

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20210904162015875.png)

 

**目录**

[ 1 创建Signal Group信号组](https://blog.csdn.net/qfmzhu/article/details/120101727#t0)

[ 2 创建Signal Group信号组前后DBC内容的差异](https://blog.csdn.net/qfmzhu/article/details/120101727#t1)

[3 结尾](https://blog.csdn.net/qfmzhu/article/details/120101727#t2)

------

**关键字：**
CANdb++ Editor
CAN Signal
SG=Signal Group信号组



##  1 创建Signal Group信号组![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20210904162121988.jpg)



![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/20210904162132586.jpg) 

##  2 创建Signal Group信号组前后DBC内容的差异

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/2021090416214883.jpg)

**推荐阅读（单击下方文字即可跳转至对应博文）：**

[1、【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC](https://blog.csdn.net/qfmzhu/article/details/111403266)

[2、【DBC专题】-2-CAN Signal信号的Multiplexor多路复用在DBC中实现](https://blog.csdn.net/qfmzhu/article/details/111406108)

[3、【DBC专题】-3-利用CANdb++ Editor在DBC文件添加帧CAN_ID和信号CAN_Signal](https://blog.csdn.net/qfmzhu/article/details/104534088)

[4、【DBC专题】-4-DBC文件中的Signal信号字节顺序Motorola和Intel介绍](https://blog.csdn.net/qfmzhu/article/details/111561710#comments_15057521)

[5、【DBC专题】-5-DBC文件格式解析](https://blog.csdn.net/qfmzhu/article/details/111598606)

[6、【DBC专题】-6-Signal信号字节顺序Motorola_LSB/MSB/Sequential/Backward，Intel_Standard/Sequential等6类格式详解](https://blog.csdn.net/qfmzhu/article/details/113873886)





[【Com通信】什么是*Signal* *Group*及为什么要用*Signal* *Group*](https://blog.csdn.net/qq_36056498/article/details/135171624)









# 【DBC专题】-9-如何在DBC中描述CAN Signal的“负数/值”

在[DBC](https://so.csdn.net/so/search?q=DBC&spm=1001.2101.3001.7020)中，提供两种方式：描述**CAN****信号**是“***\*负数\*******\*/\*******\*值\****”。



## 1 Value Type = Unsigned的“负数/值”CAN信号描述

CAN信号的Value Type = Unsigned。

**公式：**Phy值 = Hex值 * Factor + Offset。

注意：

- 如果一个CAN信号有负数/值，则Offset必须为负数；
- 负数区间的范围：[Offset,0)，既Offset决定了负数/值的下限。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/82456d9022f84c0ebcb406fd374ebe40.png)



### 1.1 举例：测试CAN Log

以Factor = 0.1，Offset = -40为例。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/4fce44d9a8fa425e9b69f54b4b6533d8.png)



## 2 Value Type = Signed的“负数/值”CAN信号描述

CAN信号的Value Type = Signed。



### 2.1 以Factor = 1，Offset = 0为例

以Factor = 1，Offset = 0为例。

**描述负数的方法：**

- CAN信号值的最高位（如果CAN信号长度12Bit,则最高位是Bit12）为0，则仅能描述正数，Phy值 = Hex值;
- CAN信号值的最高位（如果CAN信号长度12Bit,则最高位是Bit12）为1，则仅能描述负数，Phy值 = (-(“Hex值”取反 + 1))。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/6f9facc88a6f44c49594eb089aa64c25.png)

举例：

- Hex值 = 0xC18，其二进制：1100 0001 1000；
- “Hex值”取反，其二进制：0011 1110 0111；
- (“Hex值”取反 + 1)，其二进制：0011 1110 1000，其十进制：1000；
- (-(“Hex值”取反 + 1))  = -1000

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/8feb9c5d2aa349189e5a7ceb510f57d7.png)



### 2.2 以Factor 不等于 1，Offset不等于0为例

以Factor 不等于 1，Offset 不等于 0为例。

**描述负数的方法：**

- CAN信号值的最高位（如果CAN信号长度12Bit,则最高位是Bit12）为0，则描述负数是有条件的Phy值 = Hex值 * Factor + Offset。注意：如果Offset是负数，则Offset的绝对值 大于 Hex值 * Factor，才有可能描述负数。
- CAN信号值的最高位（如果CAN信号长度12Bit,则最高位是Bit12）为1，则描述负数是有条件的Phy值 = (-(“Hex值”取反 + 1)) * Factor + Offset。注意：a)如果Offset是负数；b)如果Offset是正数，Offset 小于 (“Hex值”取反 + 1) * Factor。只有这两种情况，才有可能描述负数。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/36aeba3a833e4f1396918a8d01d77c02.png)

举例：

- Hex值 = 0xC18，其二进制：1100 0001 1000；
- “Hex值”取反，其二进制：0011 1110 0111；
- (“Hex值”取反 + 1)，其二进制：0011 1110 1000，其十进制：1000；
- (-(“Hex值”取反 + 1)) * Factor + Offset = -1000 * 0.1 + (-40) = -140。

![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/d54999702f464eb7b4269c057e631203.png)













# **【DBC专题】-12-不同类型报文(应用/诊断/网管/测量标定)在DBC中配置，以及在Autosar各模块间的信号数据流向\******

## 1 Autosar中不同类型报文(应用/诊断/网络管理/测量标定)的信号数据流向

*以**\*经典CAN\***/**\*CANFD\***通信为例，不同类型报文(**\*应用/诊断/网络管理/测量标定\***)在**\*Autosar BSW层\***中的信号数据流向，见图1-1。*

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps11.jpg)* 

*图1-1*

****\*COM\***:Communication**

****\*DCM\***: Diagnostic Communication Manager**

****\*Ipdum\***: IPDU Multiplexer**

****\*PduR\***:PDU Router**

****\*Nm\***: Network Management**

****\*CanTp\***: CAN Transport Layer**

****\*CanIf\***: CAN Interface**

### ***\*1.1 普通APP应用报文信号数据流向\*****

*如图1-1所述：*

*Rx 接收一帧**\*普通的APP应用报文\***信号数据流向：CAN Driver – > CanIf -- > PduR -- > Com*

*Tx 发送一帧**\*普通的APP应用报文\***信号数据流向：Com – > PduR – > CanIf – > CAN Driver*

#### ***\*1.1.1 多路复用Multiplexer报文信号数据流向\****

*如图1-2所述：*

*Rx 接收一帧**\*多路复用Multiplexer报文\***信号数据流向：CAN Driver – > CanIf -- > PduR -- > Ipdum – > PduR -- > Com*

*Tx 发送一帧**\*多路复用Multiplexer报文\***信号数据流向：Com – > PduR – > Ipdum – > PduR -- > CanIf – > CAN Driver*

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps12.jpg)* 

*图1-2*

### ****\*1.2\*** [**\*UDS\***](https://so.csdn.net/so/search?q=UDS&spm=1001.2101.3001.7020)**\*/OBD诊断报文信号数据流向\*****

*如图1-1所述：*

*Rx 接收一帧**\*UDS/OBD诊断报文\***信号数据流向：CAN Driver – > CanIf -- > CanTp -- > PduR -- > Dcm*

*Tx 发送一帧**\*UDS/OBD诊断报文\***信号数据流向：Dcm – > PduR – > CanTp -- > CanIf – > CAN Driver*

### ***\*1.3 NM网络管理报文信号数据流向\****

*如图1-3所述：*

*Rx 接收一帧**\*NM网络管理\***信号数据流向：CAN Driver – > CanIf -- > CanNm -- > PduR -- > Com*

*Tx 发送一帧**\*NM网络管理\***信号数据流向：Com – > PduR – > CanNm -- > CanIf – > CAN Driver*

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps13.jpg)* 

*图1-3*

### ***\*1.4 XCP测量标定报文信号数据流向\****

*如图1-1所述：*

*Rx 接收一帧**\*XCP测量标定报文\***信号数据流向：CAN Driver – > CanIf -- > XCP*

*Tx 发送一帧**\*XCP测量标定报文\***信号数据流向：XCP -- > CanIf – > CAN Driver*

## 2 CAN DBC中如何定义不同类型报文(应用/诊断/网络管理/测量标定)

*CAN DBC中，不同的**\*Attribute属性\***定义，决定了不同类型的报文：**\*APP应用报文\***，**\*UDS/OBD诊断报文\***，**\*NM网络管理报文\***，**\*XCP测量标定报文\***。*

****\*Vector Davinci\***提供的《**\*TechnicalReference_DbcRules_Vector.pdf\***》文档中，说明了CAN DBC文件中，**\*不同类型报文\***的**\*Attributes\***（GenMsgILSupport,DiagState,DiagRequest,DiagResponse,NmAsrMessage）定义，见下表。**

| ***\*Attribute属性\****                 | ***\*GenMsgILSupport\**** | ***\*DiagState\**** | ***\*DiagRequest\**** | ***\*DiagResponse\**** | ***\*NmAsrMessage\**** |
| --------------------------------------- | ------------------------- | ------------------- | --------------------- | ---------------------- | ---------------------- |
| ***\*APP应用报文\****                   | *Yes*                     | *No*                | *No*                  | *No*                   | *No*                   |
| ***\*UDS/OBD诊断报文\****               | *No*                      | *Yes/No*            | *Yes/No*              | *Yes/No*               | *No*                   |
| ***\*NM网络管理报文\****                | *No*                      | *No*                | *No*                  | *No*                   | *Yes*                  |
| ***\*XCP\***\*测量标定\*****\*报文\**** | *No*                      | *No*                | *No*                  | *No*                   | *No*                   |

## 3 如何制作一个Autosar工具能够识别的CAN DBC

*制作一个完整的CAN DBC，过程可参考博文“[【DBC专题】-1-如何使用CANdb++ Editor创建并制作一个DBC![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps14.jpg)https://blog.csdn.net/qfmzhu/article/details/111403266](https://blog.csdn.net/qfmzhu/article/details/111403266)”，第3.2~3.5章节摘录了**\*message\***配置，需要重点关注的地方：发送节点，接收节点，属性设置差异。*

****\*Autosar工具链\***导入该**\*CAN DBC\***可参考博文：**

*[【DaVinci Configurator专题】-2-将CAN 2.0或CANFD Matrix的Arxml/DBC文件导入到CFG![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps15.jpg)https://blog.csdn.net/qfmzhu/article/details/115032771](https://blog.csdn.net/qfmzhu/article/details/115032771)*

### 3.1 节点的定义

*在Network nodes中，至少定义4个节点，见图3-1：*

****·** **\*当前所在ECU的节点名称\***：默认为DCDC；**

****·** **\*定义若干个该ECU的接收节点\***：以VCU为例，这些节点中存在NM帧发送节点；**

****·** **\*定义一个UDS/OBD诊断/测试仪节点\***：以Test为例。**

****·** **\*定义一个XCP测量标定节点\***：以MCD为例。**

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps16.jpg)* 

*图3-1*

### 3.2 普通App帧配置

#### ***\*3.2.1 普通App Tx发送帧配置\****

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps17.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps18.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps19.jpg)* 

#### ***\*3.2.2 普通App Rx接收帧配置\****

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps20.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps21.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps22.jpg)* 

### ****\*3.3 UDS/\***[**\*OBD\***]\*诊断帧配置\*****

#### ***\*3.3.1 UDS/OBD诊断功能请求帧配置\****

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps23.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps24.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps25.jpg)* 

#### ***\*3.3.2 UDS/OBD诊断物理请求帧配置\****

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps26.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps27.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps28.jpg)* 

#### ***\*3.3.3 UDS/OBD诊断诊断响应帧配置\****

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps29.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps30.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps31.jpg)* 

### 3.4 NM网络管理帧配置

#### ***\*3.4.1 NM网络管理发送帧配置\****

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps32.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps33.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps34.jpg)* 

#### ***\*3.4.2 NM网络管理接收帧配置\****

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps35.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps36.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps37.jpg)* 

### 3.5 XCP测量标定帧配置

#### ***\*3.5.1 XCP测量标定发送帧配置\*****

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps38.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps39.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps40.jpg)* 

#### ***\*3.5.2 XCP测量标定接收帧配置\****

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps41.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps42.jpg)* 

*![img](DBC%E4%B8%93%E9%A2%98%E8%B5%84%E6%96%99.assets/wps43.jpg)* 

### 3.6 附件：Autosar工具能够使用的Demo.dbc

将该内容复制到txt文件中，并将后缀txt修改为dbc，即可得到该DEMO DBC文件

```c++
VERSION ""
 
 
NS_ : 
	NS_DESC_
	CM_
	BA_DEF_
	BA_
	VAL_
	CAT_DEF_
	CAT_
	FILTER
	BA_DEF_DEF_
	EV_DATA_
	ENVVAR_DATA_
	SGTYPE_
	SGTYPE_VAL_
	BA_DEF_SGTYPE_
	BA_SGTYPE_
	SIG_TYPE_REF_
	VAL_TABLE_
	SIG_GROUP_
	SIG_VALTYPE_
	SIGTYPE_VALTYPE_
	BO_TX_BU_
	BA_DEF_REL_
	BA_REL_
	BA_DEF_DEF_REL_
	BU_SG_REL_
	BU_EV_REL_
	BU_BO_REL_
	SG_MUL_VAL_
 
BS_:
 
BU_: MCD Tester VCU DCDC
 
 
BO_ 1809 XCP_Tx_Message: 8 DCDC
 SG_ DCDC_to_MCD_Resp : 7|64@0+ (1,0) [0|0] ""  MCD
 
BO_ 1808 XCP_Rx_Message: 8 MCD
 SG_ MCD_Req : 7|64@0+ (1,0) [0|0] ""  DCDC
 
BO_ 513 NM_Tx_Message: 8 DCDC
 SG_ NM_Tx_Message_Signal : 7|8@0+ (1,0) [0|0] ""  VCU
 
BO_ 512 NM_Rx_Message: 8 VCU
 SG_ NM_Rx_Message_Signal : 7|8@0+ (1,0) [0|0] ""  DCDC
 
BO_ 1794 Diag_Response: 8 DCDC
 SG_ DCDC_to_Tester_Phy_Resp : 7|64@0+ (1,0) [0|0] ""  Tester
 
BO_ 1793 Diag_Physical_Request: 8 Tester
 SG_ Tester_Phy_Req : 7|64@0+ (1,0) [0|0] ""  DCDC
 
BO_ 1792 Diag_Function_Request: 8 Tester
 SG_ Tester_Fun_Req : 7|64@0+ (1,0) [0|0] ""  DCDC
 
BO_ 256 APP_Rx_Message: 8 VCU
 SG_ APP_Rx_Message_Signal : 7|8@0+ (1,0) [0|0] ""  DCDC
 
BO_ 257 APP_Tx_Message: 8 DCDC
 SG_ APP_Tx_Message_Signal : 7|8@0+ (1,0) [0|0] ""  VCU
 
 
 
BA_DEF_ BU_  "NmStationAddress" INT 0 127;
BA_DEF_  "NmBaseAddress" HEX 1152 1279;
BA_DEF_  "Manufacturer" STRING ;
BA_DEF_ SG_  "GenSigInactiveValue" INT 0 2147483647;
BA_DEF_ SG_  "GenSigSendType" ENUM  "Cyclic","OnWrite","OnWriteWithRepetition","OnChange","OnChangeWithRepetition","IfActive","IfActiveWithRepetition","NoSigSendType","OnChangeAndIfActive","OnChangeAndIfActiveWithRepetition","vector_leerstring";
BA_DEF_ SG_  "GenSigStartValue" INT 0 2147483647;
BA_DEF_ BO_  "DiagRequest" ENUM  "no","yes";
BA_DEF_ BO_  "DiagResponse" ENUM  "no","yes";
BA_DEF_ BO_  "DiagState" ENUM  "no","yes";
BA_DEF_ BO_  "DiagUudtResponse" ENUM  "false","true";
BA_DEF_ BO_  "NmAsrMessage" ENUM  "No","Yes";
BA_DEF_ BO_  "GenMsgCycleTime" INT 0 65535;
BA_DEF_ BO_  "GenMsgCycleTimeFast" INT 0 65535;
BA_DEF_ BO_  "GenMsgDelayTime" INT 0 65535;
BA_DEF_ BO_  "GenMsgFastOnStart" INT 0 65535;
BA_DEF_ BO_  "GenMsgILSupport" ENUM  "no","yes";
BA_DEF_ BO_  "GenMsgNrOfRepetition" INT 0 999;
BA_DEF_ BO_  "GenMsgSendType" ENUM  "Cyclic","NotUsed","NotUsed","NotUsed","NotUsed","NotUsed","NotUsed","IfActive","NoMsgSendType";
BA_DEF_ BO_  "GenMsgStartDelayTime" INT 0 65535;
BA_DEF_ BO_  "TpTxIndex" INT 0 255;
BA_DEF_  "BusType" STRING ;
BA_DEF_ SG_  "GenSigTimeoutTime" INT 0 65535;
BA_DEF_DEF_  "NmStationAddress" 0;
BA_DEF_DEF_  "NmBaseAddress" 1152;
BA_DEF_DEF_  "Manufacturer" "Vector";
BA_DEF_DEF_  "GenSigInactiveValue" 0;
BA_DEF_DEF_  "GenSigSendType" "";
BA_DEF_DEF_  "GenSigStartValue" 0;
BA_DEF_DEF_  "DiagRequest" "";
BA_DEF_DEF_  "DiagResponse" "";
BA_DEF_DEF_  "DiagState" "";
BA_DEF_DEF_  "DiagUudtResponse" "";
BA_DEF_DEF_  "NmAsrMessage" "";
BA_DEF_DEF_  "GenMsgCycleTime" 0;
BA_DEF_DEF_  "GenMsgCycleTimeFast" 0;
BA_DEF_DEF_  "GenMsgDelayTime" 0;
BA_DEF_DEF_  "GenMsgFastOnStart" 0;
BA_DEF_DEF_  "GenMsgILSupport" "";
BA_DEF_DEF_  "GenMsgNrOfRepetition" 0;
BA_DEF_DEF_  "GenMsgSendType" "Cyclic";
BA_DEF_DEF_  "GenMsgStartDelayTime" 0;
BA_DEF_DEF_  "TpTxIndex" 0;
BA_DEF_DEF_  "BusType" "CAN";
BA_DEF_DEF_  "GenSigTimeoutTime" 0;
BA_ "DiagRequest" BO_ 1809 0;
BA_ "DiagResponse" BO_ 1809 0;
BA_ "DiagState" BO_ 1809 0;
BA_ "DiagUudtResponse" BO_ 1809 0;
BA_ "NmAsrMessage" BO_ 1809 0;
BA_ "GenMsgILSupport" BO_ 1809 0;
BA_ "GenMsgSendType" BO_ 1809 8;
BA_ "DiagRequest" BO_ 1808 0;
BA_ "DiagResponse" BO_ 1808 0;
BA_ "DiagState" BO_ 1808 0;
BA_ "DiagUudtResponse" BO_ 1808 0;
BA_ "NmAsrMessage" BO_ 1808 0;
BA_ "GenMsgILSupport" BO_ 1808 0;
BA_ "GenMsgSendType" BO_ 1808 8;
BA_ "DiagRequest" BO_ 513 0;
BA_ "DiagResponse" BO_ 513 0;
BA_ "DiagState" BO_ 513 0;
BA_ "DiagUudtResponse" BO_ 513 0;
BA_ "NmAsrMessage" BO_ 513 1;
BA_ "GenMsgCycleTime" BO_ 513 200;
BA_ "GenMsgILSupport" BO_ 513 0;
BA_ "GenMsgSendType" BO_ 513 0;
BA_ "DiagRequest" BO_ 512 0;
BA_ "DiagResponse" BO_ 512 0;
BA_ "DiagState" BO_ 512 0;
BA_ "DiagUudtResponse" BO_ 512 0;
BA_ "NmAsrMessage" BO_ 512 1;
BA_ "GenMsgCycleTime" BO_ 512 200;
BA_ "GenMsgILSupport" BO_ 512 0;
BA_ "GenMsgSendType" BO_ 512 0;
BA_ "DiagRequest" BO_ 1794 0;
BA_ "DiagResponse" BO_ 1794 1;
BA_ "DiagState" BO_ 1794 0;
BA_ "DiagUudtResponse" BO_ 1794 1;
BA_ "NmAsrMessage" BO_ 1794 0;
BA_ "GenMsgILSupport" BO_ 1794 0;
BA_ "GenMsgSendType" BO_ 1794 8;
BA_ "DiagRequest" BO_ 1793 1;
BA_ "DiagResponse" BO_ 1793 0;
BA_ "DiagState" BO_ 1793 0;
BA_ "DiagUudtResponse" BO_ 1793 1;
BA_ "NmAsrMessage" BO_ 1793 0;
BA_ "GenMsgILSupport" BO_ 1793 0;
BA_ "GenMsgSendType" BO_ 1793 8;
BA_ "DiagRequest" BO_ 1792 0;
BA_ "DiagResponse" BO_ 1792 0;
BA_ "DiagState" BO_ 1792 1;
BA_ "DiagUudtResponse" BO_ 1792 1;
BA_ "NmAsrMessage" BO_ 1792 0;
BA_ "GenMsgILSupport" BO_ 1792 0;
BA_ "GenMsgSendType" BO_ 1792 8;
BA_ "DiagRequest" BO_ 256 0;
BA_ "DiagResponse" BO_ 256 0;
BA_ "DiagState" BO_ 256 0;
BA_ "DiagUudtResponse" BO_ 256 0;
BA_ "NmAsrMessage" BO_ 256 0;
BA_ "GenMsgCycleTime" BO_ 256 100;
BA_ "GenMsgILSupport" BO_ 256 1;
BA_ "GenMsgSendType" BO_ 256 0;
BA_ "GenMsgCycleTime" BO_ 257 100;
BA_ "DiagRequest" BO_ 257 0;
BA_ "DiagResponse" BO_ 257 0;
BA_ "DiagState" BO_ 257 0;
BA_ "DiagUudtResponse" BO_ 257 0;
BA_ "NmAsrMessage" BO_ 257 0;
BA_ "GenMsgCycleTimeFast" BO_ 257 20;
BA_ "GenMsgDelayTime" BO_ 257 10;
BA_ "GenMsgILSupport" BO_ 257 1;
BA_ "GenMsgNrOfRepetition" BO_ 257 3;
BA_ "GenMsgSendType" BO_ 257 7;
BA_ "GenMsgStartDelayTime" BO_ 257 10;
```



## 4 摘录：Autosar工具中使用的CAN DBC常用属性

| ***\*Attribute Name\****          | ***\*Object Type\**** | ***\*Value Type\****     | ***\*Values and Ranges\***\*(Bold = default)\****            | ***\*Description\****                                        |
| --------------------------------- | --------------------- | ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ***\*Manufacturer\****            | *Network*             | *String*                 | ***\*Vector\****                                             | *表示OEM。value必须是 " Vector "。*                          |
| ***\*BusType\****                 | *Network*             | *String*                 | ****\*CAN\***CAN FD**                                        | *定义CAN-2.0和CAN-FD网络。如果至少有一个CAN-FD报文，则必须设置为 "CAN FD"。* |
| ***\*VFrameFormat\****            | *Message*             | *Enum*                   | ****\*CAN\*** **\*Standard\***CAN ExtendedCAN FD StandardCAN FD Extended** | *表示CAN报文的种类。这个属性对每个报文都是可用的，在属性定义中没有声明。它的显示文本是 "ID-Format "或 "Type"。* |
| ***\*GenMsgILSupport\****         | *Message*             | *Enum*                   | *No: 0**\*Yes\***: 1*                                        | *表示一个消息将由COM处理。如果选择 "yes"，该信息将由COM处理，否则不处理。* |
| ***\*GenMsgSendType\****          | *Message*             | *Enum*                   | *Cyclic: 0,NotUsed,NotUsed,NotUsed,NotUsed,NotUsed,NotUsed,NotUsed,**\*NoMsgSendType:\*** **\*8\**** | *指定I-PDU的Tx行为。可以与任何类型的GenSigSendType相结合。*  |
| ***\*GenSigSendType\****          | *Signal*              | *Enum*                   | *Cyclic: 0,OnWrite: 1,OnWriteWithRepetition: 2,OnChange: 3,OnChangeWithRepetition: 4, NotUsed,NotUsed,**\*NoSigSendType:\*** **\*7\**** | *指定一个信号的Tx行为。OnChange仅支持<=4 Byte的信号。请注意：带重复的发送类型和不带重复的发送类型的组合将导致信息在任何时候都是带重复的发送。* |
| ***\*GenMsgCycleTime\****         | *Message*             | *Integer*                | ****\*0\***..65535**                                         | *每次循环发送信息之间的时间，单位是毫秒。*                   |
| ***\*GenMsgCycleTimeFast\****     | *Message*             | *Integer*                | ****\*0\***..65535**                                         | *如果至少有一个IfActiveSignal的默认值不同，则每次循环发送消息之间的时间（ms）。也适用于有重复的消息（即GenMsgNrOfRepetition > 0）。每次重复的时间间隔。* |
| ***\*GenSigStartValue\****        | *Signal*              | ****\*Integer\***Float** | ****\*0\***..2147483647**                                    | *这个值是信号的默认值。字符串值类型可以表示十六进制和整数值。* |
| ***\*GenSigInactiveValue\****     | *Signal*              | *Integer*                | ****\*0\***..2147483647**                                    | *表示信号的无效值。*                                         |
| ***\*GenMsgDelayTime\****         | *Message*             | *Integer*                | ****\*0\***..65535**                                         | *这是具有相同标识符的不同信息发送之间的最小时间，单位是ms。* |
| ***\*GenMsgStartDelayTime\****    | *Message*             | *Integer*                | ****\*0\***..65535**                                         | *这定义了Com_IpduGroupStart和这个I-PDU的循环部分的第一次发送之间的时间，单位是ms。* |
| ***\*GenMsgNrOfRepetition\****    | *Message*             | *Integer*                | ****\*0\***..255**                                           | *在一个初始发送请求之后的发送重复次数。重复之间的时间必须使用dbc属性GenMsgCycleTimeFast来定义。* |
| ***\*GenSigTimeoutTime_<Ecu>\**** | *Signal*              | *Integer*                | ****\*0\***..65535**                                         | *用于特定节点收到的该信号的超时时间（ms）。如果为一个消息配置了不同的GenSigTimeoutTime值，并且没有使用更新位，那么最低的超时时间（最强的定义）被用于超时监测。必须为每个接收此信号的ECU提供一个专门的属性定义（GenSigTimeoutTime_<Ecu>）。* |
| ***\*NmAsrMessage\****            | *Message*             | *Enum*                   | ****\*No\*** = 0,Yes = 1**                                   | *该属性定义了相应的消息是否是AUTOSAR NM消息*                 |
| ***\*DiagState\****               | *Message*             | *Enum*                   | ****\*No\*** = 0,Yes = 1**                                   | *设置为 "yes"，用于> Functional (UDS) requestCanTp将使用Normal addressing。* |
| ***\*DiagRequest\****             | *Message*             | *Enum*                   | ****\*No\*** = 0,Yes = 1**                                   | *设置为 "yes"，用于> Physical RequestCanTp将使用Normal addressing。* |
| ***\*DiagResponse\****            | *Message*             | *Enum*                   | ****\*No\*** = 0,Yes = 1**                                   | 设置为 "是"，用于。> Physical ResponseCanTp将使用Normal addressing。 |



 

 