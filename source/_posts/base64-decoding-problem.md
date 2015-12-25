title: Base64：window.atob 的兼容性问题
date: 2015-09-06 15:29:13
comments: true
tags: [JavaScript, Base64]
categories: 编程
---

对于 Base64 编码，浏览器提供的原生方法是 `window.btoa` 和 `window.atob`。

```JavaScript
var encodedData = window.btoa("Hello, world"); // encode a string
var decodedData = window.atob(encodedData); // decode the string
```

从 [caniuse.com](http://caniuse.com/#search=Base64) 的数据来看，IE10 才开始支持这两个方法；而 iOS 和 Android 都很早就支持了，似乎可以用在移动端。但是在使用 `atob` 的过程中，还是遇到了兼容性问题。在 Chrome for Android 中可以正常解码，但是在 iOS Safari 中出现了下面这个错误：

> DOM Exception 5: An invalid or illegal character was specified, such as in an XML name

无奈之下，还是找了个第三方库 [https://github.com/dankogai/js-base64](https://github.com/dankogai/js-base64)，用了之后 Android、iOS 都可以正常解码了。
