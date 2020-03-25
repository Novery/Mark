# 1.8新特性
### 接口默认方法
+ 接口可以通过default关键字添加非抽象方法
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
  + forEach是并行的（输出结果不严格按照顺序，但效率快），forEachOrder是串行的
+ Java8中对java.util.Comparator 和 Map.Entry 增加了新的方法用来排序。可以对HashMap, HashSet, HashTable, LinkedHashMap, TreeMap, 甚至ConcurrentHashMap都可以排序。基本思路就是先拿到集合，可以用entrySet()方法得到。然后调用stream方法，里面就可以调用sort方法了
  + map.entrySet.stream().sorted(Collections.reverseOrder(Map.Enry.comparingByValue()))(对value进行排序)