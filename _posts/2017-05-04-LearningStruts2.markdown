---
title:  Struts2总结笔记 
date: 2017-05-04
tags: Struts2
---
# Struts2学习笔记

struts1是世界上第一个mvc框架，struts2不是一个全新的框架，因此稳定性、性能等各个方面都有很好的保证，同时吸收了struts1和webwork两者的优势。

环境要求
- servlet api 2.4 
- jsp api 2.0
- java 5

#### 搭建步骤
1.下载相关jar包
	struts.apache.org/
	
2.创建web项目
	引入相关jar包
    
    - commons-fileupload-xxx.jar
    
    - commons-io-xx.jar
    
    - commons-langxxx.jar
    
    - commons-logging-xxx.jar
    
    - freemarkes-xxx.jar	（模板引擎，基于模板生成输出的通用工具）
    
    - ognl-xxx.jar	（可以理解为一个el表达式）
    
    - struts2-core-xxx.jar
    
    - xwork-core-xx.jar (struts2有些类是关联xwork的)
    
    - javassist-xxx.jar (可以理解为解析java类文件的包)
    
3.创建完善配置文件
	配置web.xml
    增加一个过滤器
    
 ```xml
    
  <filter>  
	  <filter-name>struts2</filter-name>  
	  <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class> 
  </filter>    
  
  <filter-mapping>  
	  <filter-name>struts2</filter-name>
	  <url-pattern>/*</url-pattern>
  </filter-mapping>
```
在src中创建struts.xml文件（在struts-2.5.10.1\src\apps\rest-showcase\src\main\resources中有示例）
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
"http://struts.apache.org/dtds/struts-2.3.dtd">

<struts>
    <package name="default" namespace="/" extends="struts-default">
    	<action name="helloworld" class="com.xxx.xxx.类名">
        	<result >/result.jsp</result>
        </action>
    </package>
</struts>
```

4.创建Action并测试启动

创建class继承ActionSupport(也可以不继承，如果action标签中未指定class属性则默认使用ActionSupport类)
```java
import com.opensymphony.xwork2.ActionSupport;

public class HellowrodAction extends ActionSupport{
	@Override
    	public String execute() throws Exception{
    		system.out.println("执行");
        	return SUCESS;//返回的string是视图的路径
    	}
}
```

> myeclipse可以直接右键项目->myeclipse->add Struts Cpibilities 添加



#### struts2的工作原理和执行流程
![struts2架构剖析](http://img.mukewang.com/58e9d56f00019fed12800718.jpg "struts2架构剖析")

首先HttpServletRequest请求经过一些过滤器（ActionContextCleanup、OtherFilters(SiteMesh,etc)、FilterDispatcher(struts2.1.3之后变为StrutsPrepareAndExcuteFiler)）,到达ActionMapper,如果ActionMapper决定请求是否需要请求一个action，则由struts的核心控制器（图中第三层）将控制权委派给ActionProxy,ActionProxy通过ConfigureManager加载核心控制文件Struts.xml进行配置找到需要调用的action，如果找到之后ActionProxy创建一个ActionInvocation实例（实例中包含拦截器，Action，Result）之后开始进行拦截器的执行，拦截器1，拦截器2，拦截器3...，之后执行Action，得到结果Result，获取相应视图Template（jsp，freemarker等），之后反序执行拦截器，最后HttpServletResponse响应客户端请求。


struts.xml中包含内容：
1.全局熟悉
2.用户请求和响应Action之间的对应关系
3.Action可能用到的参数和返回结果
4.各种拦截器的配置
```xml
<!--引用其他struts.xml-->
<include file="struts-default.xml"></include>
```
package提供了将多个Action组织为一个模块的方式

    package的名字必须是唯一的package可以扩展,当package扩展自另一个package时,
    该package会在本身配置的基础上加入扩展的package的配置,父package必须在子package前配置
    name：package名称
    extends：继承的父package名称
    abstract：设置package的属性为抽象， 抽象的package不能定义action 值true：false
    namespace:定义package命名空间，该命名空间影响到url的地址，例如此命名空间为/testname访问的地址为http:xxx:8080/proname/test/xx.action


定义拦截器
```xml
<interceptors>
	<interceptor name="timer" class="classpath1"></interceptor>
	<interceptor name="logger" class="classpath2"></interceptor>
    <!--拦截器栈-->
    	<interceptor-stack name="timer" class="classpath">
            <interceptor-ref name="timer" ></interceptor-ref>
        	<interceptor-ref name="logger" ></interceptor-ref>
        </interceptor-stack>
</interceptors>


<action name="" class=""> 
	<interceptor-ref name="timer"></interceptor-ref>
    <result name="success" type="dispatcher">/talk.jsp</result>
    <!--name：对应get/set-->
	<param name="url">http://www.sina.com</param>
</action>
```

struts.properties

放在struts.xml相同目录

1.struts2框架的全局属性文件，自动加载。

2.该文件包含很多key-value键值对。

3.该文件完全可以配置在struts.xml中，使用constant元素。



Action搜索顺序
http://localhost:8080/struts2/path1/path2/path3/xxx.action

    第一步：判断package是否存在，如path1/path2/path3
    第二步： 存在：判断action是否存在，如果不存在则去默认的namespace的package里面寻找action  
        	不存在：检查上一级路径的package是否存在（直到默认namespace）重复第一步。
    第三步：如果没有则报错。

#### 动态方法调用：
解决一个action对应多个请求，防止action过多。

1.method属性
	
在struts.xml中的action标签设置
```xml
    <action name="addAction" method="add" class="xxx.xx.xx">
    	<result>xxx.jsp</result>
    </action>
```
2.感叹号方式(官方不推荐这种方法)
    对应方法返回对应 return "add";
```xml
     <action name="addAction"  class="xxx.xx.xx">
    	<result>xxx1.jsp</result>
        <result name="add">xxx2.jsp</result>
        <result name="del">xxx3.jsp</result>
    </action>
    
    <constant name="struts.enable.DynamicMethodInvocation" value="true"></constant>
```
 访问方式：localhost:8080/xxx/helloworld!add.action
 
3.通配符方式(官方推荐)
```xml
    <action name="addAction_*"  method="{1}" class="xxx.xx.xx">
    	<result>{1}.jsp</result>
        <result name="add">{1}.jsp</result>
        <result name="del">{1}.jsp</result>
    </action>
```
     访问方式：localhost:8080/xxx/helloworld_add.action
      
更加简洁，可以一个项目中只配置一个action
```xml
      <action name="*_*"  method="{2}" class="xxx.xx.{1}Action">
    	<result>{2}.jsp</result>   这里默认不写的话执行execute（）方法；
        <result name="add">{2}.jsp</result>
        <result name="del">{2}.jsp</result>
      </action>
```   
指定多个配置文件
```xml
    <include file="login1.xml"></include>
    <include file="login2.xml"></include>
        
         <!--在每个配置文件中加入解决编码问题-->
        <constant name="struts.i18n.encoding" value="UTF-8"></constant>
```     
        
默认action
```xml
        <default-action-ref name="notfound"></default-action-ref>
        <action name="notfound">
        	<result>404page.jsp</result>
        </action>
```
        
  
 ```xml
 		<!--struts2后缀 -->
        <constant name="struts.action.extension" value="html"></constant>
 ```   
   > 为空则去掉后缀，或者直接不设置这个属性
   可配置地点：web.xml,struts.properties,struts.xml
        
        
   
   
#### 接收参数
    
    
        1.使用action的属性接收
        	在action中定义private属性，和表单提交name对应，实现get/set方法。
        2.使用DomainModel接收
        	在action中定义private的实体类，实体类中实现get/set 方法，action中实现实体类的get和set方法，在表单中name为	对象名.属性名 
        3.使用ModelDriven接收参数
        实现ModelDriven<T>接口
        对象需要实例化，并且用getset方法， 表单中的name则是属性名。
        
        对于List类型from表单中的name为属性名【下标】
        如果List中是对象类型则表单中未属性名[下标].属性名
        
        
 处理结果类型：
 
    省略name值则默认name值为success
    result中带/表示绝对路径 项目起始 没有则为相对路径

    com.opensymphony.xwork2.Action 类中
     
     SUCCESS	 正确执行，返回相应视图
     
     NONE	表示正确执行，并不反悔任何视图
     
     ERROR 表示Action执行失败，返回到错误处理视图
     
     LOGIN Action因为用户没有登录的原因没有正确执行，将返回该登录视图，要求用户登录。
     
     INPUT 唯一有特殊意义的，Action的执行，需要从前端界面获取参数，INPUT就是代表这个参数输入的界面，
    在一般应用中，会对这些参数进行验证，如果验证没有通过，将自动返回到该视图。
        
  在result中name换成input，当在action中数据类型转换错误的情况下会自动触发。第二种在action的方法中
      
   实现validate
```java
       @Override
       public void validate(){
        this.addFieldError("username","用户名不能为空");
       }
  ```
 jsp页面中添加struts标签
```
       <%@ taglib prefix="s" uri="/struts-tags" %>
       <input type="text" name="username"/><s:fielderror name="username"></s:fielderror>
```   
      
#### 处理结果类型

全局result需要在包中定义。
```xml
      <global-results>
      	<result name="404page">xxx.jsp</result>
      </global-results>
      
      <result name="add">
      	<param name="location">/$(#requst.path).jsp</param>
      </result>
```
> 这里为ognl表达式 表示从request中获取path属性值同requst.getAttribute("path").
      
      result的type属性
      chain:将action和另一个action链接起来，
      dispatcher
      redirect	
      redirectAction
      stream  流
      plainText
      
    freemarker
      
    

 拦截器的执行过程是一个递归的过程
 从ActionProxy出来后先依次执行intercpt1，2，3执行Action之后得到result到template（jsp，freemarker...）又执行 3 ，2 ，1 intercpt，之后给HttpServletResponse,响应客户端。
    
    
#### 自定义拦截器
 
    实现Interceptor接口
    void init()
    void destroy()
    String intercept(ActionInvocation ai)throws Exception;
    - 实现拦截功能
    - 利用ActionInvocation参数获取Action状态
    - 返回result字符串作为逻辑视图

继承AbstractInterceptor类
- 提供了init()和destroy()方法的空实现。
- 只需要实现intercept方法即可

实际开发中一般继承AbstractInterceptor类


1. 创建拦截器
```java
public class TimerInterceptor extends AbstarctInterceptor{
	@Override
    public String intercept(ActionInvocation invocation) throw Exception{
    long start=System.currentTimeMillis();
    
    //执行下一个拦截器，如果已经是最后一个拦截器则执行目标action，返回结果视图
    String result=invocation.invoke();
    
    long end=System.currentTimeMillis();
    System.out.println(end-start);
    return result;
    }
}
```
2. 配置注册拦截器
	在package标签中注册拦截器
    ```xml
    <interceptors>
    		<interceptor name="mytimer" class="xxx.xxx.xx.classname"></interceptor>
    </interceptors>
    ```
    在相应action标签中引用拦截器
    ```xml
    <interceptor-ref name="mytimer"></interceptor-ref>
	```

Struts2内建拦截器
- params拦截器
		负责将请求参数设置为Action属性
- staticParams拦截器
		将配置文件中的action元素的子元素param参数设置为Action属性
- servletConfig拦截器
		将源于ServletAPI的各种对象注入到Action，必须实现对应接口
- fileUpload拦截器
		对文件上传提供支持，将文件和元数据设置对应的Action
   		实际上还是使用了Commons-FileUpload
- exception拦截器
		捕获异常，并且将异常映射到用户自定义的错误页面
- validation拦截器
		调用验证框架进行数据验证。
    
    > 在struts2-corexx.jar中的struts-default.xml中能找到这些拦截器
	拦截器栈是拦截器的组合 defaultStack，
    default-interceptor-ref name="defaultStack"默认调用了defaultStack
    

关于拦截器的注意点：   
- 在struts-default.xml中定义了一个defaultStack拦截器栈，并指定了作为默认拦截器。
- 只要在定义包的过程中继承struts-default包，那么defaultStack将是默认拦截器。
- 当为包中的某个action显式指定了某个拦截器，则默认拦截器不会起作用。（类似继承的构造函数）
- 拦截器栈中各个拦截器顺序很重要。
    
 > 如果要使用则在之前手动调用defaultStack栈,引用方式和interceptor一样。

#### 下面是4个开发模式常用配置的简介(摘自imooc)
```xml
    <!-- 开启使用开发模式，详细错误提示 -->
    <constant name="struts.devMode" value="true"/>
    <!-- 指定每次请求到达，重新加载资源文件 -->
    <constant name="struts.i18n.reload" value="true"/>
    <!-- 指定每次配置文件更改后，自动重新加载 -->
    <constant name="struts.configuration.xml.reload" value="true"/>
    <!-- 指定XSLT Result使用样式表缓存 -->
    <constant name="struts.xslt.nocache" value="true"/>
```






