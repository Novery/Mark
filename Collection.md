###List
+ list.set(index,element);指定位置替换元素，index从0开始
+ List.contains(对象)，确定某个对象是否在list中
+ List.remove(对象)需要移除某个对象
+ LinkedList实现了queue，作为队列，先入先出，pop取出offer，clear清除队列，add，remove
###Set
+ HashSet  最快
+ 如果需要关注顺序，可以使用TreeSet，按照比较结果排序，同理适用于MAP
+ 可以使用LinkedHashSet,按照被添加的顺序保存
###Map
+ concurrentHashMap hashtable K/V都不接收空值 hashmap K/V可以   get也不支持null
+ map containsKey containsValue
+ map.keySet  返回了所有键，在foreach中用于遍历map
+ values():方法是获取集合中的所有的值----没有键，没有对应关系，
+ entrySet()：
+ Set<Map.Entry<K,V>> entrySet() //返回此映射中包含的映射关系的 Set 视图。Map.Entry表示映射关系。entrySet()：迭代后可以e.getKey()，e.getValue()取key和value。返回的是Entry接口 。
+ Map.Entry<Class<? extends Pet>,Integer> pair : map.entrySet())
###API
+ Collections.addAll，用于将一些元素添加到集合里，合并的前提是都是同一泛型或其扩展
+ ArrayList.addAll，更具体，针对list，两个能合并的前提是合并的前提是都是同一泛型或其扩展
+ 迭代器的使用，在remove时，最推荐使用迭代器，如果用集合自带的remove，从小到大索引会报错
+ 在迭代过程中，一定要调用一下iterator.next()
+ 集合，HashSet，b.addAll(a)，b.retain(a)，b.removeAll(a)集合的用法，第一个a和b合并在一起，第二个返回共用部分，第三个移除包含的元素
+ foreach可以用于任何数组或者其他任何Iterable (实现Iterable接口)

