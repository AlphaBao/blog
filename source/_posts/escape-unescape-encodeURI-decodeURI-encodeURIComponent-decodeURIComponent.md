title: URL字符集的编码与解码：escape unescape encodeURI decodeURI encodeURIComponent decodeURIComponent
date: 2015-09-06 16:20:59
comments: true
tags: [JavaScript, URL]
categories: 编程
---

>* escape
unescape

>* encodeURI
decodeURI

>* encodeURIComponent
decodeURIComponent

这六个方法功能有关联，如果不清楚每一个的作用，很容易混淆。问题的本质，是如何在 URL 中正确处理各种令人头疼的字符。<!--more-->

首先，`escape unescape` 已经废弃，应当避免使用。

>* The deprecated escape() method computes a new string in which certain characters have been replaced by a hexadecimal escape sequence. Use encodeURI or encodeURIComponent instead.

>* The deprecated unescape() method computes a new string in which hexadecimal escape sequences are replaced with the character that it represents. The escape sequences might be introduced by a function like escape. Because unescape is deprecated, use decodeURI or decodeURIComponent instead.

根据 MDN 的说明，escape 应当换用为 encodeURI 或 encodeURIComponent；unescape 应当换用为 decodeURI 或 decodeURIComponent。

那么，问题就简化为 `encodeURI decodeURI` 与 `encodeURIComponent decodeURIComponent` 的区分。encodeURI 应当用于整个 URI 的编码，encodeURIComponent 应当用于 URI 中某个部分的编码。

 如果用 URL 举例，如下：

```JavaScript
encodeURI('https://www.baidu.com/ a b c')
// "https://www.baidu.com/%20a%20b%20c"
encodeURIComponent('https://www.baidu.com/ a b c')
// "https%3A%2F%2Fwww.baidu.com%2F%20a%20b%20c"
```

而 escape 会编码成下面这样，不伦不类，所以废弃。

```JavaScript
escape('https://www.baidu.com/ a b c')
// "https%3A//www.baidu.com/%20a%20b%20c"
```

由此可知，前端开发中用到最多的应该是 encodeURIComponent/decodeURIComponent。例如下面这个将 URL Search 中的参数转化为对象的方法：

```JavaScript
var parseUrlSearch = function() {
  var query = window.location.search.slice(1);
  var result = {};
  query.split("&").forEach(function(part) {
    var item = part.split("=");
    result[item[0]] = decodeURIComponent(item[1]);
  });
  return result;
};
```
