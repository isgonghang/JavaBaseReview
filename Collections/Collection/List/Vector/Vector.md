**概述**

Vector与ArrayLIst类似, 内部同样维护一个数组, Vector是线程安全的. 方法与ArrayList大体一致, 只是加上 synchronized 关键字, 保证线程安全。

**一、Vector类初始属性**
```java
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
	// 存储的元素数组
    protected Object[] elementData;
    // 存储的元素个数
    protected int elementCount;
    // 扩容时的增加量，大于0增加capacityIncrement，否则默认以当前数组大小*2扩容
    protected int capacityIncrement;
}
```

**二、Vector类构造函数**
```java
    // 1.指定初始容量和扩容增量
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }

    // 2.创建初始容量集合，扩容增量默认为0
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }

    // 3.创建一个空的Vector，并且指定了Vector的初始容量为10(调用自定义初始容量方法)
    public Vector() {
        this(10);
    }

    // 4.根据其他集合来创建一个非空的Vector
    public Vector(Collection<? extends E> c) {
        elementData = c.toArray();
        elementCount = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
    }
```
**二、新增元素及扩容机制**

增加元素有两个主要的方法，第一个是在Vector尾部追加，第二个是在指定位置插入元素。
1. 在Vector尾部追加元素
```java
    public synchronized boolean add(E e) {
        modCount++;
        // 判断新增元素后是否需要扩容
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }


	private void ensureCapacityHelper(int minCapacity) {
        // 追加新元素后，当前数组容量为minCapacity
        // 如果新增元素后数组容量超过当前数组容量，则需要扩容
        //（此处elementData.length为当前数组最大容量，并非存储元素的个数，不能与list().size方法混淆）
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

	private void grow(int minCapacity) {
        // 获取elementData原始容量
        int oldCapacity = elementData.length;
        // capacityIncrement表示扩容容量，由构造函数传入
        // 如果扩容容量未指定大于0，默认以旧原始容量翻倍
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        // 初始扩容情况
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 将原数组数据拷贝到新数组中
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

ArrayList是线程非安全的，因为ArrayList中所有的方法都不是同步的，在并发下一定会出现线程安全问题。
另一个方法就是Vector，它是ArrayList的线程安全版本，其实现90%和ArrayList都完全一样，区别在于：

1、Vector增删改查方法均用synchronized修饰，是线程安全的，ArrayList是线程非安全的

2、Vector可以指定增长因子，如果该增长因子指定了，那么扩容的时候会每次新的数组大小会在原数组的大小基础上加上增长因子；如果不指定增长因子，那么就给原数组大小*2
```java
	int newCapacity = oldCapacity + ((capacityIncrement > 0) ? capacityIncrement : oldCapacity);
```