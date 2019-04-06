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

#### Optionals
+   