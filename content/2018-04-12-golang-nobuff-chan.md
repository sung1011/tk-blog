---
layout: post
title: "golang-grammer"
tags:
description: golang chan
 - go
description: go basic
---

# chan 

### nobuff

一个基于无缓存Channels的发送操作将导致发送者goroutine阻塞，直到另一个goroutine在相同的
Channels上执行接收操作，当发送的值通过Channels成功传输之后，两个goroutine可以继续执行
后面的语句。反之，如果接收操作先发生，那么接收者goroutine也将阻塞，直到有另一个
goroutine在相同的Channels上执行发送操作。

### buff
带缓存的Channel内部持有一个元素队列。