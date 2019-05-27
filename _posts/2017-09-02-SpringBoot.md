---
title:  Spring Boot - HelloWorld
date: 2017-09-03
tags: Spring
---
# Spring Boot

> 官网 http://projects.spring.io/spring-boot/

环境需求：java8，spring framework5.0以上

https://start.spring.io/ 官网提供了一个简单demo的生成，可以选择maven或者gradle

当然使用idea更加简洁
新建项目 选择Spring Initializr下一步填写相关maven的配置，下一步勾选web之后就可以得到一个初始的spring boot项目

下面一个类就是核心的类，通过运行这个类的main方法启动整个项目。
```java
@SpringBootApplication
public class BootstartApplication implements EmbeddedServletContainerCustomizer{

	public static void main(String[] args) {
		SpringApplication.run(BootstartApplication.class, args);
	}

	//这个方式是修改内置tomcat的端口号，如果没有这个方法默认就是8080
	@Override
	public void customize(ConfigurableEmbeddedServletContainer container){
		container.setPort(80);
	}
}
```
下面新建一个controller
```java
@RestController //该注解表明返回的是json字符串类型
@RequestMapping("/Test")
public class TestCtl {

    @RequestMapping("/test")
    public String test(){
        return "helloworld";
    }

    @RequestMapping("/hello")
    public String hello(){
        return "In hello";
    }

    @RequestMapping("/hi")
    public String hi(){
        return "In hi";
    }
}
```

接着运行上面类的main方法就可以在浏览器中访问localhost/Test/test了。

接着使用mvn package可以进行打包，打包后产生的jar可以使用
```bash
java -jar xxxxxx.jar
```
进行运行，启动项目。完全不需要进行配置。

当然如果感觉默认配置不适合可以在默认生成的resources目录下的application.properties文件里进行配置，如上面代码中进行的端口配置也可以在这个文件中配置。

> http://blog.csdn.net/catoop/article/details/50588851 spring boot可用配置

> https://spring.io/guides/ 各种Spring boot入门例子
