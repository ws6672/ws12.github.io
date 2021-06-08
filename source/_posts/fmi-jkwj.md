---
title: FMI2.0 ———— 接口与文件
date: 2021-05-21 14:51:08
tags: [co-simulation]
---

本文是来自于FMI2.0规范的前两章的部分译文，基于机翻与个人理解修订而成；描述了FMI模型交换和联合仿真，还包括了程序接口以及模型描述文件两部分。

# 一、FMI程序接口

1. 模型交换

支持该功能是为了在建模环境中可以生成动态系统模型的C代码，该模型可以被其他建模和仿真环境使用。通过代数、离散方程和具有时间，状态和阶跃事件的离散方程来描述模型。如果C代码描述了一个连续的系统，那么该系统将由使用它的环境的集成商来解决。该接口要处理的模型可以很大，可以在离线或在线仿真中使用，也可以在微处理器的嵌入式控制系统中使用。设计该接口是旨在描述大型模型。

2. 联合仿真

支持该功能的意图是提供一个接口标准，用于在联合仿真环境中与其它仿真工具协同工作。子系统之间的数据交换仅限于离散的通信点（通讯时间）；在两个通信点之外的时间中，子系统由各自的求解器彼此独立地求解。主算法控制子系统之间的数据交换以及所有模拟求解器（从属）的同步。

3. FMI通用概念

这两个接口标准有很多共同点。我们可以利用模型以及联合仿真工具的多个实例单独或协同的将它们连接在一起。这些接口独立于目标环境，因为没有使用依赖于目标环境的头文件（目标平台的数据类型除外）。这允许生成一个可在同一平台上的任何环境中使用的动态链接库。

4. FMU

模型、联合仿真从属/工具的关联部分被存储在一个称为FMU（功能模拟单元）的zip文件中，包含以下内容：
+	XML包含FMU中所有公开变量的定义以及其他静态信息。这样就可以在没有此信息的情况下在目标系统上运行FMU
+	提供了所需的模型方程式或联合仿真工具的访问权限，并提供了一小组易于使用的C函数。与其他方法相比，一种新的缓存技术可以更有效地评估模型方程；同一平台的FMU zip文件中可以包含不同平台的二进制格式
+	可以直接在FMU中提供模型方程式或联合仿真工具，或与外部工具通信的功能模块。
+	包含模型图标（位图文件）、FMU所需的文档文件、地图和表格

FMU包含文件如下：
+	XML 用于描述模型
+	DLL 运行库
+	其它用户所需资源


FMU模型的数据流：
+	数据类型为 Real, Integer, Boolean, String
+	inputs
+	outputs
+	对外暴露的变量

5. 头文件和函数命名

提供了几个头文件，它们定义了FMU的接口。在所有头文件中，均约定所有相关的C函数和类型定义均使用前缀“fmi2”开头

+	“fmi2TypesPlatform.h“：包含函数的输入和输出参数的类型定义，FMU和目标模拟器都必须使用此头文件。
+	“fmi2FunctionTypes.h“：包含FMU所有函数原型的 typedef 定义
+	“fmi2Functions.h“；包含FMU的函数原型，这些原型可以在仿真环境中访问


定义这几个接口的目的是支持FMU的文本表示和二进制表示，并且可执行文件中可能同时存在多个FMU（例如FMU A可以使用FMU B）。所以不同的fmu中函数名应当不一致，避免同时使用时冲突。

5.1 类型平台

“fmi2TypesPlatform.h”：包含类型平台的所有定义，用于唯一地标识用于编译二进制文件的头文件，相关源码如下：

```
#ifndef fmi2TypesPlatform_h
#define fmi2TypesPlatform_h
#define fmi2TypesPlatform "default"

   typedef void*           fmi2Component;               /* 该数据结构包含处理模型方程式或处理各个从站的联合仿真所需的信息 */
   typedef void*           fmi2ComponentEnvironment;    /* 可以在仿真环境和记录器功能之间传输来自modelDescription.xml文件 */
   typedef void*           fmi2FMUstate;                /* 保存实际或先前时刻的内部FMU状态*/
   typedef unsigned int    fmi2ValueReference; /* 模型变量值的句柄，句柄和基类型（例如fmi2Real）唯一地标识变量的值*/
   typedef double          fmi2Real   ;
   typedef int             fmi2Integer;
   typedef int             fmi2Boolean;
   typedef char            fmi2Char;
   typedef const fmi2Char* fmi2String;
   typedef char            fmi2Byte;
#define fmi2True  1
#define fmi2False 0
#endif

```

这个接口用于定义FMI中支持的所有数据类型。

5.2 函数返回值

文件中定义了“状态”标识（在文件“ fmi2FunctionTypes.h”中定义的fmi2Status类型的枚举），所有函数均返回该标志以指示函数调用成功：
```
	typedef	enum{
		fmi2OK,
		fmi2Warning, 
		fmi2Discard, 
		fmi2Error, 
		fmi2Fatal, 
		fmi2Pending
	} fmi2Status;
```

这个接口用于定义FMI中函数的返回值类型。

5.3 fmu函数库

部分源码如下：
```
。。。
	#define fmi2GetTypesPlatform         fmi2FullName(fmi2GetTypesPlatform)
	#define fmi2GetVersion               fmi2FullName(fmi2GetVersion)
	#define fmi2SetDebugLogging          fmi2FullName(fmi2SetDebugLogging)
	#define fmi2Instantiate              fmi2FullName(fmi2Instantiate)
	#define fmi2FreeInstance             fmi2FullName(fmi2FreeInstance)
	#define fmi2SetupExperiment          fmi2FullName(fmi2SetupExperiment)
	#define fmi2EnterInitializationMode  fmi2FullName(fmi2EnterInitializationMode)
	#define fmi2ExitInitializationMode   fmi2FullName(fmi2ExitInitializationMode)
。。。
```

这个接口用于定义FMI中使用的函数。

6. 初始化，终止和重置FMU

fmi2SetupExperiment函数会在fmi2Instantiate被调用之后和fmi2EnterInitializationMode被调用之前调用，参数公差定义和公差取决于FMU类型
```
fmi2Status fmi2SetupExperiment (fmi2Component c,
								fmi2Boolean   toleranceDefined,
								fmi2Real      tolerance, 
								fmi2Real      startTime,
								fmi2Boolean   stopTimeDefined, 
								fmi2Real    stopTime);
						
```

`fmuType = fmi2ModelExchange`: 如果“ toleranceDefined = fmi2True”，则使用数值积分方案调用模型，其中通过使用“ tolerance”控制误差来控制步长（通常使用相对公差）
`fmuType = fmi2CoSimulation`：如果“toleranceDefined = fmi2True”，通信间隔由估计误差控制；如果从站具有可变步长和估计误差的积分，那么通过公差来控制通信间隔

参数 startTime 和 stopTime 可用于检查模型在给定范围内是否有效，或分配存储结果所需的内存。如果当前时间超过 stopTime，则FMU必须返回fmi2Status = fmi2Error。如果stopTimeDefined = fmi2False，则没有定义自变量的最终值，并且参数stopTime没有意义。


相关函数定义：
```
-- 使FMU进入初始化模式
fmi2Status fmi2EnterInitializationMode(fmi2Component c);
-- 使FMU退出初始化模式
fmi2Status fmi2ExitInitializationMode(fmi2Component c);
-- 终止FMU
fmi2Status fmi2Terminate(fmi2Component c);
-- 重置FMU
fmi2Status fmi2Reset(fmi2Component c);
```

7. 偏导数的支持

fmi标准可以选择评估FMU的偏导数。对于模型交换，这意味着在特定时刻计算偏导数；对于联合仿真，这意味着要在特定的通信点上计算偏导数。fmi标准提供了一种函数来计算方向导数，此函数可用于构造所需的偏导数矩阵。

```
fmi2Status fmi2GetDirectionalDerivative(fmi2Component c, 
		const fmi2ValueReference vUnknown_ref[], 
		size_t nUnknown, 
		const fmi2ValueReference vKnown_ref[]  , 
		size_t nKnown,
		const fmi2Real dvKnown[], 
		fmi2Real dvUnknown[])

初始化模式（Initialization Mode）：在<ModelStructure> <InitialUnknowns>下列出的公开类型为实数的未知数。

连续时间模式（Continuous-Time Mode —— ModelExchange）：连续时间输出和状态导数（在<ModelStructure> <Outputs>下列出的类型为Real和variability ='continuous'的变量，以及在<ModelStructure> <Derivatives>下列出为状态导数的变量）。

事件模式（Event Mode —— ModelExchange）：与连续时间模式中相同的变量，以及<ModelStructure> <Outputs>下类型为Real和variability ='discrete'的变量

步进模式（Step Mode —— CoSimulation）：在<ModelStructure> <Outputs>下列出的类型为Real且variability ='continuous'或'discrete'的变量。如果存在<ModelStructure> <Derivatives>，则此处列出的变量也是状态导数。

```

# 二、FMI模型架构（ModelDescription）

与FMU相关的所有静态信息都以XML格式存储在文本文件 modelDescription.xml 中(FMU变量及其属性如名称，单位，默认初始值等都存储在此文件中)；使用模式文件“ fmiModelDescription.xsd”定义此XML文件的结构。该架构文件使用了以下辅助架构文件：

+	fmi2Unit.xsd
+	fmi2Type.xsd
+	fmi2Annotation.xsd
+	fmi2ScalarVariable.xsd
+	fmi2AttributeGroups.xsd
+	fmi2VariableDependency.xsd

该架构所需的数据类型（例如：xs：normalizedString）在[XML架构标准](http：//www.w3.org/TR/XMLschema-2/)中定义，类型的定义如下：
```
	xs:double 表示 double
	xs:int 表示  int
	xs:unsignedInt 表示   unsigned int
	xs:boolean 表示  char
	xs:string 表示  char*
	xs:normalizedString 表示  char*
	xs:dateTime tool 表示  specific

```

XML文件的第一行（如modelDescription.xml）必须包含XML文件的编码方案,要求编码方案始终为UTF8，需要如下定义：
```
	<?xml version="1.0" encoding="UTF-8"?>
```



以下是根级别的架构文件，包含架构文件中的所有元素，而数据由这些元素的属性定义：

![元素架构](/image/co-simulation/fmi-md.png)

***

1. fmiModelDescription（fmi模型描述）

定义如下：

![元素架构-属性](/image/co-simulation/fmi-md-attr.png)

标签结构如下：
+	fmiModelDescription
	+	属性
		+	fmiVersion（必需）：FMI规范版本
		+	modelName（必需）：生成XML的模型名称
		+	guid（必需）：全球唯一标识
		+	description：模型简短描述
		+	author：模型作者的名称和组织
		+	version：模型版本
		+	copyright：知识版权定义（例如“© My Company 2011”）
		+	generationTool：生成XMLfile的工具
		+	generationDateAndTime：生成XML文件的日期和时间（格式为“YYYY-MM-DDThh:mm:ssZ”，Z表示Zulu 时区）
		+	variableNamingConvention：该约束定义了如何构造 ScalarVariable的名称（flat表示唯一的非空字符串；structured表示使用”.“作为分隔符，名称由“_”、字母和数字组成，也可以由单撇号括起来的任何字符组成）
		+	numberOfEventIndicators：基于FMI 模型交换的事件指标的（固定）数量，联合仿真下可以忽略
	+	子标签
		+	ModelExchange：如果存在，则FMU基于“用于模型交换的FMI”（换句话说，FMU包含模型或与提供模型的工具进行通信，而环境提供模拟引擎）。
		+	CoSimulation： 如果存在，则FMU基于“用于联合仿真的FMI”（换句话说，FMU包括模型和仿真引擎，或者与提供模型和仿真引擎的工具进行通信，该环境提供了主算法来运行耦合的FMU联合仿真从设备以共同运行)。
		+	UnitDefinitions：定义单位和显示单位的全局列表（例如，将显示单位转换为模型方程式中使用的单位）。这些定义在XML元素“ ModelVariables”中使用
		+	TypeDefinitions：“ ModelVariables”中使用的类型定义列表
		+	LogCategories：日志类别的全局列表，可以设置该列表以定义FMU支持的日志信息
		+	DefaultExperiment：提供积分的默认设置，例如停止时间和相对公差
		+	VendorAnnotations：供应商可能想要存储而其他供应商可能会忽略的其他数据
		+	ModelVariables：定义了FMU的所有变量
		+	ModelStructure：定义模型的结构，包括输出，连续时间状态和初始未知数


***

2. UnitDefinitions （变量单位）

变量的单位是（可选）自定义的，单位的支持对于技术系统很重要，因为很容易发生错误。

相关定义如下

![变量单位](/image/co-simulation/fmi-unit.png)
![变量单位](/image/co-simulation/fmi-unit2.png)

标签结构如下

+	UnitDefinitions
	+	Unit（多个）
		+	属性
			+	name：
		+	标签
			+	BaseUnit（基本单位 前七个是国际单位制中的基本单位）
				+	kg：质量-千克
				+	m：距离-米
				+	s：时间-秒
				+	A：电流-安培
				+	K：温度-开尔文
				+	mol：物质的量-摩尔
				+	cd：发光强度-坎德拉
				+	rad：辐射吸收剂量-拉德
				+	factor：因子（乘法）
				+	offset：偏移量（增益）
			+	DisplayUnit
				+	name
				+	factor
				+	offset
其中，DisplayUnit用于定义转换值，转换公式如下：
```
DisplayUnit_value = factor*Unit_value + offset
```

示例如下：

```<Unit name="rad/s">
	<BaseUnit s="-1" rad="1"/>
	<DisplayUnit name="deg/s" factor= "57.29577951308232"/>
	<DisplayUnit name="rev/min" factor= "9.549296585513721"/>
</Unit>
<Unit name="bar">
	<BaseUnit kg="1", m="-1", s="-2", factor="1.0e5", offset="0"/>
</Unit>
<Unit name="Re">
	<BaseUnit/> // unit = "1"
//(dimensionless, all exponents of BaseUnit are zero)
</Unit>
<Unit name="Euro/PersonYear"/> // no mapping to BaseUnit defined
```

***

3. TypeDefinitions（类型定义）

该元素是一个名为“SimpleType”的集合，在 fmi2Type.xsd文件中的 fmi2SimpleType 架构下定义的。

相关定义如下

![类型定义](/image/co-simulation/fmi-type.png)

标签结构如下：

+	TypeDefinitions
	+	SimpleType（多个）
		+	属性
			+	name
			+	description
		+	子标签
			+	Real
				+	declaredType：定义的类型
				+	quantity：变量的物理量
				+	unit：单位
				+	displayUnit：显示单位
				+	relativeQuantity：为true，那么忽略“ displayUnit”的“ offset”
				+	min：时间最小值
				+	max：时间最大值
				+	nominal：变量的标称值。如果未定义并且没有有关标称值的其他信息，则假定标称= 1
				+	unbounded：如果为true，则表示该变量在时间积分中远大于其标称值nominal。
				+	start：如果 initiaJ= exact/approx，那么需要满足min<start<max.
				+	derivative：导数
				+	reinit：为true，运行重新初始化
			+	Integer
				+	declaredType
				+	quantity
				+	min
				+	max
				+	start
			+	Boolean
				+	declaredType
				+	start
			+	String
				+	declaredType
				+	start		
			+	Enumeration
				+	declaredType
				+	quantity
				+	min
				+	max
				+	start
				+	Items

***


4. LogCategories（日志类别）

LogCategories定义了一组无序字符串，可用于通过函数“ logger”定义日志输出。“名称”属性对于LogCategories列表的所有其他元素必须是唯一的

定义如下
![日志类别的定义](/image/co-simulation/fmi-log.png)

标签结构如下
+	LogCategories
	+	Category(多个、分类)
		+	name(在日志列表中唯一)
		+	description

***


5. DefaultExperiment（默认设置）

DefaultExperiment提供积分器的默认设置，例如开始时间、停止时间、相对公差和第一次仿真步长等（默认、可选）。对于用户来说，比较方便的是这些变量已经为模型提供了有意义的默认值。

定义如下
![默认设置](/image/co-simulation/fmi-de.png)

标签结构如下

+	DefaultExperiment
	+	tolerance：相对公差（默认值为1e-6）
	+	startTime：开始时间
	+	stopTime：结束时间
	+	stepSize：步长大小（决定最小采样周期）

***

6. VendorAnnotations（附加信息）

VendorAnnotations由一组有序的注释组成，这些注释由可以解释“ any”元素的工具名称标识。

定义如下
![附加信息](/image/co-simulation/fmi-annotation.png)

标签结构如下
+	VendorAnnotations
	+	Tool（多个）
		+	name（必须、唯一）
		+	#any：是可以由工具定义的任意XML数据结构（例如javafmi中的v2.Tool类中定义的HybridCoSim）

***


7. ModelVariables（模型变量）

fmiModelDescription的“ ModelVariables”元素是模型描述的核心部分，它提供所有公开变量的静态信息，并定义为：

![模型变量](/image/co-simulation/fmi-variables.png)


“ ModelVariables”元素由“ ScalarVariable”元素的有序集合组成（请参见上图）,标签结构如下

+	ModelVariables
	+	ScalarVariable（多个）：标量变量,即存储一个数据的变量
		+	属性
			+	name（必需）：变量唯一名称
			+	valueReference：使用被称为值引用的整数句柄识别FMU的标量变量的值。
			+	description
			+	causality（	因果关系）：默认值为local，一个连续时间状态需要设置causality = "local/output"
				+	parameter：独立参数（由环境提供的参数，在仿真期间数据值是固定的，且不能用于连接中） 
				+	calculatedParameter：计算参数（在仿真期间数据值是固定的，在初始化阶段以及可调（tunable）参数发生变化时被计算）
				+	input：该变量是其它模块或从机提供的，不允许定义initial
				+	output：该变量是提供给其它模块或从机的，与输入的代数关系是通过dependencies属性定义的（`<fmiModelDescription><ModelStructure><Outputs><Unknown>`）
				+	local（默认值）：由其他变量计算得到的局部变量或连续时间状态，不允许用于其它模块或从站。
				+	independent：自变量（通常是时间）。所有变量与这个自变量构成函数。可变性必须是“continuous”。在FMU中，最多只有一个 `ScalarVariable `可以定义为自变量。如果没有变量定义为自变量，那可以是隐式的定义（name='time',unit='s'）。如果有一个变量被定义为自变量，那它必须定义为'Real'类型，并且不设置'start'属性；也不允许调用`fmi2SetReal`函数。取而代之的是，通过`fmi2SetupExperiment`函数对自变量值进行初始化。在初始化后，模型交换类型的需要调用`fmi2SetTime`函数；联合仿真类型的，需要调用`fmi2DoStep`函数设置参数`currentCommunicationPoint`和`communicationStepSize`(实际值可以通过fmi2GetReal查询)。
			+	variability（可变性）:定义变量的时间依赖性，换句话说它规定了一个变量在什么时候可以改变它的值。
				+	constant：变量值永不改变
				+	fixed：变量值在初始化后不可调整，换句话说在调用`fmi2ExitInitializationMode`函数后变量值不能够再改变
				+	tunable：如果设置属性 `causality = "parameter/input"`以及`variability = "tunable"`，在外部事件（模型交换）间或者通讯点（联合仿真）间变量是固定不变的；如果发生了变化，那么就会触发一个外部事件（ModelExchange）或者在下一个通讯点（CoSimulation）执行改变。属性设置为`variability = "tunable"、causality = "calculatedParameter\output"`的变量必须被重新计算。
				+	discrete（离散）：在模型交换类型中，在外部事件以及内部事件（time、state、step）间是固定不变的；在联合仿真中，通常变量来自“真实”的采样数据系统，其值仅在通信点（或从站内部）更改
				+	continuous（默认值）：只有属性`ype = “Real”`的变量可以是连续的。在模型交换类型中，值的变化没有限制；在联合仿真中，变量通常来自微分。【注意，关于连续状态的信息是用元素 fmiModelDescription.ModelStructure.Derivatives 定义的】
			+	initial
				+	exact：变量用`start`进行初始化
				+	approx：该变量是代数环的迭代变量，初始化时的迭代从起始值开始
				+	calculated：该变量是在初始化期间根据其他变量计算得出的。不允许提供“起始”值
			+	canHandleMultipleSetPerTimeInstant：如果存在`canHandleMultipleSetPerTimeInstant = false`，则该变量在一个超密集时刻（模型评估）只允许调用一次 `fmi2SetXXX`函数；在模型交换中变量属性 `variability = "input"`时使用，不允许使用在代数环中。
			+	previous
		+	子标签
			+	Real
			+	Integer
			+	Boolean
			+	String
			+	Enumeration
			+	Annotations
				+	array-Tool

ScalarVariable 标签的几个属性存在关联，如下所示

![ScalarVariable](/image/co-simulation/ScalarVariable.png)

+	（a）组合没有意义，因为parameters 和 input 是从环境设置的，而常数始终是一个值。
+	（b）组合没有意义，因为causality =parameter/calculatedParameter 定义了不依赖时间的变量，而“离散” ”和“连续”定义变量，在仿真过程中这些值可以更改
+	（c）对于独立变量，只有可变性为连续才有意义。	
+	（d）固定或可调的“输入”具有与固定或可调参数完全相同的属性。为简单起见，仅应定义固定参数和可调参数
+	（e）固定或可调的“输出”具有与固定或可调的calculatedParameter完全相同的属性。为简单起见，仅应定义固定和可调的calculatedParameters



> 代数环（algebraic loop）：代数环（algebraic loop)发生在两个或多个模块在输入端口具有信号直接传递而形成反馈的情况时，直接传递的模块在不知输入端口的值的情况下无法计算出输出端的值，也就是现在时刻的输出是依赖现在时刻的输入值来计算的。当这种情况出现时simulink会在每一次迭代言算完成时，去决定它是否会有解。代数回路会减缓方真执行的速度并可能会没有解。
为简单起见，此版本的模式文件中仅支持标量变量，并且必须将结构化实体（如数组或记录）映射到标量。

***


8. ModelStructure（模型结构）

该结构是针对基础模型方程式的，独立于这些模型方程式的求解方式。

相关定义如下

![模型结构的定义](/image/co-simulation/fmi-ms.png)

标签结构如下

+	ModelStructure
	+	Outputs（多个）：所有输出的有序列表
		+	Unknown
	+	Derivatives（多个）：所有状态导数的有序列表
		+	Unknown
	+	InitialUnknowns（多个）：初始化列表
		+	Unknown
			+	index 索引
			+	dependencies 相关依赖项
			+	dependenciesKind 依赖项类型，如果存在“ dependenciesKind”，则必须存在“ dependencies”，并且必须具有相同数量的列表元素
				+	constant：常数因子  
				+	fixed
				+	tunable
				+	discrete
				+	dependent：没有特定的结构

定义例子如下：
```
<ModelVariables>    
	<ScalarVariable name="p"      , ...> ... </ScalarVariable>  <!—index="1" -->    
	<ScalarVariable name="u1"     , ...> ... </ScalarVariable>  <!—index="2" -->    
	<ScalarVariable name="u2"     , ...> ... </ScalarVariable>  <!—index="3" -->    
	<ScalarVariable name="u3"     , ...> ... </ScalarVariable>  <!—index="4" -->    
	<ScalarVariable name="x1"     , ...> ... </ScalarVariable>  <!—index="5" -->    
	<ScalarVariable name="x2"     , ...> ... </ScalarVariable>  <!—index="6" -->    
	<ScalarVariable name="x3"     , ...> ... </ScalarVariable>  <!—index="7" -->   
	<ScalarVariable name="der(x1)", ...> ... </ScalarVariable>  <!—index="8" -->    
	<ScalarVariable name="der(x2)", ...> ... </ScalarVariable>  <!—index="9" -->  
	<ScalarVariable name="der(x3)", ...> ... </ScalarVariable>  <!—index="10" -->    
	<ScalarVariable name="y"      , ...> ... </ScalarVariable>  <!—index="11" --> 
</ModelVariables> 

<ModelStructure>     
	<Outputs>      
		<Unknown index="11" dependencies="6 7" />    
	</Outputs> 
	<Derivatives>
		<Unknown index="8"  dependencies="6" />
		// 索引2是索引9的常量
		<Unknown index="9"  dependencies="2 4 5 6" dependenciesKind="constant constant dependent fixed"/>
		<Unknown index="10" dependencies="2 3 4 5 6" />
	</Derivatives>
	<InitialUnknowns>
		<Unknown index="6" dependencies="2 4 5" />
		<Unknown index="7" dependencies="2 4 5 11" />
		<Unknown index="8" ... />
		<Unknown index="10" ... />
	</InitialUnknowns>
</ModelStructure>
```

***

# 三、参考资料
> 《Functional Mock-up Interface forModel Exchange and Co-Simulation v2.0》