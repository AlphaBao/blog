title: addEventListener 多重监听问题
date: 2014-02-17 22:17:55
comments: true
tags: [JavaScript, event]
categories: 编程
---

# 不会导致多重监听的情况

```JavaScript
var listener = function(event) {
  alert("JS");
};

document.body.addEventListener('click', listener);
document.body.addEventListener('click', listener);

// 每次点击只弹出一次
```
多次 add 同一个 function 是不会重复监听的。只要 listener 没有变，add 多少次也只相当于 add 一次。

### 删除 listener
```JavaScript
document.body.removeEventListener('click', listener);
```

由于只有一个 listener，只要这样调用一次 removeEventListener 即可删除。

<!--more-->
# 会导致多重监听的情况

```JavaScript
var listener = function(event) {
  alert("JS");
};

document.body.addEventListener('click', listener);

var listener = function(event) {
  alert("JS");
};

document.body.addEventListener('click', listener);

// 每次点击会弹出两次
```

虽然还是同一个 listener，但第二次 var listener 的时候其实创建了一个新的 function（JavaScript 中 function 也是对象）。所以这相当于 add 两个不同的 listener 一样。

另外，由于第二次 var listener 时候已经丢失了第一次 var listener 的引用，所以这种情况下已经无法删除第一次添加的 listener。


# 另一种“多重”监听

```JavaScript
var listener = function(event) {
  alert("JS");
};

document.body.addEventListener('click', listener, true);
document.body.addEventListener('click', listener);

// 每次点击会弹出两次
```

这种情况下也会有两个 listener，第一个在捕获阶段执行，第二个在冒泡阶段执行。

### 删除 listener
```JavaScript
document.body.removeEventListener('click', listener, true);
document.body.removeEventListener('click', listener);
```

如上所示，这时需要 removeEventListener 两次，第一次是删除捕获阶段的 listener，第二次是删除冒泡阶段的 listener。

### 捕获与冒泡

由上面的现象可知，addEventListener 与 removeEventListener 是严格区分捕获阶段与冒泡阶段的。同一个 listener，可以分别在捕获阶段和冒泡阶段监听；相对的，删除时也要区分要删除的是捕获阶段的 listener 还是冒泡阶段的 listener。







