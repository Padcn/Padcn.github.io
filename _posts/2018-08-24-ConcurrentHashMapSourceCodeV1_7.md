---
title:  JDK1.7中的 ConcurrentHashMap
date: 2018-08-24
tags: 
 - JDK源码
 - java
 - 集合
---
# JDK1.7中的ConcurrentHashMap

ConcurrentHashMap是HashMap的考虑并发的版本，他支持并发的操作。

ConcurrentHashMap的结构是最外层是一个Segment数组，每个Segment中保存着之前我们熟悉的HashMap中的链表数组table，table数组中存放的是Entry链表。

Segment对象通过继承ReentrantLock来实现加锁操作。每次的加锁操作就是锁住整个Segment，保证每个Segment的线程安全就保证了全局的线程安全。

Segment初始化之后不能再扩容，而内部的table是可以扩容的，Segement的数量即为并发数，默认为16。

## 构造时初始化方法
初始化主要是对于ConcurrentHashMap的第一个Segment进行初始化，计算和设定一些初始值。大致流程为：
1. 判断传入参数（初始容量，loadFactor,concurrentLevel）是否在合理范围内，超出则设置为容许的最大值。
2. 计算并行级别(Segment的数量)循环左移1位，直到不小于初始化给的参数值，初始为1，这期间顺便记录下左移的位数用以计算segementShift。
3. 计算segmentShift和segmentMask，segmentShift=32-并行级别左移位数，segmentMask=并行级别-1。 这两个值后续操作有用。
4. 从最小容量循环左移1位直到满足不小于整体容量除以segment数量。计算每个Segement中的容量大小。
5. 用上述步骤计算出的每个容量初始化第一个Segement，并初始化Segment数组，将第一个Segment放入数组。

```java
public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {
    // 判断传入参数是否合法
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // 计算出2的幂次方不小于传入参数的容量(capacity)和并行级别（concurrencyLevel）
    int sshift = 0;
    int ssize = 1;
    // 计算并行级别 ssize
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    // 计算segmentShift和segementMask, 这两个值是后续要用计算Segment的下标。
    this.segmentShift = 32 - sshift;
    this.segmentMask = ssize - 1;
 	
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // 计算每个Segment中的Map的容量，保持2的幂次方并不小于给定初始化值
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    // 默认 MIN_SEGMENT_TABLE_CAPACITY 是 2
    int cap = MIN_SEGMENT_TABLE_CAPACITY; 
    while (cap < c)
        cap <<= 1;
    // 创建 Segment 数组，并创建数组的第一个元素 segment[0]
    Segment<K,V> s0 =
        new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    // 将初始化后的第一个Segment放入创建的数组中
    UNSAFE.putOrderedObject(ss, SBASE, s0); 
    this.segments = ss;
}
```
初始化完后除了Segement[0]不为null其他位置都还是null。
调用默认构造函数后初始Segement数组长度为16。segementShift=32-4=8,segmentMask=16-1=15,初始每个Segment中map容量为2,阈值为2×0.75,所以put第二个值时会进行扩容操作。

## put方法
put方法的大致步骤如下:
1. 先根据key和初始化时计算出的segmentShift和segmentMask来计算出需要put的Segment在Segment数组中的下标，并获取。
2. 获取到Segment对象后，获取Segment的独占锁，之后的操作基本和HashMap差不多了。
3. 根据key的hash值，获取HashEntry链表，然后遍历判断就行了。
4. 最后解锁，如果有旧值则返回旧值。

```java
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    /** 计算 key 的 hash 值，以初始化时计算的segmentShift和segmentMask进行计算Segement所在数组的下标。
     * 根据计算方式：
     * segmentShift=32-sgement数组长度2的几次方,所以hash右移，获得hash开头几位。
     * segmentMask=segement数组长度-1。
     * 最终计算得的j就是所需segement在数组中的下标
     */
    int hash = hash(key);
    int j = (hash >>> segmentShift) & segmentMask;
    // 如果该位置还是null，则调用ensureSegment(index)方法对该segment进行初始化。
    if ((s = (Segment<K,V>)UNSAFE.getObject
         (segments, (j << SSHIFT) + SBASE)) == null)
        s = ensureSegment(j);
    // 获取到需要操作的segment后，就可以进行put操作了
    return s.put(key, hash, value, false);
}

final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // 先获取segment的独占锁，获取锁后，接下来流程基本和HashMap中一样。
    HashEntry<K,V> node = tryLock() ? null :
        scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
    	// 根据key的table的length计算出需操作的Entry链表的表头索引位置。
        HashEntry<K,V>[] tab = table;
        int index = (tab.length - 1) & hash;
        HashEntry<K,V> first = entryAt(tab, index);
 		// 根据获取的链表表头，遍历链表判断操作
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                K k;
                // 在已经有旧值的情况下替换旧值，结束遍历
                if ((k = e.key) == key || (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            } else {
                // 如果获取到的entry的表头为空情况下，如果在获取独占锁操作时获取的node不为空，
                //则直接将改节点的node设置为该位置表头。否则直接初始化一个节点。
                if (node != null)
                    node.setNext(first);
                else
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1;
                // 这里判断是否超过了阈值，超过则扩容，否则将操作完的链表放入原位置。
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        // 解锁
        unlock();
    }
    return oldValue;
}
```
在put方法中如果获取的segement没有初始化，则需要先初始化，也就是上面put方法中的ensureSegment方法。
### 初始化Segement
主要是要判断是否已经被其他线程初始化了，这里使用的是CAS来解决这个问题。
```java
private Segment<K,V> ensureSegment(int k) {
    final Segment<K,V>[] ss = this.segments;
    long u = (k << SSHIFT) + SBASE; // raw offset
    Segment<K,V> seg;
    if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
        // 根据第一个已经初始化后的segment的的loadFacotry和table的length进行初始化。
        Segment<K,V> proto = ss[0];
        int cap = proto.table.length;
        float lf = proto.loadFactor;
        int threshold = (int)(cap * lf);
        // 初始化新segment中的table
        HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
        // 再判断一次该位置segment有没有被其他线程初始化，没有则new一个。
        if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
            Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
            // 这里使用CAS来将新的Segment来放入Segment数组。
            while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
                if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                    break;
            }
        }
    }
    return seg;
}
```
### 获取写入锁方法scanAndLockForPut
获取写入锁是在putVal方法的开始,这里已经获取到了对于的Segment了。

这个方法主要是为了获得segment的独占锁，附带如果获取锁失败时还查找了一遍key对应的节点。

```java
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    // 获取key对应的entry链表
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    // retries初始化为负数，表示正在查找key值对应节点。
    int retries = -1;
 	// 循环获取锁，在获取锁失败时先对链表进行查找。
    while (!tryLock()) {
        HashEntry<K,V> f;
        if (retries < 0) {
    		// 当retries值为负数时,表示正在查找key值对应节点。
            if (e == null) {
                if (node == null)
                	// 到这里表示该位置链表为空，没有元素，创建节点。另一个原因是该槽存在并发，不一定是该位置。
                    node = new HashEntry<K,V>(hash, key, value, null);
                retries = 0;
            } else if (key.equals(e.key)){
            	// 如果找了对于节点，则查找结束，retries标记为0
                retries = 0;
            } else {
                e = e.next;
            }
        } else if (++retries > MAX_SCAN_RETRIES) {
        	// 当重试次数超出了规定的最大值时，进入阻塞队列等待锁，lock方法获取锁后返回。
            lock();
            break;
        } else if ((retries & 1) == 0 && (f = entryForHash(this, hash)) != first) {
        	// 如果retries为0了，但是再次获取对于entry数组位置的表头已经和进入方法时不一致了，
            // 说明已经有新元素进入链表了，所以需要再重走一遍这个方法的流程。
            e = first = f; // re-traverse if entry changed
            retries = -1;
        }
    }
    return node;
}
```

### 扩容方法rehash
扩容是在put方法最后收尾阶段，如果判断加入新节点后长度大于阈值后发生。
```java
private void rehash(HashEntry<K,V> node) {
	// 这个参数node即是本次put的新节点
    HashEntry<K,V>[] oldTable = table;
    int oldCapacity = oldTable.length;
    // 扩容大小为原先的两倍
    int newCapacity = oldCapacity << 1;
    threshold = (int)(newCapacity * loadFactor);
    HashEntry<K,V>[] newTable = (HashEntry<K,V>[]) new HashEntry[newCapacity];
    int sizeMask = newCapacity - 1;
 
    // 遍历原数组，将原先的链表拆分成两个链表，之间下标相差oldCap的值。比如i和i+oldCap,i为原位置
    for (int i = 0; i < oldCapacity ; i++) {
        // e 是链表的第一个元素
        HashEntry<K,V> e = oldTable[i];
        if (e != null) {
            HashEntry<K,V> next = e.next;
            // 计算应该放置在新数组中的位置
            int idx = e.hash & sizeMask;
            if (next == null) {
                // 如果原链表只有一个元素，则直接放进新的table中
                newTable[idx] = e;
            } else {
                HashEntry<K,V> lastRun = e;
                int lastIdx = idx;
 
                // 循环该链表节点，找到需要拆分的链表节点。
                for (HashEntry<K,V> last = next;
                     last != null;
                     last = last.next) {
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }
                // 将这个节点放入新的table数组的新位置。
                newTable[lastIdx] = lastRun;
                // 将这个节点之前的节点都放到数组上对应的另一个位置。完成拆分链表。
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
            }
        }
    }
    // 将新节点放到新数组中
    int nodeIndex = node.hash & sizeMask;
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;
    table = newTable;
}
```

## get方法
get方法比较简单。
1. 计算出segment的位置，获取segment。
2. 计算出entry在segment中的位置，并获取链表。
3. 遍历链表查询。
```java
public V get(Object key) {
    Segment<K,V> s; 
    HashEntry<K,V>[] tab;
    int h = hash(key);
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    // 计算segment下标，并获取
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
        (tab = s.table) != null) {
        // 找到entry链表位置，遍历链表查询。
        for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                 (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
             e != null; e = e.next) {
            K k;
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }
    return null;
}
```








