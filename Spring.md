# Spring核心功能
#### 配置阶段
+ web.xml 配置dispatcherServlet（spring提供，完成IOC实例化、依赖注入等操作）
#### 初始化阶段，启动阶段
+ 加载spring配置文件
+ 声明IOC容器 map
+ 配置包路径，扫描到相关类
+ 扫描到的类，反射机制实例化，保持到IOC容器中
+ 依赖注入，ioc容器中需要赋值的属性赋值
#### SpringMvc
+ HandlerMapping
+ 将url对应的method保存起来
+ 封装一个内部类，list存起来
 
# Hystrix没生效 
+ 打断点发现未生成代理类
+ 检查后发现@EnableHystrix对应的版本低于其他相关包版本
+ 修改成大于后生效

# Eureka
+ 启动类注解标明为注册中心或服务提供方
+ prefer-ip-address 为true表明是服务提供方，需要注册到注册中心，注册中心填false
+ Renew：服务续约
  + Eureka客户会每隔30秒发送一次心跳来续约。 通过续约来告知Eureka Server该Eureka客户仍然存在，没有出现问题。 正常情况下，如果Eureka Server在90秒没有收到Eureka客户的续约，它会将实例从其注册表中删除。 建议不要更改续约间隔。
+ Fetch Registries：获取注册列表信息
  + Eureka客户端从服务器获取注册表信息，并将其缓存在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次



# rest+ribbon实现负载均衡
+ 定义restTemplate，加注解LoadBalanced(默认使用RibbonClientHttpRequestFactory),其构造也可接受其他factory
```
@Bean
	@LoadBalanced
	RestTemplate restTemplate(){
		return new RestTemplate();
	}
```
+ 调用使用restTemplate.getForObject/restTemplate.postForObject
+ 执行post/get时，调用factory的createRequest
+ 如果为默认的RibbonClientHttpRequestFactory，在此根据serviceId获取client，client
+ clientConfig如果不重写，ribbon提供默认的DefaultClientConfigImpl,properties和dynamicproperties记录了配置信息
+ 在archaius更新context上下文后，ribbon并没有显式的通过set去更新缓存中的配置，缓存位于ribbon提供的DefaultClientConfigImpl.dynamicProperties,其泛型设置为DynamicStringProperty,推测为通过获取引用直接更新property对象，按此思路最终定位位于其父类DynamicProperty的updateValue方法，更新stringValue
+ ribbon动态配置生效后，在实际调用时readTimeout还是旧值
  跟进源码，client在初始化时从配置上下文中获取到readTimeout、connectTimeout等和通信相关参数后，会缓存到自己的properties属性中，当archaius动态刷新ribbon的配置时，并未发生联动修改

+ issues RibbonClientHttpRequestFactory应该放开对timeout和readTimeout的修改
+ Ribbon的负载均衡，主要通过LoadBalancerClient来实现的，而LoadBalancerClient具体交给了ILoadBalancer来处理，ILoadBalancer通过配置IRule、IPing等信息，并向EurekaClient获取注册列表的信息，并默认10秒一次向EurekaClient发送“ping”,进而检查是否更新服务列表，最后，得到注册列表后，ILoadBalancer根据IRule的策略进行负载均衡。而RestTemplate 被@LoadBalance注解后，能过用负载均衡，主要是维护了一个被@LoadBalance注解的RestTemplate列表，并给列表中的RestTemplate添加拦截器，进而交给负载均衡器去处理

+ rest(rpc调用)+ribbon(负载均衡)
+ ribbon配置是懒加载的，第一次加载后ReadTimeout、ConnectTimeout写入并缓存在clinet中，且RibbonClientHttpRequestFactory并未开放对ReadTimeout、ConnectTimeout的修改，这块无法动态修改
+ @ConditionalOnMissingBean(XXX.class)在只有特定名称或者类型的Bean不存在于BeanFactory中时才创建某个Bean
+ netflix-archaius动态加载组件初始化时会实例一个ConfigurableEnvironmentConfiguration，导致自定义的configuration的没走，（自定义configuration加了ConditionalOnMissingBean）


# rpc解决方案
+ 在使用rest+ribbon时，发现耗时较nginx长，排查发现客户端使用的CloseableHttpClient慢，采用okhttp3
+ okhttp3，rest是支持的，自身带okhttprequestfactory生成okhttprequest，但问题在于request在执行时没走负载均衡客户端
+ 此处自定义了requestfactory和request，并在执行方法里走负载均衡客户端，okHttploadBalancingclient，springcloud提供
+ 遇到第一个问题，相关依赖没引入，基于懒加载的初始化和编译期都没发现
+ 调用时报错，下游参数校验为空，检查发现，post参数未传递，使用了uri拼接方式
+ 后证明使用uri拼接方式有问题，有些请求会报400，对于post请求，在合适时机初始化context，将请求体封装到entity，传入请求体长度（不穿长度会报异常，提示应传0k，实际值xxxk）
+ 再次发现socket close异常，直接原因是服务器或客户端在传输过程中主动断开连接导致，网上有说在请求头里写入keep-alive为true，补充了下这块的知识，报文是分请求行（协议信息）请求头（接受参数的类型，keep-alive这些参数也包括一些自定义的信息）和请求体，而在http1.1以后默认使用长连接，在跟代码的时候发现，对于okhttpclient，组合了一个connectionPool，除了keep-alive时间等信息主要结构是一个双端队列，在请求进来时，会先从队列里去取，如果没有则在请求后put一个进去供下次请求使用，但坑爹的负载均衡客户端（由springcloud提供）在每次请求时居然直接实例一个client，导致队列信息没共用，相当于每次请求都需要再经历一次三次握手，初步分析他每次实例化的原因是因超时时间等信息都杂糅在client里，导致不能使用单例（那为什么不用容器缓存呢，按config维度区分client啊，看hystrix，ribbon里面不都挺喜欢用缓存的），解决方案是重写负载均衡客户端，加入缓存，但这只解决了长连接问题，socket close再次观察发现规律，每次的请求时间都是5000毫秒，这个和我们设置的socket超时时间是对应上的，确定其实就是超时
+ 在线上运行后的25万请求里，出现了8个unexpected end of steam，直接原因是设置的contentLength过长，流读到一半已经没东西可读了


# bean
+ springboot 多模块注入bean ，访问不到，这是因为启动类默认扫描当前包路径下的bean
  + 修改包路径，保证模块间在同一包下
  + spring.factory


                    