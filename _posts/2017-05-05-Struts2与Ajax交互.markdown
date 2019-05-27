---
title:  Struts2与Ajax交互 
date: 2017-05-05
tags: Struts2
---
# Struts2和Ajax交互

### 1.原生方法（直接调用response输出）

原生方法就是直接使用ServletActionContext获取response进行操作。但是这种方式不被推荐。

@index.jsp
```js
//引入jqury
<script type="text/javascript" src="http://ajax.aspnetcdn.com/ajax/jQuery/jquery-3.2.0.js">
</script> 

<script type="text/javascript">
window.onload=function(){
    var actionUrl="testAjax"
    $.ajax({
  		tyep="post",
		url:actionUrl,
        data:{"name","pad"},
        success:function(data){
        	alert(data);
        }
    });
}
</script>
```

@TestAajaxAction
```java

public class AjaxTestAction extends ActionSupport{
	private String name;
	public void testAjax(){
		PrintWriter out=null;
		try{
        	//通过ServletActionContext获取Response，之后获取PrintWriter
			out=ServletActionContext.getResponse().getWriter();
		}catch(IOException e){
			e.printStackTrace();
		}
		
		out.println("I have got your message "+name);
	}
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	
}
```

@struts.xml
```xml
<action name="testAjax" method="testAjax" class="com.pad.action.AjaxTestAction"/>   
```

### 2.采用stream形式与ajax交互
```xml
<action name="testStream" class="com.pad.action.AjaxTestAction" method="testAjaxStream">
   		<result type="stream">
   			<param name="inputName">result</param>
   		</result>
   	</action>
```

```java
public class AjaxTestAction extends ActionSupport{
	private String name;
	private InputStream input;
    
	public String testAjaxStream()throws Exception{
		String tip="It`s ok to use it.";
		if("lalal".equals(name)){
			tip="UserName has been used.";
		}
		
		input=new ByteArrayInputStream(tip.getBytes());

		return "success";
	}

	public InputStream getResult(){
		return input;
	}
    
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}

}
```



### 3.使用Struts2提供的jar包

Struts2+ajax+json
网上找资料发现很多需要使用jar包struts2-json-plugin-2.5.2.jar，但是去除之后发现并不影响使用，很奇怪。也许是在xml中的json-default中已经有处理（不从json-default继承后tomcat启动加载struts.xml失败）。

@struts.xml
```xml
 <package name="json" extends="struts-default,json-default">
 	<action name="json" class="com.pad.action.AjaxTestAction" method="testAjaxSJ">
 		<result type="json">
 			<param name="root">dataMap</param>
 		</result>
 	</action>
 </package>
 ```
 @AjaxTestAction
 ```java
 
public class AjaxTestAction extends ActionSupport{
	private Map<String,Object> dataMap;
    public String testAjaxSJ(){
		List<String> list=new ArrayList<String>();
		list.add("abc1");
		list.add("abc2");
		list.add("abc3");
		list.add("abc4");
		list.add("abc5");
		
		dataMap=new HashMap<String,Object>();
		dataMap.put("list", list);
        
		return SUCCESS;
	}
	
	public Map<String, Object> getDataMap() {
		return dataMap;
	}

}

 ```
 
 直接访问后页面返回：
 
	 {"list":["abc1","abc2","abc3","abc4","abc5"]}
