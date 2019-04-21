
### List

+ list.set(index,element);指定位置替换元素，index从0开始
+ List.contains(对象)，确定某个对象是否在list中
+ List.remove(对象)需要移除某个对象
+ LinkedList实现了queue，作为队列，先入先出，pop取出offer，clear清除队列，add，remove

### Set

+ HashSet  最快
+ 如果需要关注顺序，可以使用TreeSet，按照比较结果排序，同理适用于MAP
+ 可以使用LinkedHashSet,按照被添加的顺序保存

### Map

+ concurrentHashMap hashtable K/V都不接收空值 hashmap K/V可以   get也不支持null
+ map containsKey containsValue
+ map.keySet  返回了所有键，在foreach中用于遍历map
+ values():方法是获取集合中的所有的值----没有键，没有对应关系，
+ entrySet()：
+ Set<Map.Entry<K,V>> entrySet() //返回此映射中包含的映射关系的 Set 视图。Map.Entry表示映射关系。entrySet()：迭代后可以e.getKey()，e.getValue()取key和value。返回的是Entry接口 。
+ Map.Entry<Class<? extends Pet>,Integer> pair : map.entrySet())

+ Map.Entry<Class<? extends Pet>,Integer> pair : map.entrySet())

### HashMap
#### 为什么用HashMap
+ 散列桶（数组+链表）,存储的三键值对映射
+ 采用了数据+链表的数据结构，能在查询和修改上结合了数组的线性查找和链表的寻址修改
+ 非线程安全的，更快
+ 可以接受null键和值，hashtable和concurrenthashmap则不能（equals()方法需要对象，hashmap处理过）
#### HashMap工作原理
+ put过程
+ 对key对象，调用hashcode()计算hash，放到对应桶，调用equals()判断桶中是否已存在，存在value对象替换旧值，不存在链表加一
+ key对象定义为final,可变的话hash值会改变，但map里按原hash存的，不一致
+ 当桶内数据小于8，为链表，当大于8，转为红黑树
+ get过程
+ 使用键对象的hashcode找到桶位置，调用equals方法找到链表中正确的结点，最终找到值对象
#### 红黑树的见解
+ 红黑树，平衡树，查询比二叉树快（二叉树特殊情况下会变成一条线性结构，层级深，查询缓慢）
+ 非红即黑
+ 根节点为黑
+ 叶子结点为黑空结点
+ 红的子节点为黑（反之不一定）
+ 从根结点到叶结点或空子节点的每条路径，包含相同数据的黑色节点
#### hash函数的实现
+需要根据hash算法求得对应数组桶的位置
+ 调用hashcode获取散列值
+ jdk1.8,转为32位的二进制，前16位和后16位低bit和高bit做异或
+ （n*1)&hash得到下标
#### 扩容
+ 默认负载因子大小为0.75，即当一个map填满75%的桶，就会创建两倍的桶数组，将原来的对象放入新的桶数组
#### 过程
+ 遍历老数组，重新计算hash，塞到新数组
+ 核心代码如下
```java
whie(e! = null){
    Entry next = e.next;
    int i = hash(e.hashcode()); //根据散列值重新计算下标
    e.next = newtable[i]; //队首插入（多线程下环链形成的原因，jdk1.8改为加入队尾），将新桶中的队首元素作为e的next
    newtable[i] = e; 
    e = next;
}
```
+ 死链的形成
+ 多线程情况下，一个桶内有keyA, keyA.next == keyB
+ 线程1执行到第一行被调度挂起，线程2执行完毕，队首插入，导致keyB.next == keyA,形成环链

### API

+ Collections.addAll，用于将一些元素添加到集合里，合并的前提是都是同一泛型或其扩展
+ ArrayList.addAll，更具体，针对list，两个能合并的前提是合并的前提是都是同一泛型或其扩展
+ 迭代器的使用，在remove时，最推荐使用迭代器，如果用集合自带的remove，从小到大索引会报错
+ 在迭代过程中，一定要调用一下iterator.next()
+ 集合，HashSet，b.addAll(a)，b.retain(a)，b.removeAll(a)集合的用法，第一个a和b合并在一起，第二个返回共用部分，第三个移除包含的元素
+ foreach可以用于任何数组或者其他任何Iterable (实现Iterable接口)

