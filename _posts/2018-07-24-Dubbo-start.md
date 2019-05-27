---
title: Dubbo Start
date: 2018-07-24
tags: 
- RPC
- Dubbo
- 分布式
---
# Dubbo Start

> 官网 http://dubbo.apache.org/

Dubbo是是一款高性能Java RPC框架。 开始由阿里开源，后来捐献给了apache基金会。



Dubbo既然定义就是RPC框架，主要功能也是实现服务的远程调用，remote procedure call即远程服务调用。其中几部分是：服务提供者，服务消费者，注册中心。

服务提供者将开放的服务注册到注册中心，服务消费者通过注册中心查询所需要的服务，再根据查询到的信息到服务提供者处调取服务。 图中还多了一个Monitor用来监视调用情况。

![](http://dubbo.apache.org/docs/zh-cn/user/sources/images/dubbo-architecture.jpg "dubbo流程")

Dubbo Maven坐标为

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.6.2</version>
</dependency>
```

注册中心可以是zookeeper或者redis。dubbo推荐zookeeper。这里使用zookeeper，Curator框架的使用是为了更方便的操作zookeeper，其中提供了丰富的操作。

> Apache Curator is a Java/JVM client library for [Apache ZooKeeper](https://zookeeper.apache.org/), a distributed coordination service. It includes a highlevel API framework and utilities to make using Apache ZooKeeper much easier and more reliable. It also includes recipes for common use cases and extensions such as service discovery and a Java 8 asynchronous DSL. 

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.11</version>
    <type>pom</type>
</dependency>
<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.10</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>2.8.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>2.8.0</version>
</dependency>
```

## 基于XML配置方式

这里的项目结构采用maven的多模块结构

demo-parent				父模块，打包方式为pom

​	|------demo-api			api定义模块，实体类的定义，以及服务接口方法定义。

​	|------demo-provider		服务提供方，依赖于demo-api，对demo-api中接口方法进行实现，提供服务

​	|------demo-consumer		服务消费放，依赖于demo-api, 调用demo-provider提供的服务。

### 服务方法定义

demo-api中进行服务方法的定义，如这里定义一个DemoService

@org.padcn.dubbo.demo.service.DemoService

```java
package org.padcn.dubbo.demo.service;
import java.util.List;
public interface DemoService {
    List<String> getPermissions(Long id);
}
```

### 对服务方法的实现

demo-provider中对demo-api中定义的方法进行实现,并注册到注册中心。

@org.padcn.dubbo.demo.service.impl

```java
package org.padcn.dubbo.demo.service.impl;

import org.pad.dubbo.demo.service.DemoService;
import java.util.ArrayList;
import java.util.List;

public class DemoServiceImpl implements DemoService {
    @Override
    public List<String> getPermissions(Long id) {
        List<String> list=new ArrayList<>(10);
        list.add("demo1");
        list.add("demo2");
        list.add("demo3");
        list.add("demo4");
        list.add("demo5");
        return list;
    }
}
```

注册服务到注册中心

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
	<!--声明本应用一些信息-->
    <dubbo:application name="demotest-provider" owner="programer" organization="dubbox"/>
	<!--注册中心配置-->
    <dubbo:registry address="zookeeper://localhost:2181"/>
	<!--开放调用端口-->
    <dubbo:protocol name="dubbo" port="20880"/>
	<!--spring的bean配置-->
    <bean id="demoService" class="org.pad.dubbo.demo.service.impl.DemoServiceImpl"/>
    <!--服务注册-->
    <dubbo:service interface="org.pad.dubbo.demo.service.DemoService" ref="demoService" protocol="dubbo"/>
</beans>
```



### 消费者对服务调用

demo-consumer 调用服务，这里就直接使用main方法调用了，主要还是配置文件的定义，其他都和普通程序一样使用。

```java
package org.padcn.dubbo.consumer;

import org.pad.dubbo.demo.service.DemoService;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Consumer {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context=new ClassPathXmlApplicationContext("consumer.xml");
        context.start();
        System.out.println("consumer start");
        DemoService demoService=context.getBean(DemoService.class);
        System.out.println("consumer");
        System.out.println(demoService.getPermissions(1L));
    }
}
```

配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!--声明应用信息-->
    <dubbo:application name="demotest-consumer" owner="programer" organization="dubbox"/>
    <!--配置注册中心-->
    <dubbo:registry address="zookeeper://localhost:2181"/>
    <!--声明服务-->
    <dubbo:reference id="permissionService" interface="org.pad.dubbo.demo.DemoService"/>
</beans>
```

到此就完成了一个示例demo



## 基于注解方式配置

注解方式需要dubbo版本 `2.5.7` 及以上版本支持 。

注解相比xml配置方式更简洁清晰。

> 官方文档: http://dubbo.apache.org/#!/docs/user/configuration/annotation.md?lang=zh-cn

这里使用了SpringBoot+dubbo的形式，项目结构和基于xml没有变化。

### 服务提供方

配置bean

```java
package org.padcn.demo.provider.config;

import com.alibaba.dubbo.config.ApplicationConfig;
import com.alibaba.dubbo.config.ProtocolConfig;
import com.alibaba.dubbo.config.RegistryConfig;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DubboConfiguration {

    @Bean
    public ApplicationConfig applicationConfig(){
        ApplicationConfig config=new ApplicationConfig();
        config.setName("demo-provider");
        return config;
    }

    @Bean
    public RegistryConfig registryConfig(){
        RegistryConfig config=new RegistryConfig();
        config.setAddress("zookeeper://127.0.0.1:2181");
        //config.setAddress("redis://127.0.0.1:6379");
        config.setClient("curator");
        config.setPort(12349);
        return config;
    }

    @Bean
    public ProtocolConfig protocolConfig(){
        ProtocolConfig protocolConfig=new ProtocolConfig();
        protocolConfig.setPort(12345);
        return protocolConfig;
    }
}
```

服务实现

```java
package org.padcn.demo.provider.service.impl;

import com.alibaba.dubbo.config.annotation.Service;
import org.padcn.demo.api.DemoService;
// 这里的Service注解不是Spring的注解，是dubbo的服务暴露注解
@Service(version = "2.0")
public class DemoServiceImpl implements DemoService {
    @Override
    public String sayHi(String name) {
        return "hello?"+name;
    }
}
```

在springboot启动类上配置注解扫描

```java
package org.padcn.demo.provider;

import com.alibaba.dubbo.config.spring.context.annotation.DubboComponentScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@DubboComponentScan(basePackages = "org.padcn.demo.provider.service.impl")
public class DemoProviderApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoProviderApplication.class, args);
	}
}
```



### 服务消费方配置

配置类

```java
package org.padcn.demo.consumer.config;

import com.alibaba.dubbo.config.ApplicationConfig;
import com.alibaba.dubbo.config.ConsumerConfig;
import com.alibaba.dubbo.config.ProtocolConfig;
import com.alibaba.dubbo.config.RegistryConfig;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DubboConfiguration {

    @Bean
    public ApplicationConfig applicationConfig(){
        ApplicationConfig config=new ApplicationConfig();
        config.setName("demo-consumer");
        return config;
    }

    @Bean
    public RegistryConfig registryConfig(){
        RegistryConfig registryConfig=new RegistryConfig();
        registryConfig.setAddress("zookeeper://127.0.0.1:2181");
        //registryConfig.setAddress("redis://127.0.0.1:6379");
        registryConfig.setClient("curator");
        return registryConfig;
    }
    
    @Bean
    public ConsumerConfig consumerConfig(){
        ConsumerConfig consumerConfig=new ConsumerConfig();
        consumerConfig.setTimeout(3000);
        return consumerConfig;
    }
}
```

服务调用

```java
package org.padcn.demo.consumer;

import com.alibaba.dubbo.config.annotation.Reference;
import org.padcn.demo.api.DemoService;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DemoController {
	//这里用dubbo的reference替换了注入的注解
    // 确定一个服务主要是3个要素： 版本号，以及接口，服务分组。不过这里没有写分组，属性是group
    @Reference(version = "2.0",application = "demo-consumer",url="dubbo://localhost:12345")
    private DemoService demoService;
    @RequestMapping("/test")
    public String test(String name){

        return demoService.sayHi(name);
    }
}
```

同样在springboot启动类上配置注解扫描

```java
package org.padcn.demo.consumer;

import com.alibaba.dubbo.config.spring.context.annotation.DubboComponentScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@DubboComponentScan(basePackages = "org.padcn.demo.api")
public class DemoConsumerApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoConsumerApplication.class, args);
	}
}
```

完事，之后启动zookeeper/redis后启动provider和consumer就可以了。


> 如果需要更加深入可以看这里 http://ifeve.com/dubbo-learn-book/