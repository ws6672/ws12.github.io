---
title: redis（三）安全性与持久化
date: 2020-10-10 21:00:12
tags: [Redis]
---

这一节不仅仅要讨论数据库的安全性，也要浅谈一下数据的持久化。

# 一、数据库安全性

一个命令要在Redis生效，需要经过几个安全防护措施的测试：

+	系统自带的防火墙
+	Redis的tcp端口或Unix套接字（redis.conf）
+	轻量级的认证方式（redis.conf）
+	Redis无加密，AUTH命令是明文，可以被窃听。可以使用SSL代理加密
+	禁用的特殊命令


1. 防火墙。
+	通过系统的防火墙，我们可以使非指定ip无法通过端口访问数据库。

2. tcp端口或Unix套接字。
+	采用绑定IP的方式来进行控制，通过编辑redis.conf文件设置允许访问的ip地址。

```
	# If you want you can bind a single interface, if the bind option is not
	# specified all the interfaces will listen for incoming connections.
	#
	bind 127.0.0.1
```

3. 设置密码。

+	我们可以设置密码，以提供远程登陆（redis执行速度快，密码过短容易被暴力破解）

```
# 修改redis.conf文件
	requirepass password1

# 访问redis1
cmd> redis-cli -h  Ip -p Port
auth password1

# 访问redis2
cmd> redis-cli -h  Ip -p Port  -a password1

# 设置临时密码
config set requirepass yourPassword

# 查看密码
config get requirepass
```

4. SSL加密。
+	认证层的目标是提供多一层的保护。假如防火墙或者其它任何系统防护攻击失败的话，外部客户端如果没有认证密码的话将依然无法访问Redis实例。AUTH命令就像其它Redis命令一样，是通过非加密方式发送的，因此无法防止拥有足够的访问网络权限的攻击者进行窃听。 数据加密支持。
+	可通过安装Stunnel客户端，安装后，配置加密的SSL连接。

5. 禁用的特殊命令。

在Redis中可以禁用命令或者将它们重命名成难以推测的名称，这样子普通用户就只能使用部分命令了。例如，一个虚拟化的服务器提供商可能提供管理Redis实例的服务。在这种情况下，普通用户可能不被允许调用CONFIG命令去修改实例的配置，但是能够提供删除实例的系统需要支持修改配置。在这种情况下，你可以从命令表中重命名命令或者禁用命令。这个特性可以在redis.conf文件中进行配置。
重命名命令：`rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52`
禁用该命令：`rename-command CONFIG ""`


# 二、缓存与数据库双写

如何保证缓存与数据库双写，是项目中使用缓存时需要考虑的一个问题。

### 错误的双写流程

1. 先更新缓存,再写数据库（会导致读不一致）
+	线程a更新数据库
+	线程b更新数据库
+	线程b更新缓存
+	线程a更新缓存
+	网络原因导致a线程先更新数据库，但是比b晚更新缓存，最后导致了脏数据

2. 先删除缓存,再更新数据库

+	线程a 准备写操作，先删除缓存
+	线程b 读操作，读缓存没有命中
+	线程b 读数据库，得到旧值，写入缓存
+	线程a 写新数据到数据库


### 四种常见的双写策略

相关的几种设计模式如下：
+	Cache aside（懒加载）
	+	命中：从cache获取数据，返回
	+	失效：从数据库获取数据，成功后，存放到缓存中
	+	更新：更新数据库 数据，成功后，使缓存失效
	
+	Read through
	+	在查询操作中更新缓存
	+	Cache aside由调用方更新缓存，而Read through则用缓存服务自己来加载，从而对应用方是透明的。
+	Write through
	+	在更新操作中更新缓存
	+	如果没有命中缓存，直接更新数据库，然后返回。如果命中了缓存，则更新缓存，然后再由Cache自己更新数据库（同步操作）
+	Write behind caching
	+	在更新数据的时候，只更新缓存，不更新数据库，而我们的缓存会异步地批量更新数据库
	
***redis 的并发竞争***
 多客户端同时并发写一个key，由于网速原因先到的数据晚处理，产生了脏数据；多客户端同时获取一个key，修改顺序错误导致结果错误。
 
 缓存格式可以设置为 数据+时间戳，如果新数据的时间戳更小，那么就忽略。
 

### 安全加固


1. Redis Server监听的端口默认为6379,容易被扫描攻击

```
# 修改为非默认端口
port 6379
```

2. Redis支持监听0.0.0.0,即所有ip都可以接入

```
# 设置为客户端需要连接的网卡；允许本机访问，应该只监听127.0.0.1
bind 127.0.0.1
```

3. RedisCluster方案缺省会增加一个集群端口，且是在客户端端口偏移10000

在端口矩阵中对额外的这个集群端口有说明。修改源码,新增一个redis.conf偏移量配置项cluster-port-increment，缺省配置+1，这样可以做到端口范围可控，避免冲突


4. Redis只有一个超级用户，权限过大

权限最小化原则，增加配置项，角色区分超户，普通用户和只读用户。通过ACL管理用户权限。

5. 危险命令被调用
```
redis.conf 中加入如下内容，禁用相应的命令

rename-command KEYS     ""
rename-command FLUSHALL ""
rename-command DEBUG    ""
```

# 三、ACL

Redis ACL 是 Access Control List（访问控制列表）的缩写，该功能允许对访问 Redis 的连接做一些可执行命令和可访问的 KEY 的限制。它的工作方式是，在连接之后，要求客户端进行身份验证，以提供用户名和有效密码：如果身份验证成功，该
客户端连接与给定用户绑定，并具有该用户的访问权限。 

> 注：Redis ACL向下兼容，存在default用户，拥有操作redis的所有权限。使用 requirepass 设置访问密码的方式与旧版本也保持一致，只不过 Redis 6.0 的这个密码仅对用户 “default” 有效

配置ACL的方式有两种，一种是在config文件中直接配置，另一种是在外部aclfile中配置。配置的命令是一样的，但是两种方式只能选择其中一种

1. 通过config配置
```
requirepass redis
config rewrite
# 自动在底部追加以下配置
user default on nopass ~* +@all
```

2. 通过外部 aclfile 配置
```
aclfile "./user.acl"

# 重启后，执行
config rewrite


# aclfile 内容
user alice on allcommands allkeys >alice
user bob on -@all +@set +acl ~set* >bob
```

AUTH命令修改如下：
```
AUTO <username> <password>
AUTO <password>   ===   AUTO default <password>

```

### 相关语法

ACL是使用DSL（域特定语言）定义的，该DSL描述了给定用户能够执行的操作。此类规则始终按从左到右、从第一个到最后一个执行的。

eg:
```
user default on nopass ~* +@all
# 解析
# user default：用户为default
# on：处于活动状态
# nopass：不需要密码
# ~*：访问所有可能的键
# +@all：调用所有可能的命令
```

1. 启用和禁止用户

+	on：启用用户：可以以该用户身份进行认证
+	off：禁用用户：不再可以与此用户进行身份验证，但是已经过身份验证的连接仍然可以使用

2. 启用或禁用命令

+	`+<command>`  将 `<command>` 命令添加到用户可调用的命令列表中
+	`-<command>`  从可调用的命令列表中移除 `<command>` 命令
+	`+@<category>`  允许用户调用 `<category>` 分类中的所有命令（可通过 ACL CAT 命令查看完成分类列表）
+	`-@<category>`  禁止用户调用 `<category>` 分类中的所有命令
+	`+<command>|subcommand`  允许使用原本禁用的特定类别下的特定子命令
+	`+@all`  允许调用所有命令，与使用 allcommands 效果相同。包括当前存在的命令以及将来通过模块加载的命令
+	`-@all`  禁止调用所有命令

3. 允许或禁止访问某些 KEY

+	`~<pattern>` 添加符合条件的模式
+	resetkeys 使用当前模式覆盖所有允许的模式；例如： `resetkeys ~objects:*` ，最终客户端只允许访问匹配 `~object:*` 模式的 KEY

4. 为用户配置有效密码

+	`><password>`  将密码添加到用户有效密码列表中
+	`<<password>`  将密码从用户有效密码列表中移除
+	`#<hash>`  将此 SHA-256 哈希值添加到用户的有效密码列表中。该哈希值将与 ACL 用户输入的密码的哈希值进行比较。这将允许用户将此哈希值存储在 acl 配置文件中，而不是存储明文密码。仅接受 SHA-256 哈希值，因为密码的哈希必须由 64 个字符长度的小写的十六进制字符组成。
+	`!<hash>`  从有效密码列表中删除该哈希值
+	`nopass` 删除用户所有密码，并将该用户标记为不需要密码
+	`resetpass` 清除用户可用密码列表的数据，并清除 nopass 状态;账号重新配置前，不许登陆
+	`reset`  重置用户到初始状态。该命令会执行以下操作：resetpass，resetkeys，off，-@ all


5. ACL相关命令

+	`ACL LIST`：用户不存在，检查当前活动的 ACL 列表
+	`ACL SETUSER<username>`：用户不存在，就按默认配置生成用户(用户名区分大小写)
+	`ACL SETUSER <username> <rules>`：用户不存在，则按默认规则创建用户，并增加 <rules> 
	+	用户默认规则为：
		+	处于关闭状态
		+	无法执行任何命令
		+	没有匹配的访问 KEY 的模式
		+	没有有效的密码
+	`ACL GETUSER <username>`：同 ACL LIST 作用类似。ACL GETUSER 用来获取指定用户的 ACL 状态信息
+	`ACL DELUSER <username>`：删除用户
+	`ACL USERS`：查看所有用户
+	`ACL WHOAMI`：查看当前用户
+	`ACL CAT`：显示ACL类别
+	`ACL CAT <类别>`：显示指定类别中所有的 Redis 命令
+	`ACL SAVE`：redis 配置中启用了 aclfile 选项，将当前所有的 ACL 存入 aclfile，覆盖 aclfile 内容
+	`ACL LOAD`：redis 配置中启用了 aclfile 选项，从 acl 文件中加载定义的 acl 规则
+	`ACL GENPASS`：可以使用该命令来生成 Redis 密码（256 bit 的 32 字节的伪随机字符串，并将其转换为 64 字节的字母+数字的字符串）
+	`ACL LOG`：安全日志
+	`ACL HELP`


# 四、持久化数据

Redis 提供了多种不同级别的持久化方式：
+	RDB 持久化可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）。
+	AOF 持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。 AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被追加到文件的末尾。 Redis 还可以在后台对 AOF 文件进行重写（rewrite），使得 AOF 文件的体积不会超出保存数据集状态所需的实际大小。
+	Redis 还可以同时使用 AOF 持久化和 RDB 持久化。 在这种情况下， 当 Redis 重启时， 它会优先使用 AOF 文件来还原数据集， 因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。
+	你甚至可以关闭持久化功能，让数据只在服务器运行时存在

### RDB（快照）

当符合一定条件时，redis中RDB方式会自动将内存中的所有数据生成一份副本并存储在硬盘上，这个过程称为快照。

1. 快照流程：
+	使用fork 函数复制一份当前进程（父进程）的副本（子进程）
+	父进程继续处理客户端的命令，而子进程将内存中的数据写入到硬盘中的临时文件中
+	当临时文件生成结束，临时文件会替代`dump.rdb`文件（过程中，rdb文件保持完整）

> 注：rdb持久化的时候，如果 Redis因为异常退出，就会丢失最后一次快照之后的所有数据

2. 触发快照
+	根据配置进行自动快照
+	`save`保存命令（适合线下）：阻塞主进程，客户端无法连接主进程，同步数据
+	`bgsave`后台保存命令（适合线上）：异步执行，同步数据
+	`flushall`命令：清空整个服务器数据，然后执行自动快照
+	`replication`复制：主从复制起始阶段时进行自动快照


3. 根据配置进行自动快照

```
# 修改 redis.window.conf 文件
# 格式如下：
# save N M
# N 秒内数据集至少有 M 个改动后保存一次数据

save 900 1    # 15分钟内有 1 个键被修改
save 300 10   # 5分钟内有 10 个键被修改
save 60 10000 # 1分钟内有 10000 个键被修改
```

4. 缺点
+	存储数据量大，效率低，每次快照都会读取所有数据
+	使用fork创建子进程，占用内存
+	意外宕机会导致丢失大量数据


### AOF（默认关闭）

AOF 意为Append Only File，会将执行的每一条命令追加到硬盘文件中。当使用redis 存储非临时数据时，一般需要打开aof 持久化来降低进程中止导致的数据丢失。

1. AOF写数据策略（appendfsync）

+	always(每次)：每次写入操作均同步到AOF文件中，数据零误差，性能较低，适合数据要求严格的业务场景
+	everysec(每秒)：每秒将缓冲区中的指令同步到AOF文件中，数据准确性较高，性能较高，建议使用，也是默认配置。
+	no(系统控制)：由操作系统控制每次同步到AOF文件的周期，整体过程不可控

2. `redis.window.conf` 相关配置
```
# 是否启动AOF
appendonly	yes|no # 是否启动AOF

# 写策略
appendfsync	always|everysec|no # AOF写数据策略

# aof文件 名称
appendfilename "appendonly.aof"

# aof文件 路径
dir "E:\\Util\\redis-latest"

# 自动重写触发的条件设置
auto-aof-rewrite-min-size  64mb    #设置触发aof的最小尺寸
auto-aof-rewrite-percentage 100     #达到重写的百分比
```

3. AOF 重写机制

AOF持久化机制也会导致一个问题，那就是AOF文件越来越大。Redis 提供了`bgrewriteaof命令` 压缩持久化文件，该命令会将内存中的数据以命令的方式保存到临时文件中，同时会fork出一条新进程来将文件重写。重写机制不是压缩旧的AOF文件，而是在将内存数据用命令重写一个新的临时文件，然后覆盖旧的AOF文件。

例如：
```
# 一个重写的实例

set name a
set name b
set name c

# aof 重写后，会压缩该命令，变成以下的命令
set name c 
```

相关规则如下：
+	对同一数据的多条写命令合并为一条
+	进程内超时数据忽略
+	无效指令忽略




4. 优缺点

+	优点
	+	AOF可以更好的保护数据不丢失，一般AOF会每隔1秒，通过一个后台线程执行一次fsync操作，最多丢失1秒钟的数据。
	+	没有磁盘寻址开销，写入性能高
	+	重写AOF文件，也不影响客户端读写
	+	合适灾难性恢复
+	缺点
	+	日志文件相对RDB数据文件较大
	+	写（每秒查询率QPS）相对RDB较低

5.  AOF 文件出错

服务器可能在程序正在对 AOF 文件进行写入时停机， 如果停机造成了 AOF 文件出错（corrupt）， 那么 Redis 在重启时会拒绝载入这个 AOF 文件， 从而确保数据的一致性不会被破坏。

当发生这种情况时， 可以用以下方法来修复出错的 AOF 文件：

+	为现有的 AOF 文件创建一个备份。
+	使用 Redis 附带的 redis-check-aof 程序，对原来的 AOF 文件进行修复。
	+	`redis-check-aof --fix`
+	重启 Redis 服务器，等待服务器载入修复后的 AOF 文件，并进行数据恢复

6. RDB 和 AOF 之间的相互作用

在版本号大于等于 2.4 的 Redis 中， BGSAVE 执行的过程中， 不可以执行 BGREWRITEAOF 。 反过来说， 在 BGREWRITEAOF 执行的过程中， 也不可以执行 BGSAVE 。

这可以防止两个 Redis 后台进程同时对磁盘进行大量的 I/O 操作。

如果 BGSAVE 正在执行， 并且用户显示地调用 BGREWRITEAOF 命令， 那么服务器将向用户回复一个 OK 状态， 并告知用户， BGREWRITEAOF 已经被预定执行： 一旦 BGSAVE 执行完毕， BGREWRITEAOF 就会正式开始。

当 Redis 启动时， 如果 RDB 持久化和 AOF 持久化都被打开了， 那么程序会优先使用 AOF 文件来恢复数据集， 因为 AOF 文件所保存的数据通常是最完整的。

***备份 Redis 数据***

Redis 对于数据备份是非常友好的， 因为你可以在服务器运行的时候对 RDB 文件进行复制： RDB 文件一旦被创建， 就不会进行任何修改。 当服务器要创建一个新的 RDB 文件时， 它先将文件的内容保存在一个临时文件里面， 当临时文件写入完毕时， 程序才使用 rename(2) 原子地用临时文件替换原来的 RDB 文件。

***官方建议***
+	创建一个定期任务（cron job）， 每小时将一个 RDB 文件备份到一个文件夹， 并且每天将一个 RDB 文件备份到另一个文件夹。
+	确保快照的备份都带有相应的日期和时间信息， 每次执行定期任务脚本时， 使用 find 命令来删除过期的快照： 比如说， 你可以保留最近 48 小时内的每小时快照， 还可以保留最近一两个月的每日快照。
+	至少每天一次， 将 RDB 备份到你的数据中心之外， 或者至少是备份到你运行 Redis 服务器的物理机器之外。

> 注：服务端并没有马上记录，而是放到AOF写命令刷新缓存区，到一定时间之后将命令同步到AOF文件中。





> [如何保证缓存与数据库的双写一致性？](https://www.jianshu.com/p/2936a5c65e6b)
[Redis缓存数据库安全加固指导（一）](https://www.cnblogs.com/middleware/p/9511389.html)
[Redis入门到精通（十二）——持久化AOF](https://www.cnblogs.com/wangcuican/p/12882930.html)
[Redis安全性](http://www.redis.cn/topics/security.html)
[访问控制列表](https://redis.io/topics/acl)
