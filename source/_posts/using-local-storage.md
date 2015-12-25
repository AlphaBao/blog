title: 使用 localStorage
date: 2014-10-18 15:59:54
comments: true
tags: [JavaScript, HTML5, localStorage]
categories: 编程
---

localStorage 属于 Web 存储的一个 API（另一个是 sessionStorage），以 Key-Value 的形式保存数据，key 和 value 都是字符串。

### 容量

每个浏览器的容量限制不同，现代浏览器一般是 5M。检测容量：[Web Storage Support Test](http://dev-test.nemikor.com/web-storage/support-test/)

### 安全性

以未加密的形式保存在用户的计算机上。
<!--more-->

### 有效期

永不过期。可以在程序中删除，也可以在浏览器配置项中删除。

### 作用域

同源的文档间共享同样的 localStorage 数据（使用同一个浏览器的情况下）。

### 枚举所有数据

```JavaScript
for (var i = 0; i < localStorage.length; i += 1) {
  var key = localStorage.key(i);
  var value = localStorage.getItem(key);

  console.log(key + ': ' + value);
}
```

### 存储事件（storage）

打开两个标签页，其中一个页面上 localStorage 中的数据发生改变，同源的另一个标签页接收一个存储事件。

*在 Chrome 37 上试验无效，也许是我打开的方式不对。

### 数据的实时性问题

在 iOS 7 上，两个 WebView 间传数据：第一次 A 页面 set ，然后马上跳到 B 页面 get —— 正常；之后 A 页面再 set 新的值，B 页面 get 返回的却是上一次的值（旧的值）。

在 Stack Overflow 上看到 IE9 出现的这种问题是用 setTimeout 解决的。但在 iOS 7 的 WebView 中，这种方法并不可靠。

尝试：
A 页面 set，B 页面 setTimeout(..., 300)，有时候正确。值得注意的是 setTimeout(..., 0) 总是返回旧的值。更诡异的地方是，A 页面 set，然后 A 页面 get 总是返回新的值（正确的值），但到 B 页面 get 就返回旧的值。

以上都是 iOS 7 的 WebView 中遇到的问题，Android 4.4 的 WebView 并没有这种问题。







