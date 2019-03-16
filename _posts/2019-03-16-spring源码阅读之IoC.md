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

