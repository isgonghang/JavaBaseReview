**概述**

1. 存放的元素是键值对： K-V
2. HashTable的键和值都不能为null
3. HashTable使用方法和HashMap基本一致
4. HashTable线程安全，HashMap线程不安全
5. HashTable效率较低，HashMap效率较高

***
**一、HashTable类初始属性**
```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {

    // 初始化Hash表对象
    private transient Entry<?,?>[] table;

    //元素总个数
    private transient int count;

    // 临界值
    private int threshold;

    // 负载系数
    private float loadFactor;

    // 修改次数
    private transient int modCount = 0;

    private static final long serialVersionUID = 1421746759512286392L;
}
```

***
**二、HashTable构造函数**
1. 无参构造，默认容量11，负载系数0.75
```java
    public Hashtable() {
        this(11, 0.75f);
    }
```
2. 指定初始容量构造，负载系数默认0.72
```java
    public Hashtable(int initialCapacity) {
        this(initialCapacity, 0.75f);
    }
```
3. 指定初始容量和负载系数构造
```java
    public Hashtable(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal Load: "+loadFactor);

        if (initialCapacity==0)
            initialCapacity = 1;
        this.loadFactor = loadFactor;
        table = new Entry<?,?>[initialCapacity];
        threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    }
```
4. 其它Map对象转化构造
```java
    public Hashtable(Map<? extends K, ? extends V> t) {
        this(Math.max(2*t.size(), 11), 0.75f);
        putAll(t);
    }
```

***
**三、HashTable扩容机制**
1. 新增元素
```java
	// 使用synchronized修饰，保证线程同步安全
	public synchronized V put(K key, V value) {
        // value不能为空
        if (value == null) {
            throw new NullPointerException();
        }

        // 确保key不在散列表中
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        // 计算元素在hash表中的索引，与HashMap计算方法不一样
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        // 遍历链表，判断新增元素key是否和原节点元素相同
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        // 新增节点与原节点不相同，链表添加该节点	
        addEntry(hash, key, value, index);
        return null;
    }

    private void addEntry(int hash, K key, V value, int index) {
        modCount++;

        Entry<?,?> tab[] = table;
        // 元素数量超过阈值
        if (count >= threshold) {
            // 重新计算散列表，即扩容
            rehash();

            tab = table;
            hash = key.hashCode();
            // 根据key计算出的hash获取hash表中元素对应索引
            index = (hash & 0x7FFFFFFF) % tab.length;
        }

        // Creates the new entry.
        @SuppressWarnings("unchecked")
        Entry<K,V> e = (Entry<K,V>) tab[index];
        // 添加元素
        tab[index] = new Entry<>(hash, key, value, e);
        count++;
    }

    // 重新计算散列表，扩容
    protected void rehash() {
        int oldCapacity = table.length;
        Entry<?,?>[] oldMap = table;

        // 新容量 = 原容量 * 2 + 1
        int newCapacity = (oldCapacity << 1) + 1;
        if (newCapacity - MAX_ARRAY_SIZE > 0) {
            if (oldCapacity == MAX_ARRAY_SIZE)
                // Keep running with MAX_ARRAY_SIZE buckets
                return;
            newCapacity = MAX_ARRAY_SIZE;
        }
        Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];

        modCount++;
        threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
        table = newMap;

        // 遍历原散列表，根据原节点信息重新计算扩容后散列表对应的节点索引位置
        for (int i = oldCapacity ; i-- > 0 ;) {
            for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
                Entry<K,V> e = old;
                old = old.next;

                int index = (e.hash & 0x7FFFFFFF) % newCapacity;
                e.next = (Entry<K,V>)newMap[index];
                newMap[index] = e;
            }
        }
    }
```

***
**四、HashTable底层机制**	
1. 扩容计算方式与HashMap不一致，为原容量*2+1
2. HashTable线程安全，HashMap线程不安全，HashTable主要方法均用synchronized修饰