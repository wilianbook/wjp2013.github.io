---
layout: post
title:  "同步、异步、阻塞、非阻塞以及 API 的一个小技巧"
date:   2018-12-27 12:00:00

categories: other
tags: knowledge
author: "Victor"
---

## 概念

### 例子

1. 老王用水壶煮水，并且站在那里，不管水开没开，每隔一定时间看看水开了没。*同步阻塞*
2. 老王还是用水壶煮水，不再傻傻的站在那里看水开，跑去寝室上网，但是还是会每隔一段时间过来看看水开了没有，水没有开就走人。*同步非阻塞*
3. 老王这次使用高大上的响水壶来煮水，站在那里，但是不会再每隔一段时间去看水开，而是等水开了，水壶会自动的通知他。*异步阻塞*
4. 老王还是使用响水壶煮水，跑到客厅上网去，等着响水壶自己把水煮熟了以后通知他。*异步非阻塞*

### 同步和异步

同步就是烧开水，需要自己去轮询（每隔一段时间去看看水开了没），异步就是水开了，然后水壶会通知你水已经开了，你可以回来处理这些开水了。

同步和异步关注的是 **消息通信机制 synchronous communication / asynchronous communication。**

* 同步方法调用一旦开始，在没有得到结果之前，该调用就不返回。但是一旦调用返回，就得到返回值了。
* 异步方法调用更像一个消息传递，一旦开始，方法调用就会立即返回，调用者就可以继续后续的操作，所以没有返回结果。在调用发出后，被调用者通过状态、通知等来通知调用者，或通过回调函数处理这个调用。

### 阻塞和非阻塞

阻塞就是说在煮水的过程中，你不可以去干其他的事情，非阻塞就是在同样的情况下，可以同时去干其他的事情。

也就是说阻塞调用是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。

阻塞和非阻塞关注的是 **程序在等待调用结果（消息、返回值）时的状态。**

* 阻塞：不能立即获得返回，需要等待
* 非阻塞：立即获得返回，不需要等待

### 同步、异步 和 阻塞、非阻塞 的区别

两者存在本质的区别，它们的修饰对象是不同的。

阻塞和非阻塞是指进程访问的数据如果尚未就绪，进程是否需要等待，**简单说这相当于函数内部的实现区别**，也就是未就绪时是直接返回还是等待就绪。

而同步和异步是指访问数据的机制，同步一般指主动请求并等待 I/O 操作完毕的方式，当数据就绪后在读写的时候必须阻塞，异步则指主动请求数据后便可以继续处理其它任务，随后等待 I/O 操作完毕的通知，这可以使进程在数据读写时也不阻塞。

## 非阻塞同步 API 的设计

有时候创建一个对象涉及的操作很复杂，可能消耗几分钟时间，所以 API 不能立刻返回结果。

### 请求

客户端请求：

```
POST https://api.service.io/stars
name='Death Star'
```

服务端返回 202：

```
HTTP/1.1 202 Accepted
Location: /queue/12345
```

`202 Accepted` 告诉客户端，请求已经接受，但还没有处理，可以去 Location 字段查询进展。

除了上面的头信息，服务器的回应如果有数据体，可以返回一些有效信息（比如任务完成的估计时间、当前状态等等）。

### 查询进展

过了一段时间，客户端就发出请求，查询处理的进展。

```
GET https://api.service.io/queue/12345
```

服务器回应 200。

```
HTTP/1.1 200 Ok

<response>
 <status>PENDING</status>
  <eta>2 mins.</eta>
  <link rel="cancel" method="delete" href="/queue/12345" />
</response>
```

`200 Ok` 告诉客户端，请求成功，具体情况查看数据体。数据体里给出提示，异步操作已成功或还需要等待。

### 操作成功

有一种特殊情况，用户查询异步操作的进展的时候，可能会希望，如果异步操作已经完成，就直接跳转到新资源。

这时，服务器回应 303。

```
HTTP/1.1 303 See Other
Location: /stars/97865
```

`303 see other` 告诉客户端，重定向到不同的资源。Location 字段就是跳转的目标，也就是新资源的网址。

### 删除查询链接

一旦异步操作完成，客户端可以要求服务器删除查询链接。

```
DELETE https://api.service.io/queue/12345
```

服务器回应 204。

```
HTTP/1.1 204 No Content
```

`204 No Content` 告诉客户端，删除成功。以后，客户端再访问这个查询链接，服务器回应 `404 Not Found`。

如果客户端不删除查询链接，服务器完成异步任务后，也可以自动删除。客户端再请求这个链接，服务器回应 `410 Gone`，表示该链接永久性不再可用。

## 相关阅读

* [网络IO之阻塞、非阻塞、同步、异步总结](https://www.cnblogs.com/Anker/p/3254269.html)
* [阻塞和非阻塞，同步和异步](https://www.cnblogs.com/George1994/p/6702084.html)
* [关于同步、异步与阻塞、非阻塞的理解](https://www.cnblogs.com/Anker/p/5965654.html)
* [异步 API 的设计](http://www.ruanyifeng.com/blog/2018/12/async-api-design.html?utm_source=tuicool&utm_medium=referral)