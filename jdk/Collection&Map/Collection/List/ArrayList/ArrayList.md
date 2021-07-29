ArrayList是基于数组的数据结构，ArrayList是以数组的方式存放数据的，Object[]，如下所示
```
private static final Object[] EMPTY_ELEMENTDATA = {};
```

**ArrayList构造函数**
首先分析构造函数，ArrayList共有三个构造函数，分别为：

1. ArrayList()构造一个初始容量为0的空列表（新增元素时才进行扩容，此处的elementData默认赋值的是一个空列表）
   
```
	private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

	public ArrayList() { this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA; }
```
2. ArrayList(int initialCapacity)构造一个具有指定初始容量的空列表，此处初始容量由调用方赋值
```
	private static final Object[] EMPTY_ELEMENTDATA = {};

	public ArrayList(int initialCapacity) {
		// 初始容量大于0，创建具有初始容量个数的对象数组
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        // 初始容量为0，赋值空列表，类似于无参构造方法     
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        // 初始容量小于0，抛出参数异常    
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```
3. ArrayList(Collection<? extends E> c)构造一个包含指定collection的元素的列表，这些元素是按照该collection的迭代器返回它们的顺序排列的。
```
	public ArrayList(Collection<? extends E> c) {
        // 将入参集合转化为数组
        elementData = c.toArray();
        // 数组非空
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            // 转换数组方法可能不返回对象类型的数组
            if (elementData.getClass() != Object[].class)
                // 采用数组拷贝方法，拷贝出新对象数组
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            // 转化数组为空，赋值空列表
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```
***
**ArrayList扩容机制**
接下来分析新增一个元素时，ArrayList的扩容机制，ArrayList新增元素主要方法如下：

```
    public boolean add(E e) {
        // 此处size为当前集合元素个数，size+1计算新增元素后的集合大小，确认是否需要扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 将元素放入下一个空值位置
        elementData[size++] = e;
        return true;
    }

    private void ensureCapacityInternal(int minCapacity) {
    	// 当前集合是否为默认空集合
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        	// 集合赋默认初始值（新增元素后的大小，此处还未扩容，只为计算是否需要扩容）
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
    	// 记录集合修改次数
        modCount++;

        // overflow-conscious code
        // 新增元素后的集合大小(当前元素集合大小+新增元素个数) - 当前元素集合大小
        // 若大于0则表示新增元素后将超出当前集合容量，需要进行扩容
        // 首次新增元素，空集合扩容后也走此方法，此处minCapacity为默认大小10
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    private void grow(int minCapacity) {
        // overflow-conscious code
        // 记录未扩容前集合元素个数
        int oldCapacity = elementData.length;
        // 新集合容量 = 未扩容前集合容量 + 未扩容前集合容量以二进制方式右移一位(即oldCapacity/2)
        // 即最终扩容大小 = 未扩容前集合容量 * 1.5
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 首次扩容情况，新容量赋值初始容量
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // 超出集合最大容量
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        // 通过数组拷贝形式，按新集合容量扩展数组长度
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

***
**ArrayList序列化问题**
```
transient Object[] elementData;
```
ArrayList在初始化elementData时，将元素用transient修饰，意味着elementData数组不能被序列化
这是因为序列化ArrayList的时候，ArrayList里面的elementData数组未必是满的，比如elementData的大小为10，但只用了其中的3个，显然没有必要序列化整个elementData，因此ArrayList中重写了writeObject方法：
```
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```
每次序列化的时候调用这个方法，先调用defaultWriteObject()方法序列化ArrayList中的非transient元素，elementData这个数组对象不去序列化它，而是遍历elementData，只序列化数组里面有数据的元素，这样一来，就可以加快序列化的速度，还能够减少空间的开销。