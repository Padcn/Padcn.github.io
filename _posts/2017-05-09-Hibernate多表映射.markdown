---
title:  Hibernate映射
date: 2017-05-09
tags: Hibernate
---

# Hibernate映射1-1，1-N，N-N

一般由外键所在的实体类去维护
### 注解方式
#### 1-1映射

@Person.java
```java
package com.pad.entity;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.OneToOne;
import javax.persistence.Table;

@Entity
@Table(name="t_person")
public class Person {
	private int pid;
	private String name;
	private boolean sex;
	private IdCard idcard;
	
	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)
	@Column(name="p_id")
	public int getPid() {
		return pid;
	}
	public void setPid(int pid) {
		this.pid = pid;
	}
	
	@Column(name="p_name")
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	@Column(name="p_sex")
	public boolean isSex() {
		return sex;
	}
	public void setSex(boolean sex) {
		this.sex = sex;
	}
	//表示由IdCard中的p进行维护
	@OneToOne(mappedBy="p")
	public IdCard getIdcard() {
		return idcard;
	}
	public void setIdcard(IdCard idcard) {
		this.idcard = idcard;
	}
	
}

```

@IdCard.java
```java
package com.pad.entity;

import java.util.Date;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.OneToOne;

@Entity(name="idcard")
public class IdCard {
	private int C_Id;
	private Date birth;
	private Person p;

	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)
	public int getC_Id() {
		return C_Id;
	}

	public void setC_Id(int cId) {
		C_Id = cId;
	}
	@Column(name="birth")
	public Date getBirth() {
		return birth;
	}
    
	public void setBirth(Date birth) {
		this.birth = birth;
	}

	//维护方，使用外键p_id进行关联
	@OneToOne
	@JoinColumn(name="p_id")
	public Person getP() {
		return p;
	}

	public void setP(Person p) {
		this.p = p;
	}
	
}
```
测试代码
@testHibernate.java
```java
import java.util.Date;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;
import org.hibernate.service.ServiceRegistry;
import org.hibernate.service.ServiceRegistryBuilder;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import com.pad.entity.IdCard;
import com.pad.entity.Person;


public class testHibernate {
	private SessionFactory sessionFactory;
	private Session session;
	private Transaction transaction;
	
	
	@SuppressWarnings("deprecation")
	@Before
	public void init(){
		Configuration config=new Configuration().configure();
		ServiceRegistry serviceRegistry=new ServiceRegistryBuilder()
			.applySettings(config.getProperties()).buildServiceRegistry();
		sessionFactory=config.buildSessionFactory(serviceRegistry);
		session=sessionFactory.openSession();
		transaction=session.beginTransaction();
	}
	
	@After
	public void destory(){
		transaction.commit();
		session.close();
		sessionFactory.close();
	}
	
	@Test public void testOTO(){
		Person p=new Person();
		p=(Person)session.load(Person.class, 2);//第二个参数为person表中的主键
		System.out.println(p.getIdcard().getBirth());
	}
}

```

#### 1-N
@Course.java
```java
package com.pad.entity;

import java.util.Set;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.OneToMany;

@Entity(name="course")
public class Course {
	private int c_id;
	private String c_name;
	private Set<Student> stu;
	
	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)
	public int getC_id() {
		return c_id;
	}
	public void setC_id(int cId) {
		c_id = cId;
	}
	public String getC_name() {
		return c_name;
	}
	public void setC_name(String cName) {
		c_name = cName;
	}
	@OneToMany(mappedBy="course")
	public Set<Student> getStu() {
		return stu;
	}
	public void setStu(Set<Student> stu) {
		this.stu = stu;
	}
}
```
@Student.java
```java
package com.pad.entity;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.ManyToOne;

@Entity(name="student")
public class Student {
	private int s_id;
	private String s_name;
	private int s_age;
	private Course course;
	
	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)
	public int getS_id() {
		return s_id;
	}
	public void setS_id(int sId) {
		s_id = sId;
	}
	public String getS_name() {
		return s_name;
	}
	public void setS_name(String sName) {
		s_name = sName;
	}
	public int getS_age() {
		return s_age;
	}
	public void setS_age(int sAge) {
		s_age = sAge;
	}
	
	@ManyToOne
	@JoinColumn(name="c_id")
	public Course getCourse() {
		return course;
	}
	public void setCourse(Course course) {
		this.course = course;
	}
	@Override
	public String toString() {
		return "Student [course=" + course + ", s_age=" + s_age + ", s_id="
				+ s_id + ", s_name=" + s_name + "]";
	}
}
```
测试代码：
```java
	//在运行下面Test之前插入数据，先运行testOTM()
    //在这里如果把student放在循环之外new会造成s_id是一样的，所以多个save只是会更新第一条记录。
    public void testOTM(){
		Course c=new Course();
		c.setC_name("1A");
		session.save(c);
		
		int count=5;
		while(count-->0){
			Student s=new Student();
			s.setCourse(c);
			s.setS_name("xiaom"+count);
			s.setS_age(count);
			session.save(s);
		}
	}
	@Test
	public void testOTMResult(){
		Course c=(Course)session.load(Course.class,1);
		System.out.println(c); //当把下面for循环注释掉之后控制台输出的sql语句只select了course表字段，并没有查询student表，验证了懒加载。
		for(Student s:c.getStu()){
			System.out.println(s);
		}
	}
```
#### N-N
@Teacher.java
```java
package com.pad.entity;

import java.util.Set;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.JoinTable;
import javax.persistence.ManyToMany;

@Entity(name="teacher")
public class Teacher {
	private int t_id;
	private int t_name;
	private Set<Course> course;
	
	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)
	public int getT_id() {
		return t_id;
	}
	public void setT_id(int tId) {
		t_id = tId;
	}
	public int getT_name() {
		return t_name;
	}
	public void setT_name(int tName) {
		t_name = tName;
	}
	
	@ManyToMany
	@JoinTable(
			name="t_id_c_id",
			joinColumns=@JoinColumn(name="t_id"),
			inverseJoinColumns=@JoinColumn(name="c_id")
			)
	public Set<Course> getCourse() {
		return course;
	}
	public void setCourse(Set<Course> course) {
		this.course = course;
	}
	
}

```
@Course.java
```java
package com.pad.entity;

import java.util.Set;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.ManyToMany;
import javax.persistence.OneToMany;

@Entity(name="course")
public class Course {
	private int c_id;
	private String c_name;
	private Set<Student> stu;
	private Set<Teacher> t;
	
	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)
	public int getC_id() {
		return c_id;
	}
	public void setC_id(int cId) {
		c_id = cId;
	}
	public String getC_name() {
		return c_name;
	}
	public void setC_name(String cName) {
		c_name = cName;
	}
	@OneToMany(mappedBy="course")
	public Set<Student> getStu() {
		return stu;
	}
	public void setStu(Set<Student> stu) {
		this.stu = stu;
	}
	
	@ManyToMany(mappedBy="course")
	public Set<Teacher> getT() {
		return t;
	}
	public void setT(Set<Teacher> t) {
		this.t = t;
	}
}

```
运行之后数据库表已经创建，这里如果不把对象创建为set，而只是普通的Teacher运行时会抛异常：
     org.hibernate.AnnotationException: Illegal attempt to map a non collection as a @OneToMany, @ManyToMany or @CollectionOfElements: com.pad.entity.Course.t
	at org.hibernate.cfg.annotations.CollectionBinder.getCollectionBinder(CollectionBinder.java:331)
    ....




### hbm方式

一对多 一方
```xml
	<!--set中设置inverse属性为true则由多方维护关系-->
    <set name="students" table="student" inverse="true">
        <key column="gid"></key>
        <one-to-many class="com.imooc.entity.Student"/>
    </set>
    
```
inverse属性用于指定关联关系的维护，有利于优化性能。

一对多 多方
```xml
<many-to-one name="grade" class="com.imooc.entity.Grade" column="gid"></many-to-one>
```

当cascade属性不为none时，Hibernate会自动持久化所关联对象。
cascade属性的设置会带来性能上的变动，需谨慎设置

		
    
  |  属性值 	 | 含义和作用
  |  --- 		|  ---
  | all	 		| 对所有操作进行级联操作
  |  save-update | 执行保存和更新操作时进行级联操作
  | delete		| 执行删除操作时进行级联操作
  | none		| 对所有操作不进行级联操作

```xml
	<!--set中设置inverse属性为true则由多方维护关系-->
    <set name="students" table="student" inverse="true" cascade="save-update">
        <key column="gid"></key>
        <one-to-many class="com.imooc.entity.Student"/>
    </set>
```
设置之后班级的save会级联保存学生的信息，不需要手动save学生。

如果学生对应班级没有，需要级联班级，可以在学生那边增加cascade属性 cascade="all"，这样就能级联增加班级。

### 总结
#### 实现单向一对多：
	在one方的实体中添加保存many方的集合
    在one方的配置文件中添加<one-to-many>配置
#### 实现单向多对一：
	 在many方的实体中添加one方的引用
     在many方的配置文件中添加<many-to-one>配置
    
### 逆向生成Hibernate实体类和配置文件
1. Myeclipse添加数据库连接
	右上角增加视图-打开数据库视图，新建连接
2. 为工程添加hibernate
    右键项目->myeclipse->add hibernate capicity->选择之前新建的数据库连接->create SessionFactory class(可以方便获取sesson)
3. 创建包，切换到数据库视图->选中表右键->Hibernate Reverse Engineering->指定位置->选中Create POJO<> DB Table mapping information和java Data Object（去掉创建抽象类的勾）->选择主键生成策略->勾选2个include

















