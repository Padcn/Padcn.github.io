---
title:  SpringAOP
date: 2017-05-16 
tags: Spring
---
# AOP

主要功能：日志记录，性能统计，安全控制，事务处理，异常处理等

实现方式：
- 预编译 AspcetJ
- 运行期动态代理（JDK动态代理、CGLib动态代理）
	SpringAOP JBoss
    
 ![](http://img.mukewang.com/590fbedf0001f41c12800720.jpg)
 <hr/>
 ![](http://img.mukewang.com/5913c9400001292012800720.jpg)
 
 Spring框架中AOP用途
 - 提供了声明式的企业服务，特别是EJB的替代服务的声明
 - 允许用户定制自己的方面，以完成OOP和AOP的互补使用

Spring的AOP实现
- 纯java实现，无序特殊编译过程，不需要控制类加载器
- 目前只支持方法执行连接点(通知Spring Bean的方法执行)
- Spring的AOP不是提供最完整的AOP实现，而是侧重于提供一种AOP实现和SpringIoC容器之间的整合，用于帮助企业应用中的常见问题。
- Spring AOP不会与AspectJ（是一个完整的AOP解决方案）竞争，从而提供综合全面的AOP解决方案。

有接口和无接口的Spring AOP 实现区别
- 有接口：Spring AOP默认使用标准的JavaSE动态代理作为AOP代理，这使得任何接口（或者接口集）都可以被代理。
- 无接口：Spring AOP中也可以使用CGLib代理（如果一个业务对象并没有实现一个接口）



# Schema-Based AOP
基于配置的AOP实现
Spring所有切面和通知器都必须放在一个<aop:config>内(可以配置包含多个<aop:config>元素)，每一个<aop:config>可以包含pointcut，advisor和aspect元素。（必须按照相应顺序配置）
大量使用了Spring的动态代理机制。

## 声明aspect

```xml
<aop:config>
	<aop:aspect id="moocAspectAo" ref="aspcetBean">
    </aop:aspect>
</aop:config>
<bean id="aspcetBean" class="com.pad.s"></bean>
```

## pointcut

execution(public * *(..)) 切入点为执行所有public方法时
execution(* set*(..)) 切入点为执行所有set开始的方法时
execution(* com.xyz.service.AccountService.*(..)) 切入点为执行AccountService类中所有方法时
execution(* com.xyz.service..(..)) 切入点为执行com.xyz.service包下所有方法时
execution(* com.xyz.service...*(..)) 切入点为执行com.xyz.service包及其子包下所有方法类中所有方法时

以上是springAop和AspectJ都支持的 以下只有springAop支持

within（com.xyz.service.*）
within（com.xyz.service..*）
this（com.xyz.service.AccountService）
## 声明pointcut(切入点)
```xml
<aop:config>
	<!--这里是只执行特定方法-->
	<aop:pointcut id="businnessService" expression="com.xyz.myapp.SystemArchitecture.businessService()"/>
</aop:config>
```

## advice通知
使用切面需要的maven依赖
```xml
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-aspects</artifactId>
   <version>4.1.1.RELEASE</version>
</dependency>
```
不然会抛异常
```
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.aop.aspectj.AspectJPointcutAdvisor#0':
```
### Before advice
```xml
	<aop:before pointcut-ref="dataAccessOperation" method="doAccessCheck"/>
    <!--两种方式-->
    <aop:before pointcut="execution(* com.pad.service..*(..))" method="doAccessCheck"/>
```
### After returning advice
```
	<aop:after-returing pointcut-ref="dataAccessOperation" method="doAccessCheck"/>
    <!--或者-->
    <aop:after-returing pointcut-ref="dataAccessOperation" returning="retVal" method="doAccessCheck"/>	
```

Around advice 第一个参数必须是ProceedingJoinPoint类型，不调用他的proceed()方法的话原函数就不执行了。

配置文件配置：
```xml
	<bean id="moocAspect" class="com.imooc.advice.MoocAspect"></bean>
	<bean id="aspectBiz" class="com.imooc.biz.AspectBiz"></bean>

    <aop:config>
		<aop:aspect id="moocAspectAOP" ref="moocAspect">
			<aop:pointcut expression="execution(* com.imooc.biz.AspectBiz.run())" id="moocPointcut"/>
			<aop:before method="before" pointcut-ref="moocPointcut"/>
			<aop:after method="after" pointcut-ref="moocPointcut"/>
			<aop:after-returning method="afterReturn" pointcut-ref="moocPointcut"/>
			<aop:after-throwing method="afterThrow" pointcut-ref="moocPointcut"/>
			<aop:around method="around" pointcut-ref="moocPointcut"/>

			<aop:around method="aroundInit" pointcut="execution(* com.imooc.biz.AspectBiz
			.init(String,int)) and args(name,time)"/>
		</aop:aspect>
    </aop:config>
```
切面
```java
public class MoocAspect {
    public void before(){
        System.out.println("before apsect method run.");
    }
    public void after(){
        System.out.println("after apsect method run.");
    }
    public String afterReturn(){
        System.out.println("after Return");
        String retVal="sss";
        return retVal;
    }

    public void afterThrow(){
        System.out.println("after Throw.");
    }

    public Object around(ProceedingJoinPoint p) throws Throwable{
        System.out.println("around head");
        Object obj=p.proceed();
        System.out.println("around footer");
        return null;
    }
     public Object aroundInit(ProceedingJoinPoint p,String name,int time) throws Throwable{
        System.out.println("around head");
         System.out.println(name+time);
        Object obj=p.proceed();
        System.out.println("around footer");
        return null;
    }
}
```
测试代码：
```java
public class AspectBiz {
    public void run(){
        System.out.println("testtttttttt");
//        throw new RuntimeException("ss");
    }
    public void init(String name,int time){
    }

    @Test
    public void test(){

        ApplicationContext context=new ClassPathXmlApplicationContext("applicationContext.xml");
        AspectBiz biz=(AspectBiz)context.getBean("aspectBiz");
       biz.run();
        biz.init("iii",2);
    }
}
```

## Introductions
为在types-maching中匹配的类增加新的parent，使得可以getBean后强制转换成父类接口，实现接口实现方式为default-impl中指定的类
```xml
<aop:declare-parents
	types-matching="com.xyz.service.*+"
    implement-interface="com.xyz.service.tracking.UsageTracked" 

default-impl="com.xyz.service.tracking.DefaultUsageTracked"/>
<aop:before
	pointcut="com.xyz.SystemArchitecture.businessService() and this (usageTracked)" method="recordUsage"/>
```

UsageTracked usageTracked=(UsageTracked)context.getBean("myService");

所有基于配置文件的aspects只支持单例模式singleton model

## Advisors
advisor就像一个小的自包含的方面，只有一个advice

切面自身通过一个bean表示，并且必须实现某个advice接口，同时，advisor也可以很好的利用AspectJ的切入点表达式

Spring通过配置文件中<aop:advisor>元素支持advisor实际使用中，大多数情况下它会和transactional advice配合使用

为了定义一个advisor的优先级以便让advice可以有序，可以使用order属性来定义advisor的顺序。

















