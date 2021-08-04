**概述**

1. 

***
**一、TreeMap类初始属性**
```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
{
    // 确定树化规则的比较器
    private final Comparator<? super K> comparator;

    // 树的跟节点
    private transient Entry<K,V> root;

    // 树中节点的个数
    private transient int size = 0;

    // 修改次数
    private transient int modCount = 0;
}
```
***
**二、HashSet构造函数**

1. 无参构造，不指定排序规则
```java
    public TreeMap() {
        comparator = null;
    }
```
2. 比较器构造，自定义排序规则
```java
    public TreeMap(Comparator<? super K> comparator) {
        this.comparator = comparator;
    }
```
3. 其它集合对象转化构造
```java
    public TreeMap(Map<? extends K, ? extends V> m) {
        comparator = null;
        putAll(m);
    }
```
4. SortedMap对象构造
```java
    public TreeMap(SortedMap<K, ? extends V> m) {
        comparator = m.comparator();
        try {
            buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
        } catch (java.io.IOException cannotHappen) {
        } catch (ClassNotFoundException cannotHappen) {
        }
    }
```
***
**三、TreeMap扩容机制**
1. 新增元素
在TreeMap的put()的实现方法中主要分为两个步骤，第一：构建排序二叉树，第二：平衡二叉树。
```java
    public V put(K key, V value) {
        // 根节点
        Entry<K,V> t = root;
        // 根节点为空，首次添加元素
        if (t == null) {
            // Key非空检查
            compare(key, key);
            // 将当前节点设为树的根节点
            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // split comparator and comparable paths
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            do {
                parent = t;
                // 将当前节点与树的根节点进行比较，判断数据存放于树左侧还是右侧
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    // 相等时替换根节点数据
                    return t.setValue(value);
            } while (t != null);
        }
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        Entry<K,V> e = new Entry<>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }

    // 根据传入的比较器进行比较
    final int compare(Object k1, Object k2) {
        return comparator==null ? ((Comparable<? super K>)k1).compareTo((K)k2)
            : comparator.compare((K)k1, (K)k2);
    }
```
***
**四、TreeMap底层机制**	

TreeMap put()方法实现分析

在TreeMap的put()的实现方法中主要分为两个步骤，第一：构建排序二叉树，第二：平衡二叉树。对于排序二叉树的创建，其添加节点的过程如下：
1. 以根节点为初始节点进行检索。
2. 与当前节点进行比对，若新增节点值较大，则以当前节点的右子节点作为新的当前节点。否则以当前节点的左子节点作为新的当前节点。
3. 循环递归2步骤知道检索出合适的叶子节点为止。
4. 将新增节点与3步骤中找到的节点进行比对，如果新增节点较大，则添加为右子节点；否则添加为左子节点。
