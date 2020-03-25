# Hystrix没生效
+ 打断点发现未生成代理类
+ 检查后发现@EnableHystrix对应的版本低于其他相关包版本
+ 修改成大于后生效
                                                
                            
+ 解析
  + 语料数据按“_”分割， 语句_分类_二级分类_source_权重
  + dataLocation除了指定数据的位置，也有关于分类权重等的默认配置
  + 语句插入时，同时存储charMap,语句的索引号，和每个
# HashBiMap
+ guava库
+ key都不同以外，value也都不同
+ inverse返回一个hashbimap反转，key和value反转

# Eureka
+ 启动类注解标明为注册中心或服务提供方
+ prefer-ip-address 为true表明是服务提供方，需要注册到注册中心，注册中心填false
+ defaultZone写法有问题，拼接了多个url，导致解析url有问题，连不上注册中心
+ ConnectTimeout 建立连接所用的时间
+ ReadTimeout 建立连接后从服务器读取到可用资源所用的时间

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


# websocket
+ WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端

# jps
+ 查看java进程PID

# Eureka
+ Renew：服务续约
  + Eureka客户会每隔30秒发送一次心跳来续约。 通过续约来告知Eureka Server该Eureka客户仍然存在，没有出现问题。 正常情况下，如果Eureka Server在90秒没有收到Eureka客户的续约，它会将实例从其注册表中删除。 建议不要更改续约间隔。
+ Fetch Registries：获取注册列表信息
  + Eureka客户端从服务器获取注册表信息，并将其缓存在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次

# kibana
+ Management=》index Patterns查看索引
+ Discover=》search按关键字查找，下面的下拉框选择索引
  + filter新增条件过滤
  + avaliable Fields添加需要显示的列
  + 右上角选择需要展示的日志的时间范围

# nginx
+ Linux环境下，怎么确定Nginx是以那个config文件启动的？
```
    输入命令行： ps  -ef | grep nginx
```
+ su - root 切换到root用户 36 12345678
+ 访问地址为对应小机的ip，可以检测nginx是否启动
+ 重新刷新config cd 到 nginx sbin目录 
```
./nginx -s reload
```

# 双数组
+ 双数组Trie树是Trie树的变种，是对Trie树空间上的压缩，主要应用在中文日语等，兼顾了查询效率与空间存储
+ 数组的构建
  + 关键在于如何通过数组存储树形结构，树形结构的关键在于包含其父节点或子节点信息（check数组存储）以及树形的平面结构数据如何铺到线性结构上去（base数组）
  + base数组
    + 公式：state(a) = base(state(empty)) + index(a)
    + 关于数据的铺法，
  + check(state(a)) = state(empty)

# gitlab
+ git branch -r 命令查看远端库的分支情况
+ 从已有的分支创建新的分支(如从master分支),创建一个dev分支 git checkout -b dev
+ git push origin 分支名
+ git checkout 分支名
+ git checkout 历史编码 获取历史版本 编号在项目下历史中的数字字母编码
```
    我现在在dev20181018分支上，想删除dev20181018分支

　　1 先切换到别的分支: git checkout dev20180927

　　2 删除本地分支： git branch -d dev20181018

　　3 如果删除不了可以强制删除，git branch -D dev20181018

　　4 有必要的情况下，删除远程分支：git push origin --delete dev20181018

　　5 在从公用的仓库fetch代码：git fetch origin dev20181018:dev20181018

　　6 然后切换分支即可：git checkout dev20181018
```
+ 查看编码 vi进去 :set ff  修改编码  :set ff=unix 

# linux
+ ps -ef | grep control ps命令将某个进程显示出来/grep命令是查找/中间的|是管道命令 是指ps命令与grep同时执行
+ lsof -i :3306/列出谁在使用某个端口 获取pid
+ grep 、tail 加-v 排除某些字符 使用正则
+ 根据pid获取程序位置ps -aux |grep 28990  执行cd /proc/$pid   执行ls -ail 
```
【命令格式】

grep [option] "string_to_find" filename

常见选项：

（1）-i：忽略搜索字符串的大小写

（2）-v：取反，即输出不匹配的那些文本行

（3）-n：输出行号

（4）-l：输出能够匹配模式的文件名，相反的选项为-L

（5）-q：静默输出

 (6) -a: 输出所在行
 
 -c: 统计次数

选项是可选的，根据实际需求进行选择即可

string_to_find为需要匹配的模式，可以填写字符串或者正则表达式

filename为需要查找的文件的名称
```
+ 装了Glibc的rpm软件包，把linux系统搞崩了
+ glibc是linux系统中最底层的api，几乎其他任何运行库都会依赖glibc，不要在运行中的系统安装Glibc，否则导致系统崩溃，至少应当将新Glibc安装到其他单独目录，以保障不覆盖当前正在使用的Glibc
  
# jconsole
+ -Dcom.sun.management.jmxremote
  -Dcom.sun.management.jmxremote.port=8011 (端口随意，没被占用就行)
  -Dcom.sun.management.jmxremote.ssl=false 
  -Dcom.sun.management.jmxremote.authenticate=false
  这样本地连接的问题也解决了，远程连接的问题也解决了。

# jmeter
+ 创建线程组，创建http请求，创建Aggregate Report报告
+ 从文本中读取参数 Thread ->add-> config-> csv

# InheritableThreadlocal
+ 父线程传递副本给子线程

# git
+ 提交报git did not exit cleanly (exit code 1) ，有冲突，先pull
+ pull报git did not exit cleanly (exit code 128) ，往上看error信息，这次是有文件有冲突，直接删了就好了

# log
```
onMatch="ACCEPT" 表示匹配该级别及以上
onMatch="DENY" 表示不匹配该级别及以上
onMatch="NEUTRAL" 表示该级别及以上的，由下一个filter处理，如果当前是最后一个，则表示匹配该级别及以上
onMismatch="ACCEPT" 表示匹配该级别以下
onMismatch="NEUTRAL" 表示该级别及以下的，由下一个filter处理，如果当前是最后一个，则不匹配该级别以下的
onMismatch="DENY" 表示不匹配该级别以下的
```
# 装机
+ 下压式的基本要组风道，风出不去，或者能贴着进风口
+ cpu盒装带风扇，A卡也不错

# log4j2
+ 基于大小的触发策略
  + 一旦文件达到指定大小 ，SizeBasedTriggeringPolicy将导致翻转。大小可以以字节为单位指定，后缀为KB，MB或GB，例如20MB。当与基于时间的触发策略结合使用时，文件模式必须包含％i(文件名.%d{yyyy-MM-dd}.%i)， 否则目标文件将在每次翻转时被覆盖，因为SizeBased触发策略不会导致文件名中的时间戳值发生更改。如果在没有基于时间的触发策略的情况下使用，则SizeBased触发策略将导致时间戳值更改。
+ 基于时间的触发策略
  + interval:根据日期模式中最具体的时间单位进行翻转的频率。例如，使用以小时为最特定项目的日期模式(filePattern =.... {yyyy-MM-dd} 则时间单位为d)，并且每4小时将增加4次翻转。默认值为1。
+ log4j中时间单位都与filePattern =.... {yyyy-MM-dd HHmmss}最小单位有关，比如这个就是s 
  
# NPE
+ JDK做了优化，当某个空指针异常在相同地方抛出多次，会除去堆栈信息抛出，导致无堆栈，这种在查日志时可扩大时间上的查询范围，找到带堆栈的日志，或者修改jvm参数禁用优化
+ int a = jSONObject.getInteger 可能会导致空指针，如果jSONObject.getInteger解析出来结果为空，自动拆箱时会报错

# shell
+ 执行时代码里有bash xx.sh 报文件找不到，实际上是编码格式不对
+ linux下，通过vim进入，输入命令：set ff查看编码格式，如果为dos，改成unix
+ set ff=unix修改编码格式
+ notepad++ 可以在右下角编码那里点击，改为UNIX编码格式
+ rm -f 无需确认删除文件 rm -rf 删除文件夹
+ su root 管理员
+ /etc/profile 调整环境变量
+ 执行shell脚本权限不足 chmod u+x *.sh

# jenkins
+ 系统设置 -》 系统设置 -》 ssh remote hosts 指定部署机器
+ 全局插件 -》 指定maven -》 项目下配置里，maven选项指定需要使用的maven，并配置mvn命令
+ https://plugins.jenkins.io/ 离线下载插件
+ 远程发布部署 Jenkins插件 Publish Over SSH，配置默认目录，项目中构建后操作-》 send build artifacts over SSH 
  + source files 是以Jenkins工作区间里对应项目的相对路径来找文件的
  + remove prefix 需要去掉不需要的路径，否则传递过去的包会带上路径，比如target
  + remote directory 是以全局配置的默认路径为相对路径
+ 构建前后可以指定删除工作区间的内容
+ 触发器可以指定构建的时机

# SourceCounter
+ 统计代码量

# 纵表横表
+ 经常需要扩展字段的表，或者是动态的表，使用纵表，k-v对，一般是横表

# war包启动换成jar包启动
+ pom中引入spring-boot-maven-plugin插件，包含configuration-》mainClass指定节点
  + executions->execution->goals->goal - repackage(依赖打入jar包)
  + packaging指定为jar

# jar包运行
+ 优先读jar包配置，而不是外部配置，需要exclude掉，读外部配置配置在MANIFEST.MF的class-path里

# nginx
+ 报404，机器被运维关掉了，nginx配置里ip写死了，所以会持续请求已经停掉的服务

# spring
+ springboot 多模块注入bean ，访问不到，这是因为启动类默认扫描当前包路径下的bean
  + 修改包路径，保证模块间在同一包下
  + spring.factory

```
 /**
     * 倒序排列。
     */
    @Override
    public int compareTo(Note<T> o) {
        if(o.weight>this.weight){
            return 1;
        }else if(o.weight<this.weight){
            return -1;
        }
        return 0;
    }
    /**
     * 升序排列
     */
//    @Override
//    public int compareTo(Note<T> o){
//        if(this.weight>o.weight){
//            return 1;
//        }else if(this.weight<o.weight){
//            return -1;
//        }
//        return 0;
//    }
```
# netfix
+ rest(rpc调用)+ribbon(负载均衡)
+ ribbon配置是懒加载的，第一次加载后ReadTimeout、ConnectTimeout写入并缓存在clinet中，且RibbonClientHttpRequestFactory并未开放对ReadTimeout、ConnectTimeout的修改，这块无法动态修改
+ @ConditionalOnMissingBean(XXX.class)在只有特定名称或者类型的Bean不存在于BeanFactory中时才创建某个Bean
+ netflix-archaius动态加载组件初始化时会实例一个ConfigurableEnvironmentConfiguration，导致自定义的configuration的没走，（自定义configuration加了ConditionalOnMissingBean）

# maven
+ spring-cloud-dependencies只是pom文件，对应springcloud版本，springboot与springcloud有版本对应关系
+ 下载会对应很多依赖，需要先清空本地库，依赖配置上，拉下的内容传到私服对应的库的目录上（直接使用xftp操作），刷新索引后私服上就有了
+ pom中配置指定地址 jar包 repositories 插件库 pluginRepositories
+ 私服中，public是一个映射库，映射了什么库是配置的，还有central（放置第三方jar包），release（自己的项目jar包稳定版本），snapshots(快照版本)
+ 快照版本会自动拉最新的，版本中带时间戳

# spring2.0
+ redis驱动从jedis改成Lettuce,后者基于netty

# java基础
+ 父类静态块 -> 子类静态块 -> 父类构造块 -> 父类构造函数 -> 子类构造块 -> 子类构造函数

# rpc解决方案
+ 在使用rest+ribbon时，发现耗时较nginx长，排查发现客户端使用的CloseableHttpClient慢，采用okhttp3
+ okhttp3，rest是支持的，自身带okhttprequestfactory生成okhttprequest，但问题在于request在执行时没走负载均衡客户端
+ 此处自定义了requestfactory和request，并在执行方法里走负载均衡客户端，okHttploadBalancingclient，springcloud提供
+ 遇到第一个问题，相关依赖没引入，基于懒加载的初始化和编译期都没发现
+ 调用时报错，下游参数校验为空，检查发现，post参数未传递，使用了uri拼接方式
+ 后证明使用uri拼接方式有问题，有些请求会报400，对于post请求，在合适时机初始化context，将请求体封装到entity，传入请求体长度（不穿长度会报异常，提示应传0k，实际值xxxk）
+ 再次发现socket close异常，直接原因是服务器或客户端在传输过程中主动断开连接导致，网上有说在请求头里写入keep-alive为true，补充了下这块的知识，报文是分请求行（协议信息）请求头（接受参数的类型，keep-alive这些参数也包括一些自定义的信息）和请求体，而在http1.1以后默认使用长连接，在跟代码的时候发现，对于okhttpclient，组合了一个connectionPool，除了keep-alive时间等信息主要结构是一个双端队列，在请求进来时，会先从队列里去取，如果没有则在请求后put一个进去供下次请求使用，但坑爹的负载均衡客户端（由springcloud提供）在每次请求时居然直接实例一个client，导致队列信息没共用，相当于每次请求都需要再经历一次三次握手，初步分析他每次实例化的原因是因超时时间等信息都杂糅在client里，导致不能使用单例（那为什么不用容器缓存呢，按config维度区分client啊，看hystrix，ribbon里面不都挺喜欢用缓存的），解决方案是重写负载均衡客户端，加入缓存，但这只解决了长连接问题，socket close再次观察发现规律，每次的请求时间都是5000毫秒，这个和我们设置的socket超时时间是对应上的，确定其实就是超时
+ 在线上运行后的25万请求里，出现了8个unexpected end of steam，直接原因是设置的contentLength过长，流读到一半已经没东西可读了

# web服务器
+ nginx、Apache

# 容器
+ ArrayList本质就是通过数组实现的，查找一个元素是否包含要用到遍历，时间复杂度是O（n），而HashSet的查找是通过HashMap的KeySet来实现的，判断是否包含某个元素的实现，时间复杂度是O（1）

# 倒排
+ 正常思维是，比如说古诗，根据名字去检索内容，倒排来说，根据古诗内容去检索出现在了哪些古诗里，而倒排链里存整个古诗又太占空间，故有个过渡索引，诗词标题，故和常规检索逻辑相比称之为倒排

# 总控
+ 160万日活 日请求次数500万 以用户输入的词语来联想问句，整合公司各垂类内容，通过多轮、上下文等，以对话的形式服务用户，解决用户需求。
  1、代码量，日志量
           问题：在最近的一次统计中，总控代码已扩散到1万2千行，为了减少代码进一步扩散而变得难以维护，一方面精简写法，另一方面删除无效代码，日志上多次精简，减少冗余日志
           效果：通过这两种方式，在8月份代码删减了超过700行代码，日志减少未能统计，运维隔天就删除日志了，无数据对比
        2、监控
           问题：组内引入prometheus，在外网通过grafana渲染，考虑到总控作为后端入口，功能上对下游各节点统筹、管理、调度，需要宏观的了解的下游服务的各种情况，同时也为了和组内推广、方向做一个良性互动
           效果：使用grafana语法画出超时较多的几个节点，实时统计包括熔断次数，fallback次数，服务健康度等维度信息
        3、结构、框架
            问题：总控服务起步走，技术框架老，在引入组件时经常需要兼容方案
            效果：war包模式调整为jar包模式，精简各种配置，框架升级，目前在生产上运行稳定，给超时较多的下游加入断路器，提示服务稳定性
        4、运行期异常
            问题：总控在优化前每天运行期异常在数量和种类上都较多，解决了堆栈丢失问题，升级log-util解决了原始异常丢失问题，定位问题更快速
            效果：最近一次统计一周内运行期异常2个，且在后续迭代过程中，通过升级后的log-util能快速定位增量代码问题，在较小的成本下保证总控运行期异常可控，同时解决堆栈丢失、原始异常丢失问题在组内其他服务亦可适用
        5、eureka rpc性能提升支持
            问题：使用默认http客户端，在小流量对比试验中与240性能差距较大，在引入okhttp后，总控作为试验方，积极推进进度，过程中问题均在技术框架层面，自主定位解决一系列问题，为朋朋抽出时间做其他事情
            效果：问题已全部解决，线上效果在性能上已接近240
# 联想
+ 根据source信息以及指定垂类来精确联想出特定的问句，类似百度搜索框的联想功能，联想问句可通过运营平台动态配置，内部实现一开始通过el search（基于Lucene的搜索服务器），但耗时在50毫秒，后期优化，使用倒排索引，将请求耗时从50降到5毫秒以内
# 分词
+ 通过双数组对问句进行分词，算法时间复杂度低，平均耗时2毫秒，单台qps压测成绩是3000
# eureka


# snappy
+ 用来压缩和解压缩的开发包，提供高速压缩速度和合理的压缩率，有java版本
 
# Codis
+ redis集群解决方案的一种
+ codis分片，将数据合理的分配到不同的节点，保存着槽位和实例节点之间的映射关系,槽位间的信息同步交给ZooKeeper来管理
+ codis不支持事务，无回滚机制
+ MGET、MSET和MSETNX m表示批量获取，nx表示不存在key则执行，MSETNX意思是所有给定 key 都不存在时，同时设置一个或多个 key-value 对
  
# curl 地址 & 需要转义
+ curl s10.10.80.19:9213/cache?key=\&cmd=get\&source=queCache

+ 首先要有一个流程控制服务，该服务接收请求，依照业务逻辑规则，依次调用各个微服务，并最终完成处理逻辑。
  这种方法的好处是，流程控制服务时时刻刻都知道每一笔业务究竟进行到了什么地步，监控业务成了相对简单的事情。
  这种方法的坏处是，流程控制服务很容易控制了太多的业务逻辑，耦合度过高，变得臃肿
  
# Prometheus 
+ 使用 pgrep -f prometheus 找到运行的 Prometheus 进程号
+ 使用 kill -TERM 1234 来关闭
+ cd到prometheus的目录 ./prometheus
    
    
# cpu使用率和load
+ 一般来说是正相关的
+ load指一段时间内处于可运行状态和不可中断状态，的进程平均数量。（可运行分为正在运行进程和正在等待CPU的进程，状态为R；不可中断则是它正在做某些工作不能被中断比如等待磁盘IO等，其状态为D），它是从另外一个角度体现CPU的使用状态
+ DMA模式，DMA控制器是一个外设，集成在cpu芯片内部，io过程不再像早期的pio占用cpu时间片， 你磁盘IO，减少CPU时间

# java8
+ List<String> collect = staff.stream().map(x -> x.getName()).collect(   .toList());

# CopyOnWriteArrayList
+ 线程安全，比vector更高效
+ Vector 中，读写操作都被加锁了，而 CopyOnWriteArrayList 中，只有写操作才被加锁，而读操作没有进行加锁。这样，当 读 线程数量远大于 写 线程数量的时候，CopyOnWriteArrayList 尤为高效。
+ 写时复制，当有 写类型的操作作用到 CopyOnWriteArrayList 对象的时候，它们都会先获取锁，然后复制一份当前数据作为副本，然后在当前的数据副本上做修改，最后把修改提交，然后释放锁  

# 异步开发中的问题排查
+ 使用的客户端InternalHttpAsyncClient,给请求设置的超时时间是50毫秒，请求超时发现打印的日志是50毫秒，然而通过日志时间戳发现间隔有一秒，当把超时时间设置为1秒，发现实际间隔时间2秒，网上查询无果后分析问题，时间总是固定的1秒，不太可能是程序执行的时间，一是要不了那么久，二是不可能那么稳定就1秒，也不可能是网络消耗的时间，一样道理不可能就稳定一秒，考虑了下，猜测应该是哪里配置的默认时间，故从配置的config类着手找，client打断点，观察里面的属性值，果然找到一个selectTimeout为1000的，以此找到了属性的赋值入口，通过修改配置值使selecttimeout为10毫秒，1秒的延迟消失了，后续通过查阅相关资料了解到，selector作为NIO的多路复用中的选择器，在没有事件的时候是阻塞状态的，阻塞后唤醒可以通过注册在selector上的socket有事件发生 或者 selector.select(timeOut)超时 或者 selector.wakeup()主动唤醒，当连接超时时，是没有事件唤醒的，所以会等到超时时间结束唤醒selector

# string spilt不保证顺序

# 性能优化
+ C2 CompilerThread0 是热点编译执行线程，会导致cpu使用抖动，当大部分热点代码被编译成机器代码后，这个热点编译执行线程cpu占用会降下来

+ 考虑过使用双数组，检索效率比hash和倒排链都是快的，reids是基于hash的，比redis还快，但是我们写的场景不适合，在线干预，一要响应快，二要保证服务稳定，在增量更新数据上，双数组基本没有任何优势，因为实际上也是基于前缀树的一个变种，压缩版本，通俗的说，压缩文件要改得先解压，改完以后再压缩，所以在扩容时，实际上是全量的重新构建双数组，这个过程，很慢，百万数据几十分钟，而且对于java而言，全量的io写入如何内存分配的不谨慎的话也很容易溢出，导致服务崩掉，所以不适合用双数组，也调研过网上写的那种不无数据结构设计的蛮力前缀树，其实就只是保持了树的结构，用集合去替代数组了，在检索效率上很低的，完全是兜圈子，没意义

# common-pool2池化组件（对象池），Apache的，redis用了这个组件

```java
tar -cjvf xxx.tar.bz2 xxxx

2. 解压
（解压到指定目录）

tar -xjvf 2018-10-21.tar.bz2 -C /mysql/

3. 归档

tar -czvf xxxx.tar.gz xxxx

 

4. 解压

tar -xzvf xxxx.tar.gz
```


# 字节 kb
+ 1024 字节 = 1kb

# java基础
+ 类可以用static修饰，一般用于静态内部类，表示此类属于外部类本身，而不属于外部类的某个对象
+ 抽象类可以有构造方法，但不能直接实例化，而是子类通过super.调用

# PHP
+ $name $表示变量
+ 数组 array("Volvo","BMW","Toyota")  array("key" => ""value")  =>表示键值对
+ array_diff_key() 比较两个数组的键名，并返回差集，该数组包括了所有在被比较的数组（array1）中，但是不在任何其他参数数组（array2 或 array3 等等）中的键名。
+ ->是对象执行方法或取得属性用的
+ ::是调用类中的静态方法或者常量,
+ exit(0) 提前终止脚本执行
+ 静态变量和函数被访问使用self::
+ pho -r 'var_dump(ip2long("11"));'linux上执行php

# BigInteger类自带三十六进制内任意转换功能
+ String string = new BigInteger("3244",5).toString(30);
+ 以上意思为把3244这个五进制数转成三十进制的数
+ 更优雅的字符串连接(join)收集器 Collectors.joining 

# 集合
+ Collections.reverse(list);倒序集合

# 日志
+ log4j要排除spring-boot-starter-logging，否则日志不输出到文件

# 通过jps、jstack、jmap、jstat指令监控分析当前虚拟机堆栈运行状态情况以实时排除调式运行阶段问题
+ http://www.xwood.net/_site_domain_/_root/5870/5874/t_c279948.html

# jdk下载
+ https://jingyan.baidu.com/article/9989c746064d46f648ecfe9a.html
+ jdk8u144 == jdk1.8.0.144
+ linux x64 = 64位
+ linux x86 = 32位

# jvm启动参数 -Dproperty=value
+ System.getProperty("property")获取到value

# 容器
+ 项目中有同事使用到了jdk8自带的JavaScript解释引擎ScriptEngine，出问题当天我看内网Windows环境和linux环境都是ok的，才猜测是不是测试jdk环境的问题，后面定位到因为缺失扩展包中的nashorn.jar，最终导致了engine初始化失败，返回了null，且在外网通过在本地移除此包，复现了该问题
+ mv application.yml application.yml.bak
+ cp application.yml.bak application.yml 可以在容器里调整配置文件了 软链接

# nginx
+ 日志输出规范，在nginx.xml中配置

# springcloudgateway
+ NettyRoutingFilter里指定了超时时间

# 自定义注解  
+ @Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。
+ 如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。
#
+ @EnableConfigurationProperties的作用是开启@ConfigurationProperties。
+  @ConfigurationProperties的作用是将配置文件转换成类对象，便于修改或者获取值。

# curl
+ linux下& =需要转码否则不识别，可以用“”扩起来就不用转码了， 加-v

# concurrencthashmap
+ key value 不能为空

# CROS JSONP 解决跨域问题
+ JSONP把结果以string形式包裹在callback里
+ CROS   目前主流浏览器都支持， IE10以上，浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求

# 监控 prometheus+springboot+grafana
+ https://blog.csdn.net/aixiaoyang168/article/details/100866159

# 双亲委派
+ 对于任意一个类，都需要由加载它的类加载器和这个类本身来一同确立其在Java虚拟机中的唯一性
  + 如果使用不同的类加载器加载同一个类，是两个不一样的类对象
### 流程
+ 子类先委托父类加载
+ 父类加载器有自己的加载范围，范围内没有找到，则不加载，并返回给子类
+ 子类在收到父类无法加载的时候，才会自己去加载
### jvm提供的系统加载器
+ 启动类加载器（Bootstrap ClassLoader）：C++实现，在java里无法获取，负责加载/lib下的类。
+ 扩展类加载器（Extension ClassLoader）： Java实现，可以在java里获取，负责加载/lib/ext下的类。
+ 系统类加载器/应用程序类加载器（Application ClassLoader）：是与我们接触对多的类加载器，我们写的代码默认就是由它来加载，ClassLoader.getSystemClassLoader返回的就是它。
### 双亲委派的实现
+ 同步上锁，先查看这个类是不是已经加载过，递归，双亲委派的实现，先获取父类加载器，不为空则交给父类加载器
### 破坏
+ DriverManager（也由jdk提供）要加载各个实现了Driver接口的实现类，然后进行管理，但是DriverManager由启动类加载器加载，只能记载JAVA_HOME的lib下文件，而其实现是由服务商提供的，由系统类加载器加载，这个时候就需要启动类加载器来委托子类来加载Driver实现
+ https://www.cnblogs.com/joemsu/p/9310226.html

# 小流量实验
+ 根据用户id划分，保证离散性，数据的特性是自增，hash完对100求余后分布到对应的桶中，数组size100,用以按百分比分流量，如果id不是自增的特性，也可以使用其他hash算法去保证数据的离散性
