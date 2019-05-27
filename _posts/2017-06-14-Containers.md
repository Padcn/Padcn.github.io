---
title: JavaSE集合
date: 2017-06-14
tags: JavaSE
---
# JavaSE集合
> http://blog.csdn.net/zsw101259/article/details/7570033
> http://blog.csdn.net/u014136713/article/details/52089156


![CollectionMap](http://img.my.csdn.net/uploads/201205/15/1337080070_5795.jpg)

java中集合的关系大致上就是图中所示。

List和Set实现了Colection接口，
# Collection
 Collection接口继承了Iterable接口，因此可以使用迭代器进行遍历，是List和Set的上层。
 
 其中规定了一些常用的集合需要的方法，：
 ```java
  boolean add（object o）
  boolean remove（object o）
  int size()
  boolean isEmpty()
  boolean contains(Object o)
  Iterator iterator()
  boolean containsAll(Collection c)
  boolean addAll(Collection c)
  void clear()
  void removeAll(Collection c) 去除集合c中包含的元素
  void retainAll(Collection c) 去除集合c中不包含的元素
  Object[] toArray() 将集合转换为array
 ```
 Collection中不提供get方法，如果要遍历Collection中的元素则需要使用iterator。
 
 Collection的Iterator方法返回一个Iterator，Iterator接口以迭代的方式逐一的访问元素，与c++中的iterator不同的时这里的iterator并不是指向元素的，而是类似于处在两个元素之间的位置，调用next()方法越过下一个元素，并且返回那个元素。hasNext()返回是否存在下一个元素，remove()方法删除上一次访问的元素。
 
> 迭代器是 故障快速修复（fail-fast）的。这意味着，当另一个线程修改底层集合的时候，如
果正在用 Iterator 遍历集合，那么，Iterator就会抛出 
ConcurrentModificationException （另一种 RuntimeException异常）异常并立刻失败 
 
 ## List
 List接口继承自Collection接口，允许重复元素并且有序。
 ```java
 void add(int index,Object element)
 boolean addAll(int index,Collection c) //将c中元素加入到index位置
 Object get(int index)
 int indexOf(Object o)
 int lastIndexOf(Object o)
 Object remove(int index)
 Object set(int index,Object element) //设置指定位置，返回旧元素
 ListIterator listIterator()
 ListIterator listIterator(int index) 返回列表迭代器，从index位置开始访问
 List subList(int form Index,int toIndex)
```
以及一些从Collection中继承的方法。
其中listIterator()方法返回的是ListIterator，继承自Iterator，对比Iterator不同在于可以双向移动，Iterator只能使用next()进行向后遍历，ListIterator中可以使用perivious()方法进行向前遍历。相应的hasPervious()判断是否到达头部，previousIndex()返回下次调用previous()的索引。add(Object o)添加元素和set(Object o)设置元素。

> add元素后使用元素会被增加到隐式光标前。


对于LinkedList和ArrayList来说，如果需要随机访问，而不需要在尾部以外的地方插入数据的话，使用ArrayList比较好，而如果只需要按照顺序进行访问数据，并且需要在中间灵活的插入数据，则使用LinkedList比较好。两者都实现了Cloneable接口。
### LinkedList
链表常用方法：
```java
 void addFirst(Object o)
 void addLast(Object o)
 Object getFirst()
 Object getLast()
 Object removeFirst()
 Object removeLast()
```
### ArrayList

动态数组ArrayList类封装了一个动态分配的Object[]数组，每个ArrayList对象有一个capacity。这个capacity表示存储列表中元素的容器打容量。当元素添加到ArrayList时，其capacity在常量时间内自动增加。

```java
void ensureCapacity(int minCapacity) 将ArrayList对象容量增加minCapacity
void trimToSize()整理ArrayList对象容量为当前列表大小，减少对象存储空间。
```
### Vector
Vector是一种老的动态数组，是线程同步的，效率很低，一般不赞成使用。

对比ArrayList来讲，区别在于他是线程安全的，看源码中可以知道，他每个方法都加类synchronized，但是就是因为这样所以对比ArrayList在非多线程中他的效率是比较低的，而由于只是方法加了锁，并不能保证复合操作的线程安全性，所以应该尽量少用。
> 10、为什么Vector类认为是废弃的或者是非官方地不推荐使用？或者说为什么我们应该一直使用ArrayList而不是Vector

> 　你应该使用ArrayList而不是Vector是因为默认情况下你是非同步访问的，Vector同步了每个方法，你几乎从不要那样做，通常有想要同步的是整个操作序列。同步单个的操作也不安全（如果你迭代一个Vector，你还是要加锁，以避免其它线程在同一时刻改变集合）.而且效率更慢。当然同样有锁的开销即使你不需要，这是个很糟糕的方法在默认情况下同步访问。你可以一直使用Collections.sychronizedList来装饰一个集合。

> 　　事实上Vector结合了“可变数组”的集合和同步每个操作的实现。这是另外一个设计上的缺陷。Vector还有些遗留的方法在枚举和元素获取的方法，这些方法不同于List接口，如果这些方法在代码中程序员更趋向于想用它。尽管枚举速度更快，但是他们不能检查如果集合在迭代的时候修改了，这样将导致问题。尽管以上诸多原因，Oracle也从没宣称过要废弃Vector。
> http://blog.csdn.net/u014136713/article/details/52089156


 ## Set
 Set接口继承自Coolection，常见的实现有HashSet和TreeSet,set集合中不允许出现重复。
 ### HashSet
 HashSet是无序的
 ### TreeSet
 TreeSet是有序的。
 
 ### LinkedHashSet
 LinkedHashSet扩展了HashSet，如果想跟踪添加给HashSet的元素的顺序，LinkedHashSet实
现会有帮助。 LinkedHashSet的迭代器按照元素的插入顺序来访问各个元素。它提供了一个
可以快速访问各个元素的有序集合。同时，它也增加了实现的代价，因为 哈希表元中的各个
元素是通过双重链接式列表链接在一起的。 

 ## Queue
 队列尊需FIFO先进先出，LinkedList实现类Queue接口，所以LinkedList也可以被作为Queue来使用
 ```java
 boolean add(E e);
 boolean offer(E e); //功能同add，但是在受限制容量的情况下offer比较好。
  E remove();//检索并且删除队列头部，如果队列是空抛出异常
  E poll();//检索并且删除队列头部，如果队列是空返回null
  E element();//检索但不删除队列头部，如果队列是空抛出异常
  E peek(); //检索但不删除队列头部，如果队列是空返回null
 ```
 ### PriorityQueue
 > http://blog.csdn.net/hiphopmattshi/article/details/7334487

 java5中加入。
 通过Comparator进行排序，默认按自然顺序排列,也就是数字默认是小的在队列头，字符串则按字典序排列。如果需要自定义，则需要自己实现Comparator接口，作为参数传入PriorityQueue的构造器中。
 # Map
 Map接口用于维护键值对，描述了从不重复的键到值的映射。
 
 常用方法：
 ```
 Object put(Object key, Object value) 
 
Object remove(Object key)
 
void putAll(Map t)  // 将来自特定映像的所有元素添加给该映像 
 
void clear() //从映像中删除所有映射 

Object get(Object key) //获得与关键字key相关的值，并且返回与关键字key相关的对象，如果没有在该映像中找到该关键字，则返回null 
 
boolean containsKey(Object key)//判断映像中是否存在关键字key 
 
boolean containsValue(Object value)//判断映像中是否存在值value 
 
int size()//返回当前映像中映射的数量 
 
boolean isEmpty() 

Set keySet() // 返回映像中所有关键字的视图集 
  
Collection values()//返回映像中所有值的视图集 
 
Set entrySet() //返回Map.Entry对象的视图集，即映像中的关键字/值对 
  ```
 > 键和值都可以是null，但是不能把map作为键或者值添加给自身。

## Entry
Map的entrySet()方法返回一个实现了Map.Entry接口的对象集合，集合中每个对象都是底层Map中一个特定的键值对。
```java
 Object getKey() //返回条目的关键字 
 
 Object getValue() //返回条目的值 
 
 Object setValue(Object value) //将相关映像中的值改为value，并且返回旧值 
```

 ## HashMap
构造函数如下：
```java
HashMap() //构建一个空的哈希映像 
 
HashMap(Map m)//构建一个哈希映像，并且添加映像m的所有映射 
 
 HashMap(int initialCapacity)//构建一个拥有特定容量的空的哈希映像 
 
 
HashMap(int initialCapacity, float loadFactor)//构建一个拥有特定容量和加载因子
的空的哈希映像 
 ```
 > HashMap和Hashtable的区别：Hashtable是个过时的集合类，在java4中被重写，实现类Map接口。
 > 1. HashMap和Hashtable几乎等价，除了HashMap可以接受null值和键值，而Hashtable不行。
 > 2. HashMap是非synchronized，而Hashtable是synchronized，所以多个线程可以共享一个Hashtable,如果没有进行同步处理HashMap不能安全打在多线程环境下使用，所以java5提供了ConcurrentHashMap，时HashTable的替代，比Hashtable扩展性好。 所以在非多线程环境下HashMap会比HashTable效率高。
 > 3. HashMap不能保证随着时间的推移Map中打元素次序是不变的。
> <br>http://www.importnew.com/7010.html


如果需要让HashMap同步可以使用
```java
Map m=Collections.synchronizeMap(hashMap);
```

 ## TreeMap
 对比HashMap，TreeMap是有序的，实现了SortedMap接口。
 
 在Map 中插入、删除和定位元素，HashMap 是最好的选择。但如果您要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。使用HashMap要求添加的键类明确定义了hashCode()和 
equals()的实现。 
 
 TreeMap构造函数如下：
 ```java
//TreeMap没有类似HashMap的调优选项，因为该树总处于平衡状态。 
 
TreeMap()
 
TreeMap(Map m)//构建一个映像树，并且添加映像m中所有元素 
 
TreeMap(Comparator c) //构建一个映像树，并且使用特定的比较器对关键字进行排序 
 
TreeMap(SortedMap s) //构建一个映像树，添加映像树s中所有映射，并且使用与有序映像s相同的比较器排序 
 ```
 
 
 # Collections和Arrays
 
 Collections和Arrays是提供的工具类
 Collections中提供了一系列接受对应的容器返回一个线程安全的容器的方法如下，方法接受原容器也可以传入自定义的锁对象：
- synchronizedCollection
- synchronizedSet
- synchronizedSortedSet
- synchronizedNavigableSet
- synchronizedList
- SynchronizedRandomAccessList
- synchronizedMap
- synchronizedSortedMap
- synchronizedNavigableMap
 
 Collections类提供的sort方法可以自定义排序规则。
 ```java
 //方法一：在sort中传入比较器
 public static void main( String[] args )
    {
    	List<ObjForCompare> list=new ArrayList<ObjForCompare>();
       	list.add(new ObjForCompare(2,"2"));
    	list.add(new ObjForCompare(1,"1"));
    	list.add(new ObjForCompare(3,"3"));

    	Collections.sort(list,new Comparator<ObjForCompare>(){
			@Override
			public int compare(ObjForCompare o1, ObjForCompare o2) {
				if (o1.i>o2.i){
					return 1;
				}else if(o1.i<o2.i){
						return -1;
				}
				return 0;
			}
    	});
    	System.out.println(list);
    }
}
class ObjForCompare{
	public int i;
	public String str;
	
	public ObjForCompare(int i, String str) {
		super();
		this.i = i;
		this.str = str;
	}
	@Override
	public String toString() {
		return str;
	}
    
    //方法二：直接让比较的类实现Comparator接口
    class ObjForCompare implements Comparator<ObjForCompare>{
	public int i;
	public String str;
	
	public ObjForCompare(int i, String str) {
		super();
		this.i = i;
		this.str = str;
	}
	@Override
	public String toString() {
		return str;
	}
	@Override
	public int compare(ObjForCompare o1, ObjForCompare o2) {
		if (o1.i>o2.i){
			return 1;
		}else if(o1.i<o2.i){
				return -1;
		}
		return 0;
	}
 ```
 Arrays类主要提供了一些排序，查找，比较的方法，主要针对数组。
 
 Arrays.asList()和Collections.toArray()可以互相转换。
 
 
 
 
 
 
 
 
 
 
 
