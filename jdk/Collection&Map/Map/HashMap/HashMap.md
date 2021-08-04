**概述**

1. Map接口常用的实现类： HashMap、HashTable和Properties
2. HashMap是Map接口中使用频率最高的实现类
3. HashMap以Key-Value的方式来存储数据
4. Key不能重复，添加相同的key会替换原来相同key中对应的value值
5. HashMap不保证映射的顺序，因为底层是通过key确定以hash表的方式来存储
6. HashMap没有实现同步，线程不安全

***
**一、HashMap类初始属性**
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

    private static final long serialVersionUID = 362498820763181265L;

    // 默认初始容量(一定是2的幂)  
    // 1 左移 4 位，及 2^4 = 16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    // 最大容量(一定是2的幂) 
    static final int MAXIMUM_CAPACITY = 1 << 30;

    // 在构造函数中未指定负载因子时使用的默认负载因子 = 0.75
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    // 允许链表节点树化最小的临界值
    static final int TREEIFY_THRESHOLD = 8;

    // 
    static final int UNTREEIFY_THRESHOLD = 6;

    // 允许链表节点树化最小的数组容量(即hashtable的长度)
    static final int MIN_TREEIFY_CAPACITY = 64;
}
```
***
**二、HashMap构造函数**

1. 无参构造，不指定初始容量和负载因子(初始化为空，容量为0)
```java
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    public HashMap() {
        // 只初始化负载系数 = 0.75
        this.loadFactor = DEFAULT_LOAD_FACTOR; 
    }
```
2. 仅指定初始容量，负载系数默认
```java
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
```
3. 指定初始容量和负载系数
```java
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
```
4. 根据其它集合构造
```java
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
```

***
**三、HashMap扩容机制**

1. 新增元素
```java
    // put入口方法
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        // 初始化table数组、Node节点对象
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 当前对象table为空或长度为0，表示还未对table数组进行初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            // table数组初始化扩容，获得数组长度 = 16
            n = (tab = resize()).length;
        // 当前元素计算出的hash索引在table数组上没有元素
        if ((p = tab[i = (n - 1) & hash]) == null)
            // 将节点添加到数组对应索引上
            tab[i] = newNode(hash, key, value, null);
        // 当前元素计算出的hash索引在table数组上存在元素
        else {
            Node<K,V> e; K k;
            // 判断当前元素与原节点元素key的hash值和内容是否一致，一致时将原节点替换成新节点
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 新元素添加的节点为树化节点
            else if (p instanceof TreeNode)
                // 树新增元素
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // 新元素添加的节点为链表，和链表中的每个元素逐一比较，
            // 若均不相同，则在链表最后新增该元素，若相同则break
            else {
                // 循环链表
                for (int binCount = 0; ; ++binCount) {
                    // 当前遍历的节点为最后一个节点，将新节点挂在当前节点最后
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 满足树化条件则树化
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 当前元素与原节点元素key的hash值和内容一致
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    // 替换最新节点值
                    p = e;
                }
            }

            // LinkedHashMap预留方法
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 判断新增元素后是否需要扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
**扩容方法**

```java
    final Node<K,V>[] resize() {
        // 将当前table数组赋值给oldTab对象，用于记录扩容前数组信息
        Node<K,V>[] oldTab = table;
        // 当前数组容量
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        // 当前数组临界值
        int oldThr = threshold;
        // 新容量、新临界值
        int newCap, newThr = 0;
        if (oldCap > 0) {
            // 当前数组容量已超过最大容量
            if (oldCap >= MAXIMUM_CAPACITY) {
                // 当前数组达到最大临界值
                threshold = Integer.MAX_VALUE;
                // 不扩容，返回当前数组
                return oldTab;
            }
            // 数组按当前容量扩容2倍后小于最大容量 & 当前容量大于默认初始化容量
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                // 新临界值扩充为当前容量2倍
                newThr = oldThr << 1; // double threshold
        }
        // 当前临界值大于0，且当前容量小于或等于0
        else if (oldThr > 0) // initial capacity was placed in threshold
            // 当前临界值赋值给新容量，按当前临界值进行扩容
            newCap = oldThr;
        // 当前容量和临界值均小于或等于0，初始化构造（不指定参数情况构造HashMap后首次扩容情况）    
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 新临界值为0时根据新容量和负载系数计算出临界值
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            // 扩容后每个Node节点处理
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    // 每个Node重新计算hash并确定table中的新索引
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    // 红黑树化节点
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        // 链表顺序重新分配
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
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
***
**四、HashMap底层机制**	
1. HashMap底层维护了Node类型的数组table，默认为16
2. 当创建对象时，将负载系数（loadFactor）初始化为0.75
3. 当添加key时，根据key计算出的hash值得到在table中的索引。然后判断该索引位置是否已有元素，如果没有元素则直接添加。
   如果该索引处铀元素，则调用equals()方法判断key是否相等，若相等则将key对应的value值替换为最新的value值。如果不相等则需要判断是树结构还是链表结构，做出相应处理。如果添加时发现容量不够，则需要进行扩容。
4. 第一次添加，则需要扩容table容量为16，临界值（threshold）为12
5. 后续扩容，将容量扩充为当前容量的2倍，临界值也同时扩容为当前的2倍，扩容判断条件取当前元素个数，非实际table非空元素数组长度
6. 在java8中，如果一条链表的元素个数超过THREEIFY_THRESHOLD（默认8），且table的大小 >= MIN_THREEIFY_CAPACITY（默认64），则会将链表进行树化，树化为红黑树
