---
title: 面试记录（三）三十天（服务器向工程师）
date: 2020-10-13 19:43:14
tags: [ms]
---


两轮折戟，第一轮是笔试，第二轮是技术面。记下来几个印象比较深刻的问题，当时没答出来，技术太差了。


### 一、什么是数据库中的“数据冗余”?

数据冗余是指数据之间的重复，也可以说是同一数据存储在不同数据文件中的现象。可以说增加数据的独立性和减少数据冗余是企业范围信息资源管理和大规模信息系统获得成功的前提条件。

#### 1.用途

数据库设计得不好，也会导致数据冗余。例如表A中存在某个字段，但它又同时出现在另外一个或多个表中，且意义不变，那它就是一个冗余字段。

但是，关系数据库中为了业务的需求，冗余字段又是必须的。冗余字段用途如下：
+	建立数据间的联系（外键）
+	数据恢复（转储日志文件和后备数据）
+	数据核查
+	更好的数据展示效果（视图）
+	减少数据通讯开支（分布式数据库）

####  2.成因

关系数据库的数据冗余形成的原因有如下4类：
+	表的重复
	+	分布式数据库的备份表
+	属性的重复
	+	不同表：为了维护两个表的关系
	+	同一个表：数据安全检查
+	元组的重复
	+	表内不同记录内容有时会完全相同（通过查重去除）
+	属性值的重复
	+	无限类：无限类属性值的重复。无限类属性值是指其属性值域集合的基为无限大或者数据库记录数为同一数量级的属性值，如实数、整数、日期、各种编号
	+	有限类：一对多或多对多导致的

####  3.消除方法

+	消除表的重复所引起的数据冗余为磁盘文件级的操作。
+	属性的重复所引起的数据冗余的消除为对数据库结构修改的操作。
+	元组的重复所引起的数据冗余的消除由记录级的操作完成。

### 二、SpringMVC URL路径注解

`@PathVariable`：是用来获得请求url中的动态参数的

例如：
```
@RequestMapping("update/{id}")  
public String update(@PathVariable Integer id){  
	System.out.println(id);  
	return "redirect:user/list.do";  
}  
```

### 三、MySQL中索引的建立

0. 测试数据(MySQL)

```
-- ----------------------------
-- Table structure for student
-- ----------------------------
DROP TABLE IF EXISTS `student`;
CREATE TABLE `student`  (
  `Sid` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `Sname` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `Sage` datetime(0) NULL DEFAULT NULL,
  `Ssex` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of student
-- ----------------------------
INSERT INTO `student` VALUES ('01', '赵雷', '1990-01-01 00:00:00', '男');
INSERT INTO `student` VALUES ('02', '钱电', '1990-12-21 00:00:00', '男');
INSERT INTO `student` VALUES ('03', '孙风', '1990-05-20 00:00:00', '男');
INSERT INTO `student` VALUES ('04', '李云', '1990-08-06 00:00:00', '男');
INSERT INTO `student` VALUES ('05', '周梅', '1991-12-01 00:00:00', '女');
INSERT INTO `student` VALUES ('06', '吴兰', '1992-03-01 00:00:00', '女');
INSERT INTO `student` VALUES ('07', '郑竹', '1989-07-01 00:00:00', '女');
INSERT INTO `student` VALUES ('08', '王菊', '1990-01-20 00:00:00', '女');
```

1. 建立普通索引
```
-- CREATE INDEX indexName ON table_name (column_name)

create index index_sex on student(Ssex);

-- 修改表结构
-- ALTER table tableName ADD INDEX indexName(columnName)

alter table student add index index_sex(Ssex);

-- 删除
-- DROP INDEX [indexName] ON mytable; 

DROP INDEX index_sex ON student; 
```
2. 唯一索引（值唯一、允许空值）

```
-- CREATE UNIQUE INDEX indexName ON mytable(username(length)) 

CREATE UNIQUE INDEX index_name_uq ON student(Sname);

-- ALTER table mytable ADD UNIQUE [indexName] (username(length))

ALTER table student ADD UNIQUE index_name_uq (Sname);

```


### 四、抽象类可以实现接口吗？

可以。抽象类可以实现接口，还可以决定接口的方法是否实现。Java语言这样设计的意义是为了应对以下的业务场景：

存在一个接口，有十个方法，有几十个实现类，大部分方法的实现方式是相同的，只有一两个方法的业务逻辑不同。如果我们在所有的子类中都实现所有的接口方法，那么就存在严重的代码冗余。而这个时候，我们可以通过抽象类实现接口的方法来避免这个麻烦。例如：

```
public interface TestIf {
    void a();
    void b();
    void c();
}

public abstract class AbstractTestIf implements TestIf {
	 
    public void a(){
		System.out.println("AbstractTestIf complete method a");
	}
    public void b(){
		System.out.println("AbstractTestIf complete method c");
	}
    public abstract  void c();
}

public class Test extends AbstractTestIf {
 
    @Override
    public void c() {
        System.out.println("Test complete method c");
    }
}
```

### 五、抽象类和接口中成员或方法访问权限 

1. 抽象类中成员访问权限，其基本上继承了类的特性，但由于抽象类之所以为抽象类，是因为它是作为父类来使用的，是等待子类去实现的，而类中 private的权限只能是自个访问自个，所以在抽象类中方法为abstract时只有public,protected,default三种访问权限。

2. 而接口中所有成员的属性都为public static final,即接口中的属性自动默认为 pubic static final，也就是说接口中声明的变量都是常量，不能被继承。接口中所有方法的访问属性为public, 即接口中的方法自动默认为 public，所以实现接口中的方法必须标识为public 或者 abstract,否则编译出错。在JAVA 8 或者更高的版本中的可以使用default

### 六、git如何切换分支


1. git中branch（分支）有三种类型：

+	local branch：本地分支，默认是master分支（最新的主分支名被更替为main）
+	remote branch：远程分支，是指服务器中的某个分支，用来追踪远程分支的变化
+	tracking branch：跟踪分支是一种和远程分支有直接联系的本地分支(远程分支的本地书签、别名)，跟踪分支是一种本地分支

2.  git的分支命令

```
# 下载项目
git clone 项目地址


# 查看分支
git branch -a
git branch -v


#  切换远程分支
git checkout -b 1.3 origin/release-1.3

# 创建分支
git branch myrelease
# 切换本地分支
git checkout myrelease

```

3. git的提交格式

```
type: subject

TYPE
	feat ：新功能
	fix : bug修复
	docs :文档变更
	style :与样式相关的所有变动
	refactor :既不是bug修复也未添加功能的代码更改
	test :与测试有关所有变动
	chore :改变了构建任务，程序包管理器配置等

Subject
	包含对所做更改的简短描述。长度不能超过50个字符，应以大写字母开头，命令式的语法
	
body (可选)
	说明你进行了哪些更改以及进行更改的原因
	
footer (可选)
	页脚也是可选的，主要在你使用issue追踪引用issue ID时使用。

```

4. 安全的提交

```
# 1.提交已暂存的文件
git commit

# 2.将服务器代码同步到本地（成功执行步骤4，失败执行步骤3）
git pull

# 3.还原冲突路径
git checkout -- <有冲突的文件路径>

# 4.同步到服务器（失败则再次执行步骤2）
git push origin  <本地分支名>
```

### 七、Linux如何挂载硬盘

1. 查看
```
# 查看
sudo fdisk -l

# 进入硬盘
sudo fdisk /dev/sda

# 查看磁盘使用情况
# df命令用于显示磁盘分区上的可使用的磁盘空间。默认显示单位为KB
df -h
```

2. 挂载命令的使用

```
mount [-t vfstype] [-o options] device dir

a. [-t vfstype] 硬盘类型，无需设置，自动检测
	光盘或光盘镜像：iso9660
	DOS fat16文件系统：msdos
	Windows 9x fat32文件系统：vfat
	Windows NT ntfs文件系统：ntfs
	Mount Windows文件网络共享：smbfs
	UNIX(LINUX) 文件网络共享：nfs
	
b. -o options 挂载选项
	loop用来把一个文件当成硬盘分区挂接上系统
	ro采用只读方式挂接设备
	rw采用读写方式挂接设备
	iocharset指定访问文件系统所用字符集

c. device 要挂接(mount)的设备

d. dir设备在系统上的挂接点(mount point)

```

3. 实例

```
# 1. 查看已挂载硬盘，默认存在（sda、sda1、sda2），新挂载的会显示（sdb1）

[cen@localhost ~]$ sudo fdisk -l|grep '/dev'

磁盘 /dev/sda：21.5 GB, 21474836480 字节，41943040 个扇区
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    41943039    19921920   8e  Linux LVM
磁盘 /dev/mapper/centos-root：18.2 GB, 18249416704 字节，35643392 个扇区
磁盘 /dev/mapper/centos-swap：2147 MB, 2147483648 字节，4194304 个扇区
磁盘 /dev/sdb：15.5 GB, 15472047104 字节，30218842 个扇区
/dev/sdb1   *        2048    30218239    15108096    c  W95 FAT32 (LBA)

# 2. 挂载，目录需要存在；挂载成功没有提示
sudo mount /dev/sdb1 /boot

# 3. 查看Linux挂载磁盘
df -h
# 4. 卸载
sudo umount /dev/sdb1
```

综上，我们需要通过`fdisk`获取需要挂载的盘符号，然后通过`mount`命令挂载。



### 八、参考文章

> [怎么创建一个良好的Git提交信息](https://www.cnblogs.com/three-fighter/p/13547091.html)
[git如何提交代码](https://blog.csdn.net/junli_chen/article/details/52289135)

