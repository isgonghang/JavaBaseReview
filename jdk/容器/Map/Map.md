**Map体系继承图**

-- Map
---- HashMap
------ LinkedHashMap
---- HashTable
------ Properties
---- SortedMap（接口）
------ TreeMap

**Map接口特点**

1. Map与Collection并列存在，Collection保存单列数据（即只有Key），Map保存双列映射关系数据（即Key-Value）
2. Map中的Key和Value可以是任何引用类型的数据，会封装到HashMap$Node对象中
3. Map中的Key不允许重复，重复会触发Value替换，而不是插入一对Key重复的键值对，Value值允许重复（因为只根据Key确定Hashtable中元素存放位置）
4. Map的Key，Value均可以为null，但Key为null只能有一个，Value为null可以有多个
5. 常用String类型作为Map的Key
6. Key和Value之间存在单向一对一关系，即可以通过指定的Key找到对应的Value
7. Map存放的数据以Key-Value形式存放在一个Node对象中，Node继承Map.Entry，Entry存放Node对象的引用，用于后续遍历等操作，不实际存放对象，实际对象存放在Node对象中（即Entry保存的是节点对象的地址引用，Key以Set形式保存，Value以Collection形式保存）
```java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```
***