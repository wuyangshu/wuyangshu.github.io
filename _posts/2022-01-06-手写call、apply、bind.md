---
layout: post
title: 手写call、apply和bind
date: 2022-01-06
Author: moose
tags: [js,手写,call,apply,bind]
comments: true
---
#### 温习一下
- call和apply都是改变上下文中的this并立即执行这个函数，bind方法可以让对应的函数想什么时候调就什么时候调用，并且可以将参数在执行的时候添加。
- call 和bind的传参是单个传递，而 apply 后续传递的参数是数组形式。

<!-- more -->

### 实现
#### call：
1. context为可选参数，如果不传则默认上下文为window；
2. 为context创建一个fn属性，并设置为调用的函数；
3. call可以接收多个参数，需要分离参数；
4. 调用函数然后删除对象上的函数。

```javascript
// this为调用的函数，context为参数对象
Function.prototype._call = function (context) {
	// 判断调用者是否为函数
    if(typeof this !== 'function') {
        throw new TypeError('Error')
    }
	// 不传则默认为window
    context = context || window
	// 新增fn属性并将其设置为需要调用的函数
    context.fn = this
	// 转化数组并分离参数
    let args = Array.from(arguments).slice(1)
	// 传入参数
    const res = context.fn(...args)
	// 删除fn
    delete context.fn
    return res
}
```

### apply:

1. context为可选参数，如果不传则默认上下文为window；
2. 为context创建一个fn属性，并设置为调用的函数；
3. apply是传入数组，需要对数组参数进行处理；
4. 调用函数然后删除对象上的函数。


```javascript
// this为调用的函数，context为参数对象
Function.prototype._apply = function (context) {
	// 判断调用者是否为函数
    if(typeof this !== 'function') {
        throw new TypeError('Error')
    }
	// 不传则默认为window
    context = context || window
	// 新增fn属性并将其设置为需要调用的函数
    context.fn = this
    let res
	// 判断是否传入参数，然后针对不同情况处理
    if(arguments[1]) {
        res = context.fn(...arguments[1])
    }else{
        res = context.fn()
    }
	// 删除fn
    delete context.fn
    return res
}
```
### bind:

1. bind需要返回一个函数，判断外部以哪种方式调用了该函数；

```javascript
// this为调用的函数，context为参数对象
Function.prototype._bind = function (context) {
	// 判断调用者是否为函数
    if(typeof this !== 'function') {
        throw new TypeError('Error')
    }
	// 分离传入的参数
    let args = Array.from(arguments).slice(1)
	// that指向调用的函数
    const that = this
	// 返回一个函数
    return function F() {
		// 判断调用方式
        if(this instanceof F) {
            return new that(...args, ...arguments)
        } else {
            return that.apply(context, args.concat(...arguments))
        }
    }
}
```