# AOP源码
+ \<bean>标签是默认的namespace，但\<aop:config>不是，走分支方法
```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
        if (delegate.isDefaultNamespace(root.getNamespaceURI())) {
            NodeList nl = root.getChildNodes();

            for(int i = 0; i < nl.getLength(); ++i) {
                Node node = nl.item(i);
                if (node instanceof Element) {
                    Element ele = (Element)node;
                    String namespaceUri = ele.getNamespaceURI();
                    if (delegate.isDefaultNamespace(namespaceUri)) {
                        this.parseDefaultElement(ele, delegate);
                    } else {
                        delegate.parseCustomElement(ele);
                    }
                }
            }
        } else {
            delegate.parseCustomElement(root);
        }

    }
```
+ 之前把整个XML解析为了org.w3c.dom.Document，org.w3c.dom.Document以树的形式表示整个XML，具体到每一个节点就是一个Node
+ 根据\<aop:config>node拿到Namespace="http://www.springframework.org/schema/aop"
+ 根据namespace获取对应的处理者handler，
```java
public BeanDefinition parseCustomElement(Element ele, BeanDefinition containingBd) {
        String namespaceUri = ele.getNamespaceURI();
        NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
        if (handler == null) {
            this.error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
            return null;
        } else {
            return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
        }
    }
```
+ handler初始化时，会把标签和对应的处理类缓存到map里
```java
public void init() {
        this.registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
        this.registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
        this.registerBeanDefinitionDecorator("scoped-proxy", new ScopedProxyBeanDefinitionDecorator());
        this.registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
    }
```
+ 取得上面的map，根据标签找对应处理类，执行对应parse方法
+ 以config为例，ConfigBeanDefinitionParser的parse
+ 执行configureAutoProxyCreator方法
  + 向Spring容器注册了一个BeanName为org.springframework.aop.config.internalAutoProxyCreator的Bean定义，可以自定义也可以使用Spring提供的（根据优先级来）
  + 根据配置proxy-target-class和expose-proxy，设置是否使用CGLIB进行代理以及是否暴露最终的代理
+ 遍历子节点\<aop:aspect>等
+ 针对\<aop:aspect>，处理\<aop:before>、\<aop:after>、\<aop:after-returning>、\<aop:after-throwing method="">、\<aop:around method="">这五个标签
```java
AbstractBeanDefinition advisorDefinition = this.parseAdvice(aspectName, i, aspectElement, (Element)node, parserContext, beanDefinitions, beanReferences);
```
+ 创建的AbstractBeanDefinition实例是RootBeanDefinition，这和普通Bean创建的实例为GenericBeanDefinition不同
+ 不同切入方式，对应不同的class
+ 根据标签属性设值
```java
AbstractBeanDefinition adviceDef = this.createAdviceDefinition(adviceElement, parserContext, aspectName, order, methodDefinition, aspectFactoryDef, beanDefinitions, beanReferences);

RootBeanDefinition adviceDefinition = new RootBeanDefinition(this.getAdviceClass(adviceElement));

private Class getAdviceClass(Element adviceElement) {
        String elementName = adviceElement.getLocalName();
        if ("before".equals(elementName)) {
            return AspectJMethodBeforeAdvice.class;
        } else if ("after".equals(elementName)) {
            return AspectJAfterAdvice.class;
        } else if ("after-returning".equals(elementName)) {
            return AspectJAfterReturningAdvice.class;
        } else if ("after-throwing".equals(elementName)) {
            return AspectJAfterThrowingAdvice.class;
        } else if ("around".equals(elementName)) {
            return AspectJAroundAdvice.class;
        } else {
            throw new IllegalArgumentException("Unknown advice kind [" + elementName + "].");
        }
    }
```
+ 在生成的RootBeanDefinition实例上包装一层RootBeanDefinition
```java
RootBeanDefinition advisorDefinition = new RootBeanDefinition(AspectJPointcutAdvisor.class);
            advisorDefinition.setSource(parserContext.extractSource(adviceElement));
            advisorDefinition.getConstructorArgumentValues().addGenericArgumentValue(adviceDef);
            if (aspectElement.hasAttribute("order")) {
                advisorDefinition.getPropertyValues().addPropertyValue("order", aspectElement.getAttribute("order"));
            }
```
+ 将BeanDefinition注册到DefaultListableBeanFactory
```java
parserContext.getReaderContext().registerWithGeneratedName(advisorDefinition);parserContext.getReaderContext().registerWithGeneratedName(advisorDefinition);
```
## AOP Bean定义加载----AopNamespaceHandler处理\<aop:pointcut>流程
