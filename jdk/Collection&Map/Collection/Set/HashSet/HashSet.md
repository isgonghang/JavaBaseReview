**概述**

1. HashSet实现了Set接口
2. HashSet实际上是HashMap，Key为存入元素值，Value默认为静态对象PRESENT
```java
	private static final Object PRESENT = new Object();

	public HashSet() {
        map = new HashMap<>();
    }
```
3. 可以存放null值，但是只能有一个null
4. HashSet不保证存放元素的顺序和取出的顺序一致，存放顺序取决于Key经过hash计算后在数组中的位置。
5. 不能存放重复元素
***
**一、HashSet类初始属性**
```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map;

    // 与映射中的Key关联的Value虚拟值
    private static final Object PRESENT = new Object();
}
```
***
**二、HashSet构造函数**

1. 不指定初始容量和负载因子，默认创建一个空的HashMap
```java
    public HashSet() {
        map = new HashMap<>();
    }
```
2. 指定初始容量initialCapacity
```java
    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }
```
3. 指定初始容量initialCapacity和负载因子loadFactor（参照HashMap）
```java
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }
```
4. 将其它集合对象转为HashSet
```java
	public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }
```
***
**三、HashSet扩容机制**
	HashSet底层为HashMap，扩容机制与HashMap一致，参照HashMap扩容机制
***
**四、HashSet底层机制**	
1. HashSet底层是HashMap
2. 添加一个元素时，先得到Hash值，转成索引值确定元素在数组中存放位置
3. 找到存储数据表table，看这个索引位置是否已经有存放的元素
4. 如果没有，则将新元素加入该位置
5. 如果有，则调用equals()方法比较，如果相同放弃添加，如果不相同则添加到该位置链表的最后一个节点
6. 在Jdk >= 1.8 时，如果一条链表的元素节点个数超过TREEIFY_THRESHOLD（默认为8）且table的大小 >= MIN_THREEIFY_CAPACITY（默认64），就会将链表转化为红黑树
7. 如果不指定初始容量和负载因子，第一次添加元素时，table数组默认扩容到16，负载因子为0.75
   临界值（THRESHOLD） = 容量（initialCapacity） * 负载因子（loadFactor） 
   即： 12 = 16 * 0.75 
   如果table数组使用到了临界值12，即集合中数组位置上的元素大于等于12时，就会出发扩容，扩容后大小为当前容量*2，即 16 * 2 = 32，新的临界值 = 32 * 0.75 = 24，后续扩容一次类推