title: JavaScript 函数对象的 prototype 属性
date: 2013-10-27 15:18:59
comments: true
tags: [JavaScript, prototype]
categories: 编程
---

 JavaScript 中，函数就是函数对象，函数对象是对象的一种，它们有自己的属性并且可以被调用。函数对象都有一个 prototype 属性，这是一个对象。prototype 有一个 constructor 属性，它指向当前函数对象。

```JavaScript
function func() {}

func.prototype.constructor; // function func() {}
```

如果人为改变函数对象的 prototype 属性，会有很多好玩的代码产生。例如：

```JavaScript
function inherit(Child, Parent) {
  var F = function() {};
  F.prototype = Parent.prototype;
  Child.prototype = new F();
}
```

<!--more-->用这个方式可以使 new Child() 只继承 Parent.prototype 中的属性。F 的作用是充当临时构造函数，保证以后修改 Child.prototype 不会影响到 Parent.prototype。

有趣的是，此时 Child.prototype.constructor 是 Parent 指向的函数对象。这就是因为修改了函数对象的 prototype 属性，函数对象 F、Child 的 prototype 属性都是被修改过的！现在 new Child().constructor 就是 Parent.prototype.constructor。

constructor 属性的作用主要是信息性的，所以改变 prototype 属性后人们也常常重新设置 constructor 属性。

```JavaScript
function inherit(Child, Parent) {
  var F = function() {};
  F.prototype = Parent.prototype;
  Child.prototype = new F();
  Child.prototype.constructor = Child;
}

var Fruit = function() {};
var Apple = function() {};

inherit(Apple, Fruit);

new Apple().constructor === Apple; // true
```

