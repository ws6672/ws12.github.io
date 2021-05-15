---
title: Redis Lua脚本入门
date: 2020-10-18 18:24:48
tags: [Redis]
---

Redis支持Lua脚本，可以根据需求拓展Redis数据库。

### 一、入门


Lua 是一种轻量小巧的脚本语言，用标准C语言编写并以源代码形式开放， 其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。


#### 基础


***设计目的***
其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。

***Lua 特性***
+	轻量级: 它用标准C语言编写并以源代码形式开放
+	可扩展: Lua提供了非常易于使用的扩展接口和机制：由宿主语言(通常是C或C++)提供这些功能，Lua可以使用它们，就像是本来就内置的功能一样。
+	支持面向过程(procedure-oriented)编程和函数式编程(functional programming)；
+	自动内存管理；只提供了一种通用类型的表（table），用它可以实现数组，哈希表，集合，对象；
+	语言内置模式匹配；闭包(closure)；函数也可以看做一个值；提供多线程（协同进程，并非操作系统所支持的线程）支持；
+	通过闭包和table可以很方便地支持面向对象编程所需要的一些关键机制，比如数据抽象，虚函数，继承和重载等。

***应用场景***
+	游戏开发
+	独立应用脚本
+	Web 应用脚本
+	扩展和数据库插件如：MySQL Proxy 和 MySQL WorkBench
+	安全系统，如入侵检测系统

#### Linux 安装


1. 通过curl下载

只需要下载源码包并在终端解压编译即可，本文使用了5.3.0版本进行安装：

```
# 虚拟机 NAT网络模式下出现问题:`wget: 无法解析主机地址 “www.lua.org”`
su root
echo 'nameserver 8.8.8.8'>>/etc/resolv.conf
echo 'nameserver 223.6.6.6'>>/etc/resolv.conf

curl -R -O http://www.lua.org/ftp/lua-5.3.0.tar.gz
tar zxf lua-5.3.0.tar.gz
cd lua-5.3.0
make linux test
make install
```

2. 通过window为服务器下载

[下载lua](http://www.lua.org/versions.html#5.4)

```
D:\Program Files\PuTTY>pscp C:\Users\wsz\Downloads\Compressed\lua-5.4.1.tar.gz cen@192.168.209.128:/home/cen

$ tar -zxvf lua-5.4.1.tar.gz
$ cd lua-5.4.1
$ make
$ sudo make install

$ vim hello.lua
$ 	print("Hello World!")
$ lua hello.lua
```

#### 基本语法

1. Lua 交互式编程模式可以通过命令 lua -i 或 lua 来启用：
`lua -i`

2. 单行注释
`--`

3. 多行注释
```
--[[
 多行注释
 多行注释
 --]]
```

4. 标示符
Lua 标示符用于定义一个变量，函数获取其他用户定义的项。标示符以一个字母 A 到 Z 或 a 到 z 或下划线 _ 开头后加上 0 个或多个字母，下划线，数字（0 到 9）。
Lua 不允许使用特殊字符如 @, $, 和 % 来定义标示符。 Lua 是一个区分大小写的编程语言。
最好不要使用下划线加大写字母的标示符，因为Lua的保留字也是这样的。

5. 关键词
以下列出了 Lua 的保留关键词。保留关键字不能作为常量或变量或其他用户自定义标示符：
|and|break|do|else|
|:--|:--|:--|:--|
|else if|end|False|for|
|function|if|in|local|
|nil|not|or|repeat|
|return|then|True|until|
|while|goto|||

6. 全局变量和局部变量
在默认情况下，变量总是认为是全局的。全局变量不需要声明，给一个变量赋值后即创建了这个全局变量，访问一个没有初始化的全局变量也不会出错，只不过得到的结果是：nil。

```
> print(a)
nil
> a=23
> print(a)
23
> a=nil
a = 5               -- 全局变量
local b = 5         -- 局部变量
```

7. 循环控制语句

```
-- while 循环
while( true )
do
   print("循环将永远执行下去")
end

-- for 循环
for var=exp1,exp2,exp3 do  
    <执行体>  
end

-- 数值for 循环，i初始值为10，循环到1，每次减一
for i=10,1,-1 do
    print(i)
end
输出结果：
	10
	9
	8
	7
	6
	5
	4
	3
	2
	1
	
-- for 循环
for i=1,3,5 do  
    print(i)
end

```

8. 流程控制
语法：
```
-- IF ELSE
if(布尔表达式)
then
   --[ 布尔表达式为 true 时执行该语句块 --]
else
   --[ 布尔表达式为 false 时执行该语句块 --]
end

-- if...elseif...else 语句

if( 布尔表达式 1)
then
   --[ 在布尔表达式 1 为 true 时执行该语句块 --]

elseif( 布尔表达式 2)
then
   --[ 在布尔表达式 2 为 true 时执行该语句块 --]

elseif( 布尔表达式 3)
then
   --[ 在布尔表达式 3 为 true 时执行该语句块 --]
else 
   --[ 如果以上布尔表达式都不为 true 则执行该语句块 --]
end
```

实例：
```
-- 1. IF
--[ 0 为 true ]
if(0)
then
    print("0 为 true")
end

-- 2. if...else
--[ 定义变量 --]
a = 100;
--[ 检查条件 --]
if( a < 20 )
then
   --[ if 条件为 true 时执行该语句块 --]
   print("a 小于 20" )
else
   --[ if 条件为 false 时执行该语句块 --]
   print("a 大于 20" )
end
print("a 的值为 :", a)


-- 3. if...elseif...else 语句
a=0

if(a>0)
then
	print(a)
elseif(a==0)
then
	print("zero")
else
	print(-a)
end
```

9. 数组 

```
-- 一维数组
array = {"Lua", "Tutorial"}

for i= 0, 2 do
   print(array[i])
end

-- 多维数组即数组中包含数组或一维数组的索引键对应一个数组。
-- 初始化数组
array = {}
for i=1,3 do
   array[i] = {}
      for j=1,3 do
         array[i][j] = i*j
      end
end

-- 访问数组
for i=1,3 do
   for j=1,3 do
      print(array[i][j])
   end
end

```

10. 迭代器
迭代器（iterator）是一种对象，它能够用来遍历标准模板库容器中的部分或全部元素，每个迭代器对象代表容器中的确定的地址。在 Lua 中迭代器是一种支持指针类型的结构，它可以遍历集合的每一个元素。

```
-- 泛型 for 迭代器
for k, v in pairs(t) do
    print(k, v)
end

-- 泛型 for 迭代器 实例
array = {"Google", "Runoob"}

for key,value in ipairs(array)
do
   print(key, value)
end

-- 多状态的迭代器 实例
array = {"Google", "Runoob"}
-- elementIterator是一个函数
for element in elementIterator(array)
do
   print(element)
end
```

#### 数据类型

Lua 是动态类型语言，变量不要显式进行类型定义,只需要为变量赋值。Lua 中有8 个基本类型分别为：nil、boolean、number、string、userdata、function、thread 和 table。

|数据类型|描述|
|:--|:--|
|nil|这个最简单，只有值nil属于该类，表示一个无效值（在条件表达式中相当于false）。|
|boolean|包含两个值：false和true。|
|number|表示双精度类型的实浮点数|
|string|字符串由一对双引号或单引号来表示|
|function|由 C 或 Lua 编写的函数|
|userdata|表示任意存储在变量中的C数据结构|
|thread|表示执行的独立线路，用于执行协同程序|
|table|Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字、字符串或表类型。在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。|

1. nil（空）
nil 类型表示一种没有任何有效值，它只有一个值

```
> print(type(nil))
nil

-- nil 比较
> print(type(nil)==nil)
false
> print(type(nil)=="nil")
true
```

2. boolean（布尔）

boolean 类型只有两个可选值：
+	false
	+	false
	+	nil
+	true
	+	其它

3. number（数字）
Lua 默认只有一种 number 类型 -- double（双精度）类型：
```
print(type(2))
print(type(2.2))
print(type(0.2))
print(type(2e+1))
print(type(0.2e-1))
print(type(7.8263692594256e-06))
```

4. string（字符串）
字符串由一对双引号或单引号来表示。
```
string1 = "this is string1"
string2 = 'this is string2'

string3 = [[
this is string3
]]
```

4. table（表）
在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。也可以在表里添加一些数据，直接初始化表:
```
local tb = {}
local tb = {"apple", "pear", "orange", "grape"}
```

5. function（函数）
在 Lua 中，函数是被看作是"第一类值（First-Class Value）"，函数可以存在变量里:
```
function fun1(n)
    if n == 0 then
        return 1
    else
        return n * fun1(n - 1)
    end
end
fun = fun1
```

***方法***
```

table.concat (tableName [, sep [, start [, end]]]):
	onat是concatenate(连锁, 连接)的缩写. table.concat()函数列出参数中指定table的数组部分从start位置到end位置的所有元素, 元素间以指定的分隔符(sep)隔开。

table.insert (tableName, [pos,] value):
	tble的数组部分指定位置(pos)插入值为value的一个元素. pos参数可选, 默认为数组部分末尾.

table.maxn (tableName)
	table中所有正数key值中最大的key值. 如果不存在key值为正数的元素, 则返回0。(Lua5.2之后该方法已经不存在了,本文使用了自定义函数实现)

table.remove (tableName [, pos])
	table数组部分位于pos位置的元素. 其后的元素会被前移. pos参数可选, 默认为table长度, 即从最后一个元素删起。

table.sort (tableName [, comp])
	对给定的table进行升序排序。
```

6. thread（线程）
在 Lua 里，最主要的线程是协同程序（coroutine）。它跟线程（thread）差不多，拥有自己独立的栈、局部变量和指令指针，可以跟其他协同程序共享全局变量和其他大部分东西。

7. userdata（自定义类型）
userdata 是一种用户自定义数据，用于表示一种由应用程序或 C/C++ 语言库所创建的类型，可以将任意 C/C++ 的任意数据类型的数据（通常是 struct 和 指针）存储到 Lua 变量中调用。



#### 函数

在Lua中，函数是对语句和表达式进行抽象的主要方法。既可以用来处理一些特殊的工作，也可以用来计算一些值。Lua 提供了许多的内建函数，你可以很方便的在程序中调用它们，如print()函数可以将传入的参数打印在控制台上。

函数格式如下：

```
optional_function_scope function function_name( argument1, argument2, argument3..., argumentn)
    function_body
    return result_params_comma_separated
end

optional_function_scope: 该参数是可选的制定函数是全局函数还是局部函数，未设置该参数默认为全局函数，如果你需要设置函数为局部函数需要使用关键字 local

function_name: 函数名称

argument1, argument2, argument3..., argumentn： 函数参数，多个参数以逗号隔开，函数也可以不带参数。

function_body: 函数体，函数中需要执行的代码语句块

result_params_comma_separated: 函数返回值，Lua语言函数可以返回多个值，每个值以逗号隔开
```

例如：
```
-- 1.质数
function prime(num)
	if(num<=3) then
		return false;
	elseif (num%2==0) then
		return true;
	end
	
	temp=num-1;
	for i=4,temp,1 do
		if (num%i==0)then
			return true;
		end
		temp=temp/i;
	end
	return false;
end

print(prime(43))

-- 2.多返回值
function maximum (a)
    local mi = 1             -- 最大值索引
    local m = a[mi]          -- 最大值
    for i,val in ipairs(a) do
       if val > m then
           mi = i
           m = val
       end
    end
    return m, mi
end

print(maximum({8,10,23,12,5}))

-- 3.可变参数
function add(...)  
local s = 0  
  for i, v in ipairs{...} do   --> {...} 表示一个由所有变长参数构成的数组  
    s = s + v  
  end  
  return s  
end  
print(add(3,4,5,6,7))  --->25

-- 4.可以通过 select("#",...) 来获取可变参数的数量
-- select('#', …) 返回可变参数的长度
-- select(n, …) 用于返回 n 到 select('#',…) 的参数
function average(...)
   result = 0
   local arg={...}
   for i,v in ipairs(arg) do
      result = result + v
   end
   print("总共传入 " .. select("#",...) .. " 个数")
   return result/select("#",...)
end

print("平均值为",average(10,5,3,4,5,6))
```

#### Lua 运算符

1. 算术运算符
|操作符|描述|实例|
|+|加法|A + B 输出结果 30|
|-|减法|A - B 输出结果 -10|
|*|乘法|A * B 输出结果 200|
|/|除法|B / A w输出结果 2|
|%|取余|B % A 输出结果 0|
|^|乘幂|A^2 输出结果 100|
|-|负号|-A 输出结果 -10|

2. 关系运算符
|操作符|描述|实例|
|==|等于，检测两个值是否相等，相等返回 true，否则返回 false|(A == B) 为 false。|
|~=|不等于，检测两个值是否相等，不相等返回 true，否则返回 false|(A ~= B) 为 true。|
|>|大于，如果左边的值大于右边的值，返回 true，否则返回 false|(A > B) 为 false。|
|<|小于，如果左边的值大于右边的值，返回 false，否则返回 true|(A < B) 为 true。|
|>=|大于等于，如果左边的值大于等于右边的值，返回 true，否则返回 false|(A >= B) 返回 false。|
|<=|小于等于， 如果左边的值小于等于右边的值，返回 true，否则返回 false|(A <= B) 返回 true。|


3. 逻辑运算符
|操作符|描述|实例|
|and|逻辑与操作符。 若 A 为 false，则返回 A，否则返回 B。|(A and B) 为 false。|
|or|逻辑或操作符。 若 A 为 true，则返回 A，否则返回 B。|(A or B) 为 true。|
|not|逻辑非操作符。与逻辑运算结果相反，如果条件为 true，逻辑非为 false。|not(A and B) 为 true。|

4. 其他运算符
|操作符|描述|实例|
|..|连接两个字符串|a..b ，其中 a 为 "Hello " ， b 为 "World", 输出结果为 "Hello World"。|
|#|一元运算符，返回字符串或表的长度。|#"Hello" 返回 5|

5. 优先级(从高到低)
```
^
not    - (unary)
*      /       %
+      -
..
<      >      <=     >=     ~=     ==
and
or
```


### 二、进阶

Lua有些更加强大的功能，例如文件操作、数据库连接、 模块与包等。

#### Lua 模块与包
模块类似于一个封装库，从 Lua 5.1 开始，Lua 加入了标准的模块管理机制，可以把一些公用的代码放在一个文件里，以 API 接口的形式在其他地方调用，有利于代码的重用和降低代码耦合度。

Lua 的模块是由变量、函数等已知元素组成的 table，因此创建一个模块很简单，就是创建一个 table，然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。

```
-- 文件名为 module.lua
-- 定义一个名为 module 的模块
module = {}
 
-- 定义一个常量
module.constant = "这是一个常量"
 
-- 定义一个函数
function module.func1()
    io.write("这是一个公有函数！\n")
end
 
local function func2()
    print("这是一个私有函数！")
end
 
function module.func3()
    func2()
end
 
return module
```

Lua提供了一个名为require的函数用来加载模块。要加载一个模块，只需要简单地调用就可以了。例如：

```
require("<模块名>")

require "<模块名>"

```

***加载机制***
对于自定义的模块，模块文件不是放在哪个文件目录都行，函数 require 有它自己的文件路径加载策略，它会尝试从 Lua 文件或 C 程序库中加载模块。

require 用于搜索 Lua 文件的路径是存放在全局变量 package.path 中，当 Lua 启动后，会以环境变量 LUA_PATH 的值来初始这个环境变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。

```
#LUA_PATH
export LUA_PATH="~/lua/?.lua;;"

source ~/.profile
```


#### Lua 元表(Metatable)

在 Lua table 中我们可以访问对应的key来得到value值，但是却无法对两个 table 进行操作。因此 Lua 提供了`元表(Metatable)`，允许我们改变table的行为，每个行为关联了对应的`元方法`。

有两个很重要的函数来处理元表：

+	setmetatable(table,metatable): 对指定 table 设置元表(metatable)，如果元表(metatable)中存在 __metatable 键值，setmetatable 会失败。
+	getmetatable(table): 返回对象的元表(metatable)。

例如：
```
-- 普通表
mytable = {}
-- 元表
mymetatable = {11,22}
-- 将mymetatable设置为mytable元表
setmetatable(mytable,mymetatable)

-- 获取元表
for k,v in ipairs(getmetatable(mytable)) do 
	print(v)
end
```

Lua 中 metatable 是一个普通的 table，但其主要有以下几个功能：

1.定义算术操作符和关系操作符的行为
2.为 Lua 函数库提供支持
3.控制对 table 的访问

***__index 元方法***

当你通过键来访问 table 的时候，如果这个键没有值，那么Lua就会寻找该table的metatable（假定有metatable）中的__index 键。如果__index包含一个表格，Lua会在表格中查找相应的键。

```
mytable = setmetatable({key1 = "value1"}, {
  __index = function(mytable, key)
    if key == "key2" then
      return "metatablevalue"
    else
      return nil
    end
  end
})

print(mytable.key1,mytable.key2)
```

***__newindex 元方法***
`__newindex` 元方法用来对表更新，__index则用来对表访问 。当你给表的一个缺少的索引赋值，解释器就会查找__newindex 元方法：如果存在则调用这个函数而不进行赋值操作。

```
mymetatable = {}
mytable = setmetatable({key1 = "value1"}, { __newindex = mymetatable })

print(mytable.key1)

mytable.newkey = "新值2"
print(mytable.newkey,mymetatable.newkey)

mytable.key1 = "新值1"
print(mytable.key1,mymetatable.key1)

value1
nil	新值2
新值1	nil
```
以上实例中表设置了元方法 `__newindex`，在对新索引键（newkey）赋值时（mytable.newkey = "新值2"），会调用元方法，而不进行赋值。而如果对已存在的索引键（key1），则会进行赋值，而不调用元方法 `__newindex`。

***为表添加操作符***

|模式|描述|
|:--|:--|
|__add|对应的运算符 '+'.|
|__sub|对应的运算符 '-'.|
|__mul|对应的运算符 '*'.|
|__div|对应的运算符 '/'.|
|__mod|对应的运算符 '%'.|
|__unm|对应的运算符 '-'.|
|__concat|对应的运算符 '..'.|
|__eq|对应的运算符 '=='.|
|__lt|对应的运算符 '<'.|
|__le|对应的运算符 '<='.|
|__call|__call 元方法在 Lua 调用一个值时调用|
|__tostring|__tostring 元方法用于修改表的输出行为|


#### 协同程序(coroutine)

Lua 协同程序(coroutine)与线程比较类似：拥有独立的堆栈，独立的局部变量，独立的指令指针，同时又与其它协同程序共享全局变量和其它大部分东西。

***线程和协同程序区别***
线程与协同程序的主要区别在于，一个具有多个线程的程序可以同时运行几个线程，而协同程序却需要彼此协作的运行。

在任一指定时刻只有一个协同程序在运行，并且这个正在运行的协同程序只有在明确的被要求挂起的时候才会被挂起。

协同程序有点类似同步的多线程，在等待同一个线程锁的几个线程有点类似协同

|方法|描述|
|:--|:--|
|coroutine.create()|创建 coroutine，返回 coroutine， 参数是一个函数，当和 resume 配合使用的时候就唤醒函数调用|
|coroutine.resume()|重启 coroutine，和 create 配合使用|
|coroutine.yield()|挂起 coroutine，将 coroutine 设置为挂起状态，这个和 resume 配合使用能有很多有用的效果|
|coroutine.status()|查看 coroutine 的状态|
||注：coroutine 的状态有三种：dead，suspended，running，具体什么时候有这样的状态请参考下面的程序|
|coroutine.wrap（）|创建 coroutine，返回一个函数，一旦你调用这个函数，就进入 coroutine，和 create 功能重复|
|coroutine.running()|返回正在跑的 coroutine，一个 coroutine 就是一个线程，当使用running的时候，就是返回一个 corouting 的线程号|


***实例***
```
co = coroutine.create(
	function(i, j)
		if i>j then
			print(i);
		else
			print(j);
		end
	end
)
print(coroutine.status(co))
coroutine.resume(co,99, 2)
print(coroutine.status(co))


```

***实例2***
```
-- coroutine_test.lua 文件
co = coroutine.create(
    function(i)
        print(i);
    end
)
 
coroutine.resume(co, 1)   -- 1
print(coroutine.status(co))  -- dead
 
print("----------")
 
co = coroutine.wrap(
    function(i)
        print(i);
    end
)
 
co(1)
 
print("----------")
 
co2 = coroutine.create(
    function()
        for i=1,10 do
            print(i)
            if i == 3 then
                print(coroutine.status(co2))  --running
                print(coroutine.running()) --thread:XXXXXX
            end
            coroutine.yield()
        end
    end
)
 
coroutine.resume(co2) --1
coroutine.resume(co2) --2
coroutine.resume(co2) --3
 
print(coroutine.status(co2))   -- suspended
print(coroutine.running())
 
print("----------")
```

#### Lua 文件 I/O
Lua I/O 库用于读取和处理文件。分为简单模式（和C一样）、完全模式。

+	简单模式（simple model）拥有一个当前输入文件和一个当前输出文件，并且提供针对这些文件相关的操作。
+	完全模式（complete model） 使用外部的文件句柄来实现。它以一种面对对象的形式，将所有的文件操作定义为文件句柄的方法

打开文件操作语句如下：
```
file = io.open (filename [, mode])
```

mode 的值有：

|模式|描述|
|:--|:--|
|r|以只读方式打开文件，该文件必须存在。|
|w|打开只写文件，若文件存在则文件长度清为0，即该文件内容会消失。若文件不存在则建立该文件。|
|a|以附加的方式打开只写文件。若文件不存在，则会建立该文件，如果文件存在，写入的数据会被加到文件尾，即文件原先的内容会被保留。（EOF符保留）|
|r+|以可读写方式打开文件，该文件必须存在。|
|w+|打开可读写文件，若文件存在则文件长度清为零，即该文件内容会消失。若文件不存在则建立该文件。|
|a+|与a类似，但此文件可读可写|
|b|二进制模式，如果文件是二进制文件，可以加上b|
|+|号表示对文件既可以读也可以写|

***简单模式***
简单模式使用标准的 I/O 或使用一个当前输入文件和一个当前输出文件。

```
-- 以只读方式打开文件
file = io.open("test.lua", "r")

-- 设置默认输入文件为 test.lua
io.input(file)

-- 输出文件第一行
print(io.read())

-- 关闭打开的文件
io.close(file)

-- 以附加的方式打开只写文件
file = io.open("test.lua", "a")

-- 设置默认输出文件为 test.lua
io.output(file)

-- 在文件最后一行添加 Lua 注释
io.write("--  test.lua 文件末尾注释")

-- 关闭打开的文件
io.close(file)
```

其中 io.read() 中我们没有带参数，参数可以是下表中的一个：

|模式|描述|
|:--|:--|
|"*n"|读取一个数字并返回它。例：file.read("*n")|
|"*a"|从当前位置读取整个文件。例：file.read("*a")|
|"*l"（默认）|读取下一行，在文件尾 (EOF) 处返回 nil。例：file.read("*l")|
|number|返回一个指定字符个数的字符串，或在 EOF 时返回 nil。例：file.read(5)|



其他的 io 方法有：

+	io.tmpfile():返回一个临时文件句柄，该文件以更新模式打开，程序结束时自动删除
+	io.type(file): 检测obj是否一个可用的文件句柄
+	io.flush(): 向文件写入缓冲中的所有数据
+	io.lines(optional file name): 返回一个迭代函数,每次调用将获得文件中的一行内容,当到文件尾时，将返回nil,但不关闭文件


***完全模式***
通常我们需要在同一时间处理多个文件。我们需要使用 file:function_name 来代替 io.function_name 方法。以下实例演示了如何同时处理同一个文件:

```
-- 以只读方式打开文件
file = io.open("test.lua", "r")

-- 输出文件第一行
print(file:read())

-- 关闭打开的文件
file:close()

-- 以附加的方式打开只写文件
file = io.open("test.lua", "a")

-- 在文件最后一行添加 Lua 注释
file:write("--test")

-- 关闭打开的文件
file:close()
```

其他方法:

+	file:seek(optional whence, optional offset): 设置和获取当前文件位置,成功则返回最终的文件位置(按字节),失败则返回nil加错误信息。参数 whence 值可以是:
	+	"set": 从文件头开始
	+	"cur": 从当前位置开始[默认]
	+	"end": 从文件尾开始
	+	offset:默认为0
	+	不带参数file:seek()则返回当前位置,file:seek("set")则定位到文件头,file:seek("end")则定位到文件尾并返回文件大小
+	file:flush(): 向文件写入缓冲中的所有数据
+	io.lines(optional file name): 打开指定的文件filename为读模式并返回一个迭代函数,每次调用将获得文件中的一行内容,当到文件尾时，将返回nil,并自动关闭文件。


#### 错误处理
程序运行中错误处理是必要的，在我们进行文件操作，数据转移及web service 调用过程中都会出现不可预期的错误。如果不注重错误信息的处理，就会造成信息泄露，程序无法运行等情况。

任何程序语言中，都需要错误处理。错误类型有：

+	语法错误
+	运行错误

***assert***
```
-- assert
local function add(a,b)
   assert(type(a) == "number", "a 不是一个数字")
   assert(type(b) == "number", "b 不是一个数字")
   return a+b
end
add(10)
```

***error***
语法如下：
`error (message [, level])`

Level参数指示获得错误的位置:
+	Level=1[默认]：为调用error位置(文件+行号)
+	Level=2：指出哪个调用error的函数的函数
+	Level=0:不添加错误位置信息

***pcall 和 xpcall、debug***
Lua中处理错误，可以使用函数pcall（protected call）来包装需要执行的代码。

pcall接收一个函数和要传递给后者的参数，并执行，执行结果：有错误、无错误；返回值true或者或false, errorinfo。

```
if pcall(function_name, ….) then
-- 没有错误
else
-- 一些错误
end
```

#### Lua 垃圾回收
Lua 采用了自动内存管理。 这意味着你不用操心新创建的对象需要的内存如何分配出来， 也不用考虑在对象不再被使用后怎样释放它们所占用的内存。

Lua 运行了一个垃圾收集器来收集所有死对象 （即在 Lua 中不可能再访问到的对象）来完成自动内存管理的工作。 Lua 中所有用到的内存，如：字符串、表、用户数据、函数、线程、 内部结构等，都服从自动管理。Lua 实现了一个增量标记-扫描收集器。 它使用这两个数字来控制垃圾收集循环： 垃圾收集器间歇率和垃圾收集器步进倍率。 这两个数字都使用百分数为单位 （例如：值 100 在内部表示 1 ）。

Lua 提供了以下函数collectgarbage ([opt [, arg]])用来控制自动内存管理:

+	collectgarbage("collect"): 做一次完整的垃圾收集循环。通过参数 opt 它提供了一组不同的功能：
+	collectgarbage("count"): 以 K 字节数为单位返回 Lua 使用的总内存数。 这个值有小数部分，所以只需要乘上 1024 就能得到 Lua 使用的准确字节数（除非溢出）。
+	collectgarbage("restart"): 重启垃圾收集器的自动运行。
+	collectgarbage("setpause"): 将 arg 设为收集器的 间歇率。 返回 间歇率 的前一个值。
+	collectgarbage("setstepmul"): 返回 步进倍率 的前一个值。
+	collectgarbage("step"): 单步运行垃圾收集器。 步长"大小"由 arg 控制。 传入 0 时，收集器步进（不可分割的）一步。 传入非 0 值， 收集器收集相当于 Lua 分配这些多（K 字节）内存的工作。 如果收集器结束一个循环将返回 true 。
+	collectgarbage("stop"): 停止垃圾收集器的运行。 在调用重启前，收集器只会因显式的调用运行。



#### 面向对象编程

面向对象编程（Object Oriented Programming，OOP）是一种非常流行的计算机编程架构。

以下几种编程语言都支持面向对象编程：

+	C++
+	Java
+	Objective-C
+	Smalltalk
+	C#
+	Ruby

面向对象特征
+	封装：指能够把一个实体的信息、功能、响应都装入一个单独的对象中的特性。
+	继承：继承的方法允许在不改动原程序的基础上对其进行扩充，这样使得原功能得以保存，而新功能也得以扩展。这有利于减少重复编码，提高软件的开发效率。
+	多态：同一操作作用于不同的对象，可以有不同的解释，产生不同的执行结果。在运行时，可以通过指向基类的指针，来调用实现派生类中的方法。
+	抽象：抽象(Abstraction)是简化复杂的现实问题的途径，它可以为具体问题找到最恰当的类定义，并且可以在最恰当的继承级别解释问题。

#### Lua 数据库访问

Lua 连接MySql 数据库：
```
require "luasql.mysql"

--创建环境对象
env = luasql.mysql()

--连接数据库
conn = env:connect("数据库名","用户名","密码","IP地址",端口)

--设置数据库的编码格式
conn:execute"SET NAMES UTF8"

--执行数据库操作
cur = conn:execute("select * from role")

row = cur:fetch({},"a")

--文件对象的创建
file = io.open("role.txt","w+");

while row do
    var = string.format("%d %s\n", row.id, row.name)

    print(var)

    file:write(var)

    row = cur:fetch(row,"a")
end


file:close()  --关闭文件对象
conn:close()  --关闭数据库连接
env:close()   --关闭数据库环境
```

### 三、Redis中使用Lua

***Lua 脚本***

+	EVAL：`EVAL script numkeys key [key …] arg [arg …]`
	+	使用 EVAL 命令对 Lua 脚本进行求值
+	EVALSHA：`EVALSHA sha1 numkeys key [key …] arg [arg …]`
	+	根据给定的 sha1 校验码，对缓存在服务器中的脚本进行求值
+	SCRIPT_LOAD：`SCRIPT LOAD script`
	+	加载脚本
+	SCRIPT_EXISTS：`SCRIPT EXISTS sha1 [sha1 …]`
	+	给定一个或多个脚本的 SHA1 校验和，返回一个包含 0 和 1 的列表，表示校验和所指定的脚本是否已经被保存在缓存当中。
+	SCRIPT FLUSH
	+	清除所有 Lua 脚本缓存。
+	SCRIPT KILL
	+	杀死当前正在运行的 Lua 脚本


***实例***
```
eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 val1 val2

127.0.0.1:6379> SCRIPT LOAD "return 'hello world'"
"5332031c6b470dc5a0dd9b4bf2030dea6d65de91"

127.0.0.1:6379> SCRIPT LOAD "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 
"a42059b356c875f0717db19a51f6aaca9ae659ea"
127.0.0.1:6379> EVALSHA "a42059b356c875f0717db19a51f6aaca9ae659ea" 2 key1 key2 val1 val2
1) "key1"
2) "key2"
3) "val1"
4) "val2"

-- 清空缓存
SCRIPT FLUSH

-- 杀死目前正在执行的脚本
SCRIPT KILL 
```

### 四、参考文章
> [Lua 教程](https://www.runoob.com/lua/lua-tutorial.html)

