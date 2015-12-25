title: 浮动元素与 BFC
date: 2014-02-24 22:43:36
comments: true
tags: [CSS, BFC, hasLayout]
categories: 编程
---

CSS 的默认以 document flow 方式渲染元素，而 float 与 postion 属性可以改变这种状况。

CSS 中 float 属性的本意是实现文本绕排图片，不过实际使用中 float 常常用来创建多栏布局。float 最常出现的问题是父元素不再包含它（float 之后不再属于 document flow）。

### 解决方法 1：

为父元素添加 overflow: hidden，它能可靠地迫使父元素包含其浮动的子元素。

```HTML
<section>
  <img src="images/tianfangye.png">
  <div class="clear_me"></div>
</section>
```

```CSS
section {
  border:1px solid blue;
  overflow: hidden;
}
img {
  float: left;
}
```
<!--more-->
### 解决方法 2：

同时浮动父元素。浮动父元素后，不论子元素是否浮动都会被包含。

```HTML
<section>
  <img src="images/tianfangye.png">
  <div class="clear_me"></div>
</section>
```

```CSS
section {
  border:1px solid blue;
  float: left;
}
img {
  float: left;
}
```

### 解决方法 3：

添加非浮动的清除元素。

```HTML
<section>
  <img src="images/tianfangye.png">
  <div class="clear_me"></div>
</section>
```

```CSS
section {
  border:1px solid blue;
}
img {
  float:left;
}
.clear_me {
  clear:left;
}
```

上面的方法添加了一个纯表现性元素，这种方法的一个变种是使用伪元素，即著名的 clearfix 方案。

```HTML
<section class="clearfix">
  <img src="images/tianfangye.png">
  <div class="clear_me"></div>
</section>
```

```CSS
.clearfix:after {
  content: ".";
  display: block;
  height: 0;
  visibility: hidden;
  clear: both;
}
section {
  border:1px solid blue;
}
img {
  float:left;
}
.clear_me {
  clear:left;
}
```

### BFC 与浮动

解决方法 1 和 2 实际上是创建了BFC（Block formatting context，块级格式化的上下文），BFC 是指一个区域——一个元素符合一定条件，那么这个元素的区域就是块级格式化的上下文，这个区域有自己独特的布局方式。

### 创建 BFC 的条件：

1. 根元素或其它包含它的元素
2. 浮动 (float 不为 none)
3. 绝对定位元素 (position 为 absolute 或 fixed)
4. 内联块 inline-blocks (display: inline-block)
5. 表格单元格 (display: table-cell，表格单元格默认属性)
6. 表格标题 (display: table-caption, 表格标题默认属性)
7. overflow 不为 visible
8. flex boxes (display: flex 或 inline-flex)

可见，浮动元素就是一种 BFC。

### BFC 的特性

1. 阻止外边距折叠
2. 包含浮动的元素
3. 不会绕排兄弟浮动元素

### IE6、IE7 的私有属性：hasLayout

IE6、IE7 不支持 BFC，与之相对应的概念是 hasLayout，效果上类似 BFC，但并不相同，而那些不相同的部分基本就是现在所说的 IE6、IE7 在 CSS 方面的 bug。

hasLayout 是一个比较独立的内容，可以参考[hasLayout综合](http://www.qianduan.net/comprehensive-haslayout.html)。

### 本文参考资料

1. [CSS设计指南（第3版）](http://www.ituring.com.cn/book/1111)
2. [MDN-Block formatting context](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Block_formatting_context)
3. [详说 Block Formatting Contexts (块级格式化上下文)](http://kayosite.com/block-formatting-contexts-in-detail.html)









