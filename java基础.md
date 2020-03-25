# 内部类
+ 静态内部类可以有静态成员，而非静态内部类则不能有静态成员。 
+ 静态内部类的非静态成员可以访问外部类的静态变量，而不可访问外部类的非静态变量

# 值类型间的自动转型
+ 可以这样来定义一个double型的
```
  double d = 0.1;
```
+ 然而，这样不可以通过
```
  float f = 0.1;
```
+ 来定义一个float型的，这是因为默认的小数是double型的
+ 通常，表达式最终的数据类型是最大的决定的 一个int型加一个short型 最后结果是int型 如果要把值赋给较小的类型，就需要使用造型
```sql
byte 1字节
char 2字节
short 2字节
int 4字节
float 四字节
long 八字节
double 八字节
```

# HashBiMap
+ guava库
+ key都不同以外，value也都不同
+ inverse返回一个hashbimap反转，key和value反转


# InheritableThreadlocal
+ 父线程传递副本给子线程

 
# NPE
+ JDK做了优化，当某个空指针异常在相同地方抛出多次，会除去堆栈信息抛出，导致无堆栈，这种在查日志时可扩大时间上的查询范围，找到带堆栈的日志，或者修改jvm参数禁用优化
+ int a = jSONObject.getInteger 可能会导致空指针，如果jSONObject.getInteger解析出来结果为空，自动拆箱时会报错


# java基础
+ 父类静态块 -> 子类静态块 -> 父类构造块 -> 父类构造函数 -> 子类构造块 -> 子类构造函数

# string spilt不保证顺序
