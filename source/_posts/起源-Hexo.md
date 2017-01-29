---
title: 起源:Hexo
tags: 
- hexo
- 博客
- markdown
categories: 
- 技术
- 博客
- 网站历史
date: 2017-01-29 00:24:55
---

[Hexo](https://hexo.io/) 是一个基于 Node.Js 实现的快速、简洁且高效的博客框架. 框架搭建便捷, 能够高效生成静态博客, 通过简易的配置就可以实现博客的部署与更新, 配合 Markdown 语法的文本, 让你更加专注于内容本身. 

<!--more-->

昨天是除夕, 在 Linode 上购买了人生第一台 VPS, 将自己的域名 memphis.wang 解析到了这个自己的专属 IP 下, 并且在上面部署了一台 CentOS 7, 更新了 yum 源 EPEL, 更新了 4.9 的内核, 换上了 Google 新推出的 BBR 拥塞控制算法. 最重要的是搭起了一个真正属于自己的 SS 服务器, 这感觉真的是太棒了. 原来一切近在眼前的技术, 只是止步于我不去实现. 工作太忙算得上一个客观原因, 但是主观上我并没有推动这件事发生, 导致其迟到了一年. 

称热打铁, 今天赶在大年初一, 一便将自己的个人博客搭建起来. 选择 Hexo 并没有太多的考虑, 因为看到 [子龙山人的这篇博客](https://zilongshanren.com/blog/2015-08-02-migrate-blog-to-hexo.html) 并且使用我钟爱的命令行操作的方式来构建, 部署, 编辑博客的方式. 便不再去考虑其他的选项了. 有时候瞎折腾会忘记的出发的目的, 如果有前辈趟好了坑, 就照着这个坑快速前进就对了!

### Hexo 的安装
官网描述得简介明了, 并且会得到及时的更新, 请关注官网的说明.

### 隐形的坑
值得注意的有如下几点:
- Hexo 的 git 部署器的安装指令已经更新为 `npm install hexo-deployer-git --save`, 如果你根据旧博客的指令, 而不是官网最新指令的话, 执行 `hexo deploy` 之后会得到 `ERROR Deployer not found: git` 错误.
- `hexo init` 会覆盖当前目录, **导致 Git 仓库消失**, 请尽量针对空目录初始化 hexo 仓库.

### Hexo 的工作方式
作为用户主要关注这么几个内容:
- Hexo 提供了配置文件 \_config.yml 让用户可以修改博客的相关设置. 例如名称, 简介, 主题, 每页显示文章数等. 
- 使用 `hexo new` 指令创建新的博客, 你也可以手动创建. 博文的头部中有相关的参数如 Tag, Category 可以设置.
- 使用 `hexo generate` 生成静态的网页文件到 public/ 文件夹中, 这就是你最终用于部署到服务器上的博客内容.
- 使用 `hexo deploy` 将生成的博客部署到指定的服务器. 有多种方式选择. 新时代的我们当然用 Git 啦!

### Hexo 仓库管理最佳实践
Hexo 项目在 `npm install` 之后, 在根目录下自动生成了一个 .gitignore 文件, 以此来忽略 public 等文件夹, 这说明 Hexo 项目本身也推荐我们使用 Git 来管理, 而且实际上也非常的适合使用. 再结合 Github 推出的 [GitPages](https://pages.github.com/) 功能, 我们可以将我们的博客部署到我们的 GitPages 上. 由于 GitPages 本身会占用掉我们的 Github 一个独立而且特殊的仓库 **username.github.io**, 看起来只能将 hexo 源文件和 hexo 生成的博客内容放在不同的 Git 仓库中进行管理了. 但是其实换一个思路, 我们可以在 **username.github.io** 这个仓库中创建两个完全独立不相干的分支, master 分支用于博客内容的部署, GitPage 会读取这部分内容进行网页的展示, Hexo 分支用于管理 Hexo 项目的源文件, 这样一个仓库就解决了源码和展示两个问题了!

假如更换了陌生的环境, 比如在新买的服务器上需要还原博客的书写环境, 只需要重新安装 Hexo, 找一个目录 clone 自己的 **username.github.io** 重新 `npm install` 就可以了(不需要重新 init, 因为仓库上已经初始化过了)


#### 附文章发布时, 博主的 Hexo 环境
- hexo: 3.2.2
- hexo-cli: 1.0.2
- os: Darwin 16.3.0 darwin x64
- http_parser: 2.5.0
- node: 4.0.0
- v8: 4.5.103.30
- uv: 1.7.3
- zlib: 1.2.8
- ares: 1.10.1-DEV
- modules: 46
- openssl: 1.0.2d
