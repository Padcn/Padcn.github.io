---
title:  Hibernate基础单表
date: 2017-05-08
tags: Hibernate
---

# Hibernate

### ORM
(Object/Relationship Mapping):对象/关系映射
利用面向对象思想编写的数据库应用程序最终都是把对象信息保存在关系型数据库中，于是要编写很多sql
1.不同数据库使用sql语法不同
2.同一个功能使用的SQL不同。


### Hibernate
Hibernate是java领域的一款开元的ORM框架技术
对jdbc进行了封装。

业务逻辑层通过持久化层（Hibernate）操作数据库

### 简单配置
方便Hibernate使用的eclipse插件 Hibernate Tools for Eclipse Plugin
JBoss tools

搭建步骤：
- 编写Hibernate配置文件
- 创建持久化类
- 创建对象-关系映射文件
- 通过Hibernate API编写访问数据库的代码

导入Hibernate必须的jar包
hibernate-release-*\lib\required
导入jdbc驱动

1. 创建java工程
2. 导入Hibernate必须的jar包 hibernate-release-*\lib\required ，导入jdbc驱动
3. 创建Hibernate的配置文件
    如果没有代码提示功能就需要手工导入 hibernate-release-*\project\hibernate-core\src\main\resources\org\hibernate\hibernate-mapping-3.0.dtd
    在src下创建hibernate添加了插件的话直接可以右键新建hibernate。

    ```xml
    <property name="connection.username">root</property>
    <property name="connection.password"></property>
    <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
    <!--\\\ 是简写 useUnicode防止乱码-->
    <property name="connection.url">jdbc:mysql:///hibernate?useUnicode=true&amp;characterEncoding=UTF-8</property>
    <property name="dialect">org.hibernate.dialect.MySQLDialect</property>

    <!--常用的3个属性-->
    <property name="show_sql">true</property>
    <property name="format_sql">true</property>
    <property name="hbm2ddl.auto">create</property>

    ```
4. 创建持久化层
	
    创建实体化类
    @Student.java
    ```java
    public class Students{
    	private int id;
        private String name;
        
        public int getId() {
            return id;
        }
        public void setId(int id) {
            this.id = id;
        }
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
    }
    ```
    还可以使用注解的方式进行关系映射，这样就不需要创建关系映射文件，可以跳过第5步，使用注解方式关联的实体类如下
    
    @Person.java
    ```java
    package com.pad.entity;

    import javax.persistence.Column;
    import javax.persistence.Entity;
    import javax.persistence.GeneratedValue;
    import javax.persistence.GenerationType;
    import javax.persistence.Id;
    import javax.persistence.Table;

    @Entity
    @Table(name="t_person")
    public class Person {
        private int id;
        private String name;
        private boolean sex;

        @Id
        @GeneratedValue(strategy=GenerationType.AUTO)
        @Column(name="p_id")
        public int getId() {
            return id;
        }
        public void setId(int id) {
            this.id = id;
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
    }
    ```
5. 创建对象关系映射文件
```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE hibernate-mapping PUBLIC 
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
    <hibernate-mapping package="com.pad.entity">

        <class name="Student" table="t_student">
            <id name="id">
                <!-- 交由数据库处理。如果有设置自增则自增，否则由java代码指定 -->
                <generator class="native" />
                <!-- 由代码指定主键 -->
                <!--<generator class="assigned">  -->
                <!--强制为自增-->
                <!--  <generator class="identity">  -->
                <!--指定序列,oracle，db2上-->
                <!--  <generator class="sequence">  -->
                <!--随机生成16进制的唯一ID-->
                <!-- <generator class="hexuid">  -->
            </id>
            <property name="name"></property>
            <property name="age"></property>
        </class>

    </hibernate-mapping>
```

6. 在hibernate.cfg.xml中包含

如果使用xml配置关联则
```xml
	<mapping resource="Student.hbm.xml"/>
```
如果使用注解方式则为
```xml
	<mapping class="com.pad.entity.Person"/>
```


7. 访问数据
	```java
    Configuration config=new Configuration().configure(); //创建配置对象
    ServiceRegistry serviceRegistry=new ServiceRegistryBuilder().applySettings(config.getProperties()).buildServiceRegistry();//创建服务注册对象
    sessionFactory=config.buldSessionFactory(serviceRegistry);//创建会话工厂对象
    session=sessionFactory.openSesson();//打开会话
    transaction=session.beginTransaction();//打开事务
    ```
创建测试用例
@testHibernate.java
```java
    import org.hibernate.Session;
    import org.hibernate.SessionFactory;
    import org.hibernate.Transaction;
    import org.hibernate.cfg.Configuration;
    import org.hibernate.service.ServiceRegistry;
    import org.hibernate.service.ServiceRegistryBuilder;
    import org.junit.After;
    import org.junit.Before;
    import org.junit.Test;
    import com.pad.entity.Student;

    public class testHibernate {
        private SessionFactory sessionFactory;
        private Session session;
        private Transaction transaction;

        @Before
        public void init(){
            Configuration config=new Configuration().configure();
            ServiceRegistry serviceRegistry=new ServiceRegistryBuilder().applySettings(config.getProperties())
            	.buildServiceRegistry();
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

        @Test
        public void testHiber(){
            Student stu=new Student();
            stu.setAge(12);
            stu.setName("aaa");
            session.save(stu);
        }
	}
```
### Hibernate执行流程

![Hibernate执行流程](http://img.mukewang.com/590f0c940001053412800720.jpg "Hibernate执行流程")

### hibernate.cfg.xml常用配置
 - show_sql 是否把Hibernate运行时将sql语句输出到控制台
 - format_sql 对输出到控制台的sql是否进行排版，便于阅读
 - auto 可以帮助由java代码生成数据库脚本，进而生成具体表结构。 属性有create（如果原来表存在则先删除）\|update（在原有表数据上更新）\|create-drop（先创建在删除）\|validate（对原表结构验证，如果现有表结构与原来表结构不同则不会创建表结构，不执行）
 - default_schema 默认数据库
 - dialect 数据库方言，Hibernate可以对特殊数据库进行优化。

### session（会话）
是一个操作数据库的对象
session与connection，是多对一关系，每个session都有一个与之对应的connection，一个connection不同时刻可以供多个session使用。
把对象保存在关系数据库中需要调用session的各种方法，如：save(),update(),delete(),createQuery()等。

- 获取session

    1) openSession
    
    2)getCurrentSession
  
  如果使用getCurrentSession需要在hibernate.cfg.xml文件中配置
  如果是本地事务（jdbc事务）
  ```xml
  <property name="hibernate.current_session_context_class">thread</property>
```
  如果是全局事务（jta事务）
  ```xml
  <property name="hibernate.current_session_context_class">jta</property>
```
两种方式获取session的区别：
1. 使用getCurrentSession在事务提交或者回滚之后会自动关闭，而openSession需要手动关闭。如果使用openSession而没有手动关闭，多次之后会导致连接池溢出。
2. openSession每次创建新的session对象，getCurrentSession使用现有的session对象。

### transaction（事务）
hibernate对数据库的操作都是封装在事务中，并默认是非自动提交的方式，所以session保存对象时，如果不开启事务，并且手动提交事务，对象并不会真保存进数据库。

如果想像jdbc一样自动提交事务，必须调用session对象的doWork()方法，获取jdbc的connection后，设置为自动提交事务模式。（不推荐）
```java
session.doWork(new Work(){
	@Override
    public void execute(Connection connection)throws SQLException{
    	connection.setAutoCommit(true);
    }
})；
session.save(s);
session.flush();//没有这句也不会提交
```

### hbm配置文件常用设置
```xml
<hibernate-mapping
	schema="schemaName"
    catalog="catalogName"
    default-cascade="cascade_style" //级联风格
    default-access="field|property|ClassName" //访问策略
    default-lazy="true|false" //加载策略
    package="packagename"
    />
  <class
  	name="ClassName"
    table="tableName"
    batch-size="N"   //一次抓取N条记录
    where="condition"	//抓取条件
    entity-name="EntityName"  //同一个类映射多张表
    />
    <id
    	name="propertyName"
        type="typename"	//数据类型
        column="column_name" //映射成字段名
        length="length">	//长度
        <generator class="generatorClass"/> //主键生成策略
    </id>
```

![主键生成策略](http://img.mukewang.com/5909a7100001deb812800720.jpg "主键生成策略")


## 单表映射
### 单一主键
- assigned由java应用程序负责生成（手工赋值）
- native 由底层数据库自动生成标识符，如果是MySQL就是increment,如果是Oracle就是sequence，等等。

```xml
<generator class="native" />
```

### 基本类型
![基本类型映射关系](http://img.mukewang.com/58ff54610001e2ab12800720.jpg "基本类型映射关系")

```xml
<property name="birthday" type="timestamp"></property>
```


### 对象类型
![对象类型映射关系](http://img.mukewang.com/58e44fe7000163ec07180404.jpg "对象类型映射关系")
```java
@Test
public void testWriteBlob()throwsException{
 Student s=new Students(1,"ss","m",new Date(),"wudang");
 File f=new File("d:"+File.separator+"filenamebak.jpg");
 InputStream input=new FileInputStream(f);
 Blob image=Hibernate.getLobCreator(session).createBlob(input,input.available());
 s.setPicture(image);
session.save(s);
}

@Test
public void testReadBlob() throws Exception{
	Student s=(Student)session.get(Students.class,1);
    //获得Blob对象
    Blob image=s.getPicture();
    InputStream input=image.getBinaryStream();
    File f=new File("d:"+File.separator+"filename.jpg");
    OutputStream output=new FileOutputStream(f);
    //创建缓冲区
    byte[] buff=new byte[input.available()];
    inpput.read(buff);
    output.write(buff);
    input.close();
    output.close(); 
}
```


### 组件属性

实体类中的某个属性属于用户自定义的类的对象。


```xml
<component name="address" class="com.pad.entity.Address">
<property name="postcode" column="POSTCODE"/>
<property name="phone" column="PHONE"/>
<property name="address" column="ADDRESS"/>
</component>
```
测试用例没有什么变化


### 单表CRUD实例
- save
- update
- delete
- get/load

get/load区别

1. get在调用后立即向数据库发出sql，返回持久化对象
    load方法会在调用后返回一个代理对象，该对象只是保存了实体类的id，直到使用对象的非主键属性时才会发出sql

2. 查询数据库中不存在的数据时，get返回null，
    load方法抛出异常org.hibernate.ObjectNotFoundException



### Hibernate持久化对象的三种状态
http://blog.csdn.net/lovesummerforever/article/details/19171571

![Hibernate持久化对象的三种状态](http://images2015.cnblogs.com/blog/966412/201611/966412-20161105234424768-1479445986.png "Hibernate持久化对象的三种状态")

### Hibernate注解相关
- Hibernate4注解方法(全) http://blog.csdn.net/houfeng30920/article/details/51439958
- hibernate annotation注解方式来处理映射关系 http://www.cnblogs.com/xiaoluo501395377/p/3374955.html





