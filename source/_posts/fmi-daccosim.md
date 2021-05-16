---
title: 基于FMI标准的仿真软件————Daccosim
date: 2021-05-15 22:20:52
tags: fmi
---



# 一、FMI

1.  什么是FMI

FMI代表“ Functional Mock-up Interface ”，是MODELISAR项目中的一项重要开发工作。FMI规范允许任何建模工具生成代表动态系统模型的C代码或二进制文件，然后可以将其无缝集成到另一个建模和仿真环境中。

共同仿真规范的FMI处理带有内置求解器和仿真工具耦合的模型。规范分为执行部分和模型描述部分（XML模式）。总之，实现任何FMI规范的FMU（功能模拟单元）均由以下组成：

+	XML模型描述。
+	以二进制和/或源代码格式实现C函数接口。
+	资源，例如输入数据。
+	模型的图像和文档。

遵循此标准导出的模型都可以与其他导出的模型进行协同仿真。在协同仿真的情况下，FMU包含使用XML标准格式表示的模型以及一些二进制文件，具体取决于与FMU兼容的平台。这些二进制文件是可由主算法（MA）加载的动态库。 ），并具有MA知道的标准接口。在这种情况下，FMU被视为MA命令的从属组件。MA是一种协调多个FMU（从站）执行的软件，这种协调主要涉及不同FMU模型之间的数据交换及其调度。


2. FMI标准接口，需要包含以下文件：

+	模型描述文件 （XML）；
+	仿真程序代码（DLL）；
+	其他文件（图片、文档等）

3. FMI规范介绍-设计思想

+	Model Exchange（模型交换）
+	Co-Simulation（联合仿真）


4. FMI导出文件（FMU）

+	不包含求解器，只包括输入/输出接口以及与模型相关的信息；
+	FMU可以包含大量的变量；
+	FMU可用于嵌入式系统（只需很小的开销）；
+	多个FMU可以便捷高效的连在一起求解（各组件可以相互依赖，互相“协作”）

通过遵循统一的FMI规范，输出统一的FMU文件格式，不同的仿真软件可以联合仿真
+	[Simulink协同仿真](https://www.mathworks.com/help/hdlverifier/simulink-cosimulation.html)
+	[daccosim源码](https://bitbucket.org/simulage/daccosim/src/master/)

# 二、daccosim

DACCOSIM NG是一个用于开发和运行由JavaFMI（fmu-wrapper和fmu-builder）支持的协同仿真用例的环境，JavaFMI是使用FMI标准的“协同仿真”部分实现互操作性的工具套件。DACCOSIM NG允许设计和执行协同仿真，从而提供了开发协同仿真图的机制。

### 协同仿真图（co-simulation graphs）

协同仿真图由顶点和连接顶点的箭头组成。一旦定义了顶点，就可以建立箭头来定义如何在它们之间交换变量。在协同仿真图中，可以包含不同类型的顶点：

1. FMU
这种顶点代表了FMU，它保存文件路径，用作输入和输出的变量以及变量和参数的初始值。


2. External inputs/outputs（外部输入/输出）

这类顶点允许提供固定值作为其他顶点的输入（外部输入）或者存储由输出提供的值（外部输出）。这两种顶点都可以具有多个变量，如外部输入将拥有几个可用于其他顶点的输出。


一旦定义了协同仿真图，就可以执行以下步骤：
+	加载：该过程首先打开定义了协同仿真的文件并将图形加载到内存中，打开该文件并对其进行处理后，还将加载所使用的每个FMU
+	协同初始化：在执行了协同初始化过程后，将用户选择用于导出的变量初始值写入输出文件中，以便随后进行分析
+	协同执行：在达到模拟停止时间之前，将根据需要多次调用doStep方法
+	导出结果：到达停止时间后，通过终止所有FMU并关闭导出文件来完成仿真

协同仿真的困难之一是：为所有组件设置一致的系统范围内的初始值。Daccosim协同初始化算法由基于FMU连接变量构建的全局依赖有向图开始，它使用用户建立的连接来查找源FMU的输出和宿主FMU的输入之间的外部依存关系。


关键思想是有向无环图（DAG）的拓扑排序会给出必须初始化的变量顺序，这导致研究如何将通用有向图转换为DAG。 找到的解决方案是建立与循环依赖关系相对应的强连接组件（SCC）图，将每个SCC收缩到单个顶点中的结果图是DAG。我们使用Tarjan的SCC算法（Tarjan，1972） 在许多Modelica工具中）以标识依赖关系图中的每个SCC（以线性时间运行）。 

遵循在收缩的SCC图上按拓扑排序获得的顺序： 

+	对于未收缩的顶点，只需传播其值
+	对于收缩的顶点（它们对应于循环依赖关系），我们使用迭代算法（称为JNRA（基于雅各布的牛顿-拉夫森算法））解决了初始化问题，该算法受传统牛顿-拉夫森算法的启发，经常被用于电力潮流计算。

3. 运算符（Operator）顶点

共有四个运算符：加法器，乘法器，偏移量和增益。 这些运算符允许使用其他顶点的输出进行计算，从而在要使用的输出中提供结果。 
+	加法器和乘法器有两个或多个输入和一个输出
+	偏移量只有一个固定值和一个输入之和
+	增益定义为一个固定值，该值将乘以给定的输入

所有这些顶点都可以使用实数，整数和布尔值。


### 文件结构

Daccosim文件结构如下：

![Daccosim文件结构](/image/co-simulation/Daccosim-NG-Modelica-f4.jpg)
+	simx：这是一个存档文件（zip），其中包含名为fmu的文件夹，以及在协同仿真图中使用的fmu文件以及sim，dng和dsg文件（sim，dng和dsg文件包含不同格式的协同仿真图的表示）
	+	sim 文件中包含要在编辑器中显示的可视化图形信息
	+	dng 中包含以声明性语言显示的图形
	+	dsg 中包含以json格式定义的序列化图形（协同仿真图的一种表示）
	+	fmu 文件夹：包含相关的多个FMU文件
+	dngx：此存档与simx具有相同的结构和内容，但不存在sim和dsg文件，只有dng文件
	+	dngx允许创建一个可运行文件，使用声明性语言定义了图形。

1. fmu 文件

这是一种ZIP压缩包，包括了模型的资源和文档，通过XML描述了资源的结构和作用。当使用daccosim加载simx时，会解析包含的每个FMU，将它转换为FMU顶点。当软件加载fum文件时，会将之渲染为fmu顶点。

![Daccosim文件结构](/image/co-simulation/Daccosim-NG-Modelica-f4.jpg)

每次在doStep方法中调用FMU顶点时，它们通常需要消耗大量的系统资源。 因此，执行引擎还准备在分布式环境中运行，从而可以执行大规模的协同仿真方案。
归功于抽象机制的使用，执行引擎无需知道每个FMU的实际执行位置。引擎每次与FMU交互时，都会使用抽象接口。如下图所示，一共有三种实现：

![Daccosim文件结构](/image/co-simulation/Daccosim-NG-Modelica-	1.jpg)

Daccosim-NG-Modelica-f5.jpg
+	FMULocal：使用文件系统中的FMU文件
+	FMUStub：使用与Java消息服务（JMS）的连接来与正在远程执行的FMU进行交互，源码中实现了FMUJMS接口，用于实现服务
+	FMUSoul：FMU在远程计算机中的表示

在上图中，示例展示了如何在运行分布式仿真的机器之间通信。在此示例中，有三台计算机。 在第一个实例中，Daccosim内核的一个实例负责协调分布式执行。 此实例的执行引擎使用FMU接口与要协调的三个FMU进行通信。 其中，两个正在远程执行，一个在本地执行。 但是，由于引擎仅取决于接口，因此类似于执行位置等详细信息对其执行过程来说并不重要。
每当引擎发出命令时，FMUStub都会与FMUSoul通信以执行命令，将答案提供给引擎。尽管未显示，但通信是通过JMS进行的。 JMS的使用为设计不同的分发体系结构提供了灵活性，以支持大规模的协同仿真。


2. dng文件（声明式语言）

Daccosim NG中实现的声明性语言允许用户在文本编辑器上定义一个协同仿真图或通过程序自动生成它。为此，已经设计了一种领域特定的语言来简单地定义一个协同仿真图。该语言非常简单并且易于理解。其目的是创建无法在GUI中建模的非常宽的图，其中有数百个相互连接的顶点交换成千上万的变量。此功能允许预处理工具开发兼容的模型，以便在DaccosimNG中执行。

声明式语言实例如下，
```
// 2021-05-08T03:33:24.918Z
// Generated with Daccosim NG %%VERSION%%

// 格式：FMU ID 文件相对路径 staitc？
FMU equation1 "fmu/equation1win3264.fmu"
// 格式：Output/Input FMU-ID 变量ID 变量类型 变量可变性
Output equation1 x2 Real continuous
Input equation1 x1 Real continuous

FMU equation2 "fmu/equation2win3264.fmu"
Output equation2 x1 Real continuous
Input equation2 x2 Real continuous

// 描述要连接的输入、输出
Connection equation1.x2 equation2.x2
Connection equation2.x1 equation1.x1

Export ; . 2.0E-4
Log equation1.x1 equation1.x2 equation2.x1 equation2.x2
NewtonRaphsonInitializer 20 1.0E-5
// 步长方法
ConstantStepper 1.0
// 仿真起始和结束时间
Simulation 0.0 0.0

// pipeline 管道配置：管道配置可以合并到dng文件中。在这种情况下，语法由＆和@之类的链接器组成。 ＆链接器表示＆两侧的FMU在同一阶段并行执行。 @链接器指示两侧的FMU在不同阶段执行
Pipeline equation1&equation2
```

这个例子来自于`equationsPair.simx`，该文件实际是压缩文件，解压后就可打开后缀为".dng"的文件。

3. dsg 文件（DaccosimGraph）

dsg 中包含以json格式定义的序列化图形（协同仿真图的一种表示）

相关示例如下，来自于`equationsPair.simx`

```
	{
		"settings": {
			"coInitialization": {
				"method": "NewtonRaphson",
				"residualsTolerance": 1.0E-5,
				"maxIterations": 20
			},
			"startTime": 0.0,
			"stopTime": 0.0,
			"pipeline": "equation1\u0026equation2",
			"stepper": {
				"method": "ConstantStep",
				"order": 3,
				"stepSize": 1.0,
				"safetyFactor": 0.9,
				"stepperVariables": {},
				"transmitDerivatives": false
			}
		},
		"nodes": [{
			"CLASSNAME": "eu.simulage.daccosim.view.FMU",
			"INSTANCE": {
				"path": "fmu/equation1win3264.fmu",
				"beforeInitValues": [],
				"inInitValues": [],
				"flowVariables": [],
				"static_": false,
				"id": "equation1",
				"inputs": [{
					"id": "x1",
					"type": "Real",
					"value": 0.0,
					"variability": "continuous"
				}],
				"outputs": [{
					"id": "x2",
					"type": "Real",
					"value": 0.0,
					"variability": "continuous"
				}],
				"variables": []
			}
		}, {
			"CLASSNAME": "eu.simulage.daccosim.view.FMU",
			"INSTANCE": {
				"path": "fmu/equation2win3264.fmu",
				"beforeInitValues": [],
				"inInitValues": [],
				"flowVariables": [],
				"static_": false,
				"id": "equation2",
				"inputs": [{
					"id": "x2",
					"type": "Real",
					"value": 0.0,
					"variability": "continuous"
				}],
				"outputs": [{
					"id": "x1",
					"type": "Real",
					"value": 0.0,
					"variability": "continuous"
				}],
				"variables": []
			}
		}],
		"arrows": [{
			"from": "equation1.x2",
			"to": "equation2.x2"
		}, {
			"from": "equation2.x1",
			"to": "equation1.x1"
		}],
		"export": {
			"cellSeparator": ";",
			"decimalSeparator": ".",
			"prefix": "equationsPair",
			"variables": ["equation1.x1", "equation1.x2", "equation2.x1", "equation2.x2"],
			"folder": ".",
			"interpolationInterval": 2.0E-4
		}
	}
```

### daccosim 实例

[下载daccosim 实例](https://bitbucket.org/simulage/daccosim/downloads/) ： daccosim-use-cases-windows-20201212.zip。其中，例子“1-coinit-only/1-equationsPair”就是以下两条算式：

+	2*X1^X1+5*X2=42
+	X1-6*X2=4

每条算式是一个顶点，通过协同合作，两个未知数由零开始递增，最终获得一个近似值。

Equation1.x2取决于Equation2.x1，而Equation2.x2取决于Equation1。 二者形成一个代数环路，其中对 Equation1.x1的修改会影响Equation1.x2，而对Equation2.x2的修改会影响Equation2.x1。 协同仿真程序将计算该图（为所有变量提供一致的初始值），程序将检测到一个SCC，并且在多次迭代后，x1和x2将达到以下值（x1 = 4.56，x2 = 0.09）。