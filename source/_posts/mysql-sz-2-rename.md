---
title: mysql 实战（二）数据库重命名
date: 2019-12-07 11:21:27
tags: [mysql]
---

# 一、MYDQL重命名

近几日在折腾mysql数据库，由于导入错误，需要重新修改数据库的名称，但百度一下眉头一皱，发现事情并不简单。在网上查到可以用rename命令迁移：
`rename table test.A to test2.A;`

使用rename命令，迁移后的表名必须确定，在迁移上千个表的时候，是不可能每个都手写的。我们可以通过 `show tables;` 查出该库的所有表，把查出结果导出后，放置到txt文件中，可以通过类似以下的代码生成移库代码：

```
// 修改oldDBName、newDBName、url 即可
    @org.junit.Test
    public void printRenameSql ()  {
        FileInputStream fis ,fis2;
        String oldDBName = "csg",
                newDBName="csg_test",
                url = "D:\\b.txt";
                
        try {
            fis = new FileInputStream(new File(url));
            InputStreamReader isr = new InputStreamReader(fis, "UTF-8");
            BufferedReader br = new BufferedReader(isr);
            String line = "";
            List<String> list = new ArrayList<String>();
            while ((line = br.readLine()) != null) {
                // 如果 t x t文件里的路径 不包含---字符串       这里是对里面的内容进行一个筛选
                if (line.lastIndexOf("---") < 0) { 
                    list.add(line);
                }
            }
            int i=0;
            for (String string: list) {
                list.set(i++, "rename table "+oldDBName+"."+string+" to "+newDBName+"."+string+";");
            }
            for (String string: list) {
                System.out.println(string);
            }
            br.close();
            isr.close();
            fis.close();
        }  catch (Exception e) {
            e.printStackTrace();
        }   
    }
//在命令行会输出类似【】的语句，复制执行即可。
```

# 二、在导入数据时遇到的几个问题

### 1. 【mysql 8】 caching_sha2_password，密码格式限制

mysql 8 的新特性默认使用【caching_sha2_password】插件，即用户客户端需要通过sha2插件连接服务端。但在Workbench和navicat中并没有对应的插件，所以我们可以通过修改用户的加密方式来连接mysql8.

```
-- 查看插件
select host, user, plugin from user;
-- 修改加密方式为 `mysql_native_password`，密码为`csg`
`ALTER USER 'csg'@'%' IDENTIFIED WITH mysql_native_password BY 'csg';`
```


### 2. 允许远程连接
root账号创建后，默认只能够本地连接，但我们还是可以通过修改user表来实现远程连接。

```
-- 登陆
mysql -uroot -p
mysql> use mysql;
Database changed

```

```
--查询
mysql> select host, user, plugin from user;
+-----------+------------------+-----------------------+
| host      | user             | plugin                |
+-----------+------------------+-----------------------+
| %         | csg              | mysql_native_password |
| localhost | root             | mysql_native_password |
| localhost | mysql.infoschema | caching_sha2_password |
| localhost | mysql.session    | caching_sha2_password |
| localhost | mysql.sys        | caching_sha2_password |
+-----------+------------------+-----------------------+

-- 可见，root账号的host为 localhost

-- 修改权限
update user set host='%' where user = 'root';
-- 刷新
FLUSH PRIVILEGES;
```

### 3. 修改user密码

```
update user set authentication_string='' where user='csg'; -- 5.8 需为空
ALTER USER 'csg'@'%' IDENTIFIED WITH mysql_native_password BY 'mysql@csg';
FLUSH PRIVILEGES;
```

### 4. Access denied for user 'root'@'localhost'. Account is locked
在修改完root账号的host后，发现管理员账号被锁住。不急，我们可以通过无密码的方式登陆，然后解锁即可。

启动无密码访问的方式：
```
net stop mysql[mysql服务名按实修改];
cd D:\software\mysql\bin
d:
-- 无密码登陆
mysqld.exe --defaults-file="D:\software\mysql\my.ini" --skip-grant-tables --shared-memory
```

登陆按正常登陆即可，只是无需密码。在配置远程登陆的时候，执行`select host,user, lock_tables_priv,plugin from user;`看到 有【lock_tables_priv】字段，以为是锁定账号的字段。但其实不是的，这是授权锁表的字段。

使用拥有管理员权限的账号执行下列语句即可：
```
ALTER USER 'root'@'%' ACCOUNT UNLOCK;
FLUSH PRIVILEGES;
-- 'root'@'%' 对应user表中的 user、host字段
```

# 三、授权
### 1. 基本权限

|权限|范围|
|:--|:--|
|select|表数据|
|update|表数据|
|update|表数据|
|delete|表数据|
|create、alter、drop|表|
|references |表外键|
|create temporary tables|临时表|
|index |索引|
|create/show view |创建、查看视图|

*增删改查权限实例*
```
-- grant 权限 on 数据库对象 to 用户

grant select on csg.* to root@'%'
grant insert on csg.* to root@'%'
grant update on csg.* to root@'%'
grant delete on csg.* to root@'%'

grant select,insert,update,delete on csg.* to root@'%';
```

### 2.dba

```
grant all privileges on testdb to dba@'localhost'
grant all on *.* to dba@'localhost'

-- 取消权限
revoke all on *.* from dba@localhost;
-- 刷新权限
FLUSH PRIVILEGES;
```
>	```
mysql授权表共有5个表：user、db、host、tables_priv和columns_priv。

授权表的内容有如下用途：
user表
user表列出可以连接服务器的用户及其口令，并且它指定他们有哪种全局（超级用户）权限。在user表启用的任何权限均是全局权限，并适用于所有数据库。例如，如果你启用了DELETE权限，在这里列出的用户可以从任何表中删除记录，所以在你这样做之前要认真考虑。

db表
db表列出数据库，而用户有权限访问它们。在这里指定的权限适用于一个数据库中的所有表。

host表
host表与db表结合使用在一个较好层次上控制特定主机对数据库的访问权限，这可能比单独使用db好些。这个表不受GRANT和REVOKE语句的影响，所以，你可能发觉你根本不是用它。

tables_priv表
tables_priv表指定表级权限，在这里指定的一个权限适用于一个表的所有列。

columns_priv表
columns_priv表指定列级权限。这里指定的权限适用于一个表的特定列。
```

# 四、参考文章

> [mysql grant授权](https://www.cnblogs.com/bethal/p/5512755.html)