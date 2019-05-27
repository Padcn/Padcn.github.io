---
title:  JDK1.8中的 HashMap
date: 2018-08-22
tags: 
 - JDK源码
 - java
 - 集合
---
# Jdk1.8中的HashMap

hashMap在jdk1.8中有一次比较大的更新，相对之前的数据结构来讲引入了红黑树来提高查询效率。
在1.8之前的get方法，获取entry的下标索引很快，但是查找到entry之后，是需要遍历链表entry来查找对于的key，之后返回需要的值。这样查找时间复杂度取决与链表的长度。所以在1.8中优化成当链表长度大于8之后，会将链表转换为红黑树，这样可以降低时间复杂度。

存储数据的单元也改名字改为了Node，而红黑树的节点是TreeNode。
TreeNode继承自LinkedHashMap的Entry，而Entry继承自HashMap的Node对象。

## put方法
其实逻辑和jdk1.7差不多。
1. 判断table是否已经初始化了，如果没有则进行初始化。
2. 将(length-1)&hash来计算数组下标，获取具体的Node。
3. 如果Node为空则直接初始化一个链表，存入值。
4. 如果获取的Node不为空，则先看该节点地一个值是否就是匹配key的值，如果不是则判断是Node是红黑树的对象，还是正常链表的对象。
5. 如果是红黑书的对象，则调用红黑数的putTreeVal方法进行存值。
6. 如果是普通链表的对象则遍历，判断是新增的key还是已经存在，如果有旧值则覆盖，返回旧值，如果是新值则增加到链表的表尾，如果大于8,即此次加入的是第9个节点则转换为红黑树。
7. 最后modCount++后判断size是否超过了阈值，是否要进行扩容。

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
	// 判断是否需要初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
	// 判断table数组对于位置是否有Node，无则直接插入新节点，否则再判断
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
    	// 判断是否是红黑树的节点对象，是则调用红黑树的putTreeVal
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
    		// 普通链表则遍历链表查找判断
            for (int binCount = 0; ; ++binCount) {
        		// 最多遍历到末尾为止，如果还没有则将新节点放到表尾
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
        			// 这里如果插入后链表长度大于8了则转换为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
        		// 这里判断是否已经有旧值，如果有则中断便利，这样var10就是遍历到的节点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
    	// 如果var10不为空则说明有旧值，则进行替换旧值，返回旧值
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
	// 判断是否需要进行扩容操作
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

## resize 扩容方法
大致步骤：
1. 根据是否已经初始化进行判断，如果已经扩容过，则判断是否还可以扩容。再操作。
2. 根据初始化Map的构造函数来初始化容量和阈值。
3. 创建新的table，如果是初始化扩容，则直接返回新table，如果是已经有值的扩容，则根据节点类型进行迁移。
```java
/**
 * Initializes or doubles table size.  If null, allocates in
 * accord with initial capacity target held in field threshold.
 * Otherwise, because we are using power-of-two expansion, the
 * elements from each bin must either stay at same index, or move
 * with a power of two offset in the new table.
 *
 * @return the table
 */
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
    	// 如果到到最大容量则不在扩容了，如果容量在允许范围内，则扩容两倍
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY){
            // if中扩大数组大小，这里扩大阈值
            newThr = oldThr << 1; // double threshold
        }
    }
    else if (oldThr > 0) {
    	// 这里的情况是在第一次给了初始化容量时第一次调用put时
        newCap = oldThr;
    }else {               
    	// 这个情况在于调用默认构造函数时的逻辑，使用默认容量大小进行初始化
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 如果到这阈值还是0,则用容量和loadFactor进行计算。
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    // 初始化新的table
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 在旧table有值情况下迁移数据，否则直接返回新的table
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null){
                    // 该位置只有一个节点的情况下直接转移
                    newTab[e.hash & (newCap - 1)] = e;
                }else if (e instanceof TreeNode)
                	// 判断是否是红黑树的节点，如果是调用红黑树的迁移方式
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {
                	// 将原先的一条链表拆分为两条链表放到新的数组中
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 将拆分后的链表存入新table，两个链表索引位置相差oldCap
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

## get方法
get方法还是比较简单：
1. 根据key值计算hash
2. 根据hash和length-1进行与操作计算出，该节点在table中存放的位置。
3. 获得该位置的节点后对节点进行判断，分红黑树节点和普通链表进行查找。最终返回。
```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
final Node<K,V> getNode(int hash, Object key) {
	// 这里的hash是上面根据key计算得到的hash
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 这里通过(n - 1) & hash 来获取对于节点在table中的下标，first就是获取到的该位置的第一个节点
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 先判断看看是不是一个节点就是需要的值，是的话直接返回
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
        	// 判断是否是红黑树节点，是的话通过红黑树的getTreeNode进行查找,返回值
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 如果普通链表则遍历查找，和1.7没什么区别
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```
