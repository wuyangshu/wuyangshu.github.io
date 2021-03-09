---
layout: post
title: Event Loop（事件循环机制）
date: 2021-03-09
Author: moose
tags: [js, 前端]
comments: true
---
## 前言

javascript从诞生之日起就是一门单线程的非阻塞的脚本语言。单线程就意味着，javascript代码在执行的时候，都只有一个主线程来处理所有的任务。
这就像只有一个窗口的银行，客户需要一个一个排队办理业务。同理js任务也要一个一个顺序执行。如果一个任务耗时过长，那么后一个任务也必须等着。

然而JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这就决定了它只能是单线程，否则会带来很复杂的同步问题。
<!-- more -->

## 正文

在了解事件循环机制之前，还需要了解一下基本的数据结构类型

### 堆、栈、队列

**堆**

堆是一种数据结构，是利用完全二叉树维护的一组数据，堆分为两种，一种为最大堆，一种为最小堆，将根节点最大的堆叫做最大堆或大根堆，根节点最小的堆叫做最小堆或小根堆。堆常用来实现优先队列，堆的存取是随意的，这就如同我们在图书馆的书架上取书，虽然书的摆放是有顺序的，但是我们想取任意一本时不必像栈一样，先取出前面所有的书，书架这种机制不同于箱子，我们可以直接取出我们想要的书。

（堆中存放着一些对象）

**栈**

栈在计算机科学中是限定仅在表尾进行插入或删除操作的线性表。栈是一种数据结构，它按照后进先出的原则存储数据，先进入的数据被压入栈底，最后的数据在栈顶，需要读数据的时候从栈顶开始弹出数据。这就类似于我们要在取放在箱子底部的东西（放进去比较早的物体），我们首先要移开压在它上面的物体（放进去比较晚的物体）。

（栈中存放着基础类型变量以及对象的指针）

**队列**

队列的特殊之处在于它只允许在表的前端进行删除操作，而在表的后端进行插入操作，和栈一样，队列是一种操作受限制的线性表。
而且队列是一种先进先出的数据结构，又称为先进先出的线性表。也就是说先放的先取，后放的后取，就如同行李过安检的时候，先放进去的行李在另一端总是先出来，后放入的行李会在最后面出来。


**执行栈与事件队列**

- 执行栈
  
    当我们调用一个方法的时候，js是会生成一个与这个方法对应的执行环境，当执行环境中的一系列方法被依次调用的时候，因为js是单线程的，同一时间只能执行一个方法，于是这些方法被排队在一个单独的地方。这个地方就被称为执行栈。

    当一个脚本第一次执行的时候，js引擎会解析这段代码，并将其中的同步代码按照执行顺序加入执行栈中，然后从头开始执行。如果当前执行的是一个方法，那么js会向执行栈中添加这个方法的执行环境，然后进入这个执行环境继续执行其中的代码。当这个执行环境中的代码 执行完毕并返回结果后，js会退出这个执行环境并把这个执行环境销毁，回到上一个方法的执行环境。这个过程反复进行，直到执行栈中的代码全部执行完毕。



- 事件队列

    js引擎遇到一个异步事件后并不会一直等待其返回结果，而是会将这个事件挂起，继续执行执行栈中的其他任务。当一个异步事件返回结果后，js会将这个事件加入与当前执行栈不同的另一个队列，我们称之为事件队列。被放入事件队列不会立刻执行其回调，而是等待当前执行栈中的所有任务都执行完毕， 主线程处于闲置状态时，主线程会去查找事件队列是否有任务。如果有，那么主线程会从中取出排在第一位的事件，并把这个事件对应的回调放入执行栈中，然后执行其中的同步代码，如此反复，这样就形成了一个无限的循环。这就是这个过程被称为“事件循环（Event Loop）”的原因。


**同步任务与异步任务**

Javascript单线程任务被分为同步任务和异步任务，同步任务会在调用栈中按照顺序等待主线程依次执行，异步任务会在异步任务有了结果后，将注册的回调函数放入任务队列中等待主线程空闲的时候（调用栈被清空），被读取到栈内等待主线程的执行。

**宏任务与微任务**

实际上异步任务之间也不相同，因此他们的执行优先级也有区别。不同的异步任务被分为两类：微任务（micro task）和宏任务（macro task）。

> 宏任务：setTimeout、setInterval、setImmediate(Node独有)、I/O、UI渲染

> 微任务：process.nextTick(Node独有)、promise、Object.observe、MutationObserver

我们需要记住：当当前执行栈执行完毕时会立刻先处理所有微任务队列中的事件，然后再去宏任务队列中取出一个事件。同一次事件循环中，微任务永远在宏任务之前执行。


### 浏览器的Event loop

JavaScript代码的具体流程：

- 执行全局Script同步代码，这些同步代码有一些是同步语句，有一些是异步语句（比如setTimeout等）；
- 全局Script代码执行完毕后，调用栈Stack会清空；
- 从微队列microtask queue中取出位于队首的回调任务，放入调用栈Stack中执行，执行完后microtask queue长度减1；
继续取出位于队首的任务，放入调用栈Stack中执行，以此类推，直到直到把microtask queue中的所有任务都执行完毕。注意，如果在执行microtask的过程中，又产生了microtask，那么会加入到队列的末尾，也会在这个周期被调用执行；
- microtask queue中的所有任务都执行完毕，此时microtask queue为空队列，调用栈Stack也为空；
- 取出宏队列macrotask queue中位于队首的任务，放入Stack中执行；
- 执行完毕后，调用栈Stack为空；
- 重复第3-7个步骤；
- 重复第3-7个步骤；
- ......
可以看到，这就是浏览器的事件循环Event Loop

归纳一下
1. 宏队列macrotask一次只从队列中取一个任务执行，执行完后就去执行微任务队列中的任务；
2. 微任务队列中所有的任务都会被依次取出来执行，直到microtask queue为空；
3. 只要执行UI rendering，它的节点是在执行完所有的microtask之后，下一个macrotask之前，紧跟着执行UI render。


### 练习一下

```javascript
console.log('1');

setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
})
process.nextTick(function() {
    console.log('6');
})
new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})
setTimeout(function() {
    console.log('9');
    process.nextTick(function() {
        console.log('10');
    })
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
})

```

正确答案
> 输出顺序为：1，7，6，8，2，4，3，5，9，11，10，12

#### 文章参考

- <a href="http://www.ruanyifeng.com/blog/2014/10/event-loop.html">JavaScript 运行机制详解：再谈Event Loop</a>
- <a href="https://www.cnblogs.com/cangqinglang/p/8967268.html">详解JavaScript中的Event Loop（事件循环）机制</a>
- <a href="https://zhuanlan.zhihu.com/p/55511602">一次弄懂Event Loop</a>
- <a href="https://www.jianshu.com/p/5f148c3e4f7d">数据结构中堆、栈和队列的理解</a>