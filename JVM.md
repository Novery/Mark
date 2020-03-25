# 
+ String s = new String("abc")，这里创建了两个对象，"abc"存在常量池，new String("abc")存在堆上，s引用存在栈中，指向堆内存的对象

# jvm内存结构
+ 堆（新生代，老年代[调用次数多，大对象]）
  + 一切new出来的对象
+ 方法区（类信息，常量池，静态变量，jitSUV而编译后的代码）
+ java栈
  + 对象引用和基本数据类型
+ 本地方法栈（调用native方法，执行其他底层语言）
+ 程序计数器（线程私有,为了切换线程能正常执行）
+ 在JDK1.8版本废弃了f方法区，替代的是元空间,元空间并不在JVM中，而是使用本地内存

# 内存溢出 OOM
+ Java heap space（堆溢出）
  + 虚拟机分配的到堆内存空间已经用满
  + 是否有死循环或不必要地重复创建大量对象
  + 调整jvm参数，xms,xmx
+ PermGen space（方法区溢出）
  + 载入的class等信息超过了阀值
  + -XX:PermSize和-XX:MaxPermSize设置永久代大小即可
  + 记一次堆外内存泄漏
+ Thread Stack space（栈溢出）
  + 检查程序中递归调用是否未控制好出口导致递归死循环
  + jvm参数调整xss
+ 内存泄漏
  
# 内存泄漏
+ 无效对象不能被释放
+ 借助MAT等工具检查是否泄漏
+ 单列造成的内存泄漏
  + 如果一个对象不再需要被使用，而单例还持有该对象的引用，造成不能被正常回收
+ 各种连接
  + io连接，除非其显式的调用了其close（）方法将其连接关闭，否则是不会自动被GC 回收的
+ 内部类造成的内存泄漏
  + 非静态内部类会持有它们所属外部类的引用，一旦没释放可能导致一系列的后继类对象没有释放
+ 静态集合类引起内存泄漏，引用的对象不能被释放



# gc过程（分代回收策略）
+ 一般策略是分代回收，分为新生代（Eden,存活区0，存活区1，默认比例8：1：1），老年代
+ 一般是直接分配在Eden区，Eden区满后，minorGC，没gc掉的放在存活区0
+ 当存活区0满了后，将还活着的分配到存活区1
+ 以后Eden区执行minorGC，把没gc的放到存活区1
+ 存活区的数据来回切换n次，移到老年区，对应jvm参数:-XX
+ MaxTenuringThreshold控制，大于该值进入老年代
+ 标记-清除
+ 复制算法（新生代）
+ 标记-压缩（老年代）

# gc算法
+ 标记清除
  + 遍历所有gc root，标记所有可达对象，将未被标记的对象清除
  + 缺点：清除后内存分散
+ 复制算法
  + 新生代使用的算法
  + 将内存分为两块，每次只使用一块，在回收时，将正在使用的存活对象复制到未使用的内存块中，之后清除正在使用的内存块所有对象，交换两个内存的角色，完成垃圾回收
  + 缺点：需额外的空闲内存
+ 标记整理
  + 老年代使用的算法
  + 和标记清除法类似，不过回收时，会对所有存活的对象按地址排序，其他的清除
  + 弥补了标记清除内存分散的特点，同时也比复制算法节约空间，但有个排序整理过程，所以比较慢

  


# jvm调优
+ 调优策略：不要定义太多常量，占内存。存放在方法区，不会被gc
+ 堆内存 Xms初始值和Xmx最大值设成一致 因为gc会使内存趋向于初始值，减少gc次数
+ -XX:+PrintGC      每次触发GC的时候打印相关日志
+ -Xmn               新生代堆最大可用值
+ -Xss 设置最大调用深度（解决栈溢出）
+ MaxTenuringThreshold控制，大于该值进入老年代
+ 最大堆内存最小堆内存设置为一致，因为gc会靠向最小堆内存，导致平凡的full gc
+ Perm Space(永久代)，超出大小会OutOfMemory,方法区（方法区是jvm规范，永久代和元空间是实现），保存class、运行时常量池、字段、方法、代码、JIT代码等，在jdk1.7以前，在永久代，1.8元空间，在物理内存上
+ jstajstack(跟踪线程堆栈，执行到哪一步)jmap（看内存情况）jvm调优工具
+ JDK做了优化，当某个空指针异常在相同地方抛出多次，会除去堆栈信息抛出，导致无堆栈，这种在查日志时可扩大时间上的查询范围，找到带堆栈的日志，或者修改jvm参数禁用优化
+ 大对象会直接进入老年代，这个大小也是jvm配置的，默认不会直接进入老年代
```
-XX:+UseConcMarkSweepGC 激活CMS收集器
-XX:ConcGCThreads 设置CMS线程的数量
-XX:+UseCMSInitiatingOccupancyOnly 只根据老年代使用比例来决定是否进行CMS
-XX:CMSInitiatingOccupancyFraction 设置触发CMS老年代回收的内存使用率占比
-XX:+CMSParallelRemarkEnabled 并行运行最终标记阶段，加快最终标记的速度
-XX:+UseCMSCompactAtFullCollection 每次触发CMS Full GC的时候都整理一次碎片
-XX:CMSFullGCsBeforeCompaction=* 经过几次CMS Full GC的时候整理一次碎片
-XX:+CMSClassUnloadingEnabled 让CMS可以收集永久带，默认不会收集
-XX:+CMSScavengeBeforeRemark 最终标记之前强制进行一个Minor GC
-XX:+UseParNewGC Parallel是并行的意思，ParNew收集器是Serial收集器的多线程版本，使用这个参数后会在新生代进行并行回收，老年代仍旧使用串行回收。新生代S区任然使用复制算法。操作系统是多核CPU上效果明显，单核CPU建议使用串行回收器
-XX:PretenureSizeThreshold 的意思是超过这个值的时候，对象直接在old区分配内存
-XX:+DisableExplicitGC 
    应用本身在GC堆内的对象行为良好，正常情况下很久都不发生full GC； 
    应用大量使用了NIO的direct memory，经常、反复的申请DirectByteBuffer 
    使用了-XX:+DisableExplicitGC 
    满足以上三条java.lang.OutOfMemoryError: Direct buffer memory
-XX:MaxTenuringThreshold=3  该参数主要是控制新生代需要经历多少次GC晋升到老年代中的最大阈值
XX:SurvivorRatio，它定义了新生代中Eden区域和Survivor区域（From幸存区或To幸存区）的比例，默认为8，也就是说Eden占新生代的8/10
+ -XX:CMSInitiatingOccupancyFraction=70 是指设定CMS在对内存占用率达到70%的时候开始GC(因为CMS会有浮动垃圾,所以一般都较早启动GC);
+ -XX:+UseCMSInitiatingOccupancyOnly 只是用设定的回收阈值(上面指定的70%),如果不指定,JVM仅在第一次使用设定值,后续则自动调整
+ -XX:+PrintGCDetails
+ -XX:+TraceClassLoading 打印加载class信息
```
# 逃逸分析
+ 当一个对象在方法内被定义，可能被外部方法所引用，例如作为调用参数传递到其他地方中，称为逃逸分析
```
public static StringBuffer craeteStringBuffer(String s1, String s2) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb;
}
 
public static String createStringBuffer(String s1, String s2) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb.toString();
}
```
+ 第一段代码中的sb就逃逸了，而第二段代码中的sb就没有逃逸
+ 使用逃逸分析，编译器可做如下优化
  + 同步省略，一个对象被一个线程访问，无需同步（擦除同步代码块）
  + 堆分配转为栈分配，无需gc
  + 通过参数控制
  
# 担保机制
+ 当对象无法写入eden区，会发生新生代gc，对象可达则写入存活区0，如果存活区大小不够，则发生担保，直接写入老年代
 

# -verbose参数详解
+ .java -verbose:class 在程序运行的时候有多少类被加载！你可以用verbose:class来监视，在命令行输入java -verbose:class XXX  (XXX为程序名)你会在控制台看到加载的类的情况
+ https://blog.csdn.net/u011897392/article/details/53021287



# jconsole
+ -Dcom.sun.management.jmxremote
  -Dcom.sun.management.jmxremote.port=8011 (端口随意，没被占用就行)
  -Dcom.sun.management.jmxremote.ssl=false 
  -Dcom.sun.management.jmxremote.authenticate=false
  这样本地连接的问题也解决了，远程连接的问题也解决了。

# jmeter
+ 创建线程组，创建http请求，创建Aggregate Report报告
+ 从文本中读取参数 Thread ->add-> config-> csv
