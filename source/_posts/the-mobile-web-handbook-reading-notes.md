title: PPK《移动Web手册》读书笔记
date: 2015-10-01 12:16:06
comments: true
tags: [Web, Mobile]
categories: 编程
---

## 浏览器

大多数内置浏览器都被紧密集成到底层的操作系统中，无法单独升级。设备供应商经常拒绝给他们的内置浏览器命名，iOS 的内置浏览器叫作 Safari，而 Android、黑莓、塞班的内置浏览器没有名字，作者把它们叫作「安卓WebKit」「黑莓WebKit」「塞班WebKit」。


### 安卓

一个智能系统需要一个浏览器，安卓系统提供了基于 WebKit 的浏览器；它没有名字，英文网站上一般称之为「Android Browser」，作者称之为「安卓WebKit」。

安卓WebKit 并不是 Chrome：它有完全分离的代码库，里面有完全分离的漏洞。查看 userAgent 是否含有 Chrome 这个单词，可以区分 安卓WebKit 与 Chrome。安卓WebKit 包含大量开关，用来打开或关闭特定功能，设备厂商可以随心所欲地设置这些开关。所以 三星安卓WebKit 与 索尼安卓WebKit 并不相同。
<!-- more -->

谷歌想利用 Chrome 代替 安卓WebKit，这有利于谷歌收集用户数据并提供更强大的广告；设备厂商系统自己获取用户数据，并且希望继续使自己的设备与众不同。

谷歌强制所有使用谷歌服务的设备使用 Google Chrome，但设备厂商能改变手机的内置浏览器。 HTC M8 的内置浏览器是 安卓WebKit，但它也安装了 Google Chrome；三星 S4 安装了两款基于 Chromium（人人都可以下载和修改的谷歌开源浏览器）的浏览器：三星Chrome 和 Google Chrome。

亚马逊拒绝了谷歌服务，因此 Amazon Kindle Fire 中内置的是 Silk（基于 Chromium）。

绝大多数国内厂商，比如小米、魅族的情况类似亚马逊，它们自己定制了安卓系统，也就定制了自己的浏览器（它们都不需要带 Google Chrome）。


### WebView

WebView 是留给原生应用的一个操作系统浏览接口。大体上，WebView 是独立的程序，用了内置浏览器很多底层的组件（比如渲染引擎），但是在其他方面可能会有所不同。

