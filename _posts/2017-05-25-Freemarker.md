---
title:  FreeMarker
date: 2017-05-25
tags: FreeMarker
---
# FreeMarker

官网 http://freemarker.org

通过模板对java对象数据进行解析，生成html  
模板 + 数据模型 = 输出
freemarker通过一个类似Map的数据结构进行一一对应。
maven依赖
```
<dependency>
	<groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.20</version>
</dependency>
```
# SpringMVC+freemarker简单应用demo
springmvc配置文件中配置FreeMarkerViewResolver
```xml
	<bean id="viewResolver" class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
        <property name="suffix">
            <value>.ftl</value>
        </property>
        <property name="contentType" value="text/html;charset=UTF-8"></property>
    </bean>
```
> 这里如果使用多种视图混合使用，比如jsp和freemarker混合使用，则需要在bean中设置order属性，用来设置resolver的优先级，否则只有一个视图会生效，默认的优先级为比较大的一个数，数字越小优先级越高。

spring中配置FreeMarkerConfigurer
```xml
	<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
        <property name="templateLoaderPath" value="/WEB-INF/template/"/>
        <property name="freemarkerSettings">
            <props>
                <prop key="template_update_delay">0</prop>
                <prop key="default_encoding">UTF-8</prop>
                <prop key="number_format">0.##########</prop>
                <prop key="datetime_format">yyyy-MM-dd HH:mm:ss</prop>
                <prop key="classic_compatible">true</prop>
                <prop key="template_exception_handler">ignore</prop>
            </props>
        </property>
    </bean>
```

测试页面模板

@Test.ftl
```
<html>
<head>
    <title>Welcome!</title>
</head>
<body>
<h1>Welcome ${name}!</h1>
</body>
</html>
```
controll层对应请求
```java
	@RequestMapping("/testFM1")
    public ModelAndView testFM1(HttpServletRequest request, HttpServletResponse response){
        ModelAndView mv=new ModelAndView("test");
        mv.addObject("name","pad123");
        return mv;
    }
```