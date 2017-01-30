---
title: 我就叫它.. 叉叉 Manager 吧
date: 2017-01-30 20:47:48
tags:
  - 翻译
  - 代码规范
  - 命名
categories:
  - 翻译
  - 技术译文
---
关于编程实践中的命名规范, 大学舍友在舍群里分享了一篇文章 [I Shall Call It.. SomethingManager](https://blog.codinghorror.com/i-shall-call-it-somethingmanager/), 觉得蛮有意思而且感同身受, 简单翻译一下分享给大家.

<!--more-->

Alan Green 在  [the meaninglessness of SomethingManager](http://www.bright-green.com/blog/2003_02_25/naming_java_classes_without_a.html) 中抱怨:
>你遇见过多少叫做叉叉管理器的类? 任何像样规模的商业系统似乎都有着一大堆
>- SessionManager # 对话管理器
>- ConnectionManager # 连接管理器
>- PolicyManager # 策略管理器
>- QueueManager # 队列管理器
>- UrlManager # 统一资源定位符管理器
>- ConfigurationManager # 配置管理器
>- 或者再悲剧一点 EJBManager # 一级棒管理器 2333
>
>随便在字典中扫一眼  ["manager"](http://www.merriam-webster.com/dictionary/manager) 或者 ["manage"](http://www.merriam-webster.com/dictionary/manage)  条目, 你就能发现至少十种不同的意思. 从 "使人顺从" 到 "成功做到了某事", 又比如某天我碰到一个接线员称呼自己为 "总机管理员 ", 这些各式各样的定义中, 通用的语义好像就是一个模糊的概念 --  "照看好某些东西".
>
>这样的不确定性使得在为类命名的时候,  "Manager" 成为了一个非常糟糕的选择. 举个例子, 有个叫做 UrlManager 的类 -- 你没办法从名字中知道它是存储 Url 用的还是操作 Url 用的还是用来审计 Url 的使用情况的. 从名字中获得的仅有的信息是这个类完成了一些和 Url 相关的操作. 而 "UrlBuilder" 这样的名字就能更清楚地告诉你这个类是做什么用的.
>
>在 Java 世界中, "Manager" 这个后缀到处被滥用, 一旦有一个类以任何形式对其他对象起到一个负责作用, 这个类总是被自动打上管理器的标签.

再没没有比叉叉 Manager 更含糊的概念了, 为了避免这个单词, Alan 在[他的博客](http://www.bright-green.com/blog/2003_02_25/naming_java_classes_without_a.html)中给出了几个替代性的建议, 或许能够更加精确地表达出你的类到底是用来做什么的.

给自己的类和其他对象取一个描述清晰的好名字不是一件容易的事情, Steve McConnell 在 [代码大全](https://www.amazon.cn/dp/B0061XKRXA/) 这本书中给出了几个有用的指导性意见:

>1. 描述清楚你的代码流程做的所有事情
>   对, 我们说的是实打实的所有事情, 如果这让你的命名变得可笑地冗长, 名字不是问题, 可能是你的代码流程有问题
>2. 避免无意义, 混淆, 软弱无力的动词, 比如:
>   - UrlManager # 统一资源定位符管理器
>   - HandleOutput() # 处理输出?
>   - PerformServices() # 运行服务?
>3. 别单用数字来区分不同的代码流程(真是做大死)
>   我只是为了完整性才在这里提这件事情, 如果你的代码中居然有 `OutputUser1()` 和 `OutputUser2()` 这样的名称, 那就只能让上帝保佑你和你的团队了.
>4. 名字越长越好
>   根据 McConnell 的说法, 命名的最佳长度大约在 9 - 15 个字母, 代码流程(函数)因为处理了更复杂的流程所以应该要有更长的名称, 为了使你的代码更容易地被人读懂, 请尽可能地选用长一点的名字.
>5. 对于函数, 尝试使用对返回值的描述作为名称
>   这个规则简单直接, 举些例子: `printer.IsReady()`, `pen.CurrentColor()` 等等
>6. 使用精确的反义词
>   对应 `Open()` 使用 `Close()`
>   对应 `Insert()` 使用 `Delete()`
>   对应 `Start()` 使用 `Stop()`
>7. 对通用的操作建立命名规约
>   举一个反面例子: 如果代码同时存在 `employee.id.Get()`, `dependent.GetId()`, `supervisor()`, `candidate.id()` 这些命名方式, 你能知道如何获得 `newbee` 的 id 吗

我得说我的重构行为中, 最常做的就是重命名各种类和变量. [取一个好名字是一件难事](http://martinfowler.com/bliki/TwoHardThings.html), 它也应该困难, 因为一个好名字需要利用三两个单词抓住关键的意思. 有时候在你完全写完所有逻辑之前, 你很难知道该如何命名这块代码. 而在实际生产中, 代码是逻辑总是写不到头的, 所以名字可能一直都在变.

一个简洁的, 描述性的变量, 对象或函数名能为你的代码代码许多正面积极的作用, 否则就这能沦为 [Daily WTF code](http://thedailywtf.com/) 中用来举反例的货色了.
