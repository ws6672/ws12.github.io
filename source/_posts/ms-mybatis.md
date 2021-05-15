---
title: 面试——mybatis
date: 2020-06-29 09:47:09
tags: [ms]
---

### 什么是Mybatis
`Mybatis`是一个半ORM（对象关系映射）框架，它内部封装了JDBC，开发时只需要关注SQL语句本身，不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程。程序员直接编写原生态sql，可以严格控制sql执行性能，灵活度高.

`Mybatis`通过Xml或注解配置与映射原生信息，将POJO映射成数据库记录，避免繁琐的通过JDBC与数据库交互

### JDBC执行过程

+	获取连接
+	预编译SQL
+	设置参数
+	执行SQL

### `Mybatis`常用标签

+	定义SQL语句(增删改查)
	+	`<select>`
	+	`<insert>`
	+	`<update>`
	+	`<delete>`
+	配置JAVA对象与数据库对象的映射
		`<resultMap>`
+	动态控制
	+	`<if>`
	+	`<foreach>`
	+	`<choose>`、`<when>`、`<otherwise>`
+	格式化输出
	+	`<where>`
	+	`<set>` 配合`<update>`使用
	+	`<trim>`
+	配置关联关系
	+	`<collection>`
	+	`<association>`
+	定义常量及引用
	+	`<sql>`
	+	`<include>`

### 实例1：常用标签




```
-- namespace 定义包名

<mapper namespace="com.eg.springmvc.mapper.StudentMapper" >

	-- 映射POJO到数据库表结构
  <resultMap id="BaseResultMap" type="Student" >
    <id column="ID" property="id" jdbcType="INTEGER" />
    <result column="AGE" property="age" jdbcType="INTEGER" />
    <result column="named" property="named" jdbcType="VARCHAR" />
  </resultMap>
 
	-- 常量定义
  <sql id="Base_Column_List" >
    ID, AGE, named
  </sql>

	-- select（id 方法名，resultMap 查询结果映射，parameterType 参数类型） 查询语句
	--  <include refid="Base_Column_List" /> 包含常量SQL语句
	--	ID = #{id,jdbcType=INTEGER}， #{id} 转换为 "id-value"
	<select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="java.lang.Integer" >
		select 
		<include refid="Base_Column_List" />
		from Student
		where ID = #{id,jdbcType=INTEGER}
	</select>

	-- delete 删除标签
	<delete id="deleteByPrimaryKey" parameterType="java.lang.Integer" >
		delete from Student
			where ID = #{id,jdbcType=INTEGER}
	</delete>
  
 
	-- insert 添加标签
  <insert id="insertSelective" parameterType="Student" useGeneratedKeys="true" keyProperty="id">
    insert into Student
    <trim prefix="(" suffix=")" suffixOverrides="," >
      <if test="id != null" >
        ID,
      </if>
      <if test="age != null" >
        AGE,
      </if>
      <if test="named != null" >
        named,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides="," >
      <if test="id != null" >
        #{id,jdbcType=INTEGER},
      </if>
      <if test="age != null" >
        #{age,jdbcType=INTEGER},
      </if>
      <if test="named != null" >
        #{named,jdbcType=VARCHAR},
      </if>
    </trim>
  </insert>

-- 更新标签 
  <update id="updateByPrimaryKeySelective" parameterType="Student" >
    update Student
    <set>
      <if test="age != null" >
        AGE = #{age,jdbcType=INTEGER},
      </if>
      <if test="named != null" >
        named = #{named,jdbcType=VARCHAR},
      </if>
    </set>
    where ID = #{id,jdbcType=INTEGER}
  </update>

</mapper>
```

### 实例2：`<choose>`

```
<select id="getStudentListChoose" parameterType="Student" resultMap="BaseResultMap">     
    SELECT * from STUDENT WHERE 1=1    
    <where>     
        <choose>     
            <when test="Name!=null and student!='' ">     
                   AND name LIKE CONCAT(CONCAT('%', #{student}),'%')      
            </when>     
            <when test="hobby!= null and hobby!= '' ">     
                    AND hobby = #{hobby}      
            </when>                   
            <otherwise>     
                    AND AGE = 15  
            </otherwise>     
        </choose>     
    </where>     
</select> 
```


### `#{}`和`${}`的区别是什么

`#{}`是预编译处理，`${}`是字符串替换。
Mybatis在处理`#{}`时，会将sql中的#{}替换为?号，调用PreparedStatement的set方法来赋值；

Mybatis在处理`${}`时，就是把${}替换成变量的值。

使用#{}可以有效的防止SQL注入，提高系统安全性。


### 当实体类中的属性名和表中的字段名不一样,如何处理

+	通过<resultMap>来映射字段名和实体类属性名
+	sql语句中定义字段名的别名，让字段名的别名和实体类的属性名一致

###  模糊查询like语句

```

	<select id="selectByName" resultMap="BaseResultMap" parameterType="java.lang.String" >
		select 
		<include refid="Base_Column_List" />
		from Student
		where named LIKE #{named}
	</select>
	
	String name = "%mk%";
	stuDao.selectByName(name);
```


### 如何获取自动生成的(主)键值

```
-- keyProperty，获取属性
  <insert id="insert" parameterType="Student" useGeneratedKeys="true" keyProperty="id">
    insert into Student (ID, AGE, named
      )
    values (#{id,jdbcType=INTEGER}, #{age,jdbcType=INTEGER}, #{named,jdbcType=VARCHAR}
      )
  </insert>
```


### 使用 MyBatis 的 mapper 接口调用时有哪些要求

+	Mapper.xml 文件中的 namespace 即是 mapper 接口的类路径
+	Mapper 接口方法名和 mapper.xml 中定义的每个 sql 的 id 相同
+	Mapper 接口方法的输入参数类型和 mapper.xml 中定义的每个 sql 的 parameterType 的类型相同
+	Mapper 接口方法的输出参数类型和 mapper.xml 中定义的每个 sql 的 resultType 的类型相同


Mybatis 中一级缓存与二级缓存
一级缓存（会话级）: HashMap 本地缓存，其存储作用域为 Session
二级缓存（应用级）：一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为
Mapper(Namespace)，并且可自定义存储源

