title: CodeTank Pistachio
date: 2014-01-05 19:30:37
comments: true
tags: [JavaScript, 游戏]
categories: 编程
---

<iframe width="750" height="600" src="http://codetank.alloyteam.com/?cmd=battle&param=alloyteam.fire,Tianfangye.Pistachio&mode=battle&theme=transparent"></iframe>
<!--more-->

CodeTank（代码坦克）是由 腾讯 [AlloyTeam](http://codetank.alloyteam.com/) 和 HTML5 梦工场联合出品的在线坦克仿真游戏平台，构建了一个基于互联网的在线智能坦克机器人战斗仿真引擎。算是 Javascript程序员 的编程游戏，可以用 Javascript 代码控制坦克进行战斗。

我已经创建了第一个坦克，它叫——Pistachio（开心果）。Pistachio 只考虑了一对一的情况，下文也全是针对一对一而言。

玩游戏通常赢了才开心，Pistachio 战胜官方示例坦克的概率已经很高（其他的坦克也没发现什么强大的）。

##关于 Pistachio 的代码：

fire() ：
火炮力度、瞄准范围 ---- alloyTeam.trackfire。

onScannedRobot() ：
发现敌方坦克 ---- alloyTeam.fire。

onHitRobot() ：
与敌方坦克相撞 ---- alloyTeam.tracker。

上面三部分官方示例中都有比较好的处理，后面标注的是现在 Pistachio 参考的官方示例。

这个游戏最重要也是最大的挑战在于如何移动，你的坦克移动越智能，就越强大！这方面官方示例没什么帮助，胡乱走的话很容易撞墙（减血）。
###对于智能移动，目标是：

1.灵活（能各个方向的走直线或弧线）
2.不撞墙
3.容易发现地方坦克（意味着雷达应该有适当的旋转角度与速率）

###Pistachio 的智能移动：

1.检查是否刚刚开火，如果是，先小角度转动雷达（此时敌方坦克还没跑远，小角度扫描发现的概率很大）。
2.下次移动方向、距离随机产生。
3.检查下次到达位置的坐标。
4.如果坐标超出战场范围，重新生成。
5.如果坐标靠近战场边缘，炮筒与雷达跟随坦克旋转，坦克先旋转、再走直线（避免撞墙）。
6.如果坐标安全（当前坐标与预定位置坐标都不靠近战场边缘），炮筒与雷达独立于坦克旋转，坦克走弧线。

###Pistachio 被子弹击中后的处理（onHitByBullet）：

1.检查是否刚刚开火，如果是，立即雷达扫描（敌方坦克应该还没跑远）。
2.检查是否刚刚开火，如果不是，智能移动。

###Pistachio 撞墙后的处理（onHitWall）：
智能移动。
其实智能移动就是为了避免撞墙，现在撞墙的概率已经很小。

[完整代码](https://github.com/AlphaBao/codetank)

##关于其他：
CodeTank 的 API 文档太过简略，好多功能与效果缺少必要说明。


