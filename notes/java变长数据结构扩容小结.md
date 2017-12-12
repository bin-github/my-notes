## java变长数据结构扩容小结：

---
> 线程安全
* 线程安全就是多线程访问时，采用了加锁机制，当一个线程访问该类的某个数据时，进行保护，其它线程不能进行
访问**直到该线程**读取完，其它线程菜可使用.不会出现数据不一致或者数据污染.
* 线程不安全就是不提供数据访问保护，多线程先后更改数据会产生数据不一致或者数据污染的情况.
* 一般使用synchronized关键字锁同步
---

### 1.StringBuffer（线程不安全），StringBuilder（安全）：两者的初始化和扩容机制是一样的
>1.初始大小 16。

```
 public StringBuffer() {
        super(16);
    }
```
>2.扩容策略：len * 2 + 2

```
	/**
     * This implements the expansion semantics of ensureCapacity with no
     * size check or synchronization.
     */
    void expandCapacity(int minimumCapacity) {
        int newCapacity = value.length * 2 + 2;
        if (newCapacity - minimumCapacity < 0)
            newCapacity = minimumCapacity;
        if (newCapacity < 0) {
            if (minimumCapacity < 0) // overflow
                throw new OutOfMemoryError();
            newCapacity = Integer.MAX_VALUE;
        }
        value = Arrays.copyOf(value, newCapacity);
    }
```
### 2.ArrayList：可变数组，底层使用object对象数组实现，线程不安全
>1.初始大小 0，但是只要增加一个元素，容量将会被置为10

```
public ArrayList() {
        super();
        this.elementData = EMPTY_ELEMENTDATA;
    }
```

```
public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
private void ensureCapacityInternal(int minCapacity) {
        if (elementData == EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    
// ==**扩容方法**==
  private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```


>2.扩容机制：如上面的grow方法，大小为原来的1.5倍

### 3.LinkedList：底层是链表实现，线程不安全
>1.初始大小：0

```
public LinkedList() {
    }
```
>2.扩容机制：每次只是增加一个节点

```
public boolean add(E e) {
        linkLast(e);
        return true;
    }
```

### 4.ArrayDeque：底层实现是队列，线程不安全
>1.初始大小：16

```
public ArrayDeque() {
        elements = (E[]) new Object[16];
    }
```
>2.扩容机制：len * 2

```
private void doubleCapacity() {
        assert head == tail; // 断言assert
        int p = head;
        int n = elements.length;
        int r = n - p; // number of elements to the right of p
        int newCapacity = n << 1;
        if (newCapacity < 0)
            throw new IllegalStateException("Sorry, deque too big");
        Object[] a = new Object[newCapacity];
        System.arraycopy(elements, p, a, 0, r);
        System.arraycopy(elements, 0, a, r, p);
        elements = (E[])a;
        head = 0;
        tail = n;
    }
```


### 5.Vector（线程安全），stack（线程安全）

>

类别|Vector |stack
---|---|---
初始大小 | 10 | 0
扩容机制 | len*2 |1

###注：以上的数据类型，在底层元素复制的时候全部使用的是system.arraycopy()，Arrays.copyOf底层使用的也是system.arraycopy(),system.arraycopy()是个native方法。

### 6.hashTable（线程安全）
>1.初始大小：11，填充因子0.75

```
 public Hashtable() {
        this(11, 0.75f);
    }
```
>2.扩容机制：len * 2 + 1

```
protected void rehash() {
        int oldCapacity = table.length;
        Entry<K,V>[] oldMap = table;
        // overflow-conscious code
        int newCapacity = (oldCapacity << 1) + 1;   // 扩容大小
        if (newCapacity - MAX_ARRAY_SIZE > 0) {
            if (oldCapacity == MAX_ARRAY_SIZE)
                // Keep running with MAX_ARRAY_SIZE buckets
                return;
            newCapacity = MAX_ARRAY_SIZE;
        }
        ……
    }
```
### 7.hashMap，线程不安全，LinkedHashMap，底层是新建一个hashmap，hashSet底层也是新建一个hashMap
>1.初始化大小，16，填充因子：0.75

```
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
static final float DEFAULT_LOAD_FACTOR = 0.75f;

 /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
    }
```
>2.扩容机制，len * 2

```
void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);    // 扩容机制
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }
        createEntry(hash, key, value, bucketIndex);
    }
```









