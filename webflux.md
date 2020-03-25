# 概念
### 同步与异步（synchronous/asynchronous）
+ 同步和异步关注的是消息通信机制 
+ 所谓同步，就是在发出一个*调用*时，在没有得到结果之前，该*调用*就不返回。但是一旦调用返回，就得到返回值了。
  换句话说，就是由*调用者*主动等待这个*调用*的结果
+ 而异步则是相反，*调用*在发出之后，这个调用就直接返回了，所以没有返回结果。换句话说，当一个异步过程调用发出后，调用者不会立刻得到结果。而是在*调用*发出后，*被调用者*通过状态、通知来通知调用者，或通过回调函数处理这个调用
### 阻塞与非阻塞：
+ 阻塞和非阻塞关注的是程序在等待调用结果（消息，返回值）时的状态.
+ 阻塞调用是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。
+ 非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程，可能会过几分钟check一下
# spring WebFlux与spring MVC
+ 在Spring MVC（通常是servlet应用程序）中，假定应用程序可以阻塞当前线程（例如，用于远程调用），因此，servlet容器使用大线程池来吸收请求期间的潜在阻塞。处理。
+ 在Spring WebFlux（通常是非阻塞服务器）中，假定应用程序不阻塞，因此，非阻塞服务器使用固定大小的小型线程池（event loop workers）来处理请求。
# Reactive Streams
+ Java中异步流处理的顶级概念：Reactive Streams，反应式流
### 起源
+ 异步编程并非全新的事物，java里典型的多线程处理就是异步编程
+ 异步编程存在几点问题
  + 回调地狱
  + 多个异步任务协同
  + 难以重构
+ Netflix，Pivotal和Lightbend中的工程师们，启动了Reactive Streams项目，希望为异步流(包含背压)处理提供标准
### 概念
+ Reactive Streams是一个API规范，类似JDBC，JDBC规范中，有DataSource接口，而Oracle JDBC实现了DataSource接口，MySql也实现了
+ 为我们提供了一个我们可以编写代码的API接口，而无需担心底层实现
+ https://github.com/reactive-streams/reactive-streams-jvm
+ 包含了如下接口
```java
//发布者
public  interface  Publisher < T > {
	public  void  subscribe（Subscriber <？super  T >  s）;
}

//订阅者
public  interface  Subscriber < T > {
	public  void  onSubscribe（Subscription  s）;
	public  void  onNext（T  t）;
	public  void  onError（Throwable  t）;
	public  void  onComplete（）;
}

//表示Subscriber消费Publisher发布的一个消息的生命周期
public interface Subscription {
	public void request(long n);
	public void cancel();
}

//处理器，表示一个处理阶段，它既是订阅者也是发布者，并且遵守两者的契约
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
	
}
```
### 目标
+ 管理跨异步边界的流数据交换 - 即将元素传递到另一个线程或线程池
  + 传统异步编程的写法，不同任务分别在不同的线程中执行，协调这些线程执行的先后顺序、线程间的依赖顺序是一件非常麻烦的事情，而Reactive Streams就是为了解决该问题
+ 引入了背压(Back Pressure)
  + 动态控制线程间消息交换的速率，避免生产者产生过多的消息，消费者消费不完等类似问题。
### stream
+ 流的定义：随着时间顺序排列的一组序列。一切皆是流(Everything is a stream)。我们可以把一组数据抽象为流(可以想象流是一个数组)，把对流中节点的逻辑处理，抽象成对节点的一步一步的处理，围绕该节点做加工处理，最终获得结果
+ 这跟工厂车间的流水线非常相似，发布者将半成品放到传送带上，经过层层处理后，得到成品送到订阅者手中
### 背压(back-pressure)
+ 背压是为了解决这个问题的：上游组件了过量的消息，导致下游组件无法及时处理，从而导致程序崩溃。
+ 对于正遭受压力的组件来说，无论是灾难性地失败，还是不受控地丢弃消息，都是不可接受的。既然它既不能应对压力，又不能直接做失败处理，那么它就应该向其上游组件传达其正在遭受压力的事实，并让它们降低负载。
+ 这种背压（back-pressure）是一种重要的反馈机制，使得系统得以优雅地响应负载，而不是在负载下崩溃。相反，如果下游组件比较空闲，则可以向上游组件发出信号，请求获得更多的调用。
### 具体实现框架
+ RxJava
+ Akka Streams
+ Ratpack
+ Reactor
  + Reactor是Pivotal提供的Java实现，它作为Spring Framework 5的重要组成部分，是WebFlux采用的默认反应式框架

# Reactor
### 概念
+ NIO Reactor，基于NIO中实现多路复用的一种模式，解决IO过程中同步阻塞导致的资源浪费，也是事件驱动的
+ Spring Reactor，本文的主题，实现Reactive Streams规范的反应式框架
### Reactor简介
+ 从设计模式的角度讲，Reactor相当于对观察者模式的延伸，在实际开发过程中，通常只会接触到 Publisher 这个接口，对应到 Reactor 便是 Mono 和 Flux。对于 Subscriber 和 Subcription 这两个接口，Reactor 必然也有相应的实现。但是，这些都是 Web Flux 框架用到的，如果不开发中间件，通常不会接触到
### 模块
+ Reactor 框架主要有两个主要的模块
  + reactor-core 负责 Reactive Programming 相关的核心 API 的实现 
  + reactor-ipc 负责高性能网络通信的实现，目前是基于Netty
### Mono与Flux
+ 在 Reactor 中，经常使用的类并不是很多，主要有以下两个：
  + Mono 实现了 org.reactivestreams.Publisher 接口，代表0到1个元素的发布者。
  + Flux 同样实现了 org.reactivestreams.Publisher 接口，代表0到N个元素的发表者。
+ Scheduler 表示背后驱动反应式流的调度器，通常由各种线程池实现。
# WebFlux
+ Spring 5 引入的一个基于 Netty 而不是 Servlet 的高性能的 Web 框架
+ 实际上后续的Spring Cloud Gateway，也是在WebFlux上做的封装
# 应用场景
+ 反应式编程无法大规模普及，一个很重要的原因是并不是所有库都支持反应式编程，当一些类库只能同步调用时，就无法达到节约性能的作用
+ Reactive Streams的推出统一了反应式编程的规范，并且已经被Java9集成。由此，不同的库可以互操作了，互操作性是一个重要的多米诺骨牌。
+ 例如，MongoDB实现了Reactive Streams驱动程序后，我们可以使用Reactor或RxJava来使用MongoDB中的数据。
  


