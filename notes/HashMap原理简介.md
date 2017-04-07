### HashMap简介：
#### 1、实现：
> 1、类实现

```
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```
> 2、初始化：默认大小为16，负载因子为0.75

> 3、数据结构，底层是数组，数组的类型是hashMap自实现的Entry类（java8中是Node，Node implements Map.Entry），相当于一个链表，一般叫这类结构为"散列表"。

> 4、存储：put

```
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * Implements Map.put and related methods
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0) // 1
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null) // 2
            tab[i] = newNode(hash, key, value, null);
        else { 
            // 3
            .....
        }
    }
        
```
> 1、在2标记中，如果散列后table在改位置的值为空，则直接插入，无论key是否为空，

> 2、在3中处理发生碰撞的情况，主要逻辑是：
>> 1、heshcode和key值都相同，则直接覆盖旧值，

>> 2、hashcode相同，key值不同的，使用Entry链表，并且新值为链表的头元素

> 5、获取：get

>> 获取时基本就是put的逆过程

> 6、hashmap的大小规定为2的n次方，这样做的目的只要是消除碰撞和空间浪费问题

```
// 最后的构造方法
 public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
    
/**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```
#### 2、使用
> 1、线程不安全，put方法没有同步处理

> 2、扩容：在java8中是当前容量的两倍