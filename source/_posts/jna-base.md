---
title: JNA （Java Native Access）入门
date: 2021-05-26 10:27:07
tags: [jna]
---

如果JNI是连接JAVA与C的独木桥，那么JNA就是在独木桥上的高速通道。

# 一、入门

1. 什么是JNA

JNA(Java Native Access)框架是一个开源的Java框架，是SUN公司主导开发的，建立在经典的JNI的基础之上的一个框架。JNA提供一组Java工具类用于在运行期动态访问系统本地共享类库而不需要编写任何Native/JNI代码。开发人员只要在一个java接口中描述目标native library的函数与结构，JNA将自动实现Java接口到native function的映射，大大降低了Java调用本体共享库的开发难度。

JNI是Java调用原生函数的唯一途径，而JNA就是基于JNI的开发的工具包。使用JNI访问动态链接库过于复杂，而JNA简化了操作，提供了一个动态的C语言编写的转发器（实际上也是一个动态链接库，在Linux-i386中文件名是：libjnidispatch.so）可以自动实现Java与C之间的数据类型映射。从性能上会比JNI技术调用动态链接库要低。

Java Native Access（Java本地访问）具有一个单独的组件jna.jar；支持的Native库（jnidispatch）包含在jar文件中。 JNA能够自行提取和加载Native库，因此不需要其他配置。如果Native库尚未安装在通过`System.loadLibrary`可以访问的本地系统上，则JNA会退出提取。

首先下载最新版本的JNA，然后在项目的CLASSPATH中引用jna.jar；或者通过Maven导入
```
    <dependency>
      <groupId>net.java.dev.jna</groupId>
      <artifactId>jna</artifactId>
      <version>4.1.0</version>
    </dependency>
```


2. 如下所示，示例会将标准C库映射并调用printf函数

```
public class Test {
	// 通过扩展Library接口，声明一个Java接口来保存Native库方法
    interface CLibrary extends Library {
        CLibrary INSTANCE = (CLibrary) Native.loadLibrary((Platform.isWindows() ? "msvcrt" : "c"), CLibrary.class);

        void printf(String format, Object... args);
    }

    public static void main(String[] args) {
        CLibrary.INSTANCE.printf("Hello, World\n");
        for (int i = 0; i < args.length; i++) {
            CLibrary.INSTANCE.printf("Argument %d: %s\n", i, args[i]);
        }
    }

}
```

标识要使用的Native目标库，这可以是具有导出功能的任何共享库。在平台软件包中可以找到许多常见系统库的映射示例，尤其是在Windows上。

在Java程序中使用C库，有多种方法：
+	首选方法是将jna.library.path系统属性设置为目标库的路径。此属性类似于java.library.path，但仅适用于JNA加载的库。
+	启动VM之前，请更改适当的库访问环境变量（在Windows上是PATH、在Linux上是LD_LIBRARY_PATH、在OSX上是DYLD_LIBRARY_PATH）
+	使您的Native库在类路径下{OS}-{ARCH} / {LIBRARY}下可用，其中{OS}-{ARCH}是JNANative库的规范前缀（例如win32-x86，linux-amd64）；如果资源在jar文件中，则在加载时将自动提取该资源。


我们可以通过设置类似于SYNC_INSTANCE的变量，来将库加载到局部变量中：
```
    interface CLibrary extends Library {
        CLibrary INSTANCE = (CLibrary) Native.loadLibrary((Platform.isWindows() ? "msvcrt" : "c"), CLibrary.class);
//        确保单次使用
        CLibrary SYNC_INSTANCE = (CLibrary)
                Native.synchronizedLibrary(INSTANCE);
        void printf(String format, Object... args);
    }
```

3. 结构体

C/C++里有结构体struct,甚至C#中也具有，相关语法如下：

```
-- 大括号后面的分号";"不能少，这是一条完整的语句。
struct 结构体名{
    结构体所包含的变量或数组
};

-- EXAMPLE
struct stu{
    char *name;  //姓名
    int num;  //学号
    int age;  //年龄
    char group;  //所在学习小组
    float score;  //成绩
};

-- 定义结构体数组
struct stu{
    char *name;  //姓名
    int num;  //学号
    int age;  //年龄
    char group;  //所在小组 
    float score;  //成绩
}class[5];

-- 定义结构体指针
struct 结构体名 *变量名;

struct stu{
    char *name;  //姓名
    int num;  //学号
    int age;  //年龄
    char group;  //所在小组
    float score;  //成绩
} stu1 = { "Tom", 12, 18, 'A', 136.5 };
//结构体指针
struct stu *pstu = &stu1;

-- 定义结构体的同时定义结构体指针
struct stu{
    char *name;  //姓名
    int num;  //学号
    int age;  //年龄
    char group;  //所在小组
    float score;  //成绩
} stu1 = { "Tom", 12, 18, 'A', 136.5 }, *pstu = &stu1;
```


而java中并没有结构体这个概念。当调用动态库.so和.dll时，函数接口上很多数据都是结构体，而Java通过定义一个类似于结构体的实体类来表示结构体对象。而jna提供了Structure类，用于定义结构体。例如：
```
import com.sun.jna.Library;
import com.sun.jna.Native;
import com.sun.jna.Platform;
import com.sun.jna.Structure;
import com.sun.jna.win32.StdCallLibrary;
import org.omg.Messaging.SYNC_WITH_TRANSPORT;

import java.util.Arrays;
import java.util.List;

public class Test {
	// 这是一个标准库的Java接口
    interface Kernel32 extends StdCallLibrary {
        // 方法声明，常量和结构定义在此处
        void GetSystemTime(SYSTEMTIME result);
		// 定义结构体
        static class SYSTEMTIME extends Structure {
            public short wYear;
            public short wMonth;
            public short wDayOfWeek;
            public short wDay;
            public short wHour;
            public short wMinute;
            public short wSecond;
            public short wMilliseconds;
			// 返回字段,需要与结构体保持一致
            @Override
            protected List getFieldOrder() {
                return Arrays.asList("wYear", "wMonth", "wDayOfWeek", "wDay", "wHour", "wMinute", "wSecond", "wMilliseconds");
            }
        }
    }

    public static void main(String[] args) {
        Kernel32 lib = (Kernel32) Native.loadLibrary("kernel32", Kernel32.class);
        Kernel32.SYSTEMTIME time = new Kernel32.SYSTEMTIME();
        lib.GetSystemTime(time);
        System.out.println(time.wYear + "年" +time.wMonth+ "月" + time.wDay +"日");
    }


}

```

如果需要映射结构体，我们可以定义一个从Structure库接口定义中派生的公共静态类。它允许将结构共享为库接口定义的任何选项（如自定义类型映射），但是必须按顺序在FieldOrder注解或getFieldOrder()方法返回的列表中包括每个声明的字段名称。

如果代码的结构特别长或复杂，则可以考虑使用由Olivier Chafik编写的[JNAerator](https://code.google.com/p/jnaerator/) 工具，该工具可以为您生成JNA映射。




4. 函数描述

通过`Native.loadLibrary()`实例化Native库接口时，JNA将创建一个代理。该代理通过的`invoke`函数路由所有方法来调用 `Library.Handler`。此方法会查找一个适当的 `Function`对象，该对象表示由Native库导出的函数。代理处理程序可以执行一些初始名称转换，借此从调用的代理函数中获取实际的Native库函数名称。

找到 `Function` 对象后，可以通过`invoke` 函数与所有可用参数来调用通用方法。代理函数的方法签名用于确定传入参数的类型和所需的返回类型。Function对象执行参数的转换，可以将`NativeMapped`类型转换为它们的Native表示形式，或将`TypeMapper`应用于任何传入类型，然后在函数返回时执行类似的转换。默认情况下，所有Structure对象的Java字段均会在调用Native函数之前复制到其Native内存中，并在调用后复制回去。

所有函数调用均基于其返回类型通过不同的Native方法进行路由，但所有这些Native方法均通过native / dispatch.c中的同一分派调用分派。该函数在构建供`libffi`使用的函数调用描述之前，会将Java对象最终转换为Native表示形式。libffi库需要描述目标函数的参数和返回类型，以便执行适合最终本地调用调用的特定于平台的堆栈结构。一旦libffi执行了本地调用 (via ffi_call()),它将结果复制到JNA提供的缓冲区中，然后将其转换回适当的Java对象。


5. 数据类型的使用

C语言中的数据类型与java数据类型可以通过JNA相互转换，Java基本类型（及其对等对象）可以直接映射到相同大小的NativeC类型：

|原生类型|Size|Java 类型|WINDOWS类型|
|:--|:--|:--|:--|
|char|8-bit integer|byte|BYTE, TCHAR|
|short|16-bit integer|short|WORD|
|wchar_t|16/32-bit character|char|TCHAR|
|int|32-bit integer|int|DWORD|
|int|boolean value|boolean|BOOL|
|long|32/64-bit integer|NativeLong|LONG|
|long long|64-bit integer|long|__int64|
|float|32-bit FP|float||
|double|64-bit FP|double||
|char*|C string|String|LPCSTR|
|void*|pointer|Pointer|LPVOID, HANDLE, LPXXX|

无符号类型使用与有符号类型相同的映射，C枚举通常可以与“ int”互换。


6. 指针和数组

原始数组参数（包括结构）由其对应的Java类型表示。例如：

```
//原始C声明
void  fill_buffer（ int * buf， int len）;
void  fill_buffer（ int buf []， int len）; //与数组语法相同
//等效的JNA映射
void fill_buffer（ int [] buf， int len）;
```

> 注意：如果参数由函数调用范围之外的Native函数使用，则必须使用内存或NIO直接缓冲区。Java基本数组提供的内存仅在函数调用期间有效，以供Native代码使用。



7. ByReference参数的使用

当函数接收指针到形参时，可以使用一个ByReference类型来捕获返回的值或者用户自定义的子类。

```
// C语言声明
void allocate_buffer(char **bufp, int* lenp);

// 等效JNA映射
void allocate_buffer(PointerByReference bufp, IntByReference lenp);

// 用法
PointerByReference pref = new PointerByReference();
IntByReference iref = new IntByReference();
lib.allocate_buffer(pref, iref);
Pointer p = pref.getValue();
byte[] buffer = p.getByteArray(0, iref.getValue());
```

或者，也可以使用具有所需类型的单元素Java数组，但ByReference 可以更好地传达代码的意图。 Pointer类除了GetByteArray（）之外，还提供了许多访问者方法。通过`PointerType`类，可以声明安全指针。


8. 从Java到本机的自定义映射

`TypeMapper` 类和相关接口 可以通过参数、返回值或结构成员将 任何Java类型 转换为native类（或相反操作），这种转换限于Java中默认的类型。如果用户自定义类型，那么可以实现`NativeMapped接口`，该接口可以逐级的转换原始类型。

9. 回调，函数指针和闭包（Callbacks, Function Pointers and Closures）

回调声明包括一个简单的接口，该接口扩展了`Callback `接口，并实现回调方法（或定义单个任意名称的方法）。通过在一点C代码中包装Java对象方法来实现回调。最简单的使用类似于使用匿名内部类来注册事件侦听器。以下是回调用法的示例：

```
	// 原生C定义
	typedef void (*sig_t) (int);
	sig_t signal(int sig, sig_t func);
	int SIGUSR1 = 30;
	----------------------------------------------------
	
	// 等价的JNA映射
	public interface CLibrary extends Library {
		int SIGUSR1 = 30;
		interface sig_t extends Callback {
			void invoke(int signal);
		}
		sig_t signal(int sig, sig_t fn);
		int raise(int sig);
	}
	/* ... */
	CLibrary lib = (CLibrary)Native.load("c", CLibrary.class);
	// WARNING: 您必须对回调对象进行引用直到取消解除回调;如果是回调对象被垃圾回收，原生回调的调用将可能崩溃。
	CLibrary.sig_t fn = new CLibrary.sig_t() {
		public void invoke(int sig) {
			System.out.println("signal " + sig + " was raised");
		}
	};
	CLibrary.sig_t old_handler = lib.signal(CLibrary.SIGUSR1, fn);
	lib.raise(CLibrary.SIGUSR1);

```

以下是一个更介绍的示例，使用Win32 API枚举所有本机窗口：
```
	//C声明
	typedef int (__stdcall *WNDENUMPROC)(void*,void*);
	int __stdcall EnumWindows(WNDENUMPROC,void*);

	----------------------------------------------------
	// 等价的JNA映射
	public interface User32 extends StdCallLibrary {
		interface WNDENUMPROC extends StdCallCallback {
			/** Return whether to continue enumeration. */
			boolean callback(Pointer hWnd, Pointer arg);
		}
		boolean EnumWindows(WNDENUMPROC lpEnumFunc, Pointer arg);
	}
	/* ... */
	User32 user32 = User32.INSTANCE;

	user32.EnumWindows(new WNDENUMPROC() {
		int count;
		public boolean callback(Pointer hWnd, Pointer userData) {
			System.out.println("Found window " + hWnd + ", total " + ++count);
			return true;
		}
	}, null);

```

# 二、相关资料
[官网文档](https://github.com/java-native-access/jna/blob/master/www/GettingStarted.md)

