title: 使用 Karma、Mocha、Chai 搭建支持 ES6 的测试环境
date: 2016-09-02 11:17:55
comments: true
tags: [Karma, Mocha, Webpack, ES6]
categories: 编程
---

前端开发很多是界面开发，但我们可以将相对独立的逻辑和功能从整体业务逻辑中独立出来，这样就可以对它们做单元测试。使用 Karma 可以比较方便地搭建出测试环境。

### 安装 Karma

使用 `Karma` `Mocha` `Chai`（启动器、测试框架、断言库）组合。

```shell
npm install karma karma-mocha karma-chai --save-dev
```

如果 npm 版本 >=3.0，会看到如下提示：

> UNMET PEER DEPENDENCY chai@*
> karma@1.2.0 
> karma-chai@0.1.0 
> karma-mocha@1.1.1 
> UNMET PEER DEPENDENCY mocha@*

这是因为 npm 已经不再自动安装 peerDependencies：
[We will also be changing the behavior of peerDependencies in npm@3. We won’t be automatically downloading the peer dependency anymore. Instead, we’ll warn you if the peer dependency isn’t already installed. This requires you to resolve peerDependency conflicts yourself, manually, but in the long run this should make it less likely that you’ll end up in a tricky spot with your packages’ dependencies.](http://blog.npmjs.org/post/110924823920/npm-weekly-5)

<!--more-->
于是继续安装 `mocha` `chai`

```shell
npm install mocha chai --save-dev
```

### 初始化 Karma

```shell
karma init
```

然后要回答一系列问题。

> Which testing framework do you want to use ?
> Press tab to list possible options. Enter to move to the next question.
> \> mocha
> 
> Do you want to use Require.js ?
> This will add Require.js plugin.
> Press tab to list possible options. Enter to move to the next question.
> \> no
> 
> Do you want to capture any browsers automatically ?
> Press tab to list possible options. Enter empty string to move to the next question.
> \> Chrome
> 
> What is the location of your source and test files ?
> You can use glob patterns, eg. "js/*.js" or "test/**/*Spec.js".
> Enter empty string to move to the next question.
> \> "test/**/*.spec.js"
> 01 09 2016 16:43:20.743:WARN [init]: There is no file matching this pattern.
> 
> \> 
> 
> Should any of the files included by the previous patterns be excluded ?
> You can use glob patterns, eg. "**/*.swp".
> Enter empty string to move to the next question.
> \> 
> 
> Do you want Karma to watch all the files and run the tests on change ?
> Press tab to list possible options.
> \> yes

然后就可以看到 `Karma` 已经创建的配置文件 `karma.conf.js`。如果选择使用 `PhantomJS`，需要单独安装。

### 添加 ES6 支持

现在前端开发的源码一般使用了 ES6 甚至 ES7，将这个处理工作用 webpack 搞定。

```shell
npm install karma-webpack --save-dev
```

既然将 ES6 的处理交给 webpack，如果之前没有安装过 babel 环境，还需要安装 `babel-core` `babel-preset-es2015` 以及 `babel-loader`。 

如果出现下面的 `TypeError` 错误，只要在 exclude 中加入 /node_modules/ 就好了。
> TypeError: 'caller', 'callee', and 'arguments' properties may not be accessed on strict mode functions or the arguments objects for calls to them

配置文件 `karma.conf.js` 中，需要注意的还有 `files` `preprocessors` 以及 `webpack` 部分。

```javascript
// Karma configuration
module.exports = function(config) {
  config.set({
    // ......

    files: [
      'test/**/*.spec.js'
    ],

    preprocessors: {
      'test/**/*.spec.js': ['webpack']
    },

    webpack: {
      resolve: {
        root: __dirname + "/src"
      },
      module: {
        loaders: [{
          test: /\.js$/,
          exclude: [/node_modules/, __dirname + "xxx/xxx/lib"],
          loader: "babel-loader",
          query: {
            compact: false,
            presets: ["es2015"],
            plugins: ["es6-promise"]
          }
        }]
      }
    },

    // ......
  })
}
```

### 启动 Karma

编写测试用例，这里是一个使用断言库 `Chai`，并使用它的 `expect` 断言风格的例子。

```javascript
import {getMoneyText} from "xxx/xxx.js";
import {expect} from "chai";

describe("生成价格文案", () => {
  it("价格文案：积分", () => {
    expect(getMoneyText({
      payType: 1,
      price: 100,
      points: 100,
    })).to.be.equal("100积分");
  });

  it("价格文案：人民币", () => {
    expect(getMoneyText({
      payType: 2,
      price: 100,
      points: 100,
    })).to.be.equal("￥100.00");
  });

  it("价格文案：人民币+积分", () => {
    expect(getMoneyText({
      payType: 3,
      price: 100,
      points: 100,
    })).to.be.equal("￥100.00+100积分");
  });

  it("价格文案：人民币+积分（多份数量）", () => {
    expect(getMoneyText({
      payType: 3,
      number: 5,
      price: 100,
      points: 100,
    })).to.be.equal("￥500.00+500积分");
  });
});
```

启动 `Karma`

```shell
karma start
```

关于 Mocha （Chai, expect）的入门教程可以参考：[测试框架 Mocha 实例教程](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)
