#### 1、Java8的ConcurrentHashMap
> 1、底层数据结构是 tables[] + 链表/红黑树，去掉了jdk1.7中的segment段的结构，put时是在每一个数组元素上加锁，相当于每次加锁的部分只是在相同hash值的列表，这样更加提高了map的性能。

> 2、当一个hash值的长途次数大于8个的时候，链表将会被红黑树代替：

```
if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
```
> 3、内存结构图
![image](http://note.youdao.com/yws/api/personal/file/CEBF09A6EDC545279255511D2B07F214?method=download&shareKey=69df3c42a7f6451a42e323b470e27547)

> 4、源码：

```
public V put(K key, V value) {
        return putVal(key, value, false);
    }

    /** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            /** f = tabAt(tab, i = (n - 1) & hash)
                该调语句取到对应hash值的列表/树的头/根元素
            */
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    // 下面这个条件检查保证了在进入开块代码之前，f = tabAt(tab, i = (n - 1) & hash)又被别的线程改变
                    
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    /**
                        此处将列表转换成树
                    */
                    if (binCount >= TREEIFY_THRESHOLD)
                        /** 此方法的实现中有个点不理解：
                            if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
                                tryPresize(n << 1);
                        */
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```

> 2、

> 3、
