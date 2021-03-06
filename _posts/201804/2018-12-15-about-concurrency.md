---
layout: post
title:  "处理并发的方式"
date:   2018-12-15 12:00:00

categories: other
tags: knowledge
author: "Victor"
---

并发，就会出现竞争问题，下面是几种常见的处理方式：

* 加锁。所有的参与者竞争同一个锁，持有锁的参与者可以对资源进行操作。如果资源满足读和写可以分离的条件，可以使用读写锁来 提高读的并发数量。
* 单线程顺序执行。Redis的处理IO的线程就是这样操作的，因为Redis属于I/O密集型应用而不是CPU密集型应用，所以虽然是单线程， CPU仍然不会是性能瓶颈，单线程的好处就在于处理请求的时候，一个一个按顺序来，所以无需加锁，可以简化实现。
* 原子操作。参考 Go 中的 `sync/atomic` 包。使用加锁的方式，在持有锁的时间内，可以进行多个操作，而原则操作通常是一条语句， 例如把某个整数加上n，例如当某个数等于m时，复制为n等等。

先抄这么点，以后慢慢补。

## 原文链接

* [处理并发的方式](https://jiajunhuang.com/articles/2018_11_07-concurrency.md.html)
