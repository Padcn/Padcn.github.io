---
title: MyBatis多表关联以及调用存储过程
date: 2017-05-12
tags: MyBatis
---

# MyBatis关联表
> 参考 http://www.cnblogs.com/xdp-gacl/p/4264440.html

## 一对一
MyBatis中在关联映射配置文件中使用association标签来解决一对一的关联查询

建表语句
```sql
create table people(
p_id int primary key auto_increment,
p_name varchar(50),
p_info varchar(50)
);

create table idcard(
ic_id int primary key auto_increment,
p_id int,
ic_number varchar(50),
foreign key(p_id) references people(p_id)
);
insert into people values(null,'aaa','no describe');
insert into people values(null,'bbb','no describe');
insert into people values(null,'ccc','no describe');

insert into idcard values(null,1,'678686');
insert into idcard values(null,2,'6789792');
insert into idcard values(null,3,'9635224');
```
实体类people包含3个属性p_id,p_name,p_info和一个toString方法
@Idcard.java
```java
public class Idcard {
	private int ic_id;
	private String ic_number;
	private People people;
	
	public int getIc_id() {
		return ic_id;
	}
	public void setIc_id(int icId) {
		ic_id = icId;
	}
	public String getIc_number() {
		return ic_number;
	}
	public void setIc_number(String icNumber) {
		ic_number = icNumber;
	}
	public People getPeople() {
		return people;
	}
	public void setPeople(People people) {
		this.people = people;
	}
	@Override
	public String toString() {
		return "Idcard [ic_id=" + ic_id + ", ic_number=" + ic_number
				+ ", people=" + people + "]";
	}
}
```


映射文件@idcardMapper.xml
```xml
	<select id="getICard" parameterType="int" resultMap="IdcardResultMap">
         select * from idcard i,people p where i.p_id=p.p_id and i.ic_id=#{ic_id}
     </select>
     <resultMap type="com.pad.entity.Idcard" id="IdcardResultMap">
     	<id property="ic_id" column="ic_id"/>
     	<result property="ic_number" column="ic_number"/>
     	<association property="people" javaType="com.pad.entity.People">
     		<id property="p_id" column="p_id"/>
     		<result property="p_name" column="p_name"/>
     		<result property="p_info" column="p_info"/>
     	</association>
     </resultMap>
```
在conf.xml中引入关联
```xml
<mapper resource="com/pad/mapping/idcardMapper.xml"/>
```
测试代码
```java
@Test
public void testMuti(){
	String statement="com.pad.mapping.idcardMapper.getICard";
    //查询ic_id为1的数据,这里session在@Before中已经获取
	Idcard ic=session.selectOne(statement, 1);
	System.out.println(ic);
}
```

以上是嵌套结果方式，另一种为嵌套查询，是分别用2个select查询后进行关联。

关联xml如下：
```xml
 	<select id="getICard2" parameterType="int" resultMap="IdcardResultMap2">
         select * from idcard i,people p where i.p_id=p.p_id and i.ic_id=#{ic_id}
     </select>
     <select id="getPeople2" parameterType="int" resultType="com.pad.entity.People">
         select p_id,p_name,p_info from people where p_id=#{p_id}
     </select>
     <resultMap type="com.pad.entity.Idcard" id="IdcardResultMap2">
     	<id property="ic_id" column="ic_id"/>
     	<result property="ic_number" column="ic_number"/>
     	<association property="people" column="p_id" select="getPeople2" />
     </resultMap>
```

## 一对多关联
MyBatis中使用collection标签进行一对多的关联，ofType属性指的是元素的对象类型。
实体类People中加入一个属性List&lt;Items&gt; items.
```xml
	<select id="getItems" parameterType="int" resultMap="itemsResultMap">
         select * from items,people where  items.p_id=people.p_id and people.p_id=#{p_id};
     </select>
     <resultMap type="com.pad.entity.People" id="itemsResultMap">
     	<id property="p_id" column="p_id"/>
     	<result property="p_name" column="p_name"/>
     	<result property="p_info" column="p_info"/>
     	<collection property="items" ofType="com.pad.entity.Items">
     		<id property="it_id" column="it_id"/>
     		<result property="it_name" column="it_name"/>
     		<result property="it_info" column="it_info"/>
     	</collection>
     </resultMap>
```
嵌套查询方式也是类似配置

```xml
<!--方式二：嵌套查询：通过执行另外一个SQL映射语句来返回预期的复杂类型-->
 	<select id="getItems2" parameterType="int" resultMap="itemsResultMap2">
         select * from people where p_id=#{p_id};
     </select>
     <select id="getItem2" parameterType="int" resultType="com.pad.entity.Items">
         select * from items where p_id=#{p_id};
     </select>
     
     <resultMap type="com.pad.entity.People" id="itemsResultMap2">
     	<id property="p_id" column="p_id"/>
     	<result property="p_name" column="p_name"/>
     	<result property="p_info" column="p_info"/>
     	<collection property="items" column="p_id" ofType="com.pad.entity.Items" select="getItem2"/>
     </resultMap>
```

> 有一点需要注意的是使用了resultMap之后再resultMap中未声明的属性将不会获取，无论字段名和属性名是否一致。


## 使用注解方式进行关联
> http://blog.csdn.net/luanlouis/article/details/35780175

注解中使用@Result进行达到xml中的resultMap效果，使用@One表示一方，使用@Many表示多方。

```java
public interface StudentMapper  
{  
    @Select("SELECT ADDR_ID AS ADDRID, STREET, CITY, STATE, ZIP, COUNTRY  
            FROM ADDRESSES WHERE ADDR_ID=#{id}")  
    Address findAddressById(int id);  
    @Select("SELECT * FROM STUDENTS WHERE STUD_ID=#{studId} ")  
    @Results(  
    {  
        @Result(id = true, column = "stud_id", property = "studId"),  
        @Result(column = "name", property = "name"),  
        @Result(column = "email", property = "email"),  
        @Result(property = "address", column = "addr_id",  
        one = @One(select = "com.mybatis3.mappers.StudentMapper.  
        findAddressById"))  
    })  
    Student selectStudentWithAddress(int studId);  
}  
```

```java
public interface TutorMapper  
{  
    @Select("select addr_id as addrId, street, city, state, zip,  
            country from addresses where addr_id=#{id}")  
    Address findAddressById(int id);  
    @Select("select * from courses where tutor_id=#{tutorId}")  
    @Results(  
    {  
        @Result(id = true, column = "course_id", property = "courseId"),  
        @Result(column = "name", property = "name"),  
        @Result(column = "description", property = "description"),  
        @Result(column = "start_date" property = "startDate"),  
        @Result(column = "end_date" property = "endDate")  
    })  
    List<Course> findCoursesByTutorId(int tutorId);  
    @Select("SELECT tutor_id, name as tutor_name, email, addr_id  
            FROM tutors where tutor_id=#{tutorId}")  
    @Results(  
    {  
        @Result(id = true, column = "tutor_id", property = "tutorId"),  
        @Result(column = "tutor_name", property = "name"),  
        @Result(column = "email", property = "email"),  
        @Result(property = "address", column = "addr_id",  
        one = @One(select = " com.mybatis3.  
        mappers.TutorMapper.findAddressById")),  
        @Result(property = "courses", column = "tutor_id",  
        many = @Many(select = "com.mybatis3.mappers.TutorMapper.  
        findCoursesByTutorId"))  
    })  
    Tutor findTutorById(int tutorId);  
}  
```

# 调用存储过程

调用存储过程的方式对于使用时和调用select一样，主要是映射文件方面的配置,statementType="CALLABLE"，如果没有对应的parameterMap则需要加上对应的resultType.其实可以理解为一个特殊的select.

相关配置：
```xml
<select id="getUserCount" parameterMap="getUserCountMap" statementType="CALLABLE">
    CALL mybatis.ges_user_count(?,?)
</select>

<!--如果有参数，参数的设置，如果没有则需要在select标签中加上resultType-->
<parameterMap type="java.util.Map" id="getUserCountMap">
    <parameter property="sexid" mode="IN" jdbcType="INTEGER"/>
    <parameter property="usercount" mode="OUT" jdbcType="INTEGER"/>
</parameterMap>
```

