title: ECMAScript 6 之类与继承
date: 2015-11-23 15:18:59
comments: true
tags: [JavaScript, ES6]
categories: 编程
---

```JavaScript
class Person {
  constructor() {
    this.name='Person';
  }
}

class Bao extends Person {
  constructor() {
    super();
    this.name = "Bao";
  }
}
```

```JavaScript
"use strict";

var _get = function get(_x, _x2, _x3) {
  var _again = true;
  _function: while (_again) {
    var object = _x,
      property = _x2,
      receiver = _x3;
    _again = false;
    if (object === null) object = Function.prototype;
    var desc = Object.getOwnPropertyDescriptor(object, property);
    if (desc === undefined) {
      var parent = Object.getPrototypeOf(object);
      if (parent === null) {
        return undefined;
      } else {
        _x = parent;
        _x2 = property;
        _x3 = receiver;
        _again = true;
        desc = parent = undefined;
        continue _function;
      }
    } else if ("value" in desc) {
      return desc.value;
    } else {
      var getter = desc.get;
      if (getter === undefined) {
        return undefined;
      }
      return getter.call(receiver);
    }
  }
};

function _inherits(subClass, superClass) {
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
  }
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: {
      value: subClass,
      enumerable: false,
      writable: true,
      configurable: true
    }
  });
  if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
}

function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

var Person = function Person() {
  _classCallCheck(this, Person);

  this.name = 'Person';
};

var Bao = (function(_Person) {
  _inherits(Bao, _Person);

  function Bao() {
    _classCallCheck(this, Bao);

    _get(Object.getPrototypeOf(Bao.prototype), "constructor", this).call(this);
    this.name = "Bao";
  }

  return Bao;
})(Person);
```