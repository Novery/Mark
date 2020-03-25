# 记一次堆外内存泄漏排查
+ 使用java8提供的js解释引擎ScriptEngine，上线后出现内存报警，提示内存使用量已达到百分之八十，因项目使用了NIO的direct memory,首先怀疑堆外内存占用过多
+ 项目使用springboot，集成了Micrometer实现监控，spring-boot-actuator使用了Micrometer，对prometheus进行封装，集成到springboot工程中，用以采集监控数据并接入prometheus服务，通过grafana渲染数据
+ 通过监控数据查看，服务的堆内存使用正常，正常GC，无内存泄漏情况，而direct memory的使用情况也正常，但元空间使用量较大，占用了1G
+ 元空间存储了类信息，常量池，静态变量，jitSUV而编译后的代码，而在能观测的白盒指标中也确认了类加载数据较为异常，有12k
+ 因为当晚发生GC后量又降下来了，故当时判断内存分配的不够
+ 第二日早发现服务发生了OOM，且容器自动重启了，当天开盘后流量高峰发现加载的类一直增长，对应的元空间使用也存在一直增长的情况，确认为内存泄漏的元凶。怀疑代码中存在类对象重复创建的情况
+ 在本地使用-XX:+TraceClassLoading 打印加载class信息，来确认是哪个类在重复创建类对象，无果，未能复现
+ 在开发环境容器中使用java -verbose:class 直接将class信息输出到控制台，成功复现
+ 使用java8提供的js解释引擎ScriptEngine，不断的重复创建对象，而元空间不参与GC，导致了内存泄漏
