---
title: wildfly部署war包
date: 2018-04-18
tags: 
- 服务器
- 部署
---
# wildfly-12.0.0.Final 部署war包

> 下面${wildfly_home}代表wildfly解压的目录

### 添加管理用户

windows下 运行 ${wildfly_home}/bin/add-user.bat，根据提示添加或者修改用户，这个用户是进入wildfly控制台的用户。


### 部署

放置war包到 ${wildfly_home}/standalone/deployments目录下


### 启动

windows下${wildfly_home}/bin/standalone.bat


启动后可以通过localhost:9990进入服务控制台

localhost:8080/项目名 进入部署的系统。


### 控制台

在控制台中也可以进行部署

端口以及其他一些常见配置在${wildfly_home}/standalone/configuration/standalone.xml中进行配置。

