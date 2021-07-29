首先分析构造函数，ArrayList共有三个构造函数，分别为：

1、ArrayList()构造一个初始容量为0的空列表（新增元素时才进行扩容）
```
public ArrayList() { this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA; }

```