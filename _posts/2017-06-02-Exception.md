---
title:  JavaSE-Exception
date: 2017-06-02
tags: JavaSE
---
# Exception
所有异常的根为java.lang.Throwable。

Throwable下分Error和Exception。这里我们讨论是的Exception。Error是当程序发生不可控的错误时，通常通知用户并且中断程序执行。Error是java虚拟机生成并抛出，程序不对其进行处理。

Exception总共来说分2种，一种是RuntimeException，其他类型都统称编译时异常。
## 编译时异常
编译时异常一般是在编译的时候编译器提示的异常也叫Checked异常。发生了这种异常则编译就不会进行下去。Java的设计哲学：没有完善错误处理的代码根本没有机会被执行。
> http://blog.csdn.net/woshixuye/article/details/8230407

常见的编译时异常有：
- Java.lang.ClassNotFoundException
- Java.lang.NoSuchMetodException
- java.io.IOException


## RuntimeException
运行时异常为程序运行时发生的异常

常见的有：
- IndexOutOfBoundsException 数组下标越界异常
- ArithmeticException	算数异常比如除零时
- NullPointerException 空指针异常
- NumberFormatException 数值转换异常
- java.io.FileNotFoundException 文件未找到


## 异常的处理
对于异常一般来说要么使用try..cathch进行捕获并处理，或者直接使用throws进行抛出到下一层，让下一层的人处理。

常见格式:
```
try{
	int a=1/0;
}catch(Exception e){
	e.printStackTrace();
}finally{
}
//catch可以有多个
try{
	int a=1/0;
}catch(ArithmeticException e){
	e.printStackTrace();
}catch(Exception e){
	...
}finally{
}
```
try和catch必须是一起出现，finally则不是必须的，catch可以有多个，如果有多个catch则其中的异常大小应该是范围比较小的在前面，范围大的在后面，如例子中ArithmeticException是在Exception的范围内，所以这样如果发生算数异常则捕获算数异常，运行第一个catch中的语句块，否则如果范围大的在前面那么总是不会运行到后面范围小的catch，这样不利于精确定位错误。

finally语句块主要是为了抛出异常时进行资源的回收和销毁，如数据库操作时，因为某些原因执行到中间发生异常，这个时候会中断程序，也就可能不会运行到关闭数据库连接的语句块。为了避免这种情况就使用finally语句块，finally块中的语句保证会执行。

> 如果finally中有return值，而catch块中也有return的值，那么最终返回的是finally中return的值。


## 自定义异常
如果觉得系统给定的异常不能够满足需求，想要自定义异常，可以继承Exception类，可以定义一个字符串类型的参数，给父类，最终是在Throwable类中的一个属性detailMessage，detailMessage在打印出异常时被打印，起到精确提示作用。

```java
class MyException extends Exception{

	public MyException(String msg) {
		super(msg);
	}
}
```
