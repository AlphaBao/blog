title: 理解 HTML5 语义化标签：div, section & article
date: 2014-01-26 11:19:52
comments: true
tags: [HTML5, 语义化标签]
categories: 编程
---

## 语义化标签·非语义化标签

标签名描述了标签内容，就是语义化标签（semantic elements）；标签名对标签内容毫无描述，则是非语义化标签（non-semantic elements）。img、form 和 table 是语义化标签，div、span 则是非语义化标签。


## 使用语义化标签的好处

HTML5 中增加了一批语义化标签，说明开发者应当尽量写语义化的 HTML。另外，HTML 是声明式语言，语义化的标签名称，既是给人看的，也是给浏览器看的。

语义化标签，就像写明了“产品部”、“设计部”、“研发部”的门牌——不用打开门，只需要看看门牌，就能知道这个房间的作用。

非语义化标签，就像只写了号码的门牌——光看门牌，只能看到与房间作用无关的门牌号；必须开门看看，才能知道这个房间是干什么的。
<!-- more -->

## div, section & article

div、section、article 都可以作为容器标签，但在语义上是递增的，使用方面有所不同。

### div
语义：无语义
说明：它是通用的容器标签，本身不包含语义。通常为了非语义的功能而使用，比如样式控制、脚本化 HTML。

### section
语义：部分
说明：一组逻辑上相关的内容，可以放在一个 section 里面。section 是独立的一段内容，但 section 往往是整体的一部分。

### article
语义：物件
说明：页面中独立的、自包含的内容适合放到 article 里面。

div、section、article 是由一般到特殊的过程。div 是为方便编程而使用。section 与 article 的区别在于，article 表示的是完整的内容——脱离了页面的其他标签，一个 article 依然是一个整体。

## 参考资料

[HTML5 Semantic Elements](http://www.w3schools.com/html/html5_semantic_elements.asp)


