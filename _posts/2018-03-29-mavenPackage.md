---
title: maven打包jar配置
date: 2018-03-29
tags: 
- maven 
- 构建工具
---
# maven打包成jar，指定主类

@pom.xml
```xml

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.padcn</groupId>
		<artifactId>mall</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>
	<artifactId>mall-util</artifactId>
	<build>
		<pluginManagement>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-jar-plugin</artifactId>
					<configuration>
						<source>1.8</source>
						<target>1.8</target>
						<archive>
							<manifest>
								<mainClass>org.padcn.utils.TestUtils</mainClass>
								<addClasspath>true</addClasspath>

								<classpathPrefix>lib/</classpathPrefix>
							</manifest>
						</archive>
						<classesDirectory>
						</classesDirectory>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
	</build>

</project>
```

> 标签不整齐有可能在打包时出现pom.xml: expected START_TAG or END_TAG not TEXT
>
> 所以pom.xml的文本还是需要对齐才行。



##  让jar在后台运行

### 运行：

```bash
java -jar server.jar &
```

后面加个&就可以 （使用的是linux的nohup命令）

> Linux Shell nohup命令用法 https://www.cnblogs.com/gotodsp/p/6390023.html

### 关闭：

列出java后台进程  

```bash
ps -ef | grep java
```

查看后用kill -9 加pid，直接关闭

```bash
kill -9 [pid]
```



