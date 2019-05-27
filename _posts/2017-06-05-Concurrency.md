---
title: Concurrency
date: 2017-06-05
tags: javase
---
# Concurrency
服务器编程的趋势是大容量，多线程。

在多核cpu的运行环境下，多线程有效提高程序的运行效率，如果在单核cpu环境下使用多线程，虽然会由于上下文切换拖慢效率，但是可以有效的防止程序阻塞。

实现方式
- 继承Thread
```java
Class MyThread extends Thread{
	@Override
    public void run(){
    	System.out.println("running");
    }
}
```
    
- 实现Runnable

```java
Class MyRunnable implements Runnable{
	@Override
    public void run(){
    	System.out.println("running");
    }
}

public static void main(String[] args) {
		new MyThread(new MyRunnable()).start();
}

```

- 实现Callable

```java

public class CallableTest implements Callable<Object> {
	private String name;
	
	public CallableTest(String name) {
		super();
		this.name = name;
	}

	@Override
	public Object call() throws Exception {
		return name+" return a name.";
	}

	public static void main(String[] args) throws InterruptedException, ExecutionException {
		ExecutorService service=Executors.newFixedThreadPool(2);
		Future<?> f1=service.submit(new CallableTest("c1"));
		Future<?> f2=service.submit(new CallableTest("c2"));
		
		System.out.println(f1.get().toString());
		System.out.println(f2.get().toString());
		
		service.shutdown();
	}
}
```

生命周期
1. 创建 ：new Thread（）
2. 就绪 ：调用start()方法
3. 运行 ：执行run()方法   sleep()/join()结束     yield()
4. 阻塞 ：sleep()/join()
5. 终止 ：任务执行完毕

wait()后进入等待队列， notify/notifyAll()进入锁池状态， 或者运行中synchronized，拿到锁后进入可运行状态

> stop方法强制结束，但是不建议用，会出现多个线程间数据不同步，线程没有执行完就被终止。一般在run方法中会有一个boolean值标志线程运行状态。

使用线程的start方法使得线程进入可运行状态，当抢到cpu资源后进入运行状态。
在运行中如果调用yeild()方法，则会让出cpu让重写抢占，进入可运行状态。
如果线程的run方法执行结束，则线程结束。
如果在运行状态的时候调用sleep或者另一个线程调用join方法，线程进入阻塞状态，sleep结束/另一个线程结束则进入就绪状态。
运行中锁对象调用wait，则进入等待队列，直到调用notify或notifyAll则进入。
发现锁被占用synchronized，则就会进入锁池，拿到锁则进入就绪状态。

> http://blog.csdn.net/jiafu1115/article/details/6804386


![线程状态](http://images.cnblogs.com/cnblogs_com/xuqiang/csharp/csharp_6.gif)


```java
// 锁住的是对象的实例，如果有多个实例，则这个锁就无效。
public synchronized void do(){}
//等效上，对比更加灵活
public void do(){
	synchronized(this){
    }
}

//多实例下
public static Object lock=new Object();
public void do(){
    synchronized(lock){
    }
}
```

# 一个简单的死锁示例

> http://blog.csdn.net/zhangliangzi/article/details/52729026

```java
class A {
	public synchronized void waitB(B b) throws InterruptedException{
		System.out.println(Thread.currentThread().getName() + "：正在执行a的等待方法，持有a的对象锁");  
        Thread.sleep(2000L);  
        System.out.println(Thread.currentThread().getName() + "：试图调用b的死锁方法，尝试获取b的对象锁");  
		b.dlB();
	}
	
	public synchronized void dlA(){
		System.out.println(Thread.currentThread()+"current");
	}
}
class B {
	public synchronized void waitA(A a) throws InterruptedException{
		System.out.println(Thread.currentThread().getName() + "：正在执行b的等待方法，持有b的对象锁");  
        Thread.sleep(2000L);  
        System.out.println(Thread.currentThread().getName() + "：试图调用a的死锁方法，尝试获取a的对象锁");  
		a.dlA();
	}
	
	public synchronized void dlB(){
		System.out.println(Thread.currentThread()+"current");
	}
}
public class DeadLock implements Runnable{
	A a=new A();
	B b=new B();
	
	public void init() {
		Thread.currentThread().setName("主线程");
		try {
			a.waitB(b);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		DeadLock dl=new DeadLock();
		Thread t=new Thread(dl);
		t.start();
		dl.init();
	}

	@Override
	public void run() {
		Thread.currentThread().setName("副线程");
		
		try {
			b.waitA(a);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		
	}
}
```
程序中，来个线程，主线程先执行a对象的waitB方法，进入sleep，副线程以非常小的时间差距进入waitA方法，也进入sleep，这个时候主线程sleep结束之后想要调用B类的加锁方法，但是这个时候副线程正持有b对象，需要等副线程的waitA方法执行完毕释放锁，但是副线程sleep结束后也会去想要获取正在被主线程持有的a对象的锁，所以这个时候两边都在等待对面释放锁，以至于发生了死锁。