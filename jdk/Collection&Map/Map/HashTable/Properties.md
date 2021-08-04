**Map接口工具实现类Properties**

1. Properties类继承自HashTable类并且实现了Map接口，也是通过K-V形式保存数据
2. 使用特点和HashTable一致，也使用synchronized修饰，线程安全
3. Properties主要用于读取配置文件，从xxx.properties文件中，加载数据到Properties对象，并进行读取和修改
4. xxx.properties通常作为应用参数配置文件