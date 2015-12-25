title: JavaScript 分号问题
date: 2015-06-06 20:43:52
comments: true
tags: [JavaScript]
categories: 编程
---

## 现状

JavaScript 分号之争由来已久，目前最被认可的结论是：**JavaScript 语句末尾是否加分号是一个编程风格问题，可以加，也可以不加。**


## 选择

> 事情总要有个决定的！
> ——卫斯理《只限老友》（卫斯理系列最后一句话）

那么，到底加不加分号呢？

既然是一个风格问题，我们要明白的是如何选择。选择之前，我们来看看做出选择的前提是什么：

* `加分号`：前提是你了解如何以加分号的方式写 JavaScript。

* `不加分号`：前提是你了解如何以不加分号的方式写 JavaScript。

<!--more-->

初期，了解哪种，选择哪种。如果都不了解，其实你只是胡乱写 JavaScript。若都有了解，自然可以根据喜好，任性！不过我们往往需要考虑更多，如今这个**你也造轮子，我也改框架**的时代，其实我们需要同时了解这两种规则，这是语言设计带来的问题（JavaScript 有自动插入分号的机制，然而并不可靠），避无可避。


## 不加分号

选择不加分号的风格，就要了解哪些情况下不加分号会产生歧义，然后避免。

最著名的例子：

```JavaScript
return
{
  index: 0
}
```

return 后面会自动加入上分号，这并不是期望的结果。所以，要写成：

```JavaScript
return {
  index: 0
}
```

上面是一个**不想加而被加**的例子，下面看一个**想被加而没有加**的例子：

```JavaScript
var buttons = document.querySelectorAll("button")

[].forEach.call(buttons, function(elem) {
  console.log(elem.innerText)
})
```

这段代码会报错，因为它被解释为这样：

```JavaScript
var buttons = document.querySelectorAll("button")[].forEach.call(buttons, function(elem) {
  console.log(elem.innerText)
})
```

"[", "(", "+", "-", "/" 这些符号也有同样的问题。所以，不加分号的风格，要避免这些情况。于是，可以这样：

```JavaScript
var buttons = document.querySelectorAll("button")

;[].forEach.call(buttons, function(elem) {
  console.log(elem.innerText)
})
```

也就是在 "[", "(", "+", "-", "/" 这些前面加分号，如果感觉分号不好看，还可以这样：

```JavaScript
var buttons = document.querySelectorAll("button")

void [].forEach.call(buttons, function(elem) {
  console.log(elem.innerText)
})
```

最好的方式也许是这样：

```JavaScript
var buttons = document.querySelectorAll("button")

Array.prototype.forEach.call(buttons, function(elem) {
  console.log(elem.innerText)
})
```

然而，还没有结束，`Zepto, Sea.js, Vue.js ` 都是不加分号阵营，观察它们的源代码，你会发现一个问题。这些代码中极少用 var 定义函数。这是因为：

```JavaScript
var add = function(a, b) {
  return a + b
}

[0, 1, 2].reduce(add)
```

会被解释为：

```JavaScript
var add = function(a, b) {
  return a + b
}[0, 1, 2].reduce(add)
```

如果代码是这样：

```JavaScript
var mark = [[], [], [1, 2]]

[0, 1, 2].reduce(function(pre, cur) {
  return pre + cur
})
```

还会不报错，对排错很不友好。


## 加分号

以加分号的方式写代码，不需要特别的规则，只需要熟练掌握 JavaScript 语法。加分号的风格，只有在**漏加分号**的情况下才会出问题：

```JavaScript
var add = function(a, b) {
  return a + b
}

[0, 1, 2].reduce(add)
```

这是前面用到过的例子，代码会解释为 }[] 的结构。加分号阵营有：jQuery, Underscore.js, Backbone.js...


## 推荐

我推荐的是加分号风格，因为加分号不需要多余的附加规则。当然，你也可以选择不加分号风格，只要你清楚由此带来的附加限制。


## 争议

[贺师俊](http://www.zhihu.com/people/he-shi-jun)写过一篇[《JavaScript语句后应该加分号么？》](http://hax.iteye.com/blog/1563585)。他推荐不加分号的风格，文章中谈了团队习惯、编程语言的风格、代码构建等更深入的问题，但是他的每个推论我都无法赞同。

> 行首加分号的规则比函数表达式后面加分号的规则其实要简单！

这是他推荐不加分号风格的依据，可是这个结论太过武断。他的推理过程是这样的：


```JavaScript
var a = function () {  
  ...  
}
[1,2,3].forEach(...) 
```

> var a = function () {} 后面要加分号，坑爹！

var 声明的变量后面本来就要加分号，这里只不过是定义了一个 function 类型的变量而已，当然要加。

> function if for 结构的 {} 后面不加分号，var a = function() {} 后面就要加分号，这样不一致。

本来就是不同类型的语句，只要了解 JavaScript 语法，这不是什么问题。

> 行首是否要加分号，我只要看本行的第一个字符就可以了……然而，你如果要判断“}”后面是否要加“;”，你得向上回溯，看清楚整段代码是一个结构呢？还是一个函数？如果是函数的话，是函数声明呢？还是函数表达式！许多时候，你可能向上翻几页还没找到对应的“{”！或者已经忘记了是几层缩进了！

这是编写可维护代码的问题。首先，你不应该写超长的 function。其次，缩进很多也是要尽量避免的。如果你写了超长的函数或者超多的缩进，有没有分号都一样难以阅读。其实超长 function 只应该出现在封装模块的时候：

```JavaScript
(function(root, factory) {
  //
}(this, function(root) {

  // 超长函数

}));
```

这种超长函数不会带来困扰：

1. 这种情况下超长函数不会有那么多缩进。
2. 包装的头尾可以通过构建工具来加。
3. IDE 可以展开/收缩大括号。

> 漏加分号时 jslint 提示不智能。

这虽然会带来一定困扰，但是实际操作中几乎不会带来问题，jslint 报错时我们首先检查是不是漏了分号就可以了。他的文章还列举了一些奇葩换行导致的问题，但那同样是代码可维护性差的问题，不是加不加分号的问题。那种奇葩代码，不加分号一样难读。

> 编译器（代码分析器）完全可以知道哪里应该有EOS (End of Statement)。既然所有的分号其实可以由机器自行加上（无论是加在行首还是行尾），那么我们自己还要手写所有分号的意义到底在哪里？

这是他的文章最后写到的，也很奇葩。要不要加分号之所以有争议，就是因为 JavaScript 的自动插入分号机制并不健全。如果要依赖这个机制，就像前面说的，要加入额外的限定规则。


## 结论

1. 如果涉及到改框架、造轮子等事情，需要同时了解**加分号风格**与**不加分号风格**。
2. 我们总要有个选择，我推荐**加分号风格**。



