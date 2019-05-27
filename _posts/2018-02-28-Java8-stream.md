---
title: Java8 stream
date: 2018-02-28
tags: java
---
# Java8 stream

> http://ifeve.com/stream/Java8 初体验（二）Stream语法详解

这篇只是笔记，具体还是看上面链接中的比较好

A sequence of elements supporting sequential and parallel aggregate operations. 

使用上类似于原先的iterator

使用上的例子为

```java

List<Integer> listArrays.asList(1,null,null,2,3);
long count=list.stream().filter(num->num!=null).count();
System.out.print(count);
```

这里是一个计算list中不为空元素的数量的代码。

list.stream()为获取stream，filter()函数为过滤掉匹配规则的元素，count()函数则是聚合结果。

创建Stream：

```java
//of方法
Stream<Integer> intStream=Stream.of(1,2,5,4,3);
//generator方法 ,这里生成的Stream是无限长的，懒加载的，
//一般配合limit(int)方法使用
Stream.generate(()->Math.random());
Stream.generate(Math::random);
Stream.generate(new Supplier<Double>(){
    @Override
    public Double get(){
        return Math.random();
    }
});
//iterate方法，直接使用iterate方法以获取的也是无限长的
Stream.iterate(1, item -> item + 1).limit(10).forEach(System.out::println);
```

对于原先有的容器

在Collection<E>接口中有定义

```java
default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
```

可以直接使用接口的这个方法获取如：

```
List list=new ArrayList<String>();
Stream<String> stream=list.stream();
```

### 转换

1. distinct：去重
2. filter:过滤
3. map:转换
4. flatMap:与map类似
5. peek：生成包含原Stream的新Stream，同时产生一个消费函数
6. limit:截断操作，获取前n各元素
7. skip:返回丢弃前n个元素后的stream。

实例：

```java
List<Integer> nums = Lists.newArrayList(1,1,null,2,3,4,null,5,6,7,8,9,10);
System.out.println(“sum is:”+nums.stream().filter(num -> num != null).
   			distinct().mapToInt(num -> num * 2).
               peek(System.out::println).skip(2).limit(4).sum());
   
```

值得注意的是这里转换看似经历了很多次，其实只会在最终聚合时一起进行。并不是一次一次分开转换的。

### Reduce

聚合：

在对Stream进行转换后得到的还是Stream(),而最终进行聚合才输出有意义的结果，如上面例子中的count()统计，sum()求和。

```java
//聚合为容器 
//第一个参数为生成结果的工厂类
//第二个参数为处理函数 list为第一个参数的返回结果，这里结果为一个ArrayList
//第三个参数为并发时使用
Stream<String> strStream=Stream.of("a",null,"b","d",null,"c");
		List result=strStream.filter(str->str!=null).collect(
				()->new ArrayList<String>(),
				(list,item)->list.add(item),
				(list1,list2)->list1.addAll(list2));
//这里还有一种更加简便的方式
//传入Collector<? super T, A, R> collector
List result=strStream.filter(str->str!=null).collect(Collectors.toList());
```



```java
Stream<String> strStream=Stream.of("a",null,"b","d",null,"c");
//reduce有两种实现，一种是直接提供BinaryOperator 前一个参数为返回参数，后一个参数为遍历Stream中的元素。
//这里reduce函数是第二种，接受两个参数，的第一个参数是可选的，为返回值的初始值。
System.out.println(strStream.reduce("rua",(sum,item)->sum+item));
```


