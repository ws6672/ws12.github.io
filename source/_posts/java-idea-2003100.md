---
title: idea_2003100 错误处理笔记
date: 2020-03-10 09:07:10
tags: [tool]
---

### Syntax error on token "Invalid Character", delete this token 的解决

+	标点符号是不是中文，或者全角半角的问题了
+	编码不同，【utf-8】与【utf-8 BOM】 的差异导致的异常

##### BOM
BOM（Byte Order Mark），字节顺序标记，出现在文本文件头部，Unicode编码标准中用于标识文件是采用哪种格式的编码。

在UCS 编码中有一个叫做 "Zero Width No-Break Space" ，中文译名作“零宽无间断间隔”的字符，它的编码是 FEFF。而 FFFE 在 UCS 中是不存在的字符，所以不应该出现在实际传输中。UCS 规范建议我们在传输字节流前，先传输字符 "Zero Width No-Break Space"。这样如果接收者收到 FEFF，就表明这个字节流是 Big-Endian 的；如果收到FFFE，就表明这个字节流是 Little- Endian 的。因此字符 "Zero Width No-Break Space" （“零宽无间断间隔”）又被称作 BOM。
UTF-8文件中放置BOM主要是微软的习惯，但是放在别的系统上会出现问题,不含BOM的UTF-8才是标准形式

