#### 1、hashMap
##### 1、内部方法解读：
> 1、计算最接近的2的次方数，tableSizeFor：

```
// 将目标数的每一位都无符号左移，并与之前的移动前的数做与运算，这样该目标数二进制的第一个1后面的数全部变为1，2的n次方-1，最后再加1，则为最接近的2的次方
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

> 1、
> 1、
> 1、

### 2、linkedHashMap
#### 1、如何保证顺序，
> 1、put方法是hashmap的方法，在没有键冲突的情况下是不会触发方法afterNodeAccess，即在各个键之间的顺序如何保证

> 2、顺序保证在新建节点的时候

```
// 类 haspMap
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
        // 此处如果是hashmap，则调用hashmap中的newNode方法，LinkedHashMap重写了该方法，以确保元素插入顺序
            tab[i] = newNode(hash, key, value, null);
        else {
            ……
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                // afterNodeAccess此方法也是被Linkedhashmap重写了，以保证元素顺序
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        // 次方法也重写，只是linkedHashMap没有做具体的逻辑，
        afterNodeInsertion(evict);
        return null;
    }
// 类 LinkedHashMap
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMap.Entry<K,V> p =
            new LinkedHashMap.Entry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }
```
