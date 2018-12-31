---
title: 手动实现一个promise/A+
date: 2018-12-03 14:06:06
tags:
---

开始
---
用了这么久Promise，今天突然想研究下Promise是怎么实现的，并且自己实现一个。

准备工作
---
既然要手动实现一个Promise/A+的规范，那么首先需要知道这个规范的详细内容；

Promise/A+规范
---
### Promise状态
一个Promise必须在其中之一的状态： pending， fulfilled或rejected
* pending状态：promise可以转成fulfilled和rejected状态
* fulfilled状态：promise的状态不可变更，并且必须有一个值，且这个值不能被改变；
* rejected状态： promise的状态不可变更，并且必须有一个原因，且这个原因不可改变；

### then方法
一个Promise必须提供一个then

