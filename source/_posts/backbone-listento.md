title: Backbone 中的 listenTo
date: 2013-09-18 23:43:52
comments: true
tags: [JavaScript, Backbone]
categories: 编程
---

## Backbone 0.9.x 中新增的 listenTo

Backbone 0.9.x 中新增的 listenTo 是十分有用的方法，有了它就能更方便的将 view 中的方法当做 callback 注册与解除。linstenTo 是 Backbone.Events 对象的方法。

```JavaScript
var Events = Backbone.Events = {

  // ......

  var listenMethods = {listenTo: 'on', listenToOnce: 'once'};
   
  _.each(listenMethods, function(implementation, method) {
    Events[method] = function(obj, name, callback) {
      var listeners = this._listeners || (this._listeners = {});
      var id = obj._listenerId || (obj._listenerId = _.uniqueId('l'));
      listeners[id] = obj;
      if (typeof name === 'object') callback = this;
      obj[implementation](name, callback, this);
      return this;
    };
  });

  // ......
}
```

<!--more-->each 之后产生的 listenTo 方法就是：

```JavaScript
function (obj, name, callback) {
    var listeners = this._listeners || (this._listeners = {});
    var id = obj._listenerId || (obj._listenerId = _.uniqueId("l"));
    listeners[id] = obj;
    if (typeof name === "object") callback = this;
    obj[implementation](name, callback, this);
    return this
}
```

从这部分源码发现 listenTo 使用 this.model.on('change', this.render, this) 的形式注册 callback，相当于调用 on 时指定了 context，避免由 this 关键字导致的作用域问题。

既是 listenTo，必能 stopListening。

```JavaScript
var Events = Backbone.Events = {

  // ......

  stopListening: function(obj, name, callback) {
    var listeners = this._listeners;
    if (!listeners) return this;
    var deleteListener = !name && !callback;
    if (typeof name === 'object') callback = this;
    if (obj) (listeners = {})[obj._listenerId] = obj;
    for (var id in listeners) {
      listeners[id].off(name, callback, this);
      if (deleteListener) delete this._listeners[id];
    }
    return this;
  }

  // ......
}
```

这里会取出 listenTo 时更新的 this._listeners，然后调用与 on 相对的 off 方法解除注册。
观察方法签名 object.stopListening([other], [event], [callback])；正常情况下，其中的 object 就是 Backbone view，而第一个参数 other 就是 Backbone model。无参数的调用会解除所有通过 listenTo 注册的方法，也可以通过传参只解除特定 model 注册的方法或只解除对特定事件的注册。

```JavaScript
view.stopListening();
view.stopListening(this.model);
view.stopListening(this.model, "change", this.render);
```


## 两种注册 callback 的方式

Backbone view 中有两种注册 callback 的方式：

1.将 callback 注册到 DOM 节点的事件：
```JavaScript
events: {
  "dblclick"                : "open",
  "click .icon.doc"         : "select",
  "contextmenu .icon.doc"   : "showMenu",
  "click .show_notes"       : "toggleNotes",
  "click .title .lock"      : "editAccessLevel",
  "mouseover .title .date"  : "showTooltip"
}
```

在创建新的视图即 new View() 的时候会调用 view.delegateEvents() 方法完成此种绑定。

解除此类 callback:
```JavaScript
view.undelegateEvents();
```

2.将 callback 注册到 model 对象的事件：
```JavaScript
view.listenTo(model, 'change', view.render);
```

解除此类 callback:
```JavaScript
view.stopListening();
```


## 用 listenTo 方法避免内存泄露

之所以需要 listenTo 是为了能够 stopListening，那么 stopListening 是为了什么呢？

```CoffeeScript
AppView = Backbone.View.extend

  initialize: ->
    @listenTo @model, "change", @render

  render: ->
    console.log "Knowledge is power."

Goods = Backbone.Model.extend

  initialize: ->

goods = new Goods

appView = new AppView model: goods

appView.remove()

appView = new AppView model: goods

goods.set "date", new Date()
```

运行上面的代码（CoffeeScript），控制台输出一次；注释掉其中的 appView.remove() 这一行再运行，控制台输出两次！

毫无疑问，重新给 appView 变量赋值之后，第一个 view 并没有释放掉。这是因为执行 listenTo 的时候，已经将当前 view 的一个方法当做 callback 注册到了 model，重新给 appView 变量赋值时，新 view 中的 listenTo 又向 model 注册了一个 callback，所以控制台输出两次。也因为 model 里面存在上一个 view 的引用，上一个 view 不会被 GC 回收。

想要删除一个视图的时候应该使用 view.remove() 方法：
```JavaScript
_.extend(View.prototype, Events, {

  // ......

  remove: function() {
    this.$el.remove();
    this.stopListening();
    return this;
  },

  // ......
}
```

它做了两件事：

1.调用 $el.remove() 解除了 view.delegateEvents() 方式注册的 callback。$el 是 jQuery object，所以 $el.remove() 就是调用了 jQuery 的方法，作用是：将 el 从 DOM 中删除，并且删除 el 上注册的事件和 el 上的 jQuery data。

2.调用 stopListening 解除 listenTo 方式注册的 callback。这样 model 中就没有了 view 的引用，即避免了内存泄露。




