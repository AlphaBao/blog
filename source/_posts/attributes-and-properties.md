title: Attribute 与 Property
date: 2014-09-02 17:44:41
comments: true
tags: [JavaScript, DOM, jQuery]
categories: 编程
---

DOM 元素中的 attribute 与 property 并不相同。attribute 通常翻译为“特性”，property 通常翻译为“属性”。其实它们是近义词，并不能根据特性、属性这两个词汇来区分 attribute 与 property。

>* 特性：某事物所特有的性质；特殊的品性、品质。
>* 属性：事物所具有的不可缺少的性质。

所以，attribute 与 property 都可以叫“特性”，也都可以叫“属性”。

### 区别

从 HTML 到 DOM 元素，一种是声明式的语言，一种是命令式语言。attribute 是直接收集 HTML 中的属性转为 js 对象，对象的 value 最接近原生态，也就是 HTML 标记里面的样子；property 也是转为 js 对象，但是转化的过程中会对 value 做一些处理，将 value 转为对 js 来说更有意义的值。

比如：
```
<input type="checkbox" checked="checked" />
```

```JavaScript
elem.getAttribute("checked”) // “checked”，原生态的值
elem.checked // true，对 js 来说更有意义的值
```
<!--more-->
### HTML 标准属性

对于标准属性，无论是改变 attribute 还是改变 property，对应的另一个值也会改变。

```
<input id="ak47" type="checkbox" checked="checked" />
```

```JavaScript
elem.getAttribute("id”) // "ak47"
elem.id // "ak47"
elem.id = "m16"
elem.getAttribute("id”) // "m16"
elem.setAttribute("id”, "mac10")
elem.id = "m16" // "mac10"
```

### 自定义属性

对于自定义属性，改变 attribute 或是 property，对应的另一个值不受影响（IE6、7 不区分 attribute 和 property，改变一个，对应的另一个值也会改变）。

```
<input test="abcde" type="checkbox" checked="checked" />
```

```JavaScript
elem.attributes // 类数组 test: "abcde", type: "checkbox", checked: "checked"
elem.test // undefined 不包含自定义的 attribute
elem.testName = "tianfangye" // 自定义的 property
elem.testName // "tianfangye"
elem.getAttribute("testName") // null 不包含自定义的 property
```
### boolean attribute

*boolean attribute* 不同于上面两种，这种属性有特殊的处理方式。例如 checked，只要存在 attribute，不论值是什么（checked="", checked, checked="false"），对应的 property 都是 true。

checked attribute 并不对应 checked property，而是对应 defaultChecked property。这是一个初始值，checked property 变化后它并不会改变（所以 checked attribute 也不变）。根据 jQuery 的文档，selected 与 value 也是如此。

jQuery 中检测是否选中可以用如下三种方式：

```JavaScript
if ( elem.checked )
if ( $( elem ).prop( "checked" ) )
if ( $( elem ).is( ":checked" ) )
```

### [jQuery 中的 .attr() 与 .prop()](http://jquery.com/upgrade-guide/1.9/#attr-versus-prop)

jQuery 中的 .attr() 方法稍有混乱，有些应当使用 attribute 的地方使用了 property。

```JavaScript
elem.checked // true (Boolean) Will change with checkbox state
$( elem ).prop( "checked" ) // true (Boolean) Will change with checkbox state
elem.getAttribute( "checked" ) // "checked" (String) Initial state of the checkbox; does not change
$( elem ).attr( "checked" ) // (1.6) "checked" (String) Initial state of the checkbox; does not change
$( elem ).attr( "checked" ) // (1.6.1+)    "checked" (String) Will change with checkbox state
$( elem ).attr( "checked" ) // (pre-1.6)   true (Boolean) Changed with checkbox state
```

下面两种选择器在 1.9 之前的版本中可能会不准确：

1. "input[checked]"
选择所有存在 checked attribute 的元素。

2. "input:checked"
选择所有选中的元素。



















