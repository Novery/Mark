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
