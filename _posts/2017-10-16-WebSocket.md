---
title:  WebSocket
date: 2017-10-16
tags: web
---
# WebSocket

> https://www.cnblogs.com/fuqiang88/p/5956363.html
>
> http://www.runoob.com/html/html5-websocket.html

## 概述

WebSocket是HTML5开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。

在WebSocket API中，浏览器和服务器只需要做一个握手的动作，然后，浏览器和服务器之间就形成了一条快速通道。两者之间就直接可以数据互相传送。

浏览器通过 JavaScript 向服务器发出建立 WebSocket 连接的请求，连接建立以后，客户端和服务器端就可以通过 TCP 连接直接交换数据。

当你获取 Web Socket 连接后，你可以通过 **send()** 方法来向服务器发送数据，并通过 **onmessage** 事件来接收服务器返回的数据。

## 使用场景

多用于需要服务端需要主动通知用户时，比如聊天系统，消息推送等。

在日常业务开发时有时候会遇到一些需求，比如系统中有新消息产生的时候需要通知到用户，这时候有个选择就是使用ajax进行轮询，但是大部分时候是没有新消息的，所以很浪费资源，效率还低。但是再有了webSocket之后，就可以做到让服务端有新消息时主动通知用户。

## 服务端代码

@WebSocketTest.java

```java
import java.io.IOException;
import java.util.HashSet;
import java.util.Set;

import javax.websocket.OnClose;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.ServerEndpoint;

@ServerEndpoint("/websocketTest")
public class WebSocketTest {
    /**
     * 用于存放用户对象
     */
	public static Set<Object> userSet=new HashSet<Object>();
	private Session session;
	/**
     * 接收消息时调用方法
     */
	@OnMessage
	public void onMessage(String message)throws IOException,InterruptedException{
		System.out.println("receive msg "+message);
		distributeMsg(message);
	}
	/**
     * 初始化连接websocket时方法
     */
	@OnOpen
	public void onOpen(Session session){
		this.session=session;
		userSet.add(this);
		System.out.print("socket open."+this+" connnected.");
		System.out.println(userSet.size());
	}
	/**
     * client端中断连接时调用方法,这里是移除client连接对象
     */
	@OnClose
	public void onClose(){
		System.out.println("socket close"+this+" disconnnected.");
		userSet.remove(this);
	}
	/**
     * 消息分发给set中的所有连接client
     */
	public void distributeMsg(String msg){
		System.out.println("dis size"+userSet.size());
		for(Object wst:userSet){
			try{
                // 发送消息
				((WebSocketTest)wst).session.getBasicRemote().sendText(msg);
			}catch(IOException e){
				e.printStackTrace();
			}
		}
	}

}
```



## Client端js代码

@TestWebSocket.html

```html
<!DOCTYPE html>
<html>
<head>
	<title>TestWebSocket</title>
	<meta charset="utf-8">
	<script src="https://cdn.bootcss.com/jquery/3.2.1/jquery.slim.min.js"></script>
	<script type="text/javascript">
		var ws;
		var openflag=false;
		function openWs(){
			ws=new WebSocket("ws://127.0.0.1:8080/client/websocketTest");
			if(!openflag){;
				ws.onopen=function(){
					alert("open");
					$("#ctlBtn").prop('value','close');
				}
			}else{
				ws.onclose=function(){
					alert("close");
					$("#ctlBtn").prop('value','open');
				}
			}
			openflag=!openflag;
		}
		function sendMsg(){
			var sendText=$("#nickName").val()+":"+$("#context").val();
			ws.send(sendText);
			
			ws.onmessage=function(evt){
				var msg=evt.data;
				$("#container").append(msg+"<br/>");
				$("#context").empty();
			}
		}
		$(document).ready(function(){
			//openWs();
		});
	</script>
</head>
<body>
	<div style="text-align: center;">
		<a href="#">WS run</a>
		
		<input type="text" id="nickName" name="nickName" placeholder="昵称:"/>
		<input type="text" id="context" name="context" placeholder="发送内容"/>

		<input type="button" id="ctlBtn" onclick="openWs()" value="Open"/>
		<input type="button" id="sendBtn" onclick="sendMsg()" value="Send"/>
		<hr/>
		<div id="container"></div>
	</div>
</body>
</html>
```

## maven依赖坐标

```xml
		<dependency>
		    <groupId>org.apache.tomcat</groupId>
		    <artifactId>tomcat-coyote</artifactId>
		    <version>7.0.27</version>
		</dependency>
		<dependency>
            <groupId>javax.websocket</groupId>
            <artifactId>javax.websocket-api</artifactId>
            <version>1.0-rc5</version>
            <scope>provided</scope>
        </dependency>
```

