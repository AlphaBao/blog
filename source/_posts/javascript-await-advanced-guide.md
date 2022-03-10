title: JavaScript await 进阶指南
date: 2022-03-11 00:20:55
comments: true
tags: [JavaScript, 异步, 异常处理]
categories: 编程
---

如果了解 Promise 以及 async/await 的基本使用，就已经足以写出异步流程控制代码，但还有一些略微反直觉、容易记错的点，这些点基本都与 await 有关，本文是对此点一些整理和解析（示例代码大部分来自 MDN，对这些示例代码的解析并非照搬 MDN 原文）。

### Promise 正常处理完成

await 关键字用于在 async function 的局部上下文中挂起（交出局部程序的执行权），直至右边的 Promise 返回。

最常规的情况是，await 操作符右边是一个 Promise，由于 await 将异步函数上下文中的执行权切换到了这个 Promise，这个 Promise 正常处理完成并返回其处理结果后，异步函数后续的语句才开始执行。

```javascript
function resolveAfter2Seconds(x) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(x);
    }, 2000);
  });
}

async function f1() {
  var x = await resolveAfter2Seconds(10);
  console.log(x); // 10
}

f1();
```
<!--more-->
### Thenable objects

await 操作符右边还可以是 Thenable objects，也将同样会按照 resolve 或 reject 的执行来决定返回值。

```javascript
async function f2() {
  const thenable = {
    then: function(resolve, _reject) {
      resolve('resolved!')
    }
  };
  console.log(await thenable); // resolved!
}

f2();
```

这个行为其实是为了与 Promise 的行为相匹配。

```javascript
async function f2() {
  const thenable = {
    then: function(resolve, _reject) {
      resolve('resolved!')
    }
  };
  console.log(await Promise.resolve(thenable)); // resolved!
}

f2();
```

### 转化为 Promise

await 右边的 value 如果不是 Promise，则处理为 Promise.resolve(value)。

```javascript
async function f3() {
  var y = await 20;
  console.log(y); // 20
}

f3();
```

### 处理异常

如果 Promise 的执行中发生 reject，await 位置将抛出异常，reject 的值可以被 try catch 捕获。

```javascript
async function f4() {
  try {
    var z = await Promise.reject(30);
  } catch(e) {
    console.error(e); // 30
  }
}

f4();

// 这里可以看出在应该尽量抛出原生 Error，否则会丢失错误堆栈信息，而且 catch 到的值如果是各种类型都可能出现，处理起来也会更复杂。
```

需要注意的是如果没有 await 操作符，即使在 return 语句中也无法被 try catch 捕获。不能捕获的原因，可以理解为没有使用 await 将异步操作“同步化”。

```javascript
async function f4() {
  try {
    return Promise.reject(30);
  } catch(e) {
    console.error(e); // 无法捕获
  }
}

// 不能被 catch 捕获，因为 return Promise.reject(30) 这一行并没有使用 await 将异步操作“同步化”
f4(); // Uncaught (in promise) 30
```

这个例子还有一个变体，就是使用 Promise.reject(30).catch(e => e) 处理异常，可以用在不希望被 reject 干扰程序执行流程的情况。

```javascript
async function f4() {
  try {
    var z = await Promise.reject(30).catch(e => e);
  } catch(e) {
    console.error(e); // 不会到达这里
  }
}

f4();
```

更形象的一个例子是这样，意图是总能得到一个 response 值（除非 promisedFunction 中存在不受 Promise 流程控制的 setTimeout 之类的异步流程）。

```javascript
var response = await promisedFunction().catch((err) => { console.error(err); });
// 总是得到一个 response，如果 rejected，response 就是 undefined
```

### 参考资料

1. [await - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
2. [Checklist: Best Practices of Node.JS Error Handling (2018)](https://goldbergyoni.com/checklist-best-practices-of-node-js-error-handling/)
3. [深入理解 JavaScript 特性](https://www.ituring.com.cn/book/2452)

