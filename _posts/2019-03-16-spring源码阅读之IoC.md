---
layout:     post
title:      spring 源码阅读
subtitle:   Ioc
date:       2019-03-16
author:     InkInn
header-img: img/post-bg-clock.jpg
catalog: true
tags:
    - spring
    - java
    
---


# IoC理论
IoC，即控制反转，也是依赖注入。
定义: 

**spring 容器负责对象的生命周期和对象的关系**

理解四个问题:

1. 谁控制谁
2. 控制什么
3. 为何是反转
4. 哪些方面反转了


-
解释：

1. 传统模式，new直接创建对象，依赖的对象由自己控制。但在spring中，由spring容器控制。
2. 控制对象。
3. 由原来的主动创建，演变为被动接受，由IoC创建之后注入。
4. 获取方式被反转了。


-
注入形式:

1. 构造注入
2. setter 方法注入
3. 接口注入

# 统一资源加载
Spring为何自己实现资源加载:

1. 资源定义广泛，如二进制、文件、字节流、网络等；也可以存在任何场所，网络系统、文件系统，应用程序等
2. java.net.URL 比较局限

资源加载策略要求:

1. 划分职能，资源定义与资源加载要划分清晰。
2. 统一抽象，同一资源定义和资源加载策略，如何处理资源，应有抽象接口定义。

-
实现：

1. 统一资源 Resource
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g14lqomq03j30sp09i74w.jpg)

2. 统一资源定位：ResourceLoader
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g14m8r87a1j30yf09d3zh.jpg)


# BeanDefinition

IoC容器步骤:
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g14mnpbgh7j31e005a760.jpg)

1. 获取资源
2. 获取BeanFactory
3. 根据BeanFactory创建一个BeanDefinitionReader对象，Reader对象为资源的解析器
4. 装载资源

过程分为三步:  
资源定位-----> 装载------> 注册
![](https://ws1.sinaimg.cn/large/006tKfTcgy1g14mrf7nncj30v408yaak.jpg)

1. 资源定位（外部资源描述bean）
2. 装载（外部资源描述转成IoC内部描述bean的数据结构）
	- IoC内部维护BeanDefinition Map结构
3. 注册（向IoC容器注册第二部解析的BeanDefinition,通过BeanDefinitionRegistry实现，就是将第二步的实例添加到Map中）
	- 这里并没有完成依赖注入（创建bean），bean创建是发生在第一次调用getBean，向容器索要bean的时候
1. 	- 预处理设置， lazyinit = false，bean的依赖注入就会发生在容器初始化的时候。

-

XmlBeanDefinitionReader#doLoadBeanDefinitions时序图

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g14pa3roxcj30wa0u07ap.jpg)


BeanDefinition类图
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g14r4sb7npj30m90bjmxq.jpg)

-

BeanDefinition 解析过程总结

解析入口L: DefaultBeanDefinitionDocumentReader#processBeanDefinition()

1. 解析默认标签: BeanDefinitionParserDelegate#parseBeanDefinitionElement()
2. 解析自定义标签: BeanDefinitionParserDelegate#decorateBeanDefinitionIfRequired()
3. 注册解析的BeanDefinition: BeanDefinitionReaderUtils#registerBeanDefinition()


# 自定义标签使用
步骤:

1. 创建一个需要扩展的组件
2. 定义一个XSD文件，用于描述组件内容
3. 创建一个实现`org.springframework.beans.factory.xml.AbstractSingleBeanDefinitionParser`接口的类 ，用来解析XSD文件中的定义和组件定义。
4. 创建一个Handler，继承`org.springframework.beans.factory.xml.NamespaceHandlerSupport`抽象类，用于将组件注册到Spring容器中。
5. 编写spring.handlers和Spring.schemas文件。

-

自定义标签解析过程:

1. 加载spring.handlers文件，对内容解析，形成<namespaceUri,类路径> 这样的一个映射。
2. 根据获取的namespaceUri 就可以得到相应的类路径，对其进行初始化得到相应的NamespaceHandler对象。
3. 调用NamespceHandler的parse()方法，根据标签的localName得到相应的BeanDefinitionParser实例对象
4. 调用beandefinitionParser的parse()方法，该方法定义在AbstractBeanDefinitionParser抽象类中，核心逻辑在parseInternal()中，该方法返回一个AbstractBeanDefinition实例对象，主要方法已经在AbstractSingleBeanDefinitionParser中实现。
![](https://ws2.sinaimg.cn/large/006tKfTcgy1g15r8a5li9j31dy0cmjt3.jpg)


# Bean加载

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g15sihb5jdj31e00smwgw.jpg)

Spring IoC容器作用如图，以某种方式加载Configuration Metedata, 将其解析注册到容器内部，然后根据这些信息绑定整个系统的对象，最终组装成一个可用的基于轻量级容器的应用系统

分为如下两个阶段:

1. 容器初始化阶段
 * 通过某种方式加载Configuration Metadata(主要依据Resource, ResourceLoader)
 * 对加载的MetaData进行解析和分析，组装成BeanDefinition
 * 将BeanDefinition保存注册到相应的BeanDefinitionRegistry
 * Spring IoC的初始化工作完成

2. 加载Bean的阶段
 * 显示或者隐式的调用BeanFactory#getBean()时，测绘出发加载Bean阶段
 * 容器首先检查请求的的对象是否已经初始化完成了，如果没有，则或根据注册的Bean信息实例话请求的对象，并为其注册依赖，将其返回给请求方。
 * 至此第二阶段也已经完成

 
加载过程:

1. 从缓存中获取单例Bean, 以及对Bean的实例中获取对象
2. Bean的加载，Bean的依赖处理
3. 各个作用域Bean的初始化过程
