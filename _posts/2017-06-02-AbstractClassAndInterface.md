---
title:  抽象类和接口
date: 2017-06-02
tags: JavaSE
---

抽象类和接口作为实现多态的，
# 抽象类
抽象类使用关键字abstract进行修饰
```java
abstract class Base{
	int num=0;
    abstract void absTest();
    void normalTest(){
    }
}
```
抽象方法的修饰符只能是protected及更高开放性的修饰符。

抽象类可以有非抽象方法，但是有抽象方法的类必须要是抽象类。
# 接口
接口对于对象来说是能力，通过实现接口获取能力。对于方法来说是一种标准，符合标准的对象就可以进入方法。

为什么使用接口：
1. 扩展对象的形态，让对象具有更多的能力，加强程序扩展性，实现了一种java的多继承。
2. 对方法接口是一种标准，满足标准的对象可以进入方法（作为参数传入）。
例子
```java
interface IA{
	int num=0;
	void say();
}
class ImpIA1 implements IA{
	int num=1;
	@Override
	public void say() {
		System.out.println("Imp111");
	}
}
public class TestPlantForm {
	public static void main(String[] args) {
		runImp(new ImpIA1());
	}
	public static void runImp(IA ia){
		ia.say();
	}
}
```
	接口中的访问修饰符默认是public的，并且不能修改为其他，接口中属性默认修饰符为public static final.因为接口声明出来就是为了给其他类实现的。方法默认是public abstract的。

与抽象类不同的是接口中不能有普通方法，只能有抽象方法。
