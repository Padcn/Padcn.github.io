---
title: JAXB 进行java对象和xml的转换
date: 2018-03-29
tags: 
- DTO 
- xml
---
# JAXB 进行java对象和xml的转换

> 百度百科：JAXB能够使用Jackson对JAXB注解的支持实现(jackson-module-jaxb-annotations)，既方便生成XML，也方便生成JSON，这样一来可以更好的标志可以转换为JSON对象的JAVA类。JAXB允许JAVA人员将JAVA类映射为XML表示方式，常用的注解包括：@XmlRootElement,@XmlElement等等。 



## 实体类注解

```java
package app.lkx.xml.jaxb.bean;

import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement(name="RECORD")
public class Record {
	private String id;
	private String name;
	
	@XmlElement(name="ID")
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	@XmlElement(name="NAME")
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	@Override
	public String toString() {
		return "Record [id=" + id + ", name=" + name + "]";
	}
	
}
```

## 将xml转换为实体类

```java
package app.lkx.xml.jaxb;

import java.io.StringReader;

import javax.xml.bind.JAXBContext;
import javax.xml.bind.JAXBException;
import javax.xml.bind.Unmarshaller;

import org.junit.Test;

import app.lkx.xml.jaxb.bean.Record;

public class JaxbMainTest {
	@Test
	public  void strToBeanTest() {
		Object obj=null;
		String record="<?xml version=\"1.0\" encoding=\"utf-8\"?>"
				+ "<RECORD>"
				+"<ID>00101</ID>"
				+ "<NAME>URPWWW</NAME>"
				+ "</RECORD>";
		try {
			JAXBContext content=JAXBContext.newInstance(Record.class);
			Unmarshaller unmarshaller=content.createUnmarshaller();
			StringReader sr=new StringReader(record);
			obj=unmarshaller.unmarshal(sr);
		} catch (JAXBException e) {
			e.printStackTrace();
		}
		System.out.println((Record)obj);
	}
}

```

