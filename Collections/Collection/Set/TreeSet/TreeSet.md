**概述**

1. TreeSet实现了Set接口，底层是TreeMap

***
**一、TreeSet类初始属性**
```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
{
    private transient NavigableMap<E,Object> m;

    // 与映射中的对象关联的虚拟Value值
    private static final Object PRESENT = new Object();
}
```
***
**二、TreeSet构造函数**

1. 无参构造，创建TreeMap空对象
```java
    public TreeSet() {
        this(new TreeMap<E,Object>());
    }
```
2. 传入比较器构造，自定义比较方法，元素按规则存储
```java
    public TreeSet(Comparator<? super E> comparator) {
        this(new TreeMap<>(comparator));
    }
```
3. 传入SortedSet对象构造，根据传入对象的比较器创建排序规则
```java
    public TreeSet(SortedSet<E> s) {
        this(s.comparator());
        addAll(s);
    }
```
4. 其它集合对象转化
```java
	public TreeSet(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```
***
**三、TreeSet扩容机制**

TreeSet底层是TreeMap，扩容机制与TreeMap一致

1. 新增元素
```java
    // 与映射中的Key关联的Value虚拟值
    private static final Object PRESENT = new Object();

    public boolean add(E e) {
        return m.put(e, PRESENT)==null;
    }
```
***
**四、TreeSet底层机制**	
1. TreeSet底层是TreeMap，类似于HashSet与HashMap的关系
2. 允许传入比较器Comparator对象进行自定义排序规则