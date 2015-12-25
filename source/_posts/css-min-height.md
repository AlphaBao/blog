title: CSS：min-height百分比问题
date: 2015-11-04 18:10:58
comments: true
tags: [CSS]
categories: 编程
---

对于高度不确定的页面，想实现总是撑满一屏的效果，可以结合 height 与 min-height 来设置，但是有一些瑕疵。

### 示例 1：

```CSS
html {
  min-height: 100%;
}
body {
  min-height: 100%;
}
```

html 与 body 都设置为 `min-height: 100%`，意图是想页面高度不足一屏时 body 高度为一屏的高度，高度超过一屏则 body 的高度为页面的高度。但这样设置后会发现， html 的高度变成了 100%（一屏的高度），而 body 的高度却是 0。那么将 body 的子元素设为 `min-height: 100%` 时，这个元素的高度自然也是 0（除非靠内容撑开）。

试验发现，如果将 min-height 的值设置为百分比，那么上一级元素必须有一个明确的高度（不能是 min-height: 100%），否则高度是 0。
<!--more-->

### 示例 2：

```CSS
html {
  height: 100%;
}
body {
  min-height: 100%;
}
```

如果 html 设置为 `height: 100%`，而 body 设置为 `min-height: 100%`，则 html 与 body 的高度都变成了 100%（一屏的高度）。但是对照上一条结论，min-height 的上一级元素必须有一个明确的高度，body 的子元素设为 `min-height: 100%` 时，这个元素的高度却依然是 0（除非靠内容撑开）。

### 示例 3：

```CSS
html {
  height: 100%;
}
body {
  height: 100%;
}
```

html 与 body 都设置为 `height: 100%`，则 html 与 body 的高度都变成了 100%（一屏的高度）。body 的子元素设为 `min-height: 100%` 时，这个元素的高度也是一屏高度。这种方式的问题在于，body 中内容变多后，高度超过一个屏幕时，body 的高度仍然是 100%（一屏的高度），不影响滚动，但看起来不自然。

_Chrome: Version 46.0.2490.80 (64-bit)_














