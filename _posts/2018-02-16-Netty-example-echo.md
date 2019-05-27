---
title: Netty-example-echo
date: 2018-02-16
tags: web
---
# Netty-example-echo

> https://github.com/netty/netty/tree/4.1/example/src/main/java/io/netty/example netty例子代码
>
> https://mp.weixin.qq.com/s/APJBmqakodF7r0KTZwrkhw 匠心零度的Netty（一）：入门篇

先创建maven或者Gradle项目，导入依赖

```xml
<dependency>
  <groupId>io.netty</groupId>
  <artifactId>netty-all</artifactId>
  <version>4.1.19.Final</version>
</dependency>
```

### 服务端

服务端主要是绑定端口后监听，并且做出相应的响应。首先创建Handler，具体的响应操作都在这实现。

`ChannelHandler` which adds callbacks for state changes. This allows the user to hook in to state changes easily.  

继承ChannelInboundHandlerAdapter并重写其中三个方法channelRead(ChannelHandlerContext ctx,Object msg)和channelReadComplete(ChannelHandlerContext ctx)以及exceptionCaught(ChannelHandlerContext ctx,Throwable cause)方法。

@NettyServerHandler.java

```java
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

public class NettyServerHandler extends ChannelInboundHandlerAdapter{
	/**
	 * 当服务端接受消息时调用
	 */
	@Override
	public void channelRead(ChannelHandlerContext ctx,Object msg){
		//打印出接受到的消息
		String body=(String) msg;
		System.out.println(body);
		//返回消息
		String res="感谢关注！"+System.getProperty("line.separator");
		ByteBuf resp=Unpooled.copiedBuffer(res.getBytes());
		ctx.writeAndFlush(resp);
	}
	/**
	 * 消息读取完毕时调用
	 */
	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		ctx.flush();
	};
	/**
	 * 捕获到异常时执行
	 */
	@Override
	public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause)throws Exception{
		cause.printStackTrace();
		ctx.close();
	}
}
```

使用ServerHandler

@NettyServer.java

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.LineBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;

public final class NettyServer {
	static final int PORT=8007;
	
	public static void main(String[] args) throws InterruptedException {
		//用以处理nio的accept
		EventLoopGroup bossGroup=new NioEventLoopGroup();
		//用以处理nio的Read和Write事件
		EventLoopGroup workerGroup=new NioEventLoopGroup();
		try{
			ServerBootstrap b=new ServerBootstrap();
			//channel传入构造Channel的类 ，option设置创建Channel时的设置，handler设置用来响应请求的handler
			b.group(bossGroup,workerGroup).channel(NioServerSocketChannel.class)
				.option(ChannelOption.SO_BACKLOG, 1024).handler(new LoggingHandler(LogLevel.INFO))
				.childHandler(new ChannelInitializer<Channel>() {
					@Override
					protected void initChannel(Channel ch) throws Exception {
						//Inserts ChannelHandlers at the last position of this pipeline.
						ChannelPipeline p=ch.pipeline();
						p.addLast(new LineBasedFrameDecoder(1024));
						p.addLast(new StringDecoder());
						p.addLast(new NettyServerHandler());
					}
				});
			//创建通道绑定端口,sync方法等待future执行完毕
			ChannelFuture f=b.bind(PORT).sync();
			//等待connection关闭
			f.channel().closeFuture().sync();
		}finally{
			bossGroup.shutdownGracefully();
			workerGroup.shutdownGracefully();
		}
	}
}
```

至此服务端以及开发完毕，运行后可以使用telnet进行测试了



### 客户端

客户端的开发其实与服务端类似，也需要继承实现ChannelInboundHandlerAdapter并重写其中的方法

@NettyClientHandler.java

```java
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

public class NettyClientHandler extends ChannelInboundHandlerAdapter{
	private byte[] req;

	public NettyClientHandler() {
		super();
		req=("你好零度。"+System.getProperty("line.separator")).getBytes();
	}
	/**
	 * 与服务端建立连接时调用
	 */
	@Override
	public void channelActive(ChannelHandlerContext ctx){
		ByteBuf message=null;
		message=Unpooled.buffer(req.length);
		message.writeBytes(req);
		ctx.writeAndFlush(message);
	}
	/**
	 * 通道接受信息时调用
	 */
	@Override
	public void channelRead(ChannelHandlerContext ctx,Object msg){
		String body=(String)msg;
		System.out.println(body);
	}
	/**
	 * 通道读取完毕时调用
	 */
	@Override
	public void channelReadComplete(ChannelHandlerContext ctx){
		ctx.flush();
	}
	
	@Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        // Close the connection when an exception is raised.
        cause.printStackTrace();
        ctx.close();
    }
}
```

@NettyClient.java

```java
import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.LineBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;

public class NettyClient {
	public static void main(String[] args) throws InterruptedException {
		EventLoopGroup group=new NioEventLoopGroup();
		try{
			Bootstrap b=new Bootstrap();
			b.group(group).channel(NioSocketChannel.class)
				.option(ChannelOption.TCP_NODELAY, true)
				.handler(new ChannelInitializer<Channel>() {
					@Override
					protected void initChannel(Channel ch) throws Exception {
						ChannelPipeline p=ch.pipeline();
						p.addLast(new LineBasedFrameDecoder(1024));
						p.addLast(new StringDecoder());
						p.addLast(new NettyClientHandler());
					}
				});
			//连接到netty服务端
			ChannelFuture f=b.connect("localhost",8007).sync();
			f.channel().closeFuture().sync();
		}finally{
			group.shutdownGracefully();
		}
	}
}
```

