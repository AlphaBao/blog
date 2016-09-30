title: JavaScript中的异步与并发
date: 2016-09-30 09:11:31
comments: true
tags: [JavaScript, 异步, event loop]
categories: 编程
---

## 异步

简单地说，JavaScript 是单线程执行的语言，但在使用中有很多异步执行的情况。异步的本质是用其他方式（相对同步）控制程序的执行顺序，这与其他语言中的多线程模型不同，所以常常有人对**非顺序** JavaScript 代码的运行结果感到困惑不解。

### 一段简单的小程序

任何使用过 JavaScript 的程序员都能说出下面这段代码的输出：

```JavaScript
console.log("A");

setTimeout(() => {
  console.log("B");
}, 100);

console.log("C");
```

先后顺序是 `A、C、B`，因为第二个参数的作用是指定延迟的毫秒数，这段代码只有一个 setTimeout，所以不会让人迷惑。

对类似程序的解释通常是由 setTimeout 设置一个定时器，在指定毫秒数后调用回调函数。然而，它的执行机制并不是这么简单。实际上，setTimeout 的作用是在指定的毫秒数之后，在得到机会时，将 callback 放入 event loop queue。

### Event Loop

首先要抛出一些概念，通常所说的 JavaScript Engine 是指负责执行一个一个 chunk 的程序，它依赖宿主环境的调度，也需要通过宿主环境与操作系统产生关联并得到支持。JavaScript Engine 是 JavaScript Runtime(Hosting Environment) 的一部分。

每个 chunk 通常是以 function 为单位，一个 chunk 执行完成后，才会执行下一个 chunk。下一个 chunk 是什么呢？取决于当前 event loop queue 中的队首。event loop queue 中存放的都是消息，每个消息关联着一个函数，JavaScript Engine 就按照队列中的消息顺序执行它们，也就是执行 chunk。
<!-- more -->

所以上面的 setTimeout 实际执行起来更接近这样：

1. chunk1执行：由 setTimeout 启动定时器（100毫秒）

2. chunk2执行：得到机会，将 callback 放入 event loop queue

3. chunk3执行：此 callback 执行

不难发现，**得到机会**很重要！这也就可以解释用 setTimeout 延迟 1000 不一定是准确的，而是会至少延迟一秒。因为如果还有其他的任务在前面，它要等待那些任务对应的消息都出队，也就是程序都执行完成，它才能将 callback 放入队列。也就是实际延迟会大于或等于一秒。

通常所说的触发了一个事件，就是指这个 event listener 得到了执行。与 setTimeout 这个例子中的概念一样，这也是一次 chunk 的执行。像这样一个一个执行 chunk 的过程就叫 event loop。

### 一些简单的小例子

将 setTimeout 加入 try 语句之中，结果会如何？

```JavaScript
try {
  setTimeout(() => {
    throw new Error("Error - from try statement");
  }, 0);
} catch (e) {
  console.error(e);
}
```

try catch 与 setTimeout 不在同一个 chunk，所以……你懂的。

--

下面的堆栈信息会输出 C - B - A 吗？

```JavaScript
setTimeout(function A() {
  setTimeout(function B() {
    setTimeout(function C() {
      throw new Error("Error - from function C");
    }, 0);
  }, 0);
}, 0);
```

它们并不对应同一条 event loop queue 中的消息，分别有各自的调用栈，所以错误栈里面只有 C。

有一些特殊情况，例如：一个 web worker 或者一个跨域的 iframe，则与上面的情况都有所不同。它们是独立的线程，有各自的栈，堆和消息队列。只能通过 postMessage 与它们通信。一次 postMessage 就是在另一个运行时的 event loop queue 中加入了一条消息。

### Job queue

```JavaScript
console.log("A");

setTimeout(() => {
  console.log("B - setTimeout");
}, 0);

new Promise((resolve) => {
  resolve();
})
.then(() => {
  return console.log("C - Promise 1");
})
.then(() => {
  return console.log("D - Promise 1");
});

new Promise((resolve) => {
  resolve();
})
.then(() => {
  return console.log("E - Promise 2");
})
.then(() => {
  return console.log("F - Promise 2");
})
.then(() => {
  return console.log("G - Promise 2");
});

console.log("AA");
```

## 并发



## 参考资料

[Concurrency model and Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)
[You Don't Know JS: Async & Performance](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/ch1.md)
[Async JavaScript: Build More Responsive Apps with Less Code](https://pragprog.com/book/tbajs/async-javascript)


