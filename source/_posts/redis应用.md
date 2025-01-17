---
title: redis应用
top: false
cover: false
toc: true
mathjax: true
date: 2021-03-10 21:22:09
password:
summary:
tags:
- redis
categories:
- redis
---

## 分布式锁

实现：`set lock:codehole true ex 5 nx`

注意：不要用于较长任务，可能超时释放

优化：设置value是一个随机值，保证不会被其他线程释放

可重入锁：基于线程的Threadlocal变量存储当前持有锁的计数。如果当前有该锁的记录，则计数加一，返回加锁成功。否则尝试加锁，若不存在锁则成功，其他线程/进程会加锁失败。

## 延时队列

实现：使用zset ，消息序列化成value，到期时间作为score。为保障可用性，可以使用多个线程/进程轮询到期任务进行处理（zrangebyscore）。通过zrem结果，判断任务被哪个线程/进程获取，再进一步处理。

优化：考虑将zrangebyscore 和zrem，封装lua scripting,避免获取任务的浪费操作。

## 用户一年的签到统计

使用位图，1天的签到记录只需要占据一个位，一年365位。

## 页面访问量

简单方案：使用set集合存储当天访问某一个用户ID， scard可以统计集合大小。

优化：用户很多时，使用HyperLogLog，提供了不精确的去重方案，节省空间。pfadd/pfcount

## 数据去重

场景：用户为看过的内容推荐去重；爬虫URL去重；

实现：简单去重，使用set。大数据去重，使用布隆过滤器（redis>4.0）bf.add/bf.exists。

误差：布隆过滤器返回存在，实际可能不存在；返回不存在，实际一定不存在。

原理：大型位数组和几个不一样的无偏hash函数。

## 简单限流

场景：限制用户行为在一定时间内的次数

实现：每个用户每种行为作为key，使用zset,value和score都使用时间戳。在pipeline中，增加用户行为，移除时间窗口之前数据，**获取当前剩下行为总数**，并给该key增加过期时间，避免长期占用内存。通过剩下行为总数，判断是否超额。

缺点：不适合1min操作不超过100万次这种场景。

## 漏斗限流

实现：使用redis-cell（redis 4.0），其使用漏斗算法，提供了限流指令。cl.throttle。

非常棒，被拒绝还提供了重试时间。

## 附近的人

实现：使用GeoHash.

原理：将二维经纬度数据映射到一维整数，距离近的点映射距离也会比较近。类似逐步切分蛋糕。

注意：Geo数据将被放到一个zset中。集群环境中集合迁移，可能会影响线上服务运行，建议GEO数据使用单独的redis示例部署。

