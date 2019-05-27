---
title: Unix网络编程卷一笔记（一）
date: 2017-10-15
tags: 网络编程
---

# Unix网络编程卷一笔记（一）
## 准备工作：

1. 在官网下载make所需要的库文件
http://www.unpbook.com/
下载后解压unpv13e.tar.gz文件
```bash
tar -zxvf unpv13e.tar.gz
```

2. 进入解压出的unpv13e目录根据README文件提示，进行make
```
./configure
cd lib
make
```

3. 在unp13e目录下生成了libunp.a文件复制到/usr/lib和/usr/lib32目录下
4. 将unpv13e目录下的unp.h的
```
#include "../config.h"
```
更改为
```c
#include "config.h"
```
然后将修改后的unp.h以及 unpv13e目录下的config.h复制进/usr/include目录中。

5. 测试编译：
之后的编译中需要加上静态连接库，
解压后的unpv13e/intro中有第一章的获取时间程序可以编译试试。-l参数链接静态连接库。
```
cd .intro
gcc daytimetcpcli.c -o daytimetcpcli -lunp
```
编译通过则表示配置成功。
> 参考 http://blog.csdn.net/chenhanzhun/article/details/41827241

## 获取时间的客户端
/intro/daytimetcpcli.c
```c
#include "unp.h"

int main(int argc,char **argv){        
    int sockfd,n;
    char recvline[MAXLINE+1];
    struct sockaddr_in servaddr;

    if(argc!=2)
        err_quit("usage:a.out <IPAddress>");    
    //sokect函数创建一个网际字节流套接字，
    //函数返回一个小整数描述符，以后所有函数调用都使用该描述符标识这个套接字。
    if( (sockfd=socket(AF_INET,SOCK_STREAM,0)) < 0 )
        err_sys("socket error");

    //清零结构后设置端口号
    bzero(&servaddr,sizeof(servaddr));
    servaddr.sin_family=AF_INET;
    servaddr.sin_port=htons(13);
    //inet_pton将命令行参数（这里是输入的ip）转换成合适的格式
    if(inet_pton(AF_INET,argv[1],&servaddr.sin_addr) < 0)
        err_quit("inet_pton error for %s",argv[1]);

    //与服务器建立连接，SA是书作者定义的为 struct sockaddr,是通用套接字地址结构。
    if(connect(sockfd,(SA *)&servaddr,sizeof(servaddr)) < 0)
        err_quit("connect error");

    //读取服务器应答
    while( (n=read(sockfd,recvline,MAXLINE)) > 0){
        recvline[n]=0;
        if(fputs(recvline,stdout) == EOF)
            err_sys("fputs error");
    }
    if(n<0)
        err_sys("read error");

    exit(0);
}
```

下面的Sokect函数以及Listen函数这些以大写字母开头的函数为作者自己封装进行处理的函数只是原函数进行了错误处理，如
```c
 if( (sockfd=socket(AF_INET,SOCK_STREAM,0)) < 0 )
        err_sys("socket error");
```
直接被定义成了
```c
int Socket(int family,int type,int protocol){
	int n;
	if( (n=socket(AF_INET,SOCK_STREAM,0)) < 0 )
        err_sys("socket error");
    return n;
}
```
书中称为包裹函数。

/intro/daytimetcpsrv.c
```c
#include "unp.h"
#include <time.h>

int main(int argc,char** argv){
    int listenfd,connfd;
    struct sockaddr_in servaddr;
    char buff[MAXLINE];
    time_t ticks;
	//创建套接字
    listenfd=Socket(AF_INET,SOCK_STREAM,0);
	//初始化
    bzero(&servaddr,sizeof(servaddr));
    servaddr.sin_family=AF_INET;
    servaddr.sin_addr.s_addr=htonl(INADDR_ANY);
    servaddr.sin_port=htons(13);
	//绑定端口
    Bind(listenfd,(SA*)&servaddr,sizeof(servaddr));
	//将套接字转换为一个监听套接字
    Listen(listenfd,LISTENQ);

    for(;;){
        connfd=Accept(listenfd,(SA*)NULL,NULL);

        ticks=time(NULL);
        snprintf(buff,sizeof(buff),"%.24s\r\n",ctime(&ticks));
        //返回时间
        Write(connfd,buff,strlen(buff));

        Close(connfd);
    }
}
```
## 查看网络接口信息
netstat -ni 显示网络接口的信息，-n标志以输出数值地址，而不是反向解析成名字。 lo为loopback环回接口，以太网接口称为eth0,

netstat -nr 展示路由表，其中n也是和上面相同作用。

ifconfig eth0显示接口详细信息。

找出本地网络中众多主机的ip地址方法之一是ping从ifconfig命令找出的广播地址。

> 单一UNIX规范第三版
POSIX 标准为 Portable Operating System Interface的首字母的缩写，可移植操作系统接口