---
title:  JDK1.7中的 HashMap
date: 2018-08-20
tags: 
 - JDK源码
 - java
 - 集合
---
# JDK1.7中的 HashMap
> 参考 Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析http://www.importnew.com/28263.html


1.7中的hashMap主要是由链表来实现的，在hashMap中实际进行存储的是一个叫table的数组，数组每个元素是Entry对象，每个Entry是一个链表。而数组的下标则是又key值的hash值和table的长度来决定的。

## put方法
1.首先判断是否是插入的第一个元素，如果是第一个元素，首先对存储的数组进行初始化，容量为2的n次方。

2.判断key是否为null，如果为null则将这个entry放在table[0]位置。

3.对key计算hash值，调用indexFor方法计算出key对应的table数组下标。

4.根据计算出的下标，获取Entry，进行遍历，查找是否已经存在key，如果存在替换旧值，返回旧值，如果不存在将此数据加入该链表中。

```java
pulic V put(K key,V value){
	if(table == EMPTY_TABLE){
    	// 如果map还没有初始化则进行初始化
    	inflateTable(threhold);
    }
    if(key == null){
    	// key值为空时处理，最终将数据放入table[0]
    	return putForNullKey(value);
    }
    // 计算hash以获取entry的下标。
    int hash=hash(key);
    int i=indexFor(hash,table.length);
    for(Entry<K,V> e=table[i] ; e != null ; e=e.next()){
    	// 判断是否有旧值需要替换，如果有就替换后返回旧值
        if(e.hash == hash && ((k=e.key)== key || key.equals(k))){
        	V oldValue=e.value;
            e.value=value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    // 走到这就是新的值了，加入获取到entry的链表节点中
    addEntry(hash,key,value,i);
}
```

inflateTable方法是map的初始化方法，当map初始化时调用
```java
private void inflateTable(int toSize){
	//这里容量为了保证是2的n次方
	int capacity=roundUpToPowerOf2(toSize);
	//计算扩容的阈值，等于 capacity * loadFactor
	threshold=(int)Math.min(capicity*loaderFactory,MAXIMUM_CAPACITY+1);
    table=new Entry[capicity];
    initHashSeedAsNeeded(capacity);
}
```
这里为什么要保证容量是2的n次方？

因为indexFor方法是 key的hash值与容量减一进行与操作，即(hash&(table.length-1))。如果非2的n次方，可能length-1就是的二进制最后一位为0,那么与任何值进行与操作后最后一位都为0,也就是最终得到的值为偶数，所以put的时候只会将entry存放进偶数的下标内，奇数下标的位置永远不会被存入值，等于浪费了另一半的容量。而如果是2的n次方就可以保证length-1为奇数，也就是最终得到的下标为奇数和偶数。就会相对均匀分布一些。

## 扩容：
> 超出容量会扩容，而HashMap的扩容比较消耗性能，默认容量为16,一般建议值为数量*loadFactor+1

  扩容发生在put方法时，上面的put方法最后一步中，addEntry方法中首先就是判断是否已经达到阈值，并且新插入的数组位置有值则需要进行扩容了。
```java
void addEntry(int hash,K key,V value,int bucketIndex){
	if(size>=threhold && null != table[buketIndex]){
    	// 扩容方法
        resize(2*table.length);
        hash=(null != key) ? hash(key) : 0;
        // 重新计算下标
        buketIndex=indexFor(hash,table.length);
    }
    // 链表中加入新值
    createEntry(hash,key,value,buketIndex);
}
// 将新值放在链表的表头
void createEntry(int hash,K key,V value,int buketIndex){
	Entry<K,V> e=table[buketIndex];
    table[buketIndex]=new Entry<>(hash,key,value,e);
    size++;
}
```
接下来是具体的扩容方法
```java
void resize(int newCapacity){
	Entry[] oldTable=table;
    int oldCapacity=oldTable.length;
    if(oldCapacity == MAXIMUM_CAPACITY){
    	threhold=Integer.MAX_VALUE;
        return;
    }
    Entry[] newTable=new Entry[newCapacity];
    //讲旧数组中的值转移到新数组。
    transfer(newTable,initHashSeedAsNeeded(newCapacity));
    table=newTable;
    threhold=(int)Math.min(newCapacity * loadFactory,MAXIMUM_CAPACITY +1);
}
```
由于是双倍扩容，所以原先table[i]中的节点会分拆到newTable[i]和newTable[i+oldLength]位置上。

## get方法
get方法比较简单，流程如下:

1.根据key值计算hash值

2.根据hash值和length调用indexFor方法计算出table的下标。

3.根据下标获得Entry链表后遍历根据key值比较获取value值。

```java
public V get(K key){
	if(null==key){
    	return getForNullKey();
    }
    Entry<K,V> entry=getEntry(key);
    return null == entry ? null : entry.getValue();
}
final Entry<K,V> getEntry(Object key){
	if(size == 0){
    	return null;
    }
    int hash=(key == null)?0:hash(key);
    // 根据hash值获取链表后遍历链表，查找对于的值
    for(Entry<K,V> e=table[indexFor(hash,table.length)];e!=null;e=e.next){
    	Object k;
        if((k=e.key) == key || (key != null && key.equals(e.key))){
        	return e;
        }
    }
}
```





