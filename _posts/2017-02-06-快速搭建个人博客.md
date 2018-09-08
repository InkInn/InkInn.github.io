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


# 定位问题
根据异常信息定位到
```
org.springframework.data.redis.connection.jedis.JedisConnection.zAddArgs
```


