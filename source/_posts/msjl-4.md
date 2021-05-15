---
title: 面试记录（四）某voip
date: 2020-10-14 20:21:18
tags: [ms]
---



### 零、什么是voip

基于IP的语音传输（英语：Voice over Internet Protocol，缩写为VoIP）是一种语音通话技术，经由网际协议（IP）来达成语音通话与多媒体会议，也就是经由互联网来进行通信。其他非正式的名称有IP电话（IP telephony）、互联网电话（Internet telephony）、宽带电话（broadband telephony）以及宽带电话服务（broadband phone service）。


### 一、实体、属性与联系

在E－R模型中实体、属性、联系各指的是什么？

+	实体：表示一个离散对象，其代表软件系统中客观存在的生zhi活中的实物，如人、动物，物体、列dao表、部门、项目等。
+	属性：是实体中的所有特性。如用户有姓名、性别、住址、电话等。"实体标识符"是在一个实体中，能够唯一标识实体的属性和属性集的标示符。实体的属性用椭圆框表示，框内写上属性名，并用无向边与其实体集相连。
+	联系：实体不会是单独存在的，实体和其他的实体之间有着联系。实体间的联系用菱形框表示，联系以适当的含义命名，名字写在菱形框中，用无向连线将参加联系的实体矩形框分别与菱形框相连，并在连线上标明联系的类型。


ER图中的四个基本成分：
1. 矩形框，表示实体
2. 菱形框，表示实体之间的联系
3. 椭圆形框，表示实体或联系的属性
4. 直线，连接实体、属性、和联系。直线端部标注联系的种类（1：1、1：N或M：N）

ER图如下：

![ER图](/image/mysql/stu_course_tea.png)

### 二、java实现亿级数据等分存储到不同的Redis节点

假设存在四亿数据，存在Redis四个节点，请设计get，set方法，把数据均匀的分配到Redis服务器上。类似的业务逻辑代码如下

```
public class RedisTool {
//    假设clustersList 为集群节点合集
	List<RedisCluster> clustersList;

    public void set(String key, String value) {
        int no = getRedisNo(key, clustersList.size());
        RedisCluster node = clustersList.get(no);
		Jedis jedis = new Jedis(cluster.getHostIp(), Integer.valueOf(cluster.getPort()));  
		jedis.set(key, value);
    }

    public String get(String key) {
        int no = getRedisNo(key, clustersList.size());
        RedisCluster node = clustersList.get(no);
		Jedis jedis = new Jedis(cluster.getHostIp(), Integer.valueOf(cluster.getPort()));  
		return jedis.get(key);
    }

    public static int getRedisNo(String key, int nodeSize) {
        long hashKey = hash(key);
        int redisNo = (int) (hashKey % nodeSize);
        return redisNo;
    }

    public static long hash(String k) {
        CRC32 crc32 = new CRC32();
        crc32.update(k.getBytes());
        return crc32.getValue();
    }
}
```

##### CRC32

CRC（Cyclic Redundancy Check）校验实用程序库在数据存储和数据通讯领域，为了保证数据的正确，就不得不采用检错的手段。在诸多检错手段中，CRC是最著名的一种。CRC的全称是循环冗余校验。

CRC32会把字符串，生成一个long长整形的唯一性ID。

1. 相关源码

```
package java.util.zip;

import java.nio.ByteBuffer;
import sun.nio.ch.DirectBuffer;

public class CRC32 implements Checksum {
    private int crc;

    /**
     * Creates a new CRC32 object.
     */
    public CRC32() {
    }


    /**
     * Updates the CRC-32 checksum with the specified byte (the low
     * eight bits of the argument b).
     *
     * @param b the byte to update the checksum with
     */
    public void update(int b) {
        crc = update(crc, b);
    }

    /**
     * Updates the CRC-32 checksum with the specified array of bytes.
     *
     * @throws  ArrayIndexOutOfBoundsException
     *          if {@code off} is negative, or {@code len} is negative,
     *          or {@code off+len} is greater than the length of the
     *          array {@code b}
     */
    public void update(byte[] b, int off, int len) {
        if (b == null) {
            throw new NullPointerException();
        }
        if (off < 0 || len < 0 || off > b.length - len) {
            throw new ArrayIndexOutOfBoundsException();
        }
        crc = updateBytes(crc, b, off, len);
    }

    /**
     * Updates the CRC-32 checksum with the specified array of bytes.
     *
     * @param b the array of bytes to update the checksum with
     */
    public void update(byte[] b) {
        crc = updateBytes(crc, b, 0, b.length);
    }

    /**
     * Updates the checksum with the bytes from the specified buffer.
     *
     * The checksum is updated using
     * buffer.{@link java.nio.Buffer#remaining() remaining()}
     * bytes starting at
     * buffer.{@link java.nio.Buffer#position() position()}
     * Upon return, the buffer's position will
     * be updated to its limit; its limit will not have been changed.
     *
     * @param buffer the ByteBuffer to update the checksum with
     * @since 1.8
     */
    public void update(ByteBuffer buffer) {
        int pos = buffer.position();
        int limit = buffer.limit();
        assert (pos <= limit);
        int rem = limit - pos;
        if (rem <= 0)
            return;
        if (buffer instanceof DirectBuffer) {
            crc = updateByteBuffer(crc, ((DirectBuffer)buffer).address(), pos, rem);
        } else if (buffer.hasArray()) {
            crc = updateBytes(crc, buffer.array(), pos + buffer.arrayOffset(), rem);
        } else {
            byte[] b = new byte[rem];
            buffer.get(b);
            crc = updateBytes(crc, b, 0, b.length);
        }
        buffer.position(limit);
    }

    /**
     * Resets CRC-32 to initial value.
     */
    public void reset() {
        crc = 0;
    }

    /**
     * Returns CRC-32 value.
     */
    public long getValue() {
        return (long)crc & 0xffffffffL;
    }

    private native static int update(int crc, int b);
    private native static int updateBytes(int crc, byte[] b, int off, int len);

    private native static int updateByteBuffer(int adler, long addr,
                                               int off, int len);
}
```

2. java 中使用CRC32对字符串进行编码

```
    public static long hash(String k) {
        CRC32 crc32 = new CRC32();
        crc32.update(k.getBytes());
        return crc32.getValue();
    }
```

### 三、什么是TCP协议中滑动窗口


滑动窗口通俗来讲就是一种流量控制技术。它本质上是描述接收方的TCP数据报缓冲区大小的数据，发送方根据这个数据来计算自己最多能发送多长的数据，如果发送方收到接收方的窗口大小为0的TCP数据报，那么发送方将停止发送数据，等到接收方发送窗口大小不为0的数据报的到来。


对于TCP会话的发送方，任何时候在其发送缓存内的数据都可以分为4类：

+	已经发送并得到对端ACK的（发送窗口只有收到对端对于本段发送窗口内字节的ACK确认，才会移动发送窗口的左边界）
+	已经发送但还未收到对端ACK的（窗口不会移动，并不对后续字节确认）
+	未发送但对端允许发送的（接收方发送窗口大小不为0的TCP数据报）
+	未发送且对端不允许发送（接收方发送窗口大小为0的TCP数据报）

应用程序在需要（如内存不足）时，通过API通知TCP协议栈缩小TCP的接收窗口。然后TCP协议栈在下个段发送时包含新的窗口大小通知给对端，对端按通知的窗口来改变发送窗口，以此达到减缓发送速率的目的。

### 四、参考文章
> [TCP 滑动窗口（发送窗口和接收窗口）](https://my.oschina.net/xinxingegeya/blog/485650)

