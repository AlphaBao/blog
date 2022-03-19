title: CSRF 原理、攻击、防御及其他
date: 2022-03-19 17:20:08
comments: true
tags: [CSRF, Cookie, 网络安全]
categories: 编程
---

# CSRF 概述：

CSRF 就是 Cross Site Request Forgery，跨站请求伪造。这种攻击利用了浏览器端状态（Cookie）管理规则与服务端鉴权规则。CSRF 在攻击过程中不需要获取到用户的登录凭据，而是借用户之手发出恶意请求。

CSRF 的存在是由于 Cookie 架构的设计“非常古老”，已经有些不适应网络安全的需要。但是如下文所述，现代浏览器已经给 Cookie 规则打了很多“补丁”，安全性大大提升。实际上，现代浏览器条件下，很多“传统的” CSRF 攻击手段已经失效，但是由于网络安全的短板效应，永远不能放松警惕，只有深入理解攻击方式的来龙去脉，才能从整体上尽可能确保安全。

另一个 Web 开发者常听说的攻击方式叫 XSS（Cross Site Scripting），跨站脚本，了解它的人应该更多一些。相比而言，CSRF 攻击构造起来可以更简单，更低成本，但是攻击方式更精巧，背后的原理的也更复杂一些。更容易导致混乱的地方在于，不同的攻击方式可能混合出现。比如考虑到 UGC（User-generated content）场景下可能出现的 CSRF，就变得比典型的 CSRF 要复杂。对于整体防御思路来说，是很大的挑战。

狭义上的 CSRF 知识不难理解，但要深入理解就比较繁琐，因为网络安全的事情往往都是很多不同层面的知识互相关联，CSRF 尤其如此。

# 与 CSRF 关系密切的领域：

1. 网络协议
2. 浏览器特性
3. API设计
4. 认证方式
5. 密码学
6. 社会工程

可见，CSRF 本身不复杂，甚至看起来比较简单，但是每个相关的领域，都是个非常深的坑，本文会尽可能设计到足够广的领域。阅读本文之后，会发现每个领域的知识都有很多可以继续探索的地方。<!--more-->

# CSRF 存在的原因：

浏览器 Cookie 架构的安全性不足，传统上是通过 domain 和 path 与 document 的匹配情况来携带。

恶意页面可以通过诱导点击链接或者通过隐藏的 iframe 提交 form 表单（通常自动提交），让用户打开恶意页面就可能触发请求，由于带着 Cookie，这个请求在服务端会被认为是合法的。

# CSRF 攻击的特点：

1. 可以权限提升；
2. 具有隐蔽性（被攻击者不易察觉）；

# CSRF 防御设计的应用范围：

1. 对于触发数据修改的接口；
2. 对于获取敏感信息的接口；

# 典型的 CSRF 攻击示例

## Forge GET

```html
<a href="http://mysite.com/transfer.do?acct=MARIA&amount=100000">View my Pictures!</a>

<img src="http://mysite.com/transfer.do?acct=MARIA&amount=100000" width="0" height="0" border="0">
```

## Forge POST

```html
<form action="/action\_page\_binary.asp" method="post" enctype="multipart/form-data">
  <label for="fname">First name:</label>
  <input type="text" id="fname" name="fname">
  <label for="lname">Last name:</label>
  <input type="text" id="lname" name="lname">
  <input type="submit" value="Submit">
</form>
```

## CSRF 攻击的示例步骤：

### CSRF GET：

1. Bob 创建了含有 GET 请求的恶意链接
2. Bob 通过聊天工具、邮件或网站传播恶意链接
3. Alice 点击了恶意链接
4. 以 Alice 的身份发起了 GET 请求
5. 服务端以 Alice 认证的 Cookie 来处理请求
6. 服务端按照 Alice 的身份更新了状态（Alice 并不知情）

### CSRF POST：

1. Bob 创建了恶意网页
2. Bob 传播该网页
3. Alice 点击恶意网页并提交了表单（可能有填写动作、也可能是自动提交）
4. 表单发起提交到另一个站点的 POST 请求
5. 服务端以 Alice 身份处理 POST 请求
6. 服务端状态已改变

# CSRF 攻击手段：

## 通过恶意页面上的 URL 或者 form 表单（通常使用 JavaScript 自动提交）：

让用户打卡恶意页面就触发请求，由于带着 Cookie，这个请求在服务端会被认为是合法的。

## 攻击者有权限在本域发布内容（UGC 场景）：

CSRF 攻击大多数情况下来自第三方域名，但并不能排除可能从本域发起的攻击。如果攻击者有权限在网站（本域）发布评论（尤其是可以设置链接、图片等），那么它可以直接在本域发起攻击，这种情况下**同源策略**无法达到防护的作用。

## 服务端请求伪造（SSRF，Server Side Request Forgery）：

这类攻击本质上是一种注入。利用了服务端的有代理请求功能：服务端可能根据用户的输入，在服务端构造一个 URL，并向其发起请求，通过访问其他的服务端资源来完成正常的页面展示。

这类情况下，如果恶意用户提交的 URL 是操作内网的敏感资源，就可能意外修改的服务端资源的状态，也可能通过一系列操作“提权”成功。由于请求是从内往发起，往往已经绕过大部分的认证和授权机制，这一 “action” 就会具备很高的权限，所以 SSRF 很可能产生非常严重的后果。

不幸的是，对于各种注入攻击，往往难以 100% 发现和防范，越是功能复杂的网站越是如此。

### SSRF 示例：

```
http://image.mysite.com/search/detail?callback_url=127.0.0.1:8081/update/1/1
```

攻击者可能通过 SSRF 直接提权，也可能通过 SSRF 拿到了服务端返回的内部错误提示、源码等信息，根据这些信息可能找到了一个 SQL 注入的漏洞，再利用 SQL 注入攻击拿到内网的命令执行权限。

# CSRF 防御手段：

## 通过浏览器原生 Header：

这个防御策略主要针对 CSRF 最常规的情况——跨域场景。它的设计思路是在服务端拦截外域（不受信任的域名）对服务端发起的请求。在 HTTP 协议中，有两个 Header 字段可以用来判断来源域：Origin 和 Referer。

浏览器发送请求时，这两个字段会自动携带，而且前端 JavaScript 代码无法对其进行修改。Origin 和 Referer 的规则在老旧浏览器上会有差异（可能不支持，也就无法起到防御的作用）。

## 通过浏览器 Custom Header：

通过 JavaScript 添加自定义的 Header 字段。缺点主要是使用成本（代码的侵入性）。

## CSRF Token：

CSRF Token 是比较彻底的防御 CSRF 攻击的方法，可以防御从简单到复杂的各种情况（比如同域、XSS 等）。它的原理是将 CSRF Token 放在 HTTP Header 或者请求参数中，服务端对其进行校验。

### CSRF Token 应该满足三个特性：

1. 每个用户唯一
2. 机密性
3. 不可预测(Unpredictable)    

如果需要防范本域的CSRF，需要为每一个 form 表单生成唯一的 Token，并且在 form 提交时验证 Token，就是 CSRF Token 的实现思路，但是 Token 需要保证不可预测，并且要区分内链、外链（避免将 Token 发送给其他服务器造成 Token 泄漏）。

一个优化手段是无状态（stateless）CSRF Token（服务端保存密钥），利用密码学原理，不需要服务端保存每个 Token，而是只需要保存一个密钥，Token 都根据这个密钥生成，给每个用户的 Token 加盐（salt）。如此一来，既可以保证能够识别这个 Token 是否是服务端生成的（其他人无法伪造），也可以保证每个用户的 Token 不相同（其他人无法猜到我的 Token 是什么）。

这种无状态设计是很多开发领域都推崇的设计原则，它的优点在于占用的服务器资源更少，而且通常能够通过减少耦合有效降低系统的复杂性。

### koa csrf 的防御方式（Stateless CSRF Token）：

koa csrf 的实现方式就是无状态的 CSRF Token，即前面所述的服务端只保存一个密钥。好处也是显而易见的，对于分布式 Web 应用，为了达到同步状态的效果，Session 会使用中间件存储或者动态计算。koa 的中间件存储方案就是将 Token 存储在 Redis 上，这样可以保证不同服务器取得的同一登录用户的 Session 值一致；由于 koa csrf 的密钥存储在 Session 中，可以保证任何一台分布式服务器取到 Token 后都可以执行解密操作并进行数据正确性比对。

### koa csrf 相关源码

[https://github.com/koajs/csrf](https://github.com/koajs/csrf)
[https://github.com/pillarjs/csrf](https://github.com/pillarjs/csrf)

## 超短期 Cookie

具体的 Token 发送方式，还有一种是使用超短期 Cookie；如1秒后就过期的Cookie，在请求时候前端 set 这个超短期 Cookie。

## Double Submit Cookie

Double Submit Cookie 基本都设计为无状态，因为使用这种方式本身就是出于便利性的考虑。服务端将满足随机性的 CSRF Token 放在 Cookie 中，浏览器端发起请求时将这个 Token 放在请求参数或 HTTP Header 中。

考虑 UGC 场景和**子域安全性**的话，需要将 Cookie 中的 Token 加密 / hash（消息摘要）/ HMAC。

### egg 主要使用 Double Submit Cookie 策略

[https://github.com/eggjs/egg/issues/260](https://github.com/eggjs/egg/issues/260)

### egg csrf 源码

[https://github.com/eggjs/egg-security](https://github.com/eggjs/egg-security)

### Double Submit Cookie 与 Login Cookie 和 CSRF Token 的关系

1. CSRF Token 与 Cookie 没有必然联系，只要满足 CSRF Token 应该满足的那三个特性就可以，比如还可以放在 input type="hidden" 中，甚至任何资源中（动态html、js）；
2. 无状态又没有过期时间的情况下，如果 Token 泄漏就可以重放攻击，或者在本域攻击的条件下用低敏感操作的 Token 做高敏感操作；
3. 登录认证的安全性与 csrf 侧重点不同，所以通常不使用同一个 Cookie，其实用同一个也可以。登录认证可能是需要加密解密的信息（成本高于 csrf 的 hash/hmac），那么认证信息也就不能是无状态的；
4. 如果登录认证使用无状态 Token 校验（服务端只需要知道这个 Token 是不是自己生成的，不需要解密或者不需要其他状态信息），可以当 CSRF Token 使用；

## SameSite Cookie

SameSite Cookie 就是浏览器为增强 Cookie 安全性而打的补丁之一。从 Chrome 80 开始，如果开发者不做设置，则当作 SameSite=Lax 对待（规则见下文）。这也是文章开头说很多“传统的” CSRF 攻击手段已经失效的原因，即浏览器对第三方 Cookie 的限制越来越严格了。

### 设置为 Strict

完全禁止第三方 Cookie。

### 设置为 Lax

携带 Cookie：

1.  从第三方站点的链接打开；
2.  预加载请求；
3.  从第三方站点提交 Get 表单；

不会携带 Cookie：

1.  第三方站点 Post 表单；
2.  通过 img、iframe、script 等标签加载的 URL；

### 设置为 None

不针对第三方 Cookie 进行限制。

## 通过人机交互手段进行防御

1. 验证码
2. 人脸识别

比 CSRF Token 更可靠的方式是验证码、人脸识别等“人工形态的 CSRF Token”。当然，缺点也非常明显，需要中断操作流程，对用户是个强打扰。

# GET 与 POST “真正”的区别

从安全层面来说，GET 与 POST 真正的区别在于，GET 是一种缺省操作，很多时候出于业务需要，不得不放行 GET 请求。

例如来自搜索引擎页面或者其他正常站点的链接，用户点击之后的跳转就是 GET 请求，不应该判定为 CSRF 攻击。由此可见，如果 Web 应用实现上允许用户通过 GET 请求发送敏感操作，更容易出现问题，所以敏感操作不使用 GET 请求通常是一个好的实践。

# JSON 劫持（JSON Hijacking）

JSON 劫持是指通过 script 标签可以载入跨域资源的特性，获取接口数据或者执行恶意的 JSONP callback function。

实际上，由于 SameSite=Lax 的保护（通过 script 标签加载时不会发送 Cookie），再加上 JavaScript 原生对象劫持的失效，这是个基本已经失效的攻击方式（由于 JavaScript 语言的动态性极强，相对容易出现这类劫持漏洞，也许还有新的劫持方法），下面的示例只用于理解攻击意图。

## 示例：利用 JavaScript 原生对象劫持

攻击者可以在自己的站点通过 script 标签载入接口数据。

```html
<script src="http://www.mysite.com/api/getXXX"></script>
```

用户已登陆 www.mysite.com 的情况下，如果访问了这个恶意页面，因为 script 标签会自动解析返回的 JSON 数据，攻击者通过 Object.prototype.__defineSetter__ 劫持原生对象的行为，触发攻击。

## 示例：利用 JSONP callback function

```html
<script>
function doSomeAction (v) {
  console.log(v);
}
</script>
<script src="http://www.mysite.com/api/getXXX?callback=doSomeAction"></script>
```

攻击者通过指定自己的恶意 JSONP callback function 进行攻击。

## 对 JSON 劫持的防御

1. 对于敏感 Cookie 不使用 SameSite=None；
2. 在获取的字符串第一行设置死循环语句。自己的程序中，获取数据时先当作字符串，去掉第一行，再 Parse JSON String；

# 参考资料

1. [Cross Site Request Forgery (CSRF)](https://owasp.org/www-community/attacks/csrf)
2. [Understanding Cross-Site Request Forgery (CSRF or XSRF)](https://danilocesar.hashnode.dev/understanding-cross-site-request-forgery-csrf-or-xsrf)

