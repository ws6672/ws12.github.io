---
title: java基础（三）BigDecimal
date: 2019-12-12 17:33:29
tags: [java]
---


# 一、基础
float只能用来进行科学计算或工程计算，在大多数的商业计算中，一般采用java.math.BigDecimal 类来进行精确计算。因为BigDecimal类的精度更高，计算结果可以更为精确。


*BigDecimal定义*
	`public class BigDecimal extends Number implements Comparable<BigDecimal> `

### 初始化
```
// 构造方法 参数大体有三类char、String、double
public BigDecimal(char[] val);
public BigDecimal(String val);
public BigDecimal(double val);

// 静态方法获取
public static BigDecimal valueOf(long val);
```

### 加减乘除

```
public BigDecimal add(BigDecimal value);                        //加法
public BigDecimal subtract(BigDecimal value);                   //减法 
public BigDecimal multiply(BigDecimal value);                   //乘法
public BigDecimal divide(BigDecimal value);                     //除法
```

### valueOf方法

// 常用值 0-10对应的BigDecimal对象已经初始化在【zeroThroughTen】中

```
public static BigDecimal valueOf(long val) {
	if (val >= 0 && val < zeroThroughTen.length)
		return zeroThroughTen[(int)val];
	else if (val != INFLATED)
		return new BigDecimal(null, val, 0, 0);
	return new BigDecimal(INFLATED_BIGINT, val, 0, 0);
}
```



# 二、实战

### 增删改查后值不变的困惑

下面的代码执行后，sum的值是没有变化的，查看了源码，才发现虽然调用add方法，但对象本身的值是不变的。
```
for (MdaBasPlant plant: plants) {
	sum.add(new BigDecimal(plant.getZzjrl()));
}
```

// add方法 
// 如果【A】调用add方法做加法操作，但【A】的值是不变的，需要保存返回值
private static BigDecimal add(long xs, long ys, int scale){
	long sum = add(xs, ys);
	if (sum!=INFLATED)
		return BigDecimal.valueOf(sum, scale);
	return new BigDecimal(BigInteger.valueOf(xs).add(ys), scale);
}

//divide 也有类似的代码段，返回的是新的对象

```
private static BigDecimal divideAndRound(BigInteger bdividend, BigInteger bdivisor, int scale, int roundingMode,
										 int preferredScale) {
	 ......
	if (!isRemainderZero) {
		if (needIncrement(mdivisor, roundingMode, qsign, mq, mr)) {
			mq.add(MutableBigInteger.ONE);
		}
		return mq.toBigDecimal(qsign, scale);
	} else {
		......
	}
}
```

###  【divide】Non-terminating decimal expansion; no exact representable decimal result
注：BigDecimal的divide方法出现了无限循环小数
`public BigDecimal divide(BigDecimal divisor, int scale, int roundingMode) `


```
//指定两位小数，向上转型
result.divide(sum,5,BigDecimal.ROUND_ROUND_FLOOR)
```

### 打印字符串
作为非基础类型，BigDecimal 类型转为字符串需要按照以下的方法：
`String str = String.valueOf(sum.longValue());`

# 三、总结
+	BigDecimal都是不可变的（immutable）的，在进行每一步运算时，都会产生一个新的对象，所以在做加减乘除运算时需要保存操作后的值。
+   使用divide方法前需要对传入参数做空判断
+	使用divide方法最好限制位数，避免无限小数的错误

# 参考文章
>	[Java BigDecimal详解](https://www.cnblogs.com/sevenStar-cn/p/11018186.html)