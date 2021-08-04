**概述**

1. LinkedHashSet是HashSet的子类
2. LinkedHashSet底层是一个LinkedHashMap，底层维护了一个数组+双向链表
3. LinkedHashSet根据元素的hash值确定元素在数组上的存储位置（与HashMap原理一致），同时使用双向链表维护哥哥节点元素之间的顺序，保证顺序与插入顺序一致
4. LinkedHashSet不能存放重复元素
***
**一、HashSet类初始属性**
```java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {

    private static final long serialVersionUID = -2851667679971038690L;
}
```
***
**二、LinkedHashSet构造函数**

1. 不指定初始容量和负载因子，默认创建一个LinkedHashMap，初始容量16，负载因子0.75
```java
    public LinkedHashSet() {
        super(16, .75f, true);
    }
```
因为LinkedHashSet继承自HashSet()，故super()方法调用HashSet类中的构造方法，创建一个LinkedHashMap对象并默认
```java
    // 重载方法，dummy字段用于判断是否创建LinkedHashMap
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
```
2. 指定初始容量initialCapacity
```java
    public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }
```
3. 指定初始容量initialCapacity和负载因子loadFactor（参照HashMap）
```java
    public LinkedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor, true);
    }
```
4. 将其它集合对象转为HashSet
```java
    public LinkedHashSet(Collection<? extends E> c) {
        super(Math.max(2*c.size(), 11), .75f, true);
        addAll(c);
    }
```
***
**三、LinkedHashSet扩容机制**

LinkedHashSet底层为LinkedHashMap，扩容机制与HashMap一致，参照HashMap扩容机制

1. 新增元素
LinkedHashMap继承自HashMap，新增元素方法与HashMap一致
HashMap在putVal()方法预留了两个空方法供LinkedHashMap进行扩展，HashMap中的方法如下：
```java
    void afterNodeAccess(Node<K,V> p) { }

    void afterNodeInsertion(boolean evict) { }
```
LinkedHashMap中根据双向链表的特点对这两个方法进行重写
```java
    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }


    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
```

***
**四、LinkedHashSet底层机制**	
1. 在LinkedHashSet中维护了一个Hash表和双向链表（LinkedHashSet有head和tail，即双向链表中的头尾指针）
2. 每一个节点有pre和next属性，确定前一个和后一个节点从而形成双向链表
3. 在添加一个元素时，先求hash值，再求数组中的索引位置，确定该元素在hashtable（数组）中的位置，然后将添加的元素加入到双向链表中（判断元素是否重复与HashSet一致，使用equals()判断）
```java
    tail.next = newElement;
    netElement.pre = tail;
    tail = newElement;
```
4. 形成双向链表后，遍历时按链表顺序读取，就能保证元素插入的顺序与遍历顺序一致
