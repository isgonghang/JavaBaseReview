LinkedList 是基于双向循环链表数据结构设计的，
**ArrayList 和 LinkedList 的区别**
1. 二者线程都不安全，但是效率比 Vector 的高；
2. ArrayList 底层是以数组的形式保存数据，随机访问集合中的元素比 LinkedList 快（LinkedList 要移动指针）；
3. LinkedList 内部以链表的形式保存集合里面数据，它随机访问集合中的元素性能比较慢，但是新增和删除时速度比 ArrayList 快（ArrayList 要移动数据）。



**一、LinkedList类初始属性**
```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;
}
```
LinkedList总共有三个初始变量，分别是：

**size：** 双向链表的节点个数
**first：** 双向链表指向头节点的指针
**last：** 双向链表指向尾节点的指针

*注意：first 和 last 是由引用类型 Node 连接的，这是它的一个内部类。*

**二、内部类Node节点**
```java
private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```
LinkedList 是通过双向链表实现的，而双向链表就是通过 Node 类来实现的，Node 类中通过 item 变量存储当前元素，通过 next 变量指向当前节点的下一个节点，通过 prev 变量指向当前节点的上一个节点。

**三、构造函数**
1. 无参构造方法
LinkedList 的无参构造就是构造一个空的 list 集合。
```java
    public LinkedList() {

    }
```
2. 有参构造方法

传入一个 Collection<? extends E> 类型参数，构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序。
```java
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```
**四、常用方法**
1. 新增元素：add(E e) 方法，将指定的元素**追加到此列表的末尾**
```java
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
```
其中，调用的 linkLast() 方法，设置元素 e 为最后一个元素：
```java
    void linkLast(E e) {
    	// 获取当前链表最后一个节点
        final Node<E> l = last;
        // 创建一个新节点，并将最后一个节点设为新增节点的prev节点
        final Node<E> newNode = new Node<>(l, e, null);
        // 将新节点赋值给最后一个节点，使新节点成为最后一个节点
        last = newNode;
        // 如果最后一个节点为null，则表示链表为空,将newNode赋值给first节点
        // (新增第一个元素时，first、last均指向第一个节点)
        if (l == null)
            first = newNode;
        else
        // 尾节点的last指向newNode。
            l.next = newNode;
        // 链表长度加1    
        size++;
        modCount++;
    }
```
2. add(int index, E element) 方法，在指定位置插入元素
```java
    public void add(int index, E element) {
    	// 首先检查索引 index 的位置，看下标是否越界
        checkPositionIndex(index);

        // 如果 index==size，直接在链表的最后插入元素，相当于 add(E e) 方法
        if (index == size)
            linkLast(element);
        else
        // 调用 node 方法将 index 位置的节点找出，接着调用 linkBefore 方法
            linkBefore(element, node(index));
    }

    // 检查索引 index 的位置
	private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

	private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }
```
在增加元素的时候，调用了 linkBefore() 方法，在非 null 节点 succ 之前插入元素 e：
```java
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        // 获取指定位置节点的前节点
        final Node<E> pred = succ.prev;
        // 创建新的节点，前节点为指定位置的前节点，后续节点为指定位置的当前节点，新增e元素就是插入在succ之前
        final Node<E> newNode = new Node<>(pred, e, succ);
        // 将新增元素节点设为原指定位置节点的前节点
        succ.prev = newNode;
        // 如果前节点为null，则把newNode赋值给first；否则构建双向列表
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```
