---
title: MyBastis环境搭建以及单表CURD
date: 2017-05-12
tags: MyBatis
---

# MyBatis
MyBatis 是支持普通 SQL查询，存储过程和高级映射的优秀持久层框架。MyBatis 消除了几乎所有的JDBC代码和参数的手工设置以及结果集的检索。MyBatis 使用简单的 XML或注解用于配置和原始映射，将接口和 Java 的POJOs（Plain Old Java Objects，普通的 Java对象）映射成数据库中的记录。

## 1. 相关jar包下载并且导入项目

https://github.com/mybatis/mybatis-3/releases

## 2. 配置核心配置文件
> 如果下载的是source code可以在下面目录找到核心配置文件模板：<br>
> mybatis-*/src/test/java/org/apache/ibatis/submitted/complex_property/Configuration.xml

@conf.xml(这里文件名可以自己取，主要是后续自己通过方法加载)
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <!-- 配置数据库连接信息 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mybatisDB" />
                <property name="username" value="root" />
                <property name="password" value="123" />
            </dataSource>
        </environment>
    </environments>
    <mappers>
    	 <!--第一种是通过xml进行映射加载的xml，第二种是通过注解方式映射加载的是接口-->
	     <mapper resource="com/pad/mapping/cardMapper.xml"/>
	     <mapper class="com.pad.mapping.IUserMapper"/>
    </mappers>
</configuration>
```

## 3. 在数据库中建立对应表并建立实体类
```sql
create table user(
u_id int primary key auto_increment,
u_name varchar(20),
u_age int
);
```

@User.java
```java
public class User {
	private int u_id;
	private String u_name;
	private int u_age;
	
	
	public int getU_id() {
		return u_id;
	}
	public void setU_id(int uId) {
		u_id = uId;
	}
	public String getU_name() {
		return u_name;
	}
	public void setU_name(String uName) {
		u_name = uName;
	}
	public int getU_age() {
		return u_age;
	}
	public void setU_age(int uAge) {
		u_age = uAge;
	}
}
```

## 4. 建立映射关系
这里有2种方式注解方式和xml配置方式，因为下面将使用注解方式这里就使用xml配置方式：

@userMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--命名空间可以自定义，但是习惯上是包名+映射文件名，可以保证唯一-->
 <mapper namespace="com.pad.mapping.userMapper">
 	<!--依次为指定id到时候调用，指定参数类型（这里u_id为int所以是int），指定返回类型，这里返回的是User类-->
 	<select id="getUser" parameterType="int" resultType="com.pad.entity.User">
    	select u_id,u_name,u_age from user where u_id=#{u_id};
    </select>
    <insert id="addUser" parameterType="com.pad.entity.User" resultType="int">
    	insert into user(u_name,u_age) values(#{u_name},#{u_age});
    </insert>
    <delete id="deleteUser" parameterType="int" resultType="int">
    	delete from user where u_id=#{u_id}
    </delete>
    <update id="updateUser" parameterType="com.pad.entity.User" resultType="int">
    	update user set u_name=#{u_name},u_age=#{u_age} where u_id=#{u_id}
    </update>
 </mapper>
```
> 如果觉得写类型时打全路径比较麻烦可以考虑取别名，在核心配置文件conf.xml的configuration标签范围内使用
```
 <typeAliases>  
    <typeAlias type="com.pad.entity.User" alias="_User"/>
    <!--
    	如需为包下面所有设置别名则如下，默认会为这个包下所有设置别名，别名即类名
    	<package name="com.pad.entity"/>
    -->
</typeAliases>  
 ```
> 之后就可以愉快的直接使用 _User 替代 com.pad.entity.User 了

写完配置文件之后再conf.xml中引入
```xml
<mappers>
	<mapper resource="com/pad/mapping/userMapper.xml" />
</mappers>
```

## 5. 编写测试代码

MyBatis 向DAO层提供sqlsession对象
SqlSession作用
1. 向SQL语句传入参数
2. 执行SQL语句
3. 获取执行SQL语句的结果
4. 事务的控制

获取SqlSession
1.通过配置文件获取数据库连接相关信息
2.通过配置文件构建SqlSessionFactory
3.通过SqlSessionFactory打开数据库会话

```java
/*获取SqlSession函数
 *抛出IOException，在调用时try...catch 在finally中close()
 */
public SqlSession getSqlSession() throws IOException{
    Reader reader=Resources.getResourceAsReader("com/imooc/confg/xxx.xml");
    SqlSessionFactory sqlSessionFactory=new SqlSessionFactoryBuilder().build(reader);
    SqlSession sqlSession=sqlSessionFactory.openSession();

    return sqlSession;
}
```
@TestMB.java
```java
	@Before
	public void init() throws IOException{
    	//加载配置文件，build方法可以接收Reader或者InputStream，用以产生XMLConfigBuilder对象
		Reader reader=Resources.getResourceAsReader("conf.xml");
		SqlSessionFactory sqlSessionFactory=new SqlSessionFactoryBuilder().build(reader);
		session=sqlSessionFactory.openSession();//可以接收一个boolean值，如为true则自动commit
	}
	
	@After
	public void destory(){
		session.commit();
		session.close();
	}
    
	@Test
	public void testAddU(){
    	String statement="com.pad.mapping.addUser";
		User u=session.selectOne(statement,1);
		u.setU_name("xiaoming");
		u.setU_age(18);
		user.addUser(u);
	}
```


# 注解+接口方式实现CURD
实体类@User.java
```java
public class User {
	private int u_id;
	private String u_name;
	private int u_age;
	
	
	public int getU_id() {
		return u_id;
	}
	public void setU_id(int uId) {
		u_id = uId;
	}
	public String getU_name() {
		return u_name;
	}
	public void setU_name(String uName) {
		u_name = uName;
	}
	public int getU_age() {
		return u_age;
	}
	public void setU_age(int uAge) {
		u_age = uAge;
	}
}
```

接口 @IUserMapper.java
```java
import java.util.List;
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;
import com.pad.entity.User;

public interface IUserMapper {

	@Insert("insert into user(u_name,u_age) values(#{u_name},#{u_age})")
	int addUser(User u);
	
	@Delete("delete from user where u_id=#{u_id}")
	int deleteUser(User u);
	
	@Update("update user set u_name=#{u_name},u_age=#{u_age} where u_id=#{u_id}")
	int updateUser(User u);
	
	@Select("select u_id,u_name,u_age from user where u_id=#{u_id}")
	User selectUser(int u_id);
	
	@Select("select u_id,u_name,u_age from user")
	List<User> selectUsers();
}

```

测试用例@TestMB.java
```java
import java.io.IOException;
import java.io.Reader;
import java.util.Date;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import com.pad.entity.Card;
import com.pad.entity.User;
import com.pad.mapping.IUserMapper;

public class TestMB {
	private SqlSession session;
	
	@Before
	public void init() throws IOException{
//		String res="mybatis.xml";
//		InputStream is=TestMB.class.getClassLoader().getResourceAsStream(res);
//		SqlSessionFactory sessionfactory=new SqlSessionFactoryBuilder().build(is);
//		SqlSession session=sessionfactory.openSession();
		
		Reader reader=Resources.getResourceAsReader("conf.xml");
		SqlSessionFactory sqlSessionFactory=new SqlSessionFactoryBuilder().build(reader);
		session=sqlSessionFactory.openSession();
	}
	
	@After
	public void destory(){
		session.commit();
		session.close();
	}
	
	
	@Test
	public void testAddU(){
		IUserMapper user=(IUserMapper)session.getMapper(IUserMapper.class);
		User u=new User();
		u.setU_name("xiaoming");
		u.setU_age(18);
		user.addUser(u);
	}
	
	@Test
	public void testDeleteU(){
		IUserMapper user=(IUserMapper)session.getMapper(IUserMapper.class);
		User u=new User();
		u.setU_id(1);
		u.setU_name("xiaoming");
		u.setU_age(18);
		System.out.println(user.deleteUser(u));
	}
	@Test
	public void testUpdateU(){
		IUserMapper user=(IUserMapper)session.getMapper(IUserMapper.class);
		User u=new User();
		u.setU_id(2);
		u.setU_name("烫烫烫");
		u.setU_age(999);
		System.out.println(user.updateUser(u));
	}
	
	@Test
	public void testSelectU(){
		IUserMapper user=(IUserMapper)session.getMapper(IUserMapper.class);
		System.out.println(user.selectUser(2));
	}
	
	@Test
	public void testSelectUs(){
		IUserMapper user=(IUserMapper)session.getMapper(IUserMapper.class);
		System.out.println(user.selectUsers());
	}
}
```
conf.xml引入映射
```xml
<mapper class="com.pad.mapping.IUserMapper"/>
```

# 因为字段名和属性不相同导致查询不出结果问题
在一些情况下表中字段名和实体类中属性名不一致，如User表字段为u_id,u_name， User类属性为 uid,uname。在这种情况下直接使用select* 是查询不出结果的。有以下两种方式解决。
## 1. 为查询字段取别名

```sql
select u_id uid,u_name uname from user;
```

## 2. 为字段和属性建立映射
建立之后就可以使用select \*来查找(当然select \* 性能上比较差，还是手写字段名把)
```xml
<resultMap type="com.pad.entity.User" id="userResultMap">
	<id property="uid" column="u_id"/>
    <result property="uname" column="u_name"/>
</resultMap>
```

# 一些配置方面优化

>conf.xml对于标签的顺序是有规定的，以下是当顺序出错时提示的格式信息： (properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,reflectorFactory?,plugins?,environments?,databaseIdProvider?,mappers?)

## 提取分离数据库配置信息
可以把核心配置文件中的数据库相关信息提取出来，放入一个.properties文件中例如db.properties文件
@db.properties
```
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatisdb
name=root
password=123
```
在conf.xml中引入
```xml
<properties resource="db.properties"/>
<environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <!-- 配置数据库连接信息 -->
            <dataSource type="POOLED">
                <property name="driver" value="${driver}" />
                <property name="url" value="${url}" />
                <property name="username" value="${name}" />
                <property name="password" value="${password}" />
            </dataSource>
        </environment>
    </environments>
```

> 关于在实体类映射文件中使用的sql #{属性名}与\${属性名}的区别，#{}方式比较安全，可以防止sql注入，估计调的是peraparedStatement，第二种可以被sql注入，估计是Statement。














