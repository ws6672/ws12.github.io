---
title: FMI2.0 ———— 联合仿真
date: 2021-05-27 10:58:12
tags: [co-simulation]
---

本文是来自于FMI2.0规范的第四章联合仿真的译文，基于机翻与个人理解修订而成。

# 一、开题

本文定义了函数模拟接口（FMI）用于在联合仿真环境中联合两个或更多仿真模型（FMI进行共模）。联合仿真是一种相当常见的方法，用于仿真仿真的技术系统和相关的物理现象，着重于稳定性（time-dependent）问题。

> 联合模拟(coupling simulation ):应用共享网格技术,解决数字化环境下多学科CAE视图模型耦合仿真的难题。
时间依赖性：time-dependent


联合仿真的设计是为了与多个子系统模型耦合，它们已由其仿真器及其求解器导出为可运行代码；还可用于仿真工具的耦合（仿真器耦合，工具耦合）。

![联合仿真](/image/co-simulation/cos.png)

在工具耦合情况下，仿真工具提供的FMU实现类将FMI函数的调用包装为API调用（`C->其它语言`）。此外，还需要使用仿真工具来对FMU进行联合仿真。最常见的是，基于工具耦合的联合仿真在分布式硬件上实现了由不同计算机处理不同的子系统，其中包括不同的操作系统（集群计算机，计算机农场，不同位置的计算机）。

子系统之间的数据交换和通信通常使用一种网络通信技术（例如MPI，TCP / IP）完成。该通信层的定义不是FMI标准的一部分，但可以使用FMI来实现分布式联合仿真方案，如下图所示
![分布式联合仿真基础结构](/image/co-simulation/cos-distributed.png)

主机必须实现通信层，用于建立网络通信的其他参数（例如，远程计算机的标识，端口号，用户帐户）将通过主机的GUI进行设置，而这些数据都不会通过FMI API传输。


# 二、数学描述

1. 基础

联合仿真利用了仿真过程中所有的阶段都会遇到的耦合问题（模块化结构），从针对不同仿真工具（可以是功能强大的仿真器以及简单的C程序）中各个子系统单独的模型设置和预处理开始。

在进行时间积分时，将限制子系统间的数据交换只能在离散的通信点tc<sub>i</sub>中进行， 对所有子系统独立执行仿真。对于模拟耦合，子系统中的仿真工具独自完成仿真数据的可视化和后处理。在不同的上下文中，通信点tc<sub>i</sub>、通信步长tc<sub>i</sub>->tc<sub>i+1</sub> 以及通信步长总个数hc<sub>i</sub>:=tc<sub>i+1</sub>- tc<sub>i</sub> 分别称为取样点（同步点）、宏步长和取样率。FMI中用于联合仿真的术语“通信点”是指联合仿真环境中子系统之间的通信，不应与用于将仿真结果保存到文件中的输出点混合使用，

> 取样：指的是取样的时间点
宏步长：指的是两次取样的时间间隔
取样率（Sampling Rate）是指在数码音频和视频技术应用中，当进行模拟/数码转换时，每秒钟对模拟信号进行取样时的快慢次数。例如，CD和MD的取样率为44.1kHz，表示每秒钟对模拟音频信号进行了44100次取样。取样率越高，转换的精度就越高，重放出的模拟信号波形就越接近原始的模拟信号波形。取样率的高低。决定了所能转换模拟信号的频率上限。


FMI为联合仿真提供了一个接口标准，提供了与时间依赖性相关的耦合系统的解决方案，该耦合系统由时间连续（由平稳差分方程描述的模型组件）或时间离散（由差分方程描述的模型组件，例如离散控制器）的子系统组成。在耦合系统的块表示中，子系统由具有（内部）状态变量<i>x(t)</i>的块表示，这些块通过子系统输入 <i>u(t)</i> 和子系统输出 <i>y(t)</i> 连接到其他耦合的子系统（模块）的模块。在这个框架中，子系统间的物理连接由所有子系统的输入 <i>u(t)</i> 和输出 <i>y(t)</i> 间的数学耦合条件表示。

![通信点的数据流](/image/co-simulation/cos-data-flow.png)

> 差分方程：包含未知函数的差分及自变数的方程。在求微分方程的数值解时，常把其中的微分用相应的差分来近似，所导出的方程就是差分方程。通过解差分方程来求微分方程的近似解，是连续问题离散化的一个例子。 

为了进行联合仿真，必须实现两个基本函数组：
+	子系统之间的数据交换函数
+	用于算法问题的函数，以同步所有子系统的仿真，并从初始时间<i>tc<sub>0</sub></i>:= <i>t<sub>start</sub></i>到结束时间<i>tc<sub>N</sub></i>:=<i>t<sub>stop</sub></i>执行通信步长<i>tc<sub>i</sub></i>-><i>tc<sub>i+1</sub></i>。

在用于联合仿真的FMI中，这两个函数都在一个软件组件中实现，即联合仿真主机（Master）。子系统（Slave）之间的数据交换仅通过主站进行，从站之间没有直接通信；主函数可以通过特殊的软件工具（单独的仿真背板）或所涉及的仿真工具之一来实现。最常见的是，耦合系统可以在嵌套的协同仿真环境中进行仿真，并且用于协同仿真的 FMI 适用于层次结构的每个级别。

用于联合仿真的FMI定义了在联合仿真环境中主机与所有从机（子系统）之间进行通信的接口规则。最常见的主站算法是在每个通信点tc<sub>i</sub>停止所有从站的仿真（时间积分）、收集所有子系统的输出y（tc<sub>i</sub>）、调整子系统输入u（tc<sub>i</sub>）将这些子系统输入分配给从站，然后继续与下一个交互步骤`tc<sub>i</sub>->tc<sub>i+1</sub> = tc<sub>i</sub> + hc`的（协同）仿真，交互步长为固定参数hc。在每个从站中，适合的求解器应当被用于给定交互步长 tc<sub>i</sub>->tc<sub>i+1</sub> 的子系统之一求积分。最简单的联合仿真算法通过冻结数据u(tc<sub>i</sub>)来近似 tc<sub>i</sub>≤t <tc<sub>i</sub> + 1的（未知）子系统输入u(t),(t> tc<sub>i</sub>)。用于联合仿真的FMI支持这种经典的暴力解法以及更复杂的主算法，旨在支持一类非常通用主算法，但它本身并未定义主算法。

从站支持更复杂的主站算法的能力，在从站的XML描述中包含一组功能标志（capability flags）。典型的例子是
+	具有处理可变通信步长hc<sub>i</sub>的 能力
+	具有使用减小的通信步长重复执行被拒绝的通信步长tc<sub>i</sub>≤t <tc<sub>i</sub> + 1的 能力
+	提供支持插值的输出时间的导数
+	提供雅可比行列式的能力

用于联合仿真的 FMI 仅限于具有以下属性的从站：

+	所有计算参数值 v(t) 是在预先定义的时间间隔（t<sub>start</sub> <= t <= t<sub>stop</sub>）中的时间依赖函数（fmi2SetupExperiment，stopTimeDefined=fmi2True）
+	通常，所有计算（模拟）都随着时间的增加而进行。当前时间<i>t</i>从t<sub>start</sub> 到 t<sub>stop</sub>逐步允许。从站的算法可能拥有属性在[t<sub>start</sub>, t<sub>stop</sub>]整个时间间隔或部分间隔中重复仿真。
+	可以为从站提供一个时间值tc<sub>i</sub>（t<sub>start</sub> <= tc<sub>i</sub> <= t<sub>stop</sub>）
+	当时间到达tc<sub>i</sub>，从站能够中断模拟
+	在中断仿真期间，从站（独立求解器）可以从输入u(tc<sub>i</sub>)获取值，将值发送到输出 y(tc<sub>i</sub>)。
+	每当从站中的模拟中断时，新的时间取值tc<sub>i+1</sub>（tc<sub>i</sub> <= tc<sub>i+1</sub> <= t<sub>stop</sub>）可以给出时间子区间（tc<sub>i</sub> < t <= tc<sub>i+1</sub>）
+	子间隔长度hc<sub>i</sub>是第i个通讯步的长度（通信步长必须大于零）：hc<sub>i</sub>=tc<sub>i+1</sub>-tc<sub>i</sub>

用于联合仿真的FMI相关流程如下
+	该流程从实例化和初始化（准备好所有从设备进行计算，建立通信链接）开始
+	然后进行仿真（强制从设备模拟通信step）
+	最后在完成时关闭

2. 数学模型

本节包含联合仿真 FMU 的正式数学模型，可以作出以下基本假设：
+	从模拟器被主模拟器视为纯粹的采样数据系统，可以是
	+	“真实的”采样数据系统（离散控制器；输入和输出可以是Real，Integer，Boolean，String或枚举类型。此类变量的定义为variability =“ discrete”；最小FMU外部可访问的采样周期由元素DefaultExperiment中的属性stepSize定义）
	+	混合ODE被集成在通信点间（对时间连续系统的采样存取），在其中内部事件可能发生并被处理，但在FMU外部看不到。在此假定此混合ODE的所有输入和所有输出均为实际信号（以variability =“ continuous”定义）
	+	上述系统的组合

+	主站和从站之间的通信仅在一组离散的时刻进行，这些时间点称为通信点。

FMI联合仿真模型相关变量描述：
+	t：自变量time∈ℝ（causality =“independent”定义的变量）;第i个通信点表示为t=tc<sub>i</sub>,通信步长表示为hc<sub>i</sub> = tc<sub>i+1</sub> - tc<sub>i</sub>
+	v：所有暴露变量的向量
+	p：仿真过程中不变的参数
+	不带下标符号引用p 表示独立参数（causality = "parameter"）；
+	因变参数 p<sub>calculated</sub>（causality = "calculatedParameter"）
+	可调参数 p<sub>tune</sub>（causality = "parameter"、variability= "tunable"）
+	u(tc<sub>i</sub>)：输入变量
+	y(tc<sub>i</sub>)：输出变量
+	w(tc<sub>i</sub>)：不能用于FMU连接的FMU的局部变量,通过causality = "local"定义
+	x<sub>c</sub>(t)：用实际连续时间变量的向量表示连续时间状态
+	x<sub>d</sub>(t)：离散时间变量
+	<sup>.</sup>x<sub>d</sub>(t)：上一个离散时间变量

存在如下定义：
+	在通信点上，主站向从站提供通用输入
+	从机向主机提供通用输出
+	Initialization：从站是一个采样数据系统，其内部状态（连续时间或离散时间都没关系）需要初始化为t=tc<sub>0</sub>。这是通过辅助函数执行的[此关系在<ModelStructure> <InitialUnknowns>下的xml文件中定义]。计算FMI联合仿真模型的解意味着将解过程分为两个阶段，并且在每个阶段中使用不同的方程式和解方法。可以根据以下模式对阶段进行分类
	+	Initialization Mode：如果从站与其他模型循环连接，则可以对FMU方程进行迭代。在这种模式下，将求解代数方程
	+	Step Mode：通过对常微分、代数和离散方程进行数值求解，此模式用于计算通信点上所有（实际）连续时间和离散时间变量的值。如果从站与其他模型循环连接，则不可能对FMU方程进行迭代


下表中使用的函数fmi2SetXXX是fmi2SetReal，fmi2SetBoolean，fmi2SetInteger和fmi2SetString的缩写。函数fmi2GetXXX是函数fmi2GetReal，fmi2GetBoolean，fmi2GetInteger和fmi2GetString的缩写

|方程式 |FMI函数|
|:--|:--|
|<b>初始化模式之前的方程式（状态机中的“实例化”）</b>||
|设置 i=0并设置自变量的tc<sub>i</sub>|fmi2SetupExperiment|
|设置变量v<sub>initial=exact</sub>以及v<sub>initial=approx</sub>|fmi2SetXXX|
|<b>初始化模式之后的方程式</b>||
|在 t=t<sub>0</sub>时进入初始化模式（激活初始化，离散时间和连续时间方程式）|fmi2EnterInitializationMode|
|设置变量v<sub>initial=exact</sub>（包括初始值为x<sub>c,initial=exact</sub>独立参数p和连续时间状态）|fmi2SetXXX|
|设置连续时间以及离散时间输入 u<sub>c+d</sub>(tc<sub>0</sub>)以及可以选择设置连续时间导数的输入u<sub>c</sub><sup>j</sup>(tc<sub>0</sub>)|fmi2SetXXX、fmi2SetRealInputDerivative|
| v<sub>InitialUnknows</sub>:=f<sub>init</sub>(u<sub>c</sub>,u<sub>d</sub>,t<sub>0</sub>,v<sub>initial=exact</sub>)|fmi2GetXXX、fmi2GetDirectionalDerivativ|
|退出初始化模式（停用初始化方程式）|fmi2ExitInitializationMode|
|<b>Step模式下的方程式（状态机中的“ stepComplete”，“ stepInProgress”</b>||
|设置独立的可调参数p<sub>tune</sub>（不要设置其他参数p<sub>other</sub>）|fmi2SetXXX|
|设置参数连续时间和离散时间输入 u<sub>c+d</sub>(tc<sub>i</sub>)以及可选参数连续时间输入 u<sub>c</sub><sup>(j)</sup>(tc<sub>i</sub>)的导数|fmi2SetXXX、fmi2SetRealInputDerivative|
|tc<sub>i+1</sub>:=tc<sub>i</sub>+hc<sub>i</sub><br/> (y<sub>c+d</sub>,y<sub>c</sub><sup>(j)</sup>,w<sub>c+d</sub>)<sub>tc<sub>i+1</sub></sub>	:= f<sub>doStep</sub>(u<sub>c+d</sub>,u<sub>c</sub><sup>(j)</sup>,tc<sub>i</sub>,hc<sub>i</sub>,p<sub>tune</sub>,p<sub>other</sub>)<sub>tc<sub>i</sub></sub><br/>tc<sub>i</sub>:=tc<sub>i+1</sub><br/>其中f<sub>doStep</sub>也是内部变量的函数|fmi2DoStep<br/>fmi2GetXXX<br/>fmi2GetRealOutputDerivatives<br/>fmi2GetDirectionalDerivative|



# 三、FMI应用程序编程接口

本节包含用于从C程序访问联合仿真从站的输入/输出数据和状态信息的接口说明。

1. 输入/输出值和参数的传输

输入变量、输出变量以及变量 通过本节中定义的fmi2GetXXX和fmi2SetXXX函数进行传输。

为了使从机能够在通信步长间使用连续实数作为输入的插值，可以提供输入对时间的导数；为了允许更高阶的插值，还可以设置更高阶的导数。从机是否能够进行插值由函数属性canInterpolateInputs提供（假如输入为速度，那么需要提供速度对时间的导数，即速率；那么在两个时间点间，可以使用`速率*时间t`求出时间t时的速度）。以下是相关函数：


`fmi2SetRealInputDerivatives`：
```
-- 设置实数输入变量在第n个时间点的导数
fmi2Status fmi2SetRealInputDerivatives(fmi2Component c, constfmi2ValueReference vr[], size_t nvr, constfmi2Integer order[], constfmi2Real value[])
	“vr”：包含变量值的向量，定义了需要设置导数的变量
	“nvr”：向量维数
	“order[]”：导数的阶（1表示一阶导数，不允许0）。
	“value[]”：导数的向量
```

使用该函数的限制与fmi2SetReal函数相同。输入及其导数是在通信时间步开始时设置的。为了允许实际输出变量作为其他从站输入在通讯步间进行插值/逼近，可以读取输出相对于时间的导数。从站是否能够提供输出导数由函数标志`MaxOutputDerivativeOrder`(无符号整数类型)决定；它定义了输出导数的阶数。如果实际阶数较低（因为积分算法的阶数较低），则检索到的值为0。例如：内部多项式为1阶，并且主机请求输出变量的二阶导数，则从机将返回零。


`fmi2GetRealOutputDerivatives`：
```
-- 检索输出值的第n个导数
fmi2Status fmi2GetRealOutputDerivatives (fmi2Component c, constfmi2ValueReference vr[], size_t nvr, constfmi2Integer order[], fmi2Real value[]);
	“vr”：包含变量值的向量，定义了需要设置导数的变量
	“nvr”：向量维数
	“order[]”：导数的阶（1表示一阶导数，不允许0）。
	“value[]”：导数的向量

```

返回的输出变量对应于当前从机时间，例如在成功执行fmi2DoStep后，返回值与通信时间步的结束有关。

2. 计算

2.1 fmi2DoStep

时间步长的计算由以下函数控制
```
	fmi2Status fmi2DoStep(fmi2Component c, 
	fmi2Real currentCommunicationPoint, 
	fmi2Real communicationStepSize, 
	fmi2Boolean noSetFMUStatePriorToCurrentPoint)
```

该函数用于计算时间步长，参数currentCommunicationPoint表示主机的当前通信点（tc<sub>i</sub>）、参数communicationStepSize表示通信步长（hc<sub>i</sub> > 0.0）。从站必须集成，直到tc<sub>i+1</sub> = tc<sub>i</sub> +hc<sub>i</sub>.【调用环境定义了通信点，fmi2DoStep必须求精确到tc<sub>i</sub> +hc<sub>i</sub> 的积分来与这些点进行同步。而如何实现此目标取决于fmi2DoStep】。

在调用`fmi2ExitInitializationMode`函数之后，第一次调用fmiDoStep时，参数`currentCommunicationPoint`必须等于由`fmi2SetupExperiment`设置的`startTime`参数。【参数`currentCommunicationPoint`不会被正式使用，定义它是为了解决主节点与从节点间FMU状态不匹配的问题：由前一个的`fmi2DoStep`或`fmi2SetFMUStatecall`函数定义的参数`currentCommunicationPoint`、从节点中的FMU状态必须彼此一致】


例如，如果从节点未按照上述要求对自变量使用更新公式（tc<sub>i+1</sub> = tc<sub>i</sub> +hc<sub>i</sub>），而是在内部使用自己的更新公式，例如tc<sub>s,i+1</sub> = tc<sub>s,i</sub> +hc<sub>s,i</sub> （fmi2DoStep函数中，tc<sub>i</sub>=currentCommunicationPoint）；则从节点可以使用时间增量hc<sub>s,i</sub> :=(tc<sub>i</sub>-tc<sub>s,i</sub>)+hc<sub>i</sub>(替代hc<sub>s,i</sub> :=hc<sub>i</sub>)去避免主节点时间tc<sub>i+1</sub>和从节点内部时间tc<sub>s,i+1</sub>偏差过大。

在此次模拟运行中，如果在currentCommunicationPoint到预先设定的时刻之间不再调用`fmi2SetFMUState`函数，则设置参数`noSetFMUStatePriorToCurrentPoint` = “fmi2True” [从节点可以使用该标志刷新结果缓冲区]。

函数返回值：
+	`fmi2OK` —— 直到通信步长（主时间步）结束，每一子步长计算都成功。
+	`fmi2Discard` —— 从节点只成功计算出通讯步长的部分间隔，主节点可以调用相应的 `fmi2GetXXXStatus` 函数来获取更多信息。如果可能的话，主节点应使用更短的通信步长重复模拟。只有函数`fmi2GetFMUState`当前（失败）步骤开始时记录了 FMU 状态，才能重复步长；这是通过调用 `fmi2SetFMUState` 并随后使用新的`communicationStepSize` 调用`fmi2DoStep` 来实现的。
+	`fmi2Error` —— FMU 遇到错误。该 FMU 实例无法继续模拟。如果其中一个函数返回 fmi2Error，则可以尝试通过调用 `fmi2SetFMUstate` 从以前存储的 FMU 状态重新启动仿真。
+	`fmi2Fatal` —— 所有 FMU 实例的模型计算都无法修复（例如，由于在执行 fmi 函数期间出现访问冲突或整数除以零等运行时异常）
+	`fmi2Pending` —— 从节点异步执行，即开始执行时立即返回。如果从节点返回这个状态，主节点必须调用函数`fmi2GetStatus(...,fmi2DoStep,...) `来确定从节点是否完成；另一种方法是等待slave调用回调函数`fmi2StepFinished`；而调用函数`fmi2CancelStep`可以取消当前的计算。如果返回这个值，在`fmi2DoStep`执行期间，不允许调用其它函数。


如果`fmi2DoStep`返回了`fmi2Pending`，则可以调用`fmi2CancelStep`函数停止当前的异步执行。
```
fmi2Status fmi2CancelStep(fmi2Component c);
```

如果用户或从节点之一停止运行联合仿真，则主节点调用次函数。之后，只允许调用`fmi2Reset`或`fmi2FreeInstance`。

3. 获取从节点的状态信息

从节点通过以下函数回复当前状态信息：

```
fmi2Status fmi2GetStatus(fmi2Component c,const fmi2StatusKind s, fmi2Status* value); 
fmi2Status fmi2GetRealStatus   (fmi2Component c, const fmi2StatusKind s,fmi2Real* value); 
fmi2Status fmi2GetIntegerStatus(fmi2Component c, const fmi2StatusKind s, fmi2Integer* value); 
fmi2Status fmi2GetBooleanStatus(fmi2Component c, const fmi2StatusKind s, fmi2Boolean* value); 
fmi2Status fmi2GetStringStatus (fmi2Component c, const fmi2StatusKind s, fmi2String* value);

-- 通知主节点，模拟运行的实际状态（响应正文）
typedefenum{
// （fmi2Status）当 fmi2DoStep 函数返回 fmi2Pending 时可以调用。如果计算未完成，该函数将传递 fmi2Pending；否则，函数返回异步执行的 fmi2DoStep 调用的结果
	fmi2DoStepStatus, 
// （fmi2String）当 fmi2DoStep 函数返回 fmi2Pending 时可以调用。该函数提供一个字符串，通知当前运行的异步 fmi2DoStep 计算的状态
	fmi2PendingStatus,
// （fmi2Real）在调用fmi2DoStep(...)函数返回 fmi2Discard状态后，返回最后一次成功完成的通信步长的结束时间，可用于重置doStep
	fmi2LastSuccessfulTime,
// （fmi2Boolean）如果从站想要终止模拟，则返回 true；可以在 fmi2DoStep(...) 返回 fmi2Discard 后调用；然后使用 fmi2LastSuccessfulTime 确定从站终止的时刻。
	fmi2Terminated 
} fmi2StatusKind;

-- （响应）
typedef enum {
    fmi2OK, // 通信步骤已成功计算到结束
    fmi2Warning, // 警告
    fmi2Discard, // 从节点成功计算出通讯步骤的一个子间隔
    fmi2Error, // 错误
    fmi2Fatal, // 发生错误，导致FMU无法修复损坏
    fmi2Pending // 异步
} fmi2Status;

```

通知主节点有关模拟运行的实际状态，由参数`fmi2StatusKind`决定要返回的状态信息。从节点可以提供哪些状态信息取决于从节点的能力。如果需要从节点无法检索的状态，它将返回状态`fmi2Discard`。

结构体fmi2StatusKind包含以下状态：

|状态| 类型 |描述|
|:--|:--|:--|
|fmi2DoStepStatus|fmi2Status|当fmi2DoStep函数返回fmi2Pending时可以调用。如果计算未完成，该函数将提供fmi2Pending。否则，该函数将返回异步执行的fmi2DoStep调用的结果。|
|fmi2PendingStatus|fmi2String|当fmi2DoStep函数返回fmi2Pending时可以调用。该函数提供一个字符串，该字符串告知当前正在运行的异步fmi2DoStep计算的状态|
|fmi2LastSuccessfulTime|fmi2Real|返回上一个成功完成的通信步骤的结束时间。可以在fmi2DoStep（...）返回fmi2Discard之后调用。|
|fmi2Terminate|fmi2Boolean|如果从节点希望终止仿真，则返回true。可以在fmi2DoStep（...）返回fmi2Discard之后调用。使用fmi2LastSuccessfulTime确定从节点终止的时刻|

3. 从主节点到从节点调用次序的状态机

以下状态机定义了fmi规范所支持的调用序列

![以UML 2.0状态机形式表示的联合仿真C函数的调用序列](/image/co-simulation/cos-status-machine.png)

状态机的每个状态对应于模拟的某个特定阶段，如下所示：

+	instantiated（实例化）：在这种状态下，可以设置起始值和估计值（变量属性initial = “exact” or “approx.”）、设置导数、设置仿真条件

+	Initialization Mode（初始化模式）：在这种状态下，方程式可用于确定所有输出（以及导出工具到处的其他可选变量）、可以通过fmi2GetXXX调用变量是在xml文件中的<ModelStructure> <InitialUnknowns>下定义的（causality="output"）变量、可以设置initial="exact"的变量以及具有variability="input"的变量

+	slaveInitialized（从节点初始化）：在这种状态下，将对从节点进行初始化并执行联合仿真计算；使用函数“ fmi2DoStep”执行计算直到下一个通讯点。根据返回值，从节点处于不同的状态（step complete, step failed, step canceled）。

+	terminated（终止）：在这种状态下，可以获取仿真最后时刻的解。

> 注：在初始化模式下，可以根据xml文件中的元素`<ModelStructure>、 <InitialUnknowns>`定义模型结构，使用fmi2SetXXX设置输入变量，并使用fmi2GetXXX获取输出变量。【例如，如果一个输出y1取决于两个输入u1，u2，则必须先设置这两个输入，然后才能获取y1。 如果另外输出y2取决于输入u3，则可以设置u3，然后再获取y2；可以通过使用适当的数值算法来处理初始化模式下连接的FMU上的人工或“真实”代数环。】

“slaveInitialized”状态还有一个额外的限制，即不允许在fmi2SetXXX函数之后，如果没有fmi2DoStep调用，则不允许调用fmi2GetXXX函数。

与模型交换类型的FMI相反，为了避免缓存存在不同的解释，联合仿真的`fmi2DoStep`函数将执行实际的计算，而不是使用`fmi2GetXXX`函数。因此，模型交换时调用的`fmi2GetXXX`、`fmi2SetXXX`序列无法处理通讯点处的虚拟代数环。

# 四、FMU联合仿真（CoSimulation）

1. 标签的定义

联合仿真功能在模型描述文件中的 <CoSimulation></CoSimulation>标签定义，相关定义如下：

![CoSimulation](/image/co-simulation/cos-CoSimulation.png)

相关结构如下：
+	CoSimulation
	+	modelIdentifier：类名缩写
	+	needsExecution：决定是否需要外部工具执行模型
	+	canHandleVariableCommunicationStepSize：从节点可以处理可变的通信步长。对于每个校准，通信步长（fmi2DoStep函数的参数communicationStepSize）在每次调用中不需要固定不变
	+	canInterpolateInputs：从节点能够对连续输入插值
	+	maxOutputDerivativeOrder：从节点能够提供最大阶数的输出导数
	+	canRunAsynchronuously ：异步
	+	canBeInstantiatedOnlyOncePerProcess：单FMU单实例（如果需要多个实例，则必须在不同的进程中实例化 FMU）
	+	canNotUseMemoryManagementFunctions：如果为true，则从节点使用自身的函数进行内存分配和释放；忽略fmi2Instantiate中给出的回调函数 allocateMemory、freeMemory
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


# 五、结语

模型交换指的是将单独一个仿真环境的某一个仿真实例生成符合 FMI 接口定义的一个可移植可调用的数学模型库；而联合仿真指的是将不同仿真环境（分布式）的输入输出按照 FMI 标准定义好，通过Master Algorithm（ssp、dcp）传输数据。

> 《Functional Mock-up Interface forModel Exchange and Co-Simulation v2.0》

