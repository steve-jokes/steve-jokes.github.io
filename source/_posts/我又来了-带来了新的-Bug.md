---
title: '我又来了, 带来了新的 Bug'
date: 2017-02-13 00:45:49
tags:
  - qt
  - debug
  - 社区
categories:
  - 技术
  - Qt
  - Debug
---

这并不是一次复杂 Debug 过程, 也不是一篇充满干货的记录, 但这是我生平第一次为开源社区提交了我发现的 Bug, 具有象征性的意义. 鼓励着我融入这个理想主义大家庭, 贡献自己微薄的力量.

<!--more-->
![](title_bg.jpg)

# Bug! Bug! We had a Bug!
Qt 作为一个开发框架, 跨平台是它最值得称道的特性之一. 自然我的程序会有 Windows 平台的用户, 平常我在 Mac 下开发, 到了发布版本的时候, 就需要一台 Windows 开发机器来完成我的发布工作. 接上回我发布了程序的 Mac 版本之后, 我开始了我的 Windows 平台工作流程.

- 开机, 更新 Qt 5.8, Pull 最新的仓库代码, So Far So Good ... 嗯哼, 让我们编译看看...
- 嗯哼? 下载的 Qt 版本和 MSVC 版本对不上? 没关系, 安装 MSVC 2015, 继续编译...
- 嗯哼? Qt 找不到 MSVC 2015 的编译器? 当初安装 2013 的时候可不是这样... let's see...
- 嗯哼? 原来是 VisualStudio 14 开始就不默认安装 C++ 语言环境了...
- 好办, 打开 VisualStudio Repair 一下, 选择 Language 选项, 补充安装一下 C++ 语言环境...
- 好的, 看来差不多了, 接下来就是见证奇迹的时刻, 编译走起... balabala...
- **我靠**, 冒红字了...

# Debug
> the kit has configuration issues which might be the root cause for this problems

*构建套件有些配置问题可能是这个问题的主要原因*

![WTF?](黑人问号脸.jpg)

这啥错误, 没关系直接丢 Google, 看看 StackOverFlow 上的那群家伙怎么说的. 但是经过一番查找并没有找到任何有价值的信息. 大概是错误比较新, 或者是描述得太笼统了. 
回归 Debug, 记住大学老师姚仰光说过的, 永远从你遇到的第一个错误开始找起. 回头翻翻翻, 我看到了这么一个提示:
```
moc: Too many input files specified: 'Code/xxx/moc_predefs.h' '..\xxx\src\utilities\http_session.h'
```
我天, 我可不记得有 Code 这么个文件夹, Program Code 倒是有一个, 这么说来问题就水落石出了. Qt 另一个值得称道的 MOC(元对象编译器) 程序 出了一个粗心的纰漏: 没有对 --include 参数获得的构建路径做 quote 处理, 导致在遇到带空格的路径时, 将路径分成了两个部分, 前半部分成为了一个废路径, 后半部分成为了一个废指令. 想想这是一个 Release 版本, 看起来问题还蛮严重的嘛. 
于是我更换了一个不带空格的路径, 经过验证, 果然编译成功.

# Bug Report
哦吼吼? 这么说来我又发现一个 Bug.
经过了上次查找 Bug 的经验, 我决定去 [Qt 官方 Bug 汇报平台](https://bugreports.qt.io) 上先看看有没有其他开发者和我遇到了同样的问题, 结果是没有, 看来大家基于经验, 都很谨慎地选择了保送的路径. 那么这个 Bug Report 的看来是要交给我来完成啦.
人生的第一个 Bug Report 就这样诞生了: [QTBUG-58764](https://bugreports.qt.io/browse/QTBUG-58764)

# The End
再一次赞一下 Qt 官方, 获得 Bug 以后就迅速做出相应, 非常及时地就进入到 Code Review 的环节, 连补丁都有了. 不过我使用的是发布版本, 暂时也不考虑折腾编译 Qt 这件事儿, 所以可能只能在下一个版本获得这个解决啦.


