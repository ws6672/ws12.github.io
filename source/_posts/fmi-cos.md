---
title: FMI2.0 ———— 协同仿真
date: 2021-05-27 10:58:12
tags: [co-simulation]
---

本文是来自于FMI2.0规范的第四章协同仿真的译文，基于机翻与个人理解修订而成。

# 一、开题

本文定义了函数模拟接口（FMI），用于在协同仿真环境中耦合两个或更多仿真模型（FMI进行共模）。协同仿真是一种相当常见的方法，用于耦合仿真技术系统和耦合工程中的物理现象，着重于稳定性（时间依赖性）问题。
 
 > 耦合仿真(coupling simulation ):应用共享网格技术,解决数字化环境下多学科CAE视图模型耦合仿真的难题。
耦合：两个或两个以上的电路构成一个网络时，若其中某一电路中电流或电压发生变化，能影响到其他电路也发生类似的变化，这种网络叫做耦合电路。耦合的作用就是把某一电路的能量输送（或转换）到其他的电路中去 。
时间依赖性：time-dependent

FMI for Co-Simulation：协同仿真的FMI

协同仿真的设计是为了与子系统模型耦合，子系统模型已由其仿真器及其求解器导出为可运行代码；还用于仿真工具的耦合（仿真器耦合，工具耦合）。

![协同仿真](/image/co-simulation/cos.png)

在工具耦合情况下，FMU实现将FMI函数调用包装到由仿真工具（例如COM或CORBA API）提供的API调用中。除FMU外，还需要使用仿真工具来进行协同仿真。最常见的是，基于工具耦合的协同仿真是在分布式硬件上实现的，子系统由可能具有不同OS（群集计算机、计算机场、位于不同位置的计算机）的不同计算机处理。

子系统之间的数据交换和通信通常使用一种网络通信技术（例如MPI，TCP / IP）完成。该通信层的定义不是FMI标准的一部分，但可以使用FMI来实现分布式协同仿真方案，如下图所示
![分布式协同仿真基础结构](/image/co-simulation/cos-distributed.png)

主机必须实现通信层，用于建立网络通信的其他参数（例如，远程计算机的标识，端口号，用户帐户）将通过主机的GUI进行设置，而这些数据都不会通过FMI API传输。


# 二、数学描述

1. 基础

协同仿真在仿真过程的所有阶段中利用耦合问题的模块化结构，从针对不同仿真工具（可以是功能强大的仿真器以及简单的C程序）中各个子系统的单独的模型设置和预处理开始。

在时间积分时，将再次针对所有子系统独立执行仿真，从而将子系统之间的数据交换限制为离散的通信点tc<sub>i<sub>。对于模拟耦合，还可以在它的本机仿真工具中为每个子系统分别完成仿真数据的可视化和后处理。在不同的上下文中，通信点tc<sub>i<sub>、通信步长tc<sub>i<sub>->tc<sub>i+1<sub> 以及通信步长个数`hc<sub>i<sub>:=tc<sub>i+1<sub>- tc<sub>i<sub>` 分别称为取样点（同步点），宏步长和取样率。FMI中用于协同仿真的术语“通信点”是指协同仿真环境中子系统之间的通信，不应与用于将仿真结果保存到文件中的输出点混合使用，

> 取样：指的是取样的时间点
宏步长：指的是两次取样的时间间隔
取样率（Sampling Rate）是指在数码音频和视频技术应用中，当进行模拟/数码转换时，每秒钟对模拟信号进行取样时的快慢次数。例如，CD和MD的取样率为44.1kHz，表示每秒钟对模拟音频信号进行了44100次取样。取样率越高，转换的精度就越高，重放出的模拟信号波形就越接近原始的模拟信号波形。取样率的高低。决定了所能转换模拟信号的频率上限。

协同仿真提供了一个接口标准，用于解决时间连续（由平稳差分方程描述的模型组件）或时间离散（由差分方程描述的模型组件，例如，例如离散控制器）的子系统。在耦合系统的块表示中，子系统由具有（内部）状态变量<i>x(t)</i>的块表示，这些状态变量通过子系统输入 <i>u(t)</i> 和子系统输出 <i>y(t)</i> 连接到其他具有耦合问题的子系统（模块）的模块。在这个框架中，子系统之间的物理连接由所有子系统的输入 <i>u(t)</i> 和输出 <i>y(t)</i> 之间的数学耦合条件表示。

![通信点的数据流](/image/co-simulation/cos-data-flow.png)
> 差分方程：包含未知函数的差分及自变数的方程。在求微分方程*的数值解时，常把其中的微分用相应的差分来近似，所导出的方程就是差分方程。通过解差分方程来求微分方程的近似解，是连续问题离散化的一个例子。 

为了进行协同仿真，必须实现两个基本函数组：
子系统之间的数据交换函数
用于算法问题的函数，以同步所有子系统的仿真，并从初始时间<i>tc<sub>0</sub></i>:= <i>t<sub>start</sub></i>到结束时间<i>tc<sub>N</sub></i>：= <i>t<sub>stop</sub></i>执行通信步长<i>tc<sub>i</sub></i>-><i>tc<sub>i+1</sub></i>。

在用于协同仿真的FMI中，这两个功能都在一个软件组件中实现，即协同仿真主机（Master）。子系统（Slave）之间的数据交换仅通过主站进行，从站之间没有直接通信。可以通过专用软件工具（单独的仿真底板）或所涉及的仿真工具之一来实现主函数。最常见的例子，可以在嵌套的协同仿真环境中对耦合系统进行仿真，并且用于协同仿真的FMI适用于层次结构的每个级别。

用于协同仿真的FMI定义了在协同仿真环境中主机与所有从机（子系统）之间进行通信的接口例程。最常见的主站算法是在每个通信点tc<sub>i</sub>停止所有从站的仿真（时间积分），收集所有子系统的输出y（tc<sub>i</sub>），调整子系统输入u（tc<sub>i</sub>），将这些子系统输入分配给从站，然后继续与下一个交互步骤`tc<sub>i</sub>->tc<sub>i+1</sub> = tc<sub>i</sub> + hc`的（协同）仿真，交互步长为固定参数hc。在每个从站中，适合的求解器应当被用于给定交互步长 tc<sub>i</sub>->tc<sub>i+1</sub> 的子系统之一求积分。最简单的协同仿真算法通过冻结数据u(tc<sub>i</sub>)来近似 tc<sub>i</sub>≤t <tc<sub>i</sub> + 1的（未知）子系统输入u(t),(t> tc<sub>i</sub>)。用于协同仿真的FMI支持这种经典的暴力解法以及更复杂的主算法，旨在支持一类非常通用主算法，但它本身并未定义主算法。

从站支持更复杂的主站算法的能力，在从站的XML描述中包含一组功能标志（capability flags）。典型的例子是
+	具有处理可变的通信步长hc<sub>i</sub>的 能力
+	以减小的通信步长重复拒绝的通信步长tc<sub>i</sub>≤t <tc<sub>i</sub> + 1的 能力
+	提供衍生工具的能力允许插值的输出时间
+	提供雅可比行列式的能力


用于协同仿真的FMI相关流程如下
+	该流程从实例化和初始化（准备好所有从设备进行计算，建立通信链接）开始
+	然后进行仿真（强制从设备模拟通信step）
+	最后在完成时关闭

2. 数学模型

本节包含一个协同仿真FMU的正式数学模型。做出以下基本假设：
+	从模拟器被主模拟器视为纯粹的采样数据系统，可以是
	+	“真实的”采样数据系统（离散控制器；输入和输出可以是Real，Integer，Boolean，String或枚举类型。此类变量的定义为variability =“ discrete”；最小FMU外部可访问的采样周期由元素DefaultExperiment中的属性stepSize定义）
	+	集成在通信点之间的混合ODE（称为“对时间连续系统的采样访问”），在其中可能发生并处理内部事件，但从FMU外部看不到事件。在此假定此混合ODE的所有输入和所有输出均为实信号（以variability =“ continuous”定义）
	+	上述系统的组合

+	主站和从站之间的通信仅在一组离散的时刻进行，这些时间点称为通信点。

FMI协同仿真模型相关变量描述：
+	t：自变量time∈ℝ（causality =“independent”定义的变量）;第i个通信点表示为t=tc<sub>i</sub>,通信步长表示为hc<sub>i</sub> = tc<sub>i+1</sub> - tc<sub>i</sub>
+	v：所有暴露变量的向量
+	u(tc<sub>i</sub>)：输入变量
+	y(tc<sub>i</sub>)：输出变量
+	w(tc<sub>i</sub>)：不能用于FMU连接的FMU的局部变量,通过causality = "local"定义
+	x<sub>c</sub>(t)：用实际连续时间变量的向量表示连续时间状态
+	x<sub>d</sub>(t)：离散时间变量
+	<sup>.</sup>x<sub>d</sub>(t)：上一个离散时间变量

存在如下定义：
+	在通信点上，主站向从站提供通用输入
+	从机向主机提供通用输出
+	Initialization：从站是一个采样数据系统，其内部状态（连续时间或离散时间都没关系）需要初始化为t=tc<sub>0</sub>。这是通过辅助函数执行的[此关系在<ModelStructure> <InitialUnknowns>下的xml文件中定义]。计算FMI协同仿真模型的解意味着将解过程分为两个阶段，并且在每个阶段中使用不同的方程式和解方法。可以根据以下模式对阶段进行分类
	+	Initialization Mode：如果从站与其他模型循环连接，则可以对FMU方程进行迭代。在这种模式下，代数方程被求解
	+	Step Mode：通过对常微分、代数和离散方程进行数值求解，此模式用于计算通信点上所有（实际）连续时间和离散时间变量的值。如果从站与其他模型循环连接，则不可能对FMU方程进行迭代


下表中使用的函数fmi2SetXXX是fmi2SetReal，fmi2SetBoolean，fmi2SetInteger和fmi2SetString的缩写。函数fmi2GetXXX是函数fmi2GetReal，fmi2GetBoolean，fmi2GetInteger和fmi2GetString的缩写

|方程式 |FMI函数|
|:--|:--|
|初始化模式之前的方程式（状态机中的“实例化”）||
|设置 i=0并设置自变量的tc<sub>i</sub>|fmi2SetupExperiment|
|设置变量v<sub>initial=exact</sub>以及v<sub>initial=approx</sub>|fmi2SetXXX|
|初始化模式之后的方程式||
|在 t=t<sub>0</sub>时进入初始化模式（激活初始化，离散时间和连续时间方程式）|fmi2EnterInitializationMode|
|设置变量v<sub>initial=exact</sub>（包括初始值为x<sub>c,initial=exact</sub>独立参数p和连续时间状态）|fmi2SetXXX|
|设置连续时间以及离散时间输入 u<sub>c+d</sub>(tc<sub>0</sub>)以及可以选择设置连续时间导数的输入u<sub>c</sub><sup>j</sup>(tc<sub>0</sub>)|fmi2SetXXX、fmi2SetRealInputDerivative|
| v<sub>InitialUnknows</sub>:=f<sub>init</sub>(u<sub>c</sub>,u<sub>d</sub>,t<sub>0</sub>,v<sub>initial=exact</sub>)|fmi2GetXXX、fmi2GetDirectionalDerivativ|
|退出初始化模式（停用初始化方程式）|fmi2ExitInitializationMode|
|Step模式下的方程式（状态机中的“ stepComplete”，“ stepInProgress”||
|设置独立的可调参数p<sub>tune</sub>（不要设置其他参数p<sub>other</sub>）|fmi2SetXXX|
|设置参数连续时间和离散时间输入 u<sub>c+d</sub>(tc<sub>i</sub>)以及可选参数连续时间输入 u<sub>c</sub><sup>(j)</sup>(tc<sub>i</sub>)的导数|fmi2SetXXX、fmi2SetRealInputDerivative|
|tc<sub>i+1</sub>:=tc<sub>i</sub>+hc<sub>i</sub><br/> (y<sub>c+d</sub>,y<sub>c</sub><sup>(j)</sup>,w<sub>c+d</sub>)<sub>tc<sub>i+1</sub></sub>	:= f<sub>doStep</sub>(u<sub>c+d</sub>,u<sub>c</sub><sup>(j)</sup>,tc<sub>i</sub>,hc<sub>i</sub>,p<sub>tune</sub>,p<sub>other</sub>)<sub>tc<sub>i</sub></sub><br/>tc<sub>i</sub>:=tc<sub>i+1</sub><br/>其中f<sub>doStep</sub>也是内部变量的函数|fmi2DoStep<br/>fmi2GetXXX<br/>fmi2GetRealOutputDerivatives<br/>fmi2GetDirectionalDerivative|



# 三、FMI应用程序编程接口

本节包含用于从C程序访问协同仿真从站的输入/输出数据和状态信息的接口说明。

1. 输入/输出值和参数的传输

输入变量、输出变量以及变量 通过本节中定义的fmi2GetXXX和fmi2SetXXX函数进行传输。

为了使从机能够在通信步骤之间内插连续的实际输入，可以提供关于时间的输入导数。为了允许更高阶的插值，还可以设置更高的导数。从属是否能够和需要插值，此信息由函数属性canInterpolateInputs提供。

1.1 fmi2SetRealInputDerivatives

```
-- 设置实数输入变量的第n次时间导数
fmi2Status fmi2SetRealInputDerivatives(fmi2Component c, constfmi2ValueReference vr[], size_t nvr, constfmi2Integer order[], constfmi2Real value[])
```
+	“vr”：是值引用的向量，这些值定义了应设置其导数的变量。
+	数组“order”：包含各个导数的阶（1表示一阶导数，不允许0）。
+	参数“value”：是带有导数值的向量。 
+	“nvr”：是向量的维数

使用该函数的限制与fmi2SetReal函数相同。输入和它们的派生是相对于通信时间步长的开始进行设置的。为了允许在通讯步骤之间对实际输出变量进行插值/逼近（如果将它们用作其他从站的输入），则可以读取输出相对于时间的导数。从站是否能够提供输出的派生由无符号整数类型的函数标志`MaxOutputDerivativeOrder`决定，它提供了输出导数的最大阶数。如果实际顺序较低（因为积分算法的顺序较低），则检索到的值为0。如果内部多项式为1阶，并且主机查询输出的二阶导数，则从机将返回零。

1.2 fmi2GetRealOutputDerivatives 

可以通过以下方式检索导数：

```
-- 检索输出值的第n个导数
fmi2Status fmi2GetRealOutputDerivatives (fmi2Component c, constfmi2ValueReference vr[], size_t nvr, constfmi2Integer order[], fmi2Real value[]);
```

+	“vr”：是值引用的向量，这些值定义了应检索其导数的变量。
+	数组“order”：包含各个导数的阶（1表示一阶导数，不允许0）。
+	参数“value”：是带有导数值的向量。 
+	“nvr”：是向量的维数

返回的输出对应于当前从站时间。例如在成功执行fmi2DoStep（...）之后，返回的值与通信时间步长有关。

2. 计算

2.1 fmi2DoStep
时间步长的计算由以下函数控制
```
fmi2Status fmi2DoStep(fmi2Component c, fmi2Real currentCommunicationPoint, fmi2Real communicationStepSize, fmi2Boolean noSetFMUStatePriorToCurrentPoint)
```

该函数表示开始计算时间步。参数currentCommunicationPoint是主机的当前通信点（tc<sub>i<sub>），参数communicationStepSize是通信步长（hc<sub>i<sub> > 0.0）。从站必须集成，直到tc<sub>i+1<sub> = tc<sub>i<sub> +hc<sub>i<sub>.【调用环境定义了通信点，并且fmi2DoStep必须通过始终准确地在 tc<sub>i<sub> +hc<sub>i<sub> 时与这些点同步。如何实现此目标取决于fmi2DoStep】。

在调用fmi2ExitInitializationMode之后第一次调用fmiDoStep时，currentCommunicationPoint必须等于由fmi2SetupExperiment设置的startTime。【不需要正式的参数currentCommunicationPoint。存在它是为了解决主节点与从属节点间FMU状态不匹配的问题：由之前的fmi2DoStepor或fmi2SetFMUStatecall定义的从属节点的currentCommunicationPoint和FMU状态必须彼此一致】

例如，如果从站未按照上述要求对自变量使用更新公式（tc<sub>i+1<sub> = tc<sub>i<sub> +hc<sub>i<sub>），则（使用fmi2DoStep的参数= currentCommunicationPoint）在内部使用自己的更新公式，例如tc<sub>s,i+1<sub> = tc<sub>s,i<sub> +hc<sub>s,i<sub> 可以使用时间增量hc<sub>s,i<sub> :=(tc<sub>i<sub>-tc<sub>s,i<sub>)+hc<sub>i<sub>(替代hc<sub>s,i<sub> :=hc<sub>i<sub>)去避免主机时间tc<sub>i+1<sub>和从机内部时间tc<sub>s,i+1<sub>不匹配度过大。

如果在此模拟运行中在currentCommunicationPoint之前的某个时刻不再调用fmi2SetFMUState [从站可以使用该标志刷新结果缓冲区]，参数`noSetFMUStatePriorToCurrentPoint` = “fmi2True”。

函数返回值：
+	fmi2OK-通信步骤已成功计算到结束
+	fmi2Discard –从站成功计算出通讯步骤的一个子间隔
+	fmi2Fatal –发生错误，导致FMU无法修复损坏
+	fmi2Pending –从站异步执行功能。主机必须调用fmi2GetStatus（...，fmi2DoStep，...）来确定从机是否完成。如果返回这个值，那么fmi2DoStep在此期间不允许调用其它函数。

2.2 fmi2CancelStep

如果fmi2DoStep返回了fmi2Pending，则可以调用该命令以停止当前的异步执行。

```
fmi2Status fmi2CancelStep(fmi2Component c);
```
如果例如用户或从机之一停止了协同仿真运行，则主机调用此功能。之后，仅允许调用fmi2Reset或fmi2FreeInstance

3. 获取从站的状态信息
通过以下功能从从站检索 状态信息：
```
 fmi2GetStatus(fmi2Component c,constfmi2StatusKind s, fmi2Status* value); 
fmi2Status fmi2GetRealStatus   (fmi2Component c, constfmi2StatusKind s,fmi2Real* value); 
fmi2Status fmi2GetIntegerStatus(fmi2Component c, constfmi2StatusKind s, fmi2Integer* value); 
fmi2Status fmi2GetBooleanStatus(fmi2Component c, constfmi2StatusKind s, fmi2Boolean* value); 
fmi2Status fmi2GetStringStatus (fmi2Component c, constfmi2StatusKind s, fmi2String* value);

-- 定义要查询从站的状态信息
typedefenum{
	fmi2DoStepStatus, //步进
	fmi2PendingStatus,  //异步状态
	fmi2LastSuccessfulTime, //返回上一个成功完成的通信步骤的结束时间
	fmi2Terminated //终止
} fmi2StatusKind

typedef enum {
    fmi2OK, // 通信步骤已成功计算到结束
    fmi2Warning, // 警告
    fmi2Discard, // 从站成功计算出通讯步骤的一个子间隔
    fmi2Error, // 错误
    fmi2Fatal, // 发生错误，导致FMU无法修复损坏
    fmi2Pending // 异步
} fmi2Status;


```

通知主机有关模拟运行的实际状态。由参数fmi2StatusKind指定要返回的状态信息。取决于从站的功能，从站可以提供哪些状态信息（请参见4.3.1）。如果需要从站无法检索的状态，它将返回 fmi2Discard。

结构体fmi2StatusKind包含以下状态：

|状态|类型|描述|
|:--|:i2DoStep函数返回fmi2Pending时可以调用。如果计算未完成，该函数将提供fmi2Pending。否则，该函数将返回异步执行的fmi2DoStep调用的结果。|
|fmi2PendingStatus|fmi2String|当fmi2DoStep函数返回fmi2Pending时可以调用。该函数提供一个字符串，该字符串告知当前正在运行的异步fmi2DoStep计算的状态|
|fmi2LastSuccessfulTime|fmi2Real|返回上一个成功完成的通信步骤的结束时间。可以在fmi2DoStep（...）返回fmi2Discard之后调用。|
|fmi2Terminate|fmi2Boolean|如果从站希望终止仿真，则返回true。可以在fmi2DoStep（...）返回fmi2Discard之后调用。使用fmi2LastSuccessfulTime确定从站终止的时刻|

3. 从主站到从站调用次序的状态机

以下状态机定义了fmi规范所支持的调用序列

[以UML 2.0状态机形式表示的协同仿真C函数的调用序列](/image/co-simulation/cos-status-machine.png)

状态机的每个状态对应于模拟的某个特定阶段，如下所示：

+	instantiated（实例化）：在这种状态下，可以设置起始值和估计值（变量属性initial = “exact” or “approx.”）

+	Initialization Mode（初始化模式）：在这种状态下，方程将激活以确定所有输出（以及导出工具公开的其他可选变量）。可以通过fmi2GetXXX调用检索的变量是在xml文件中的<ModelStructure> <InitialUnknowns>下定义的或者causality="output"的变量。可以设置initial="exact"的变量以及具有variability="input"的变量
+	slaveInitialized（从机初始化）：在这种状态下，将对从站进行初始化，并执行协同仿真计算。使用功能“ fmi2DoStep”执行直到下一个通讯点的计算。根据返回值，从站处于不同的状态（step complete, step failed, step canceled）。

+	terminated（终止）：在这种状态下，可以获取仿真最后时刻的解。

注意，在初始化模式下，可以根据xml文件中的元素<ModelStructure>、 <InitialUnknowns>定义的模型结构，使用fmi2SetXXX设置输入变量，并使用fmi2GetXXX互换获取输出变量。【例如，如果一个输出y1取决于两个输入u1，u2，则必须先设置这两个输入，然后才能获取y1。 如果另外输出y2取决于输入u3，则可以设置u3，然后再获取y2。 结果，可以通过使用适当的数值算法来处理初始化模式下连接的FMU上的人工或“真实”代数环。】

“slaveInitialized”状态还有一个额外的限制，即不允许在fmi2SetXXX函数之后调用fmi2GetXXX函数，而在两者之间没有fmi2DoStep调用。

原因是要避免对缓存进行不同的解释，因为与ModelExchange的FMI相反，fmi2DoStep将执行实际的计算，而不是fmi2GetXXX，因此，通信点处的虚拟代数循环无法由适当的fmi2GetXXX序列处理，fmi2SetXXX调用与ModelExchange'

# FMU协同仿真（CoSimulation）

1. 标签的定义

协同仿真功能在模型描述文件中的 <CoSimulation></CoSimulation>标签定义，相关定义如下：

[CoSimulation](/image/co-simulation/cos-CoSimulation.png)

相关结构如下：
+	CoSimulation
	+	modelIdentifier：类名缩写
	+	needsExecution：决定是否需要外部工具执行模型
	+	canHandleVariableCommunicationStepSize：从站可以处理可变的通信步长。对于每个校准，通信步长（fmi2DoStep函数的参数communicationStepSize）在每次调用中不需要固定不变
	+	canInterpolateInputs：从站能够对连续输入插值
	+	maxOutputDerivativeOrder：从站能够提供最大阶数的输出导数
	+	canRunAsynchronuously ：异步
	+	canBeInstantiatedOnlyOncePerProcess：单FMU单实例（如果需要多个实例，则必须在不同的进程中实例化 FMU）
	+	canNotUseMemoryManagementFunctions：如果为true，则从机使用自身的函数进行内存分配和释放；忽略fmi2Instantiate中给出的回调函数 allocateMemory、freeMemory
	+	canGetAndSetFMUstate：如果为true，则 仿真环境可以查询和恢复内部FMU状态（即支持fmi2GetFMUstate,fmi2SetFMUstate,fmi2FreeFMUstate）
	+	canSerializeFMUstate：如果为true，则仿真环境可以序列化内部FMU状态，即FMU支持fmi2SerializedFMUstateSize，fmi2SerializeFMUstate，fmi2DeSerializeFMUstate。如果是这种情况，则标记canGetAndSetFMUstate也必须为true。
	+	providesDirectionalDerivative：如果为true，在通信点时可以使用fmi2GetDirectionalDerivative（..）计算方程的方向导数。
	+	SourceFiles：资源文件
	
2. 实例

```
<?xml version="1.0" encoding="UTF8"?>
<fmiModelDescription   fmiVersion="2.0"   modelName="MyLibrary.SpringMassDamper"   guid="{8c4e810f-3df3-4a00-8276-176fa3c9f9e0}"   description="Rotational Spring Mass Damper System"   version="1.0" generationDateAndTime="2011-09-23T16:57:33Z"   variableNamingConvention="structured">

<CoSimulation modelIdentifier="MyLibrary_SpringMassDamper" canHandleVariableCommunicationStepSize="true"     canInterpolateInputs="true"/>

<UnitDefinitions>
	<Unit name="rad">
		<BaseUnit rad="1"/>
		<DisplayUnit name="deg" factor="57.2957795130823"/>
	</Unit>

	<Unit name="rad/s">
		<BaseUnit s="-1" rad="1"/>
	</Unit>

	<Unit name="kg.m2">
		<BaseUnit kg="1" m="2"/>
	</Unit>
</UnitDefinitions>

<TypeDefinitions>
	<SimpleType name="Modelica.SIunits.Inertia">
		<Real quantity="MomentOfInertia" unit="kg.m2" min="0.0"/>
	</SimpleType>
	
	<SimpleType name="Modelica.SIunits.Torque">
		<Real quantity="Torque" unit="N.m"/>
	</SimpleType>
	
	<SimpleType name="Modelica.SIunits.AngularVelocity">
		<Real quantity="AngularVelocity" unit="rad/s"/>
	</SimpleType>
	
	<SimpleType name="Modelica.SIunits.Angle">
		<Real quantity="Angle" unit="rad"/>
	</SimpleType>
</TypeDefinitions>

<DefaultExperiment startTime="0.0" stopTime="3.0" tolerance="0.0001"/>

<ModelVariables>
	<ScalarVariable name="inertia1.J" valueReference="1073741824" description="Moment of load inertia"       causality="parameter" variability="fixed">
		<Real declaredType="Modelica.SIunits.Inertia" start="1"/>
	</ScalarVariable>

	<ScalarVariable name="torque.tau" valueReference="536870912" description="Accelerating torque acting at flange (= -flange.tau)" causality="input">
		<Real declaredType="Modelica.SIunits.Torque" start="0"/>
	</ScalarVariable>

	<ScalarVariable name="inertia1.phi" valueReference="805306368" description="Absolute rotation angle of component"       causality="output">
		<Real declaredType="Modelica.SIunits.Angle" />
	</ScalarVariable>

	<ScalarVariable name="inertia1.w" valueReference="805306369" description="Absolute angular velocity of component (= der(phi))" causality="output">
		<Real declaredType="Modelica.SIunits.AngularVelocity" />
	</ScalarVariable>
</ModelVariables>

<ModelStructure>
	<Outputs>
		<Unknown index="3"/>
		<Unknown index="4"/>
	</Outputs>
	<InitialUnknowns>
		<Unknown index="3"/>
		<Unknown index="4"/>
	</InitialUnknowns>
</ModelStructure>
</fmiModelDescription>
```


# 四、结语

模型交换指的是将单独一个仿真环境的某一个仿真实例生成符合 FMI 接口定义的一个可移植可调用的数学模型库；而联合仿真指的是将不同仿真环境（分布式）的输入输出按照 FMI 标准定义好，通过Master Algorithm（ssp、dcp）传输数据。

> 《Functional Mock-up Interface forModel Exchange and Co-Simulation v2.0》

