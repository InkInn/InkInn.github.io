---
layout:     post
title:      记一次spring-data-redis踩坑之旅
subtitle:   记一次spring-data-redis踩坑之旅
date:       2018-09-08
author:     InkInn
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - redis
    - jedis
    - spring
    - java
    
---


# 问题
最近项目经常出现下图报警

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fv25srhw72j317w04sju3.jpg)
错误信息：Bulk add of multiple elements with the same score is not supported. Add the elements individually。意思就是SortedSet不能批量add多个相同score的元素


# 定位
根据异常信息首先定位到

```
org.springframework.data.redis.connection.jedis.JedisConnection.zAddArgs
```

这个函数貌似就是用来做版本控制的，如果jedis版本低于要求，且批量操作score相同，则会抛出异常

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fv2h54kntij31ge0jejv3.jpg)

要求jedis版本不低于2.4

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fv2h9lu3xqj30wy0663zc.jpg)

# 解决
那么问题就很明显了，当前项目肯定是最近不小心引入了低于2.4版本的jedis。打开maven依赖图，果不其然，之前一直引用的是2.4.2版本的jedis，但是最近新需求开发过程中，引入其他项目组的依赖包含了2.1.0版本的jedis。立马加上exclusion，就万事大吉啦！



# 思考
spring-data-redis在这块为什么要限制jedis版本呢？对比了半天jedis的2.3和2.4版本，没发现什么不同。查阅资料redis的zadd历史
![](https://ws1.sinaimg.cn/large/006tNbRwgy1fv2htqwyfkj30y406mmy0.jpg)
redis 2.4版本之后才支持批量操作。但是，这和jedis版本没多大关系吧😂😂
.........持续思考中（撸码中）..........




