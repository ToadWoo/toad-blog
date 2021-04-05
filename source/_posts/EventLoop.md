---
title: 重学JavaScript - 事件循环
date: 2021-03-19 23:55:44
categories:
  - 重学JavaScript
---

JavaScript 语言的一大特点就是**单线程**，为什么是单线程而不设计成多线程的呢？ 这因为起初设计 js 目的就是用于简单的用户交互以及 DOM 操作，如果设计成多线程会带来很多问题，比如一个线程在添加 DOM 元素，一个线程在删除 DOM 元素，浏览器就不知道要以哪个线程为准。但是这个单线程也有很大的弊端，必须要等当前任务执行完成才能进入下一个任务，这样用户体验就很差，比如现在发送了一个 AJAX 请求，页面就无法操作，只能等待，在浏览器环境以及 Node 环境都是基于**事件循环**来解决异步回调的问题，只是不同的宿主环境有一定的区别。

虽然 JS 引擎是单线程的，但是浏览器是多线程的，浏览器中有

- GUI 渲染线程
- JS 引擎线程
- 事件线程
- 定时器触发线程
- 异步 HTTP 请求线程

JS 中异步回调都是通过 WebApi 去支持的，例如`setTimeout`、 `XMLHttpRequest` 事件回调`onclick`等等，而这些 API 在浏览器都提供了独立的线程去执行，这才使得 JS 能有地方去处理定时、事件监听等，实际就是**JS 引擎通过这些 API 与浏览器的其他线程进行通信**。这里需要注意的是 JS 的执行与 GUI 渲染是互斥的，虽然他们是不同的线程，但是 js 的执行会对 UI 产生影响，**所以 JS 线程执行的时候，渲染线程是暂停的**。

## 事件循环

**事件循环**它就是 JS 引擎在*等待任务*，*执行任务*和*进入休眠状态等待更多任务*这几个状态之间转换循环的一种机制。

```javascript
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```

<img src="../resource/event-loop1.png" alt="event-loop" style="zoom:67%;" />

JS Runtime 运行机制：

1. 主线程不断循环
2. 对于同步任务，创建执行上下文，按顺序进入调用栈
3. 对于异步任务
   - 与步骤 2 相同，同步执行这段代码
   - 将相应的宏任务或者微任务添加到任务队列
   - 由其他线程来执行具体异步操作
4. 当主线程执行完当前调用栈，就会去任务队列中取任务开始执行
5. 重复以上步骤

你可以使用[Event Loop 演示工具](https://zh.javascript.info/event-loop)来看看这个事件循环的整个过程。

**为何 setTimeout 回调的执行时间与指定的时间可能不一致？**

当 setTimeout 倒计时结束时，如果当前执行栈中还未执行完，是不会重任务队列中获取任务来执行的。

## 宏任务与微任务

**宿主发起的任务称为宏任务**，例如：setTimout、XMLHttpRequest, 而**JS 引擎发起的任务称为微任务**,例如`Promise`、`MutationObserver`

![macro-micro](../resource/event-loop2.png)

两者关系：

- 调用栈选择最先进入队列的 MacroTask（通常是 script 整体代码），如果有则执行；

- 检查是否存在 MicroTask，如果存在则不停的执行，直至清空 MicroTask Queue；

- 浏览器更新渲染（render），每一次事件循环，浏览器都可能会去更新渲染；

- 重复以上步骤。

```javascript
// 执行顺序问题，考察频率挺高
setTimeout(function () {
  console.log(1);
});
new Promise(function (resolve, reject) {
  console.log(2);
  resolve(3);
}).then(function (val) {
  console.log(val);
});
console.log(4);
```

以上代码输入顺序是 2->4->3->1

这是因为 promise then 中的回调是一个微任务，setTimout 中的是一个宏任务，在执行下一个宏任务之前会先执行所有的微任务

**参考资料：**

What the heck is the event loop anyway?：https://www.youtube.com/watch?v=8aGhZQkoFbQ

Event Loop 演示工具：https://zh.javascript.info/event-loop

事假循环：微任务和宏任务： https://zh.javascript.info/event-loop

并发模型与事件循环：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop

深入：微任务与 Javascript 运行时环境：https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_DOM_API/Microtask_guide/In_depth

JavaScript 运行机制详解：再谈 EventLoop ：https://www.ruanyifeng.com/blog/2014/10/event-loop.html
