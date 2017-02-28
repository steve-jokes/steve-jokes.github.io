---
title: 拥抱 Git
tags:
  - git
  - githug
  - 安利
  - 技术
  - 教学
categories:
  - 技术
  - 教学
date: 2017-03-01 01:39:02
---


Git 无疑是当今互联网最酷, 最炙手可热的版本管理系统, 没有之一. 由 Linux 之父 Linus 一手缔造, 其由来也一如 Linux 一般传奇. 如果你还未了解它, 请参看迟建强的一篇文章 [Linus，一生只为寻找欢笑](https://zhuanlan.zhihu.com/p/19796979). 如果你想要学习它, 推荐从廖雪峰的 [Git 教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000) 入门, 然后再通过阅读书籍等途径深入学习.

今天我要安利的并非 Git, 而是 **Githug**. [Githug](https://github.com/Gazler/githug) 是 [Gary Rennie](https://github.com/Gazler) 在 Github 上建立的一个开源项目. 是一个基于 Git 的闯关小游戏, 将许多的应用场景提炼成关卡, 用来练习你的 Git 技能.

<!--more-->
![embrace git](git-home-hero.jpg)

如何获取和进行游戏请自行阅读项目的 README.md 文件, 不赘述了, 这是程序员的必备技能. 

# 精华
对于 Githug 我想总结的其中的部分题目. 他们真实地反应了实际工作中的一些棘手的应用场景:

## git commit --amend
工作过程中难免会遇到一个 Bug 来回修改的情况, 有时你以为解决了, 转眼 QA 告诉你还有一些问题, 可是你的改动已经 commit 过了, 反反复复几次以后, 你可能会有好几个 commit 都在解决同一个问题. -m 的信息都一模一样. 当然你可以在最后通过 rebase 合并这几个 commit, 不过最简单的办法应该是每次更新都使用 `git commit -- amend` 将新的修复加入到前一次 commit 中, 不需要额外有其他工作量.

## git cherry-pick
有时候你从 develop 分支切了几个不同的 feature 分支出来开发不同的功能, 这些 feature 不一定都能最终完成, 可能偶尔需要 放弃某些 feature. 可是这个 feature 中的某一个 commit 原计划需要提交到 develop. 删了这个废弃的分支这些 commit 就得重写, 又不能合并这个分支, 因为有太多废弃的代码. 这时候你可以使用 `git cherry-pick` 来从 feature 分支中挑出那个 commit 合并到 develop 分支来. 就算是处于中间的 commit 也可以哦!

## git rebase -i
这个指令可以做到很多事情: 重新编辑 commit 名称, 调整 commit 顺序, 合并多个 commit. 只要你的本地分支还没有 push 的远端仓库, 一切都来得及! 你可以从容地整理你已经提交的 commit, 满意之后, 再 push 到远程仓库. 不过一旦 push 以后就不推荐这么做了, 可能会引起一些冲突.

## git add -p
如果你平常提交代码使用的是 sourcetree, 一定对它的 stage lines 功能印象深刻, 它让你能够 add 一份文件中的部分代码到 stage area, 是一项很酷炫的功能. 但你可能不了解的是, 这个功能不是 sourcetree 开发的, 而是 git 天然支持的. 像我这样的命令行党就有福音了! 再也不用离开我的 terminal.

## git revert
人呐, 总有提交错代码的时候. 如果代码提交到了远程仓库, 你最好不要去修改已存在的 commit 记录, 免得引起冲突. 这时候你需要一条 `git revert` 选择你需要反转的记录, 将它所做的改动撤销掉. 注意这个办法是创建了一次新的 commit, 只是刚好和你指定的 commit 操作相反而已. 

## submodule
子仓库通常是用来分开管理仓库代码, 降低耦合. 减少对不必要的仓库的下载用的. 这里着重介绍它, 是符合我国国情的: 
- Gitbug 几乎是绝大部分团队首选的 Git 服务器, 同时带有技术社交基因, 有着程序员的第一张名片之称. 出于通用, 便捷和安全的考虑, 我们通常将代码托管在 Github 上. Git 的一大优点是快! 而由于众所周知的原因, 国内网络对 Github 的访问差强人意.
- Git 天生的属性使得它易于管理文本文件的版本, 不建议将二进制大文件加入仓库. 于是 Github 推出的 git lfs 应运而生, 用于承担对仓库内大文件的支持, 惨绝人寰的是, 我们对 git lfs 使用的 aws 服务器的访问连通性更差, 而且 lfs 本身还有自身的挺多问题. 

当仓库中需要许多二进制大文件时, 一个简单的 clone 就可能耗费掉你半天的时间(_渣渣网速+可能断掉的风险_). 出于这两点考虑, 我的建议是. 将与保密性无关的二进制大文件托管在国内的 [Coding](https://coding.net/) 上, 将其作为子仓库加入到代码仓库中. 这样 clone 的时候, 速度就和飞起来一样!

# 总结
Githug 这个游戏本身不难, 在有 git 基础的情况下通关只需要一个晚上, 对照攻略会更快. 带给你的确实游戏闯关的极致体验, 和许多启发性质的 git 技能, 相信能够在我们的 coding 生活中带来极大的便利.
