title: 从上下文，到作用域（彩蛋：理解闭包）
date: 2017-10-14 17:51:02
comments: true
tags: [context, scope, closure, JavaScript]
categories: 编程
---

## 前言

近几天在编程群中的聊天，让我发现了很多人并不清楚什么是上下文（context）、什么是作用域（scope），而且纠结在其中。我当初对这两个概念也只有粗浅的理解，不过我从一开始就不怎么困惑，因为我清楚自己对这一问题的认识边界。现在，我对它们的认识也只加深了一点点。不过，群聊中小伙伴的热情鼓舞了我——很多最最初学的小伙伴，想到和思考的是很多我从没考虑过的问题，小伙伴们真是达到了“进一寸有一寸的欢喜”这一境界。见贤思齐，我决定把这一点点进步记录下来。


## 上下文与作用域的关系

很多人弄不清除，原因当然是既不了解上下文，也不了解作用域——我是说，几乎没有人明白上下文是什么而不明白作用域是什么，反之亦然。`上下文（context）`和`作用域（scope）`都是编译原理的知识，具体编程语言有具体的实现规则，本文关注 JavaScript 语言的实现。首先需要关注的是，这两个概念的关系非常密切，所以先了解它们的关系，有助于理解它们到底是什么。

`上下文（context）`和`作用域（scope）`的关系：

**上下文是一段程序运行所需要的最小数据集合；作用域是当前上下文中，按照具体规则能够访问到的标识符（变量）的范围。**

后文是对上下文和作用域更详细的解释，知道了上面指出的关系，往下阅读时就可以加深对这一关系的理解了。<!--more-->


## 上下文

**上下文（context）**是一段程序运行所需要的最小数据集合。我们可以从**上下文交换（context switch）**来理解上下文，在多进程或多线程环境中，任务切换时首先要中断当前的任务，将计算资源交给下一个任务。因为稍后还要恢复之前的任务，所以中断的时候要**保存现场**，即当前任务的上下文，也可以叫做环境。即上下文就是恢复现场所需的最小数据集合。容易把人弄晕的一点是，我们这里说的**上下文**、**环境**有时候也称作**作用域（scope）**，即这两个概念有时候是混用的。不过，它们有不同的侧重点，下一节将会说明。

另外，JavaScript 中常见的情形是一个方法/函数的执行。从一段程序的角度看，这段程序运行所需的所有变量，就是它的上下文。


## 作用域

**作用域（scope）**是标识符（变量）在程序中的可见性范围。**作用域规则**是按照具体规则维护标识符的可见性，以确定当前执行的代码对这些标识符的访问权限。作用域（scope）是在具体的作用域规则之下确定的。

前面说过，有时候上下文、环境、作用域是同义词；不过，上下文（context）指代的是整体环境，作用域关注的是标识符（变量）的可访问性（可见性）。上下文确定了，根据具体编程语言的作用域规则，作用域也就确定了。这就是上下文与作用域的**关系**。

写 JavaScript 代码时，如果 Function 作为参数，可以指定它在具体对象上调用时，这个对象常常叫做 context：

```javascript
function callWithContext(fn, context) {
  return fn.call(context);
}

const apple = {
  name: "Apple"
};

const orange = {
  name: "Orange"
};

function echo() {
  console.log(this.name);
}

callWithContext(echo, apple);  // Apple
callWithContext(echo, orange); // Orange
```

为什么将这个参数叫做 context？因为它关系到调用环境，指定了它，就指定了函数的调用上下文。再加上具体的作用域规则，作用域也确定了。

在 JavaScript 中，这个具体的作用域规则就是**词法作用域（lexical scope）**，也就是 JavaScript 中的作用域链的规则。词法作用域是的变量在编译时（词法阶段）就是确定的，所以词法作用域又叫**静态作用域（static scope）**，与之相对的是**动态作用域（dynamic scope）**。

*You Don't Know JS: Scope & Closures* 用简单例子解释过动态作用域，下面用一个类似的例子说明一下：

```javascript
function foo() {
  console.log(a);
}

function bar() {
  let a = 3;
  foo();
}

let a = 2;

bar(); // 2
```

有一定 JavaScript 编程经验的人都能看出，这段程序会输出 2，但如果在动态作用域的规则下，应该输出 3，即 a 的引用不再是编译时确定，而是调用时确定的。这有点像 JavaScript 中的 `this`，所以 MDN 中，function.bind 的方法签名中第一个形参名称用的是 `thisArg` 这一更**科学**的名字：

`fun.bind(thisArg[, arg1[, arg2[, ...]]])`

同样情况的还可见于 Lodash 的文档：

`_.bind(func, thisArg, [partials])`


## 彩蛋：理解闭包

上一节中的代码中，之所以输出 2，是因为 foo 是一个闭包函数。如果从本文中理解了上下文和作用域的概念，对于**闭包是什么**这一问题是不是感到豁然开朗？

前面说过，词法作用域也叫静态作用域，变量在词法阶段确定，也就是定义时确定。虽然在 bar 内调用，但由于 foo 是闭包函数，即使它在自己定义的词法作用域以外的地方执行，它也一直保持着自己的作用域。所谓闭包函数，即这个函数封闭了它自己的定义时的环境，形成了一个[闭包](/2013/06/16/how-to-explain-closure/)，所以 foo 并不会从 bar 中寻找变量，这就是静态作用域的特点。

一个更加典型的例子是：

```javascript
function fn() {
  let a = 0;
  function func() {
    console.log(a);
  }
  return func;
}

let a = 1;
let sub = fn();

sub(); // 0;
```

sub 就是 func 这一返回值，func 定义在 fn 内部并且被传递出来了，所以 fn 执行之后垃圾回收器依然没有回收它的内部作用域，因为 func/sub 在使用。sub 依然持有 func 定义时的作用域的引用，而这个引用就叫作闭包。调用 sub 时，它可以访问 func 定义时的词法作用域，因此找到的 a 是 fn 内部的变量 a，它的值是 0。


## 参考资料

[You Don't Know JS: Scope & Closures](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch2.md)  
[Context (computing)](https://en.wikipedia.org/wiki/Context_%28computing%29)  
[Scope (computer science)](https://en.wikipedia.org/wiki/Scope_%28computer_science%29)  
[Function.prototype.bind()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)  
[Function _.bind()](https://lodash.com/docs/4.17.4#bind)  
