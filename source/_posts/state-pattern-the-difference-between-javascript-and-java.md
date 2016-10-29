title: 从状态模式看 JavaScript 与 Java
date: 2016-10-29 19:51:31
comments: true
tags: [JavaScript, Java, 设计模式, 动态语言, 静态语言]
categories: 编程
---

这篇文章缘起于前几天微博上有关动态语言与静态语言的讨论，因为有几个编程高手参加，所以能看到一些特别有启发性的发言。本文主要是下面这一条微博的读后感，也是我的练习与思考。

[@有个梨UGlee：如果你去看四人帮的Design Pattern里，就有State Pattern；State Pattern用类型编码State，就是我们说的问题；但是动态语言里写出来非常简单，类型语言里写得极其繁琐。](http://weibo.com/1655747731/EebsQBk82)

关于动态语言与静态语言，有很多比较和讨论它们的文章，但大部分都没有抓住重点。而上面一条微博，提到了一个很好的切入点，那就是「状态模式（State Pattern）」。

## 状态模式：蝙蝠侠/布鲁斯·韦恩

蝙蝠侠（英语：Batman）是一名出现于DC漫画的虚构超级英雄角色，由鲍勃·凯恩和比尔·芬格创作。他的名字叫 Bruce，是一位美国亿万富翁，这是他的正常身份，用于正常生活，例如进行参加宴会之类的活动。他的另一个身份是 Batman，是打击犯罪的黑暗骑士。

这是一个状态模式的好示例，我用 Java 和 JavaScript 各写了一个示例，体会体会「极其繁琐」与「非常简单」。

<!--more-->

## 极其繁琐：Java 版本

目录结构：

    com
      |-- tianfangye
        |-- Client.java
        |-- person
            |-- Person.java
        |-- state
            |-- BatmanState.java
            |-- BruceState.java
            |-- IState.java

**Person.java**

```Java
package com.tianfangye.person;

import com.tianfangye.state.IState;
import com.tianfangye.state.BruceState;

public class Person {

  // 当前状态
  private IState state;

  public Person() {
    this.state = new BruceState();
  }

  // 设置当前状态
  public void changeState(IState state) {
    this.state = state;
  }

  // 改变状态（变身）
  public void convertState() {
    this.state.convertState(this);
  }

  // 开始行动
  public void takeAction() {
    this.state.doActivities();
  }
}
```

**IState.java**

```Java
package com.tianfangye.state;

import com.tianfangye.person.Person;

public interface IState {

  // 转换状态
  public void convertState(Person person);

  // 执行活动
  public void doActivities();
}
```

**Bruce.java**

```Java
package com.tianfangye.state;

import com.tianfangye.person.Person;

public class BruceState implements IState {

  private String name = "- Bruce -";

  // 转换状态
  public void convertState(Person person) {
    person.changeState(new BatmanState());
  }

  // 执行活动
  public void doActivities() {
    System.out.println(this.name + " <> " + "参加宴会");
  }
}
```

**Batman.java**

```Java
package com.tianfangye.state;

import com.tianfangye.person.Person;

public class BatmanState implements IState {

  private String name = "- Batman -";

  // 转换状态
  public void convertState(Person person) {
    person.changeState(new BruceState());
  }

  // 执行活动
  public void doActivities() {
    System.out.println(this.name + " <> " + "打击犯罪");
  }
}
```

**Client.java**

```Java
package com.tianfangye;

import com.tianfangye.person.Person;

public class Client {
  public static void main(String[] args) {
    Person person = new Person();
    person.takeAction();
    person.convertState();
    person.takeAction();
  }
}
```

程序输出：

* `- Bruce - <> 参加宴会`
* `- Batman - <> 打击犯罪`

关于状态模式的实现，GoF 的 Design Patterns 里面提到过一些需要考虑的方面，其中之一是「谁来定义状态的转换」。可以由状态的使用者（Person）实现，也可以由每个状态各自实现，各有利弊。上例由每个状态各自实现，接下来的 JavaScript 示例也是这种选择（对于示例程序，另一种实现更简单）。

## 非常简单：JavaScript 版本

```JavaScript
const state = {
  _Bruce: {
    name: "- Bruce -",
    convertState() {
      this.identity = state._Batman;
    },
    takeAction() {
      window.console.log(`${this.name} <> 参加宴会`);
    }
  },

  _Batman: {
    name: "- Batman -",
    convertState() {
      this.identity = state._Bruce;
    },
    takeAction() {
      window.console.log(`${this.name} <> 打击犯罪`);
    }
  }
};

const person = {
  identity: state._Bruce,
  convertState() {
    this.identity.convertState.call(this);
  },
  takeAction() {
    this.identity.takeAction();
  }
};

person.takeAction();
person.convertState();
person.takeAction();
```

程序输出：

* `- Bruce - <> 参加宴会`
* `- Batman - <> 打击犯罪`

可以看到，相比静态语言的版本，动态语言的版本竟是如此浑然天成！甚至不是在实现「设计模式」，只是对语言特性的正常使用而已。所以，什么是设计模式？聪明人应该想通了——设计模式是一个衍生问题，不是本质问题。本质问题是程序设计，是静态语言与动态语言，是静态类型与动态类型，是编程语言抽象。

[@vczh](https://www.zhihu.com/people/excited-vczh)的一个知乎回答对这一点讲得很透彻：

[设计模式说白了就是在不允许使用dynamic\_cast的情况下如何让你的设计通过类型系统的检验，于是发明出了一大堆行之有效的做法。为什么不能用dynamic\_cast？因为通常如果你可以通过修改设计来避免所有dynamic_cast，那你就得到了最优的性能（通常指的是系数，不是复杂度）。](https://www.zhihu.com/question/23757237/answer/102665315)

[但是很多人都知其然不知其所以然，盲目的背诵却不练习（这并不是让你不要去背诵），喜欢过度设计控制不住自己，还可以拿它来装逼等等。其实这都不是设计模式的问题，而是人的劣根性的问题，这些人不管学什么都是一样的，只不过设计模式把这些人的效果放大了。](https://www.zhihu.com/question/23757237/answer/102665315)

显而易见，类型是个大问题！动态语言能做到非常简洁是由于动态类型系统的灵活性，而这些灵活性并非没有代价（性能）。另外，关于动态语言与静态语言的选择，一直有很多工程问题上的争议（从文章开头的微博往下探索，可以找到大段这方面的争论）。我个人对于动态语言、静态语言没有明显偏好，只是喜欢把问题弄清楚——不论有没有偏好，偏好强烈还是微弱，这都是首要的一步。
