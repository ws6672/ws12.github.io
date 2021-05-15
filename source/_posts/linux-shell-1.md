---
title: Bash Shell 学习篇（一）基础
date: 2019-09-13 13:07:00
tags: linux
---

开卷语
> Bash Shell 是Linux中的脚本语言，通过它我们可以基于常用的命令，编写出通用、高效的工具脚本。


最近，我在学习Neo4j时用Linux搭建了图服务器，为此重新温习了 Linux 的命令。借此，系统的学习一下 【什么是Shell】。

***

### 一 、Shell 基础

+ 什么是 shell
	+ Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。  
	+ Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。  
	+ Ken Thompson 的 sh 是第一种 Unix Shell，Windows Explorer 是一个典型的图形界面 Shell。  

+ Shell 脚本
	+ Shell 脚本（shell script），是一种为 shell 编写的脚本程序。业界所说的 shell 通常都是指 shell 脚本，而shell 【开发Shell】和 shell script 【Shell脚本编程】是两个不同的概念；和它相似的是，批处理语言。  
	
+ Shell 分类
	+ Bourne Shell（/usr/bin/sh或/bin/sh）
	+ Bourne Again Shell（/bin/bash）
	+ C Shell（/usr/bin/csh）
	+ K Shell（/usr/bin/ksh）
	+ Shell for Root（/sbin/sh）

绝大部分Linux系统所用的Shell脚本都是【bash、sh】，这两者中bash是sh的超集，即sh是bash的一个子集。所有的sh编译通过的命令都可以在bash上编译 。在实际使用中，并没有区分【bash】和【sh】。而我所使用的Linux系统是【CentOS 7】，使用的正是【bash】脚本。


+ 运行脚本的几种方法

	+ 通过【sh】或者【bash】命令来执行；

	+ 通过【source】来执行；

	+ 通过【./xx.sh】来执行，需要【r & w】权限，即读写权限。

***

### 二 、基本语法
由于我是在学完批处理【bat】基础后才学习【shell】的，发现两者有许多语法是类似的，只是运行的平台不一样罢了。

##### 1. hello world

***开头***

```
第一行可以是以下的两种，表示的是要调用哪个Shell编译器
 #!/bin/sh
 #!/bin/bash
```

***实例【hello world】***

```
 #!/bin/bash
 echo "hello world"
```

##### 2. shell 的变量

变量的命名规则如下：

+ 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。

+ 中间不能有空格，可以使用下划线（_）；变量与等号间不能有空格

+ 不能使用标点符号, 不加【$】符号

+ 不能使用bash里的关键字（可用help命令查看保留关键字）。



***变量类型*** 

+ 局部变量：使用【local】在函数中定义
+ 环境变量：系统定义
+ shell变量：脚本中自定义，默认是全局变量

***使用***

```
# 输出变量 'echo'
	the_name="jack"
	echo $the_name
	
# 注释 '#'
	# 单行注释
	echo '单行注释'

# 多行注释
	#!/bin/bash
	:<<!
	echo 多行注释
	!
	echo '上面是一个多行注释'



# 只读变量 'readonly'
	#!/bin/sh
	the_name="jack"
	readonly the_name
	echo $the_name
	# script.sh: line 5: the_name: readonly variable 变量不可变
	the_name="po" 

# 删除变量 unset
	#!/bin/sh
	the_name="jack"
	unset the_name
	echo $the_name
```



##### 3.  字符串 

使用单引号，双引号包裹。

+	单引号字符串：原样输出，变量无效；一对单引号里可以使用另一对单引号。
+	双引号字符串：可以有变量；可以有转义字符。

***使用***

```
# 拼接字符串
	#!/bin/sh
	your_name='pt'
	greeting_1="hello, ${your_name} !"
	greeting_2='hello, '$your_name' !'
	echo $greeting_1
	echo $greeting_2

# 字符串长度
	#!/bin/sh
	your_name="abcdefg"
	echo ${#your_name}

# 提取子字符串
	#!/bin/sh
	str="runoob is a great site"
	echo ${str:1:4} # 输出 unoo
```

##### 4.  Shell数组

支持一维数组，无限定大小。在Shell中，数组用空格分隔开来：`数组名=(值1 值2 ... 值n)`

***使用***

```
# 输出元素
	#!/bin/sh
	arr=("a" "b" "c" "d")
	echo ${arr[2]} # 读取单个元素
	echo ${arr[@]} # 读取所有元素

# 获取长度
	# 取得数组元素的个数
	length=${#array_name[@]}
	length=${#array_name[*]}
	# 取得数组单个元素的长度
	lengthn=${#array_name[n]}

```


##### 5. 参数传递

在脚本中，使用【$n】来获取传递到脚本的参数，n是数字，代表第几个参数。

***参数传递实例***

```
#!/bin/bash
echo "hello world"
the_name=$1
echo "hello $the_name"

$ bash hello.sh jack
	hello world
	hello jack
```


***带特殊字符的参数***

```
# 显示参数的个数
echo $#
# 显示所有参数,等价于【"1 2 3" 一个参数】
echo $*
# 显示当前进程ID号
echo $$
# 显示最后一个进程的ID号
echo $!
# 显示所有参数，结果带引号,等价于【"1" "2" "3" 三个参数】
echo $@
# 类似【set】命令
echo $-
# 显示命令的退出状态
echo $?
```

##### 7. 基本运算符

+ 算数运算符 (`+ - * / % = == !=`)
	+	实例：```
	条件表达式【== ！=】需要放置在'[]' 中，并空格分隔；
	其它算数运算符要用命令【expr】进行求值，需要使用【`】括起符号；
	乘以号前需要加【\】。
	
		a=10
		b=100        
		num=`expr $a + $b`
		echo $num
		num2=`expr $a \* $b`
		echo $num2
		
	但是，使用expr命令执行速度慢(需要调用一个新的 shell 来处理)，而且乘号需要转义，可读性差.
	所以，可以使用【$(( ))】,中间放置表达式即可，无须前置 $ 符。	
	```

+ 关系运算符
	+	分类
		-eq 相等 
		-ne 不等
		-gt 左边大于右边
		-lt 左边小于右边
		-ge 左边大于等于右边
		-lw 左边小于等于右边
		加前缀【-】，成立返回【true】
		数字比大小，字符串比长度
		
	+	实例：```
			#!/bin/sh
			a=1
			b=5
			if [ $a -gt $b ]
				then
					echo "$a"
				else
					echo "$b"
			fi
		```

+ 布尔运算符
	+	实例：```
	! 非运算
	-o 或运算
	-a 与运算
	```
+ 逻辑运算符
	+	实例：```
	&& 逻辑与
	|| 逻辑或
	```
	
+ 字符串运算符
	+	实例：```
	=	检测两个字符串是否相等，相等返回 true。	[ $a = $b ] 返回 false。
	!=	检测两个字符串是否相等，不相等返回 true。	[ $a != $b ] 返回 true。
	-z	检测字符串长度是否为0，为0返回 true。	[ -z $a ] 返回 false。
	-n	检测字符串长度是否为0，不为0返回 true。	[ -n "$a" ] 返回 true。
	$	检测字符串是否为空，不为空返回 true。	[ $a ] 返回 true。
	```
+ 文件测试运算符




##### 8. echo

【echo】是shell一个基础的命令，用于输出字符串。

**显示字符串**

```
echo "hello"
```

**使用转义字符**

```
echo "\"hello\""
```

**使用变量**

```
 echo "$num"
```

**显示换行**

```
echo -e "haha\n"
```

**显示不换行**

```
echo -e "haha\c"
```

**输出到文件**

```
echo "haha" > myfile
```

**显示命令执行结果，使用反引号【`】**

```
echo `date`
```


##### 9. 流程控制

**if..else语句**

```
	if [ $a -gt $b ]
	then
	 echo "gt"
	elif [ $a -lt $b ]
	then
	 echo "lt"
	else
	 echo "equal"
	fi
```

**for循环**

```
for var in item1 item2 ... itemN
do
     echo "The value is: $var"
done
```

**while语句**

```
while condition
do
    command
done
```


**死循环**

```
while :
do
    command
done
```

**until语句**

```
until condition
do
    command
done
```

**case语句**

与case公用的两个关键字：break（跳出所有循环），continue（跳出当前循环）

```
case 值 in
模式1)
    command1
    ...
    commandN
    ;;
模式2）
	command1
    ...
    commandN
    ;;
	
```

##### 10. Shell 输入输出重定向

```
command > file	将输出重定向到 file。
command < file	将输入重定向到 file。
command >> file	将输出以追加的方式重定向到 file。
n > file	将文件描述符为 n 的文件重定向到 file。
n >> file	将文件描述符为 n 的文件以追加的方式重定向到 file。
n >& m	将输出文件 m 和 n 合并。
n <& m	将输入文件 m 和 n 合并。
<< tag	将开始标记 tag 和结束标记 tag 之间的内容作为输入。
```

***

### 三、bash计算器
bash 中的基本算术运算只支持整数运算，如果需要进行浮点运算，可以安装bash计算器。bc不只是一个命令，它还是一个编程语言，它支持多种格式的数据。

##### 1. 支持的数据格式

+ 数字（包括整数和浮点数）
+ 变量（简单变量和数组）
+ 注释（# 或 /* */ 都能识别）
+ 表达式
+ 编程语句（if-then 等）
+ 函数

##### 2. 安装

```
	sudo yum install bc
```

##### 3. 在脚本中使用

使用反引号`` 或者 $() 将 bc 命令包含起来即可。

可以参数【options】，用于进行一些变量的设置；如果存在多个变量，用分号隔开即可。

```
value=$(echo "options;expression" | bc)
```

***

### 四、test命令
test 命令用于检查某个条件是否成立。
 
##### 1. 分类

+ 数值测试
+ 字符串测试
+ 文件测试

##### 2. 数值测试

```
	-eq	等于则为真
	-ne	不等于则为真
	-gt	大于则为真
	-ge	大于等于则为真
	-lt	小于则为真
	-le	小于等于则为真
```

```
	if test $[num1] -eq $[num2]
```

##### 3. 字符串测试

```
	=	等于则为真
	!=	不相等则为真
	-z 字符串	字符串的长度为零则为真
	-n 字符串	字符串的长度不为零则为真
```

```
	if test $num1 = $num2
```

##### 4. 文件测试

```
	-e 文件名	如果文件存在则为真
	-r 文件名	如果文件存在且可读则为真
	-w 文件名	如果文件存在且可写则为真
	-x 文件名	如果文件存在且可执行则为真
	-s 文件名	如果文件存在且至少有一个字符则为真
	-d 文件名	如果文件存在且为目录则为真
	-f 文件名	如果文件存在且为普通文件则为真
	-c 文件名	如果文件存在且为字符型特殊文件则为真
	-b 文件名	如果文件存在且为块特殊文件则为真
```

```
	if test -e ./bash
```

***

### 五、printf 命令
+ printf 命令模仿 C 程序库（library）里的 printf() 程序。
+ printf 由 POSIX 标准所定义，因此使用 printf 的脚本比使用 echo 移植性好。
+ printf 使用引用文本或空格分隔的参数，外面可以在 printf 中使用格式化字符串，还可以制定字符串的宽度、左右对齐方式等。默认 printf 不会像 echo 自动添加换行符，我们可以手动添加 \n。


##### 1. 语法

```
printf  format-string  [arguments...]
```

##### 2. 转义字符

```
\a	警告字符，通常为ASCII的BEL字符
\b	后退
\c	抑制（不显示）输出结果中任何结尾的换行字符（只在%b格式指示符控制下的参数字符串中有效），而且，任何留在参数里的字符、任何接下来的参数以及任何留在格式字符串中的字符，都被忽略
\f	换页（formfeed）
\n	换行
\r	回车（Carriage return）
\t	水平制表符
\v	垂直制表符
\\	一个字面上的反斜杠字符
\ddd	表示1到3位数八进制值的字符。仅在格式字符串中有效
\0ddd	表示1到3位的八进制值字符
```

***

### 六、函数

##### 1. 基本语法

linux shell 可以用户定义函数，然后在shell脚本中可以随便调用。
调用格式如下：

```
	[ function ] funname [()]
	{
		action;
		[return int;]
	}
```

##### 2. 函数参数的使用

在Shell中，调用函数时可以向其传递参数。在函数体内部，通过 $n 的形式来获取参数的值，例如，$1表示第一个参数，$2表示第二个参数.
当n>=10时，就需要使用${n}来获取参数。


*2.1 特殊占位符*
与【$n】相同的是，以下的占位符都可以在函数中使用

```
$#	传递到脚本的参数个数
$*	以一个单字符串显示所有向脚本传递的参数
$$	脚本运行的当前进程ID号
$!	后台运行的最后一个进程的ID号
$@	与$*相同，但是使用时加引号，并在引号中返回每个参数。
$-	显示Shell使用的当前选项，与set命令功能相同。
$?	显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。
```

*2.2 实例*
```
funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2
```

