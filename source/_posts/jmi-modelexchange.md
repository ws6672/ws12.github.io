---
title: FMI2.0 ———— 	模型交换
date: 2021-05-25 17:43:04
tags: [co-simulation]
---


# 一、模型交换的数学描述


### 基础

下图包含了在C程序访问dll文件的接口示意图

![接口示意图](/image/co-simulation/me-view.png)

模型交换接口的目标是用数值方法求解微分、代数和离散时间方程组。在此版本的接口中，需要处理在状态空间中带有事件的常微分方程（缩写为“hybrid ODE”）。代数方程系统可能包含在FMU中；FMU也​​可能仅包含离散时间方程，例如描述采样数据控制器。

> ODE意为常微分方程，hybrid ODE 意为 混合常微分方程


自变量时间t属于元祖 t=(t<sub>R</sub>...t<sub>i</sub>)。其中tR为实数，t<sub>i</sub>为{0,1,2,...}；
+	实数部分t<sub>R</sub>，用于描述事件模型的连续时间行为；
+	而整数部分t<sub>i</sub>是一个计数器，用于枚举在同一连续时间中发生的事件，该时间定义在文献中也称为“超密集时间”，参见（Lee and Zheng 2007）。

相关符号的含义如下
![记号](/image/co-simulation/me-t-symbol.png)

### 事件

FMI支持的混合ODE被描述为分段连续时间系统，不连续性可能发生在T<sub>0</sub>....T<sub>n</sub>时刻，其中T<sub>i</sub> < T<sub>i+1</sub>。这些特殊时刻称为“事件”，事件可以提前被声明（time事件），也可以隐式定义（state、step事件）。在两个事件间，变量的值是连续的或不变的：如果变量仅在事件瞬间更改其值，则该变量称为离散时间；否则称为连续时间，只有实数变量可以是连续时间。

相关例子如下：

![记号](/image/co-simulation/me-event.png)

事件实例定义如下：

1. 外部事件
+	离散时间的输入值发生变化
+	连续时间的输入存在一个非连续的修改
+	可调参数（tunable）发生了变化
这几个被称之为外部事件。（如果A输出到B，那么如果A触发了一个外部事件，那么B也应当触发一个外部事件）

2. 时间事件
在预先定义的时刻 T<sub>i</sub> =（T<sub>next</sub>(t<sub>i-1</sub>), 0)，该时刻由FMU的前一个事件时刻t<sub>i-1</sub>定义，这样的事件称之为时间事件。

3. 状态事件
事件指示器 Z<sub>j</sub>(t)的取值发生类似从Z<sub>j</sub> > 0修改为Z<sub>j</sub> <= 0的变化，这种事件称之为状态事件。所有事件指示符都是分段连续的，并收集在一个实数向量z（t）的向量中。

4. 步进事件（step）
在Integrator的每个完成步骤中，都必须调用fmi2CompletedIntegratorStep；如果返回参数nextMode = EventMode，则此时会发生一个事件（步进事件）。 

***

### 模式

计算FMI模型的解意味着将解过程划分为不同的阶段，并且在每个阶段都使用不同的方程式和解方法，可根据以下模式对阶段进行分类：

1. 初始化模式

该模式用于在起始时间时，通过使用其他方程式中不存在的额外方程式（例如用于定义状态或状态导数起始值的方程式）来计算连续时间状态以及先前（内部）离散时间状态的初始值。

2. 连续时间模式

通过数值求解常微分方程和代数方程，此模式用于计算事件之间所有（实时）连续时间变量的值。在此阶段中，所有离散时间变量都是固定的，并且不分析相应的离散时间方程

3. 事件模式

给定上一个时刻的变量值，此模式用于计算所有连续时间变量以及在当前事件时刻激活的所有离散时间变量的新值。这是通过求解由所有连续时间方程和所有活动离散时间方程组成的代数方程来执行的。在FMI 2.0中，没有对应机制能够在 瞬时事件 中提供离散时间变量是否变化的信息。因此，在环境中必须假定某个瞬时事件总是计算所有离散时间变量，尽管在FMU内部只有一个新的子集可能会被计算出来。将FMU连接在一起时会出现回路结构，这会带来新的困难。因为可能会存在实数变量、布尔值和整数变量中的线性或非线性代数方程组。为了有效地解决此类方程系统，需要相关性（dependency ）信息（如输出直接取决于输入）。

> 注：可以在元素<ModelStructure>下的xml文件中提供此数据。如果未提供此数据，则必须假定最坏的情况（例如，所有输出变量代数取决于所有输入变量）

在某个瞬时事件中离散系统的代数方程可以描述为 前离散时间状态（内部） 以及 离散时间输入 的函数。如果FMU被循环连接，则会迭代调用这些代数方程，直到找到解决方案为止。如果实际离散时间状态和前离散时间状态不同，则更新离散时间状态；时间的整数部分将增加，并执行新的事件迭代。


FMU在 初始化模式 下使用init方法进行初始化。该函数的输入参数包括输入变量（causality =“input”）和自变量（causality =“independent”；通常为默认值“time”）。所有的变量都具有一个初始化（initial = ”exact“），这是为了在初始时间 t<sub>0<sub>中，计算连续时间状态和输出变量。例如，可通过为状态提供初始值或声明状态导数为零来定义初始化。

初始化本身是一个困难的话题，而且还要求FMU在初始化模式下解决FMU内部明确定义的初始化问题。调用fmi2ExitInitializationMode之后，FMU隐式处于事件模式，所有的离散时间变量和连续时间变量都会在初始化时间实例中被计算处理。如果喜欢，还可以依靠代数循环进行迭代。完成后必须调用fmi2NewDiscreteStates，并且根据返回参数的值，FMU要么在初始时刻继续事件迭代、要么切换到连续时间模式。

切换到连续时间模式后，开始计算积分；在此阶段中，将计算连续状态的导数。如果FMU和子模型连接在一起，则这些模型的输入是其他模型的输出，因此必须计算相应的FMU输出。每当存储结果值时，通常在模拟开始之前定义的输出点上，都必须调用与所需变量有关的fmi2GetXXX函数。

事件瞬间由时间，状态或步进事件或由环境触发的外部事件确定。为了确定状态事件，必须在每个完成的积分器步骤中查询事件指示器z。一旦事件指示符发出信号指示其域的更改，将在前一个积分器步骤与实际完成的积分器步骤之间执行随时间的迭代，以便确定达到一定精度的域更改的瞬间。

触发事件后，需要将FMU切换到事件模式。在这种模式下，可以解决连接的FMU上的方程组（类似于连续时间模式）。一旦达到收敛，就必须调用fmi2NewDiscreteStates（..）来增加超密集时间。根据离散时间模型，可能需要新的事件迭代（例如，因为FMU在内部描述了状态机，并且转换仍然可以触发，但是必须考虑新的输入）。

***

# 二、FMI应用程序编程接口

本节包含接口说明，用于评估C程序中的不同模型零件

### 提供自变量并重新初始化缓存

根据实际情况需要计算不同的变量；而为了提高效率，接口仅需要计算当前上下文中所需的变量。例如
+	在积分的迭代过程中，只要未连接模型的输出，仅需要计算状态导数。
+	如果积分步骤已完成，则还需要计算事件指示器功能。为了提高效率，在调用计算事件指示符函数时，如果状态导数已经在当前时刻进行了计算，则不要重新计算状态导数。这意味着状态导数支持重用，此功能被称为“变量缓存”。


缓存要求模型评估可以检测输入参数（例如：时间或状态）何时已更改。这是通过用函数调用显式设置它们来实现的，因为每个这样的函数调用都会精确地发出相应变量更改的信号。因此，本节包含用于设置方程式求值函数的输入自变量的函数。这对于时间和状态变量来说没有问题，但是对于参数和输入则涉及更多，因为后者可能具有不同的数据类型。

```
-- 设置新的时刻，更新依赖该参数的变量缓存，新时间需要与当前时间不一致
fmi2Statusfmi2SetTime(fmi2Component c, fmi2Realtime);

-- 设置一个新的状态变量，更新依赖该参数的变量缓存
fmi2Statusfmi2SetContinuousStates(fmi2Component c, constfmi2Realx[], size_tnx);

-- 为（独立）参数，起始值和输入设置新值，并重新初始化依赖于这些变量的变量的缓存。
fmi2Status fmi2SetXXX(..)
```

### Evaluation of Model Equations（模型方程式的评估）

1. fmi2Statusfmi2EnterEventMode

```
-- 模型从连续时间模式进入事件模式，离散时间方程可能变为活动状态（并且关系未“冻结”）。
fmi2Statusfmi2EnterEventMode(fmi2Component c);
```
DiscreteStates

```
fmi2Status fmi2NewDiscreteStates(fmi2Component  c , fmi2EventInfo* fmi2eventInfo);

typedef struct{     
	fmi2Boolean newDiscreteStatesNeeded;
	fmi2Boolean terminateSimulation;
	fmi2Boolean nominalsOfContinuousStatesChanged;
	fmi2Boolean valuesOfContinuousStatesChanged;
	fmi2Boolean nextEventTimeDefined;
	fmi2Real    nextEventTime; // 如果设置nextEventTimeDefined=fmi2True，则显示下一个事件 
} fmi2EventInfo;
```

fmi2EventInfo 的参数如下：

+	FMU处于事件模式，并且此调用会增加超密集时间。如果调用fmi2NewDiscreteStates之前的超密集时间为（tR，tI），则调用后的瞬时时间为（tR，tI + 1）。如果返回参数（fmi2eventInfo-> newDiscreteStatesNeeded = fmi2True），则FMU应该保持事件模式，并且FMU要求将新输入设置为FMU（输入为fmi2SetXXX），以计算并获取输出（输出为fmi2GetXXX），并再次调用fmi2NewDiscreteStates 。根据与其他FMU的连接，环境应根据以下条件设置。而当FMU终止（Terminate）时，假定记录器功能会打印一条适当的消息以说明终止的原因。
	+	`如果至少有一个FMU返回 参数 TerminateSimulation = fmi2True，则调用fmi2Terminate函数；
	+	如果所有FMU返回newDiscreteStatesNeeded = fmi2False，则调用fmi2EnterContinuousTimeMode。
	+	否则，保持活动模式
+	如果 nominalsOfContinuousStatesChanged = fmi2True，则状态的值由于函数调用而发生了变化，可以使用 fmi2GetNominalsOfContinuousStates 进行查询
+	如果 valuesOfContinuousStatesChanged = fmi2True，则由于函数调用，连续状态向量中至少一个元素已更改其值。可以使用 fmi2GetContinuousStates 查询状态的新值。如果连续状态向量的任何元素均未更改其值，则 valuesOfContinuousStatesChanged 必须返回fmi2False（如果在这种情况下将返回fmi2True，则可能发生无限事件循环）。
+	如果 nextEventTimeDefined = fmi2True，则模拟将最大次数的进行积分，直到time = nextEventTime，并且此时应调用fmi2EnterEventMode。如果由于状态事件而在nextEventTime之前停止积分，那么nextEventTime的定义将过时。


3. fmi2Statusfmi2EnterContinuousTimeMode
```
fmi2Statusfmi2EnterContinuousTimeMode(fmi2Component c);
```

模型进入连续时间模式，所有离散时间方程变为非活动状态，所有关系都被“冻结”。从事件模式更改为（在所有涉及的FMU和其他模型的事件模式中的全局事件迭代已收敛之后）连续时间模式时，必须调用此函数。


4. fmi2CompletedIntegratorStep

```
fmi2Status fmi2CompletedIntegratorStep(fmi2Component c, fmi2Boolean  noSetFMUStatePriorToCurrentPoint, fmi2Boolean* enterEventMode, fmi2Boolean* terminateSimulation);
```

如果函数标志 completedIntegratorStepNotNeeded = false，则在积分器的每个完成步骤之后，环境都必须调用此函数。如果在此模拟运行中不再针对当前时间之前的时刻调用 fmi2SetFMUState，则参数 noSetFMUStatePriorToCurrentPoint为fmi2True [FMU可以使用此标志刷新结果缓冲区]

如果FMU调用fmi2EnterEventMode，则该函数返回enterEventMode以向环境发出信号，并且如果仿真应终止，则该函数返回TerminateSimulation发出信号。如果enterEventMode = fmi2False并且TerminateSimulation = fmi2False，则FMU保持在连续时间模式，而无需再次调用fmi2EnterContinuousTimeMode

当积分器步骤完成并且状态由积分器修改后（例如通过BDF方法进行校正）时，则在调用fmi2CompletedIntegratorStep（..）之前，必须调用fmi2SetContinuousStates（..）更新状态。

当积分器步骤完成并且一个或多个事件指示器更改符号（相对于先前完成的积分器步骤）时，则积分器或环境必须确定最接近先前完成的步骤的符号更改的时刻。达到一定的精度（通常是机器epsilon的很小的倍数）。这通常是通过迭代来执行的，其中时间是变化的，并且迭代期间所需的状态变量是通过插值确定的。必须在此状态事件定位过程之后而不是在积分算法成功计算时间步长之后调用函数fmi2CompletedIntegratorStep。该函数调用的预期目的是向FMU指示，在此阶段，所有输入和状态变量都具有有效（接受）的值。

调用fmi2CompletedIntegratorStep之后，仍然允许其返回时间（调用fmi2SetTime）并使用fmi2GetXXX在先前的时刻查询变量的值（例如，确定输出点处的非状态变量的值）：但是，它不是允许在上一个完成的IntegratorStep或上一个fmi2EnterEventMode调用之前返回时间。

在以下几种情况中，需要调用这个函数：
+	延迟：delay(...)操作中使用的所有变量都存储在适当的缓冲区中，并且函数返回 nextMode = fmi2ContinuousTimeMode
+	动态状态选择：检查动态选择的状态在数值上是否仍然合适。如果合适，则函数返回enterEventMode = fmi2False，否则返回enterEventMode=fmi2True。在第二种情况下，必须调用fmi2EnterEventMode（..），然后通过fmi2NewDiscreteStates（..）函数动态更改状态。

> 注意：此函数不用于检测时间或状态事件。例如，通过将前一个事件的指示符与fmi2CompletedIntegratorStep（..）的当前调用进行比较，来检测时间或状态事件。这些类型的事件是在环境中检测到的，在这种情况下，环境必须调用fmi2EnterEventMode（..），而与fmi2CompletedIntegratorStep（..）的返回参数enterEventMode是fmi2True还是fmi2False无关。

5. 导数与事件指示符
```
fmi2Status fmi2GetDerivatives    (fmi2Component c, fmi2Real derivatives[], size_t nx); 
fmi2Status fmi2GetEventIndicators(fmi2Component c, fmi2Real eventIndicators[], size_tni)
```

在当前时刻和当前状态下，计算状态导数和事件指示符：

+	导数作为带有“ nx”个元素的向量返回，事件指示符作为带有“ ni”个元素的向量返回。
+	当事件指示器的域从z<sub>j</sub> > 0变为z<sub>j</sub> ≤0或相反操作时，将触发状态事件。 FMU必须保证在事件发生时重启z<sub>j</sub>≠0，例如通过以较小值改变z<sub>j</sub>来保证。此外，z<sub>j</sub>应在FMU中使用其标称值进行缩放（因此，返回的矢量“ eventIndicators”的所有元素应按“ one”的顺序排列）。

导数向量的元素的顺序与状态向量的顺序相同（例如，导数[2]是x [2]的导数），而事件指示器不一定与模型描述文件中的变量相关。

> 注：fmi2Status = fmi2Discard对于上述两个函数来说都是可能的取值。

6. fmi2GetContinuousStates
```
fmi2Status fmi2GetContinuousStates(fmi2Component c, fmi2Realx[], size_tnx);
```

返回新的（连续）状态向量x。如果函数以eventInfo-> valuesOfContinuousStatesChanged = fmi2True返回（标识（连续时间）状态向量已更改），则必须在调用函数 fmi2EnterContinuousTimeMode 之后直接调用此函数。

7. fmi2GetNominalsOfContinuousStates

```
fmi2Status fmi2GetNominalsOfContinuousStates(fmi2Component c, fmi2Realx_nominal[],size_tnx);
```

返回连续状态的标称值。如果此函数返回eventInfo-> nominalsOfContinuousStatesChanged = fmi2True，则在调用函数 fmi2NewDiscreteStates 之后应始终调用此函数，因为连续状态的标称值已更改[例如，因为连续状态与变量的关联由于内部动态状态选择而发生了变化。如果FMU没有与连续状态i有关的标称值信息，则应返回标称值x_nominal [i] = 1.0。注意，要求x_nominal [i]> 0.0。 【通常，连续状态的标称值用于计算积分器所需的绝对公差：`absoluteTolerance[i] = 0.01*tolerance*x_nominal[i];`】

### 基于函数调用顺序的状态机

FMI的每个实现都必须根据以下状态图支持函数的调用序列（以UML 2.0状态机的形式进行的模型交换C函数的调用序列）：

![状态机](/image/co-simulation/me-state-chart.png)

定义状态图的最初目的是为了定义 FMI函数允许的调用顺序：
FMI不支持状态图不接受的调用顺序。对于这样的调用序列，FMU的行为是不确定的。例如，状态图指示当用于模型交换的FMU处于状态“连续时间模式”时，不支持对离散输入的fmi2SetReal调用。状态图在此处以UML 2.0状态机的形式给出。如果一个过渡带有一个或多个函数名（例如fmi2GetReal，fmi2GetInteger）标记，则表示成功调用了这些函数中的任何一个，就进行了过渡。注意，由于每个状态都是通过特定的函数调用（例如fmi2EnterEventMode）或特定的返回值（例如fmi2Fatal）输入的，因此FMU始终可以确定它处于哪种状态。

状态机的每个状态对应于仿真的特定阶段，如下所示

1. instantiated(实例化)

在这种状态下，可以设置起始值和估计值（initial ="exact/approx"的变量）

2. Initialization Mode（初始化模式）
在此状态下，方程式可确定所有连续时间状态以及所有输出（以及可选的导出工具公开的其他变量）。可以通过fmi2GetXXX调用检索的变量为：
（1）在xml文件中的<ModelStructure> <InitialUnknowns>下定义的
（2）具有 causality ="output"的变量；可以设置为 initial ="exact" 的变量以及 variability ="input" 的变量。

3. Continuous-Time Mode（连续时间模式）
在这种状态下，连续时间模型方程处于活动状态，并执行了积分步进。如果在完成积分器步进结束时检测到至少一个事件指示器的域发生变化，则可以确定状态事件的事件时间。

4. Event Mode（事件模式）
如果在连续时间模式下触发了事件，则通过调用fmi2EnterEventMode进入事件模式。在这种模式下，所有的连续时间方程和离散时间方程都是有效的，并且可以计算和检索事件中的未知数。事件完全处理后，必须调用fmi2NewDiscreteStates函数并根据返回参数（newDiscreteStatesNeeded）决定保持状态图处于事件模式或切换到连续时间模式。当初始化模式以fmi2ExitInitializationMode终止时，将直接进入事件模式，并根据在初始化模式下确定的初始连续时间状态来计算初始时间的连续时间和离散时间变量。

5. terminated（终止）
在这种状态下，可以获取模拟的最后一次结果。

> 注1：仅在连续的时间间隔内才允许在时间上向后仿真。一旦发生事件（调用了fmi2EnterEventMode），就禁止及时返回，因为fmi2EnterEventMode / fmi2NewDiscreteStates只能计算下一个离散状态，而不能计算前一个离散状态。
注2：在初始化，事件和连续时间模式期间，可以根据xml文件中元素<ModelStructure>下定义的模型结构，使用fmi2SetXXX设置输入变量，并使用fmi2GetXXX互换获取输出变量。

下表汇总了各个状态下允许的函数调用：
（黄色的仅适用于模型交换，而其它函数使用于模型交换以及协同仿真）

![允许的函数调用](image/co-simulation/me-allowed-function.png)

"x"表示：在相应状态下允许调用
数字表示：如果指示的条件成立，则允许调用
（1）variability ≠"constant"initial = "exact/approx"
（2）causality = "output/" 或者 continuous-time 状态 或者状态导数 
（3）variability≠"constant" & (hasinitial="exact" || causality="input")
（4）causality = "input" || (causality = "parameter" & variability = "tunable")
（5）causality = "input" & variability = "continuous"
（7）检索到的值仅可用于调试

***

# 三、FMI描述文件（Schema-ModelExchange）

本节节将定义“模型交换”特定元素“ModelExchange”。


1. modelexchange标签相关定义

![modelexchange](/image/co-simulation/me-modelexchange.png)

2. modelexchange标签相关结构

modelexchange
+	modelIdentifier：根据C语法的简短类名称，例如“ A_B_C”
+	needsExecutionTool：如果为true，则需要一个工具来执行模型，并且FMU仅包含与此工具的通信。
+	completedIntegratorStepNotNeeded：如果为true，则无需调用函数 fmi2CompletedIntegratorStep（这将使集成效率稍微提高一点）。如果调用该函数，则无效。如果为false（默认值），则必须在完成每个积分器步骤后调用该函数，请参见3.2.2节。
+	canBeInstantiatedOnlyOncePerProcess：该标志指示情况（尤其是对于嵌入式代码），其中每个FMU只能有一个实例（多个实例默认为false；如果需要多个实例且此标志为true，则必须在不同的进程中实例化FMU）。
+	canNotUseMemoryManagementFunctions：如果为true，则FMU使用其自身的函数仅用于内存分配和释放。 fmi2Instantiate中的回调函数allocateMemory和freeMemorygiven被忽略
+	canGetAndSetFMUstate：如果为true，则环境可以查询内部FMU状态并可以将其还原。也就是说，FMU支持函数fmi2GetFMUstate、fmi2SetFMUstate和fmi2FreeFMUstate。
+	canSerializeFMUstate：如果为true，则环境可以序列化内部FMU状态，换句话说，FMU支持函数fmi2SerializedFMUstateSize、fmi2SerializeFMUstat 、fmi2DeSerializeFMUstate。如果是这种情况，则标记canGetAndSetFMUstate也必须为true
+	providesDirectionalDerivative：如果为true，则可以使用fmi2GetDirectionalDerivative（..）计算方程的方向导数。
+	sourceFiles
	+	name
	
3. 示例XML描述文件

```
<?xml version="1.0" encoding="UTF8"?>
<fmiModelDescription   fmiVersion="2.0"   modelName="MyLibrary.SpringMassDamper"   guid="{8c4e810f-3df3-4a00-8276-176fa3c9f9e0}"   description="Rotational Spring Mass Damper System"   version="1.0"   generationDateAndTime="2011-09-23T16:57:33Z"   variableNamingConvention="structured"   numberOfEventIndicators="2"> 
<ModelExchange     modelIdentifier="MyLibrary_SpringMassDamper"/>
	<UnitDefinitions> 
		<Unit name="rad">
			<BaseUnit rad="1"/> 
			<DisplayUnit name="deg" factor="57.2957795130823"/></Unit> <Unit name="rad/s">
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
		<ScalarVariable name="inertia1.J" valueReference="1073741824" description="Moment of load inertia" causality="parameter" variability="fixed">
			<Real declaredType="Modelica.SIunits.Inertia" start="1"/>
		</ScalarVariable>
		<!—index="1" -->

		<ScalarVariable name="torque.tau" valueReference="536870912" description="Accelerating torque acting at flange (= -flange.tau)"       causality="input"> 
			<Real declaredType="Modelica.SIunits.Torque" start="0" />
		</ScalarVariable>
		<!—index="2" -->

		<ScalarVariable name="inertia1.phi" valueReference="805306368" description="Absolute rotation angle of component" causality="output">
			<Real declaredType="Modelica.SIunits.Angle" />
		</ScalarVariable>
		<!—index="3" -->

		<ScalarVariable name="inertia1.w" valueReference="805306369" description="Absolute angular velocity of component (= der(phi))"       		causality="output">
			<Real declaredType="Modelica.SIunits.AngularVelocity" />
		</ScalarVariable>
		<!—index="4" -->

		<ScalarVariable name="x[1]" valueReference="0", initial="exact"> 
			<Real/>
		</ScalarVariable>
		<!—index="5" -->

		<ScalarVariable name="x[2]" valueReference="1", initial="exact">
			<Real/>
		</ScalarVariable>
		<!—index="6" -->

		<ScalarVariable name="der(x[1])" valueReference="2">
			<Real derivative="5"/>
		</ScalarVariable>
		<!—index="7" -->

		<ScalarVariable name="der(x[2])" valueReference="3">
			<Real derivative="6"/>
		</ScalarVariable>
		<!—index="8" -->

	</ModelVariables>

	<ModelStructure>
		<Outputs>
			<Unknown index="3" /> 
			<Unknown index="4" />
		</Outputs>

		<Derivatives>  
		   <Unknown index="7" /> 
		   <Unknown index="8" />
		</Derivatives>

		<InitialUnknowns>
			<Unknown index="3" />
			<Unknown index="4" />
			<Unknown index="7" dependencies="5 2" />             
			<Unknown index="8" dependencies="5 6" /> 
		</InitialUnknowns>  

	</ModelStructure> 

</fmiModelDescription>
```

# 四、参考资料
> 《Functional Mock-up Interface forModel Exchange and Co-Simulation v2.0》