# 前言
### 组件范围
+ 用于处理远程调用的一致API
+ 基于非阻塞NIO的公共组件
### 组件目标
+ http调用的基本封装
+ 异步调用的支持
# 基本概念
### 同步与异步（synchronous/asynchronous）
+ 同步是一种可靠的有序运行机制，当我们进行同步操作时，后续的任务是等待当前调用返回，才会进行下一步；而异步则相反，其他任务不需要等待当前调用返回，通常依靠事件、回调等机制来实现任务间次序关系
### 阻塞与非阻塞：
+ 在进行阻塞操作时，当前线程会处于阻塞状态，无法从事其他任务，只有当条件就绪才能继续，比如ServerSocket新连接建立完毕，或者数据读取、写入操作完成；而非阻塞则是不管IO操作是否结束，直接返回，相应操作在后台继续处理
# 结构
### 组件结构
+ com.hexin.utils4j.http.async.config.HttpAsyncConnectionPoolConfig
  + 初始化异步httpClient客户端
+ com.hexin.utils4j.http.async.entity.DispatcherContext
  + 组合了servlet3.0以后版本提供的异步上下文，通过继承此类，可实现自定义调度上下文
+ HexinFutureCallback
  + 执行回调
+ HttpAsyncProcessor
  + 向客户端提供的接口，支持post、get方式的异步调用 
### 组件依赖
#### HttpAsyncClient
+ 异步http客户端，httpclient在4.x之后开始提供基于nio的异步版本
#### HttpCore
+ 基于阻塞与非阻塞IO的底层组件
+ HttpCore NIO基于Reactor模式 
# Reactor模式
#### 经典阻塞IO模型
+ 入站->反序列化->计算->序列化->出站，整个流程在同一线程中处理，每一次连接请求占用一个线程

#### Reactor模式
+ 将入站出站（IO）剥离出来，反序列化->计算->序列化通过业务线程驱动
+ I / O反应器的目的是对I / O事件作出反应，并将事件通知分派到各个I / O会话
+ IO Reactors使用少量的分派线程（至少一个）将I/O事件通知分派给数量更多（通常多达数千个）的I/O会话和连接，建议每个CPU内核有一个调度（dispatch）线程
+ 通过卸载非IO操作提升Reactor线程处理能力
#### 调度（dispatch）线程
+ 事件方法中发生的处理不会太长时间阻塞分配线程
+ IOEventDispatch 接口 定义的通用I / O事件
  + connected：  在创建新会话时触发。
  + inputReady：  在会话有待处理的输入时触发。
  + outputReady：  在会话准备输出时触发。
  + timeout：  会话超时时触发。
  + disconnected：  会话终止时触发。
# 容器对异步支持
+ servlet3.0后提供了对异步的支持，使Servlet 线程不再需要一直阻塞，直到业务处理完毕才能再输出响应，最后才结束该 Servlet 线程。在接收到请求之后，Servlet 线程可以将耗时的操作委派给另一个线程来完成，自己在不生成响应的情况下返回至容器
+ ServletRequest实例提供的startAsync()开启异步，同时返回异步上下文（AsyncContext），存储了servlet原生的request与response，实例存储在线程副本，无需担心请求间数据读写串了，但要注意在子线程或线程池中获取的线程中无法获得该实例
+ AsyncContext实例提供的
  + complete()结束异步请求
  + dispatch（）重定位
  + setTimeout 异步超时时间，开启异步后，超过预设时间容器会抛出timeout异常
```
 @Resource
 private HttpServletRequest servletRequest
 ...
{
...
AsyncContext context = servletRequest.startAsync();
...
}

```
# 开始使用
#### 异步httpClient/httpProcessor初始化
```
@Bean
public HttpAsyncClient httpAsyncClientConfig() throws Exception{
    HttpAsyncConnectionPoolConfig httpAsyncConnectionPoolConfig = new HttpAsyncConnectionPoolConfig();
    httpAsyncConnectionPoolConfig.setHttpConfig(httpConfigConfig());
    return httpAsyncConnectionPoolConfig.init();
}

public HttpConfig httpConfigConfig(){
    ...
    return httpConfig;
}
@Bean 
public HttpAsyncProcessor httpAsyncProcessorConfig() throws Exception {
    HttpAsyncProcessor httpProcessor = new HttpAsyncProcessor();
    httpProcessor.setHttpAsyncConfig()
    httpProcessor.setHttpAsyncClient(httpAsyncClientConfig());
    return httpProcessor;
}
```
+ httpConfig中，异步特有的ioThreadCount指调度（dispatch）线程数量，selectInterval指会话超时检查的间隔时间，注意，过多的调度线程或者selectInterval设置过短而导致的频繁唤醒调度线程检查超时会导致cpu负载过高，建议线程数设置为cpu核数或者核数*2
+ HttpAsyncProcessor提供了对http调用的基本封装，并对外提供了统一的API，获取HttpAsyncProcessor实例，并通过提供的API执行异步调用
```
 @Resource
 private HttpAsyncProcessor httpAsyncProcessor
 ...
{
...
httpAsyncProcessor.sendAsyncRequest(routeConfig, postParam, hexinFutureCallback);
...
}


```
#### 编写自定义的调度上下文
+ 异步调用在执行时可能会涉及几次逻辑相关的请求-响应，需要特定的容器来保留状态信息，使应用程序能维持处理状态
+ 通过实现组件提供的抽象类DispatcherContext，可以很好的扩展除response，request外需要在请求周期中保持的对象，需要注意上下文中的内容，可能在多个线程之间共享，需要保证线程安全
```
public class ControlDiapatcherContext extends DispatcherContext{
    // 异步调用后响应的内容
    private List<ResultObject> result0bjectList = new Vector<>();
    
    public ControlDiapatcherContext (AsyncContext asyncContext) {
         super (asyncContext);
    }
    ...   
} 
```
#### HexinFutureCallback
+ httpcore提供了与netty类似回调的机制，提供了FutureCallback接口，组件在此基础上提供了HexinFutureCallback
+ HexinFutureCallback已提供了回调的简单实现
  + 当成功获取响应时，包括http code为非200的报文，执行completed方法，获取异步请求响应的实体对象response，解码，将结果冲刷到response，响应调用方，结束异步请求
  + 当请求过程中抛出异常时，包括超时等其他受检查或运行期异常，执行failed方法
  + 根据业务需求重写completed、failed
+ HexinFutureCallback组合了DispatcherContext
```
@Override
 public void completed(HttpResponse response){
    try {
        HttpEntity e = response.getEntity();
        PrintWriter pw = this.dispatcherContext.getAsyncContext().getResponse().getWriter();
        pw.print(EntityUtils.toString(e, "UTF-8"));
        pw.flush();
        pw.close();
        this.dispatcherContext.getAsyncContext().complete();
    } catch (Exception e) {
        e.printStackTrace();
    }
}

@Override
public void failed (Exception e) {
     e.printStackTrace();
}

```
