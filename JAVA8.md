# 1.8新特性
### 接口默认方法
+ 接口可以通过default关键字添加非抽象方法
### Lambda表达式
+ 一些语言一开始就支持Lambda,比如.net
+ 逗号分隔的参数列表、–>符号与函数体三部分表示
### 方法引用
+ 静态、构造（类::方法）
### 重复注解
+ 重复注解机制本身必须用@Repeatable注解
### 类型推断
+ 使用泛型接口时，原来需要指定泛型
### 类库的新特性
+ Optional
+ Stream，简化了集合框架的处理
  + Stream API不仅仅处理Java集合框架。像从文本文件中逐行读取数据这样典型的I/O操作也很适合用Stream API
  +  Stream< String > lines = Files.lines( path, StandardCharsets.UTF_8 )
### JavaScript引擎
+ 然而巨坑，导致了一次线上OOM
### ConcurrentHashMap
+ 大量的利用了volatile，final，CAS等lock-free技术来减少锁竞争对于性能的影响
+  内存位置（V）、预期原值（A）和新值(B)。如果内存地址里面的值和A的值是一样的，那么就将内存里面的值更新成B。CAS是通过无限循环来获取数据的，若果在第一轮循环中，a线程获取地址里面的值被b线程修改了，那么a线程需要自旋，到下次循环才有可能机会执行。
+ 1.7 分段锁+数组 1.8 放弃了Segment转而采用的是Node，其设计思想也不再是JDK1.7中的分段锁思想
+ 1.8结构上也加上了红黑树，意味着查询更快 
### HashMap
+ JDK1.7用的是头插法(并发情况下导致死链)，而JDK1.8及之后使用的都是尾插法
+ JDK1.7的时候使用的是数组+ 单链表的数据结构。但是在JDK1.8及之后时，使用的是数组+链表+红黑树的数据结构

### 元空间
+ 永久代被移除，换成元空间，使用直接内存

### Lambda表达式
#### 简化代码
+ 老版对java字符串排序
```java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
```
+ 1.8无需使用传统的匿名对象方式
```java
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
```
+ 对于函数体只有一行代码的，你可以去掉大括号{}以及return关键字
```java
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```
+ List本身有个sort方法，java编译器可以自动推导参数类型
```java
names.sort((a, b) -> b.compareTo(a));
```
+ (parameters) -> expression 或 (parameters) ->{ statements; }
+ (int x, int y) -> x + y  
+ 代替匿名内部类
```
// 1.1使用匿名内部类  
new Thread(new Runnable() {  
    @Override  
    public void run() {  
        System.out.println("Hello world !");  
    }  
}).start();  
  
// 1.2使用 lambda expression  
new Thread(() -> System.out.println("Hello world !")).start();  
```
+ 循环
```
List<Person> javaProgrammers = new ArrayList<Person>() {  
  {  
    add(new Person("Elsdon", "Jaycob", "Java programmer", "male", 43, 2000));  
    add(new Person("Tamsen", "Brittany", "Java programmer", "female", 23, 1500));  
    add(new Person("Floyd", "Donny", "Java programmer", "male", 33, 1800));  
    add(new Person("Sindy", "Jonie", "Java programmer", "female", 32, 1600));  
    add(new Person("Vere", "Hervey", "Java programmer", "male", 22, 1200));  
    add(new Person("Maude", "Jaimie", "Java programmer", "female", 27, 1900));  
    add(new Person("Shawn", "Randall", "Java programmer", "male", 30, 2300));  
    add(new Person("Jayden", "Corrina", "Java programmer", "female", 35, 1700));  
    add(new Person("Palmer", "Dene", "Java programmer", "male", 33, 2000));  
    add(new Person("Addison", "Pam", "Java programmer", "female", 34, 1300));  
  }  
};  

javaProgrammers.forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
```
#### 函数式接口
+ 仅包含一个抽象方法，有零个或多个非抽象方法（default关键字修饰）
+ 表达式实现唯一的抽象方法
```java
Convert<String, Integer> convert = (from) -> Integer.valueOf(from);
```
#### 方法和构造函数引用
+ 还可以通过引用表示
```java
Convert<String, Integer> convert = Integer::valueOf;
```
+ 非静态的话，则是对象::方法
+ 如果是构造，则是类名::new

#### Optional
+ 解决null安全问题的API
```
public static String getName(User u) {
    if (u == null)
        return "Unknown";
    return u.name;
}
```
==>
```
public static String getName(User u) {
    return Optional.ofNullable(u)
                    .map(user->user.name)
                    .orElse("Unknown");
}
```
+ Optional.of()或者Optional.ofNullable()：创建Optional对象，差别在于of不允许参数是null，而ofNullable则无限制
+ map(Function)：对Optional中保存的值进行函数运算，并返回新的Optional(可以是任何类型)
+ orElse(value)：如果optional对象保存的值不是null，则返回原来的值，否则返回value
```
public static String getChampionName(Competition comp) throws IllegalArgumentException {
    if (comp != null) {
        CompResult result = comp.getResult();
        if (result != null) {
            User champion = result.getChampion();
            if (champion != null) {
                return champion.getName();
            }
        }
    }
    throw new IllegalArgumentException("The value of param comp isn't available.");
}
```
==>
```
public static String getChampionName(Competition comp) throws IllegalArgumentException {
    return Optional.ofNullable(comp)
            .map(c->c.getResult())
            .map(r->r.getChampion())
            .map(u->u.getName())
            .orElseThrow(()->new IllegalArgumentException("The value of param comp isn't available."));
}
```
+ Optional可以用来检验参数的合法性
+ filter(Predicate)：判断Optional对象中保存的值是否满足Predicate，并返回新的Optional。
```
public void setName(String name) throws IllegalArgumentException {
    this.name = Optional.ofNullable(name).filter(User::isNameValid)
                        .orElseThrow(()->new IllegalArgumentException("Invalid username."));
}

```
#### Comparator接口实现排序
+ 对任意类型集合对象进行排序
+ 将此接口实现传递给Collections.sort或者Arrays.sort
+ 实现int compare(T o1,T o2),返回正数，零，负数分别表示大于，等于，小于
```
List<Student> stus = new ArrayList<Student>(){
			{
				add(new Student("张三", 30));	
				add(new Student("李四", 20));	
				add(new Student("王五", 60));	
			}
		};
		//对users按年龄进行排序
		Collections.sort(stus, new Comparator<Student>() {

			@Override
			public int compare(Student s1, Student s2) {
				// 升序
				//return s1.getAge()-s2.getAge();
				return s1.getAge().compareTo(s2.getAge());
				// 降序
				// return s2.getAge()-s1.getAge();
				// return s2.getAge().compareTo(s1.getAge());
			}
		});
		// 输出结果
		...
```
+ 使用lambda表达式简化
```
List<Student> stus = new ArrayList<Student>(){
			{
				add(new Student("张三", 30));	
				add(new Student("李四", 20));	
				add(new Student("王五", 60));	
			}
		};
		//对users按年龄进行排序
		Collections.sort(stus, (s1,s2)->(s1.getAge()-s2.getAge()));
```

#### stream
+ 自然序排序一个list
```
list.stream().sorted() 
```
+ 自然序逆序元素，使用Comparator 提供的reverseOrder() 方法
```
list.stream().sorted(Comparator.reverseOrder()) 
```
+ 使用Comparator 来排序一个list
```
list.stream().sorted(Comparator.comparing(Student::getAge)) 
```
+ 把上面的元素逆序
```
list.stream().sorted(Comparator.comparing(Student::getAge).reversed()) 
```
+ forEachOrdered和forEach
  + 
+ Java8中对java.util.Comparator 和 Map.Entry 增加了新的方法用来排序。可以对HashMap, HashSet, HashTable, LinkedHashMap, TreeMap, 甚至ConcurrentHashMap都可以排序。基本思路就是先拿到集合，可以用entrySet()方法得到。然后调用stream方法，里面就可以调用sort方法了
  + map.entrySet.stream().sorted(Collections.reverseOrder(Map.Enry.comparingByValue()))(对value进行排序)
+ parallelStream是并行执行的stream
+ List<String> collect = staff.stream().map(x -> x.getName()).collect(Collectors.toList());