---
title: Gradle使用
date: 2018-02-25
tags: 构建工具
---
# Gradle使用

> http://www.importnew.com/15881.html

### 无依赖构建

​	在项目文件夹下创建'src/main/java/hello'这样的目录结构，

```bash
mkdir -p src/main/java/hello
```

进入目录后终端中执行，

```shell
#查看可用命令
gradle tasks 
```

gradle的配置文件为build.gradle,位置放在项目根路径，与src文件夹同级。

创建build.gradle

```groovy
apply plugin: 'java'
```

这里配置了插件启用了java的构建功能。

再次使用gradle tasks命令后会发现多出了一些可用功能。



在hello文件夹下创建HelloWorld.java文件

@HelloWorld.java

```java
package hello;

public class HelloWorld{
  public static void main(String[] args){
    System.out.println("hello");
  }
}
```



执行

```shell
gradle build
```

进行构建执行后会发现项目下生成了一个build目录，下面就是构建后的结果。

1. classes：编译后的class文件
2. lib:组装好的项目包

目前直接使用java -jar运行打包后的jar还是不能运行项目的，因为没有指定运行的类会提示 ‘没有主清单属性’。需要在build.gradle进行配置

```
apply plugin:'application'

mainClassName='hello.HelloWorld'
```

保存后再次构建回多出几个目录，scipts和distributions，通过scrpts文件夹中的批处理文件可以运行项目。

### 导入第三方依赖

```groovy
//对于第三方库来源的配置，告诉构建系统通过maven中央仓库来检索项目所需依赖
repositories{
	mavenLocal()
	mavenCentral()
}
//外部依赖，相当于maven坐标
dependencies{
	compile "joda-time:joda-time:2.2"
}
//对生成的jar包命名，版本号 最终生成为baseName-version.jar
jar{
	baseName='gs-gradle'
	version='0.1.0'
}
```



gradle clean 清理项目

gralde run 直接运行项目