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